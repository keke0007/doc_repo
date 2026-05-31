# 15 · MongoDB 分片集群之 router

## 一、知识点梳理

### 1. router (mongos) 是什么
- mongos 是分片集群的入口进程，**不存数据**，仅做路由 + 聚合。
- 启动时连接 configsvr 副本集，拉取 chunk 路由表（chunk → shard 映射）。
- 客户端的所有请求都打到 mongos，由它决定该走哪个 shard。

### 2. router 配置 `/data/mongodb/27017/mongodb.conf`
```yaml
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/27017/mongodb.log
processManagement:
  fork: true
net:
  port: 27017
  bindIp: 127.0.0.1
sharding:
  configDB: shijiangeconf/127.0.0.1:28017,127.0.0.1:28018,127.0.0.1:28019
```
要点：
- mongos 不需要 `storage` / `replication` 段（无本地数据）。
- `configDB` 格式必须是 **`<configReplSetName>/<host:port,host:port,...>`**。

### 3. 启动 mongos
```bash
/usr/local/mongodb/bin/mongos -f /data/mongodb/27017/mongodb.conf
/usr/local/mongodb/bin/mongos -f /data/mongodb/27018/mongodb.conf
```
- 进程名是 `mongos`（与 mongod 区分）。
- 同一份 configDB 可对应**多个 mongos**，做负载均衡或灰度。

### 4. 验证
- 此时还没添加 shard，不能验证业务读写；需要 16 章 shardsvr 起来、并在 mongos 中 `sh.addShard(...)` 之后才能完整验证。
- 连接验证：
  ```bash
  /usr/local/mongodb/bin/mongo 127.0.0.1:27017
  > sh.status()
  ```

### 5. mongos 故障转移
- 应用层应连接**多个 mongos**：`mongodb://m1:27017,m2:27018,m3:27019/db`
- 任意一个 mongos 挂掉，应用自动切到下一个；configDB 副本集发生主备切换 mongos 也会自动跟随。

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | "需要等到数据角色搭建完才能够进行验证" | 表述对，但缺细节 | 即使 shard 还没加，`sh.status()` 也能跑，看到一个空集群拓扑 |
| 2 | 配置中没有 `bindIp` 安全说明 | 演示用 127.0.0.1，生产应明示 | 生产监听内网 IP，并启用鉴权 (`security.keyFile`) |
| 3 | 未说明 mongos 是无状态进程 | 对扩缩容有重要影响 | 可以随意增删 mongos 实例，无需特殊处理；只要 configDB 不变 |
| 4 | "配置多个 router，任何一个都能正常的获取数据" | 表述粗 | 准确：**多 mongos 必须使用同一 configDB**，否则会路由到错误集群 |
| 5 | mongos 启动命令使用了 `mongos` | 没问题，但要点出与 `mongod` 二进制不同 | 单独二进制：`/usr/local/mongodb/bin/mongos` |

## 三、router 与 configsvr 协作（ASCII）

```
   App / Driver
        │ mongodb://mongos1,mongos2,.../db
        ▼
 ┌───────────────┐         ┌───────────────┐
 │  mongos #1    │         │  mongos #2    │
 │  (无状态)     │         │               │
 └──────┬────────┘         └──────┬────────┘
        │ 启动 / 周期心跳         │
        └────────┬────────────────┘
                 ▼
       ┌──────────────────────────────┐
       │  configsvr 副本集            │
       │  shijiangeconf               │
       │  28017(P) 28018(S) 28019(S)  │
       │   ─ shards 列表              │
       │   ─ chunk → shard 映射       │
       │   ─ balancer 状态            │
       └────────┬─────────────────────┘
                │ 元数据驱动路由
                ▼
       ┌──────────────────────────────┐
       │ mongos 内存 chunk routing    │
       │ table（带 epoch / 版本号）   │
       └────────┬─────────────────────┘
                │ 业务请求到来
                ▼
   ┌────────────┴───────────────────────────┐
   ▼                                         ▼
 shardsvr shijiangedata1              shardsvr shijiangedata2
 (29017/29018)                        (29019/29020)
```
