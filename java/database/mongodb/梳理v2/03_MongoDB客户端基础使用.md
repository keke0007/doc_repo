# 03 · MongoDB 客户端基础使用

## 一、知识点梳理

### 1. 基础概念对照
| MongoDB | MySQL | 说明 |
|---------|-------|------|
| database | database | 数据库 |
| collection | table | 集合 |
| document | row | 一条记录（BSON 文档） |
| field | column | 字段 |
| _id | 主键 | 默认主键，类型 ObjectId |

### 2. 进入客户端
```bash
/usr/local/mongodb/bin/mongo                 # 默认 127.0.0.1:27017
/usr/local/mongodb/bin/mongo 127.0.0.1:27017
```
客户端支持 `Tab` 补全。

### 3. 库与集合的隐式创建
```js
use shijiange                                   // 切换/创建数据库（写入数据后才真正落库）
db.myuser.insert({ name: 'shijiange1', age: 28 })   // 集合 myuser 自动创建
show dbs;
show collections;
```

### 4. 增删改查
```js
db.myuser.find()                                            // 查全部
db.myuser.find({ name: 'shijiange1' })                      // 条件查询
db.myuser.update({ age: 28 }, { $set: { age: 30 } })        // 更新
db.myuser.remove({ name: 'shijiange2' })                    // 按条件删除
db.myuser.remove({})                                        // 清空集合
db.myuser.drop()                                            // 删除集合
db.dropDatabase()                                           // 删除当前库
```

### 5. 优雅关闭服务端
```js
use admin
db.shutdownServer()
```

### 6. 自带库不要动
- `admin`：账号/权限/集群命令。
- `local`：节点本地数据，副本集 oplog 即在此。
- `config`：分片元数据。

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | `filed` | 拼写错误 | `field` |
| 2 | `jhow tables;` | 拼写错误 | `show tables;`（mongo shell 中 `show tables` 与 `show collections` 等价） |
| 3 | `db.myuser.update(...)` 未写第 3、4 个参数 | 仅默认更新匹配到的**第一条**，并不是所有匹配项 | 想批量更新需加 `{ multi: true }`，或使用 `updateMany()` |
| 4 | `db.myuser.remove(...)` | 该 API 在 5.0+ 已废弃 | 推荐使用 `deleteOne()` / `deleteMany()` |
| 5 | "key value 的字典方式插入" | 文档实际是 BSON，并非纯字典 | BSON 是二进制扩展 JSON，支持 ObjectId、Date、Decimal128 等额外类型 |
| 6 | `insert()` | 4.x 起逐步弃用 | 推荐 `insertOne()` / `insertMany()` |

## 三、客户端到服务端调用关系（ASCII）

```
   user 输入命令
        │
        ▼
 ┌─────────────────────┐    TCP 27017     ┌──────────────────────┐
 │   mongo (shell)     │ ───────────────▶ │  mongod (服务端)     │
 │   解析 JS 表达式    │                  │  WiredTiger 引擎     │
 └─────────────────────┘ ◀─────────────── │  data / journal      │
            ▲   游标/结果 BSON           └──────────────────────┘
            │
            └── show dbs / find() 等命令最终都翻译成 wire protocol 的 OP_MSG
```
