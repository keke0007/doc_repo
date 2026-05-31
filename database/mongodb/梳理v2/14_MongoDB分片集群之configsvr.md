# 14 · MongoDB 分片集群之 configsvr

## 一、分片集群整体认识

### 1. 为什么要分片
- 副本集解决了**高可用 + 读扩展**，但每个成员都是同一份完整数据，**写吞吐与存储容量受限于单机**。
- 分片集群把数据**水平切分**到多个 shard 上，每个 shard 自己又是一个副本集。
- 适用：单库 TB 级以上、单集合写入压力高、需横向扩展存储容量。

### 2. 三种角色
| 角色 | 进程 | 说明 |
|------|------|------|
| `mongos` | router | 应用入口，无数据，转发请求 |
| `configsvr` | mongod (`clusterRole:configsvr`) | 存储集群元数据（chunk 分布、shard 列表、版本号等） |
| `shardsvr` | mongod (`clusterRole:shardsvr`) | 真正存数据；建议用副本集 |

### 3. 实战拓扑（学习版）
| 角色 | 端口 |
|------|------|
| configsvr | 28017 / 28018 / 28019（一组副本集 `shijiangeconf`） |
| router (mongos) | 27017 / 27018 / 27019 |
| shardsvr - data1 | 29017 / 29018（副本集 `shijiangedata1`） |
| shardsvr - data2 | 29019 / 29020（副本集 `shijiangedata2`） |

> 学习用 4 个 shardsvr 端口（每个 shard 仅 2 实例）演示。**生产环境每个 shard 必须 3+ 实例的副本集**，否则该 shard 一旦宕机即不可写。

## 二、configsvr 搭建

### 1. 配置文件 `/data/mongodb/28017/mongodb.conf`
```yaml
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/28017/mongodb.log
storage:
  dbPath: /data/mongodb/28017/
  journal:
    enabled: true
processManagement:
  fork: true
net:
  port: 28017
  bindIp: 127.0.0.1
replication:
  replSetName: shijiangeconf
sharding:
  clusterRole: configsvr
```
为 28018、28019 复制一份配置，对应改端口和路径。

### 2. 启动 3 个 configsvr
```bash
/usr/local/mongodb/bin/mongod -f /data/mongodb/28017/mongodb.conf
/usr/local/mongodb/bin/mongod -f /data/mongodb/28018/mongodb.conf
/usr/local/mongodb/bin/mongod -f /data/mongodb/28019/mongodb.conf
```
`configsvr` 进程仍然用 `mongod`，不是 `mongos`。

### 3. 初始化 configsvr 副本集
```bash
/usr/local/mongodb/bin/mongo 127.0.0.1:28017
```
```js
config = {
  _id: "shijiangeconf",
  configsvr: true,
  members: [
    { _id: 0, host: "127.0.0.1:28017" },
    { _id: 1, host: "127.0.0.1:28018" },
    { _id: 2, host: "127.0.0.1:28019" }
  ]
}
rs.initiate(config)
rs.status()
```
要点：
- **必须**含 `configsvr: true`，否则 mongos 后续连接会拒绝。
- 自 MongoDB 3.4 起，configsvr 必须以副本集（CSRS）形式部署。

## 三、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | "shardsvr 使用 29017，29018，29019，29020 四个端口" | 文档自己都说"生产肯定要三个端口"，但学习配置直接 2 个 | 写明这是为节省演示资源，生产每 shard ≥ 3 个 mongod 副本集 |
| 2 | "configsvr 角色…依赖于配置角色" | 表述含糊 | configsvr 存的是 chunk → shard 的映射、集合的 shard key、balancer 状态等 |
| 3 | 文档把 shardsvr 写成 `sharedsvr` | 拼写错误 | 应为 `shardsvr` |
| 4 | "router 不存储数据" 但未解释路由如何获取元数据 | 缺细节 | mongos 启动时连接 configsvr，缓存集合的分布信息 (chunk map)；balancer 由 configsvr 节点驱动 |
| 5 | 缺写「configsvr 必须副本集」前置条件 | 关键约束 | 3.4 起 SCCC 已被淘汰，必须 CSRS（Replica Set） |

## 四、分片集群整体架构（ASCII）

```
                          ┌───────────────────────────────┐
                          │  Application / Driver         │
                          │  连接：mongos URI             │
                          └──────────────┬────────────────┘
                                         │
                  ┌──────────────────────┼─────────────────────┐
                  ▼                      ▼                     ▼
           ┌─────────────┐        ┌─────────────┐       ┌─────────────┐
           │ mongos:27017│        │ mongos:27018│       │ mongos:27019│
           │ (router)    │        │ (router)    │       │ (router)    │
           └──────┬──────┘        └──────┬──────┘       └──────┬──────┘
                  └──────────────────────┼─────────────────────┘
                                         │ 启动时拉取 + 实时跟踪元数据
                                         ▼
                       ┌────────────────────────────────────┐
                       │ configsvr 副本集 shijiangeconf     │
                       │  28017 (P) ── 28018 (S) ── 28019(S)│
                       │  存：chunk → shard 映射 / 版本号  │
                       └─────────────┬──────────────────────┘
                                     │
            ┌────────────────────────┴────────────────────────┐
            ▼                                                  ▼
   ┌──────────────────────────┐                   ┌──────────────────────────┐
   │ shardsvr 副本集          │                   │ shardsvr 副本集          │
   │ shijiangedata1           │                   │ shijiangedata2           │
   │  29017 (P) ── 29018 (S)  │                   │  29019 (P) ── 29020 (S)  │
   │  存数据 chunk #0 / #2 ...│                   │  存数据 chunk #1 / #3 ...│
   └──────────────────────────┘                   └──────────────────────────┘

  请求路径：
   App ── mongos ── 查 configsvr 元数据 ── 路由到对应 shard ── 落到 PRIMARY
```
