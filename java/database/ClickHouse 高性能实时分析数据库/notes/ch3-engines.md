# 第3章 深入理解表引擎

> 本章梳理自原文「第3章 深入理解表引擎」(篇幅最大的一章)。覆盖:表引擎四大家族、MergeTree 物理结构、MergeTree 全家、Log 家族、集成引擎(MySQL/HDFS/Kafka)、特殊引擎(Buffer/Distributed/Dictionary/Memory/Null)、数据库引擎(Atomic/Ordinary/Lazy/MySQL)。

---

## 0. 鸟瞰:四大引擎家族

| 家族 | 代表 | 定位 |
|------|------|------|
| **MergeTree** | `MergeTree`、`Replacing/Summing/Aggregating/Collapsing/VersionedCollapsing` 等 + 它们的 `Replicated*` 复制版 | 海量数据 OLAP 核心,生产首选 |
| **Log** | `TinyLog`、`StripeLog`、`Log` | 临时/中间数据,顺序追加,无索引 |
| **Integration** | `MySQL`、`PostgreSQL`、`Kafka`、`HDFS`、`S3`、`URL`、`JDBC`、`ODBC`、`MongoDB` | 连接外部系统,实现"原地查询" |
| **Special** | `Memory`、`Buffer`、`Distributed`、`Dictionary`、`Null`、`View`、`MaterializedView`、`Merge`、`File`、`Join` | 解决特定问题 |

查询当前支持的所有引擎:
```sql
SHOW ENGINES;
```

---

## 1. MergeTree:核心中的核心

### 1.1 三大核心概念

- **分区 (`PARTITION BY`)**:把数据拆到不同物理目录。分区裁剪让 `WHERE` 命中分区时直接跳过其他分区。
- **数据片段 (Data Part)**:每次 `INSERT` 都会生成一个新的、独立的 Part(目录),内部按列存储且按 `ORDER BY` 排好序。
- **排序键 (`ORDER BY`)**:决定 Part 内部物理顺序;也作为 **稀疏主键索引**(默认 `index_granularity = 8192`,即每 8192 行打一个标记)。

```
┌───────────────────────  table  ───────────────────────┐
│                                                       │
│  PARTITION 202310/                                    │
│  ├── 202310_1_1_0/    ← Data Part(INSERT 1)         │
│  │   ├── primary.idx          稀疏主键索引            │
│  │   ├── partition.dat        分区值                  │
│  │   ├── minmax_<col>.idx     分区列 min/max          │
│  │   ├── columns.txt          列结构                  │
│  │   ├── count.txt            行数                    │
│  │   ├── checksums.txt        校验和                  │
│  │   ├── <colA>.bin / .mrk    列数据 + 标记文件       │
│  │   ├── <colB>.bin / .mrk                            │
│  │   └── skp_idx_<x>.idx/.mrk 跳数索引(如果有)       │
│  ├── 202310_2_2_0/    ← Data Part(INSERT 2)         │
│  └── 202310_1_2_1/    ← 后台合并后产生的更大 Part     │
│  PARTITION 202311/ ...                                │
└───────────────────────────────────────────────────────┘
```

> Part 目录名格式:`{partition}_{minBlockNum}_{maxBlockNum}_{level}[_{mutation}]`。`level` 表示合并代数,数字越大代表合并次数越多。

### 1.2 后台合并(Merge)

```
   INSERT  INSERT  INSERT  INSERT
     │       │       │       │
     ▼       ▼       ▼       ▼
   Part1   Part2   Part3   Part4   ← 同分区下产生很多小 Part
     \       │      /       /
      \      │     /       /
       \     │    /       /
        ▼    ▼   ▼       /
       ┌───────────────┐/
       │   Merged Part │   ← 后台合并线程异步合并
       │   (更大、有序)│
       └───────────────┘
```

- 合并仅在 **同一分区内** 进行,不同分区永远不会合并。
- 不同变种(Replacing/Summing/Aggregating/Collapsing)就是在合并这一步做不同语义。
- `OPTIMIZE TABLE t [PARTITION ...] [FINAL]` 可手动触发合并,生产不要频繁用。

### 1.3 完整 DDL

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER c]
(
    col1 Type1 [DEFAULT|MATERIALIZED|ALIAS expr] [CODEC(...)] [TTL ...],
    ...
    INDEX idx_name expr TYPE bloom_filter|minmax|set(N)|tokenbf_v1|ngrambf_v1 (...)
        GRANULARITY g,
    PROJECTION p_name (SELECT cols [GROUP BY ...] [ORDER BY ...])
)
ENGINE = MergeTree()
ORDER BY expr                          -- 必填
[PARTITION BY expr]
[PRIMARY KEY expr]                     -- 默认等于 ORDER BY,可指定为 ORDER BY 的前缀
[SAMPLE BY expr]
[TTL expr [DELETE | TO DISK 'x' | TO VOLUME 'y'] [WHERE ...] [GROUP BY ...]]
[SETTINGS index_granularity = 8192, ...]
```

关键 SETTINGS:

| 名称 | 含义 | 默认 |
|------|------|------|
| `index_granularity` | 稀疏索引粒度(每多少行打一个 mark) | 8192 |
| `index_granularity_bytes` | 自适应粒度的字节阈值 | 10 MiB(`10*1024*1024`) |
| `enable_mixed_granularity_parts` | 开启自适应粒度 | 1 |
| `merge_with_ttl_timeout` | TTL 合并的最小间隔 | 14400 秒 |
| `storage_policy` | 多盘/分级存储策略 | `default` |

---

## 2. MergeTree 全家:每种变种解决什么

### 2.1 `MergeTree`(基础)

无去重、无聚合,纯按 `ORDER BY` 排序存储。**主键允许重复**。

```sql
CREATE TABLE website_hits(
  EventDate Date, CounterID UInt32, UserID UInt64, URL String, Income UInt32
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, UserID);
```

### 2.2 `ReplacingMergeTree(ver)`:按主键去重

合并时**同排序键**的多行,只保留 `ver` 列最大的一行。`ver` 省略时按写入顺序保留"最后一行"。

```sql
CREATE TABLE user_profiles (
  user_id UInt64, profile String, updated_ts DateTime
) ENGINE = ReplacingMergeTree(updated_ts)
ORDER BY user_id;
```

⚠️ **去重只发生在合并时**,任意时刻查询都可能看到未合并的副本行。要"立即看到去重后的结果",查询可加 `FINAL`(性能下降)或 `OPTIMIZE TABLE ... FINAL`(手动触发合并)。

### 2.3 `SummingMergeTree([cols])`:按主键求和

合并时同排序键的多行,数值列做 `sum`,非主键的字符串列保留第一行的值。

```sql
CREATE TABLE daily_stats (
  day Date, page_id UInt32, visits UInt64, clicks UInt64
) ENGINE = SummingMergeTree()       -- 不指定则 sum 所有非排序键的数值列
ORDER BY (day, page_id);
```

### 2.4 `AggregatingMergeTree`:增量预聚合

存储聚合的**中间状态**(`AggregateFunction(func, types)`),需要配合 `*State` 写入、`*Merge` 查询。常作为物化视图的目标表。

```sql
CREATE TABLE agg_table (
  id UInt8,
  ctime Date,
  cnt          AggregateFunction(count),
  total_money  AggregateFunction(sum, UInt64)
) ENGINE = AggregatingMergeTree()
PARTITION BY toDate(ctime)
ORDER BY (id, ctime);

-- 写入:用 -State
INSERT INTO agg_table
SELECT id, ctime, countState(), sumState(money)
FROM detail_table GROUP BY id, ctime;

-- 查询:用 -Merge
SELECT id, ctime, countMerge(cnt), sumMerge(total_money)
FROM agg_table GROUP BY id, ctime;
```

### 2.5 `CollapsingMergeTree(Sign)`:用 ±1 抵消"删除"

引入 `Sign Int8` 列(只能是 `1` 或 `-1`)。合并时,**同排序键**的 `+1` 和 `-1` 成对抵消;若数量不等,则保留"最后一条 +1"和"第一条 -1"。

```sql
CREATE TABLE user_sessions (
  user_id String, session_id String,
  start_time DateTime,
  Sign Int8
) ENGINE = CollapsingMergeTree(Sign)
ORDER BY (user_id, session_id);
```

**查询"当前有效行"的安全写法**(不依赖合并):
```sql
SELECT user_id, session_id, max(start_time) AS t, sum(Sign) AS state
FROM user_sessions
GROUP BY user_id, session_id
HAVING state > 0;
```

⚠️ 对**乱序写入**敏感(`-1` 比 `+1` 先到会折叠失败)。要解决乱序,用下面的 Versioned 版本。

### 2.6 `VersionedCollapsingMergeTree(Sign, Version)`:乱序也能折叠

在 `Sign` 之外加 `Version`(整数,如时间戳/递增 ID)。`-1` 抵消同排序键 + 同 `Version` 的 `+1`;不同 Version 的行按版本最大者保留。**乱序也能正确折叠**。

```sql
CREATE TABLE app_configs (
  config_key String, config_value String,
  Version UInt64, Sign Int8
) ENGINE = VersionedCollapsingMergeTree(Sign, Version)
ORDER BY config_key;
```

### 2.7 `CoalescingMergeTree`(新引擎)

合并时,同排序键的多行,**每列取最新非 NULL 值**——可以理解为列级 `last_value`。用于稀疏更新的报表表。

> 注意:这个引擎是 24.x 才进入主线的较新引擎,旧版本可能没有。

### 2.8 `Replicated*` 前缀:多副本版本

所有 MergeTree 变种都有 `Replicated{X}MergeTree(zk_path, replica_name)` 版本,通过 ZooKeeper/Keeper 协调副本同步,见第 7 章。

---

## 3. 物化视图驱动的预聚合(`MergeTree + AggregatingMergeTree + MV`)

物化视图在 ClickHouse 里是 **插入触发器**:它本身不存数据,只把"对源表的写入"转换成"对目标表的写入"。

### 3.1 触发链

```
   ┌──────────────────────────────────────────────────────────────────┐
   │                                                                  │
   │  Application                                                     │
   │      │  INSERT  raw_logs (...)                                   │
   │      ▼                                                           │
   │  ┌───────────────────┐                                           │
   │  │  raw_logs         │ (MergeTree)  ← 真正的源数据落地            │
   │  └────────┬──────────┘                                           │
   │           │  触发已绑定的 MV                                     │
   │           ▼                                                      │
   │  ┌───────────────────────────┐                                   │
   │  │  mv_daily_summary         │  ←  仅是触发器(没有数据!)        │
   │  │  SELECT toDate(...) AS d, │                                   │
   │  │         countState() AS pv│                                   │
   │  │  FROM raw_logs            │                                   │
   │  │  GROUP BY d               │                                   │
   │  └────────┬──────────────────┘                                   │
   │           │  转换后的"中间状态"行                                │
   │           ▼                                                      │
   │  ┌───────────────────┐                                           │
   │  │ daily_summary     │ (AggregatingMergeTree)  ← 真正落地        │
   │  │  d Date           │                                           │
   │  │  pv  Agg(count)   │                                           │
   │  │  uv  Agg(uniq,UInt64)                                         │
   │  └────────┬──────────┘                                           │
   │           │  查询用 *Merge                                       │
   │           ▼                                                      │
   │  SELECT d, countMerge(pv), uniqMerge(uv) ...                     │
   │                                                                  │
   └──────────────────────────────────────────────────────────────────┘
```

### 3.2 DDL 模板

```sql
-- ① 源表
CREATE TABLE raw_logs (log_time DateTime, user_id UInt64, url String)
ENGINE = MergeTree() PARTITION BY toYYYYMM(log_time) ORDER BY log_time;

-- ② 聚合目标表
CREATE TABLE daily_summary (
  summary_date Date,
  pv AggregateFunction(count),
  uv AggregateFunction(uniq, UInt64)
) ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(summary_date)
ORDER BY summary_date;

-- ③ 物化视图(转换 + 路由,不存数据)
CREATE MATERIALIZED VIEW mv_daily_summary TO daily_summary AS
SELECT toDate(log_time) AS summary_date,
       countState() AS pv,
       uniqState(user_id) AS uv
FROM raw_logs
GROUP BY summary_date;
```

要点:
- `POPULATE` 关键字会在创建 MV 时把源表历史数据全量灌进去(慎用,资源大)。
- MV 本身的 `SELECT` 不能修改,要改逻辑只能 `DROP VIEW` + `CREATE`。
- 删除 MV **不会** 删目标表。
- 暂停/恢复 MV:`DETACH TABLE mv` / `ATTACH TABLE mv`。

详见第 5 章 5.6 节。

---

## 4. Log 家族(轻量,不要放生产核心数据)

| 引擎 | 是否多文件 | 是否有 marks 文件 | 是否并发读 | 适用 |
|------|-----------|-------------------|-----------|------|
| `TinyLog` | 每列一个 .bin | 无 | 单线程读写 | 极小数据、单元测试 |
| `Log` | 每列一个 .bin | 有 `__marks.mrk` | 支持并发读 | 中间结果,小批量 |
| `StripeLog` | 所有列存在一个 `data.bin` | 有 marks | 支持并发读 | 列多但每列小的临时表 |

共同特征:
- **无索引**,查询是全表扫描。
- **写时锁表**,不适合高并发写。
- **原子追加写**,写入要么完整成功要么失败。

> ⚠️ 警告:任何核心业务表都不应该用 Log 家族。

---

## 5. 集成引擎:打破数据孤岛

集成引擎是**代理/连接器**,自己不存数据,把查询翻译成对外部系统的请求。

### 5.1 MySQL 引擎(表级别)

```sql
CREATE TABLE mysql_users (
  user_id UInt32, user_name String, registration_city String
) ENGINE = MySQL('host:3306', 'db', 'users', 'user', 'pwd');

-- 联邦查询:本地事实表 JOIN 远程维度表
SELECT o.order_id, o.amount, u.user_name
FROM local_orders o
JOIN mysql_users u ON o.user_id = u.user_id;
```

```
   ┌──────────────────────┐    SELECT ...        ┌────────────────────┐
   │  ClickHouse Server   │ ───────────────────► │  MySQL Server      │
   │                      │                      │                    │
   │  ┌────────────────┐  │ ◄─────结果集─────── │  users 表          │
   │  │ mysql_users    │  │                      └────────────────────┘
   │  │ (引擎=MySQL)   │  │
   │  └─────┬──────────┘  │
   │        │             │
   │        │  在本地参与 │
   │        │  JOIN/聚合  │
   │        ▼             │
   │  返回给 Client       │
   └──────────────────────┘
```

可写入(`INSERT`),但生产中通常只读使用。

### 5.2 HDFS 引擎 / `hdfs()` 表函数

```sql
-- 一次性查询
SELECT *
FROM hdfs(
  'hdfs://nn:9000/logs/dt=2023-11-15/*.parquet',
  'Parquet',
  'event_time DateTime, level String, message String'
) LIMIT 10;

-- 持久表
CREATE TABLE hdfs_logs (
  event_time DateTime, level String, message String
) ENGINE = HDFS('hdfs://nn:9000/logs/dt=*/', 'Parquet');
```

支持通配符 `*`、`?`、`{a,b}`、`{n..m}`。

### 5.3 Kafka 引擎(适配器,不存数据)

ClickHouse 通过 Kafka 引擎做 **流式摄入**,典型架构 = `Kafka 引擎表 + MV + MergeTree 目标表`。

```
   ┌──────────────────────┐                                            
   │   Kafka Topic        │  实时事件流                                
   └──────────┬───────────┘                                            
              │  ClickHouse 持续 consume                              
              ▼                                                       
   ┌──────────────────────┐                                            
   │ Kafka 引擎表(管道) │  ← 不落地、不支持二次查询                  
   │  SELECT 一次即消费   │                                            
   └──────────┬───────────┘                                            
              │  触发                                                 
              ▼                                                       
   ┌──────────────────────────┐                                       
   │ Materialized View(水泵)│                                       
   │  从 Kafka 表 SELECT     │                                       
   └──────────┬───────────────┘                                       
              │  转换 + INSERT                                        
              ▼                                                       
   ┌──────────────────────┐                                            
   │ MergeTree 目标表     │  ← 真正落地、可查询                       
   └──────────────────────┘                                            
```

DDL:

```sql
CREATE TABLE account_store (
  user_id UInt64, name String, city String
) ENGINE = MergeTree() PARTITION BY city ORDER BY user_id;

CREATE TABLE account (
  user_id UInt64, name String, city String
) ENGINE = Kafka
SETTINGS
  kafka_broker_list   = 'k1:9092,k2:9092,k3:9092',
  kafka_topic_list    = 'click_data',
  kafka_group_name    = 'g1',
  kafka_format        = 'JSONEachRow',          -- ⚠ 注意是 JSONEachRow,不是 JSON
  kafka_num_consumers = 1,
  kafka_handle_error_mode = 'stream';

CREATE MATERIALIZED VIEW user_actions_pump TO account_store AS
SELECT user_id, name, city FROM account;
```

监控消费延迟:
```sql
SELECT table, partition, last_committed_offset, current_offset,
       (current_offset - last_committed_offset) AS lag, last_error
FROM system.kafka_consumers
WHERE table = 'account';
```

容错:`kafka_skip_broken_messages = N` 跳过 N 条解析失败的"毒丸消息"。

---

## 6. 特殊引擎

### 6.1 `Buffer`:写入缓冲

把高频小批量写入聚合成大批,再 flush 到目标表。**注意:数据先在内存,服务崩了会丢**。

```sql
CREATE TABLE logs_buffer (
  timestamp DateTime, level String, message String
) ENGINE = Buffer(
    default,        -- 目标库
    logs_mergetree, -- 目标表
    16,             -- num_layers
    10, 100,        -- min_time, max_time
    10000, 1000000, -- min_rows, max_rows
    10000000, 100000000 -- min_bytes, max_bytes
);
```

只要某一个 **max_** 阈值达到就会 flush。

### 6.2 `Distributed`:分片路由器(自己不存数据)

```sql
ENGINE = Distributed(cluster_name, db, local_table, sharding_key)
```

- 查询:把 SQL 下发到所有分片的本地表并行执行,协调节点聚合结果。
- 写入(可选):按 `sharding_key` 路由到对应分片。
- 详细机制见第 7 章。

### 6.3 `Dictionary`:用字典替代 JOIN(尤其小维度表)

```
   ┌─────────────────────┐        启动/定时刷新       ┌──────────────────┐
   │  外部源:           │ ─────────────────────────► │  ClickHouse      │
   │  MySQL / CH 表 /   │                            │  内存字典缓存    │
   │  HTTP / 文件 / ... │                            │ (key -> attrs)   │
   └─────────────────────┘                            └─────────┬────────┘
                                                               │
                                          dictGet(name,attr,id)│
                                                               ▼
                                              ┌─────────────────────────┐
                                              │   SELECT ... 中点对点    │
                                              │   O(1) 查询,无需 JOIN  │
                                              └─────────────────────────┘
```

XML 配置示例:

```xml
<dictionaries>
  <dictionary>
    <name>user_info_dict2</name>
    <source>
      <clickhouse>
        <host>localhost</host><port>9000</port>
        <user>default</user><password/>
        <db>learning</db><table>user_info</table>
      </clickhouse>
    </source>
    <layout><hashed/></layout>
    <structure>
      <id><name>user_id</name></id>
      <attribute><name>user_name</name><type>String</type><null_value/></attribute>
      <attribute><name>city</name><type>String</type><null_value/></attribute>
    </structure>
    <lifetime><min>300</min><max>360</max></lifetime>
  </dictionary>
</dictionaries>
```

`config.xml` 中引入:
```xml
<dictionaries_config>/etc/clickhouse-server/dicts/*.xml</dictionaries_config>
```

查询使用:
```sql
SELECT view_time, user_id, url,
       dictGet('user_info_dict2', 'user_name', user_id) AS user_name,
       dictGet('user_info_dict2', 'city',      user_id) AS city
FROM page_views;
```

> 现代写法:用 SQL DDL 创建字典(`CREATE DICTIONARY ... ENGINE = ...`),不再依赖 XML。

### 6.4 `Memory`

数据放内存,**不压缩**,服务重启全部消失。极速读写,适合临时表 / 测试。

### 6.5 `Null`

写入即丢弃,读取永远为空。常用于:
1. 与 Distributed 配对,做单向写入入口。
2. 性能基准测试(衡量传输/解析开销)。
3. 上游有 MV 把数据"分流"到多个目标表时,源表本身用 Null 引擎(成为纯触发点)。

---

## 7. 综合实战:Buffer + ReplicatedMergeTree + Distributed

经典生产架构(适合海量日志):

```
   App (logger)                                                    
        │                                                          
        │  INSERT 高频小批量                                       
        ▼                                                          
   ┌───────────────────────────────┐                                
   │  Node 上的 logs_buffer        │  ← 内存缓冲                   
   │  (Buffer 引擎)                │                                
   └─────────┬─────────────────────┘                                
             │  达到阈值 flush                                      
             ▼                                                      
   ┌───────────────────────────────┐    ZK 同步操作日志              
   │  logs_local (ReplicatedMT)    │ ◄─────────────► ┌──────────────┐
   │  每分片每副本一张             │                  │  ZooKeeper / │
   └─────────┬─────────────────────┘                  │   Keeper     │
             │                                         └──────────────┘
             │ 各分片并行                                              
             │ 副本间互相同步                                          
             ▼                                                         
   ┌───────────────────────────────┐                                   
   │  logs_distributed (全集群一张)│  ← 客户端查询入口                
   │  (Distributed 引擎)           │                                   
   └───────────────────────────────┘                                   
```

DDL:

```sql
-- 本地分片表(每个分片每个副本都有)
CREATE TABLE default.logs_local ON CLUSTER my_cluster (
  timestamp DateTime, hostname String, level String, message String
) ENGINE = ReplicatedMergeTree(
    '/clickhouse/tables/{shard}/logs_local', '{replica}'
)
PARTITION BY toYYYYMM(timestamp)
ORDER BY (timestamp, hostname);

-- 写入缓冲(每节点一张,指向本节点的 logs_local)
CREATE TABLE default.logs_buffer ON CLUSTER my_cluster (
  timestamp DateTime, hostname String, level String, message String
) ENGINE = Buffer(default, logs_local, 16, 10, 100, 10000, 1000000, 10485760, 104857600);

-- 查询入口(任一节点都可查)
CREATE TABLE default.logs_distributed ON CLUSTER my_cluster (
  timestamp DateTime, hostname String, level String, message String
) ENGINE = Distributed(my_cluster, default, logs_local, rand());
```

---

## 8. 数据库引擎(房间)

### 8.1 `Ordinary`(老默认)

- 元数据 = 每个表一个 `.sql` 文件。
- `RENAME` / `EXCHANGE` **不是原子**。
- 现仅为兼容老库存在,不推荐新建。

### 8.2 `Atomic`(v20.10+ 起默认)

- 元数据集中管理,DDL **原子**,支持 `EXCHANGE TABLES`、原子 RENAME、`DROP TABLE` 不立即物理删除(可恢复 `UNDROP TABLE`,21.x+)。
- 新项目应统一用 Atomic。

```sql
CREATE DATABASE atomic_db ENGINE = Atomic;
EXCHANGE TABLES t1 AND t2;          -- 一行原子搞定
```

### 8.3 `Lazy(seconds)`:为海量小表而生

只在内存中保留最近 `seconds` 秒被访问过的表,其余的从内存卸载,从而压缩元数据占用、加快启动。表本身仍在磁盘上。

```sql
CREATE DATABASE lazy_db ENGINE = Lazy(60);
```

### 8.4 `MySQL` / `PostgreSQL`(数据库级映射)

把整个外部数据库以"任意门"形式映射进来,无需逐表 `CREATE TABLE`。

```sql
CREATE DATABASE mysql_bridge
ENGINE = MySQL('host:3306', 'db', 'user', 'pwd');

SHOW TABLES FROM mysql_bridge;
SELECT * FROM mysql_bridge.products LIMIT 5;
```

适合查维度表/小表;海量事实表仍应导入本地。

---

## 9. 本章勘误区

### ✗ 原文(3.3 Replicated)
> "ZooKeeper 路径:`/clickhouse/tables/{shard}/table_name`"

### ✓ 补充
官方推荐路径同时包含 **数据库名**,例如:
`/clickhouse/tables/{shard}/{database}/{table}` 或带 `{uuid}` 的 Atomic 风格路径
`/clickhouse/tables/{uuid}/{shard}`。
仅用表名会在多个库存在同名表时冲突,踩过坑的人不少。

---

### ✗ 原文(3.3.5 AggregatingMergeTree 示例)
```sql
cnt AggregateFunction(count, UInt64) ,
total_money AggregateFunction(sum, UInt64) ,   -- 注意末尾逗号
```
列定义末尾多了 `,`,后面紧跟 `) ENGINE=...`,严格 ANSI 解析会报错;ClickHouse 旧版本也会报错。

并且写入示例:
```sql
INSERT INTO agg_table SELECT id, ctime, countState(id), sumState(money) ...
```
- `countState(id)` 在新版 ClickHouse 中通常应写 `countState()`(无参数),因为 `count(*)` 不带列。若坚持 `count(col)`,目标列类型必须是 `AggregateFunction(count, T)` 与实参类型一致。
- 输出行
```
   id  | ctime      | countMerge(cnt) | sumMerge(total_money)
    1  | 2025-08-06 |               9 |                  1500
```
是基于"插入 5 行明细 +1 行预期"伪造的结果——按照示例插入语句实际是 3 行明细(id=1) 与 2 行明细(id=2),`countMerge` 应为 3 / 2,`sumMerge` 应为 500 / 400。**示例数字不自洽**,讲课时请用真实输出。

---

### ✗ 原文(3.3.6 CoalescingMergeTree 示例)
```sql
INSERT INTO test_table VALUES (3,2),(4,1),(3,8);
```
而表结构是 `(id UInt32, name String, age UInt32)`——3 列,VALUES 只给了 2 列,语法直接报错。

### ✓ 正解
要么三列都给:
```sql
INSERT INTO test_table VALUES (3, 'x', 2),(4, 'y', 1),(3, NULL, 8);
```
要么明确写出要插入的列:
```sql
INSERT INTO test_table (id, age) VALUES (3,2),(4,1),(3,8);
```
且 `CoalescingMergeTree` 主要意义是合并稀疏 NULL 值,数据集需要构造 NULL,否则演示不出"列级最新非空"的效果。

---

### ✗ 原文(Kafka 引擎设置)
```
kafka_format = 'JSON',
```
紧接着输入是:
```
{"user_id": 101, "event": "login", "ts": "..."}
{"user_id": 102, "event": "purchase", "ts": "..."}
```

### ✓ 正解
**每行一个 JSON 对象** 的格式是 `JSONEachRow`,**不是** `JSON`。`JSON` 是带元数据的整体 JSON 格式(`{"meta":..., "data":[...]}`),用错会无法解析。

---

### ✗ 原文(Distributed 配置示例的 XML 根标签)
```xml
<clickhouse_remote_servers>
  <cluster1> ... </cluster1>
</clickhouse_remote_servers>
```

### ✓ 正解
新版 ClickHouse 的标准是:
```xml
<clickhouse>
  <remote_servers>
    <cluster1> ... </cluster1>
  </remote_servers>
</clickhouse>
```
(老版本根标签是 `<yandex>`,后兼容 `<clickhouse>`)。`<clickhouse_remote_servers>` 不是标准节点名,放在 `config.xml` 根下不会被识别。

---

### ✗ 原文(Buffer 配置中的"原子性")
> "Buffer 表不保证数据的原子性和持久性"

### ✓ 修正
**单条 INSERT 进入 Buffer 仍然是原子的**(要么写入缓冲、要么失败);Buffer 不保证的是**持久性**(服务崩溃丢失内存数据)和**跨表事务**。措辞上把"原子性"去掉、保留"持久性"更准确。

---

### ✗ 原文(集成引擎章节序号)
原文小节序号:`1 课程基本信息` → `2 核心概念` → `3 MySQL` → `4 HDFS` → `6 Kafka`,跳过了 `5`。

### ✓ 处理
这里把 Kafka 单独列出,不再保留断号。

---

### ⚠ 表述模糊
> "**Dictionary 解决与低基数维度表 JOIN 的性能瓶颈**"

### ✓ 补充
Dictionary 的精确适用条件:
1. 维度表能完整放进 **内存**(几 MB ~ 几 GB)。
2. 数据更新不频繁(或可接受 `lifetime` 内的陈旧)。
3. 关联方式是 **按主键 / 复合键的等值查找**。
高基数 + 复杂条件的 JOIN 不适合换字典。
