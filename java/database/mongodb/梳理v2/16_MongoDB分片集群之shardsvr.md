# 16 · MongoDB 分片集群之 shardsvr

## 一、知识点梳理

### 1. 角色定位
- shardsvr 才是真正承载数据的角色。
- 每个 shard **必须使用副本集**，单点 shardsvr 一旦宕机会导致该分片不可写、数据有丢失风险。
- 学习场景：每个 shard 用 2 个实例方便观察；生产至少 3 个。

### 2. shardsvr 配置 `/data/mongodb/29017/mongodb.conf`
```yaml
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/29017/mongodb.log
storage:
  dbPath: /data/mongodb/29017/
  journal:
    enabled: true
processManagement:
  fork: true
net:
  port: 29017
  bindIp: 127.0.0.1
replication:
  replSetName: shijiangedata1   # 同 shard 的成员保持一致
sharding:
  clusterRole: shardsvr
```
另外三份配置文件：29018（也属 `shijiangedata1`）、29019/29020（属 `shijiangedata2`）。

### 3. 启动 4 个实例
```bash
/usr/local/mongodb/bin/mongod -f /data/mongodb/29017/mongodb.conf
/usr/local/mongodb/bin/mongod -f /data/mongodb/29018/mongodb.conf
/usr/local/mongodb/bin/mongod -f /data/mongodb/29019/mongodb.conf
/usr/local/mongodb/bin/mongod -f /data/mongodb/29020/mongodb.conf
```

### 4. 在每个 shard 上各自初始化副本集
登录 29017 执行：
```js
config = {
  _id: "shijiangedata1",
  members: [
    { _id: 0, host: "127.0.0.1:29017" },
    { _id: 1, host: "127.0.0.1:29018" }
  ]
}
rs.initiate(config)
```
登录 29019 执行：
```js
config = {
  _id: "shijiangedata2",
  members: [
    { _id: 0, host: "127.0.0.1:29019" },
    { _id: 1, host: "127.0.0.1:29020" }
  ]
}
rs.initiate(config)
```

### 5. 后续添加到 mongos
（详见 17 章）
```js
sh.addShard("shijiangedata1/127.0.0.1:29017,127.0.0.1:29018")
sh.addShard("shijiangedata2/127.0.0.1:29019,127.0.0.1:29020")
sh.status()
```

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | 文档名 `sharedsvr` | 拼写错 | `shardsvr` |
| 2 | 文档结尾"9 台服务器 / shijiangedata1 占用三台" | 与上文 2 实例的配置自相矛盾 | 学习场景同机部署 4 进程；若按生产规划，每 shard 副本集 3 节点 + 3 configsvr + 2~3 mongos，至少需 8~12 实例（不一定每实例独占主机） |
| 3 | "数据角色一定得使用副本集" | 描述对，但未解释代价 | 没有副本集的 shard 一旦宕机，整个 shard 写入挂起，并阻塞涉及该 shard 的查询；且不能享受 oplog → balancer 迁移 |
| 4 | shardsvr 与 configsvr 端口 28017/29017 默认值 | 提示不全 | shardsvr 默认 27018、configsvr 默认 27019，本演示按学习需要自定义端口 |
| 5 | 缺数据迁移说明 | 关键概念缺失 | shardsvr 之间通过 chunk migration（balancer 协调）搬运数据，相互连接需互相能 reach |

## 三、shardsvr 与整体集群关系（ASCII）

```
        ┌──────────────────────────────────────────┐
        │       configsvr 副本集 shijiangeconf      │
        │     28017(P) ─ 28018(S) ─ 28019(S)        │
        │   元数据：shards、chunks、版本号、balancer │
        └──────────────────┬───────────────────────┘
                           │
                ┌──────────┴──────────┐
                ▼                     ▼
       ┌──────────────────┐   ┌──────────────────┐
       │ mongos:27017     │   │ mongos:27018     │
       └──────┬───────────┘   └──────┬───────────┘
              │ 路由请求             │
   ┌──────────┼──────────────────────┴───────────┐
   ▼          ▼                                   ▼
 shard "shijiangedata1"                    shard "shijiangedata2"
 ┌─────────────────────────┐              ┌─────────────────────────┐
 │ 29017 (PRIMARY)         │              │ 29019 (PRIMARY)         │
 │ 29018 (SECONDARY)       │              │ 29020 (SECONDARY)       │
 │ ── data chunk #0,#2,... │              │ ── data chunk #1,#3,... │
 └─────────────────────────┘              └─────────────────────────┘
              ▲                                   ▲
              │ chunk migration (balancer)        │
              └─────── 直连：29017 ↔ 29019 ───────┘

  写入流程：
   App → mongos → 由 chunk map 找到目标 shard 的 PRIMARY
                  → shard PRIMARY 写 oplog → SECONDARY 同步
                  → ack 返回 mongos → 返回 App
```
