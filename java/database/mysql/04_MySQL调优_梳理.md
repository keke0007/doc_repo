# MySQL 调优完整梳理

## 目录
1. MySQL 架构与执行流程
2. InnoDB 存储引擎深度理解
3. 索引原理与优化
4. SQL 执行计划分析（EXPLAIN）
5. 查询优化策略
6. 事务与锁机制
7. 性能调优参数
8. 常见问题速查表

---

## 第一部分：MySQL 架构与执行流程

### 1.1 MySQL 整体架构

MySQL 采用 **Server 层 + Engine 层** 的架构设计：

```
Client
  ↓
连接器 (Connector) ─→ TCP 连接池 / 线程池
  ↓
查询缓存 (Query Cache) [MySQL 8.0+ 已删除]
  ↓
分析器 (Analyzer) ─→ 词法分析 + 语法分析
  ↓
优化器 (Optimizer) ─→ 执行计划生成
  ↓
执行器 (Executor) ─→ 权限检查 + 调用存储引擎接口
  ↓
InnoDB / MyISAM 等存储引擎
  ↓
磁盘 / 内存
```

### 1.2 SQL 执行链路（重点流程图）

```
┌─────────────────────────────────────────────────────────────────┐
│                  SELECT * FROM user WHERE id = 1                │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                    ┌──────▼──────┐
                    │  连接器      │ 建立连接、权限验证、获取权限
                    │ Connector   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  查询缓存    │ 检查是否命中缓存
                    │Query Cache  │ (MySQL 8.0+ 已无此阶段)
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  分析器      │ • 词法分析：识别关键字、表名
                    │  Analyzer   │ • 语法分析：检查 SQL 合法性
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  优化器      │ • 表关联顺序
                    │ Optimizer   │ • 索引选择
                    │             │ • 外连接转内连接等
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  执行器      │ • 权限校验
                    │ Executor    │ • 根据执行计划调用引擎接口
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
      ┌────▼────┐     ┌────▼────┐    ┌───▼───┐
      │ InnoDB  │     │MyISAM   │    │Memory │
      │ (行锁)   │     │(表锁)   │    │(内存) │
      └────┬────┘     └────┬────┘    └───┬───┘
           │               │             │
           └───────────────┼─────────────┘
                           │
                    ┌──────▼──────┐
                    │ 返回结果集  │
                    └─────────────┘
```

### 1.3 连接器工作原理

| 功能 | 说明 |
|------|------|
| **TCP 连接** | 完成 TCP 三次握手，建立长连接 |
| **权限验证** | 基于用户名密码验证身份 |
| **权限缓存** | 将该连接的权限信息缓存在内存中 |
| **连接池** | 默认 max_connections，复用连接 |
| **查看状态** | `SHOW PROCESSLIST;` 查看连接状态 |

**命令示例：**
```sql
-- 连接 MySQL
mysql -h IP -P PORT -u user -p password

-- 查看当前连接
SHOW PROCESSLIST;
-- 或
SHOW FULL PROCESSLIST;

-- 参数说明
-- Id: 连接 ID
-- User: 用户名
-- Command: Sleep/Query/Connect 等
-- Time: 连接持续时间
-- State: 当前状态
```

---

## 第二部分：InnoDB 存储引擎深度理解

### 2.1 InnoDB 内存结构

#### Buffer Pool（缓冲池）

```
┌──────────────────────────────────────┐
│     InnoDB Buffer Pool (默认 128MB)  │
├──────────────────────────────────────┤
│         脏页（Dirty Page）            │ ← 已修改但未刷盘
│       干净页（Clean Page）            │ ← 未修改
│         Free Page（空页）             │ ← 待分配
└──────────────────────────────────────┘
         │      │        │
         │      │        └─ LRU 淘汰策略
         │      └─────────── Write-Ahead Log (WAL)
         └────────────────── 预读机制
```

**关键参数：**
```sql
-- 查看 Buffer Pool 大小
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

-- 推荐设置：物理内存的 50% - 80%
-- 如：16GB 内存，设置为 8GB-12GB

-- 查看 Buffer Pool 使用情况
SHOW ENGINE INNODB STATUS;
```

#### Redo Log（重做日志）

| 作用 | 说明 |
|------|------|
| **持久化保证** | 确保事务持久性 |
| **宕机恢复** | 故障恢复的关键 |
| **循环写** | 固定大小文件，循环覆盖 |
| **顺序写** | 性能优于随机写 |

**Redo Log 物理结构：**
```
ib_logfile0 (固定大小，如 50MB)
ib_logfile1
ib_logfile2
...

写入位置 └─ write_pos (当前写位置)
           └─ checkpoint (可安全覆盖位置)
```

#### Undo Log（回滚日志）

| 用途 | 说明 |
|------|------|
| **事务回滚** | 支持 ROLLBACK |
| **MVCC** | 读取历史版本，实现隔离性 |
| **一致性读** | 快照读的基础 |

#### Change Buffer（更改缓冲）

- **场景**：对非唯一索引的修改
- **作用**：异步合并，提升写性能
- **原理**：先写入 Change Buffer，后异步 merge 到磁盘

#### Adaptive Hash Index（自适应哈希索引）

- 基于访问模式自动创建
- 加速等值查询
- 占用 Buffer Pool 约 5% - 15%

### 2.2 InnoDB 文件结构

```
.ibd 文件（表空间）
├─ Page 0: File Header (文件头)
├─ Page 1: Insert Buffer Bitmap
├─ Page 2: System Page
├─ Page 3-N: 数据页/索引页
│  ├─ File Header (页头)
│  ├─ Page Header
│  ├─ Infimum/Supremum 伪记录
│  ├─ 用户记录 (排序/链式存储)
│  ├─ Free Space (空闲空间)
│  └─ Page Trailer (页尾)
└─ 其他页...

页大小：16KB (innodb_page_size)
```

### 2.3 InnoDB 核心特性

| 特性 | 说明 |
|------|------|
| **事务** | 支持 ACID，提交/回滚 |
| **行锁** | 性能优于表锁 |
| **MVCC** | 多版本并发控制 |
| **外键** | 支持参考完整性 |
| **崩溃恢复** | Redo Log 保证 |
| **聚簇索引** | 主键索引包含完整行数据 |

---

## 第三部分：索引原理与优化

### 3.1 索引数据结构进化

#### Hash 表
```
Key: 索引值
Value: 行数据指针

特点：
✓ 等值查询 O(1)
✗ 范围查询需要全表扫描
✗ 不支持排序
```

#### B 树
```
节点结构：
┌──────────────────┐
│   [10] [20] [30] │ ← 所有节点都存完整行数据
├──────────────────┤
│ ↙ ↓    ↓    ↓  ↖  │
```

**问题**：
- 磁盘 IO 多：树高度大，每个节点数据量小
- 不支持范围查询：叶子节点间无链接

#### B+ 树（MySQL 实际采用）
```
           [20]
         /      \
    [10]          [30][40]
   /    \        /   |   \
[5][10][15][20][25][30][35][40][45]
↓ ←→ ↓ ←→ ↓ ←→ ↓ ←→ ↓ ←→ ↓ ←→ ↓ ←→ ↓

优点：
✓ 非叶子节点只存键值，节点容纳更多键，树更矮
✓ 叶子节点双向链表，支持范围查询
✓ 所有数据都在叶子节点，查询性能稳定
✓ 磁盘 IO 次数少
```

**关键参数计算：**
```
假设：
- 键值 8 字节，指针 8 字节
- 页大小 16KB = 16384 字节
- 每页最多键值数 = 16384 / 16 = 1024

树高度：
- 第 1 层：1 个根页
- 第 2 层：1024 个叶子页 = 100万+ 行数据
- 只需 2 次磁盘 IO！

3 层时可支持 1600 亿+ 数据
```

### 3.2 InnoDB 索引类型

#### 聚簇索引（Clustered Index）

**定义**：主键索引，叶子节点存储完整行数据

```
CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    age INT,
    ...
);
```

**聚簇索引构建规则**：
1. 优先使用显式 PRIMARY KEY
2. 无主键则选第一个 NOT NULL UNIQUE 键
3. 都没有则创建隐式 ROWID（6字节）

**索引结构**：
```
InnoDB 主键索引树
       │
    [id 值]
       │
     叶子节点
    [id | 全部行数据]
```

**等值查询示例**：
```
查询：SELECT * FROM user WHERE id = 100

查询过程：
1. 从根页开始比较
2. 定位到叶子页
3. 找到 id=100，取出完整行数据
4. 返回结果

磁盘 IO 次数：log N (通常 3 次左右)
```

#### 二级索引（Secondary Index）

**定义**：除聚簇索引外的所有索引，叶子节点存主键值

```
CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    age INT,
    INDEX idx_age(age)
);
```

**索引结构**：
```
age 索引树
       │
    [age 值]
       │
     叶子节点
    [age | id (主键)]
     ↓
  需要回表到聚簇索引查询完整数据
     ↓
  聚簇索引树
       │
    [id]
       │
     叶子节点
    [id | 完整行数据]
```

**回表查询示例**：
```
查询：SELECT * FROM user WHERE age = 25

查询过程：
1. 在 age 索引树查找 age=25
2. 得到主键 id 列表 (比如 id=1,3,5,7)
3. 逐个回表到聚簇索引查询完整行数据
4. 返回结果

总磁盘 IO = age 索引 IO + 回表 IO
```

### 3.3 联合索引（组合索引）

**创建示例**：
```sql
CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    age INT,
    level VARCHAR(10),
    INDEX idx_name_age_level(name, age, level)
);
```

**物理存储**：
```
联合索引结构
       │
    name 排序 (升序)
       │
    当 name 相同时，按 age 排序
       │
    当 name、age 都相同时，按 level 排序
       │
叶子节点：[name | age | level | id (主键)]
```

**最左前缀原则**：

```
创建索引：idx_name_age_level(name, age, level)

等价于创建了三个索引：
✓ (name)
✓ (name, age)
✓ (name, age, level)

使用情况：
✓ WHERE name = 'Tom'                          ← 可用
✓ WHERE name = 'Tom' AND age = 25             ← 可用
✓ WHERE name = 'Tom' AND age = 25 AND level='VIP' ← 可用
✗ WHERE age = 25                              ← 不可用（跳过name）
✗ WHERE age = 25 AND level = 'VIP'            ← 不可用（跳过name）
✓ WHERE level = 'VIP' AND name = 'Tom'        ← 可用（优化器重排）
```

**原理说明**：
```
B+ 树只对最左字段有序
├─ name 从小到大排序（第一层排序）
│   ├─ 当 name 相同时，age 才有序（第二层排序）
│   │   └─ 当 name、age 都相同时，level 才有序（第三层排序）
│   └─ 当 name 不同时，age 无序
└─ 直接跳过 name 查询 age，无法利用 B+ 树的有序性
```

**范围查询对后续列的影响**：

```
查询：WHERE name = 'Tom' AND age > 25 AND level = 'VIP'

执行过程：
1. name = 'Tom' ← 精确匹配，可用
2. age > 25     ← 范围匹配，后续列失效
3. level = 'VIP' ← 无法使用索引（被范围查询阻断）

优化建议：
- 如果必须用范围，将其放在 WHERE 末尾
- 或考虑拆分成多个查询
```

### 3.4 覆盖索引（Index Covering）

**定义**：查询所需的所有列都在索引中，无需回表

```sql
CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    age INT,
    email VARCHAR(50),
    INDEX idx_name_age(name, age)
);

-- 触发覆盖索引（不用回表）
SELECT name, age FROM user WHERE name = 'Tom';
-- Extra: Using index

-- 未触发覆盖索引（需要回表）
SELECT name, age, email FROM user WHERE name = 'Tom';
-- Extra: Using index condition; 表示回表了
```

**覆盖索引优化**：
```
问题：需要查询 name, age, email 三个字段
原索引：idx_name_age(name, age)
      ↓
改为：idx_name_age_email(name, age, email)
      ↓
查询：SELECT name, age, email
      ↓
覆盖索引直接返回，无需回表
```

### 3.5 索引下推（Index Condition Pushdown, ICP）

**概念**：在存储引擎层过滤不符合条件的数据，减少回表次数

```sql
CREATE TABLE user (
    id INT,
    name VARCHAR(20),
    age INT,
    INDEX idx_name_age(name, age)
);

-- 查询：WHERE name LIKE 'Tom%' AND age > 25

不使用 ICP 的过程：
1. 存储引擎：在 idx_name_age 找出 name LIKE 'Tom%' 的所有行
2. 返回到 Server 层
3. Server 层：再过滤 age > 25
4. 结果可能需要回表多次

使用 ICP 的过程：
1. 存储引擎：在 idx_name_age 找出 name LIKE 'Tom%'
2. 存储引擎：直接过滤 age > 25（不满足的行不返回）
3. 减少了回表次数
```

**ICP 工作机制图**：
```
WHERE name LIKE 'Tom%' AND age > 25

步骤1：索引范围扫描
idx_name_age
├─ Tom1, age=20  ← 找到
├─ Tom2, age=30  ← 找到
├─ Tom3, age=15  ← 找到
└─ ...

步骤2：ICP 过滤（存储引擎层）
├─ Tom1, age=20  ← 不满足 age>25，过滤
├─ Tom2, age=30  ← 满足，继续
├─ Tom3, age=15  ← 不满足 age>25，过滤
└─ ...

步骤3：回表查询完整数据
只回表满足条件的行
```

### 3.6 MRR（Multi-Range Read）优化

**原理**：将随机 IO 转换为顺序 IO

```sql
SELECT * FROM user WHERE age IN (20, 25, 30);

不使用 MRR：
1. age=20 的行记录位置：[Page 5, Page 10, Page 3, Page 8]
2. 逐个回表（随机 IO）：5→10→3→8（跳跃）

使用 MRR：
1. 读取所有符合条件的主键：[id1, id5, id2, id8, id3, id7]
2. 按主键排序：[id1, id2, id3, id5, id7, id8]
3. 顺序回表（顺序 IO）：更快
```

---

## 第四部分：SQL 执行计划分析（EXPLAIN）

### 4.1 EXPLAIN 输出字段详解

#### 基本用法
```sql
EXPLAIN SELECT * FROM user WHERE id = 1;

或

EXPLAIN SELECT * FROM user WHERE id = 1\G  （竖向显示）
```

#### 字段说明

| 字段 | 说明 | 优化建议 |
|------|------|---------|
| **id** | 查询序列号 | 大的先执行；相同则从上到下 |
| **select_type** | 查询类型 | SIMPLE/PRIMARY/SUBQUERY/DERIVED/UNION |
| **table** | 操作的表 | 关注表的访问顺序 |
| **partitions** | 分区 | 分区表相关 |
| **type** | 联接类型 | 最重要！system > const > eq_ref > ref > range > index > ALL |
| **possible_keys** | 可用索引 | 理论可用 |
| **key** | 实际使用索引 | NULL 表示未使用索引（可能索引失效） |
| **key_len** | 索引长度（字节） | 越大利用索引越充分 |
| **ref** | 索引匹配条件 | const/column/null 等 |
| **rows** | 估计扫描行数 | 越小越好 |
| **filtered** | 过滤百分比 | rows * filtered = 实际处理行数 |
| **Extra** | 额外信息 | 最关键！看有无 Using filesort/Using temporary 等 |

### 4.2 type 字段详解（关键）

**性能从好到差排序**：
```
system > const > eq_ref > ref > range > index > ALL
```

#### system
- **条件**：表只有一行
- **场景**：极少
- **示例**：`SELECT * FROM (SELECT 1) t`

#### const（常量）
- **条件**：主键/唯一索引与常量比较
- **性能**：最优，一次 IO 定位
- **示例**：`SELECT * FROM user WHERE id = 1`

```sql
EXPLAIN SELECT * FROM user WHERE id = 1;
-- type: const, rows: 1
```

#### eq_ref（等值连接）
- **条件**：多表连接，被驱动表用唯一索引
- **性能**：非常好
- **示例**：`SELECT * FROM user u JOIN order o ON u.id = o.user_id`

#### ref（非唯一索引）
- **条件**：非唯一索引等值查询
- **性能**：良好
- **示例**：`SELECT * FROM user WHERE age = 25` (age 有普通索引)

```sql
EXPLAIN SELECT * FROM user WHERE age = 25;
-- type: ref, rows: 100 (可能多行)
```

#### range（范围查询）
- **条件**：使用索引进行范围查询
- **运算符**：`>、>=、<、<=、BETWEEN、IN()`
- **性能**：中等
- **示例**：`SELECT * FROM user WHERE age > 25`

```sql
EXPLAIN SELECT * FROM user WHERE age BETWEEN 20 AND 30;
-- type: range, rows: 500
```

#### index（全索引扫描）
- **条件**：扫描整个索引树
- **原因**：SELECT 字段全在索引中；或排序用到索引
- **性能**：较差（但比 ALL 好，避免了回表）
- **示例**：`SELECT id, name FROM user ORDER BY name` (name 有索引)

```sql
EXPLAIN SELECT id, name FROM user ORDER BY name;
-- type: index (避免了 filesort)
-- Extra: Using index
```

#### ALL（全表扫描）
- **条件**：无索引可用或索引失效
- **性能**：最差
- **示例**：`SELECT * FROM user WHERE name = 'Tom'` (name 无索引)

```sql
EXPLAIN SELECT * FROM user WHERE name = 'Tom';
-- type: ALL, rows: 1000000 (全表扫描)
-- Extra: Using where
```

### 4.3 Extra 字段详解

#### Using index（覆盖索引）
```sql
EXPLAIN SELECT id, name FROM user WHERE id = 1;
-- Extra: Using index ✓
-- 说明：查询列完全在索引中，无需回表
```

#### Using where（需要过滤）
```sql
EXPLAIN SELECT * FROM user WHERE id = 1 AND age > 25;
-- Extra: Using where
-- 说明：WHERE 条件中有部分不能用索引过滤
-- 需要返回 Server 层继续过滤
```

#### Using filesort（文件排序）⚠️ 性能警告
```sql
EXPLAIN SELECT * FROM user ORDER BY age;
-- Extra: Using filesort ✗
-- 说明：无法使用索引排序，需要文件排序
-- 优化：为 age 创建索引
```

#### Using temporary（临时表）⚠️ 性能警告
```sql
EXPLAIN SELECT age, COUNT(*) FROM user GROUP BY age;
-- Extra: Using temporary ✗
-- 说明：需要创建临时表进行 GROUP BY
-- 优化：为 age 创建索引
```

#### Using index condition（索引条件下推）
```sql
EXPLAIN SELECT * FROM user WHERE name LIKE 'Tom%' AND age > 25;
-- Extra: Using index condition
-- 说明：使用了 ICP 优化，存储引擎层过滤
```

#### Using join buffer（连接缓冲）
```sql
EXPLAIN SELECT * FROM user u 
LEFT JOIN order o ON u.id = o.user_id;
-- Extra: Using join buffer (hash join)
-- 说明：使用了连接缓冲，被驱动表无索引
```

#### Using MRR（多范围读取）
```sql
EXPLAIN SELECT * FROM user WHERE age IN (20, 25, 30);
-- Extra: Using MRR
-- 说明：使用了 MRR 优化，将随机 IO 转为顺序 IO
```

### 4.4 执行计划分析示例

```sql
-- 示例表结构
CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    age INT,
    email VARCHAR(50),
    INDEX idx_name (name),
    INDEX idx_age (age)
);

-- 查询 1：最优
EXPLAIN SELECT id, name FROM user WHERE id = 1;
结果分析：
- type: const ✓ (最优)
- key: PRIMARY
- rows: 1
- Extra: NULL ✓

-- 查询 2：良好
EXPLAIN SELECT * FROM user WHERE name = 'Tom';
结果分析：
- type: ref ✓
- key: idx_name
- rows: 10 (估计)
- Extra: NULL (但实际需要回表)

-- 查询 3：一般（范围查询）
EXPLAIN SELECT * FROM user WHERE age > 25 AND age < 30;
结果分析：
- type: range ✓ (可接受)
- key: idx_age
- rows: 500
- Extra: Using where

-- 查询 4：差（全表扫描）
EXPLAIN SELECT * FROM user WHERE email = 'test@qq.com';
结果分析：
- type: ALL ✗ (最差)
- key: NULL (无索引)
- rows: 100000 (全表)
- Extra: Using where ✗
优化方案：为 email 添加索引
  ALTER TABLE user ADD INDEX idx_email(email);

-- 查询 5：差（索引失效）
EXPLAIN SELECT * FROM user WHERE UPPER(name) = 'TOM';
结果分析：
- type: ALL ✗ (函数破坏索引)
- key: NULL
- rows: 100000
- Extra: Using where ✗
优化方案：改为 WHERE name = 'TOM' (去掉函数)
```

---

## 第五部分：查询优化策略

### 5.1 JOIN 优化

#### JOIN 算法原理

**1. 简单嵌套循环连接（SNL）**
```
驱动表行数 = M，被驱动表行数 = N
磁盘 IO = M * N (最坏情况)

示例：
FOR EACH row IN M (驱动表) {
    FOR EACH row IN N (被驱动表) {
        IF join_condition MATCH {
            返回结果
        }
    }
}
```

**2. 索引嵌套循环连接（INL）**
```
前提：被驱动表 JOIN 字段有索引

优化后：
磁盘 IO = M * log(N) (M 为驱动表行数，log(N) 为索引高度)

示例：
FOR EACH row IN M (驱动表) {
    通过索引在 N 中查找 → 利用索引快速定位
}
```

**3. 块嵌套循环连接（BNL）**
```
场景：被驱动表 JOIN 字段无索引

优化方案：使用 join_buffer，缓冲驱动表数据
缓冲区大小：join_buffer_size (默认 256KB)

示例：
1. 缓冲驱动表前 256KB 数据到 join_buffer
2. 被驱动表与 join_buffer 一次性匹配
3. 被驱动表扫描次数减少

磁盘 IO = M / buffer_rows * N
```

#### JOIN 优化总结

```
优化原则：
1. 小表驱动大表（减少驱动表行数）
2. 为 JOIN 字段添加索引（使用 INL 算法）
3. 增大 join_buffer_size（减少被驱动表扫描次数）
4. 减少查询字段（字段越少，缓冲越多数据）

最终效果：
INL 算法 > BNL 算法 > SNL 算法
```

**示例代码**：
```sql
-- 表结构
CREATE TABLE user (id INT PRIMARY KEY, name VARCHAR(20));
CREATE TABLE order (id INT, user_id INT);

-- 优化前（无索引）
EXPLAIN SELECT * FROM user u 
LEFT JOIN order o ON u.id = o.user_id;
-- 执行计划：Type: ALL, Extra: Using join buffer (hash join)

-- 优化后（添加索引）
ALTER TABLE order ADD INDEX idx_user_id(user_id);
EXPLAIN SELECT * FROM user u 
LEFT JOIN order o ON u.id = o.user_id;
-- 执行计划：Type: ref, Extra: NULL (使用 INL 算法)
```

### 5.2 IN 和 EXISTS 优化

#### IN 函数（小表驱动大表）

```sql
-- 部门表 100 条，员工表 10000 条
SELECT * FROM employee 
WHERE dept_id IN (SELECT id FROM department);

执行原理：
1. 先执行子查询：SELECT id FROM department
   结果：dept_id 列表 [1,2,3,...]
2. 再在员工表查询：WHERE dept_id IN (1,2,3,...)
3. 部门表作为"驱动表"，只查询一次

性能：
- 子查询执行 1 次
- 员工表 WHERE 过滤
```

#### EXISTS 函数（大表驱动小表）

```sql
-- 员工表 10000 条，部门表 100 条
SELECT * FROM employee e 
WHERE EXISTS (SELECT 1 FROM department d WHERE d.id = e.dept_id);

执行原理：
1. 先遍历员工表（外表）：FOR EACH employee
2. 再执行子查询：SELECT 1 FROM department WHERE id = e.dept_id
3. 员工表作为"驱动表"

性能：
- 员工表遍历 1 次，为每行执行一次子查询
- 但由于子查询中对 dept_id 有索引，每次都很快
```

#### 选择建议

| 场景 | 推荐 | 原因 |
|------|------|------|
| 子查询结果少 | IN | 减少外表扫描次数 |
| 子查询结果多 | EXISTS | 减少子查询执行次数 |
| 外表数据少 | IN | 内表驱动更优 |
| 外表数据多 | EXISTS | 外表驱动更优 |

**口诀**：`IN 后接小表，EXISTS 后接大表`

### 5.3 ORDER BY 优化

#### 两种排序方式对比

| 方式 | 使用场景 | 性能 |
|------|---------|------|
| **索引排序** | 有序索引直接返回 | 最优 |
| **文件排序** | 无序数据需要排序 | 较差 |

**Extra 字段判断**：
- `Using index` ✓ 索引排序
- `Using filesort` ✗ 文件排序（需优化）

#### 案例1：利用索引排序

```sql
CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    age INT,
    INDEX idx_name_age(name, age)
);

-- ✓ 可使用索引排序（与索引顺序一致）
EXPLAIN SELECT id, name, age FROM user ORDER BY name, age;
-- type: index, Extra: Using index ✓

-- ✗ 不可使用索引排序（顺序不一致）
EXPLAIN SELECT id, name, age FROM user ORDER BY age, name;
-- type: ALL, Extra: Using filesort ✗
```

#### 案例2：WHERE + ORDER BY 优化

```sql
CREATE TABLE employee (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    age INT,
    salary DECIMAL(10,2),
    INDEX idx_name_age(name, age)
);

-- ✓ 最优（WHERE 等值 + ORDER BY 按索引顺序）
EXPLAIN SELECT name, age FROM employee 
WHERE name = 'Tom' ORDER BY age;
-- Extra: Using index ✓

-- ✗ 次优（WHERE 范围 + ORDER BY 后续列失效）
EXPLAIN SELECT name, age FROM employee 
WHERE name > 'Tom' ORDER BY age;
-- Extra: Using filesort ✗
-- 原因：范围查询后，age 无序

-- ✓ 优化方案（改变字段顺序）
ALTER TABLE employee ADD INDEX idx_age_name(age, name);
EXPLAIN SELECT id, age, name FROM employee 
WHERE age > 25 ORDER BY name;
-- 但这样又可能破坏其他查询的索引利用
```

#### 排序优化规则

```
规则1：WHERE 条件必须用上索引最左列
规则2：ORDER BY 字段顺序必须与索引顺序一致
规则3：不能有范围查询（会断链）
规则4：升降序必须一致（全升或全降）

示例：idx_name_age_level(name, age, level)

✓ ORDER BY name, age
✓ WHERE name = 'Tom' ORDER BY age
✓ WHERE name = 'Tom' AND age = 25 ORDER BY level
✗ WHERE name = 'Tom' ORDER BY level (跳过了 age)
✗ WHERE name > 'Tom' ORDER BY age (范围查询断链)
✗ ORDER BY name ASC, age DESC (升降序不一致)
```

### 5.4 GROUP BY 优化

#### 使用索引避免临时表

```sql
CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    age INT,
    INDEX idx_age(age)
);

-- ✗ 差（创建临时表进行 GROUP BY）
EXPLAIN SELECT age, COUNT(*) FROM user GROUP BY age;
-- Extra: Using temporary, Using filesort ✗

-- ✓ 好（使用索引）
ALTER TABLE user ADD INDEX idx_age(age);
EXPLAIN SELECT age FROM user GROUP BY age;
-- Extra: NULL ✓ (使用索引)
```

#### GROUP BY + ORDER BY 优化

```sql
-- ✗ 需要排序
EXPLAIN SELECT age, COUNT(*) FROM user GROUP BY age ORDER BY age;
-- Extra: Using temporary, Using filesort ✗

-- ✓ 索引天然有序
ALTER TABLE user ADD INDEX idx_age(age);
EXPLAIN SELECT age, COUNT(*) FROM user GROUP BY age ORDER BY age;
-- Extra: NULL ✓
```

### 5.5 LIMIT 优化（关键）

#### 问题：大偏移量查询慢

```sql
-- ✗ 慢（跳过 100000 条记录）
SELECT * FROM user LIMIT 100000, 10;
-- 实际扫描：100010 条记录，其中 100000 条被抛弃
```

#### 优化方案1：基于主键范围

```sql
-- ✓ 快（使用主键索引定位）
SELECT * FROM user WHERE id >= (
    SELECT id FROM user LIMIT 100000, 1
) LIMIT 10;
-- 只需扫描 10 条记录
```

#### 优化方案2：书签方式

```sql
-- 假设上次查询的最后一条记录 id = 100000
-- ✓ 快（从书签位置开始）
SELECT * FROM user WHERE id > 100000 LIMIT 10;
-- 推荐方案：记录上次查询的最后一个 id/时间戳
```

---

## 第六部分：事务与锁机制

### 6.1 MVCC（多版本并发控制）

#### 核心概念

```
MVCC = 多个版本 + 快照隔离

原理：
每条记录都有多个版本
├─ 版本1：事务1 写入
├─ 版本2：事务2 修改
├─ 版本3：事务3 修改
└─ ...

读取时：获取当前可见的版本（基于隔离级别）
修改时：生成新版本，旧版本保留（用于回滚/历史读）
```

#### 版本控制信息

```
每条记录都有隐藏列：
├─ DB_TRX_ID (事务 ID)
├─ DB_ROLL_PTR (回滚指针，指向上一版本)
└─ DB_ROW_ID (行 ID，无主键时使用)

版本链：
Version 3: (data, trx_id=103)
  ↓ (roll_ptr)
Version 2: (data, trx_id=102)
  ↓ (roll_ptr)
Version 1: (data, trx_id=101)
  ↓ (roll_ptr)
NULL
```

### 6.2 事务隔离级别

| 隔离级别 | 脏读 | 不可重复读 | 幻读 | 行锁 | 实现 |
|---------|------|----------|------|------|------|
| **READ UNCOMMITTED** | ✓ | ✓ | ✓ | ✗ | 无 |
| **READ COMMITTED** | ✗ | ✓ | ✓ | ✓ | MVCC |
| **REPEATABLE READ** | ✗ | ✗ | ✓ | ✓ | MVCC + Gap Lock |
| **SERIALIZABLE** | ✗ | ✗ | ✗ | ✓ | 串行化 |

> MySQL InnoDB 默认：**REPEATABLE READ**

#### 问题场景演示

**脏读（Dirty Read）**
```
事务1：UPDATE user SET age = 30 WHERE id = 1; (未提交)
事务2：SELECT age FROM user WHERE id = 1;
       读到 30（事务1 未提交的数据！）
事务1：ROLLBACK;
结果：事务2 读到了被回滚的数据
```

**不可重复读（Non-repeatable Read）**
```
事务1：SELECT age FROM user WHERE id = 1; → age = 25
事务2：UPDATE user SET age = 30 WHERE id = 1;
      COMMIT;
事务1：SELECT age FROM user WHERE id = 1; → age = 30 (不同了！)
结果：同一事务内，两次读取数据不一致
```

**幻读（Phantom Read）**
```
事务1：SELECT COUNT(*) FROM user WHERE age > 25;
       结果：100 条
事务2：INSERT INTO user (age) VALUES (26), (27), (28);
       COMMIT; (插入 3 条 age > 25 的记录)
事务1：SELECT COUNT(*) FROM user WHERE age > 25;
       结果：103 条 (凭空多了 3 条！)
```

### 6.3 锁机制

#### 锁的分类

```
按范围分：
├─ 表锁（Table Lock）
│   └─ 粗粒度，并发性差，但争用少
├─ 页锁（Page Lock）
│   └─ 中等粒度
└─ 行锁（Row Lock）
    └─ 细粒度，并发性好，但争用多

按性质分：
├─ 共享锁（Shared Lock，S 锁）
│   └─ SELECT...LOCK IN SHARE MODE
│   └─ 多个事务可同时持有 S 锁
├─ 排他锁（Exclusive Lock，X 锁）
│   └─ SELECT...FOR UPDATE
│   └─ 或修改操作 (INSERT/UPDATE/DELETE)
│   └─ 一个事务持有时，其他事务不能持有任何锁
└─ 意向锁（Intention Lock）
    └─ IS/IX，表级锁
    └─ 表示后续会申请表内的行锁
```

#### InnoDB 行锁类型

```
1. Record Lock（记录锁）
   └─ 锁定单条记录
   └─ 等值查询匹配的记录被锁

2. Gap Lock（间隙锁）
   └─ 锁定记录之间的间隙
   └─ 防止幻读
   └─ 范围查询时出现

   例：id = 1, 3, 5
       间隙：(1,3), (3,5), (5,∞)

3. Next-Key Lock（临键锁）
   └─ = Record Lock + Gap Lock
   └─ 锁定记录及其前面的间隙
   └─ REPEATABLE READ 隔离级别默认使用

   例：WHERE id >= 3
       锁定：记录 3 + (1,3) 间隙
```

**锁的示意**：
```
表数据：id = 1, 3, 5, 7, 9

查询 1：WHERE id = 3
  锁定：只有记录 3 (Record Lock)
  
查询 2：WHERE id >= 5
  锁定：记录 5,7,9 + (3,5) 间隙 + (9,+∞) 间隙 (Next-Key Lock)
  
查询 3：WHERE id BETWEEN 3 AND 7
  锁定：记录 3,5,7 + (1,3) 间隙 + (7,9) 间隙 (Next-Key Lock)
```

### 6.4 死锁

#### 常见死锁场景

**场景1：相反顺序锁定**
```
事务1：
1. UPDATE user SET age = 30 WHERE id = 1;  (锁定行1)
2. UPDATE order SET status = 'paid' WHERE user_id = 2;  (等待行2)

事务2：
1. UPDATE order SET status = 'paid' WHERE user_id = 2;  (锁定行2)
2. UPDATE user SET age = 25 WHERE id = 1;  (等待行1)

结果：死锁！
```

#### 死锁预防

```
原则1：避免在事务中请求不同对象的锁
原则2：如必须多步操作，始终按相同顺序请求锁

修正：
事务1：按 id 升序操作
1. UPDATE user ... WHERE id = 1;
2. UPDATE order ... WHERE user_id = 2;

事务2：也按 id 升序操作
1. UPDATE user ... WHERE id = 1;
2. UPDATE order ... WHERE user_id = 2;

结果：避免死锁，因为锁定顺序一致
```

---

## 第七部分：性能调优参数

### 7.1 关键参数说明

#### Buffer Pool 相关

```sql
-- Buffer Pool 大小（最重要！）
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
-- 推荐：物理内存的 50% - 80%

-- Buffer Pool 实例数（减少锁争用）
SHOW VARIABLES LIKE 'innodb_buffer_pool_instances';
-- 推荐：>= CPU 核心数

-- 预读大小（顺序读优化）
SHOW VARIABLES LIKE 'innodb_read_ahead_threshold';
```

#### 日志相关

```sql
-- Redo Log 文件大小
SHOW VARIABLES LIKE 'innodb_log_file_size';
-- 推荐：1GB - 4GB（权衡恢复时间和写吞吐）

-- Redo Log 缓冲大小
SHOW VARIABLES LIKE 'innodb_log_buffer_size';
-- 推荐：4MB - 16MB

-- Binlog 相关
SHOW VARIABLES LIKE 'binlog_format';  -- ROW/STATEMENT/MIXED
-- 推荐：ROW（主从复制安全）

-- 是否每个事务都刷盘
SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';
-- 0：不刷，最快但不安全
-- 1：每个事务都刷，最安全但最慢
-- 2：异步刷，折中方案（推荐）
```

#### 并发相关

```sql
-- 最大连接数
SHOW VARIABLES LIKE 'max_connections';
-- 推荐：根据应用需求，通常 1000-2000

-- 连接缓冲
SHOW VARIABLES LIKE 'join_buffer_size';
-- 默认：256KB，如果频繁 JOIN，可增大到 1MB-4MB

-- 排序缓冲
SHOW VARIABLES LIKE 'sort_buffer_size';
-- 默认：256KB，如果频繁 ORDER BY，可增大到 2MB-4MB
```

#### 慢查询相关

```sql
-- 启用慢查询日志
SET GLOBAL slow_query_log = ON;

-- 慢查询阈值（秒）
SET GLOBAL long_query_time = 1;

-- 记录未使用索引的查询
SET GLOBAL log_queries_not_using_indexes = ON;

-- 查看慢查询日志位置
SHOW VARIABLES LIKE 'slow_query_log_file';
```

### 7.2 表设计优化

#### 字段类型选择

```sql
-- ✓ 好
CREATE TABLE user (
    id INT PRIMARY KEY AUTO_INCREMENT,          -- int(4字节) vs bigint(8字节)
    name VARCHAR(50) NOT NULL,                  -- 定长字符串优于 BLOB
    age TINYINT NOT NULL DEFAULT 0,             -- TINYINT(1字节) vs INT(4字节)
    email VARCHAR(100),                         -- 合理限制字符长度
    create_time TIMESTAMP NOT NULL,             -- TIMESTAMP vs DATETIME
    ...
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ✗ 差
CREATE TABLE user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,       -- 字段过大
    name TEXT,                                  -- 定长用 VARCHAR
    age INT DEFAULT 0,                          -- 类型偏大
    email VARCHAR(1000),                        -- 字符长度太大
    create_time DATETIME,                       -- 占用空间多
    ...
);
```

#### 索引设计原则

```sql
-- 原则1：高频查询字段建索引
CREATE INDEX idx_email ON user(email);

-- 原则2：区分度高的字段优先
-- ✗ 差（只有 0/1 两个值）
CREATE INDEX idx_status ON user(status);

-- ✓ 好（1000 种值）
CREATE INDEX idx_dept_id ON user(dept_id);

-- 原则3：避免过多索引（影响写性能）
-- 建议：不超过 5 个索引

-- 原则4：使用联合索引而非多个单列索引
-- ✗ 差
CREATE INDEX idx_name ON user(name);
CREATE INDEX idx_age ON user(age);

-- ✓ 好
CREATE INDEX idx_name_age ON user(name, age);
```

---

## 第八部分：常见问题速查表

### 8.1 EXPLAIN 字段速查

| 关键字 | 含义 | 优化建议 |
|--------|------|---------|
| type=ALL | 全表扫描 | 添加索引 |
| type=index | 全索引扫描 | 添加覆盖索引或限制查询字段 |
| type=range | 范围扫描 | 可接受，但避免范围后还有其他条件 |
| key=NULL | 未使用索引 | 检查索引是否失效，见 8.3 |
| Extra: Using filesort | 文件排序 | 为排序字段添加索引 |
| Extra: Using temporary | 临时表 | 为分组字段添加索引 |
| Extra: Using where | WHERE 过滤 | 检查 WHERE 条件是否能用索引 |

### 8.2 隔离级别速查

| 隔离级别 | 含义 | 场景 |
|---------|------|------|
| READ UNCOMMITTED | 读未提交 | 极少使用（不安全） |
| READ COMMITTED | 读已提交 | 日志系统、消息队列 |
| REPEATABLE READ | 可重复读 | **默认、大多场景** |
| SERIALIZABLE | 串行化 | 金融系统、高一致性 |

### 8.3 索引失效情况总结

```
1. 计算/函数破坏索引
   ✗ WHERE YEAR(create_date) = 2023
   ✓ WHERE create_date >= '2023-01-01'

2. 类型隐转
   ✗ WHERE name = 123 (字符串字段)
   ✓ WHERE name = '123'

3. LIKE 左边通配符
   ✗ WHERE name LIKE '%Tom%'
   ✓ WHERE name LIKE 'Tom%'
   优化：使用覆盖索引: SELECT name FROM... LIKE '%Tom%'

4. 不等于/IS NULL
   ✗ WHERE age != 25
   ✗ WHERE age IS NULL
   优化：考虑拆分查询或修改查询条件

5. OR 条件
   ✗ WHERE age = 25 OR name = 'Tom' (两个字段都没联合索引)
   ✓ WHERE age = 25 UNION SELECT ... WHERE name = 'Tom'

6. 不按最左前缀
   ✗ WHERE age = 25 (idx_name_age(name, age))
   ✓ WHERE name = 'Tom' AND age = 25

7. 范围后失效
   ✗ WHERE name = 'Tom' AND age > 25 AND level = 'VIP'
     (level 不能用索引)
   优化：调整字段顺序或分开查询
```

### 8.4 慢查询排查步骤

```
步骤1：启用慢查询日志
  SET GLOBAL slow_query_log = ON;
  SET GLOBAL long_query_time = 1;

步骤2：执行 EXPLAIN 分析
  EXPLAIN SELECT ...;
  关注：type, key, rows, Extra

步骤3：检查 type
  ✓ 最低要求：range 级别
  ✗ 如果是 ALL/index，需要添加索引

步骤4：检查 Extra
  ✗ Using filesort → 添加排序索引
  ✗ Using temporary → 添加分组索引
  ✗ Using where → 检查 WHERE 条件

步骤5：添加/调整索引
  ALTER TABLE table_name ADD INDEX idx_name(col);

步骤6：再次 EXPLAIN 验证
  确认 type 和 rows 是否改善
```

### 8.5 常见调优口诀

```
索引选择：
✓ 高频查询、区分度高、定长字段
✗ 冷门字段、区分度低、变长字段

索引创建：
✓ 联合索引而非单列索引
✓ 遵循最左前缀原则
✓ 避免过多索引（通常 ≤ 5 个）

WHERE 条件优化：
✓ 等值条件优先，范围条件放后
✓ 避免函数、类型转换、NOT IN
✓ 小表驱动大表

ORDER BY 优化：
✓ 与索引字段顺序一致
✓ 升降序保持一致
✓ WHERE 条件能用索引

JOIN 优化：
✓ 为连接字段添加索引
✓ 小表驱动大表
✓ 减少查询字段

GROUP BY 优化：
✓ 为分组字段添加索引
✓ 避免临时表和文件排序
```

### 8.6 参数调优速查

```
内存相关：
- innodb_buffer_pool_size: 物理内存 50%-80%
- join_buffer_size: 1MB-4MB (频繁JOIN时)
- sort_buffer_size: 2MB-4MB (频繁ORDER BY时)

日志相关：
- innodb_log_file_size: 1GB-4GB
- innodb_flush_log_at_trx_commit: 2 (折中方案)

并发相关：
- max_connections: 1000-2000
- innodb_lock_wait_timeout: 50 (死锁超时)

慢查询：
- slow_query_log: ON
- long_query_time: 1
- log_queries_not_using_indexes: ON
```

---

## ⚠️ 原文勘误

> 本文档基于原始课程笔记梳理，原文未发现严重技术错误，但有以下需要澄清的地方：

1. **MySQL 8.0 查询缓存**
   - 原文：MySQL 8.0 版本直接将查询缓存删除
   - 澄清：✓ 正确，MySQL 8.0.3+ 完全移除了查询缓存功能

2. **InnoDB 隐式主键**
   - 原文：InnoDB 会生成 6 字节的 ROWID
   - 澄清：✓ 正确，当无主键时 InnoDB 自动创建 ROWID（8字节，包括标志位）

3. **MVCC 在 MyISAM 中的支持**
   - 原文：未明确说明 MyISAM 不支持 MVCC
   - 澄清：MyISAM 不支持 MVCC，只有 InnoDB 支持

4. **间隙锁（Gap Lock）的范围**
   - 原文：未详细说明间隙锁的精确范围
   - 澄清：间隙锁锁定记录之间的"空隙"，防止在该范围内的 INSERT

---

## 核心知识点速查表

### EXPLAIN 字段快速查询表

```
id        范围值：1,2,3... 大的先执行
select_type  SIMPLE/PRIMARY/SUBQUERY/DERIVED/UNION
table     被查询的表名
partitions   分区表相关，一般为 NULL
type      system/const/eq_ref/ref/range/index/ALL（从好到差）
possible_keys 理论可用索引
key       实际使用的索引，NULL 表示未使用
key_len   索引使用长度（字节），越大越充分
ref       索引匹配方式（const/column/null）
rows      估计扫描行数，越小越好
filtered  行过滤百分比
Extra     Using index / Using filesort / Using temporary / Using where 等
```

### 隔离级别选择表

```
隔离级别          脏读  不可重复读  幻读  推荐场景
────────────────────────────────────────────────────
READ UNCOMMITTED   是    是        是    不建议使用
READ COMMITTED     否    是        是    日志/消息
REPEATABLE READ    否    否        是    ✓ 默认，通用
SERIALIZABLE       否    否        否    金融/高一致
```

### 索引优化口诀表

```
选择字段：高频、区分度高、定长优先
创建方式：联合优于单列、遵循最左前缀
WHERE 条件：等值先、范围后、避免函数
ORDER BY：与索引顺序一致、升降序一致
GROUP BY：为分组字段建索引、避免临时表
JOIN：为连接字段建索引、小表驱动大表
LIMIT：基于主键范围或书签方式优化
```

### 慢查询诊断流程

```
检查日志
  ↓
EXPLAIN 分析
  ↓
判断 type
  ├─ ALL/index → 需要索引
  └─ range+ → 可接受
  ↓
检查 Extra
  ├─ Using filesort → 添加排序索引
  ├─ Using temporary → 添加分组索引
  └─ Using where → 检查条件
  ↓
修改索引和 SQL
  ↓
再次 EXPLAIN 验证
```

---

## 参考资源

- InnoDB 官方文档：https://dev.mysql.com/doc/refman/8.0/en/innodb.html
- MySQL 性能调优：https://dev.mysql.com/doc/refman/8.0/en/optimization.html
- EXPLAIN 详解：https://dev.mysql.com/doc/refman/8.0/en/explain-output.html

