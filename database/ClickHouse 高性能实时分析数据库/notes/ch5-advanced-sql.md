# 第5章 释放查询的全部潜力

> 本章梳理自原文「第5章 释放查询的全部潜力」。覆盖:聚合函数、JOIN(ALL/ANY/GLOBAL)、子查询与 CTE、Array/Tuple/Map 函数、窗口函数、物化视图。

---

## 1. 聚合函数

### 1.1 标配 vs ClickHouse 特色

| 标配 | ClickHouse 特色 |
|------|----------------|
| `count`、`sum`、`avg`、`min`、`max` | `uniq`(HyperLogLog 近似去重) |
|  | `uniqExact`(精确去重,小数据集) |
|  | `uniqCombined`(HLL + 抽样,通用首选) |
|  | `quantile`、`quantileTDigest`、`quantilesExact(0.5,0.9,0.99)` |
|  | `topK(N)`、`topKWeighted(N)` |
|  | `groupArray`、`groupUniqArray`、`groupArrayMovingSum` |
|  | `argMin(arg, val)`、`argMax(arg, val)` |
|  | `corr`、`covarSamp`、`stddevSamp`、`varPop` |
|  | `sumMap`、`minMap`、`maxMap` |
|  | `anyHeavy`(Misra-Gries 重击数) |
|  | `windowFunnel(t)(cond1, cond2, ...)`(漏斗分析) |
|  | `sequenceMatch / sequenceCount`(序列模式) |
|  | `retention(c1, c2, ...)`(留存分析) |

### 1.2 `uniq` vs `uniqExact`

```
   ┌──────────────────────────────────────┐
   │  uniq         ≈ HyperLogLog          │   误差 ~ ±2.5%,常数级内存,海量数据首选
   │  uniqCombined ≈ HLL + sampling       │   误差更小,内存略大,通用推荐
   │  uniqExact    ≈ HashSet              │   精确,内存随基数线性增长,小集合再用
   └──────────────────────────────────────┘
```

### 1.3 聚合组合器(Combinator)

ClickHouse 允许在聚合函数名后加后缀,改变行为:

| 组合器 | 作用 | 示例 |
|--------|------|------|
| `-If` | 满足条件才聚合 | `sumIf(amount, status='paid')` |
| `-Array` | 聚合一个数组列 | `sumArray(prices)` 等价 `sum(arrayJoin(prices))` |
| `-State` | 输出中间状态(不出最终值) | `uniqState(user_id)` |
| `-Merge` | 把中间状态合并出最终结果 | `uniqMerge(uv)` |
| `-OrNull` | 空集返回 NULL 而不是 0 | `avgOrNull(price)` |
| `-OrDefault` | 空集返回默认值 | `sumOrDefault(price)` |
| `-Distinct` | 仅唯一值 | `sumDistinct(price)` |
| `-Resample(from, to, step)` | 按区间分桶 | `uniqResample(1, 10, 1)(uid, age)` |

`-If` 经典写法:
```sql
SELECT
  page_id,
  count()                            AS total_pv,
  countIf(device='mobile')           AS mobile_pv,
  uniqIf(user_id, device='mobile')   AS mobile_uv,
  sumIf(amount, status='paid')       AS paid_amount
FROM access_logs
GROUP BY page_id;
```

`-State`/`-Merge` 是 `AggregatingMergeTree` + 物化视图的核心(见 5.6 和第 3 章)。

---

## 2. JOIN

### 2.1 类型一览

`INNER`、`LEFT`、`RIGHT`、`FULL`、`CROSS`,语义同标准 SQL。

ClickHouse 特色:

| 修饰符 | 行为 |
|--------|------|
| `ALL` | 右表多匹配时,左行复制多次(标准 SQL 默认) |
| `ANY` | 右表多匹配时,只取一条,左行不复制 |
| `ASOF`(LEFT) | 时间最近匹配(`>= / <`),典型用于事件 + 价格曲线对齐 |
| `GLOBAL` | 分布式时,右表从协调节点广播给所有分片 |

### 2.2 `ALL` vs `ANY`

数据陷阱:
```
左表 orders:
   (order_id=1, user_id=101, amount=99)
右表 users_repeated:
   (user_id=101, name='悟空')
   (user_id=101, name='行者')

ALL LEFT JOIN  → 两行 (order_id=1 出现两次,amount 加和会被放大!)
ANY LEFT JOIN  → 一行 (只匹配第一条右表行)
```

经验:**不确定右表是否唯一,就用 `ANY`**。在数仓场景下 `ANY JOIN` 更常用、更安全。

### 2.3 分布式 JOIN:`GLOBAL` 几乎是必选

普通 JOIN 在分布式表上的隐患:每个分片节点会**独立去查右表**(可能去 N-1 个节点拉全表),造成 N×N 的网络扩散。

```
   普通 JOIN(慢):
   ┌─────────────────────────────────────────┐
   │ Coordinator 收到查询                    │
   ├─────────────────────────────────────────┤
   │ 分发到 Shard1 / Shard2 / Shard3 ...     │
   │ 每个 Shard 各自再去其他节点拉右表       │
   │                                         │
   │ Shard1 ──► Shard2.users  ◄── Shard3     │
   │ Shard2 ──► Shard1.users  ◄── Shard3     │
   │ ...                                      │
   │                                         │
   │ 网络流量 N×N,内存重复构建哈希表        │
   └─────────────────────────────────────────┘
```

```
   GLOBAL JOIN(快):
   ┌─────────────────────────────────────────┐
   │ Coordinator 先把右表(子查询)算好     │
   │ 一次性广播给所有 Shard                  │
   │                                         │
   │            ┌───────────┐                │
   │            │ 右表数据集 │                │
   │            └─────┬─────┘                │
   │       ┌──────────┼──────────┐           │
   │       ▼          ▼          ▼           │
   │   Shard1     Shard2     Shard3          │
   │  本地 JOIN  本地 JOIN  本地 JOIN        │
   │                                         │
   │ 网络流量 1×N,各分片本地哈希一次        │
   └─────────────────────────────────────────┘
```

DDL 写法:
```sql
SELECT o.order_id, u.name
FROM orders_distributed AS o
GLOBAL ANY LEFT JOIN users AS u ON o.user_id = u.user_id;

-- 右表也是分布式时,用子查询
SELECT ...
FROM orders_distributed AS o
GLOBAL ANY LEFT JOIN (
    SELECT DISTINCT user_id, name FROM users_distributed
) AS u ON o.user_id = u.user_id;
```

### 2.4 性能与限制

- ClickHouse 的 JOIN 是**右表加载进内存建哈希**,所以**右表必须放得进内存**。
- 大表 × 大表 JOIN 极其昂贵,优先考虑宽表 / 字典 / `IN` 子查询。
- `IN` 子查询通常比 `JOIN` 更高效,优化器会自动转换。`GLOBAL IN` 比 `GLOBAL JOIN` 网络/内存开销更低。
- v22+ 提供了新算法选择:`join_algorithm = 'hash' | 'parallel_hash' | 'partial_merge' | 'grace_hash' | 'direct' | 'auto'`。

### 2.5 `ASOF JOIN`(时间最近匹配)

```sql
SELECT t.event_time, t.user_id, t.price, q.market_price
FROM trades AS t
ASOF LEFT JOIN quotes AS q
  ON t.symbol = q.symbol AND t.event_time >= q.event_time;
```
为每笔交易匹配"事件时间之前最近"的报价,典型金融/IoT 场景。

---

## 3. 子查询与 CTE

### 3.1 三种子查询位置

| 位置 | 用法 | 备注 |
|------|------|------|
| `FROM (...) AS t` | 派生表 | 推荐改写为 CTE |
| `WHERE col IN (SELECT ...)` | 过滤 | 优化器会重写为半连接;分布式建议 `GLOBAL IN` |
| `SELECT (...)` 标量子查询 | 标量值 | 性能差,首选窗口函数或 JOIN |

### 3.2 CTE(WITH 子句)

```sql
WITH
  cte1 AS ( SELECT ... ),
  cte2 AS ( SELECT ... FROM cte1 ... )
SELECT ... FROM cte1 JOIN cte2 ...;
```

复杂查询拆步骤:
```sql
WITH
  CategorySalesByCountry AS (
    SELECT u.country, p.category, sum(oi.price * oi.quantity) AS total_sales
    FROM order_items oi
    JOIN orders   o ON oi.order_id = o.order_id
    JOIN users    u ON o.user_id = u.user_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY u.country, p.category
  ),
  RankedSales AS (
    SELECT country, category, total_sales,
           row_number() OVER (PARTITION BY country ORDER BY total_sales DESC) AS rn
    FROM CategorySalesByCountry
  )
SELECT country, category, total_sales
FROM RankedSales
WHERE rn = 1;
```

CTE 优点:可读性、可复用、可拆分排查;ClickHouse 的 CTE 在新版引擎里是真正的"物化一次,多处引用",而不仅仅是字面替换。

> ClickHouse 暂不支持递归 CTE(`WITH RECURSIVE`)。

### 3.3 标量子查询的代替方案

```sql
-- ✗ 标量子查询(慢)
SELECT order_id, user_id, amount,
       (SELECT count() FROM orders o2 WHERE o2.user_id = o1.user_id) AS user_total
FROM orders o1;

-- ✓ 用窗口函数
SELECT order_id, user_id, amount,
       count() OVER (PARTITION BY user_id) AS user_total
FROM orders;
```

---

## 4. Array / Tuple / Map 函数

### 4.1 Array

- **索引从 1 开始**:`arr[1]` 是第一个元素。
- 创建:`[a, b, c]` 或 `array(a, b, c)`。

| 函数 | 作用 |
|------|------|
| `arrayJoin(arr)` | 把一行炸裂成多行(王牌) |
| `arrayMap(x -> f(x), arr)` | 元素 1:1 加工 |
| `arrayFilter(x -> p(x), arr)` | 过滤 |
| `arrayReduce('agg', arr)` / `arraySum`, `arrayAvg` | 聚合 |
| `has(arr, v)` / `hasAll` / `hasAny` | 包含判断 |
| `indexOf(arr, v)` | 元素首次位置(从 1,不存在返回 0) |
| `arrayDistinct(arr)` | 去重 |
| `arrayConcat`, `arraySlice`, `arrayReverse` | 形变 |
| `arraySort`, `arrayReverseSort` | 排序 |
| `arrayEnumerate(arr)` | 生成下标数组 |
| `arrayEnumerateUniq(arr)` | 每个元素的出现序号 |

`ARRAY JOIN` 关键示例:
```sql
SELECT tag, count() AS product_count
FROM products
ARRAY JOIN tags AS tag
GROUP BY tag;
```

⚠️ 注意区分:
- `ARRAY JOIN`(SQL 子句):展开行,典型用法。
- `arrayJoin(arr)`(函数):同义,但更随便,放 SELECT 列表里也行。

### 4.2 Tuple

- 长度固定,元素类型可异构。
- 访问 `t.1`、`t.2`,或在创建时取名 `t.name`。
- `arrayMap` 的 lambda 可接收多个并列数组,自动打包成 tuple 处理:
  ```sql
  SELECT arrayFilter(
      (evt, ts) -> evt = 'login',
      events, timestamps
  ) AS login_timestamps
  ```

### 4.3 Map

`Map(K, V)`,适合稀疏属性、HTTP Header、用户画像。

| 操作 | 用法 |
|------|------|
| 取值 | `m['key']` |
| 存在判断 | `mapContains(m, 'key')` |
| 全部键 / 值 | `mapKeys(m)` / `mapValues(m)` |
| 转 Array of Tuples | `m AS Array(Tuple(K,V))`(隐式) |
| 修改 | `mapUpdate(m1, m2)`、`mapApply(...)` |

---

## 5. 窗口函数

核心思想:在与当前行相关的"窗口"上做计算,**不改变行数**。

```sql
FUNCTION_NAME() OVER (
  [PARTITION BY ...]
  [ORDER BY ...]
  [ROWS|RANGE BETWEEN ... AND ...]
)
```

| 类别 | 典型函数 | 备注 |
|------|---------|------|
| 聚合 | `sum`、`avg`、`count`、`min`、`max` 接 `OVER` | 不改行数,附加列 |
| 排名 | `row_number()`、`rank()`、`dense_rank()` | 需 `ORDER BY` |
| 偏移 | `lag(col, n, default)`、`lead(col, n, default)` | 上下行 |
| 首尾值 | `first_value(col)`、`last_value(col)`、`nth_value(col, n)` | 帧内 |
| 累计 | `sum(...) OVER (ORDER BY ... ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` | 累计求和 |

帧边界:
```
[UNBOUNDED PRECEDING | n PRECEDING | CURRENT ROW | n FOLLOWING | UNBOUNDED FOLLOWING]
ROWS  —— 物理行偏移
RANGE —— 逻辑值偏移(需要 ORDER BY 列是有序可比较的)
```

3 日移动平均:
```sql
SELECT
  employee, sale_date, amount,
  avg(amount) OVER (
    PARTITION BY employee
    ORDER BY sale_date
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
  ) AS moving_avg_3
FROM sales;
```

---

## 6. 物化视图(再总结一次)

| 项 | ClickHouse 物化视图 |
|----|--------------------|
| 本质 | 插入触发器 + 转换规则 |
| 数据真正存放 | 在 **目标表**(`TO target_table`),不在 MV 本身 |
| 触发时机 | 源表每次 `INSERT` 写入新 Part 时 |
| 处理范围 | **只处理本次写入的新数据块**,不会重新扫源表 |
| 已存量数据 | 默认不处理,需 `POPULATE` 或后续手动回填 |
| 修改逻辑 | MV 的 SELECT 不可改,只能 `DROP` + `CREATE` |
| 删除 MV | 不影响目标表;目标表的数据仍在 |
| 暂停/恢复 | `DETACH TABLE mv` / `ATTACH TABLE mv` |
| 失败处理 | MV 执行报错会让源 INSERT 失败(可配 `materialized_views_ignore_errors`) |

### 6.1 经典流水线

```
   ┌────────────┐  INSERT  ┌────────────┐ MV  ┌──────────────────────────┐
   │ Application│ ───────► │ raw_logs   │ ──► │ a_min  (AggregatingMT)   │
   └────────────┘          │ MergeTree  │     │ 每分钟 PV/UV 中间态       │
                           └────────────┘     └────────────┬─────────────┘
                                                           │ MV
                                                           ▼
                                           ┌──────────────────────────┐
                                           │ a_hour (AggregatingMT)   │
                                           │ 每小时 PV/UV 中间态        │
                                           └──────────────────────────┘
```

要点:**MV 之间可以串联**,形成数据处理流水线。

### 6.2 目标表引擎选择

| 目标表引擎 | 适用 |
|-----------|------|
| `AggregatingMergeTree` | 需要 `uniq`/`avg`/`quantile` 等复杂聚合 |
| `SummingMergeTree` | 只 `sum`/`count` 的预聚合 |
| `MergeTree` | 仅做 ETL(清洗/拆列/类型转换),不预聚合 |
| `ReplicatedAggregatingMergeTree` 等 | 上述任一在集群下的复制版本 |

---

## 7. 本章勘误区

### ✗ 原文(2.5 节 `GLOBAL ANY LEFT JOIN`)
原文示例的右表是"`users`",声称 "users 是一个存在于某个节点上的普通表"。

### ✓ 修正 / 补充
- 当右表是**分布式表**时,`GLOBAL ANY LEFT JOIN` 才有"广播给各分片"的语义。
- 当右表本身就是**所有节点都有的本地表**(各节点表名相同),普通 JOIN 在多数情况下就够了——每个分片本地查就行。
- 标准写法是右表用**子查询**:`GLOBAL ANY LEFT JOIN (SELECT ... FROM users_distributed) AS u ON ...`,这样语义最清晰。

---

### ✗ 原文(标量子查询性能提示)
> "在 ClickHouse 中,`IN` 子查询通常会被自动优化成 `JOIN`。"

### ✓ 修正
反过来才对:**`IN` 子查询在 ClickHouse 里通常比 `JOIN` 高效**;ClickHouse 不会把 IN 改写成 JOIN,而是用 set 过滤(`PreparedSet`)实现,内存开销更小、计算更高效。"自动改成 JOIN"是 MySQL 的优化器行为,不是 ClickHouse 的。

---

### ✗ 原文(Array `arrayMap` 示例)
```sql
insert into tb_names (1 , ['a' ,'b' ,'ab' , 'ao']) ;
```
缺少 `VALUES`,且括号位置不对,会报语法错误。

### ✓ 修正
```sql
INSERT INTO tb_names VALUES (1, ['a','b','ab','ao']);
```

---

### ✗ 原文(终极挑战 SQL)
```sql
SELECT DISTINCT user_id FROM click_stream
WHERE
    indexOf(event_names, 'view_product') AS view_pos > 0
AND
    indexOf(event_names, 'add_to_cart') AS cart_pos  > 0
AND
    cart_pos > view_pos;
```
在 `WHERE` 子句中用 `AS` 起别名是 **非法语法** (绝大多数 SQL 方言不允许,ClickHouse 也不允许)。

### ✓ 正解
要么用 CTE / 子查询提前算位置,要么在 SELECT 里命名后再过滤:
```sql
SELECT user_id
FROM (
  SELECT
    user_id,
    indexOf(event_names, 'view_product') AS view_pos,
    indexOf(event_names, 'add_to_cart')  AS cart_pos
  FROM click_stream
)
WHERE view_pos > 0 AND cart_pos > 0 AND cart_pos > view_pos
GROUP BY user_id;          -- 等价于 DISTINCT
```

---

### ✗ 原文(窗口函数示例输出)
原文示例 `sum(amount) OVER (PARTITION BY name)`,但在 SELECT 列表里用 `employee` 这个字段,而表结构里实际列名是 `name`(从建表 SQL 可看出)。

### ✓ 修正
建表语句中是 `name String`,后续 SQL 写 `PARTITION BY name`、结果列起名 `employee_total_sales`,展示表头中又把 `name` 渲染成了 `employee`。代码要么把表列改成 `employee`,要么把 SQL 中的 `employee` 全部改回 `name`,统一名字才能运行。

---

### ⚠ 表述不严谨
> "ClickHouse 是 OLAP 数据库,不擅长处理大表之间的 JOIN。"

### ✓ 补充
说"不擅长"在 25.x 已经不太准了——v22+ 引入了 `parallel_hash`、`grace_hash`(支持外溢到磁盘)、`partial_merge`、`direct`(走字典),配合 `optimize_min_inequality_conjunction_chain_length`、`max_bytes_in_join`、`join_algorithm='auto'` 等参数,大表 JOIN 已经可以正确跑完(只是比专门 OLTP 数据库慢)。准确说法:**大表 JOIN 仍然代价高,优先用宽表/字典/IN 规避;必须 JOIN 时调优 `join_algorithm`**。
