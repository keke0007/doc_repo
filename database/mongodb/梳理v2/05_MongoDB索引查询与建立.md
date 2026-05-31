# 05 · MongoDB 索引查询与建立

## 一、知识点梳理

### 1. 准备 50 万条数据
```js
use shijiange
for (i = 1; i <= 500000; i++) {
  db.myuser.insert({ name: 'mytest' + i, age: i })
}
```

### 2. 慢查询
- MongoDB 默认 `slowms = 100ms`，慢操作会写入 `mongodb.log`。
- 查询/调整慢日志阈值：
  ```js
  db.getProfilingStatus()
  db.setProfilingLevel(1, { slowms: 50 })   // 1=只记录慢查询；2=记录全部
  ```

### 3. explain 查执行计划
```js
db.myuser.find({ age: 9999 }).explain(true)
```
关键字段：
- `winningPlan.stage`：`COLLSCAN`（全表扫描）或 `IXSCAN`（索引扫描）。
- `executionStats.totalDocsExamined`：实际扫描的文档数。

### 4. 普通索引
```js
db.myuser.getIndexes()                        // 默认有 _id 索引
db.myuser.createIndex({ age: 1 })             // 1 升序，-1 降序
db.myuser.dropIndex({ age: 1 })
```

### 5. 唯一索引
```js
db.myuser.remove({})
db.myuser.createIndex({ userid: 1 }, { unique: true })
db.myuser.insert({ userid: 1 })
db.myuser.insert({ userid: 1 })   // 报 E11000 duplicate key
```

### 6. 正则与索引
- **前缀** + **大小写敏感** 的正则能走索引：
  ```js
  db.myuser.createIndex({ name: 1 })
  db.myuser.find({ name: /^mytest1/ })   // 走索引
  ```
- 中间匹配类正则（如 `/99999/`）一律全表扫描，必须 `COLLSCAN`。

### 7. 其它常见索引类型
| 类型 | 创建示例 | 场景 |
|------|----------|------|
| 复合索引 | `{ age:1, name:1 }` | 多字段联合查询，遵循「最左前缀」 |
| 多 key 索引 | 数组字段自动建立 | 数组元素查询 |
| 文本索引 | `{ name: "text" }` | 全文搜索 |
| Hash 索引 | `{ _id: "hashed" }` | 分片、哈希均匀分布 |
| TTL 索引 | `createIndex({createdAt:1},{expireAfterSeconds:3600})` | 自动过期数据 |

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | `db.myuser.ensureIndex(...)` | `ensureIndex` 自 3.0 起被 `createIndex` 取代，5.0 之后已彻底废弃 | 统一改用 `db.myuser.createIndex({ age: 1 })` |
| 2 | "使用正则的话，索引无效果" | 表述太绝对 | 正则中**前缀且大小写敏感**（`/^abc/`）依然能走索引；只有非锚定/带 `i` 标志的才退化为全表 |
| 3 | "默认是超过 100ms 会记录慢日志 mongodb.log" | 仅当 `profile` ≥ 1 或 `slowOpThresholdMs` 命中且日志级别允许时才会记录 | 默认 `profile = 0`，但慢操作仍会写入 `mongodb.log`（由 `slowOpThresholdMs` 控制） |
| 4 | 未提到 `_id` 索引可用 `db.collection.find({_id:..})` 直接命中 | 知识点不全 | `_id` 是默认且不可删除的唯一索引 |
| 5 | "因为是唯一索引，所以会报错"未给出实际错误信息 | 模糊 | 实际抛出 `E11000 duplicate key error collection: shijiange.myuser` |

## 三、查询是否走索引的判定（ASCII）

```
              find({ age: 9999 })
                       │
                       ▼
        ┌─────────────────────────────┐
        │  查询优化器 (Plan Cache)    │
        └────────────┬────────────────┘
                     │ 是否存在 age 字段索引？
           ┌─────────┴──────────┐
           │ 否                  │ 是
           ▼                     ▼
   ┌───────────────┐     ┌─────────────────────┐
   │   COLLSCAN    │     │     IXSCAN          │
   │   全表扫描    │     │  B-Tree 索引查找    │
   │   50 万次比较 │     │  log2(50万)≈19 跳   │
   └───────┬───────┘     └──────────┬──────────┘
           │                        │
           ▼                        ▼
        慢查询                   微秒级返回
        写入 log                  
```
