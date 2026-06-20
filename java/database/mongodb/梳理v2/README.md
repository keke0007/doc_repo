# MongoDB 课件知识点梳理

本目录是对 `mongo/` 下 17 篇课件的二次加工：
- 抽取每篇文档的核心知识点
- 对原文中的命令、概念错误进行勘误
- 涉及多文件 / 多进程协同的场景，用 ASCII 流程图描述执行链路

## 目录

| # | 文档 | 主题 |
|---|------|------|
| 01 | [01_MongoDB简介与安装启动.md](./01_MongoDB简介与安装启动.md) | MongoDB 简介、二进制安装、配置文件、启动/关闭 |
| 02 | [02_MongoDB服务器启动优化.md](./02_MongoDB服务器启动优化.md) | THP、ulimit、普通用户、access control |
| 03 | [03_MongoDB客户端基础使用.md](./03_MongoDB客户端基础使用.md) | mongo shell、库/集合/文档基础 CRUD |
| 04 | [04_MongoDB集合的多种查询方式.md](./04_MongoDB集合的多种查询方式.md) | limit/skip/sort、比较运算、$or/$and、正则 |
| 05 | [05_MongoDB索引查询与建立.md](./05_MongoDB索引查询与建立.md) | 慢查询、explain、单列/唯一/复合/正则索引 |
| 06 | [06_MongoDB数据库的监控命令.md](./06_MongoDB数据库的监控命令.md) | mongostat、serverStatus、currentOp |
| 07 | [07_MongoDB副本集的搭建.md](./07_MongoDB副本集的搭建.md) | 副本集配置、初始化、数据同步 |
| 08 | [08_MongoDB副本集故障自动切换.md](./08_MongoDB副本集故障自动切换.md) | 多数派选举、故障演示 |
| 09 | [09_MongoDB副本集的优先级.md](./09_MongoDB副本集的优先级.md) | priority、votes、hidden、reconfig |
| 10 | [10_MongoDB副本集的伸缩.md](./10_MongoDB副本集的伸缩.md) | rs.add / rs.remove、初始同步 |
| 11 | [11_MongoDB的备份和恢复.md](./11_MongoDB的备份和恢复.md) | mongodump / mongorestore、--oplog |
| 12 | [12_Python简单操作MongoDB.md](./12_Python简单操作MongoDB.md) | pymongo CRUD、副本集 URI、鉴权 |
| 13 | [13_Python获取MongoDB状态信息.md](./13_Python获取MongoDB状态信息.md) | serverStatus 关键指标、QPS 差分采样 |
| 14 | [14_MongoDB分片集群之configsvr.md](./14_MongoDB分片集群之configsvr.md) | configsvr 副本集（CSRS） |
| 15 | [15_MongoDB分片集群之router.md](./15_MongoDB分片集群之router.md) | mongos 配置与路由 |
| 16 | [16_MongoDB分片集群之shardsvr.md](./16_MongoDB分片集群之shardsvr.md) | shardsvr 副本集 |
| 17 | [17_MongoDB分片集群使用.md](./17_MongoDB分片集群使用.md) | sh.addShard、enableSharding、hash 分片 |

## 阅读建议

- 第 1~6 篇打基础（单机 + shell + 索引 + 监控）。
- 第 7~10 篇专攻副本集（高可用）。
- 第 11 篇备份恢复，副本集起来后才好谈。
- 第 12~13 篇 Python 操作与监控。
- 第 14~17 篇分片集群，按 configsvr → mongos → shardsvr → 使用 这个顺序看。

## 分片集群整体架构（速览）

```
   App ── mongos(无数据,做路由)
            │     │
            │     └── 启动时连接 ──▶ configsvr 副本集（元数据）
            │                              │
            └── 路由到 ──▶ shardsvr 副本集 1（chunk 0,2,...）
                          shardsvr 副本集 2（chunk 1,3,...）
                          ...
```

每篇文档末尾包含 ASCII 流程图，方便对照命令理解执行链路。

## 主要勘误集中点

1. mongo shell 的废弃 API：`ensureIndex` → `createIndex`、`rs.slaveOk()` → `rs.secondaryOk()`、`rs.printSlaveReplicationInfo()` → `rs.printSecondaryReplicationInfo()`、`insert/update/remove` → `*One/*Many`。
2. 副本集选举条件应以**多数派**而非"集群台数 ≥ 2"理解。
3. 分片集群的 configsvr 自 3.4 起强制为副本集（CSRS）；shard 必须副本集才能容灾。
4. `mongodump`/`mongorestore` 自 4.4 起为独立 database tools 包；备份可在 hidden secondary 上做，不必锁死 PRIMARY。
5. PyMongo 副本集连接应使用 URI 模式 `?replicaSet=...`，列表参数不会启用故障转移。
6. 课件中端口写错（27017 写成 2017）、`shardsvr` 写成 `sharedsvr`、`field` 写成 `filed`、`show tables` 写成 `jhow tables`、collection 写成 colloection 等常见笔误已逐项指出。
