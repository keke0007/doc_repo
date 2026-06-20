# 04 · MongoDB 集合的多种查询方式

## 一、知识点梳理

### 1. 准备数据
```js
use shijiange
db.myuser.insert({ name: "shijiange1", age: 20 })
db.myuser.insert({ name: "shijiange2", age: 28 })
db.myuser.insert({ name: "shijiange3", age: 38 })
db.myuser.insert({ name: "zhangsan1",  age: 58 })
db.myuser.insert({ name: "zhangsan2",  age: 68 })
db.myuser.insert({ name: "zhangsan3",  age: 25 })
```

### 2. 美化输出
```js
db.myuser.find().pretty()
```

### 3. 限制条数与分页
```js
db.myuser.find().limit(2)               // 取前 2 条
db.myuser.find().skip(2).limit(2)       // 跳过 2 条，再取 2 条

// 分页（每页 2 条）
db.myuser.find().skip(0).limit(2)       // 第 1 页
db.myuser.find().skip(2).limit(2)       // 第 2 页
db.myuser.find().skip(4).limit(2)       // 第 3 页
```
注意：`skip()` 在数据量大时性能差，生产环境推荐基于 `_id` 或业务有序键的「游标式分页」。

### 4. 排序
```js
db.myuser.find().sort({ age: 1 })       // 升序
db.myuser.find().sort({ age: -1 })      // 降序
```

### 5. 比较运算符
| 运算符 | 含义 |
|--------|------|
| `$gt` | 大于 |
| `$gte` | 大于等于 |
| `$lt` | 小于 |
| `$lte` | 小于等于 |
| `$ne` | 不等于 |
| `$in` | 在集合内 |
| `$nin` | 不在集合内 |

```js
db.myuser.find({ age: { $lt: 30 } })
db.myuser.find({ age: { $in: [20, 28] } })
```

### 6. 逻辑组合
```js
db.myuser.find({ $or:  [ { name: 'shijiange1' }, { name: 'shijiange2' } ] })
db.myuser.find({ $and: [ { name: 'shijiange1' }, { age: 20 } ] })
```
- 同一个 field 平铺写多个条件相当于隐式 `$and`：`{ age: { $gt: 20, $lt: 50 } }`。

### 7. 正则
```js
db.myuser.find({ name: { $regex: "shijiange[1-9]" } })
db.myuser.find({ name: { $regex: "(zhangsan)" } })
db.myuser.find({ name: /^shijiange/ })          // 前缀匹配可走索引
```
- **前缀型正则**（`^xxx` 且大小写敏感）能利用索引；其它正则均退化为全表扫描。

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | "支持普通正则和扩展正则" | 描述不准确 | MongoDB 用 PCRE 兼容引擎，没有"普通/扩展"之分；只是支持完整正则语法（含分组等） |
| 2 | 未提及 `find().count()`、投影 `find(query, projection)` | 知识点缺失 | `db.myuser.find({age:{$lt:30}}, {name:1,_id:0})` 可只返回需要字段 |
| 3 | `$or`/`$and` 仅写组合，没有提到性能含义 | 易误用 | `$or` 多个条件需各自能命中索引否则全表扫描；同字段范围用 `$and` 没必要，直接合并即可 |
| 4 | `db.myuser.find().skip(N).limit(M)` 当作分页万能解法 | skip 在大表上 O(N) 扫描 | 对大集合改用 `_id > lastId` 之类的游标分页 |

## 三、查询执行顺序（ASCII）

```
   db.myuser.find(<filter>, <projection>)
        .sort(...).skip(N).limit(M)
                 │
                 ▼
   ┌─────────────────────────────────────────┐
   │ 1. 选择执行计划（IXSCAN / COLLSCAN）    │
   │    explain(true) 可查看                 │
   ├─────────────────────────────────────────┤
   │ 2. 应用 filter（$lt $gt $or $and ...）  │
   ├─────────────────────────────────────────┤
   │ 3. 排序（有索引则边扫边出，否则内存排序）│
   │    SORT 阶段内存上限 100MB              │
   ├─────────────────────────────────────────┤
   │ 4. skip / limit                         │
   ├─────────────────────────────────────────┤
   │ 5. projection（投影裁剪字段）           │
   └─────────────────────┬───────────────────┘
                         ▼
                   返回 BSON 游标
```
