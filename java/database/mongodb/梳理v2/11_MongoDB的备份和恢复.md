# 11 · MongoDB 的备份与恢复

## 一、知识点梳理

### 1. 备份/恢复工具
| 工具 | 用途 |
|------|------|
| `mongodump` | 把数据库导出为 BSON 文件（集合 + metadata.json） |
| `mongorestore` | 将 dump 目录导入到另一台 MongoDB |
| `mongoexport` / `mongoimport` | JSON/CSV 格式，适合跨工具迁移 |
| 物理备份 | 关停 mongod 后整目录拷贝 dbPath，或对文件系统/卷做快照 |

注意：`mongodump`/`mongorestore` 在 4.4+ 版本被拆分到独立的 [database tools](https://www.mongodb.com/try/download/database-tools) 包，`/usr/local/mongodb/bin/` 中默认仅 4.0 时代自带。

### 2. 备份所有库
```bash
/usr/local/mongodb/bin/mongodump -h 127.0.0.1:27017 -o /data/mongodbbackup/
ll -h /data/mongodbbackup/
```
常用扩展：
```bash
mongodump -h 127.0.0.1:27017 -d shijiange  -o /tmp/bk     # 仅备份单库
mongodump -h ... -d shijiange -c myuser    -o /tmp/bk     # 仅备份单集合
mongodump --uri="mongodb://user:pwd@h:27017/?authSource=admin" -o /tmp/bk
mongodump --gzip --archive=/tmp/bk.gz                     # 单文件压缩归档
```

### 3. 恢复
```bash
/usr/local/mongodb/bin/mongorestore -h 127.0.0.1:27018 /data/mongodbbackup/
```
可选：
```bash
mongorestore --drop ...                       # 恢复前先 drop 同名集合（防数据混杂）
mongorestore --gzip --archive=/tmp/bk.gz      # 配合 mongodump --archive 使用
mongorestore -d newname --nsInclude='shijiange.*' --nsFrom='shijiange.*' --nsTo='newname.*' ...
```

### 4. 拓扑要求
- **单实例**：直接连接备份。
- **副本集**：必须连接 PRIMARY；若想避免影响线上读写，可连接 SECONDARY 并加 `--readPreference=secondary`。
- **分片集群**：推荐对每个 shard 的副本集分别 dump；或对 config server + 各 shard 分开备份；商用场景更适合用文件系统快照。

### 5. 备份策略建议
- 全量 + oplog：`mongodump --oplog` 可追加 oplog 备份，配合 `mongorestore --oplogReplay` 实现"接近时间点恢复"。
- 周期：每日全量；高频场景再加 oplog 增量。
- 异地：dump 落到独立机器/对象存储，避免与原机一并丢失。

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | `mongorestore -h 127.0.0.1:27018` | 文档前面备份用 27017，恢复又指向 27018，未交代是另起一台 | 应说明：演示场景下「备份在 27017，恢复演示去到一台空 mongod 27018」 |
| 2 | "副本集需要连接到 primary 上备份" | 不严谨 | 也可连 SECONDARY 备份，需 `--readPreference=secondary`。甚至生产中常见做法就是在专门的备份从节点（hidden）上跑 mongodump |
| 3 | 未提及一致性与 oplog | 缺关键能力 | 大集合下加 `--oplog`，恢复时 `--oplogReplay`，可获得一致快照 |
| 4 | 未提及 4.4+ 工具分离 | 易踩坑 | 4.4 起需要单独下载 database-tools，否则找不到 mongodump |
| 5 | 未对鉴权场景说明 | 实际生产 100% 开启 auth | 备份时需 `-u/-p --authenticationDatabase admin` |

## 三、备份/恢复流程图（ASCII）

```
                       ┌──────────────────────────────┐
                       │   生产 mongod (27017)        │
                       │   shijiange / admin / ...    │
                       └────────────┬─────────────────┘
                                    │ TCP wire protocol
                          mongodump (-h / --uri)
                                    │
                                    ▼
                  ┌─────────────────────────────────┐
                  │ /data/mongodbbackup/            │
                  │   admin/                        │
                  │   shijiange/                    │
                  │     myuser.bson                 │
                  │     myuser.metadata.json        │
                  │   ...                           │
                  └────────────┬────────────────────┘
                               │
                               │  传输到目标机器（可压缩归档）
                               ▼
                  ┌─────────────────────────────────┐
                  │ /data/mongodbbackup/  (目标机)  │
                  └────────────┬────────────────────┘
                               │ mongorestore -h target:27018
                               ▼
                       ┌──────────────────────────────┐
                       │  目标 mongod (27018)         │
                       │  完成集合/索引的重放         │
                       └──────────────────────────────┘

  副本集场景：mongodump 连 PRIMARY 或 hidden SECONDARY
  分片集群：分别对 configsvr + 每个 shardsvr 副本集做 dump
```
