# MongoDB 基本使用 - 知识梳理

## 第一部分：NoSQL与分布式理论

### 1. NoSQL 概念与特点

#### NoSQL 定义
- **全称**: Not Only SQL（不仅仅是SQL）
- **性质**: 非关系型数据库管理系统
- **应用场景**: 超大规模数据存储（Google、Facebook等）

#### RDBMS vs NoSQL 对比

| 对比项 | RDBMS | NoSQL |
|--------|--------|--------|
| **数据结构** | 高度组织化结构化数据 | 半结构化、非结构化数据 |
| **查询语言** | SQL | 无声明性查询语言 |
| **Schema** | 预定义Schema | 无预定义模式 |
| **数据存储** | 表(Table) | 键值对、列存储、文档、图数据库 |
| **一致性** | ACID属性 | 最终一致性 |
| **关系** | 支持表连接 | 无表连接 |

#### NoSQL 优缺点

**优点**
- 高可扩展性（水平扩展）
- 分布式计算能力
- 低成本部署
- 架构灵活性，支持半结构化数据
- 无复杂关系管理

**缺点**
- 无标准化
- 查询功能有限
- 最终一致性不够直观

### 2. 分布式理论基础

#### CAP 定理

**三个特性**
1. **C (Consistency)**: 一致性 - 所有节点同时具有相同数据
2. **A (Availability)**: 可用性 - 每个请求都能收到响应
3. **P (Partition Tolerance)**: 分区容错性 - 系统可持续提供服务

**核心结论**: 不可能同时满足三个特性，最多只能同时满足两个

**分类**
- **CA**: 单点集群，强一致性和可用性，但可扩展性差
- **CP**: 强一致性和分区容错，但性能一般
- **AP**: 高可用和分区容错，但一致性较弱

#### BASE 理论

**三个核心概念**
- **Basically Available**: 基本可用
- **Soft State**: 软状态 - 允许中间状态存在，系统不同节点间副本可异步同步
- **Eventually Consistent**: 最终一致性 - 数据在一段时间后达到一致

**适用场景**: NoSQL数据库对可用性和一致性的弱要求

---

## 第二部分：MongoDB 基础概念

### 3. MongoDB 简介与存储结构

#### MongoDB 特性
- C++ 编写，开源分布式文档数据库
- 基于**文档(Document)**数据模型
- 数据格式: **BSON**(Binary JSON) - 类似JSON
- 支持临时查询(ad-hoc queries)
- 索引结构: B-tree(3.2版本+ 支持 LSM树)
- 支持二级索引提升查询效率

#### 基本概念对应关系

| 传统数据库 | MongoDB | 说明 |
|----------|---------|------|
| Database | Database | 数据库 |
| Table | Collection | 集合 |
| Row | Document | 文档 |
| Column | Field | 字段 |
| Index | Index | 索引 |
| Primary Key | _id | 主键(自动生成) |

#### 文档示例

```json
{
    "_id": ObjectId("5146bb52d8524270060001f3"),
    "name": "John",
    "age": 25,
    "email": "john@example.com",
    "tags": ["mongodb", "database"],
    "address": {
        "city": "Beijing",
        "country": "China"
    }
}
```

### 4. 数据库、集合、文档操作

#### 数据库操作

```javascript
// 查看所有数据库
show dbs;

// 显示当前数据库
db;

// 创建/切换数据库
use dbname;

// 删除数据库
db.dropDatabase();

// 注意: 数据库只有在插入集合后才会真正创建
db.xxx.insertOne({"name":"test"});
```

**数据库命名规范**
- 不能为空字符串
- 不允许出现 `.`, `$`, `/`, `\`, `\0` 字符
- 建议使用小写
- 不超过64字节

**特殊数据库**
- `admin`: root级别数据库，拥有全库权限
- `local`: 单机本地数据库，不可复制
- `config`: 分布式部署时存储shard配置

#### 集合操作

```javascript
// 创建集合
db.createCollection("blog");

// 查看集合
show collections;
show tables;

// 删除集合
db.blog.drop();
```

**集合命名规范**
- 不允许空字符串
- 不能包含 `\0` 字符
- 不能创建 `system.` 开头的集合

#### 文档字段规范

- **Key**: UTF-8字符串
  - 不能包含 `\0` 字符
  - 保留 `.` 和 `$` (特殊修饰符)
  - 保留 `_` 开头的key(如`_id`)
- **Value**: 弱类型，可嵌入文档、数组
- **有序性**: key/value在MongoDB中是有序的
- **大小写敏感**: `{foo: 3}` 和 `{Foo: 3}` 是不同的文档

---

## 第三部分：CRUD 操作

### 5. 插入操作

#### insert() - 不推荐(已弃用)

```javascript
db.blog.insert({"title": "MongoDB教程", "likes": 100});

// 若主键重复，抛出 DuplicateKeyException
db.blog.insert({"_id": "1", "title": "Test"});
db.blog.insert({"_id": "1", "title": "Test2"}); // 错误！
```

#### insertOne() - 单条插入(推荐)

```javascript
db.blog.insertOne({
    "title": "MongoDB教程",
    "description": "MongoDB是NoSQL数据库",
    "likes": 100
});

// 返回值
{
    "acknowledged": true,
    "insertedId": ObjectId("6076b89de04dc9112613227f")
}
```

#### insertMany() - 批量插入(推荐)

```javascript
db.blog.insertMany([
    {"title": "教程1", "likes": 100},
    {"title": "教程2", "likes": 200}
]);

// 返回值
{
    "acknowledged": true,
    "insertedIds": [
        ObjectId("6077fc70aec5824bb77a5237"),
        ObjectId("6077fc70aec5824bb77a5238")
    ]
}
```

### 6. 查询操作

#### 基本查询

```javascript
// 查询所有文档
db.blog.find();

// 格式化输出
db.blog.find().pretty();

// 查询第一个文档
db.blog.findOne();

// 等值查询
db.blog.find({"title": "MongoDB教程"}).pretty();

// 只显示指定列(投影)
db.blog.find({}, {"title": 1}).pretty();      // 只显示title
db.blog.find({}, {"title": 1, "by": 1}).pretty(); // 显示多列
db.blog.find({}, {"title": 0, "by": 0}).pretty(); // 不显示指定列

// 注意: 投影中 0 和 1 不能混用(除了_id可以单独设置)
db.blog.find({}, {"title": 1, "by": 0}); // 错误！
```

### 7. 查询操作符

#### 比较操作符

| 操作符 | 含义 | 示例 | SQL等价 |
|--------|------|------|---------|
| `$eq` | 等于 | `{"pop": {$eq: 100}}` | `pop = 100` |
| `$lt` | 小于 | `{"pop": {$lt: 10}}` | `pop < 10` |
| `$lte` | 小于等于 | `{"pop": {$lte: 50}}` | `pop <= 50` |
| `$gt` | 大于 | `{"pop": {$gt: 100000}}` | `pop > 100000` |
| `$gte` | 大于等于 | `{"pop": {$gte: 90000}}` | `pop >= 90000` |
| `$ne` | 不等于 | `{"pop": {$ne: 0}}` | `pop != 0` |

#### 包含操作符

```javascript
// $in - 包含查询(OR逻辑)
db.zips.find({"state": {$in: ["MA", "NY"]}}).pretty();

// $nin - 不包含查询
db.zips.find({"state": {$nin: ["MA", "NY"]}}).pretty();
```

#### 字段存在判断

```javascript
// 查询存在某字段的文档
db.zips.find({"state": {$exists: true}}).pretty();

// 查询不存在某字段的文档
db.zips.find({"state": {$exists: false}}).pretty();
```

#### 多条件查询

```javascript
// 同一字段多个条件(AND)
db.zips.find({
    "pop": {$gte: 10, $lt: 50}
}).pretty();
```

### 8. 逻辑操作符

#### AND 操作

```javascript
// 隐式AND(推荐)
db.zips.find({
    "state": "NY",
    "pop": {$gt: 100000}
}).pretty();

// 显式AND(标准写法)
db.zips.find({
    $and: [
        {"state": "NY"},
        {"pop": {$gt: 100000}}
    ]
}).pretty();
```

#### OR 操作

```javascript
db.zips.find({
    $or: [
        {"state": "NY"},
        {"pop": {$lt: 0}}
    ]
}).pretty();
```

#### NOT 操作

```javascript
// 查询人数小于10的城市
db.zips.find({
    "pop": {$not: {$gte: 10}}
}).pretty();
```

#### 复杂条件组合

```javascript
// (state='NY' AND pop>10 AND pop<=50) OR (state IN ['MD','VA'] AND pop>10 AND pop<=50)
db.zips.find({
    $or: [
        {
            $and: [
                {"state": "NY"},
                {"pop": {$gt: 10, $lte: 50}}
            ]
        },
        {
            $and: [
                {"state": {$in: ["MD", "VA"]}},
                {"pop": {$gt: 10, $lte: 50}}
            ]
        }
    ]
}).pretty();
```

### 9. 排序、分页、统计

#### 排序

```javascript
// 升序排序(1表示升序，-1表示降序)
db.zips.find().sort({"pop": 1}).pretty();

// 降序排序
db.zips.find().sort({"pop": -1}).pretty();

// 多字段排序(先按state升序，相同则按pop降序)
db.zips.find({"pop": {$gt: 1000}})
    .sort({"state": 1, "pop": -1})
    .pretty();
```

#### 分页(不推荐方式)

```javascript
// 第一页(跳过0条，返回10条)
db.zips.find({}, {"_id": 1}).skip(0).limit(10);

// 第二页(跳过10条，返回10条)
db.zips.find({}, {"_id": 1}).skip(10).limit(10);

// 第三页(跳过20条，返回10条)
db.zips.find({}, {"_id": 1}).skip(20).limit(10);

// 注意: skip()会扫描全部文档再返回结果，性能差
```

#### 分页(推荐方式)

```javascript
// 第一页
db.zips.find({}, {_id: 1}).sort({"_id": 1}).limit(10);

// 第二页(基于第一页的最后一条记录)
db.zips.find({"_id": {$gt: "01020"}}, {_id: 1})
    .sort({"_id": 1}).limit(10);

// 优点: 避免扫描已看过的记录，性能更优
// 缺点: 不适合跳页
```

#### ObjectId 有序性

```javascript
// ObjectId构成: 时间戳(4字节) + 机器码(3字节) + PID(2字节) + 计数器(3字节)
// 举例: ObjectId("5b1886f8965c44c78540a4fc")
// 前8个字符(5b1886f8)为时间戳(16进制)

// Java代码查看ObjectId生成规则
public ObjectId(Date date) {
    this(dateToTimestampSeconds(date), 
         MACHINE_IDENTIFIER, 
         PROCESS_IDENTIFIER, 
         NEXT_COUNTER.getAndIncrement(), 
         false);
}

// 问题: 同一秒内顺序无保证，分布式环境时钟同步困难
// 解决: 使用雪花算法ID作为排序基准
```

#### 统计查询

```javascript
// count() - 统计符合条件的文档数
db.zips.find({"pop": {$not: {$gte: 10}}}).count();

// 返回结果: 118

// distinct() - 去重查询(返回数组)
db.zips.distinct("state");

// 有条件的去重
db.zips.distinct("state", {"pop": {$gt: 70000}});
// 返回: [ "CA", "FL", "IL", "MD", "MI", "NY", "PA", "TX", "WV" ]
```

### 10. 更新操作

#### update() 更新

```javascript
// 使用$set修饰符更新指定字段
db.blog.update({"_id": "1"}, {$set: {"likes": 666}});

// 返回值
{
    "nMatched": 1,      // 匹配的文档数
    "nUpserted": 0,     // upsert插入的文档数
    "nModified": 1      // 修改的文档数
}
```

#### save() 更新

```javascript
// save(): _id存在则更新，不存在则插入(全量替换)
db.blog.save({
    "_id": "1",
    "title": "MySQL教程",
    "likes": 100000
});

// 返回值(如果_id不存在，执行插入)
{
    "nMatched": 0,
    "nUpserted": 1,
    "nModified": 0,
    "_id": "1"
}

// 返回值(如果_id存在，执行更新)
{
    "nMatched": 1,
    "nUpserted": 0,
    "nModified": 1
}
```

### 11. 删除操作

#### remove() 删除

```javascript
// 条件删除
db.blog.remove({"_id": "1"});

// 只删除第一个匹配的文档
db.blog.remove({}, true);  // justOne=true

// 删除所有文档
db.blog.remove({});

// 返回值
{
    "nRemoved": 1
}
```

#### deleteOne() 删除单个(推荐)

```javascript
// 只删除符合条件的第一个文档
db.blog.deleteOne({});

// 返回值
{
    "acknowledged": true,
    "deletedCount": 1
}
```

#### deleteMany() 批量删除(推荐)

```javascript
// 删除所有匹配的文档
db.blog.deleteMany({});

// 返回值
{
    "acknowledged": true,
    "deletedCount": 8
}
```

---

## 第四部分：索引

### 12. 索引基础

#### 索引定义与作用

**定义**: 对集合中特定字段建立的数据结构，加速查询

**比喻**: 书籍目录 - 通过索引(目录)快速找到内容位置(页码)

**工作原理**
- 数据结构: B-tree
- 时间复杂度: O(log N) vs O(N)(全表扫描)
- 存储位置: 高速缓存(内存)

#### 索引性能分析

```javascript
// 假设B树度数d=100，记录数N=1亿
// O(log_d N) = O(log_100 10^8) = 4
// O(N) = 10^8
// 性能差异巨大！

// 索引优点:
// 1. 基于B树的O(logN)查询复杂度
// 2. 索引存储在内存，减少IO操作

// 索引缺点:
// 1. 写入操作需要额外的索引维护时间
// 2. 索引占用内存空间
```

### 13. 索引操作

#### 查看索引

```javascript
// 查看集合的所有索引
db.zips.getIndexes();

// 返回值
[
    {
        "v": 2,
        "key": {"_id": 1},
        "name": "_id_"
    }
]

// 查看数据库所有集合的索引
db.getCollectionNames().forEach(function(collection){
    indexes = db[collection].getIndexes();
    print("Indexes for [" + collection + "]:");
    printjson(indexes);
});
```

#### 创建索引

```javascript
// 基本语法
db.collection.createIndex(keys, options)

// 创建单字段索引(升序)
db.zips.createIndex({"pop": 1});

// 创建单字段索引(降序)
db.zips.createIndex({"pop": -1});

// 返回值
{
    "createdCollectionAutomatically": false,
    "numIndexesBefore": 1,
    "numIndexesAfter": 2,
    "ok": 1
}

// 创建带参数的索引
db.zips.createIndex(
    {"pop": -1},
    {
        "name": "pop_union_index",
        "expireAfterSeconds": 5
    }
);
```

#### 索引参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `background` | Boolean | 后台创建索引，默认false(阻塞其他操作) |
| `unique` | Boolean | 创建唯一索引，默认false |
| `name` | String | 索引名称，默认自动生成 |
| `sparse` | Boolean | 对不存在的字段不索引，默认false |
| `expireAfterSeconds` | Integer | TTL索引，指定过期时间(秒) |
| `partialFilterExpression` | Document | 局部索引条件 |

#### 删除索引

```javascript
// 根据索引名删除
db.zips.dropIndex("loc_2d");

// 根据字段删除
db.zips.dropIndex({"pop": 1});

// 删除所有非_id索引
db.zips.dropIndexes();

// 返回值
{
    "nIndexesWas": 2,
    "msg": "non-_id indexes dropped for collection",
    "ok": 1
}
```

### 14. 索引类型

#### 单键索引

```javascript
// 最简单最常用的索引类型
db.zips.createIndex({"pop": 1});

// 查看执行计划，验证索引生效
db.zips.find({"pop": {$gt: 10000}}).explain();

// 输出中stage字段: IXSCAN(索引扫描)表示走索引
```

#### 复合索引

```javascript
// 在多个字段上建立索引
db.zips.createIndex({
    "city": 1,      // 升序
    "state": -1     // 降序
});

// 索引排序规则: 先按city升序，city相同则按state降序

// 最佳左前缀法则: 索引字段顺序很重要
db.zips.find({"city": "CUSHMAN"}).explain();           // 走索引(左侧列)
db.zips.find({"city": "CUSHMAN", "state": "NY"}).explain(); // 走索引(全部列)
db.zips.find({"state": "NY"}).explain();               // 不走索引(跳过左侧列)

// 注意: MongoDB中的复合索引最多32个字段
```

#### 地理索引

```javascript
// 平面地理索引(小范围，平面坐标)
db.zips.createIndex({"loc": "2d"});

// 球面地理索引(大范围，考虑地球弧度)
db.zips.createIndex({"loc": "2dsphere"});

// 数据格式: GeoJSON
{
    "_id": "01002",
    "city": "CUSHMAN",
    "loc": [-72.51565, 42.377017],  // [经度, 纬度]
    "state": "MA"
}
```

#### 唯一索引

```javascript
// 创建唯一索引(强制字段值唯一)
db.zips.createIndex(
    {"id": 1, "city": 1},
    {unique: true, name: "id_union_index"}
);

// 作用: 防止重复插入
```

#### 局部索引(Partial Index)

```javascript
// 只对符合条件的文档建立索引
db.zips.createIndex(
    {pop: 1},
    {
        partialFilterExpression: {
            pop: {$gt: 10000}
        }
    }
);

// 查询人数<=10000不走索引(全表扫描)
db.zips.find({"pop": 9999}).explain();
// stage: "COLLSCAN"

// 查询人数>10000走索引
db.zips.find({"pop": 99999}).explain();
// stage: "IXSCAN"
```

---

## 第五部分：执行计划与性能优化

### 15. 执行计划分析

#### explain() 基本用法

```javascript
// 查看执行计划
db.zips.find({"pop": 99999}).explain();

// 返回示例
{
    "queryPlanner": {
        "plannerVersion": 1,           // 查询计划版本
        "namespace": "zips-db.zips",   // 查询集合
        "indexFilterSet": false,       // 是否使用索引
        "parsedQuery": {               // 解析后的查询条件
            "pop": {"$eq": 99999}
        },
        "winningPlan": {               // 最佳执行计划
            "stage": "FETCH",          // 查询方式
            "inputStage": {
                "stage": "IXSCAN",     // 索引扫描
                "indexName": "pop_1",
                "keyPattern": {"pop": 1}
            }
        },
        "rejectedPlans": []            // 被拒绝的执行计划
    }
}
```

#### explain() 参数及含义

```javascript
// 1. queryPlanner(默认)
db.zips.find({"pop": 99999}).explain("queryPlanner");
// 返回查询计划信息，不执行查询

// 2. executionStats - 返回执行统计信息
db.zips.find({"pop": 99999}).explain("executionStats");
// 包含nReturned, executionTimeMillis, totalKeysExamined等

// 3. allPlansExecution - 所有执行计划
db.zips.find({"pop": 99999}).explain("allPlansExecution");
```

#### 关键参数解释

| 参数 | 含义 |
|------|------|
| `stage` | 查询方式: COLLSCAN(全表扫描), IXSCAN(索引扫描), FETCH(根据索引检索文档), IDHACK(针对_id查询) |
| `nReturned` | 返回的文档数 |
| `totalKeysExamined` | 索引扫描次数 |
| `totalDocsExamined` | 文档扫描次数 |
| `executionTimeMillis` | 执行耗时(毫秒) |
| `executionSuccess` | 是否执行成功 |
| `rejectedPlans` | 被拒绝的执行计划 |

### 16. 慢查询分析

#### Profiling 简介

**作用**: 记录超过阈值的查询操作

**原理**: 使用system.profile(capped collection)存储日志

#### Profiling 级别

| 级别 | 说明 |
|------|------|
| 0 | 关闭，不收集任何数据 |
| 1 | 收集慢查询数据(默认100ms) |
| 2 | 收集所有数据 |

#### 开启/关闭 Profiling

```javascript
// 查看状态
db.getProfilingStatus();
// 返回: { "was": 0, "slowms": 100, "sampleRate": 1 }

// 开启级别2(收集所有)
db.setProfilingLevel(2);

// 开启级别1(收集慢查询，阈值100ms)
db.setProfilingLevel(1, 100);

// 关闭
db.setProfilingLevel(0);

// 全局设置(启动时配置)
mongod --profile=1 --slowms=200

// 配置文件设置
# mongodb.conf
profile = 1
slowms = 200
```

#### 分析慢查询

```javascript
// 查看所有慢查询日志
db.system.profile.find().pretty();

// 返回日志示例
{
    "op": "query",              // 操作类型(query, insert, update, remove)
    "ns": "zips-db.zips",       // 操作集合
    "command": {...},           // 命令详情
    "keysExamined": 0,          // 索引扫描次数
    "docsExamined": 0,          // 文档扫描次数
    "nreturned": 0,             // 返回文档数
    "millis": 544,              // 执行耗时(毫秒)
    "ts": ISODate("..."),       // 执行时间
    "client": "10.10.86.171",   // 连接IP
    "user": "...",              // 操作用户
    "locks": {...}              // 锁信息
}
```

#### 优化步骤

1. **定位慢查询**: 通过system.profile找到millis>200ms的语句
2. **分析执行计划**: 使用explain()查看是否走索引
3. **优化决策**: 
   - 若nscanned远大于nreturned，考虑建立索引
   - 若执行计划为COLLSCAN，需要创建索引

#### system.profile 日志解析

| 字段 | 含义 |
|------|------|
| `op` | 操作类型: query, insert, update, remove, getmore, command |
| `ns` | 操作的集合命名空间 |
| `ntoskip` | skip()方法跳过的文档数 |
| `nscanned` | 索引中浏览的文档数 |
| `nscannedObjects` | 集合中浏览的文档数 |
| `keyUpdates` | 索引更新的数量 |
| `numYield` | 操作为让其他操作完成而放弃的次数 |
| `millis` | 执行耗时(毫秒) |
| `responseLength` | 返回数据的字节长度 |

#### 最佳实践

```javascript
// 期望的查询方式(性能好)
Fetch + IDHACK
Fetch + IXSCAN
Limit + (Fetch + IXSCAN)
PROJECTION + IXSCAN
SHARDING_FILTER + IXSCAN

// 避免的查询方式(性能差)
COLLSCAN          // 全表扫描
SORT              // 内存排序(无索引)
不合理的SKIP      // skip()扫描被跳过的文档
SUBPLA            // 未用索引的$or
```

---

## 第六部分：Spring Data MongoDB 集成

### 17. Spring Boot 整合 MongoDB

#### 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

#### 配置文件

```yaml
server:
    port: 8080
spring:
    application:
        name: spring-boot-mongodb
    data:
        mongodb:
            database: test
            host: 192.168.10.30
            port: 27017
            # 可选: 用户认证
            # username: admin
            # password: admin
            # uri: mongodb://username:password@host:port/database
```

#### 定义实体类

```java
@Document("blog")  // 指定集合名称
public class Blog {
    @Id
    private String id;
    
    @Field("title")
    private String title;
    
    private String by;
    private String url;
    private List<String> tags;
    private int likes;
    
    // getter/setter...
}
```

#### DAO 层实现

```java
@Component
public class BlogDao {
    @Autowired
    private MongoTemplate mongoTemplate;
    
    // 插入文档
    public void insert(Blog blog) {
        mongoTemplate.insert(blog);
    }
    
    // 根据ID查询
    public Blog findById(String id) {
        return mongoTemplate.findById(id, Blog.class);
    }
    
    // 条件查询
    public List<Blog> find(Blog blog) {
        if (null == blog) {
            return null;
        }
        Criteria criteria = getFilter(blog);
        return mongoTemplate.find(Query.query(criteria), Blog.class);
    }
    
    // 构建查询条件
    public Criteria getFilter(Blog blog) {
        Criteria criteria = new Criteria();
        
        if (!StringUtils.isEmpty(blog.getTitle())) {
            criteria.andOperator(
                Criteria.where("title").is(blog.getTitle())
            );
        }
        
        if (!StringUtils.isEmpty(blog.getBy())) {
            criteria.andOperator(
                Criteria.where("by").is(blog.getBy())
            );
        }
        
        if (null != blog.getTags() && !blog.getTags().isEmpty()) {
            criteria.andOperator(
                Criteria.where("tags").in(blog.getTags())
            );
        }
        
        return criteria;
    }
    
    // 根据ID删除
    public void deleteById(String id) {
        mongoTemplate.remove(
            Query.query(Criteria.where("_id").is(id)), 
            Blog.class
        );
    }
}
```

#### Controller 层

```java
@RestController
@RequestMapping("/blog")
public class BlogController {
    @Resource
    private BlogDao blogDao;
    
    // 根据ID获取博客
    @RequestMapping("/{id}")
    @ResponseBody
    public String getBlogInfo(@PathVariable("id") String id) {
        Blog blog = blogDao.findById(id);
        if (null == blog) {
            return "访问的数据不存在";
        }
        return JSON.toJSONString(blog);
    }
    
    // 添加博客
    @RequestMapping("/add")
    @ResponseBody
    public String addBlog(@RequestBody Blog blog) {
        blogDao.insert(blog);
        return JSON.toJSONString(blog);
    }
    
    // 查询博客
    @RequestMapping("/find")
    @ResponseBody
    public String findBlog(@RequestBody Blog blog) {
        List<Blog> blogs = blogDao.find(blog);
        return JSON.toJSONString(blogs);
    }
    
    // 删除博客
    @RequestMapping("/delete/{id}")
    @ResponseBody
    public String deleteBlog(@PathVariable("id") String id) {
        blogDao.deleteById(id);
        return "删除成功";
    }
}
```

---

## 附录：ASCII 查询流程图

### 查询流程图 1: 客户端到服务器的查询链路

```
┌─────────────────────────────────────────────────────────────┐
│                     Java 应用客户端                          │
│                  (Spring Data MongoDB)                       │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           │ MongoTemplate.find()
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                 MongoDB Java Driver                          │
│              (驱动程序封装和连接管理)                         │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           │ Network Protocol
                           │ (Socket通信)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   MongoDB Server                             │
│                  (mongod 进程)                               │
├─────────────────────────────────────────────────────────────┤
│ ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│ │ 查询优化器    │  │ 索引管理     │  │ 存储引擎      │       │
│ │(Query        │  │(Index        │  │(Storage      │       │
│ │Optimizer)    │  │Manager)      │  │Engine)       │       │
│ └──────────────┘  └──────────────┘  └──────────────┘       │
│         │                 │                 │               │
│         └─────────────────┴─────────────────┘               │
│                        │                                    │
│                        ▼                                    │
│         ┌───────────────────────────────┐                  │
│         │     执行计划生成器             │                  │
│         │  (Query Execution Engine)    │                  │
│         └───────────┬───────────────────┘                  │
│                     │                                       │
│      ┌──────────────┴──────────────┐                       │
│      │ 选择执行策略                 │                       │
│      ▼                               ▼                       │
│  ┌─────────────┐            ┌──────────────┐              │
│  │ COLLSCAN    │            │ IXSCAN       │              │
│  │ (全表扫描)  │            │ (索引扫描)   │              │
│  └────┬────────┘            └──────┬───────┘              │
│       │                            │                      │
│       └────────────────┬───────────┘                       │
│                        ▼                                    │
│            ┌──────────────────────┐                        │
│            │ FETCH (取回文档)     │                        │
│            └──────────┬───────────┘                        │
│                       │                                    │
│                       ▼                                    │
│         ┌──────────────────────────┐                      │
│         │  检索集合/索引           │                      │
│         │  (Memory/Disk I/O)      │                      │
│         └──────────┬───────────────┘                      │
│                    │                                       │
│                    ▼                                       │
│         ┌──────────────────────────┐                      │
│         │   返回结果集合           │                      │
│         └──────────┬───────────────┘                      │
└──────────────────────┼───────────────────────────────────┘
                       │
                       │ Network Protocol
                       │ (返回文档JSON)
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                 MongoDB Java Driver                          │
│              (反序列化BSON为Java对象)                        │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           │ Document List
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                     Java 应用客户端                          │
│                   (获得List<Blog>)                          │
└─────────────────────────────────────────────────────────────┘
```

### 查询流程图 2: 索引匹配与检索过程

```
                    查询请求
                       │
                       ▼
            ┌─────────────────────────┐
            │   解析查询条件          │
            │   (parse query)         │
            └────────┬────────────────┘
                     │
                     ▼
        ┌────────────────────────────┐
        │   匹配可用索引             │
        │   (match indexes)          │
        └────────┬───────────────────┘
                 │
         ┌───────┴─────────┬──────────┐
         │                 │          │
         ▼                 ▼          ▼
    ┌────────┐    ┌──────────┐   ┌──────┐
    │单键索引 │    │复合索引  │   │无索引│
    │INDEX   │    │COMPOUND  │   │SCAN  │
    │SCAN    │    │INDEX     │   │ALL   │
    └────┬───┘    └────┬─────┘   └──┬───┘
         │             │            │
         │      ┌──────▼─────────┐  │
         │      │ 最佳左前缀法则 │  │
         │      │ (Leftmost      │  │
         │      │  Prefix Rule)  │  │
         │      └──────┬─────────┘  │
         │             │            │
         └─────────────┼────────────┘
                       │
                       ▼
        ┌──────────────────────────────┐
        │   生成执行计划               │
        │   (Create Execution Plan)    │
        └────────┬─────────────────────┘
                 │
         ┌───────┴────────┐
         │                │
         ▼                ▼
    ┌────────┐      ┌──────────┐
    │ IXSCAN │      │COLLSCAN  │
    │(索引扫描)│      │(全表扫描)│
    └────┬───┘      └────┬─────┘
         │               │
         │     ┌─────────┘
         │     │
         ▼     ▼
    ┌──────────────────┐
    │  FETCH 检索文档  │
    │  (Get Document)  │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │   应用过滤条件   │
    │   (Apply Filter) │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │   应用排序       │
    │   (Sort)         │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │   应用分页       │
    │   (Limit/Skip)   │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │   返回结果集合   │
    │   (Return)       │
    └──────────────────┘
```

### 查询流程图 3: Java 操作链路(CRUD)

```
Java 应用
   │
   ├─────────────────────────────────────────────┐
   │                                             │
   ▼                                             ▼
┌─────────────┐                         ┌──────────────┐
│  MongoTemplate.insert()                │MongoTemplate.find()
│  │ insertOne()                         │ │ find(Query)
│  │ insertMany()                        │ │ findOne()
│  ▼                                     │ ▼
│ Document → BSON                        │ BSON → Document
│ │                                      │ │
│ ▼                                      │ ▼
│ MongoDB Server                         │ MongoDB Server
│ (Insert Operation)                     │ (Query Operation)
│                                        │
└────────┬─────────────────────────────┬─┘
         │                             │
         │  (其他操作)                 │
         ▼                             ▼
    ┌────────────┐              ┌──────────────┐
    │  update()  │              │  delete()    │
    │  save()    │              │  remove()    │
    └────────────┘              └──────────────┘
         │                             │
         ▼                             ▼
    Update Operation             Delete Operation
         │                             │
         ▼                             ▼
    MongoDB Server             MongoDB Server
```

---

## 附录：核心知识点速查表

### 常用操作速查

| 操作 | 命令示例 | 说明 |
|------|---------|------|
| 创建数据库 | `use dbname` | 插入数据后才真正创建 |
| 创建集合 | `db.createCollection("name")` | 可选操作 |
| 插入单个 | `db.col.insertOne(doc)` | 推荐 |
| 插入多个 | `db.col.insertMany([doc1, doc2])` | 批量操作 |
| 查询所有 | `db.col.find()` | 基本查询 |
| 等值查询 | `db.col.find({key: value})` | 简写方式 |
| 大于查询 | `db.col.find({key: {$gt: val}})` | 比较操作符 |
| 排序查询 | `db.col.find().sort({key: 1})` | 1升序,-1降序 |
| 分页查询 | `db.col.find().skip(10).limit(20)` | 不推荐 |
| 更新操作 | `db.col.update({}, {$set: {}})` | 局部更新 |
| 替换操作 | `db.col.save(doc)` | 全量替换 |
| 删除操作 | `db.col.deleteOne({})` | 推荐 |
| 创建索引 | `db.col.createIndex({key: 1})` | 性能优化 |
| 查看执行计划 | `db.col.find().explain()` | 查询优化 |
| 统计数量 | `db.col.count(query)` | 计数 |
| 去重查询 | `db.col.distinct("key")` | 返回数组 |

### 查询操作符速查

| 操作符 | 含义 | 示例 |
|--------|------|------|
| `$eq` | 等于 | `{pop: {$eq: 100}}` |
| `$lt` | 小于 | `{pop: {$lt: 100}}` |
| `$lte` | 小于等于 | `{pop: {$lte: 100}}` |
| `$gt` | 大于 | `{pop: {$gt: 100}}` |
| `$gte` | 大于等于 | `{pop: {$gte: 100}}` |
| `$ne` | 不等于 | `{pop: {$ne: 100}}` |
| `$in` | 包含 | `{state: {$in: ["MA", "NY"]}}` |
| `$nin` | 不包含 | `{state: {$nin: ["MA", "NY"]}}` |
| `$and` | 逻辑与 | `{$and: [{cond1}, {cond2}]}` |
| `$or` | 逻辑或 | `{$or: [{cond1}, {cond2}]}` |
| `$not` | 逻辑非 | `{pop: {$not: {$gte: 10}}}` |
| `$exists` | 存在性 | `{field: {$exists: true}}` |
| `$set` | 更新设置 | `{$set: {field: value}}` |

### MongoDB vs MySQL 对应关系

| MySQL | MongoDB | 说明 |
|--------|---------|------|
| Database | Database | 数据库 |
| Table | Collection | 表/集合 |
| Row | Document | 行/文档 |
| Column | Field | 列/字段 |
| Index | Index | 索引 |
| Primary Key | _id | 主键 |
| `SELECT * FROM t1` | `db.t1.find()` | 查询所有 |
| `SELECT col FROM t1` | `db.t1.find({}, {col: 1})` | 字段投影 |
| `WHERE id = 1` | `{id: 1}` | 等值条件 |
| `WHERE id > 1` | `{id: {$gt: 1}}` | 比较条件 |
| `WHERE id IN (1,2)` | `{id: {$in: [1,2]}}` | IN条件 |
| `WHERE a=1 AND b=2` | `{a: 1, b: 2}` | AND条件 |
| `WHERE a=1 OR b=2` | `{$or: [{a: 1}, {b: 2}]}` | OR条件 |
| `ORDER BY id ASC` | `.sort({id: 1})` | 升序排序 |
| `ORDER BY id DESC` | `.sort({id: -1})` | 降序排序 |
| `LIMIT 10 OFFSET 20` | `.skip(20).limit(10)` | 分页 |
| `UPDATE SET x=1` | `{$set: {x: 1}}` | 更新操作 |
| `DELETE FROM t1` | `db.t1.deleteMany({})` | 删除操作 |
| `CREATE INDEX` | `db.col.createIndex()` | 创建索引 |

---

## 原文勘误清单

### 1. insertOne()返回字段错误
**位置**: 第87行  
**错误**: `"insertedBy": ObjectId("6076b89de04dc9112613227f")`  
**更正**: 应为 `"insertedId": ObjectId("6076b89de04dc9112613227f")`  
**影响**: 字段名错误，实际返回值是insertedId

### 2. find()输出示例格式错误
**位置**: 第1178行  
**错误**: `{"_id": Algorithm("6077f9e8aec5824bb77a5235")...}`  
**更正**: 应为 `{"_id": ObjectId("6077f9e8aec5824bb77a5235")...}`  
**影响**: 输出格式不正确，Algorithm不是有效的类型

### 3. MongoDB连接字符串示例错误
**位置**: 第526行  
**错误**: `mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb`  
**影响**: 虽然不严格错误，但较复杂，简化版本应为 `mongodb://127.0.0.1:27017`

### 4. dropDatabase()返回值文档不完整
**位置**: 第818行  
**错误**: 显示 `{"dropped": "tmpdb", "ok": 1}` 但实际可能包含更多字段  
**影响**: 返回值示例不完全

### 5. SQL比较表格中操作符符号错误
**位置**: 第2082-2088行  
**错误**: 在查询示例中，操作符显示为 `{("pop":{$lt:10}}` 缺少右括号  
**更正**: 应为 `{"pop": {$lt: 10}}`  
**影响**: JSON格式错误，示例代码无法执行

### 6. 部分查询语句中引号缺失
**位置**: 第2126-2222行  
**错误**: 多处出现 `{"pop":{"lte:0}}` 和 `{"pop":{"gt:...}}` 缺少引号  
**更正**: 应为 `{"pop": {"$lte": 0}}` 和 `{"pop": {"$gt": ...}}`  
**影响**: JSON语法错误，可能导致解析失败

### 7. 复合条件查询示例缺失右括号
**位置**: 第2441行  
**错误**: `db.zips.find({"pop":{"gte:10,$lte:50}})`  
**更正**: 应为 `db.zips.find({"pop": {"$gte": 10, "$lte": 50}})`  
**影响**: JSON格式和操作符都有误

### 8. Controller方法注解错误
**位置**: 第4622行  
**错误**: `@RequestMapping Floorid")` 格式错误  
**更正**: 应为 `@RequestMapping("/{id}")`  
**影响**: 注解语法完全错误，代码无法编译

### 9. findById()方法名错误
**位置**: 第4625行  
**错误**: 调用 `blogDao.findById(id)` 但DAO定义中方法是 `findByID`  
**更正**: 方法名应统一使用驼峰命名法  
**影响**: 方法调用失败，运行时错误

### 10. 分页查询中ID字段引用不一致
**位置**: 第3071-3072行  
**错误**: 先查询 `{"id":{$gt:"01020"}}` 但数据实际用 `"_id"` 字段  
**更正**: 应为 `{"_id": {$gt: "01020"}}`  
**影响**: 查询条件不匹配，无法正确分页

---

## 学习总结

### 核心知识要点

1. **理论基础**: CAP定理和BASE理论是理解NoSQL设计哲学的关键
2. **数据模型**: 文档模型相比关系型更灵活，但也更需要规范
3. **CRUD操作**: insertOne/insertMany、find、updateOne、deleteOne是日常操作
4. **查询优化**: 索引创建和explain()分析是性能优化的重要手段
5. **集成框架**: Spring Data MongoDB简化了Java应用开发

### 常见陷阱

- skip()分页性能差，大数据量使用范围查询分页
- 复合索引需遵循最佳左前缀法则
- ObjectId的时间戳精度为秒级，不能用于毫秒级排序
- 投影中0和1不能混用

### 实战建议

- 建立索引前先用explain()验证查询计划
- 定期检查system.profile发现慢查询
- 使用MongoTemplate而非原生操作
- 合理设计Schema避免频繁修改

---

**文档生成时间**: 2026-05-24  
**基础版本**: MongoDB 4.4.5  
**Java版本**: 适用于 Spring Data MongoDB  
