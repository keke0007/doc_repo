# 第4章 高性能模式设计

> 本章梳理自原文「第4章 高性能模式设计」。覆盖:Schema 设计五原则、数据类型、主键稀疏索引、跳数索引、数据导入。

---

## 1. 五个核心设计原则

### 1.1 宽表优先,反范式化

ClickHouse 是**为大宽表扫描而生**的。MySQL 里"三范式 + JOIN 三五张表"在 CH 里通常是反模式。能在数据接入层把维度字段冗余到事实表中,就尽量做宽表。

| 关系型思路 | ClickHouse 思路 |
|------------|----------------|
| `orders` JOIN `users` JOIN `products` | `orders_wide`(已带 `user_name`、`product_name`、`city` 等) |
| `UPDATE users` 后 JOIN 实时拿最新值 | 在数据管道做维度补全;或用 Dictionary 内存补全 |

### 1.2 选对排序键 `ORDER BY`

`ORDER BY` 决定**数据在磁盘上的物理顺序**,也是稀疏主键索引的依据,是 ClickHouse 性能调优的"第一公里"。

设计法则:
1. **把最常用作 WHERE 过滤的列放前面**。
2. **左前缀**有效:`ORDER BY (a, b, c)` 时,`WHERE a=...`、`WHERE a=... AND b=...` 都能用索引,单独 `WHERE c=...` 用不上。
3. **基数从高到低**:把过滤性强的列放前面,索引颗粒切分更均匀。
4. **不要塞太多列**:每加一列都会让 Part 内排序成本变大。

> 反例:大部分查询是 `WHERE campaign_id=...`,却 `ORDER BY EventDate` —— 主键索引几乎无效。

### 1.3 合理分区 `PARTITION BY`

- 分区是**逻辑/目录级别**的切分,便于裁剪 + 整分区删除/TTL。
- 常见策略:`toYYYYMM(event_date)` 按月、`toDate(event_date)` 按天。
- **禁忌**:分区粒度过细(按秒/分钟)→ 海量小 Part,合并压力大,反而慢。

经验值:每个分区数据量在 **1GB~100GB**、Part 数控制在 **几十到几百** 比较健康。

### 1.4 用最小够用的数据类型

| 列含义 | 别用 | 应用 |
|--------|------|------|
| 年龄 0~150 | `Int64` | `UInt8` |
| 国家代码 2 字符 | `String` | `FixedString(2)` 或 `LowCardinality(String)` |
| IP 地址 | `String` | `IPv4` / `IPv6` |
| 枚举状态(几十种) | `String` | `LowCardinality(String)` 或 `Enum8/16` |
| 日期 | `String '2024-05-30'` | `Date` / `Date32` |
| 时间戳 | `Int64` | `DateTime` / `DateTime64(3)` |
| 大概率为空 | `Nullable(...)` | 能不用就别用,`Nullable` 有额外开销 |

### 1.5 用引擎做"预计算"

固定模式的聚合查询应在写入侧消化:
- `AggregatingMergeTree` + 物化视图(预聚合)
- `Projection`(投影,自动维护的"内嵌物化视图")
- `LowCardinality` 字典编码

---

## 2. ClickHouse 数据类型一览

| 类别 | 类型 |
|------|------|
| 整数 | `Int8/16/32/64/128/256`,`UInt8/16/32/64/128/256` |
| 浮点 | `Float32 / Float64` |
| 高精度 | `Decimal(p,s)`、`Decimal32/64/128/256` |
| 字符串 | `String`、`FixedString(N)` |
| 低基数包装 | `LowCardinality(T)` |
| 日期/时间 | `Date`、`Date32`、`DateTime[(tz)]`、`DateTime64(p, tz)` |
| 网络 | `IPv4`、`IPv6` |
| 枚举 | `Enum8('a'=1,'b'=2)`、`Enum16(...)` |
| UUID / JSON | `UUID`、`JSON`(实验性,推荐生产用)、`Object('json')`(老,已弃用) |
| 复合 | `Array(T)`、`Tuple(T1,T2,...)`、`Map(K,V)`、`Nested(...)` |
| 可空 | `Nullable(T)` |
| 几何 | `Point`、`Ring`、`Polygon`、`MultiPolygon` |
| 特殊 | `AggregateFunction(...)`,`SimpleAggregateFunction(...)` |

### 2.1 `LowCardinality(String)` 详解

> 当一列字符串重复率很高(如国家、城市、设备类型,通常基数 < 10⁵~10⁶),用 `LowCardinality(String)` 几乎总是赚到。

```
┌──────────────── 列存原始 ───────────────┐     ┌──────── LowCardinality ────────┐
│ "Beijing"                                │     │ 字典: 1→"Beijing"             │
│ "Beijing"                                │     │       2→"Shanghai"            │
│ "Shanghai"                               │  →  │       3→"Guangzhou"           │
│ "Beijing"                                │     │ 实际存储列: [1,1,2,1,3,2,1...] │
│ "Guangzhou"                              │     │ (UInt8/16,压缩极佳)            │
└──────────────────────────────────────────┘     └────────────────────────────────┘
```

收益:存储变小、I/O 变少、`GROUP BY`/`WHERE` 命中字典直接比较整数。

### 2.2 综合示例:用户行为日志表

```sql
CREATE TABLE user_behavior (
    event_date    Date,
    event_time    DateTime,
    user_id       UInt64,
    event_type    LowCardinality(String),     -- 几十种事件类型 → 字典化
    country_code  FixedString(2),             -- 国家代码定长 2 位
    url           String,
    user_agent    Nullable(String),           -- 可能缺失
    extra_params  Map(String, String)         -- 动态属性
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY (event_date, event_type, user_id);
```

---

## 3. 索引体系:稀疏主键索引 + 跳数索引

### 3.1 稀疏主键索引(Sparse Primary Index)

MySQL 是**密集索引**(每行一条索引项);ClickHouse 是**稀疏索引**(每 `index_granularity` 行,默认 8192,打一个"标记")。

```
┌── primary.idx (内存中,极小) ─────────────────────────────────────┐
│ mark0 : (key="A",  rowOffset=0)                                  │
│ mark1 : (key="An", rowOffset=8192)                               │
│ mark2 : (key="Ap", rowOffset=16384)                              │
│ mark3 : (key="B",  rowOffset=24576)                              │
│ ...                                                               │
└───────────────────────────────────────────────────────────────────┘
        │                                                            
        │  查找 "Ant":二分定位到 mark1~mark2                        
        ▼                                                            
   读取 [16384, 24576) 这一段的列 .bin (一个 granule)                
   在 8192 行里精确匹配                                              
```

查询走主键时只需:
1. 在内存里二分定位 mark 区间。
2. 通过 `.mrk` 文件找到 `.bin` 中的字节偏移。
3. 解压并扫描这一段(典型 8192 行)。

### 3.2 主键索引设计要点

- 列的**顺序**比"放进哪些列"更重要。
- 主键不要太长(整数 32 位 > 字符串 > UUID;主键越长 `.idx` 越大,内存占用越大)。
- 可让 `PRIMARY KEY` 比 `ORDER BY` 短(前缀),减少内存占用:
  ```sql
  ORDER BY (campaign_id, browser, EventDate)
  PRIMARY KEY (campaign_id, browser)
  ```

### 3.3 跳数索引(Data Skipping Index)

跳数索引附加在 **granule** 级别,用于对非主键列做"是否需要读这块"的预判。

| 类型 | 用途 | 适合场景 |
|------|------|---------|
| `minmax` | 记录列在 granule 内的最小/最大值 | 数值/日期,范围查询 |
| `set(N)` | 记录前 N 个唯一值(N=0 即无上限) | 低基数等值查询 |
| `bloom_filter([fp_rate])` | 概率性"是否包含" | 高基数等值/`has()`/`mapContains()` |
| `tokenbf_v1(size, hashes, seed)` | 把字符串分词后塞布隆 | LIKE/全文搜素 |
| `ngrambf_v1(n, size, hashes, seed)` | n-gram 切片后塞布隆 | 模糊匹配 |

DDL 语法:
```sql
INDEX idx_name <expr> TYPE <type>(...) GRANULARITY <g>
```

`GRANULARITY g` 表示**每多少个主索引颗粒做一个跳数索引块**;默认 `g=1`,即一颗粒一条目;`g=4` 时每 4 个颗粒(默认即 4×8192 行)做一个块,换取更小的索引文件、稍弱的过滤精度。

### 3.4 跳数索引建表/`ALTER` 示例

```sql
-- 建表时
CREATE TABLE user_behavior_with_index (
    event_date Date, event_time DateTime, user_id UInt64,
    event_type LowCardinality(String), url String,
    INDEX idx_url url TYPE bloom_filter(0.01) GRANULARITY 1
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY (event_date, event_type, user_id);

-- 后加
ALTER TABLE access_logs ADD INDEX idx_req_id  request_id TYPE bloom_filter() GRANULARITY 4;
ALTER TABLE access_logs ADD INDEX idx_code    http_code  TYPE set(0)         GRANULARITY 4;

-- 让索引对老数据生效(否则只对新写入的 Part 起作用)
ALTER TABLE access_logs MATERIALIZE INDEX idx_req_id;
```

### 3.5 验证索引是否生效

```sql
EXPLAIN indexes = 1
SELECT count() FROM access_logs WHERE request_id = 'xxx';
```

或开启日志后,在 `clickhouse-server.log` 查找:
```
... Selected N/M parts by partition key
... Selected n/m marks  by primary  key
... Selected k/l granules by skipping indexes  ← 这一行表示跳数索引在工作
```

### 3.6 索引选型小抄

| 列特征 | 推荐 |
|--------|------|
| 数值 / 日期 范围查询 | `minmax` |
| 低基数字符串等值 | `set(N)` |
| 高基数字符串等值 / `has(arr,...)` / `mapContains(...)` | `bloom_filter` |
| `LIKE '%abc%'` 等模糊匹配 | `tokenbf_v1` / `ngrambf_v1` |

---

## 4. 数据导入

### 4.1 黄金法则:大批量、少次数

```
   ⚡ 反例(慢、产生大量小 Part):                  ✅ 正例(一次大批):                        
   ┌─────────────┐                                   ┌─────────────┐                            
   │ INSERT 1行  │ ─►Part1                           │ INSERT       │                            
   │ INSERT 1行  │ ─►Part2                           │  10000 行   │ ─►Part1 (一个 Part)        
   │ INSERT 1行  │ ─►Part3                           └─────────────┘                            
   │ ...         │ ─►PartN                                                                       
   └─────────────┘                                                                              
   后台 Merge 不堪重负                                                                          
```

经验值:每次 INSERT **≥ 1000 行,理想 10K~100K 行**。如果数据源是流(Kafka、应用日志),在应用层 / 消息队列攒批再写。

### 4.2 常用导入手段

```bash
# CSV
clickhouse-client --query "INSERT INTO user_behavior FORMAT CSV" < behaviors.csv

# JSONEachRow(每行一个对象)
clickhouse-client --query "INSERT INTO t FORMAT JSONEachRow" < events.jsonl

# Parquet / ORC(列存格式,带 schema)
clickhouse-client --query "INSERT INTO t FORMAT Parquet" < part.parquet

# 远端文件 via 表函数
INSERT INTO t SELECT * FROM s3('https://.../*.parquet', 'Parquet');
INSERT INTO t SELECT * FROM url('http://.../data.csv',  'CSV');
INSERT INTO t SELECT * FROM hdfs('hdfs://.../*.orc',    'ORC');
```

### 4.3 常用 FORMAT 速查

| 格式 | 适合 |
|------|------|
| `CSV` / `TSV` | 通用,逗号/制表符 |
| `CSVWithNames` / `TSVWithNames` | 首行是列名 |
| `JSONEachRow` | 一行一个 JSON 对象,日志/事件流 |
| `JSON` | 整体一个对象(带 meta + data),BI 接口常用 |
| `Parquet` / `ORC` | 列存,与 Spark/Hadoop 互通 |
| `Avro` | 与 Kafka Schema Registry 配合 |
| `Native` | ClickHouse 内部格式,最高性能 |
| `Protobuf` | 跨语言 RPC 数据 |

---

## 5. 本章勘误区

### ✗ 原文
> "稀疏索引 (`primary.idx`):这个文件非常小,因为它只存储每个 Granule 的'路标'值。例如,如果 `ORDER BY` 是 `(event_date)`,那么索引文件里存的就是每个 Granule 的起始日期。"

### ✓ 补充
更准确:`primary.idx` 存放的是 **每个 granule 第一行的 PRIMARY KEY 完整值**(主键里所有列的拼接);标记到 `.bin` 字节偏移的映射另由 `<col>.mrk` / `<col>.mrk2` / `<col>.mrk3` 文件维护——两类文件配合才能完成"主键 → 数据块字节" 的定位。

---

### ✗ 原文(主键设计要点)
> "把基数更高(筛选能力更强)的列放在前面。例如 `ORDER BY (event_date, user_id)` 就比 `ORDER BY (user_id, event_date)` 要好"

### ✓ 修正
这一条**只在大多数查询用 `event_date` 过滤时成立**。"高基数放前面"不是普适法则,通用法则应当是 **"最常用于 WHERE 的列放前面"**。如果业务 90% 的查询是按 `user_id` 拉用户全量历史,那么 `(user_id, event_date)` 才是对的。所以原文这条要么去掉,要么改成 "把'最常出现在 WHERE 中的列'放前面,且尽量在它后面跟一个能进一步切分数据的列(如时间)"。

---

### ✗ 原文(GRANULARITY 注释)
```sql
INDEX idx_url url TYPE bloom_filter() GRANULARITY 1 -- 8192   2*8192
```
注释自相矛盾(1 不是 2)且没说清单位。

### ✓ 正解
`GRANULARITY g` 是**主键索引颗粒数的倍数**。配合 `index_granularity = 8192`:
- `GRANULARITY 1` → 每 8192 行做一个跳数索引块。
- `GRANULARITY 4` → 每 32768 行做一个跳数索引块。

---

### ✗ 原文(添加跳数索引)
```sql
ALTER TABLE access_logs ADD INDEX idx_code http_code TYPE set(0) GRANULARITY 4;
```
注释解释为"记录前 N 个唯一值",但 `set(0)` 在新版语义里表示"不限制集合大小",并不是常见的 N。如果想要"前 100 个"应写 `set(100)`。

### ✓ 补充
`set(0)` 的实际含义:**不设上限**,只要 granule 内的唯一值数量超出 ClickHouse 配置的 max,索引就退化(变成总是"可能存在")。对低基数列(几十个)写 `set(0)` 没问题;对高基数列写 `set(0)` 反而会让索引几乎失效。

---

### ⚠ 表述不严谨
> "用 `LowCardinality(String)` 替代任何字符串列。"

### ✓ 补充
`LowCardinality` **不**适合:
1. 唯一值数量极高(数百万级)的列——字典本身膨胀,反而变慢。
2. 主键的第一列——会破坏主键索引的设计直觉(可以用,但要清楚后果)。
3. 频繁变更值的列——字典命中率低,效率不如直接存字符串。
经验阈值:列基数 < 10⁶ 时考虑;高于这个量级要测一下。

---

### ⚠ 表述不准
> "**`PARTITION BY toYYYYMM(timestamp)`**: 按月对数据进行'物理切分'"

### ✓ 补充
分区是**目录级**的切分(每个分区一个独立的子目录),不是物理切分到不同存储。要把不同分区放到不同存储(冷/热分层),需要配合 `storage_policy` + `TTL ... TO DISK/VOLUME`。
