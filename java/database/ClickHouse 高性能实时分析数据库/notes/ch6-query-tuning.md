# 第6章 查询分析与性能调优

> 本章梳理自原文「第6章 查询分析与性能调优」。聚焦 ClickHouse 的高性能原理、`EXPLAIN` / `system.query_log` 诊断、模型与查询两条优化路径,以及一个完整的案例。

---

## 1. 性能瓶颈的五大根源

1. **数据模型设计不当**:`ORDER BY` 没用对、分区粒度不合理、类型选大、没有冗余宽表化。
2. **引擎选择不当**:核心数据用了 Log 家族;频繁聚合却没用 `AggregatingMergeTree`/`Projection`/MV。
3. **查询写法糟糕**:`SELECT *`、把所有过滤条件丢给 `WHERE`、跨大表 JOIN、`FINAL` 滥用。
4. **硬件资源限制**:CPU(向量化并行度不够)、内存(JOIN/GROUP BY 溢出)、I/O(磁盘小、未做读缓存)。
5. **配置不合理**:`max_threads`、`max_memory_usage`、`background_pool_size` 与负载不匹配。

本章主要解决前 3 项,这也是开发者能直接控制的部分。

---

## 2. 必须理解的三个核心原理

### 2.1 列式存储

```
   行存:        列存:
   r1 [a1 b1 c1]      colA: [a1 a2 a3 a4 ...]
   r2 [a2 b2 c2]      colB: [b1 b2 b3 b4 ...]
   r3 [a3 b3 c3]      colC: [c1 c2 c3 c4 ...]
   r4 [a4 b4 c4]
```

- `SELECT avg(a) FROM t`:列存只读 `colA.bin`,行存要读全部列。
- **调优启示**:永远只 `SELECT` 你需要的列,`SELECT *` 是性能杀手。

### 2.2 稀疏主键索引 + 数据裁剪

```
   ┌─── primary.idx (常驻内存) ──────────────────┐
   │ mark 0: ORDER BY key 起始值                  │
   │ mark 1: ...                                  │
   │ ...                                          │
   │ mark M: ...                                  │
   └─────────────────────────────────────────────┘
        │
        │  WHERE 命中主键时,二分定位需要读的 marks 区间
        ▼
   ┌─── Part 目录 ───────────────────────────────┐
   │ <col>.mrk      标记 → 字节偏移               │
   │ <col>.bin      压缩列数据                    │
   └─────────────────────────────────────────────┘
        │
        ▼
   只解压目标 granule,典型 N×8192 行
```

调优启示:
- `ORDER BY` 是 ClickHouse 性能调优的"第一公里"。
- `WHERE` 尽量命中主键前缀。
- 配合分区裁剪(`PARTITION BY`)和跳数索引,可以叠加效果。

### 2.3 向量化执行

```
   每次循环处理 1 行 (传统):     每次循环处理 8192 行 (向量化):

   for r in rows:                  for block in 8192-blocks:
       a, b = r.A, r.B                 vA = block.A
       out  = a + b                    vB = block.B
       emit(out)                       vOut = SIMD_add(vA, vB)
                                       emit(vOut)
```

CPU 缓存命中率高 + SIMD 指令并行 + 函数调用次数大幅减少 = 10× 起步的加速。

---

## 3. 查询诊断工具

### 3.1 `EXPLAIN` 系列

```sql
EXPLAIN SYNTAX  query;     -- 看语法重写、优化器重组后的 SQL
EXPLAIN PLAN    query;     -- 看执行计划骨架
EXPLAIN PIPELINE query;    -- 看物理流水线、并行度
EXPLAIN indexes = 1 query; -- 看主键 / 分区 / 跳数索引各筛掉了多少
EXPLAIN ESTIMATE query;    -- 估算要读的行数 / Part 数
EXPLAIN AST     query;     -- 看抽象语法树(调试)
```

关注 `ReadFromMergeTree` 节点输出:
```
ReadFromMergeTree
   Selected parts:  3
   Selected ranges: 5
   Selected marks:  17/12500   ← 主键索引筛掉绝大多数
```
若 `Selected marks` 接近总数,基本是全扫,索引没发挥作用。

### 3.2 `system.query_log`(慢查询日志的本质)

```sql
SELECT
    event_time, query_duration_ms,
    read_rows, read_bytes,
    memory_usage,
    result_rows, result_bytes,
    exception
FROM system.query_log
WHERE type = 'QueryFinish'
  AND is_initial_query = 1
ORDER BY query_duration_ms DESC
LIMIT 10;
```

关键指标:
| 字段 | 看什么 |
|------|--------|
| `query_duration_ms` | 总耗时 |
| `read_rows` / `read_bytes` | 真正扫描了多少 → 比 `result_*` 大很多即索引未发挥 |
| `memory_usage` | 高内存预警:JOIN / DISTINCT / 聚合溢出 |
| `ProfileEvents` | 细到"读 mark 多少次、解压多少 byte、CPU 多少 ns" |

### 3.3 其它系统表

| 表 | 用途 |
|----|------|
| `system.processes` | 当前正在执行的查询(可 KILL) |
| `system.parts` | 每个 Part 的大小、行数、合并代数,排查"小 Part 过多" |
| `system.merges` | 当前合并中的任务 |
| `system.mutations` | `ALTER UPDATE/DELETE` 异步任务进度 |
| `system.replication_queue` | 副本同步任务,卡住时排查用 |
| `system.metrics` / `system.events` | 瞬时 + 累计指标 |
| `system.errors` | 各类错误码计数 |

KILL 查询:
```sql
KILL QUERY WHERE query_id = '...';
```

---

## 4. Schema 层的"治本"调优

### 4.1 选对主键(`ORDER BY`)

反例:
```sql
ORDER BY EventDate
```
大部分查询是 `WHERE campaign_id = 'xxx'` → 索引几乎无效。

正解:
```sql
ORDER BY (campaign_id, browser, EventDate)
```

### 4.2 用 `LowCardinality(String)`

低基数文本列(状态、品牌、城市、device、event_type)是无脑收益。

### 4.3 `Projection` 自动预聚合(v21.8+)

```sql
ALTER TABLE user_events
ADD PROJECTION p_event_type (
    SELECT EventType, count()
    GROUP BY EventType
);
```

之后 `SELECT EventType, count() ... GROUP BY EventType` 会**自动**走投影。投影是 ClickHouse 内部维护的二级"物化视图",对应用透明,无需重写 SQL。

> 对比物化视图:Projection 与原表绑定、自动维护,但只能定义"在表内的子集查询";MV 更灵活,可写到任意目标表,但需要自己维护数据流。

### 4.4 类型最小化、能不 `Nullable` 就不 `Nullable`

```sql
ALTER TABLE my_table MODIFY COLUMN country LowCardinality(String);
```

---

## 5. 查询层的"治标"调优

### 5.1 `PREWHERE`

`PREWHERE` 是 ClickHouse 特有的"先粗后精"过滤:在数据从磁盘解压之前用一个轻量过滤,减少 I/O。

```sql
SELECT UserID, URL
FROM hits
PREWHERE CounterID = 123             -- 廉价、选择性强的列
WHERE   URL LIKE '%/some/path%';     -- 重过滤
```

经验:
- 主键列保留在 `WHERE`(ClickHouse 自己会用索引)。
- 选择性强、未走索引的非主键条件 → 显式放 `PREWHERE`。
- 大部分情况下 `optimize_move_to_prewhere = 1` 会自动做,但显式写更稳。

### 5.2 避免 `SELECT *`

```sql
-- ✗ 全列扫,内存+IO 都浪费
SELECT * FROM events WHERE user_id = 123 LIMIT 100;

-- ✓ 只取需要的
SELECT event_time, event_type FROM events WHERE user_id = 123 LIMIT 100;
```

### 5.3 `JOIN`

- 大表 × 小表:用 `GLOBAL ANY LEFT JOIN`,或考虑改字典 `dictGet`。
- 大表 × 大表:能不 JOIN 就不 JOIN(宽表化);必须时调 `join_algorithm = 'parallel_hash' / 'grace_hash'`。
- `IN` 子查询 / `GLOBAL IN` 通常比 `JOIN` 更高效。

### 5.4 `GROUP BY`

- 高基数列 `GROUP BY` 会消耗大量内存;必要时 `GROUP BY ... WITH TOTALS` 或 `SET max_bytes_before_external_group_by = ...` 让其溢出到磁盘。
- 固定 `GROUP BY` 模式 → 用物化视图 / Projection 预聚合。

### 5.5 慎用 `FINAL`

`FINAL` 会在查询时强制合并 `Replacing/Collapsing/Aggregating` 表的去重/折叠/聚合逻辑,**单线程**执行,大表会很慢。能用 `GROUP BY ... HAVING ...` 改写就别用 FINAL。

### 5.6 合理批量插入

每次 INSERT ≥ 1000 行;Kafka 引擎的 `kafka_max_block_size`、`stream_flush_interval_ms` 也是按这个思路调。

---

## 6. 案例:一步步把 30s 干到 50ms

**业务**:仪表盘按"活动 + 浏览器"统计 UV / PV。数据量大,但每次查询只关心一个 `campaign_id`。

### Step 0 — 原始表(性能陷阱)
```sql
CREATE TABLE visits (
    EventDate Date, UserID UInt64,
    campaign_id String, browser String, ...
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY EventDate;            -- ⚠ 主键全是日期,campaign 过滤走不上
```
原查询:
```sql
SELECT browser, count()
FROM visits
WHERE campaign_id = 'camp_xyz'
GROUP BY browser;
```
诊断:`read_rows` 巨大,耗时 30 s。

### Step 1 — 改主键(治本)
```sql
CREATE TABLE visits_optimized (...)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY (campaign_id, browser, EventDate);
```
效果:`campaign_id` 命中索引,耗时降到 ~2 s。

### Step 2 — 加 `LowCardinality`
```sql
campaign_id LowCardinality(String),
browser     LowCardinality(String),
```
效果:存储更小、`GROUP BY` 更快,耗时 ~800 ms。

### Step 3 — Projection 终极加速
```sql
ALTER TABLE visits_optimized
ADD PROJECTION p_campaign_browser (
  SELECT campaign_id, browser, count()
  GROUP BY campaign_id, browser
);
```
效果:命中投影,耗时 ~50 ms。

```
   30s ── 改主键 ──► 2s ── LowCardinality ──► 800ms ── Projection ──► 50ms
```

---

## 7. 调优自检清单

### Schema 阶段(治本)
- [ ] `ORDER BY` 是否覆盖最常用的 WHERE 列?顺序是否合理?
- [ ] 是否对低基数字符串列用了 `LowCardinality`?
- [ ] 日期/时间列是否用 `Date`/`DateTime` 而不是 `String`?
- [ ] 数值类型是否选了最小够用的?
- [ ] 是否对固定模式的聚合查询做了 Projection / 物化视图?
- [ ] 分区粒度是否合理(月/天,通常),是否避免小分区?
- [ ] 是否冗余了维度字段成为宽表?

### 查询阶段(治标)
- [ ] `SELECT` 是否只取了必要列?
- [ ] `WHERE` 是否走主键 / 跳数索引?(`EXPLAIN indexes = 1` 验证)
- [ ] 选择性强的非主键条件是否放进 `PREWHERE`?
- [ ] JOIN 右表是否足够小?是否使用 `GLOBAL` / 字典?
- [ ] `GROUP BY` 基数是否过高,是否能下推?
- [ ] 写入是否大批量?是否避免了一行一条?
- [ ] 是否避免了不必要的 `FINAL`?

---

## 8. 本章勘误区

### ✗ 原文
> "`PREWHERE` ...先在压缩数据上进行粗粒度过滤,只解压可能满足条件的列。"

### ✓ 补充
更准确:`PREWHERE` 的工作顺序是
1. **只读取并解压 `PREWHERE` 用到的列**(往往很短/很轻);
2. 根据这一步的结果生成一个"行掩码";
3. 再只读取 `SELECT` 和 `WHERE` 需要的其他列,且只读那些"行掩码为真"的行(以 granule 为粒度)。
"在压缩数据上过滤"是简化说法,真正的关键是**减少了其他大列的解压量**。

---

### ✗ 原文(`EXPLAIN PLAN` 示例)
```
ReadFromMergeTree (Selected parts: 1, Selected marks: 2)
...
ReadFromMergeTree (Selected parts: 50, Selected marks: 15000)
```
真实 EXPLAIN 输出是多行树状结构,不会用括号一行写完。

### ✓ 修正
真实输出更像:
```
(Expression)
ExpressionTransform
  (ReadFromMergeTree)
  ReadFromMergeTree
    Parts: 1/50
    Granules: 2/15000
```
括号格式只是讲义压缩,实际跑出来要按真实格式解读;`Granules` 的比例分子分母都很关键。

---

### ⚠ 表述模糊
> "投影是 ClickHouse 21.8 版本后引入的强大功能。它相当于一个**由 ClickHouse 自动维护的、预聚合的'物化视图'**。"

### ✓ 补充
Projection 与 MV 的关键差别:
1. Projection **必须与原表绑定**,生命周期跟随原表(`ALTER` 增/删);MV 是独立的表对象。
2. Projection 在查询时是**对原表透明**的——查询语句不变,自动选择走哪个投影;MV 要求显式查目标表。
3. Projection 不能跨表;MV 可以从一张表生成,落到另一张表。
两者可以同时存在,搭配使用。

---

### ⚠ 表述不严谨
> "如果只需要总计和小计,使用 `GROUP BY ... WITH TOTALS`,而不是 `UNION ALL` 两个查询。"

### ✓ 补充
`WITH TOTALS` 只能在末尾追加一行"全局总计",不能直接做"多级小计"。多级小计需要 `WITH ROLLUP` 或 `WITH CUBE`:
- `GROUP BY a, b WITH ROLLUP` → 输出 (a,b)、(a)、() 三层。
- `GROUP BY a, b WITH CUBE` → 输出 (a,b)、(a)、(b)、() 四层(所有维度组合)。
