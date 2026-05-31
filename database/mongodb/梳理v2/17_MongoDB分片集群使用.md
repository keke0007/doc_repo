# 17 · MongoDB 分片集群使用

## 一、把 shard 接入 mongos

```bash
/usr/local/mongodb/bin/mongo 127.0.0.1:27017       # 连接到 mongos
```
```js
sh.addShard("shijiangedata1/127.0.0.1:29017,127.0.0.1:29018")
sh.addShard("shijiangedata2/127.0.0.1:29019,127.0.0.1:29020")
sh.status()
```
`addShard` 的字符串格式：`<replSetName>/<host:port,host:port,...>`，写一个或多个种子节点即可，mongos 会自动发现剩余成员。

## 二、不分片 vs 分片

### 1. 默认（未启分片）
```js
use shijiange
for (i = 1; i <= 500; i++) {
  db.myuser.insert({ name: 'mytest' + i, age: i })
}
db.dropDatabase()        // 测试完清理
```
- 不开启分片时，新建数据库由 mongos 选一个 **primary shard**，所有数据集中在那台。
- 这种集合在 `sh.status()` 下不会出现 chunk 列表。

### 2. 开启分片（hash 分片）
```js
use admin
db.runCommand({ enablesharding : "shijiange" })
db.runCommand({ shardcollection : "shijiange.myuser",
                key             : { _id: "hashed" } })
```
然后再次写入：
```js
use shijiange
for (i = 1; i <= 500; i++) {
  db.myuser.insert({ name: 'mytest' + i, age: i })
}
```
- 每条 doc 会按 `hash(_id)` 落入不同 chunk → 不同 shard。
- 通过 `sh.status()` 可看到该集合 chunk 在两个 shard 上的分布。

### 3. 分片键的两种方式
| 类型 | 示例 | 特点 |
|------|------|------|
| **range（范围）** | `{ user_id: 1 }` | 写入有局部性，适合范围查询；但易热点 |
| **hashed（哈希）** | `{ _id: "hashed" }` | 写入均匀，但范围查询要广播到所有 shard |

### 4. 验证容灾
- 关掉 configsvr 的 1 台 SECONDARY，集群仍可读写（剩余两节点维持多数派）。
- 关掉某个 shard 的 PRIMARY，shard 内会自动切主；本 shard 临时不可写但其它 shard 正常。
- mongos 是无状态的，可任意启停；客户端 URI 含多 mongos 时无感切换。

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | "默认添加数据没有分片存储" | 描述粗 | 准确：**未启用 sharding** 的库 / 集合，全部写入会落到该数据库的 *primary shard*；并非"没有分片"那么简单 |
| 2 | `db.runCommand({shardcollection:"shijiange.myuser",key:{_id:"hashed"}})` 未先 `enableSharding` 时也能跑？ | 实际必须先 `enablesharding` 才允许 `shardcollection` | 流程顺序：`enablesharding → shardcollection`（文档已写但需强调顺序） |
| 3 | "分片存储就会同一个 collection 分配两个数据角色" | 表述笼统 | 实际上是按 chunk（默认 64MB）切片分布到所有 shard，并非简单地"对半分" |
| 4 | "配置角色如果挂掉一台会不会有影响" 未给结论 | 知识点断 | 3 节点 configsvr 副本集挂 1 → 多数派仍在，集群继续工作；挂 2 → 元数据只读，新建集合/balancer 等会失败但已有读写仍可继续 |
| 5 | "验证 mongos 多个入口是否能够正常使用" 没给方法 | 缺操作 | 方法：在驱动 URI 写多个 mongos 地址，关掉其中一个，应用应能继续工作 |
| 6 | colloection 拼写错误 | typo | collection |

## 三、分片读写完整流程（ASCII）

```
   App / pymongo ── URI: mongodb://mongos1:27017,mongos2:27018,mongos3:27019/?...
        │
        ▼
   ┌──────────────────────────────────────────────┐
   │        mongos (任意一台)                     │
   │  ① 解析操作 (insert/find/update)            │
   │  ② 查 routing table（来自 configsvr）       │
   │       chunk_for(shardKey, doc)               │
   └──────────────┬───────────────────────────────┘
                  │
   ┌──────────────┴────────────────────────────────────┐
   │                                                   │
   ▼ 命中单个 shard（targeted）                         ▼ 跨 shard（broadcast / scatter-gather）
 ┌──────────────────────┐                       ┌──────────────────────┐
 │ shardsvr shijiange-  │                       │ 全部 shard 并行执行  │
 │  data1 / data2 PRIMARY│                       │ mongos 合并/排序结果 │
 └──────────┬───────────┘                       └──────────┬───────────┘
            │ 写入：oplog → SECONDARY                     │
            ▼                                              ▼
 ack 回 mongos ────────────────────────────────────────── ack 回 mongos
            │
            ▼
        返回 App

  分片键 vs 操作：
   ┌───────────┬────────────────────────────┐
   │ 分片键    │ 操作类型                    │
   ├───────────┼────────────────────────────┤
   │ 命中条件  │ targeted（单 shard，最快） │
   │ 范围查询  │ range 分片可只走部分 shard │
   │ 无键查询  │ scatter-gather（广播）     │
   └───────────┴────────────────────────────┘

  balancer：
   后台周期检查 chunk 分布是否倾斜，
   不平衡时跨 shard 迁移 chunk（写在 configsvr 协调）
```
