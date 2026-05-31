# 12 · Python 简单操作 MongoDB

## 一、知识点梳理

### 1. 驱动选择
- Python 操作 MongoDB 通用方式是 [PyMongo](https://pypi.org/project/pymongo/) 官方驱动。
- 安装：
  ```bash
  pip install pymongo            # 当前版本仅支持 Python 3.7+
  ```

### 2. 连接单实例
```python
import pymongo

client   = pymongo.MongoClient('127.0.0.1', 27017)
db       = client['shijiange']      # 等价 client.shijiange
coll     = db['myuser']             # 等价 db.myuser

for item in coll.find():
    print(item)
```

### 3. 连接副本集（推荐 URI）
```python
import pymongo

uri = 'mongodb://127.0.0.1:27017,127.0.0.1:27018,127.0.0.1:27019/?replicaSet=shijiange'
client = pymongo.MongoClient(uri)

coll = client.shijiange.myuser
coll.insert_one({'age': 20, 'name': 'shijiange'})

for item in coll.find():
    print(item)
```
- 用副本集 URI 时，驱动会自动跟踪主备切换。
- 列表形式 `MongoClient([h1,h2,h3])` 也可工作，但**没有显式声明 `replicaSet`，会被当作"多个独立实例"处理**，不会启用故障转移逻辑。

### 4. 常用 CRUD
```python
coll.insert_one ({'name':'a','age':1})
coll.insert_many([{'name':'b'},{'name':'c'}])

coll.find_one({'name':'a'})
list(coll.find({'age':{'$gt':0}}, {'_id':0}).sort('age',1).limit(10))

coll.update_one ({'name':'a'}, {'$set':{'age':2}})
coll.update_many({'age':{'$lt':5}}, {'$inc':{'age':1}})

coll.delete_one ({'name':'a'})
coll.delete_many({})
```

### 5. 鉴权连接
```python
uri = 'mongodb://app:secret@127.0.0.1:27017/?authSource=admin'
client = pymongo.MongoClient(uri)
```

### 6. 副本集服务端配置回顾
```yaml
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/27017/mongodb.log
storage:
  dbPath: /data/mongodb/27017/
  journal: { enabled: true }
processManagement:
  fork: true
net:
  port: 27017
  bindIp: 0.0.0.0
replication:
  replSetName: shijiange
```
副本集初始化（`mongo` shell 中）：
```js
config = { _id:"shijiange", members:[
  {_id:0, host:"127.0.0.1:27017"},
  {_id:1, host:"127.0.0.1:27018"},
  {_id:2, host:"127.0.0.1:27019"}
]}
use admin
rs.initiate(config)
```

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | `yum install python2-pip -y` 后 `pip install pymongo` | Python 2 已停止维护，最新 pymongo 不再支持 | 使用 `python3-pip` 与 Python 3.7+ |
| 2 | `myuser.insert(myvar)` | `insert()` 在 4.x 被废弃 | 使用 `insert_one()` / `insert_many()` |
| 3 | `MongoClient(['h1:p','h2:p','h3:p'])` "代码支持自动切换" | 不严谨 | 该写法不会自动启用副本集模式，正确做法是使用 URI 并显式 `replicaSet=shijiange` |
| 4 | 缺鉴权示例 | 生产场景必备 | 用 `mongodb://user:pwd@host/?authSource=admin` |
| 5 | 缺关闭连接 | 长进程容易泄露连接 | 使用 `with pymongo.MongoClient(uri) as client:` 或显式 `client.close()` |
| 6 | 直接 `for item in myuser.find():` 大集合会一次性内存溢出？ | 实际上 PyMongo 的 cursor 是惰性的，不会一次取完，但应了解 `batch_size`、`limit` 等控制手段 | 长查询配合 `.batch_size(1000)`，或使用 `with cursor:` 显式释放 |

## 三、PyMongo 与副本集交互（ASCII）

```
   ┌──────────────────────────┐
   │   Python 应用            │
   │   pymongo.MongoClient    │
   │   uri=...?replicaSet=... │
   └─────────────┬────────────┘
                 │ 拿到 SeedList: h1,h2,h3
                 ▼
   ┌──────────────────────────┐
   │ ServerMonitor 线程       │
   │  1) 周期 isMaster/hello  │
   │  2) 维护 TopologyDescr.  │
   └─────────────┬────────────┘
                 │ 找到 PRIMARY
                 ▼
   ┌──────────────────────────┐         ┌──────────────────────────┐
   │ 写操作 → PRIMARY (h1)    │ ──oplog→│  SECONDARY (h2)          │
   │ 读操作 → PRIMARY 或      │         │  SECONDARY (h3)          │
   │   readPreference=...     │         │  根据 readPreference 服务读 │
   └──────────────────────────┘         └──────────────────────────┘

  当 PRIMARY 失联：
   ServerMonitor 发现拓扑变更 → 自动选定新 PRIMARY → 客户端无感切换
```
