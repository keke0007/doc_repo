# 13 · Python 获取 MongoDB 的状态信息

## 一、知识点梳理

### 1. 监控目标
所有 mongod 实例（包含 PRIMARY、SECONDARY、configsvr、shardsvr）都应单独纳入监控，因为 `serverStatus()` 数据是「本进程视角」的。

### 2. 拉取完整状态
```python
import pymongo

client = pymongo.MongoClient('127.0.0.1', 27017)
db     = client.admin
status = db.command('serverStatus')
print(status)
```
`db.command('serverStatus')` 与 `mongo` shell 中的 `db.serverStatus()` 完全等价。

### 3. 单独输出每个 key
```python
for key, value in status.items():
    print(key)
    print(value)
    print()
```

### 4. 推荐重点采集 3 类
| key | 含义 |
|------|------|
| `connections` | 连接数（current/available/totalCreated） |
| `network` | 进出口字节、请求数 |
| `opcounters` | insert/query/update/delete/getmore/command 累计计数 |

```python
status = db.command('serverStatus')
print(status['connections'])
print(status['network'])
print(status['opcounters'])
```

### 5. 衍生指标（生产更有用）
- 通过两次采样差值得到 QPS、流量速率：
  ```python
  import time
  s1 = db.command('serverStatus')['opcounters']
  time.sleep(5)
  s2 = db.command('serverStatus')['opcounters']
  qps = {k: (s2[k] - s1[k]) / 5 for k in s1}
  print(qps)
  ```
- 其它有价值字段：`mem`, `metrics`, `wiredTiger.cache`, `globalLock`, `repl`(副本集)，分片场景下还可看 `sharding`。

### 6. 鉴权
生产实例几乎都开启了 auth，连接需带凭证：
```python
client = pymongo.MongoClient(
  'mongodb://monitor_user:pwd@127.0.0.1:27017/?authSource=admin'
)
```
建议为监控创建独立角色 `clusterMonitor`，不给数据写权限。

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | "一般状态信息需要每个实例都进行监控" | 表述对，但未解释原因 | `serverStatus` 是 per-mongod 的，副本集 / 分片必须每节点独立采样 |
| 2 | 缺采样间隔与差分 | 把累计计数当瞬时值 | `opcounters` 为累计值，必须做 Δ/Δt 才得到 QPS |
| 3 | `db = client.admin` 用作所有命令的入口 | 实际上 `serverStatus` 在任何库都能跑 | 但 `replSetGetStatus`、`shardConnPoolStats` 等只能在 admin 上执行，仍以 admin 为统一入口更稳 |
| 4 | 未鉴权 | 生产用法 | 加 `authSource=admin` 与最小权限角色 `clusterMonitor` |
| 5 | `print()` 输出大字典对人不友好 | 体验差 | 推荐 `pprint` 或 `json.dumps(..., default=str, indent=2)` |

## 三、Python 监控数据流（ASCII）

```
  ┌──────────────────────────────────────────────────────────┐
  │  各 mongod 实例（mongod_27017、27018、29017 ...）        │
  │   serverStatus()  ← 内部计数器/速率                      │
  └─────────────────────────────┬────────────────────────────┘
                                │ wire protocol (admin.$cmd)
                                ▼
                        ┌──────────────────┐
                        │ Python 监控脚本  │
                        │  pymongo client  │
                        │  循环采样        │
                        └────────┬─────────┘
                                 │ 差分 / 阈值判断
              ┌──────────────────┼─────────────────────┐
              ▼                  ▼                     ▼
       Prometheus           Zabbix item            日志/告警
      (push gateway)        (custom check)          (DingTalk/微信)

  关键采集：
   - connections.current  → 连接趋势
   - network.bytesIn/Out  → 带宽
   - opcounters.*         → 增删改查 QPS
   - repl                 → 副本集状态（仅副本集成员）
```
