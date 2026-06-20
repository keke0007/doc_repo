# MySQL 调优知识点梳理

> 基于 `mysql.md` 整理。重点覆盖 MySQL 架构、索引原理、慢查询定位、EXPLAIN、JOIN/order by 优化、索引案例与索引失效场景。涉及多组件调用或执行链路的内容使用 ASCII 流程图说明。

---

## 目录

1. [MySQL 架构设计](#一mysql-架构设计)
2. [存储引擎](#二存储引擎)
3. [索引基础与数据结构](#三索引基础与数据结构)
4. [InnoDB 与 MyISAM 索引实现](#四innodb-与-myisam-索引实现)
5. [回表、覆盖索引与联合索引](#五回表覆盖索引与联合索引)
6. [慢查询与 EXPLAIN](#六慢查询与-explain)
7. [JOIN、in/exists 与 order by 优化](#七joininexists-与-order-by-优化)
8. [索引优化案例](#八索引优化案例)
9. [索引优化原则与失效场景](#九索引优化原则与失效场景)

---

## 一、MySQL 架构设计

### 1.1 Server 层与 Engine 层

MySQL 逻辑架构主要分为两层：

- **Server 层**：负责连接管理、权限认证、SQL 解析、优化、执行调度等通用能力。
- **Engine 层**：负责具体的数据存储、索引组织、事务、锁、崩溃恢复等底层能力。

### 1.2 一条 SQL 的执行流程

```text
+-------------+
|   Client    |
+------+------+ 
       |
       | SQL
       v
+------+------+
|  Connector  |  建立连接 / 认证 / 读取权限 / 维护连接
+------+------+
       |
       v
+------+------+
| Query Cache |  MySQL 8.0 已移除；5.x 命中则直接返回
+------+------+
       |
       v
+------+------+
|  Analyzer   |  词法分析 / 语法分析 / 识别表和字段
+------+------+
       |
       v
+------+------+
| Optimizer   |  选择索引 / JOIN 顺序 / 生成执行计划
+------+------+
       |
       v
+------+------+
|  Executor   |  权限检查 / 调用存储引擎接口
+------+------+
       |
       v
+------+------+
|   Engine    |  访问数据页 / 索引树 / 返回记录
+------+------+
       |
       v
+------+------+
|   Client    |
+-------------+
```

### 1.3 关键组件

- **连接器 Connector**
  - 负责客户端连接、身份认证、权限读取、连接维护。
  - 一个系统通常会与 MySQL 建立多个长连接，MySQL 通过连接管理和线程处理请求。
  - 常用命令：`show processlist` 查看连接状态。

- **查询缓存 Query Cache**
  - 按 SQL 字符串作为 key 缓存结果。
  - 任意表更新会使该表相关缓存失效，失效频繁，通常收益低。
  - MySQL 8.0 已删除查询缓存功能。

- **分析器 Analyzer**
  - 词法分析：识别 `select`、表名、字段名、条件等 token。
  - 语法分析：判断 SQL 是否符合 MySQL 语法。

- **优化器 Optimizer**
  - 决定使用哪个索引、表连接顺序、是否将子查询改写为连接等。
  - 优化器输出的是执行计划，此时还没有真正访问表数据。

- **执行器 Executor**
  - 执行前检查权限。
  - 根据表的存储引擎调用对应 Engine 接口。
  - 无索引时逐行扫描；有索引时先走索引树，必要时回表。

---

## 二、存储引擎

### 2.1 常见引擎对比

| 引擎 | 核心特点 | 适用场景 |
| --- | --- | --- |
| InnoDB | 默认引擎；支持事务、行锁、外键、崩溃恢复；聚簇索引 | 绝大多数 OLTP 业务、频繁更新删除、事务一致性要求高 |
| MyISAM | 查询和插入快；不支持事务；表锁；数据文件和索引文件分离 | 读多写少、事务要求低的旧场景 |
| MEMORY | 数据在内存中；速度快；安全性低；不支持 BLOB/TEXT | 临时表、小数据量高速访问 |

### 2.2 InnoDB 重点

- 支持 ACID、提交、回滚、崩溃恢复。
- 支持行级锁和外键。
- 维护 Buffer Pool 缓存数据页和索引页。
- 表数据按主键组织，主键索引的叶子节点保存整行数据。
- 如果没有主键，会选择第一个非空唯一索引；再没有则生成 6 字节隐式 `ROWID`。

### 2.3 MyISAM 重点

- `.frm` 存表定义，`.MYD` 存数据，`.MYI` 存索引。
- 主键索引和普通索引的叶子节点都保存数据行的磁盘地址。
- 不支持事务，锁粒度主要是表锁。

---

## 三、索引基础与数据结构

### 3.1 索引是什么

索引是帮助 MySQL 高效获取数据的数据结构。可以理解为书的目录：不需要逐页扫描，而是通过有序结构快速定位目标数据。

### 3.2 索引类型

- **主键索引**：唯一、非空，一张表通常只有一个主键。
- **普通索引**：提升普通字段查询效率，不保证唯一。
- **联合索引**：多个字段组成一个索引，遵循最左前缀原则。
- **唯一索引**：索引列值唯一，可允许 `NULL`，具体行为与数据库实现相关。
- **全文索引**：面向文本检索。

### 3.3 索引优缺点

优点：

- 提高查询效率，降低扫描行数。
- 可用于排序、分组，减少 `filesort` 和临时表。
- 覆盖索引可避免回表。

缺点：

- 占用额外磁盘空间。
- 写入、更新、删除时需要维护索引树。
- 索引过多会增加优化器选择成本和维护成本。

### 3.4 为什么 MySQL 常用 B+ 树

| 数据结构 | 特点 | 局限 |
| --- | --- | --- |
| Hash | 等值查询快 | 不适合范围查询、排序、前缀匹配 |
| 二叉树 | 结构简单 | 可能退化为链表 |
| 平衡二叉树 | 保持平衡 | 树高较高，磁盘 IO 次数多 |
| B 树 | 多路平衡，降低树高 | 非叶子节点也存数据，范围扫描不如 B+ 树 |
| B+ 树 | 非叶子节点只存 key，叶子节点链表相连 | 更适合磁盘页、范围查询和顺序扫描 |

---

## 四、InnoDB 与 MyISAM 索引实现

### 4.1 InnoDB 主键索引

InnoDB 使用聚簇索引组织数据。主键索引叶子节点保存整行记录。

```text
InnoDB 主键等值查询: select * from user where id = 28

+-------------+
| Root Page   |
+------+------+
       |
       | 比较 key, 选择子节点
       v
+------+------+
| Branch Page |
+------+------+
       |
       | 继续比较
       v
+------+------+
| Leaf Page   |  叶子节点保存整行数据
+------+------+
       |
       v
+------+------+
| Return Row  |
+-------------+
```

### 4.2 InnoDB 辅助索引与回表

辅助索引叶子节点保存的是主键值，不保存完整行。查询非索引列时，需要再通过主键索引查整行，这就是回表。

```text
select * from user where age = 19

+--------------------+
| Secondary Index    |  idx_age
| leaf: age -> id    |
+---------+----------+
          |
          | 找到主键 id
          v
+---------+----------+
| Clustered Index    |  PRIMARY
| leaf: id -> row    |
+---------+----------+
          |
          v
+---------+----------+
| Full Row Result    |
+--------------------+
```

### 4.3 InnoDB 联合索引

联合索引如 `(a, b, c)` 的排序规则是：

1. 先按 `a` 排序。
2. `a` 相同再按 `b` 排序。
3. `a`、`b` 都相同再按 `c` 排序。

因此 `(a, b, c)` 可以等价支持 `(a)`、`(a, b)`、`(a, b, c)` 三种前缀查询，但不能直接高效支持跳过 `a` 的查询。

```text
idx_abc(a,b,c)

Global order:
  a ordered
  |
  +-- within same a: b ordered
      |
      +-- within same a,b: c ordered

可用:
  where a = ?
  where a = ? and b = ?
  where a = ? and b = ? and c = ?

通常不可用:
  where b = ?
  where b = ? and c = ?
```

### 4.4 MyISAM 索引

MyISAM 数据文件和索引文件分离，索引叶子节点保存行记录的磁盘地址。主键索引和辅助索引结构基本一致。

```text
MyISAM 查询流程

+-------------+       +-------------+
|  .MYI Index | ----> | Row Address |
+------+------+       +------+------+
       |                     |
       |                     v
       |              +------+------+
       +------------> |  .MYD Data  |
                      +------+------+
                             |
                             v
                      +------+------+
                      | Return Row  |
                      +-------------+
```

---

## 五、回表、覆盖索引与联合索引

### 5.1 回表查询

回表发生在 InnoDB 辅助索引查询中：辅助索引只能拿到主键值，如果 `select` 的字段不在辅助索引中，就要回到主键索引查整行。

### 5.2 覆盖索引

覆盖索引不是一种新索引，而是一种查询状态：查询所需字段都能从同一棵索引树中拿到，不需要回表。

示例：

```sql
-- 原索引: (name)
select id, name, sex from user where name = 'zhangsan';

-- 优化为联合索引: (name, sex)
-- 因 InnoDB 二级索引叶子节点天然包含主键 id
-- id/name/sex 都能从索引中获取，减少回表
```

```text
未覆盖索引:

idx_name -> id -> PRIMARY -> row -> return

覆盖索引:

idx_name_sex -> id,name,sex -> return
```

### 5.3 联合索引创建原则

- 高频查询字段优先。
- 区分度高的字段优先。
- 尽量满足最左前缀。
- 能形成覆盖索引时，可把高频返回字段放入联合索引。
- 多个单列索引能合并成一个高价值联合索引时，优先考虑联合索引。

---

## 六、慢查询与 EXPLAIN

### 6.1 性能优化基本路径

```text
+-------------------+
| Enable Slow Log   |
+---------+---------+
          |
          v
+---------+---------+
| Find Slow SQL     |
+---------+---------+
          |
          v
+---------+---------+
| EXPLAIN SQL       |
+---------+---------+
          |
          v
+---------+---------+
| Read type/key/rows|
| Extra/key_len     |
+---------+---------+
          |
          v
+---------+---------+
| Rewrite SQL or    |
| Add/Adjust Index  |
+---------+---------+
          |
          v
+---------+---------+
| Verify Again      |
+-------------------+
```

### 6.2 慢查询日志

常用参数：

- `slow_query_log`：是否开启慢查询日志。
- `slow_query_log_file`：慢查询日志文件路径。
- `long_query_time`：超过多少秒记为慢查询。
- `log_queries_not_using_indexes`：是否记录未使用索引的查询。

临时开启：

```sql
set global slow_query_log = on;
set global long_query_time = 1;
```

慢查询日志重点字段：

- `Query_time`：SQL 执行耗时。
- `Lock_time`：Server 层等待表锁时间。
- `Rows_sent`：返回行数。
- `Rows_examined`：扫描行数，越大越需要关注。

### 6.3 EXPLAIN 关键字段

| 字段 | 作用 | 关注点 |
| --- | --- | --- |
| `id` | 查询执行顺序 | id 越大优先级越高；相同则从上到下 |
| `select_type` | 查询类型 | SIMPLE、PRIMARY、SUBQUERY、DERIVED、UNION |
| `table` | 当前访问表 | 判断驱动表和被驱动表 |
| `type` | 访问类型 | 至少达到 range，最好 ref 及以上 |
| `possible_keys` | 可能使用的索引 | 有候选不代表实际使用 |
| `key` | 实际使用的索引 | `NULL` 代表未使用索引或索引失效 |
| `key_len` | 使用索引长度 | 判断联合索引是否充分利用 |
| `ref` | 索引匹配值来源 | const 或其他表字段 |
| `rows` | 预估扫描行数 | 越小越好 |
| `filtered` | 过滤后比例 | 结合 rows 判断结果集大小 |
| `Extra` | 额外执行信息 | 重点看 filesort、temporary、Using index 等 |

### 6.4 type 访问类型

常见性能从好到差：

```text
system > const > eq_ref > ref > range > index > ALL
```

- `const`：主键或唯一索引等值命中，最多一行。
- `eq_ref`：连接中使用主键或唯一索引，每个驱动表行只匹配一行。
- `ref`：非唯一索引等值匹配。
- `range`：范围扫描，如 `between`、`>`、`in`、`like 'abc%'`。
- `index`：扫描整棵索引树，比全表扫描轻一些。
- `ALL`：全表扫描，通常需要优化。

### 6.5 Extra 常见值

| Extra | 含义 | 优化方向 |
| --- | --- | --- |
| `Using filesort` | 额外排序 | 给排序字段建立合适索引，或调整排序字段顺序 |
| `Using temporary` | 使用临时表 | 优化 group by/order by，减少中间结果 |
| `Using where` | Server 层还要过滤 | 结合 type 判断是否需要补索引 |
| `Using index` | 覆盖索引 | 通常是好现象 |
| `Using join buffer` | JOIN 使用连接缓存 | 关联字段补索引，缩小驱动表 |
| `Using index condition` | 索引条件下推，但仍可能回表 | 检查联合索引和返回字段 |

---

## 七、JOIN、in/exists 与 order by 优化

### 7.1 JOIN 驱动表原则

驱动表是多表关联时第一个被处理的表。原则：在不影响结果的前提下，优先让小结果集驱动大结果集。

### 7.2 三种 JOIN 算法

#### Simple Nested-Loop Join

```text
for row in outer_table:
    for row in inner_table:
        compare join condition
```

```text
+-------------+       scan all        +-------------+
| Outer Table | --------------------> | Inner Table |
+------+------+                       +------+------+
       |                                     |
       | N rows                              | M rows each time
       v                                     v
Cost ~= N * M comparisons
```

#### Index Nested-Loop Join

被驱动表关联字段有索引时，内层不再全表扫描，而是走索引定位。

```text
+-------------+       key lookup      +------------------+
| Outer Table | --------------------> | Inner Table Index |
+------+------+                       +--------+---------+
       |                                       |
       v                                       v
Cost ~= outer rows * index height        matching rows
```

#### Block Nested-Loop Join

关联字段无索引时，MySQL 使用 `join_buffer` 批量缓存驱动表数据，减少被驱动表扫描次数。

```text
+-------------+       put join cols       +-------------+
| Outer Table | ------------------------> | Join Buffer |
+------+------+                           +------+------+
       |                                         |
       | batch compare                           v
       |                                  +------+------+
       +--------------------------------> | Inner Table |
                                          +-------------+
```

JOIN 优化要点：

- 小结果集驱动大结果集。
- 给被驱动表的关联字段建索引。
- 减少 `select *`，让 `join_buffer` 能容纳更多行。
- 必要时调整 `join_buffer_size`，但不能盲目调大。

### 7.3 in 与 exists

`in` 适合子查询结果集较小、主查询表较大且主查询字段有索引的场景。

```text
IN:

subquery small table -> cache ids
          |
          v
main big table checks whether key in cached ids
```

`exists` 适合主查询结果集较小、子查询表较大且子查询关联字段有索引的场景。

```text
EXISTS:

outer small table row
          |
          v
subquery checks existence by indexed key
          |
          v
true -> keep outer row
```

记忆：

- `in` 后面跟小表。
- `exists` 后面跟大表。

### 7.4 order by 优化

MySQL 排序方式：

- **索引排序**：利用 B+ 树有序性直接返回，避免额外排序。
- **额外排序 filesort**：无法利用索引顺序，需要额外排序。

#### 全字段排序

```text
where index -> primary key -> fetch full selected fields
          |
          v
put all selected fields into sort_buffer
          |
          v
sort by order column
          |
          v
return result
```

优点：排序后可直接返回。缺点：占用内存大。

#### rowid 排序

```text
where index -> primary key
          |
          v
put order column + id into sort_buffer
          |
          v
sort
          |
          v
use sorted ids to fetch full rows
          |
          v
return result
```

优点：占用内存小。缺点：排序后还要回表。

order by 索引使用规则：

- 排序字段顺序要匹配联合索引顺序。
- 排序方向最好一致，全部 ASC 或全部 DESC。
- 查询字段若能被索引覆盖，更容易避免回表。
- where 中范围查询可能导致范围字段右侧索引无法继续用于排序。
- 排序字段分散在多个索引中，通常不能合并利用排序。

---

## 八、索引优化案例

### 8.1 单表：LIKE + ORDER BY

需求：查询名字包含“李”的用户姓名和手机号，并按 `user_id` 排序。

问题：

- `LIKE '%李%'` 左侧通配符导致普通前缀索引难以定位。
- `ORDER BY user_id` 可能触发 `Using filesort`。

优化思路：

```sql
ALTER TABLE user_contacts ADD INDEX idx_unm(user_id, name, mobile);
```

效果：

- 通过联合索引覆盖 `name`、`mobile`。
- `user_id` 放在前面满足排序，减少 `Using filesort`。

### 8.2 单表：手机号前缀统计

需求：统计手机号以 135、136、186、187 开头的用户数量。

优化：

```sql
ALTER TABLE user_contacts ADD INDEX idx_m(mobile);
```

原因：

- `mobile LIKE '135%'` 属于右侧通配符，可走范围索引。
- 单独索引比不合适的联合索引更直接。

### 8.3 单表：日期函数导致索引失效

反例：

```sql
WHERE DATE_FORMAT(create_date, '%Y-%m-%d') = '2017-02-16'
```

问题：在索引列上使用函数，索引失效。

优化：

```sql
WHERE create_date BETWEEN '2017-02-16 00:00:00'
                      AND '2017-02-16 23:59:59'
```

### 8.4 深分页优化

反例：

```sql
SELECT * FROM user_contacts LIMIT 100000, 100;
```

问题：需要扫描并丢弃大量前置记录。

优化方向：

```sql
SELECT * FROM user_contacts
WHERE id >= 100001
LIMIT 100;
```

或先定位偏移位置的主键，再基于主键范围查询。

```text
deep limit:
scan 100000 rows -> discard -> return 100

id range:
locate start id -> range scan by primary key -> return 100
```

### 8.5 多表 JOIN 优化

LEFT JOIN 中左表通常是驱动表，右表是被驱动表。优化关联查询时，优先给被驱动表的关联字段加索引。

```sql
ALTER TABLE ugncy_cntct_psn ADD INDEX idx_userid(user_id);
```

```text
mob_autht ma                  ugncy_cntct_psn ucp
driver table                  driven table
     |                              ^
     | ma.user_id = ucp.user_id     |
     +------------------------------+
                         index on ucp.user_id
```

### 8.6 低区分度字段不适合单独建索引

如 `audit_mod_cde` 只有“人工/智能”两个值，单独建索引可能扫描大量记录并回表，收益低。

优化方向：

- 结合时间范围等高区分条件。
- 创建更符合查询模式的联合索引。
- 避免只因为字段出现在 where 中就建索引。

---

## 九、索引优化原则与失效场景

### 9.1 全值匹配

联合索引 `(user_name, user_age, user_level)` 中，条件覆盖越完整，索引利用越充分。

```sql
WHERE user_name = 'tom'
WHERE user_name = 'tom' AND user_age = 17
WHERE user_name = 'tom' AND user_age = 17 AND user_level = 'A'
```

### 9.2 最左前缀法则

联合索引必须从最左列开始连续使用。

```text
idx_nal(user_name, user_age, user_level)

Good:
  user_name
  user_name + user_age
  user_name + user_age + user_level

Bad:
  user_age
  user_age + user_level
```

注意：where 条件书写顺序可能被优化器重排，但建索引和理解执行计划时仍应按联合索引顺序分析。

### 9.3 不要在索引列上做计算或函数

反例：

```sql
WHERE LEFT(user_name, 6) = '112233'
WHERE DATE_FORMAT(create_date, '%Y-%m-%d') = '2017-02-16'
```

原因：索引树保存的是原始值，函数计算后的值无法直接按索引顺序定位。

### 9.4 范围之后右侧列可能失效

联合索引中遇到范围查询后，右侧列通常无法继续用于精确定位。

```sql
-- idx_nal(user_name, user_age, user_level)
WHERE user_name = 'tom'
  AND user_age > 17
  AND user_level = 'A'
```

这里 `user_level` 可能无法充分利用索引。

### 9.5 尽量使用覆盖索引

少用 `select *`，查询列尽量和索引列一致。

```sql
-- 更容易 Using index
SELECT user_name, user_age, user_level
FROM users
WHERE user_name = 'tom'
  AND user_age = 17
  AND user_level = 'A';
```

### 9.6 不等于可能导致索引失效

```sql
WHERE user_name != 'tom'
WHERE user_name <> 'tom'
```

不等于通常需要扫描大量数据，优化器可能放弃索引。

### 9.7 is null / is not null

`IS NOT NULL` 往往选择性差，容易全表扫描。是否能用索引取决于字段定义、数据分布和优化器成本估算，不能只看语法。

### 9.8 like 通配符位置

```text
LIKE 'tom%'   可能使用索引，按前缀范围查找
LIKE '%tom'   通常不能使用索引定位
LIKE '%tom%'  通常不能使用索引定位
```

原因：

```text
B+Tree order is based on prefix

'tom%'  -> can find prefix range
'%tom'  -> suffix has no global order in index
'%tom%' -> middle match has no global order in index
```

如果查询列被覆盖索引完全覆盖，即使不能范围定位，也可能从全表扫描变成全索引扫描，减少回表成本。

### 9.9 字符串不加引号

反例：

```sql
WHERE user_name = 123
```

如果 `user_name` 是 varchar，可能触发隐式类型转换，导致索引失效。应写为：

```sql
WHERE user_name = '123'
```

### 9.10 少用 or

`or` 连接条件可能导致索引失效或优化器选择全表扫描。可根据场景考虑：

- 改写为 `union all`。
- 为每个分支设计合适索引。
- 使用 `in` 替代同字段多个等值条件。

---

## 十、调优检查清单

- 是否开启慢查询并定位到真实慢 SQL。
- EXPLAIN 中 `type` 是否至少达到 `range`，最好达到 `ref`。
- `key` 是否为预期索引，`key_len` 是否说明联合索引用足。
- `rows` 是否明显偏大。
- `Extra` 是否出现 `Using filesort`、`Using temporary`、`Using join buffer`。
- JOIN 是否小结果集驱动大结果集。
- 被驱动表关联字段是否有索引。
- where 条件是否对索引列做函数、计算或隐式类型转换。
- 联合索引是否满足最左前缀。
- 范围查询是否截断了联合索引右侧字段。
- order by 字段顺序和方向是否匹配联合索引。
- 查询列是否能被覆盖索引覆盖。
- 低区分度字段是否被误建为单列索引。
- 深分页是否能改为基于主键或游标的范围查询。
