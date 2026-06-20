# 06 · MongoDB 数据库的监控命令

## 一、知识点梳理

### 1. mongostat — 实时监控
```bash
/usr/local/mongodb/bin/mongostat --help
/usr/local/mongodb/bin/mongostat -h 127.0.0.1:27017
```
关键列：
| 列 | 含义 |
|----|------|
| insert/query/update/delete | 每秒增/查/改/删次数 |
| getmore | 游标续读次数 |
| command | 命令次数 |
| dirty / used | WiredTiger 缓存脏数据/已用比例 |
| qr / qw | 等待读/写的客户端排队数 |
| ar / aw | 活跃读/写连接数 |
| netIn / netOut | 网络流入/流出 |
| conn | 当前连接数 |

### 2. 压测脚本
```js
use shijiange
for (i = 1; i <= 300000; i++) {
  db.myuser.insert({ name: 'mytest' + i, age: i })
}
```
执行该脚本时另起一个窗口看 `mongostat`，可观察 insert 列瞬时上升。

### 3. db.serverStatus()
```js
db.serverStatus()                  // 全量
db.serverStatus().connections      // 连接信息
db.serverStatus().network          // 流量
db.serverStatus().opcounters       // 增删改查计数
db.serverStatus().mem              // 内存
db.serverStatus().wiredTiger       // 引擎细节
```

### 4. 非交互式调用
```bash
echo 'db.serverStatus().opcounters' | /usr/local/mongodb/bin/mongo 127.0.0.1:27017
```
也可用 `--eval`：
```bash
/usr/local/mongodb/bin/mongo --quiet --eval 'JSON.stringify(db.serverStatus().opcounters)' 127.0.0.1:27017/admin
```

### 5. 推荐重点关注的 3 个指标
- **connections.current / available**：连接是否被打满。
- **network.bytesIn / bytesOut**：实例进出口带宽。
- **opcounters**：QPS 走势，读写比。

### 6. 其它常用命令
```js
db.currentOp()                  // 当前正在执行的操作
db.killOp(opid)                 // 杀掉慢操作
db.stats()                      // 库级统计
db.myuser.stats()               // 集合级统计（大小、索引大小、存储大小）
rs.status()                     // 副本集状态
sh.status()                     // 分片状态
```

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | 仅介绍了 `serverStatus()`、`mongostat` | 监控体系不完整 | 补 `currentOp` / `db.stats()` / 集合 `stats()` 等 |
| 2 | `echo 'db.serverStatus()' \| mongo` | 输出体量大，难直接阅读 | 推荐 `--quiet --eval` 配合 `JSON.stringify(...)`，便于 grep/jq |
| 3 | 未提到鉴权下的访问 | 实际部署都开启 auth | 命令需带 `-u/-p` 或在 admin 库 `db.auth()` |
| 4 | mongostat 的输出列含义未解释 | 仅给一行命令 | 补「dirty/used/qr/qw/ar/aw」等列含义 |

## 三、监控数据来源关系（ASCII）

```
                            ┌────────────────────────┐
                            │    mongod 进程内部     │
                            │  ────────────────────  │
                            │  连接管理 / opcounter  │
                            │  WiredTiger 缓存       │
                            │  网络收发计数          │
                            └──────────┬─────────────┘
                                       │ 内部统计采样
              ┌────────────────────────┴──────────────────────────┐
              ▼                                                    ▼
   ┌──────────────────────┐                         ┌──────────────────────────┐
   │  mongostat (CLI)     │  poll 1s                │  db.serverStatus()       │
   │  按列实时刷新        │ ─────────────────────▶  │  返回完整 BSON 文档      │
   └──────────────────────┘                         └────────────┬─────────────┘
              ▲                                                  │
              │  bash 管道                                       │
   ┌──────────┴───────────┐                          ┌───────────▼────────────┐
   │ echo "..." \| mongo  │                          │  Python pymongo        │
   │ 非交互式抓单次值     │                          │  db.command(...)       │
   └──────────────────────┘                          └────────────────────────┘
```
