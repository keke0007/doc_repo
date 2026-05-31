# SparkSQL 知识点总结

> 来源：`SparkSQL .md`  
> 目标：把 SparkSQL 的核心知识整理成一份便于复习、面试回顾和建立知识框架的学习笔记。

## 1. SparkSQL 基础

### 1.1 SparkSQL 是什么

SparkSQL 是 Spark 基于 Spark Core 提供的结构化数据处理模块。

它的核心作用：

- 处理结构化和半结构化数据。
- 提供 DataFrame、Dataset 等高级数据抽象。
- 支持 SQL、DSL API、HiveQL 等多种使用方式。
- 把 SQL 或 DataFrame/Dataset 操作转换成底层 RDD 执行计划。

可以简单理解：

```text
RDD = 计算逻辑 + 分区信息 + 数据位置信息
DataFrame = RDD + Schema
Dataset = 强类型 DataFrame
DataFrame = Dataset[Row]
```

### 1.2 SparkSQL 的核心价值

| 能力 | 说明 |
|---|---|
| 结构化计算 | 基于 Schema 描述数据结构，比 RDD 更适合表格型数据 |
| SQL 支持 | 可以直接写 SQL 完成查询、过滤、聚合、Join 等操作 |
| API 统一 | 同一套 API 可以读取 CSV、JSON、Parquet、ORC、JDBC、Hive 等数据源 |
| 性能优化 | 内置 Catalyst 优化器和 Tungsten 执行引擎 |
| Hive 兼容 | 支持 HiveQL、Hive 表、Hive 元数据、SerDe、UDF |
| BI 对接 | 通过 ThriftServer 提供 JDBC/ODBC 连接能力 |

### 1.3 SparkSQL 与 SparkCore 的关系

SparkSQL 不是脱离 SparkCore 单独运行的系统，而是建立在 SparkCore 之上的高级模块。

执行链路可以概括为：

```text
SQL / DataFrame / Dataset
        ↓
逻辑计划
        ↓
优化后的逻辑计划
        ↓
物理计划
        ↓
RDD 执行
```

面试要点：

- SparkSQL 面向结构化数据，SparkCore 面向更底层的 RDD 编程。
- SparkSQL 可以自动利用 Schema 做优化，RDD 需要开发者自己组织计算逻辑。
- DataFrame/Dataset 最终仍会转成底层 RDD 任务执行。

## 2. SparkSession

### 2.1 SparkSession 是什么

SparkSession 是 Spark 2.x 之后统一的编程入口。

它整合了：

- SparkContext
- SQLContext
- HiveContext

常见创建方式：

```scala
val spark = SparkSession.builder()
  .appName("spark sql app")
  .master("local[*]")
  .getOrCreate()
```

常用对象：

| 对象 | 作用 |
|---|---|
| `spark` | SparkSession，SparkSQL 主入口 |
| `spark.sparkContext` | 获取底层 SparkContext |
| `spark.read` | 读取外部数据，得到 DataFrame |
| `spark.sql(...)` | 执行 SQL 语句 |
| `spark.udf` | 注册 UDF 函数 |

### 2.2 IDEA 开发依赖

SparkSQL 常见 Maven 依赖：

```xml
<dependency>
  <groupId>org.apache.spark</groupId>
  <artifactId>spark-sql_2.12</artifactId>
  <version>3.1.3</version>
</dependency>
```

如果需要整合 Hive，通常还需要加入 Hive 相关依赖，并启用 Hive 支持：

```scala
val spark = SparkSession.builder()
  .appName("spark hive")
  .master("local[*]")
  .enableHiveSupport()
  .getOrCreate()
```

## 3. DataFrame 与 Dataset

### 3.1 DataFrame

DataFrame 是带有 Schema 的分布式数据集，可以理解为一张分布式表。

特点：

- 每一行数据类型是 `Row`。
- 每一列有列名和数据类型。
- 支持 SQL 查询和 DSL 操作。
- 不强调编译期类型安全。

示例：

```scala
val df = spark.read
  .option("header", "true")
  .option("inferSchema", "true")
  .csv("data/users")

df.printSchema()
df.show()
```

### 3.2 Dataset

Dataset 是强类型分布式数据集。

特点：

- 有明确的泛型类型，例如 `Dataset[User]`。
- 编译期能检查字段和类型。
- 既支持函数式 API，也可以注册成临时视图执行 SQL。
- Scala 中通常结合 case class 使用。

示例：

```scala
case class User(id: Int, name: String, age: Int, gender: String)

import spark.implicits._

val ds = Seq(
  User(1, "tom", 20, "M"),
  User(2, "lucy", 18, "F")
).toDS()
```

### 3.3 DataFrame、Dataset、RDD 对比

| 对比项 | RDD | DataFrame | Dataset |
|---|---|---|---|
| 数据结构 | 无 Schema | 有 Schema | 有 Schema + 泛型类型 |
| 类型安全 | 编译期安全 | 运行期检查 | 编译期安全 |
| API 风格 | 函数式算子 | SQL + DSL | SQL + DSL + 强类型算子 |
| 优化能力 | 优化较少 | Catalyst 优化 | Catalyst 优化 |
| 使用场景 | 非结构化、复杂底层逻辑 | 结构化数据分析 | 结构化且需要类型安全 |

记忆方式：

```text
RDD：底层、灵活、类型安全，但优化少
DataFrame：表结构、SQL 友好、优化强，但 Row 弱类型
Dataset：DataFrame 的强类型版本
```

## 4. 创建 DataFrame

### 4.1 从文件创建

SparkSQL 支持多种结构化文件源。

| 数据源 | 读取方式 |
|---|---|
| CSV | `spark.read.csv(path)` |
| JSON | `spark.read.json(path)` |
| Parquet | `spark.read.parquet(path)` |
| ORC | `spark.read.orc(path)` |
| Text | `spark.read.text(path)` |

CSV 常用配置：

```scala
val df = spark.read
  .option("header", "true")
  .option("inferSchema", "true")
  .csv("data/users")
```

配置说明：

| 配置 | 作用 |
|---|---|
| `header` | 是否把第一行当作列名 |
| `inferSchema` | 是否自动推断字段类型 |
| `sep` | 指定分隔符 |
| `encoding` | 指定编码 |

注意：

- 不指定 `header=true` 时，列名通常是 `_c0`、`_c1`、`_c2`。
- 不指定 `inferSchema=true` 时，CSV 字段默认可能被读取成字符串。
- 生产中更推荐手动指定 Schema，减少推断开销和类型误判。

### 4.2 手动指定 Schema

常见写法：

```scala
import org.apache.spark.sql.types._

val schema = StructType(Seq(
  StructField("id", IntegerType, nullable = true),
  StructField("name", StringType, nullable = true),
  StructField("age", IntegerType, nullable = true),
  StructField("gender", StringType, nullable = true)
))

val df = spark.read
  .schema(schema)
  .csv("data/users")
```

核心类型：

| 类型 | 说明 |
|---|---|
| `StructType` | 整体表结构 |
| `StructField` | 单个字段定义 |
| `StringType` | 字符串类型 |
| `IntegerType` | 整数类型 |
| `DoubleType` | 双精度类型 |
| `DateType` | 日期类型 |
| `TimestampType` | 时间戳类型 |

### 4.3 从 RDD 创建

常见方式：

```scala
import spark.implicits._

val rdd = spark.sparkContext.textFile("data/users")

val df = rdd.map(line => {
  val arr = line.split(",")
  (arr(0).toInt, arr(1), arr(2).toInt, arr(3))
}).toDF("id", "name", "age", "gender")
```

也可以通过 case class 创建：

```scala
case class User(id: Int, name: String, age: Int, gender: String)

val df = rdd.map(line => {
  val arr = line.split(",")
  User(arr(0).toInt, arr(1), arr(2).toInt, arr(3))
}).toDF()
```

### 4.4 从外部系统创建

| 外部系统 | 创建方式 |
|---|---|
| JDBC | `spark.read.format("jdbc")...load()` |
| Hive | `spark.sql("select ... from hive_table")` |
| HBase | 通常通过 Spark-HBase Connector 或自定义读取 |

JDBC 示例：

```scala
val df = spark.read
  .format("jdbc")
  .option("url", "jdbc:mysql://host:3306/db")
  .option("dbtable", "tb_user")
  .option("user", "root")
  .option("password", "123456")
  .load()
```

## 5. DataFrame 输出

### 5.1 控制台输出

```scala
df.show()
df.show(20)
df.show(false)
df.printSchema()
```

说明：

- `show()` 默认展示 20 行。
- `show(false)` 表示字段内容不截断。
- `printSchema()` 查看表结构。

### 5.2 保存到文件

```scala
df.write
  .mode("overwrite")
  .option("header", "true")
  .csv("output/users")
```

常见保存格式：

```scala
df.write.json(path)
df.write.parquet(path)
df.write.orc(path)
df.write.csv(path)
```

写入模式：

| 模式 | 说明 |
|---|---|
| `append` | 追加写入 |
| `overwrite` | 覆盖已有数据 |
| `ignore` | 路径存在时忽略写入 |
| `error` / `errorifexists` | 路径存在时报错，默认行为 |

### 5.3 保存到 JDBC

```scala
df.write
  .format("jdbc")
  .option("url", "jdbc:mysql://host:3306/db")
  .option("dbtable", "tb_result")
  .option("user", "root")
  .option("password", "123456")
  .mode("append")
  .save()
```

### 5.4 写入 Hive

```scala
df.write
  .mode("overwrite")
  .saveAsTable("db.table_name")
```

使用前提：

- 创建 SparkSession 时启用 `enableHiveSupport()`。
- 配置好 Hive metastore。
- 有对应 Hive 依赖和 `hive-site.xml`。

### 5.5 分区输出

```scala
df.write
  .mode("overwrite")
  .partitionBy("year", "month")
  .parquet("output/orders")
```

分区输出适合按日期、地区、业务类型等字段组织数据。

注意：

- 分区字段基数不能过高，否则会产生大量小文件。
- 分区字段通常是查询过滤中高频使用的字段。
- 写入前可以通过 `repartition` 或 `coalesce` 控制输出文件数量。

## 6. DataFrame 运算

### 6.1 SQL 风格

先注册临时视图：

```scala
df.createOrReplaceTempView("tb_user")
```

再执行 SQL：

```scala
val result = spark.sql(
  """
    |select gender, avg(age) as avg_age
    |from tb_user
    |group by gender
    |""".stripMargin)
```

常用 SQL 能力：

- `select`
- `where`
- `group by`
- `order by`
- `join`
- `union`
- 子查询
- 窗口函数

### 6.2 DSL 风格

需要导入隐式转换：

```scala
import spark.implicits._
import org.apache.spark.sql.functions._
```

常见操作：

```scala
df.select($"name", $"age")
df.select(col("name"), col("age") + 1)
df.filter($"age" > 30)
df.where("age > 30")
df.groupBy("gender").agg(avg("age").as("avg_age"))
df.orderBy($"age".desc)
df.withColumnRenamed("name", "username")
df.withColumn("new_age", $"age" + 1)
df.drop("gender")
```

### 6.3 SQL 与 DSL 对比

| 对比项 | SQL | DSL |
|---|---|---|
| 表达方式 | 接近数据库 SQL | 接近 Spark API |
| 可读性 | 对 SQL 熟悉者友好 | 对 Scala/Java 开发者友好 |
| 动态拼接 | 方便但要注意 SQL 注入 | 类型和结构更清晰 |
| 复杂逻辑 | 适合复杂查询 | 适合和程序逻辑混写 |

实际开发中两者经常混用。

## 7. Join、Union、窗口函数

### 7.1 Join

常见写法：

```scala
val joined = df1.join(df2, Seq("id"), "inner")
```

常见 Join 类型：

| 类型 | 说明 |
|---|---|
| `inner` | 内连接，只保留两边都匹配的数据 |
| `left` / `left_outer` | 左外连接，保留左表全部数据 |
| `right` / `right_outer` | 右外连接，保留右表全部数据 |
| `full` / `full_outer` | 全外连接，保留两边全部数据 |
| `left_semi` | 只保留左表中能匹配右表的行 |
| `left_anti` | 只保留左表中不能匹配右表的行 |

注意：

- Join 容易产生 Shuffle，是 SparkSQL 性能优化重点。
- 小表 Join 大表时可以考虑广播 Join。
- Join 字段类型必须一致，否则可能匹配失败或触发额外类型转换。

广播 Join 示例：

```scala
df1.join(broadcast(df2), Seq("id"), "inner")
```

### 7.2 Union

```scala
val all = df1.union(df2)
```

注意：

- `union` 要求两边字段数量和字段类型兼容。
- 默认按位置合并，不按字段名匹配。
- 如果要按字段名合并，可以使用 `unionByName`。

```scala
df1.unionByName(df2)
```

### 7.3 窗口函数

窗口函数适合做组内排序、组内 TopN、累计求和等分析。

示例：每个年份中按销售额排序。

```scala
import org.apache.spark.sql.expressions.Window
import org.apache.spark.sql.functions._

val window = Window.partitionBy("year").orderBy($"amount".desc)

val result = df.withColumn("rn", row_number().over(window))
  .where($"rn" === 1)
```

常见函数：

| 函数 | 说明 |
|---|---|
| `row_number()` | 组内连续编号，不并列 |
| `rank()` | 排名，有并列会跳号 |
| `dense_rank()` | 排名，有并列不跳号 |
| `lag()` | 取上一行 |
| `lead()` | 取下一行 |
| `sum().over(...)` | 窗口内累计求和 |

## 8. RDD、DataFrame、Dataset 互转

### 8.1 RDD 转 DataFrame

```scala
import spark.implicits._

val df = rdd.map(line => {
  val arr = line.split(",")
  (arr(0).toInt, arr(1), arr(2).toInt)
}).toDF("id", "name", "age")
```

### 8.2 RDD 转 Dataset

```scala
case class User(id: Int, name: String, age: Int)

val ds = rdd.map(line => {
  val arr = line.split(",")
  User(arr(0).toInt, arr(1), arr(2).toInt)
}).toDS()
```

### 8.3 DataFrame 转 RDD

```scala
val rdd = df.rdd
```

DataFrame 转 RDD 后，每行是 `Row`。

Row 取值方式：

```scala
row.getInt(0)
row.getString(1)
row.getAs[Int]("age")
row.getAs[String]("name")
```

注意：

- 按索引取值性能直接，但可读性较差。
- 按字段名取值可读性更好，但要注意字段名是否存在。
- Row 中字段类型要和实际 Schema 保持一致。

### 8.4 DataFrame 转 Dataset

```scala
case class User(id: Int, name: String, age: Int)

val ds = df.as[User]
```

前提：

- 导入 `import spark.implicits._`。
- DataFrame 字段名和 case class 字段名能匹配。
- 字段类型能转换。

## 9. UDF 与 UDAF

### 9.1 UDF

UDF 是用户自定义普通函数，通常用于一行输入产生一个结果。

注册方式：

```scala
spark.udf.register("len", (s: String) => s.length)
```

SQL 使用：

```scala
spark.sql("select name, len(name) as name_len from tb_user")
```

DSL 使用：

```scala
val lenUdf = udf((s: String) => s.length)
df.select($"name", lenUdf($"name").as("name_len"))
```

注意：

- UDF 会让 Catalyst 难以理解内部逻辑，可能影响优化。
- 能用内置函数时，优先使用 `org.apache.spark.sql.functions` 中的函数。
- UDF 中要处理 null，否则容易出现空指针。

### 9.2 UDAF

UDAF 是用户自定义聚合函数，类似 `sum`、`avg` 这种聚合逻辑。

适合场景：

- 自定义平均值。
- 自定义同比、环比计算。
- 自定义复杂统计指标。

弱类型 UDAF 通常继承 `UserDefinedAggregateFunction`，需要定义：

| 方法 | 作用 |
|---|---|
| `inputSchema` | 输入数据结构 |
| `bufferSchema` | 聚合缓冲区结构 |
| `dataType` | 最终返回值类型 |
| `deterministic` | 相同输入是否返回相同结果 |
| `initialize` | 初始化缓冲区 |
| `update` | 每条数据更新缓冲区 |
| `merge` | 合并不同分区的缓冲区 |
| `evaluate` | 计算最终结果 |

强类型 UDAF 通常通过 `Aggregator[IN, BUF, OUT]` 实现：

| 类型参数 | 说明 |
|---|---|
| `IN` | 输入类型 |
| `BUF` | 缓冲区类型 |
| `OUT` | 输出类型 |

核心流程：

```text
zero：初始化缓冲区
reduce：分区内聚合
merge：分区间合并
finish：输出最终结果
```

## 10. SparkSQL 执行原理

### 10.1 总体流程

SparkSQL 的执行过程可以概括为：

```text
SQL 文本
  ↓
Parser 解析
  ↓
Unresolved Logical Plan 未解析逻辑计划
  ↓
Analyzer 绑定元数据
  ↓
Resolved Logical Plan 已解析逻辑计划
  ↓
Optimizer 逻辑优化
  ↓
Optimized Logical Plan 优化后逻辑计划
  ↓
SparkPlanner 生成物理计划
  ↓
Selected Physical Plan 选中的物理计划
  ↓
RDD / WholeStageCodegen 执行
```

### 10.2 SessionCatalog

SessionCatalog 负责管理 SparkSQL 中的元数据信息。

包括：

- 数据库。
- 表。
- 临时视图。
- 函数。
- 字段名和字段类型。

Analyzer 会通过 SessionCatalog 把 SQL 中未解析的表名、字段名绑定到真实元数据。

### 10.3 Parser

Parser 负责把 SQL 字符串解析成逻辑计划。

例如：

```sql
select name, age from tb_user where age > 30
```

会先被解析成一个尚未绑定真实表和字段的逻辑计划。

### 10.4 Analyzer

Analyzer 负责解析和绑定：

- 表名是否存在。
- 字段名是否存在。
- 字段类型是否匹配。
- 函数是否存在。

如果 SQL 写错字段名，通常会在 Analyzer 阶段报错。

### 10.5 Optimizer

Optimizer 是 Catalyst 优化器的重要组成部分，负责对逻辑计划做优化。

常见优化：

| 优化 | 说明 |
|---|---|
| 谓词下推 | 尽量把过滤条件提前到数据源读取阶段 |
| 列裁剪 | 只读取真正需要的列 |
| 常量替换 | 把可提前计算的常量表达式直接替换 |
| 常量折叠 | 把 `1 + 2` 这类表达式提前算成 `3` |
| 投影合并 | 合并连续的 select/project |
| 条件简化 | 简化无效或重复过滤条件 |

### 10.6 SparkPlanner

SparkPlanner 负责把优化后的逻辑计划转换成物理计划。

同一个逻辑计划可能有多种物理执行方式，例如：

- SortMergeJoin
- BroadcastHashJoin
- ShuffledHashJoin
- HashAggregate
- SortAggregate

Spark 会根据规则和统计信息选择合适的物理计划。

### 10.7 查看执行计划

```scala
df.explain()
df.explain(true)
```

`explain(true)` 通常可以看到：

- Parsed Logical Plan
- Analyzed Logical Plan
- Optimized Logical Plan
- Physical Plan

这是排查 SQL 执行效率、Join 策略和谓词下推是否生效的重要工具。

## 11. SparkSQL 实战思路

原文实战围绕销售数据分析，核心不是记代码，而是掌握分析拆解方式。

### 11.1 常见分析流程

```text
1. 加载维度表和事实表
2. 注册临时视图
3. 先做基础清洗和字段转换
4. 用 group by 聚合指标
5. 用 join 补充维度信息
6. 用窗口函数处理组内 TopN
7. 输出结果到文件、Hive 或数据库
```

### 11.2 每年销售单数和销售总额

典型 SQL：

```sql
select
  year,
  count(*) as order_count,
  sum(amount) as total_amount
from tb_order
group by year
```

知识点：

- 年份字段可能来自日期维表，也可能从订单日期中截取。
- `count` 统计订单数。
- `sum` 统计销售额。
- 按年聚合一定会触发 Shuffle。

### 11.3 每年最大金额订单

常见解法：

```sql
select *
from (
  select
    *,
    row_number() over(partition by year order by amount desc) as rn
  from tb_order
) t
where rn = 1
```

知识点：

- `partition by year` 表示每年单独排序。
- `order by amount desc` 表示按金额降序。
- `row_number = 1` 取每组第一条。

### 11.4 每年最畅销货品

拆解方式：

```text
1. 统计每年每个货品的销售额
2. 对每年内部按销售额倒序排名
3. 取排名第一的货品
4. 如需商品名称，再 Join 商品维表
```

典型 SQL：

```sql
select *
from (
  select
    year,
    product_id,
    sum(amount) as product_amount,
    row_number() over(
      partition by year
      order by sum(amount) desc
    ) as rn
  from tb_order
  group by year, product_id
) t
where rn = 1
```

## 12. SparkSQL 整合 Hive

### 12.1 整合方式

SparkSQL 可以通过 Hive metastore 访问 Hive 中已有的库表。

关键配置：

- Spark 应用需要启用 `enableHiveSupport()`。
- `hive-site.xml` 放到 Spark 或应用 classpath 下。
- Spark 能访问 Hive metastore 服务。
- Spark 能访问 Hive 表底层存储，例如 HDFS。

### 12.2 SparkSQL 与 Hive 的关系

| 对比项 | Hive | SparkSQL |
|---|---|---|
| 默认执行引擎 | MapReduce / Tez / Spark | Spark |
| 交互速度 | 相对慢 | 通常更快 |
| 元数据 | Hive Metastore | 可复用 Hive Metastore |
| SQL 方言 | HiveQL | 兼容 HiveQL，支持 SparkSQL 扩展 |
| 适用场景 | 离线数仓 | 离线分析、交互式分析、统一数据访问 |

### 12.3 ThriftServer

SparkSQL 可以启动 HiveServer2 兼容服务，对外提供 JDBC/ODBC 访问。

适合场景：

- BI 工具连接 SparkSQL。
- 数据平台统一 SQL 查询入口。
- 用 JDBC 客户端提交 SQL。

常见启动方式：

```bash
start-thriftserver.sh \
  --master yarn \
  --deploy-mode client
```

## 13. 高频易混点

### 13.1 DataFrame 是不是 RDD

DataFrame 底层依赖 RDD 执行，但它不是普通 RDD。

关键区别：

- DataFrame 有 Schema。
- DataFrame 可以被 Catalyst 优化。
- DataFrame 的一行是 Row。
- DataFrame 最终会被转换为物理计划和 RDD 任务执行。

### 13.2 临时视图和表的区别

| 类型 | 生命周期 | 说明 |
|---|---|---|
| 临时视图 | 当前 SparkSession 内有效 | `createOrReplaceTempView` |
| 全局临时视图 | 当前 Spark 应用内跨 Session 有效 | `createGlobalTempView` |
| Hive 表 | 持久化存在 | `saveAsTable` 或 Hive 建表 |

### 13.3 `repartition` 和 `coalesce`

| 方法 | 是否 Shuffle | 适合场景 |
|---|---|---|
| `repartition(n)` | 通常会 Shuffle | 增加或重新均衡分区 |
| `coalesce(n)` | 默认尽量不 Shuffle | 减少分区数量 |

写文件前经常用它们控制输出文件数量。

### 13.4 `cache` 和 `persist`

当同一个 DataFrame 被多次复用时，可以缓存：

```scala
df.cache()
df.persist()
```

注意：

- 缓存是懒执行，需要 Action 触发。
- 缓存会占用内存，不再使用时可以 `unpersist()`。
- 不是所有 DataFrame 都应该缓存，只有复用且计算代价较高时才值得。

### 13.5 小文件问题

SparkSQL 写出数据时，每个 Task 通常会生成一个或多个文件。

常见原因：

- 分区数过多。
- Hive 动态分区字段基数过高。
- 多批次 append 写入。

常见处理：

```scala
df.coalesce(10).write...
df.repartition(10).write...
df.repartition($"dt").write.partitionBy("dt")...
```

## 14. 面试速记

### 14.1 SparkSQL 执行流程

```text
SQL -> Parser -> Analyzer -> Optimizer -> SparkPlanner -> Physical Plan -> RDD
```

### 14.2 Catalyst 做什么

Catalyst 是 SparkSQL 的查询优化器，主要负责：

- 解析 SQL。
- 绑定元数据。
- 优化逻辑计划。
- 生成和选择物理计划。

### 14.3 DataFrame 为什么比 RDD 更容易优化

因为 DataFrame 有 Schema，SparkSQL 知道字段名、字段类型和表达式语义，可以进行谓词下推、列裁剪、Join 策略选择等优化。

RDD 中的函数对 Spark 来说更像黑盒，优化空间更小。

### 14.4 UDF 为什么可能影响性能

因为普通 UDF 的内部逻辑对 Catalyst 来说不透明，优化器很难对 UDF 内部表达式做列裁剪、谓词下推、常量折叠等优化。

优先级：

```text
内置函数 > SQL 表达式 > UDF
```

### 14.5 什么时候用窗口函数

适合解决组内排序和组内取值问题，例如：

- 每年销售额最高的订单。
- 每个用户最近一次登录。
- 每个品类销量 Top3 商品。
- 按时间累计求和。

## 15. 建议学习顺序

1. 先掌握 SparkSession、DataFrame、Dataset 的概念。
2. 熟练读取 CSV、JSON、Parquet、JDBC、Hive。
3. 熟练使用 SQL 和 DSL 完成过滤、聚合、Join、窗口函数。
4. 理解 RDD、DataFrame、Dataset 互转。
5. 学会使用 UDF 和 UDAF，但优先使用内置函数。
6. 重点理解 Catalyst 执行流程和常见优化。
7. 最后结合 Hive、JDBC、分区输出、小文件问题做实战。

