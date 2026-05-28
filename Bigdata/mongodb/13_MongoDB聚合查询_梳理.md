# MongoDB聚合查询完整梳理

## 一、聚合查询基础概念

### 1.1 什么是聚合查询

聚合操作用于处理多个文档的数据，通过条件分组后执行统计操作（求和、平均值、最大/最小值等），返回单个或多个计算结果。

**核心特点：**
- 将来自多个文档的值组合在一起
- 按条件分组，返回计算后的数据结果
- 类似SQL中的 `count(*)`、`group by` 等操作

### 1.2 聚合查询的两种方式

| 方式 | 特点 | 适用场景 |
|-----|------|---------|
| **Pipeline** | 查询速度快，单阶段100MB内存限制 | 常规数据聚合、报表统计 |
| **MapReduce** | 可在多台服务器并行执行，强大灵活 | 复杂业务逻辑、大规模数据处理 |

### 1.3 聚合管道 (Aggregation Pipeline)

**定义：** 将MongoDB文档通过多个阶段(stage)处理的管道，每个文档经过前一阶段的输出成为下一阶段的输入。

**基本语法：**
```javascript
db.collection.aggregate(pipeline, options)
```

**参数说明：**
- `pipeline`: 数组，包含一系列数据聚合操作或阶段
- `options`: 可选的其他配置项

**重要提示：**
- 直接调用 `db.collection.aggregate()` 会报错
- 调用 `db.collection.aggregate([])` 返回所有文档（类似find）

---

## 二、聚合管道阶段详解

### 2.1 $match 阶段 - 过滤文档

**作用：** 过滤文档，仅将符合指定条件的文档传递到下一阶段

**语法：**
```javascript
{$match: { <query> }}
```

**特点：**
- 接受查询条件文档，查询语法与读操作相同
- 除地理空间外，支持所有常规查询操作符
- 可使用除 `$` 外的所有聚合操作符

**性能优化：**
- 尽可能将 `$match` 放在管道前面位置
- 好处1：快速过滤不需要的文档，减少后续处理工作量
- 好处2：在投射和分组前执行可使用索引

**限制：**
- 不能在 `$match` 中使用 `$` 作为聚合管道的一部分
- 要使用 `$text`，`$match` 必须是管道的第一阶段
- 视图不支持文本搜索

**示例：**
```javascript
db.zips.aggregate([
    {
        "$match": {
            "state": "NY"
        }
    }
])
```

### 2.2 $group 阶段 - 分组统计

**作用：** 按指定的表达式对文档进行分组，输出每个不同分组的统计信息

**语法：**
```javascript
{ 
    $group: { 
        _id: <expression>,
        <field1>: { <accumulator1>: <expression1> },
        ...
    } 
}
```

**核心特点：**
- `_id` 字段必填（可设为 null 为整个输入计算累计值）
- 输出文档包含 `_id` 字段和计算字段
- 不输出具体文档，只输出统计信息
- 内存限制：100MB（可设 `allowDiskUse: true` 使用临时文件）

**Accumulator 操作符汇总：**

| 操作符 | 描述 | SQL类比 | 示例 |
|-------|------|--------|------|
| `$sum` | 计算总和 | sum() | `{$sum: "$pop"}` |
| `$avg` | 计算平均值 | avg() | `{$avg: "$pop"}` |
| `$min` | 获取最小值 | min() | `{$min: "$pop"}` |
| `$max` | 获取最大值 | max() | `{$max: "$pop"}` |
| `$first` | 返回首个文档 | limit 0,1 | `{$first: "$field"}` |
| `$last` | 返回最后文档 | - | `{$last: "$field"}` |
| `$push` | 元素添加到数组 | - | `{$push: "$field"}` |
| `$addToSet` | 无重复添加到集合 | - | `{$addToSet: "$field"}` |
| `$stdDevPop` | 总体标准偏差 | - | `{$stdDevPop: "$field"}` |
| `$stdDevSamp` | 样本标准偏差 | - | `{$stdDevSamp: "$field"}` |

**$push 与 $addToSet 区别：**
- `$push`: 无条件添加，包含重复值
- `$addToSet`: 只在不存在时添加，不含重复值（无序）

**示例1 - 按州分组统计：**
```javascript
db.zips.aggregate([
    {
        "$group": {
            "_id": "$state",
            "totalPop": { "$sum": "$pop" },
            "avgPop": { "$avg": "$pop" },
            "count": { "$sum": 1 }
        }
    }
])
```

**示例2 - 按城市分组，收集state列表：**
```javascript
db.zips.aggregate([
    {
        "$group": {
            "_id": "$city",
            "stateItem": { "$push": "$state" }
        }
    }
])
```

**示例3 - 使用系统变量 $$ROOT：**
```javascript
db.zips.aggregate([
    {
        "$group": {
            "_id": "$city",
            "item": { "$push": "$$ROOT" }
        }
    }
]).pretty()
```

### 2.3 $project 阶段 - 字段投射

**作用：** 从文档中选择想要的字段，可添加新计算字段，支持数学、日期、字符串、逻辑操作

**语法：**
```javascript
{ $project: { <specification> } }
```

**规范说明：**
- `<field>: 1 or true` - 包含该字段
- `<field>: 0 or false` - 排除该字段
- `<field>: <expression>` - 添加新字段或重置现有字段值

**特殊处理：**
- `_id` 字段默认包含，需排除时设 `_id: 0`
- v3.6+：支持 `$$REMOVE` 变量条件排除字段
- v3.2+：支持用方括号 `[]` 直接创建数组字段
- 嵌入文档可用点符号 `.` 访问

**重要规则：**
- 默认 `_id` 包含在输出中
- 要包含输入文档的其他字段，必须明确指定
- 指定包含文档中不存在的字段，会被忽略
- v3.4+：排除字段后，所有其他字段自动返回
  - 若排除非 `_id` 字段，不能同时指定包含/重置/添加新字段
  - 除非使用 `$$REMOVE` 条件排除

**示例1 - 选择特定字段：**
```javascript
db.zips.aggregate([
    {
        "$project": {
            "_id": 1,
            "city": 1,
            "state": 1
        }
    }
])
```

**示例2 - 排除 _id 字段：**
```javascript
db.zips.aggregate([
    {
        "$project": {
            "_id": 0,
            "city": 1,
            "state": 1
        }
    }
])
```

**示例3 - 排除特定字段：**
```javascript
db.zips.aggregate([
    {
        "$project": {
            "loc": 0
        }
    }
])
```

**示例4 - 条件排除字段（v3.6+）：**
```javascript
db.zips.aggregate([
    {
        "$project": {
            "_id": 1,
            "city": 1,
            "pop": 1,
            "loc": {
                "$cond": {
                    "if": { "$gt": ["$pop", 1000] },
                    "then": "$$REMOVE",
                    "else": "$loc"
                }
            }
        }
    }
]).pretty()
```

**示例5 - 条件修改字段值：**
```javascript
db.zips.aggregate([
    {
        "$project": {
            "_id": 1,
            "city": 1,
            "pop": 1,
            "loc": {
                "$cond": {
                    "if": { "$gt": ["$pop", 1000] },
                    "then": [0, 0],
                    "else": "$loc"
                }
            }
        }
    }
])
```

**示例6 - 添加新计算字段：**
```javascript
db.zips.aggregate([
    {
        "$project": {
            "_id": 1,
            "city": 1,
            "pop": 1,
            "desc": {
                "$cond": {
                    "if": { "$gt": ["$pop", 1000] },
                    "then": "人数过多",
                    "else": "人数过少"
                }
            }
        }
    }
])
```

### 2.4 $unwind 阶段 - 数组展开

**作用：** 从输入文档解构数组字段，为数组的每个元素输出一个文档

**语法 v3.2前：**
```javascript
{$unwind: "<field path>"}
```

**语法 v3.2+（增强）：**
```javascript
{
    $unwind: {
        path: "<field path>",
        includeArrayIndex: "<string>",    // 可选，存储数组索引
        preserveNullAndEmptyArrays: <boolean>  // 可选，默认false
    }
}
```

**参数说明：**
- `path`: 字段路径，必须以 `$` 开头（如 `"$loc"`）
- `includeArrayIndex`: 新字段名存储元素的数组索引，不能以 `$` 开头
- `preserveNullAndEmptyArrays`: 
  - `false`（默认）：忽略缺失、null或空数组的文档
  - `true`：输出这些文档

**示例1 - 基础展开：**
```javascript
db.zips.aggregate([
    {
        "$match": { "_id": "01002" }
    },
    {
        "$unwind": "$loc"
    }
])
```

**示例2 - 保留数组索引和空数组：**
```javascript
db.zips.aggregate([
    {
        "$match": { "_id": "01002" }
    },
    {
        "$unwind": {
            "path": "$loc",
            "includeArrayIndex": "locIndex",
            "preserveNullAndEmptyArrays": true
        }
    }
]).pretty()
```

### 2.5 $sort 阶段 - 排序

**作用：** 对所有输入文档进行排序，按排序顺序返回到管道

**语法：**
```javascript
{ $sort: { <field1>: <sort order>, <field2>: <sort order> ... } }
```

**排序顺序值：**
- `1` - 升序
- `-1` - 降序
- `{$meta: "textScore"}` - 按文本搜索评分降序排列

**示例 - 多字段排序：**
```javascript
db.zips.aggregate([
    {
        "$sort": {
            "pop": -1,    // 人口数降序
            "city": 1     // 城市名升序
        }
    }
])
```

**性能优化：**
当 `$sort` 紧跟在 `$limit` 前时，MongoDB只会在内存中维持前 n 个结果。即使 `allowDiskUse: true`，这个优化仍然适用。

### 2.6 $limit 和 $skip 阶段

#### $limit 阶段
**作用：** 限制传递到下一阶段的文档数

**语法：**
```javascript
{ $limit: <positive integer> }
```

**示例：**
```javascript
db.zips.aggregate([
    { "$limit": 5 }
])
```

#### $skip 阶段
**作用：** 跳过指定数量的文档，将剩余文档传递到下一阶段

**语法：**
```javascript
{$skip: <positive integer>}
```

**示例：**
```javascript
db.zips.aggregate([
    { "$skip": 5 }
])
```

**分页最佳实践：**
```javascript
// 分页查询：跳过 (page-1)*pageSize 条，取 pageSize 条
db.collection.aggregate([
    { "$skip": (page - 1) * pageSize },
    { "$limit": pageSize }
])
```

### 2.7 $count 阶段 - 计数

**作用：** 返回输入到该阶段的文档计数

**语法：**
```javascript
{$count: "<string>"}
```

**等价于：**
```javascript
db.zips.aggregate([
    {
        "$group": {
            "_id": null,
            "count": { "$sum": 1 }
        }
    },
    {
        "$project": {
            "_id": 0
        }
    }
])
```

**示例 - 查询人口>100000的城市数量：**
```javascript
db.zips.aggregate([
    {
        "$match": {
            "pop": { "$gt": 100000 }
        }
    },
    {
        "$count": "count"
    }
])
```

### 2.8 $sortByCount 阶段

**作用：** 按表达式分组统计文档数，结果按降序排列

**语法：**
```javascript
{ $sortByCount: <expression> }
```

**特点：**
- 输出包含 `_id`（分组值）和 `count`（文档数）
- 自动按count降序排列

---

## 三、Pipeline 与 MySQL 聚合对比

| SQL 操作 | MongoDB Pipeline | 说明 |
|---------|-----------------|------|
| `WHERE` | `$match` | 过滤条件 |
| `GROUP BY` | `$group` | 分组 |
| `HAVING` | `$match` | 分组后筛选 |
| `SELECT` | `$project` | 字段选择 |
| `ORDER BY` | `$sort` | 排序 |
| `LIMIT` | `$limit` | 限制行数 |
| `SUM()` | `$sum` | 求和 |
| `COUNT()` | `$sum: 1` | 计数 |
| `JOIN` | `$lookup` | 关联查询 |

---

## 四、Pipeline 常用聚合示例

### 4.1 统计所有数据

**SQL等价：**
```sql
SELECT count(1) FROM zips;
```

**MongoDB Pipeline：**
```javascript
db.zips.aggregate([
    {
        "$group": {
            "_id": null,
            "count": { "$sum": 1 }
        }
    }
])
```

### 4.2 对所有城市人数求合

**SQL等价：**
```sql
SELECT sum(pop) AS total FROM zips;
```

**MongoDB Pipeline：**
```javascript
db.zips.aggregate([
    {
        "$group": {
            "_id": null,
            "total": { "$sum": "$pop" }
        }
    }
])
```

### 4.3 按州分组统计人数

**SQL等价：**
```sql
SELECT state, sum(pop) AS total FROM zips GROUP BY state;
```

**MongoDB Pipeline：**
```javascript
db.zips.aggregate([
    {
        "$group": {
            "_id": "$state",
            "total": { "$sum": "$pop" }
        }
    }
])
```

### 4.4 分组计数大于100的州

**SQL等价：**
```sql
SELECT state, count(1) AS total FROM zips GROUP BY state HAVING count(1) > 100;
```

**MongoDB Pipeline：**
```javascript
db.zips.aggregate([
    {
        "$group": {
            "_id": "$state",
            "total": { "$sum": 1 }
        }
    },
    {
        "$match": {
            "total": { "$gt": 100 }
        }
    }
])
```

### 4.5 复杂聚合 - $match + $group

**SQL等价：**
```sql
SELECT cust_id, sum(amount) AS total FROM orders WHERE status = 'A' GROUP BY cust_id;
```

**MongoDB Pipeline：**
```javascript
db.orders.aggregate([
    {
        "$match": { "status": "A" }
    },
    {
        "$group": {
            "_id": "$cust_id",
            "total": { "$sum": "$amount" }
        }
    }
])
```

---

## 五、Pipeline 数据流与性能

### 5.1 聚合管道处理流程

```
输入文档集合
    ↓
[$match 阶段] 过滤文档
    ↓
[$project 阶段] 字段投射
    ↓
[$group 阶段] 分组统计
    ↓
[$sort 阶段] 排序
    ↓
[$limit 阶段] 限制数量
    ↓
输出聚合结果
```

### 5.2 关键性能特征

**内存限制：**
- 单个聚合管道阶段限制：100MB
- 超过限制时产生错误
- 解决方案：设置 `allowDiskUse: true` 使用临时文件

**优化策略：**
1. 尽早使用 `$match` 减少后续处理文档数
2. 在投射和分组前执行 `$match` 可使用索引
3. `$sort` + `$limit` 组合时，只需维持前n个结果在内存
4. 选择合适的 `_id` 值减少分组工作量

**输出限制：**
- 返回值限制为单个文档：16MB（BSON Document限制）
- 解决方案：使用指针返回（v2.6+）

**分片集合支持：**
- Pipeline 可运行在分片集合，但结果不能输出到分片集合
- MapReduce 可输入和输出都在分片集合

---

## 六、MapReduce 详解

### 6.1 MapReduce 基本概念

**定义：** 将大批量工作（数据）分解(MAP)执行，再将结果合并成最终结果(REDUCE)的计算模型

**特点：**
- 使用JavaScript语法编写
- 基于V8引擎解析执行
- 可处理比Pipeline更复杂的业务逻辑
- 支持多服务器并行执行
- 代码量多、调试困难

**对比Pipeline：**
- Pipeline：查询速度快，100MB内存限制
- MapReduce：灵活强大，支持多服务器并行，内存消耗更大

### 6.2 MapReduce 执行阶段

**三个核心阶段：**

1. **Map 阶段**
   - 处理每个文档
   - Emit 一个或多个 key-value 对

2. **Combine/Shuffle 阶段**
   - 按key分组
   - 相同key的value集合到一起

3. **Reduce 阶段**
   - 对Map结果进行聚合操作
   - 将 key-values 合并为 key-value

4. **Finalize 阶段（可选）**
   - 对Reduce结果进行最后调整

### 6.3 MapReduce 语法

```javascript
db.collection.mapReduce(
    function() {
        // map 函数
        // this 代表当前文档
        emit(key, value);
    },
    function(key, values) {
        // reduce 函数
        // values 是数组，需要聚合为单个值
        return reduceFunction
    },
    {
        // 配置选项
        out: collection,
        query: document,
        sort: document,
        limit: number,
        finalize: <function>,
        scope: <document>,
        jsMode: <boolean>,
        verbose: <boolean>
    }
)
```

### 6.4 Map 函数详解

**职责：** 处理每个文档，emit key-value 对用于分组

**特点：**
- 一次调用可多次emit（也可不emit）
- 无需返回值
- 输入为当前document（可用 `this.<fieldName>` 访问）
- 是闭包函数，不能访问外部资源（collection、database）
- 可访问 `scope` 中的变量
- emit值不能超过16MB（BSON文档大小限制）

**示例 - 多次emit：**
```javascript
function() {
    this.items.forEach(function(item) {
        emit(item.sku, 1);
    });
}
```

### 6.5 Reduce 函数详解

**职责：** 对Map结果进行聚合，将 key-values 合并为 key-value

**特点：**
- 接收 `key` 和 `values` 两个参数
- `values` 为分组后的数组
- 一个key可能调用多次reduce（前次结果可能作为input）
- 返回结果结构需与value一致（幂等性）
- 返回值不超过8MB（document最大尺寸一半）
- 是闭包函数，不能访问外部资源
- 可访问 `scope` 中的变量

**重要原则：**
- 算法需要是幂等的（可多次应用得相同结果）
- 与input values的顺序无关

**示例：**
```javascript
// mapper
function() {
    emit(this.categoryId, {count: 1});
}

// reducer
function(key, values) {
    var current = {count: 0};
    values.forEach(function(item) {
        current.count += item.count;
    });
    return current;
}
```

### 6.6 出参数 (out) 详解

**作用：** 指定reduce结果的保存方式

**语法：**
```javascript
out: {
    <action>: <collectionName>,
    [db: <dbName>],
    [sharded: <boolean>],
    [nonAtomic: <boolean>]
}
```

**Action 类型：**

| Action | 说明 | 场景 |
|--------|------|------|
| `replace` | 替换集合内容（先临时存储，再rename） | 覆盖旧数据 |
| `merge` | 合并结果，相同_id覆盖原值 | 增量更新 |
| `reduce` | 合并后再次调用reduce，适合增量统计 | 用户留存、增量统计 |
| `inline` | 内存输出，返回cursor | 结果较小，无需持久化 |

**其他参数：**
- `db`: 结果保存的database（默认当前db）
- `sharded`: 输出集合使用sharding模式，以_id为shard key
- `nonAtomic`: 仅对merge/replace有效
  - `false`（默认）：对output集合加锁，原子性操作
  - `true`：不加锁，允许其他客户端读取中间状态数据

**Inline 输出示例：**
```javascript
out: { inline: 1 }
```

### 6.7 其他配置参数

**query：**
- 筛选条件，只有满足条件的文档才会调用map函数
- 类似Pipeline中的 `$match`

**sort：**
- 对筛选后的文档排序，再传递给mapper
- 可优化分组机制（减少reduce次数）
- 排序必须能使用索引

**limit：**
- 限定输入到map的文档数量上限

**finalize：**
- 在输出前调整reduce结果的JavaScript函数
- 接收 `key` 和 `value` 参数
- 可修改value作为最终输出

**示例：**
```javascript
function(key, value) {
    var final = {count: 0, key: ""};
    final.key = key;
    return final;
}
```

**scope：**
- Document结构，保存全局变量
- 可在map、reduce、finalize中访问

**jsMode：**
- `false`（默认）：mapper输出转换为BSON再存储临时文件，reduce时再转回JavaScript对象
  - 优点：支持大数据集
  - 缺点：转换性能消耗
- `true`：不进行类型转换，数据暂存内存
  - 优点：性能高
  - 缺点：key个数不能超过50W，内存消耗大
  - 建议：生产环境设为false

**verbose：**
- 是否在结果中包含时间信息（默认false）

### 6.8 MapReduce 示例

**场景：** 按州统计人口超过100的文档数

**SQL等价：**
```sql
SELECT state, count(1) FROM zips WHERE pop > 100 GROUP BY state;
```

**MapReduce实现：**
```javascript
db.zips.mapReduce(
    function() {
        emit(this.state, 1);
    },
    function(key, values) {
        return Array.sum(values);
    },
    {
        query: {pop: {$gt: 100}},
        out: "result001"
    }
)
```

**查看结果：**
```javascript
db.result001.find({})
```

---

## 七、聚合性能优化

### 7.1 Pipeline 性能优化

**优化原则：**
1. **尽早过滤** - 将 `$match` 放在管道前面
2. **索引利用** - 在投射和分组前执行 `$match` 可使用索引
3. **内存管理** - 超过100MB使用 `allowDiskUse: true`
4. **减少输出** - `$limit` 和 `$skip` 减少处理文档数

**最佳实践：**
```javascript
db.orders.aggregate([
    // 1. 尽早过滤
    { "$match": { "status": "A" } },
    
    // 2. 投射必要字段
    { "$project": { "cust_id": 1, "amount": 1 } },
    
    // 3. 分组统计
    { "$group": {
        "_id": "$cust_id",
        "total": { "$sum": "$amount" }
    }},
    
    // 4. 排序和限制
    { "$sort": { "total": -1 } },
    { "$limit": 10 }
])
```

### 7.2 MapReduce 性能优化

**优化策略：**
1. **Pre-filtering** - 使用 `query` 参数预过滤文档
2. **Pre-sorting** - 使用 `sort` 可减少reduce调用次数
3. **Limit** - 限制输入文档数量
4. **jsMode** - 权衡内存和速度（生产环境用false）

**监控指标：**
- 单阶段内存占用
- 排序和分组操作次数
- 中间结果大小
- 磁盘I/O次数（allowDiskUse时）

---

## 八、API 语法速查

### 8.1 聚合操作基础语法

```javascript
// 基础调用
db.collection.aggregate(pipeline, options)

// 带选项
db.collection.aggregate(
    [
        { $stage1: {...} },
        { $stage2: {...} }
    ],
    {
        allowDiskUse: true,
        batchSize: 1000,
        maxTimeMS: 30000
    }
)
```

### 8.2 常用聚合表达式

```javascript
// 算术表达式
{ $add: [expr1, expr2, ...] }
{ $subtract: [expr1, expr2] }
{ $multiply: [expr1, expr2, ...] }
{ $divide: [expr1, expr2] }

// 比较表达式
{ $eq: [expr1, expr2] }
{ $gt: [expr1, expr2] }
{ $gte: [expr1, expr2] }
{ $lt: [expr1, expr2] }
{ $lte: [expr1, expr2] }

// 逻辑表达式
{ $and: [expr1, expr2, ...] }
{ $or: [expr1, expr2, ...] }
{ $not: expr }

// 条件表达式
{ $cond: {
    if: <boolean-expr>,
    then: <true-expr>,
    else: <false-expr>
}}
```

---

## 九、⚠️ 原文勘误统计

### 勘误1：数据显示错误
**位置：** 第718-727行 $project 示例
**原文：**
```
...    "id":0,     // 错误！应为 "_id"
```
**勘误：** 应为 `"_id": 0`，不是 `"id": 0`

**影响：** 无法正确排除_id字段

---

### 勘误2：语法错误
**位置：** 第787行、792行 $project 输出示例
**原文：**
```json
"id": "01028"     // 错误！应为 "_id"
"id": "01030"     // 错误！应为 "_id"
```
**勘误：** MongoDB中主键字段名固定为 `_id`，不是 `id`

**影响：** 代码执行结果展示错误

---

### 勘误3：引号错配
**位置：** 第917行 $project 中 $cond 示例
**原文：**
```javascript
if: { $gt:[$pop",1000]},    // 错误！$pop后多一个引号
```
**勘误：** 应为 `if: { $gt: ["$pop", 1000] }`

**影响：** 代码语法错误，无法执行

---

### 勘误4：同名参数混淆
**位置：** 第1330行 MapReduce 结果展示
**原文：**
```json
"_id": "RI", "total": 1003218     // 应检查是否为"RI"
```
**说明：** 在第1257行中，同一结果集曾显示 `"RT"`，第1330行为 `"RI"`，两者不一致

**影响：** 示例数据可能有复制错误

---

### 勘误5：字段大小写不一致
**位置：** 第999行、1000行、1006行 $project 示例
**原文：**
```javascript
"disc": {           // 错误！应为 "desc"
```
**勘误：** 应为 `"desc"` 而非 `"disc"`（后续解释中使用的是"desc"）

**影响：** 字段命名与文档描述不符

---

### 勘误6：数组索引错误
**位置：** 第600-603行 $unwind 输出示例
**原文：**
```json
"loc": -72.51565,    // 错误！数组元素不应分开显示
"loc": 42.377017,    // 应该展开为两个文档
```
**勘误：** $unwind 应为每个数组元素生成一个单独的文档，而非显示单个数值

**影响：** 理解 $unwind 行为可能混淆

---

## 十、核心知识点速查表

### 常用Stage速查

| Stage | 功能 | 内存限制 | 是否改变文档结构 |
|-------|------|---------|-----------------|
| `$match` | 过滤 | 无 | 否 |
| `$group` | 分组统计 | 100MB | 是 |
| `$project` | 字段投射 | 无 | 是 |
| `$unwind` | 数组展开 | 无 | 是 |
| `$sort` | 排序 | 受限 | 否 |
| `$limit` | 限制数量 | 无 | 否 |
| `$skip` | 跳过数量 | 无 | 否 |
| `$count` | 计数 | 无 | 是 |

### Accumulator 操作速查

```javascript
$sum: 1           // 计数
$sum: "$field"    // 求和
$avg: "$field"    // 平均值
$min: "$field"    // 最小值
$max: "$field"    // 最大值
$first: "$field"  // 首个值
$last: "$field"   // 最后值
$push: "$field"   // 元素入数组（有重复）
$addToSet: "$field"  // 元素入集合（无重复）
```

### Pipeline 处理流程图

```
输入集合
  ↓
$match [过滤] → 减少处理文档
  ↓
$project [投射] → 精简字段
  ↓
$group [分组] → 聚合统计（100MB限制）
  ↓
$sort [排序] → 结果排序
  ↓
$limit [限制] → 控制输出
  ↓
$skip [跳过] → 分页处理
  ↓
输出结果
```

### 性能优化检查清单

- [ ] $match 放在管道前面？
- [ ] 在分组前执行 $match（可使用索引）？
- [ ] 使用 $project 减少字段量？
- [ ] 大数据集设置 allowDiskUse: true？
- [ ] $sort + $limit 组合？
- [ ] 合理设置 limit 和 skip？
- [ ] MapReduce 使用 query 预过滤？
- [ ] 监控管道内存占用？

### 常见错误预防

- **错1：** 直接调用 `aggregate()` 不加参数 → 必须传数组 `aggregate([])`
- **错2：** 在 $match 中使用 `$` 变量 → 只有后续stage可用
- **错3：** $group 超过100MB → 使用 `allowDiskUse: true`
- **错4：** $project 同时包含/排除字段 → 排除后不能再包含
- **错5：** $unwind 空数组被忽略 → 使用 `preserveNullAndEmptyArrays: true`
- **错6：** MapReduce reduce 不幂等 → 确保算法顺序无关

---

## 十一、扩展阅读建议

**推荐学习路径：**
1. 掌握基础Pipeline（$match, $group, $project）
2. 学习优化策略（提前过滤、索引利用）
3. 理解复杂场景（多stage组合）
4. 对比SQL理解概念
5. 性能测试和调优

**进阶主题：**
- 地理查询聚合
- 文本搜索聚合
- 时间序列聚合
- 分片集合聚合特殊考虑
- Transactions与聚合组合

