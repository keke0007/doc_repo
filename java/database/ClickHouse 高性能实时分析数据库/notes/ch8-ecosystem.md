# 第8章 融入现代数据栈

> 本章梳理自原文「第8章 融入现代数据栈」。原文这一章为大纲式骨架,以下基于大纲展开为可操作的知识点,并补充常见用法和最佳实践。

---

## 1. 数据注入(Ingestion)

ClickHouse 入库的"三大典型路径":

```
                          ┌────────────────────────┐
                          │ A. SQL INSERT          │
   App / API ─────────►   │   (clickhouse-driver,  │ ──► ClickHouse
                          │    JDBC,curl 等)       │
                          └────────────────────────┘

                          ┌────────────────────────┐
                          │ B. Kafka 引擎 + MV     │
   Producer ── Kafka ──►  │   持续 consume          │ ──► MergeTree 目标表
                          │   适配器,不存数据       │
                          └────────────────────────┘

                          ┌────────────────────────┐
                          │ C. 外部 ETL 工具       │
   各种源数据 ─────────►  │   Vector / Logstash /  │ ──► ClickHouse
                          │   Flink / Spark / NiFi │
                          └────────────────────────┘
```

### 1.1 Kafka 引擎(原生流式接入)

完整链路见第 3 章 5.3 节,核心三件套:
- Kafka 引擎表 = 消费者(适配器,不存数据)
- 物化视图 = 触发器 + 转换
- MergeTree 目标表 = 真正落地

### 1.2 Vector / Logstash / Fluentd

适合"日志/事件类"的轻量管道:
- **Vector**:ClickHouse 官方推荐,有 `clickhouse` sink,性能优秀,支持 TLS、批量、重试。
- **Logstash**:`clickhouse-logstash-plugin`,生态最广。
- **Fluentd / Fluent-bit**:Kubernetes 日志的事实标准。

### 1.3 Flink / Spark

适合"需要复杂 ETL / 流批一体"的场景:
- **Flink**:`flink-connector-clickhouse`,Sink 端可调批大小、并行度;严格 exactly-once 需要 2PC 或目标端用 `ReplacingMergeTree` 自去重。
- **Spark**:`spark-clickhouse-connector`(housepower 等社区)或 `spark-jdbc`(性能差,大数据量慎用)。

### 1.4 写入参数与最佳实践

| 项 | 建议值 |
|----|--------|
| 单批行数 | 1k ~ 100k 行/批 |
| 单批字节 | 几 MB ~ 几十 MB |
| 写入并发 | 不要超过 `max_threads / 节点数` |
| Distributed 写入 | 优先 `internal_replication=true`,只写主副本 |
| 重试 | 重试要保证幂等(主键唯一 / Replacing) |
| 错误处理 | `Kafka`: `kafka_skip_broken_messages`;批量插入用 `INSERT ... FORMAT JSONEachRow` + 行级跳过 |

---

## 2. 数据湖集成

核心思想:**不导入数据,直接查询远端文件**。

### 2.1 表函数

| 表函数 | 协议 / 存储 | 典型用法 |
|--------|------------|---------|
| `s3(url, [aws_key, aws_secret,] format, structure)` | AWS S3 / 兼容对象存储 | 直接查 Parquet/ORC |
| `gcs(url, ...)` | GCS | 同上 |
| `azureBlobStorage(...)` | Azure Blob | 同上 |
| `hdfs(uri, format, structure)` | HDFS | 见第 3 章 |
| `url(url, format, structure)` | HTTP/HTTPS | 任意网络文件 |
| `file(path, format, structure)` | 本地文件 | 单机测试 |
| `mysql / postgresql / mongodb(...)` | 关系/文档数据库 | 联邦查询 |
| `cluster(cluster, table[, sharding_key])` | 分布式调用 | 在某集群上跑一句查询 |
| `merge(db, regexp)` | 同库下名字匹配的多张表 | 横向合并 |

### 2.2 直接查 S3 Parquet 示例

```sql
SELECT count() FROM s3(
    'https://my-bucket.s3.amazonaws.com/dt=2024-05-*/part-*.parquet',
    'Parquet',
    'event_time DateTime, user_id UInt64, url String'
);

-- 一次性导入到本地表
INSERT INTO local_events
SELECT * FROM s3(
    'https://my-bucket.s3.amazonaws.com/dt=2024-05-30/*.parquet',
    'AWS_KEY', 'AWS_SECRET',
    'Parquet'
);
```

通配符:`*`、`?`、`{a,b,c}`、`{n..m}`。

### 2.3 持久 `S3` 引擎表

```sql
CREATE TABLE s3_logs (
    event_time DateTime, url String, ...
) ENGINE = S3(
    'https://bucket.s3.region.amazonaws.com/path/*.parquet',
    'Parquet'
);
```

### 2.4 何时"原地查询" vs 何时"先导入"

| 选择 | 原地查询 | 先导入 |
|------|---------|--------|
| 适合 | 小批量 ad-hoc、与本地数据 JOIN、低频探查 | 高频/大量、需要走主键索引的 OLAP |
| 优点 | 不复制数据、节省存储 | 用上 ClickHouse 全部加速能力 |
| 缺点 | 远端 I/O、无主键索引、按需扫描整文件 | 复制数据、有冗余 |

---

## 3. BI 与可视化

ClickHouse 提供 HTTP 8123 + 兼容 MySQL/PostgreSQL 协议,各种 BI 工具都能接:

| 工具 | 连接方式 | 特点 |
|------|---------|------|
| **Grafana** | 官方 `grafana-clickhouse-datasource` | 监控看板首选,时序数据 |
| **Apache Superset** | SQLAlchemy + `clickhouse-connect` | 开源 BI 老大,自助 SQL/可视化 |
| **Metabase** | 官方 ClickHouse driver | 易用,适合非技术用户 |
| **Tableau** | ODBC / JDBC | 企业 BI |
| **Power BI** | ODBC / Direct Query | 微软生态 |
| **Apache Zeppelin / DataGrip / DBeaver** | JDBC | SQL 开发 |

接入参数(以 HTTP 为例):
- Host: `<server>` Port: 8123(HTTPS 用 8443)
- User / Password
- Database: `default` 或自定义

性能建议:
- BI 工具默认会 `SELECT *`,务必在视图里**只暴露需要的列**。
- 给 BI 用专门用户,通过 `<readonly>` profile 限制只读、限制 `max_memory_usage` / `max_execution_time`。
- 复杂指标在 ClickHouse 端做物化视图 / Projection,前端只查预聚合表。

---

## 4. 编程语言客户端

### 4.1 Python

- `clickhouse-connect`(官方,HTTP 协议,**推荐**)。
- `clickhouse-driver`(社区,TCP/Native 协议,性能更高)。
- `chdb`(嵌入式 ClickHouse,内置在 Python 进程里,无需 server)。

```python
# clickhouse-connect 示例
import clickhouse_connect

client = clickhouse_connect.get_client(
    host='localhost', port=8123,
    username='default', password='',
    database='learning',
)

# 查询
df = client.query_df('SELECT browser, count() FROM t_web_hits GROUP BY browser')
print(df)

# 批量写入(推荐用 insert + Numpy/Pandas)
import pandas as pd
data = pd.DataFrame({...})
client.insert_df('t_web_hits', data)
```

### 4.2 Java

`clickhouse-jdbc`(官方)+ `clickhouse-client`(原生 TCP)。BI 工具几乎都通过 JDBC 接入。

```java
String url = "jdbc:clickhouse://host:8123/default";
try (Connection conn = DriverManager.getConnection(url, "default", "")) {
    try (PreparedStatement ps = conn.prepareStatement(
            "INSERT INTO t (a,b,c) VALUES (?,?,?)")) {
        for (Row r : rows) {
            ps.setLong(1, r.a); ps.setString(2, r.b); ps.setDouble(3, r.c);
            ps.addBatch();
        }
        ps.executeBatch();
    }
}
```

### 4.3 Go

`clickhouse-go`(官方),支持原生 TCP 和 HTTP。

```go
conn, _ := clickhouse.Open(&clickhouse.Options{
    Addr: []string{"host:9000"},
    Auth: clickhouse.Auth{Database: "default", Username: "default"},
})
ctx := context.Background()
batch, _ := conn.PrepareBatch(ctx, "INSERT INTO t (a, b)")
for _, r := range rows {
    batch.Append(r.a, r.b)
}
batch.Send()
```

### 4.4 其他

- **Node.js**:`@clickhouse/client`(官方)。
- **Rust**:`clickhouse-rs`(社区)。
- **.NET / C#**:`ClickHouse.Client` (NuGet)。

---

## 5. 生态集成的常见架构图

```
   ┌──────────────────────────────────────────────────────────────────┐
   │                            Ingestion                              │
   │                                                                   │
   │   App ──► Kafka ──► CH Kafka 引擎 ──► MV ──► MergeTree (本地)   │
   │                                                                   │
   │   App ──► Vector ──► CH HTTP ──► Buffer ──► MergeTree (本地)    │
   │                                                                   │
   │   Spark/Flink ──► CH JDBC/Native ──► MergeTree (本地)           │
   │                                                                   │
   │   S3 / HDFS ──[on demand]── ClickHouse s3()/hdfs() 表函数         │
   └──────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
   ┌──────────────────────────────────────────────────────────────────┐
   │                     ClickHouse Cluster                            │
   │   Shards × Replicas + ZK/Keeper                                  │
   │   AggregatingMT / Projection / Dictionary 加速预计算             │
   └──────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
   ┌──────────────────────────────────────────────────────────────────┐
   │                       Serving / BI                                │
   │                                                                   │
   │   Grafana(监控) | Superset/Metabase(自助 BI) | Tableau(企业)  │
   │   App via HTTP/JDBC/Native | Python/Go/Java SDK                  │
   └──────────────────────────────────────────────────────────────────┘
```

---

## 6. 本章勘误区

> 原文这一章是大纲,具体内容很少,主要补充而非纠错。

### ⚠ 原文表述
> "8.2. 数据湖集成 - 使用 `s3`, `gcs`, `hdfs` 表函数直接查询云存储或 HDFS 上的 Parquet/CSV 文件。"

### ✓ 补充
"数据湖"在 2024-2025 年的语境往往是指 **Iceberg / Delta Lake / Hudi** 这类带元数据层的存储格式,不是单纯的"文件 + 对象存储"。ClickHouse 在新版中已经提供:
- `iceberg()` / `Iceberg` 表函数与表引擎(查询 Apache Iceberg 表)
- `deltaLake()` 表函数(查询 Delta Lake)
- `hudi()` 表函数(查询 Apache Hudi,只读)
直接查文件只是入门级,**用 lake 元数据层做联邦查询**是 2024+ 的趋势,教学应跟上。

---

### ⚠ 原文表述
> "Python (`clickhouse-connect`, `clickhouse-driver`)"

### ✓ 补充
两者定位不同:
- `clickhouse-connect`:**官方**,走 HTTP/HTTPS,支持 Pandas/Numpy/PyArrow,云环境(Cloud)和 BI 工具背后大多用它。
- `clickhouse-driver`:**社区**,走原生 TCP,长连接、性能更高,但与 CH Cloud 的一些功能(如 HTTP 限流策略)不兼容。
教学时按场景推荐,不要并列叙述。
