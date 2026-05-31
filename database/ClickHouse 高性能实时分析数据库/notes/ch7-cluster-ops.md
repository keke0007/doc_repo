# 第7章 集群管理与运维

> 本章梳理自原文「第7章 集群管理与运维」。覆盖:集群组件、分片/副本、ZooKeeper(Keeper)协同机制、`ReplicatedMergeTree`+`Distributed` 实战、监控(Prometheus+Grafana)、`clickhouse-backup` 备份恢复、滚动升级。

---

## 1. 现代集群组件全景

```
                    ┌───────────────────────────────────────┐
                    │            Operator / Helm             │
                    │  (K8s 中维护"期望状态"的机器人)        │
                    └─────────────────┬─────────────────────┘
                                      │ apply chi.yaml
        ┌─────────────────────────────┼──────────────────────────────┐
        │                             │                              │
        ▼                             ▼                              ▼
 ┌──────────────┐            ┌──────────────┐              ┌───────────────────┐
 │ Shard 1      │            │ Shard 2      │              │ ClickHouse Keeper │
 │  Replica A   │            │  Replica A   │   元数据/    │ (取代 ZooKeeper)  │
 │  Replica B   │ ◄────►     │  Replica B   │   协调  ───► │   3/5 节点集群     │
 │  Replica C   │            │  Replica C   │              │                   │
 └──────┬───────┘            └──────┬───────┘              └───────────────────┘
        │                           │
        └────────────┬──────────────┘
                     ▼
         ┌────────────────────────────┐
         │ Distributed 表 (查询入口)   │
         │  自身不存数据,只做路由     │
         └────────────────────────────┘
                     ▲
                     │  SQL
                     │
                  Client / BI / 应用
```

核心组件:
1. **ClickHouse Nodes**:真正存数据 + 计算。
2. **ClickHouse Keeper**:官方内置协调服务,**完全替代 ZooKeeper**(v22+ 推荐),兼容 ZK 协议。
3. **ClickHouse Operator**(K8s 场景):用 YAML 描述期望集群,自动创建并维护。
4. **Distributed 表引擎**:与用户交互的统一入口。

---

## 2. 集群设计的黄金法则:Shard + Replica

| 维度 | 分片 (Shard) | 副本 (Replica) |
|------|--------------|----------------|
| 目的 | 扩展性(单机存不下/算不动) | 高可用(节点宕机不丢服务) |
| 数据布局 | **水平切分**,各分片各持一份 | 同一分片 **完全镜像** |
| 写入 | 按 `sharding_key` 路由 | 写入任一副本即可,异步同步 |
| 读取 | 查询广播到各分片,本地并行 | 自动选健康副本读 |
| 协调 | 不需要 ZK | 必须依赖 ZK / Keeper |
| 关键引擎 | `Distributed` | `Replicated*MergeTree` |

```
        ┌─── Shard 1 ─────────────┐    ┌─── Shard 2 ─────────────┐
        │  Replica A  Replica B   │    │  Replica A  Replica B   │
        │   (镜像)   (镜像)       │    │   (镜像)   (镜像)       │
        └───────────┬─────────────┘    └───────────┬─────────────┘
                    │  数据 = 全量的 1/N            │
                    │                              │
                    └──────────────┬───────────────┘
                                   ▼
                            Distributed 表
                            (集群虚拟表)
```

分片键(`sharding_key`)选择:
- 业务高基数列(`user_id`、`device_id`、`session_id`)→ 数据均匀。
- 若希望按维度路由查询(只查特定 shard),用 `cityHash64(key) % N` 等可预测函数。
- 写入随机分布:`rand()`(简单但破坏数据局部性)。

---

## 3. 集群部署:配置文件

### 3.1 监听地址

`config.xml` 中取消注释,允许远程连接:
```xml
<listen_host>0.0.0.0</listen_host>
```

### 3.2 ZooKeeper 配置

每台节点都需要,位置可放主 `config.xml` 或独立 `config.d/zookeeper.xml`:

```xml
<clickhouse>
  <zookeeper>
    <node index="1"><host>zk01</host><port>2181</port></node>
    <node index="2"><host>zk02</host><port>2181</port></node>
    <node index="3"><host>zk03</host><port>2181</port></node>
  </zookeeper>
</clickhouse>
```

并把这份配置同步到所有 CH 节点,然后逐个重启 `clickhouse-server`。

> 用 ClickHouse Keeper 时,配置标签是 `<keeper_server>`,客户端这一边仍用 `<zookeeper>` 指向 Keeper 节点。

### 3.3 集群拓扑

`remote_servers` 描述了"集群里有哪些分片、每个分片有哪些副本":

```xml
<clickhouse>
  <remote_servers>
    <my_cluster>
      <!-- 分片 1 -->
      <shard>
        <internal_replication>true</internal_replication>
        <replica><host>ch01</host><port>9000</port></replica>
        <replica><host>ch02</host><port>9000</port></replica>
      </shard>
      <!-- 分片 2 -->
      <shard>
        <internal_replication>true</internal_replication>
        <replica><host>ch03</host><port>9000</port></replica>
        <replica><host>ch04</host><port>9000</port></replica>
      </shard>
    </my_cluster>
  </remote_servers>
</clickhouse>
```

关键点:
- `<internal_replication>true</internal_replication>` 表示**写入由 `ReplicatedMergeTree` 自己处理副本同步**,Distributed 表只写主副本;`false` 则 Distributed 会"写所有副本"(传统 Replicated 表上几乎不用)。
- 各分片的 `<host><port>` 必须能互相直接访问(包括 `interserver_http_port` 9009)。

### 3.4 宏(Macros)定义

每台节点的 `config.d/macros.xml` 唯一:
```xml
<clickhouse>
  <macros>
    <shard>01</shard>          <!-- 第几个分片 -->
    <replica>ch01</replica>    <!-- 副本标识,通常用主机名 -->
  </macros>
</clickhouse>
```

`{shard}` / `{replica}` 会被 `ReplicatedMergeTree` 的路径模板替换。

---

## 4. `ReplicatedMergeTree` 实战

### 4.1 ZK 路径与副本名

```sql
ENGINE = ReplicatedMergeTree(
    '/clickhouse/tables/{shard}/{database}/{table}',  -- 同分片所有副本必须一致
    '{replica}'                                       -- 同分片不同副本必须不同
)
```

- 路径模板里通常包含 `{shard}`、`{database}`、`{table}`(或 `{uuid}` 风格)。
- 同分片各副本指向**同一**ZK 路径,通过 `{replica}` 区分。

### 4.2 一次性建表(`ON CLUSTER`)

```sql
CREATE TABLE default.logs_local ON CLUSTER my_cluster (
    event_date Date, id UInt64, msg String
)
ENGINE = ReplicatedMergeTree(
    '/clickhouse/tables/{shard}/default/logs_local',
    '{replica}'
)
PARTITION BY toYYYYMM(event_date)
ORDER BY id;
```

`ON CLUSTER my_cluster` 让该 DDL 在集群所有节点上各自执行(通过 ZK 任务队列协调,见 5.2)。

---

## 5. ZooKeeper / Keeper 在两大场景里的工作流

### 5.1 写入与副本同步(写时复制)

```
   Client INSERT into logs_local (在 Replica A 上)
        │
        ▼
   ┌────────────────────────┐
   │  Replica A             │
   │  1. 本地写 Data Part   │
   │  2. 在 ZK 写一条 log:  │
   │     "GET_PART <name>   │
   │      from replica A"   │
   └─────────┬──────────────┘
             │
             ▼
   ┌────────────────────────────────────────────┐
   │             ZooKeeper / Keeper             │
   │  /clickhouse/tables/{shard}/{db}/{tbl}/log │
   │   ├── log-00000  (操作日志条目)            │
   │   ├── log-00001                            │
   │   └── ...                                  │
   │  /replicas/<replica>/queue/                │
   └────────────┬───────────────────────────────┘
                │ Replica B 监听到新日志条目
                ▼
   ┌────────────────────────┐
   │  Replica B             │
   │  从 Replica A 的       │
   │  interserver_http(9009)│
   │  接口拉取 Part 数据    │
   │  → 本地落盘 → 完成同步 │
   └────────────────────────┘
```

要点:
- ZK 只传**操作日志**(几十字节);**数据走 CH 节点间 HTTP 9009**。
- 副本是**多主**:任一副本写入都会发起同步,客户端不需要"找主副本"。

### 5.2 `ON CLUSTER` 分布式 DDL

```
   Client: CREATE TABLE ... ON CLUSTER my_cluster
        │
        ▼
   ┌─────────────────────┐
   │ 任意发起节点         │
   │  把 DDL 任务写入 ZK  │
   └───────────┬─────────┘
               │
               ▼
   ┌──────────────────────────────────────────┐
   │  ZK:  /clickhouse/task_queue/ddl/        │
   │  └─ query-NNNN { SQL, cluster, initiator}│
   └──┬──────────────┬──────────────┬─────────┘
      │              │              │
   集群中每个节点同时 watch 自己应执行的任务
      │              │              │
      ▼              ▼              ▼
   Node1          Node2          Node3
   本地执行       本地执行        本地执行
   CREATE TABLE   CREATE TABLE   CREATE TABLE
      │              │              │
      └──── 把执行结果回写到 ZK 节点 ──┘
                     │
                     ▼
              发起方读取每个节点结果
              或等待 distributed_ddl_task_timeout
```

支持的语句:`CREATE/ALTER/DROP/RENAME/TRUNCATE` 都可加 `ON CLUSTER`。

---

## 6. `Distributed` 表:查询分发与读时聚合

### 6.1 建表

```sql
CREATE TABLE default.logs_distributed ON CLUSTER my_cluster
AS default.logs_local
ENGINE = Distributed(
    my_cluster,    -- 集群名
    default,       -- 远端库
    logs_local,    -- 远端本地表
    rand()         -- 分片键(写入时用)
);
```

### 6.2 查询路径

```
   Client → 任意节点(协调节点)
        │
        │  SELECT ... FROM logs_distributed WHERE ... GROUP BY ...
        ▼
   ┌─────────────────────────────────────────────────────────────┐
   │ 协调节点:                                                    │
   │  1. 把查询改写为子查询 SELECT ... FROM logs_local            │
   │  2. 并行发到每个分片的一个健康副本(优先本地副本)            │
   │                                                              │
   │       ┌── Shard1 Replica ──┐  ┌── Shard2 Replica ──┐         │
   │       │ 本地执行          │  │ 本地执行          │  ...     │
   │       │ 聚合中间结果      │  │ 聚合中间结果      │          │
   │       └──────┬────────────┘  └──────┬────────────┘         │
   │              │                       │                       │
   │              └───────────┬───────────┘                       │
   │                          ▼                                   │
   │ 3. 协调节点 Merge 各分片的部分结果(`-State`/二次聚合)        │
   │ 4. 返回最终行                                                │
   └─────────────────────────────────────────────────────────────┘
```

### 6.3 写入路径

```
   Client INSERT INTO logs_distributed
        │
        ▼
   协调节点根据 sharding_key 计算目标分片
        │
        ▼
   推送到目标分片的"一个副本"  ←  internal_replication=true
        │                          (其他副本由 ReplicatedMergeTree 自己同步)
        ▼
   目标副本本地写 + 同分片其他副本通过 ZK 拉数据
```

### 6.4 故障容错

- 写入:Distributed 跳过宕机副本,选健康副本;副本恢复后由复制队列追平。
- 读取:协调节点跳过宕机副本,选健康副本;若分片所有副本都宕机,该分片数据缺失,可配 `skip_unavailable_shards`。

---

## 7. 监控

### 7.1 关键系统表

| 表 | 内容 |
|----|------|
| `system.metrics` | 瞬时指标(QPS、当前 Merge 数、副本延迟) |
| `system.events` | 累计计数器(`SelectQuery`、`InsertedRows`、`FailedQuery`) |
| `system.merges` | 当前合并任务详情 |
| `system.replicas` | 复制状态、Lag、队列大小 |
| `system.replication_queue` | 副本同步队列详情 |
| `system.parts` | Part 行数/字节,排查"小 Part 过多" |
| `system.query_log` | 查询历史 |
| `system.errors` | 错误码累计 |

副本延迟告警必看:
```sql
SELECT metric, value
FROM system.metrics
WHERE metric IN ('Query', 'Merge', 'ReplicasMaxAbsoluteDelay');
```

### 7.2 Prometheus + Grafana 链路

```
   ┌─────────────────────┐
   │  ClickHouse Node 1  │   ┌──────────────────────┐
   │  + clickhouse-      │ ──┤ /metrics (HTTP)      │
   │    exporter         │   └──────────┬───────────┘
   └─────────────────────┘              │
   ┌─────────────────────┐              ▼
   │  ClickHouse Node 2  │   ┌──────────────────────┐    PromQL    ┌─────────┐
   │  + exporter         │ ──┤   Prometheus        │ ◄─────────── │ Grafana │
   └─────────────────────┘   │   (scrape & store)  │              └─────────┘
   ┌─────────────────────┐   └──────────┬───────────┘                  ▲
   │  ClickHouse Node N  │              │ rules                        │
   │  + exporter         │ ──┐          ▼                              │
   └─────────────────────┘   │  ┌──────────────────────┐                │
                              │  │ Alertmanager        │ → 钉钉/Slack   │
                              │  └──────────────────────┘                │
                              │                                          │
                              └──────────────────────────────────────────┘
                                       数据可视化与告警
```

> v22+ 起,ClickHouse 也自带 Prometheus 端点(`/metrics`,启用 `prometheus.endpoint`),理论上可不再需要单独的 exporter。

### 7.3 必看告警指标

| 指标 | 阈值 / 含义 |
|------|------------|
| `ReplicasMaxAbsoluteDelay`(秒) | > 300 即副本同步严重滞后 |
| `clickhouse_up`(exporter 探活) | = 0 → 节点宕 |
| `rate(FailedQuery[5m])` | 突增 → 应用层 bug 或恶意查询 |
| `histogram_quantile(0.95, query_duration_ms)` | 同比上涨明显 → 整体劣化 |
| `Merge`(当前合并任务数) | 持续高位 + Part 数飙升 → IO 即将瓶颈 |
| 磁盘剩余 / 内存 / CPU | 通用主机指标 |

---

## 8. 备份与恢复(`clickhouse-backup`)

### 8.1 为什么不只用 `FREEZE`

`ALTER TABLE ... FREEZE PARTITION` 是基础(写时复制硬链 + 拷贝),但只是第一步;还要自己管理目录、上传远端、增量计算、批量恢复——繁琐易错。`clickhouse-backup` 把这些自动化了。

### 8.2 工作原理

```
   clickhouse-backup create
        │
        ▼
   ┌───────────────────────────────────────┐
   │ 1. 对每张表执行 ALTER TABLE ... FREEZE │
   │    → /var/lib/clickhouse/shadow/      │
   │ 2. 拷贝 shadow 目录到 backup 目录     │
   │    /var/lib/clickhouse/backup/<name>/ │
   │ 3. 拷贝表的 metadata (.sql)           │
   │ 4. (可选) 上传到 S3/GCS/FTP/SFTP     │
   │ 5. 记录到本地索引                     │
   └───────────────────────────────────────┘

   增量备份(--diff-from <prev>):
   只复制相对 <prev> 多出的 Part(基于 Part 名硬链,极轻)
```

### 8.3 关键命令

```bash
clickhouse-backup tables                       # 列出可备份的表
clickhouse-backup create <name>                # 全量备份
clickhouse-backup create --diff-from <prev> <name>   # 增量备份
clickhouse-backup create -t db.t1,db.t2 <name>       # 仅指定表
clickhouse-backup create --upload <name>             # 备份并上传到远端

clickhouse-backup list  local                  # 本地备份
clickhouse-backup list  remote                 # 远端备份

clickhouse-backup download <name>              # 远端 → 本地
clickhouse-backup restore  <name>              # 恢复全表(结构+数据)
clickhouse-backup restore --schema -t db.t <name>    # 只恢复结构
clickhouse-backup restore --data   -t db.t <name>    # 只恢复数据

clickhouse-backup delete   local  <name>       # 删除本地备份
clickhouse-backup delete   remote <name>       # 删除远端备份
```

### 8.4 全量 + 增量 + 恢复流程

```
   周日: full_backup_20231022
            │
            ▼ INSERT 周一新数据
   周一: --diff-from full_backup_20231022  →  inc_backup_20231023
            │
            ▼ INSERT 周二新数据
   周二: --diff-from inc_backup_20231023   →  inc_backup_20231024

   故障 → 表丢失
            │
            ▼ 恢复必须按顺序:
   1) restore full_backup_20231022     -- 表结构 + 周日数据
   2) restore inc_backup_20231023      -- + 周一增量
   3) restore inc_backup_20231024      -- + 周二增量
```

⚠️ 关键约束:
- 任何增量备份都必须以全量备份为基线。
- 恢复**必须按时间顺序**逐个执行,不能跳跃。
- 命名规范化(`full_*` / `inc_*` + 日期),便于自动化。

### 8.5 自动化(cron + 脚本)

`/usr/local/sbin/backup_clickhouse.sh`:
```bash
#!/bin/bash
LOG_DATE=$(date +%Y-%m-%d)
LOG_FILE="/var/log/clickhouse-backup/backup-${LOG_DATE}.log"
mkdir -p /var/log/clickhouse-backup

/usr/local/bin/clickhouse-backup create -u >> "${LOG_FILE}" 2>&1

# 7 天前的日志清理
find /var/log/clickhouse-backup -type f -mtime +7 -delete
```

`crontab -e`:
```
30 2 * * *  /usr/local/sbin/backup_clickhouse.sh
```

### 8.6 SQL 内置 `BACKUP` / `RESTORE`(v22+)

ClickHouse 自身在 v22 之后内置了 SQL 级备份接口,可以不依赖 clickhouse-backup:
```sql
BACKUP DATABASE analytics TO Disk('backups', 'analytics_2024-05-30.zip');
RESTORE DATABASE analytics FROM Disk('backups', 'analytics_2024-05-30.zip');
```
支持 S3 目标、增量、`ALL` 全部数据库等。生产可二选一。

---

## 9. 平滑升级(Rolling Upgrade)

### 9.1 黄金原则

1. **绝不跨大版本跳跃**:21.x → 22.x → 23.x → 24.x,不要直接 21 → 24。
2. **必读 Release Notes**:重点看"Backward Incompatible Changes"。
3. **先测试环境演练**,且数据量、表结构应与生产一致。
4. **升级前全量备份**。
5. **先升级副本,再升级写入热点节点**,降低写入影响。
6. **每升级一个节点观察 15-30 分钟**,日志/监控无异常再继续。

### 9.2 升级单个节点(RHEL/CentOS 示例)

```bash
# 1. 停服务
sudo systemctl stop clickhouse-server

# 2. 安装目标版本
sudo yum install -y \
    clickhouse-common-static-<NEW> \
    clickhouse-client-<NEW> \
    clickhouse-server-<NEW>

# 3. 启动
sudo systemctl start clickhouse-server
sudo systemctl status clickhouse-server

# 4. 观察日志、查询、副本延迟
tail -F /var/log/clickhouse-server/clickhouse-server.log
clickhouse-client -q "SELECT version()"
clickhouse-client -q "SELECT * FROM system.replicas WHERE absolute_delay > 60"
```

### 9.3 回滚

- 准备好旧版本 RPM 包。
- 出现严重问题先 `systemctl stop`,降级安装回旧版,重启。
- 数据文件格式向前兼容(老版本能读新版本的);但偶尔有不兼容字段或新引擎需要注意。

---

## 10. 本章勘误区

### ✗ 原文
> "**ClickHouse Keeper**: 官方出品、内置的协调服务,完全替代 ZooKeeper。"

### ✓ 补充
正确,但要补两点:
1. Keeper 在 v21.6 进入实验、v22.3 标记 production-ready,**v22+ 起官方推荐用 Keeper 替代 ZooKeeper**。
2. Keeper 完整兼容 ZK 协议,所以从 ZK 切换到 Keeper 时,**ClickHouse 节点的 `<zookeeper>` 配置不需要改**,只把后端从 ZK 集群换成 Keeper 集群即可。

---

### ✗ 原文(ZooKeeper 配置示例)
```xml
<zookeeper>
 <node index="1"> <host>doitedu01</host> <port>2181</port> </node>
 <node index="2"> <host>doitedu02</host> <port>2181</port> </node>
 <node index="3"> <host>doitedu02</host> <port>2181</port> </node>   ← 重复 doitedu02
</zookeeper>
```
第二条和第三条 `host` 都是 `doitedu02`,应为 `doitedu03`。

---

### ✗ 原文(`cluster3` XML 注释)
> "集群三 多个分片 保留副本 注意一个主机只使用一次"

实际配置里 `cluster3` 中 4 台主机 `doit01~doit04` 每台只用一次,没有问题。但**前面的 `cluster2` 写法**:
```xml
<shard>
  <replica><host>linux01</host></replica>
  <replica><host>linux02</host></replica>
  <replica><host>linux03</host></replica>
</shard>
```
被注释成"一个分片 一个副本"——错了,这里是**一个分片,三个副本**(同分片下有 3 个 replica 标签)。中文注释口径要纠正。

---

### ✗ 原文(`Distributed` 与 `internal_replication`)
原文配置示例:
```xml
<shard>
  <internal_replication>true</internal_replication>
  <replica><host>ch_node1</host><port>9000</port></replica>
</shard>
```
紧接的"无副本"示例又没写 `<internal_replication>`。

### ✓ 补充
- 用 `ReplicatedMergeTree` 时:**必须**设 `<internal_replication>true</internal_replication>`,否则 Distributed 会重复写每个副本,造成数据翻倍。
- 用普通 `MergeTree` 时(纯靠 Distributed 来写副本,不推荐):可以设 false。
- 文档要补充对这俩取值的清晰对比。

---

### ✗ 原文(`5 分布式DDL` 的 `alter` 示例最后多了减号)
```
alter table t3 on cluster cluster1 add column age Int8 ;-
- 删除字段 -- 删除分区 
```
末尾的 `;-` 是错别字 / 排版残留。

---

### ✗ 原文
> "**ALTER TABLE ... FREEZE** 命令可以将数据分区的快照'冻结'到磁盘的 `shadow` 目录下"

### ✓ 补充
精确机制:`FREEZE` 在 `/var/lib/clickhouse/shadow/<N>/` 下创建 Part 的**硬链接**(同一物理 inode),所以"冻结"本身**几乎不占额外空间**(零拷贝快照)。如果之后 Merge 把原 Part 删了,硬链接仍指向原文件直到 shadow 也被清理。所以 `FREEZE` 是备份的"零成本"基石。

---

### ⚠ 表述含糊
> "ClickHouse的同步操作日志都通过 ZooKeeper 传输。"

### ✓ 修正
区分两件事:
1. **元数据 / 操作日志条目** → 经过 ZooKeeper。
2. **实际数据 Part 文件** → 走 CH 节点之间的 HTTP(`interserver_http_port`,默认 9009)。
ZK/Keeper 不参与数据流,只参与控制流。
