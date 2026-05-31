# 07 · MongoDB 副本集的搭建

## 一、知识点梳理

### 1. 为什么要副本集
- 单机：宕机即不可用，数据有丢失风险。
- 副本集：多个 mongod 维持同一份数据，自动主备切换，能提供数据冗余、读写分离的基础。

### 2. 实战拓扑
- 文档建议生产至少 **3 节点**，本例用三个实例（可分布在 2 ~ 3 台机器上）：
  - `192.168.237.128:27017`
  - `192.168.237.129:27018`
  - `192.168.237.128:27019`
- 三节点是最小高可用部署：宕一台仍能完成多数派选主。

### 3. 节点配置（每个端口一份）
```yaml
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/27017/mongodb.log
storage:
  dbPath: /data/mongodb/27017/
  journal:
    enabled: true
processManagement:
  fork: true
net:
  port: 27017
  bindIp: 0.0.0.0
replication:
  replSetName: shijiange     # 同一副本集名称必须一致
```

### 4. 启动 + 初始化
```bash
/usr/local/mongodb/bin/mongod -f /data/mongodb-cluster/27017/mongodb.conf
/usr/local/mongodb/bin/mongod -f /data/mongodb-cluster/27018/mongodb.conf
/usr/local/mongodb/bin/mongod -f /data/mongodb-cluster/27019/mongodb.conf
```

任选一台进入 `mongo` shell：
```js
config = {
  _id: "shijiange",
  members: [
    { _id: 0, host: "192.168.237.128:27017" },
    { _id: 1, host: "192.168.237.129:27018" },
    { _id: 2, host: "192.168.237.128:27019" }
  ]
}
use admin
rs.initiate(config)
rs.status()
```

### 5. 数据同步验证
```js
// 在 PRIMARY 上：
use shijiange
db.myuser.insert({ userid: 1 })

// 在 SECONDARY 上：
rs.secondaryOk()           // 5.0+ 推荐用法
db.myuser.find()
```
SECONDARY 默认禁止读，必须显式声明可读。从库不允许写入。

### 6. 复制延迟监控
```js
rs.printSecondaryReplicationInfo()    // 5.0+
// 或老版本：rs.printSlaveReplicationInfo()
```

### 7. 注意事项
- 各节点优化参数（THP、ulimit、时区）必须保持一致，避免选主异常。
- 节点间需要双向连通，主机名/IP 写入 config 后不可随意变化。
- `rs.initiate()` 后会随机选出一个 PRIMARY，可通过优先权重指定（见 09 章）。

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | "使用两台服务器实战" 但又给出 3 个实例 | 不准确，最少应当是 3 个**投票**节点（仲裁节点也算投票成员） | 副本集成员**为奇数**才能可靠选主，3 节点是最小推荐配置 |
| 2 | `rs.slaveOk()` | 已废弃 | 4.4 起改为 `rs.secondaryOk()` |
| 3 | `rs.printSlaveReplicationInfo()` | 已废弃 | 4.4 起改为 `rs.printSecondaryReplicationInfo()` |
| 4 | "primary 是主，只有 primary 能写入" | 描述正确但不全 | SECONDARY 默认无法读，需 `secondaryOk` 或在驱动层指定 `readPreference` |
| 5 | 配置文件路径写 `/data/mongodb/27017/`，启动时却用 `/data/mongodb-cluster/27017/` | 前后路径不一致，新手容易踩坑 | 二者要保持一致；建议固定一种路径规范，例如 `/data/mongodb-cluster/27017/` |
| 6 | "复本集"出现拼写差异 | 与"副本集"混用 | 统一用「副本集 (Replica Set)」 |

## 三、副本集初始化与同步流程（ASCII）

```
启动三个 mongod (各持空 dbPath，replSetName=shijiange)
        │             │             │
        ▼             ▼             ▼
  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │ :27017   │  │ :27018   │  │ :27019   │
  │ STARTUP  │  │ STARTUP  │  │ STARTUP  │
  └────┬─────┘  └────┬─────┘  └────┬─────┘
       └──────┬──────┴──────┬──────┘
              │  rs.initiate(config) 在任一节点执行
              ▼
       选举 (Raft-like)：版本号/优先级/心跳
              │
        ┌─────┴─────────────────┐
        ▼                       ▼
  ┌───────────┐           ┌─────────────┐
  │ PRIMARY   │ ──写入──▶ │  oplog.rs   │ (capped collection in local 库)
  │ :27017    │           └─────┬───────┘
  └─────┬─────┘                 │ tail
        │ 心跳/复制 (Pull 模型) │
        ▼                       ▼
  ┌───────────┐           ┌───────────┐
  │ SECONDARY │ ◀── pull ─│ SECONDARY │
  │ :27018    │           │ :27019    │
  └───────────┘           └───────────┘

读：默认 PRIMARY
写：仅 PRIMARY；按 writeConcern 等待多数派确认
```
