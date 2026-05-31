# 第2章 极速安装与基础操作

> 本章梳理自原文「第2章 极速安装与基础操作」。覆盖单机安装、客户端工具、SQL 基础(建库/建表/增查)、ClickHouse 特色 `FORMAT` 写入。

---

## 1. 环境搭建

### 1.1 RHEL/CentOS 上用官方 RPM 仓库安装

```bash
# 添加官方源
sudo yum-config-manager --add-repo https://packages.clickhouse.com/rpm/clickhouse.repo

# 安装 server + client
sudo yum install -y clickhouse-server clickhouse-client

# 启用并启动服务
sudo systemctl enable clickhouse-server
sudo systemctl start  clickhouse-server
sudo systemctl status clickhouse-server
```

> 若要安装**指定版本**(例如 22.8 LTS):
> ```bash
> sudo yum install -y clickhouse-server-22.8.7.34 clickhouse-client-22.8.7.34
> ```
> 注意:`clickhouse-common-static` 是 server/client 共同依赖,通常会被自动拉下来。

### 1.2 默认目录与端口

| 项 | 路径 / 值 |
|---|---|
| 配置目录 | `/etc/clickhouse-server/` |
| 数据目录 | `/var/lib/clickhouse/` |
| 日志目录 | `/var/log/clickhouse-server/` |
| HTTP 端口 | `8123`(BI 工具/curl 默认) |
| TCP/Native 端口 | `9000`(原生客户端、原生驱动) |
| 集群节点互联 | `9009`(`interserver_http_port`,副本拉数据用) |
| MySQL 协议 | `9004` |
| PostgreSQL 协议 | `9005` |

> ⚠️ 想让远程客户端能连上,需要在 `config.xml` 里取消注释 `<listen_host>0.0.0.0</listen_host>`(默认只监听 127.0.0.1)。

---

## 2. 客户端

### 2.1 `clickhouse-client` 命令行

```bash
clickhouse-client -m      # -m 允许多行 SQL,以 ; 结束
```

进入交互后:
```sql
SHOW DATABASES;
exit                       -- 或 Ctrl+D
```

常用参数:
- `--host` / `--port` / `-u` 用户 / `--password`
- `-q "SQL"` 直接执行一句
- `--query "SQL" --format CSV` 指定输出格式
- `--multiquery` 一次执行多条 SQL

### 2.2 HTTP / HTTPS 接口

```bash
# 简单查询
curl 'http://localhost:8123/' --data-binary 'SELECT 1'

# 指定输出格式
curl 'http://localhost:8123' \
     --data-binary 'SELECT number FROM system.numbers LIMIT 5 FORMAT CSV'
```

带身份验证:
```bash
curl 'http://localhost:8123/?user=default&password=xxxx' --data-binary '...'
```

### 2.3 图形化工具(DBeaver / DataGrip)

1. 新建数据库连接 → 选择 ClickHouse。
2. 填写主机、端口(图形工具默认走 **HTTP 8123**)、用户、密码。
3. 测试连接 → 保存。

> DBeaver 走 HTTP/JDBC,适合日常 DDL/查询;高吞吐写入仍建议走 TCP/Native。

---

## 3. SQL 基础

### 3.1 建库

```sql
CREATE DATABASE IF NOT EXISTS learning;
USE learning;
```

不指定引擎时,新版默认使用 **Atomic** 引擎(支持原子 RENAME/EXCHANGE,见第 3 章)。

### 3.2 建表

ClickHouse 的 `CREATE TABLE` 与标准 SQL 最大区别:**`ENGINE` 是必填项**。

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [NULL|NOT NULL] [DEFAULT|MATERIALIZED|EPHEMERAL|ALIAS expr1]
          [COMMENT 'col comment'] [CODEC(codec1)] [TTL expr1],
    name2 [type2] ...
) ENGINE = engine
[ORDER BY expr]
[PARTITION BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr ...]
[SETTINGS name = value, ...]
[COMMENT 'table comment'];
```

入门示例:
```sql
CREATE TABLE t_web_hits
(
    timestamp    DateTime,            -- 访问时间
    user_id      UInt64,              -- 用户 ID
    url          String,              -- URL
    os           String,              -- 操作系统
    browser      String,              -- 浏览器
    duration_ms  UInt32               -- 停留时长(毫秒)
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)      -- 按月分区
ORDER BY (timestamp, user_id);        -- 物理排序键(也是稀疏索引)
```

关键点:

- 列类型用 ClickHouse 类型(`DateTime`, `UInt64`, `String`, `LowCardinality(String)`, `Nullable(T)`,见第 4 章)。
- `PARTITION BY` 把数据按分区目录切分,便于按分区裁剪/删除。
- `ORDER BY` 决定**数据在磁盘上的物理顺序**,是查询性能的灵魂。

### 3.3 插入数据

`VALUES`:
```sql
INSERT INTO t_web_hits VALUES
  (now(), 1001, '/page/a', 'iOS',     'Safari', 1500),
  (now(), 1002, '/page/b', 'Windows', 'Chrome', 3200);
```

`FORMAT`(ClickHouse 特色,大批量推荐):
```sql
INSERT INTO t_web_hits FORMAT JSONEachRow
{"timestamp":"2023-10-27 10:00:00","user_id":1003,"url":"/home","os":"Android","browser":"Chrome","duration_ms":800}
{"timestamp":"2023-10-27 10:01:00","user_id":1002,"url":"/profile","os":"Windows","browser":"Chrome","duration_ms":12000}
;
```

从 CSV 文件批量导入:
```bash
clickhouse-client --query "INSERT INTO t_web_hits FORMAT CSV" < hits.csv
```

> 写入原则:**一次写一大批,而不是一次一行**,详见第 4.4 节。

### 3.4 查询

```sql
SELECT * FROM t_web_hits;
SELECT * FROM t_web_hits WHERE user_id = 1001;

SELECT browser, count() AS view_count
FROM t_web_hits
GROUP BY browser
ORDER BY view_count DESC;

SELECT user_id, avg(duration_ms) AS avg_dur
FROM t_web_hits
GROUP BY user_id;
```

### 3.5 元数据查看

```sql
SHOW DATABASES;
SHOW TABLES FROM learning;
DESCRIBE TABLE learning.t_web_hits;
SHOW CREATE TABLE learning.t_web_hits;
```

---

## 4. 一次 SQL 在客户端/服务器间的整体流转(直观图)

```
   ┌──────────────┐        SQL 文本         ┌─────────────────────────────────┐
   │ Client       │ ──────────────────────► │ ClickHouse Server               │
   │ (CLI/JDBC/   │                         │                                 │
   │  curl/...)   │                         │  ┌────────────┐                 │
   │              │                         │  │ Parser     │  解析为 AST    │
   │              │                         │  └─────┬──────┘                 │
   │              │                         │        ▼                        │
   │              │                         │  ┌────────────┐                 │
   │              │                         │  │ Analyzer   │  权限/类型/优化│
   │              │                         │  └─────┬──────┘                 │
   │              │                         │        ▼                        │
   │              │                         │  ┌────────────┐                 │
   │              │                         │  │ Planner    │  生成执行计划  │
   │              │                         │  └─────┬──────┘                 │
   │              │                         │        ▼                        │
   │              │                         │  ┌──────────────┐               │
   │              │                         │  │ Executor     │              │
   │              │                         │  │  ▸ MergeTree │  读分区/Part │
   │              │                         │  │    Reader    │  跳数索引    │
   │              │                         │  │  ▸ Vector    │  向量化算子  │
   │              │                         │  │    Pipeline  │              │
   │              │                         │  └─────┬────────┘               │
   │              │                         │        │ 结果块                 │
   │              │ ◄───────────────────────│  ┌─────▼──────┐                 │
   │              │  按 FORMAT 输出         │  │ Formatter  │                 │
   └──────────────┘  (Native/CSV/JSON/...)  │  └────────────┘                 │
                                            └─────────────────────────────────┘
```

---

## 5. 本章勘误区

### ✗ 原文
> "**端口:** 8123 (注意,大部分图形化工具默认走 HTTP 端口)"

### ✓ 正解 / 补充
表述正确但不完整。补一句:**clickhouse-client 默认走 TCP 9000**,所以同一台机器,DBeaver 用 8123、命令行用 9000——别看到端口不同就以为有两份数据,后端是同一个 Server。

---

### ✗ 原文(2.1 节)
```bash
sudo yum install -y clickhouse-server clickhouse-client
sudo yum install clickhouse-server-22.8.7.34
```

### ✓ 正解 / 补充
先装最新版,再降级到 22.8.7.34 会失败/卡住——`yum install` 不会自动降级,需要 `yum downgrade`,或者一开始就指定版本一次性装好:

```bash
sudo yum install -y clickhouse-server-22.8.7.34 \
                    clickhouse-client-22.8.7.34 \
                    clickhouse-common-static-22.8.7.34
```

且 22.8 已不在维护期(2025 年的 LTS 是 24.x 系列),教学示例可以用,生产环境应选当前 LTS。

---

### ⚠ 表述不严谨
> "`ORDER BY` 才是真正的'指挥官':它规定了每个数据片段内部的数据物理排序规则。"

### ✓ 补充
更精确:`ORDER BY` 既是 **物理排序键** 也是 **稀疏主键索引**(除非显式指定 `PRIMARY KEY`)。区分这点对后面理解"为何 `WHERE` 走主键就快"很关键。
