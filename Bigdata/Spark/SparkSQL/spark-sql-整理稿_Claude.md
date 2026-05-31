# SparkSQL 整理稿

> 原笔记版本基准:**Spark 3.x SparkSQL**
> 涵盖:SparkSession、DataFrame/Dataset/RDD 互转、UDF/UDAF、Catalyst 编译流水线、Hive 集成。

---

## 一、知识点总览

```
SparkSQL
├─ 编程入口
│   ├─ SparkSession.builder()
│   └─ 内部封装 SparkContext + SQLContext + HiveContext
├─ 数据结构
│   ├─ RDD       强类型,无 Schema,无优化
│   ├─ DataFrame 有 Schema,行级类型 Row(== Dataset[Row])
│   └─ Dataset   强类型 + Schema + Catalyst 优化
├─ 数据源
│   ├─ 结构化文件:JSON / Parquet / ORC / CSV / Avro
│   ├─ JDBC:format("jdbc")
│   └─ Hive Metastore(enableHiveSupport)
├─ 函数
│   ├─ UDF
│   ├─ UDAF
│   │    ├─ 弱类型:UserDefinedAggregateFunction (已 deprecated)
│   │    └─ 强类型:Aggregator[IN, BUF, OUT]
│   └─ UDTF(SparkSQL 原生不直接支持,通过 Hive UDTF 引入)
├─ Catalyst 执行流水线
│   SQL/DSL → Unresolved Plan → Analyzed Plan → Optimized Plan → SparkPlan → RDD
└─ 与 Hive 整合
    ├─ 共享 metastore_db
    └─ spark-sql / thriftserver
```

---

## 二、SparkSession 入口

```
SparkSession.builder()
  .appName("...")
  .master("local[*]")
  .enableHiveSupport()           // 可选,开启则使用 HiveExternalCatalog
  .config("spark.sql.shuffle.partitions", "200")
  .getOrCreate()
        │
        ├─ 内部持有 SparkContext (复用)
        ├─ SessionState(sqlParser, analyzer, optimizer, planner, catalog)
        └─ SharedState(externalCatalog, cacheManager)
```

**⚠ 原笔记纠错(关键点 1):** 原文说 "SparkSession 是 SparkContext 的子类"。**错。**
`SparkSession` 内部**组合**了 SparkContext(`session.sparkContext` 拿到),不是继承关系。SparkContext 仍然是物理资源入口,一个 JVM 内只能存在一个;SparkSession 可以同 JVM 创建多个(用 `newSession()` 隔离 catalog/temp view),共享 SparkContext。

---

## 三、RDD / DataFrame / Dataset 互转

```
                  toDF()                  as[T]
   ┌────────┐  ─────────►  ┌────────────┐ ─────────►  ┌─────────────┐
   │  RDD   │              │ DataFrame  │             │ Dataset[T]  │
   │        │  ◄─────────  │ (=DS[Row]) │  ◄────────  │             │
   └────────┘  .rdd        └────────────┘   toDF()    └─────────────┘
       ▲
       │ createDataFrame(rdd, schema)  显式 Schema
       │ rdd.toDF("col1","col2")       样例类反射推断
       │
```

**判别准则:**
- 需要类型安全(编译期检查列名/类型)→ Dataset
- 需要 SQL 字符串 / catalyst 优化、不需要类型 → DataFrame
- 需要细粒度算子(zipWithIndex 等 Dataset 没有的)→ 转 RDD

**⚠ 原笔记纠错(关键点 2):** 原文说 "DataFrame 是 Dataset 的子类"。**实质相反:**
- 源码:`type DataFrame = Dataset[Row]`(Scala 类型别名)
- 所以 DataFrame 不是 Dataset 的"子类",而是 `Dataset` 在泛型为 `Row` 时的**别名**。

---

## 四、UDF / UDAF

### 4.1 UDF 注册

```
spark.udf.register("strLen", (s: String) => s.length)
df.selectExpr("strLen(name)")
// 或 DSL:
import org.apache.spark.sql.functions.udf
val strLen = udf((s: String) => s.length)
df.select(strLen($"name"))
```

### 4.2 UDAF 两种风格

```
弱类型(已 @deprecated 自 Spark 3.0):
  class MyAvg extends UserDefinedAggregateFunction {
    def inputSchema: StructType ...
    def bufferSchema: StructType ...
    def dataType: DataType ...
    def initialize(buf) ...
    def update(buf, in) ...
    def merge(buf1, buf2) ...
    def evaluate(buf) ...
  }
  spark.udf.register("myAvg", new MyAvg)

强类型(推荐,Aggregator):
  case class Avg(sum: Long, cnt: Long)
  object MyAvg extends Aggregator[Long, Avg, Double] {
    def zero = Avg(0, 0)
    def reduce(b: Avg, a: Long) = Avg(b.sum + a, b.cnt + 1)
    def merge(b1: Avg, b2: Avg) = Avg(b1.sum+b2.sum, b1.cnt+b2.cnt)
    def finish(r: Avg) = r.sum.toDouble / r.cnt
    def bufferEncoder = Encoders.product[Avg]
    def outputEncoder = Encoders.scalaDouble
  }
  // 注册到 SQL:
  spark.udf.register("myAvg", functions.udaf(MyAvg))
```

**⚠ 原笔记纠错(关键点 3):** 原文说 "强类型 UDAF 只能用在 DSL 不能用 SQL"。**Spark 3.0+ 已支持:** `functions.udaf(MyAvg)` 把 Aggregator 包装为 UserDefinedFunction,可注册并以 SQL 函数调用。Spark 2.x 才有这个限制。

---

## 五、Catalyst 编译流水线(核心)

```
SQL 字符串 / DSL DataFrame
        │
        ▼
┌──────────────────────────┐
│ ① SqlParser              │  ANTLR4 词法/语法分析
│   sqlText → LogicalPlan  │  生成 Unresolved LogicalPlan
│   (UnresolvedRelation,   │  ── 表名/列名都还没绑定 ──
│    UnresolvedAttribute)  │
└─────────────┬────────────┘
              ▼
┌──────────────────────────┐
│ ② Analyzer               │  使用 SessionCatalog 解析
│   batch rules:           │  ├─ ResolveRelations:  表名→具体 LogicalPlan
│     ResolveRelations     │  ├─ ResolveReferences: 列名→AttributeReference
│     ResolveReferences    │  └─ ResolveFunctions:  函数签名校验
│     ResolveFunctions     │
│   → Analyzed Plan        │
└─────────────┬────────────┘
              ▼
┌──────────────────────────┐
│ ③ Optimizer              │  RBO(基于规则)
│   规则:                  │  ├─ PushDownPredicate (谓词下推)
│     谓词下推              │  ├─ ColumnPruning      (列裁剪)
│     列裁剪                │  ├─ ConstantFolding    (常量折叠)
│     常量折叠              │  ├─ BooleanSimplification
│     ...                  │  └─ CombineFilters
│   → Optimized Plan       │
└─────────────┬────────────┘
              ▼
┌──────────────────────────┐
│ ④ SparkPlanner           │  生成多个候选 PhysicalPlan(SparkPlan)
│   策略(Strategies):      │  ├─ JoinSelection:Broadcast/SortMerge/ShuffleHash
│     JoinSelection        │  └─ Aggregation:HashAgg/SortAgg
│     Aggregation          │  CBO(若启用 spark.sql.cbo.enabled)按 stats 选最优
│     ...                  │
└─────────────┬────────────┘
              ▼
┌──────────────────────────┐
│ ⑤ Prepare / Execute      │  WholeStageCodegen:把多个算子拼成一个 Java 类
│   physicalPlan.execute() │   → 编译成字节码,减少虚函数调用和 boxing
│   → RDD[InternalRow]     │  → toRdd → DAGScheduler.runJob
└──────────────────────────┘
```

### 5.1 多文件调用链

```
spark.sql("SELECT ...")
  └─> SparkSession.sql
        └─> sessionState.sqlParser.parsePlan(sql)         // ① 解析
              └─> Dataset.ofRows(self, logicalPlan)
                    └─> QueryExecution(self, logicalPlan)
                          ├─ analyzed = analyzer.execute(logical)        // ②
                          ├─ optimizedPlan = optimizer.execute(analyzed) // ③
                          ├─ sparkPlan = planner.plan(optimized).next()  // ④
                          ├─ executedPlan = prepareForExecution(sparkPlan)
                          └─ toRdd                                       // ⑤
                                └─ executedPlan.execute()  → RDD[InternalRow]
```

**⚠ 原笔记纠错(关键点 4):** 原文说"SparkPlanner 输出唯一物理计划"。**错。**
`planner.plan(plan)` 返回 `Iterator[SparkPlan]`,代表多个候选物理计划。早期 Spark 取 `.next()` 即第一个;Spark 3.x 启用 CBO 后会按 cost 排序选最优(`plan.bestEffortPlan`)。

---

## 六、关键优化器规则示例

```
原始 SQL:
   SELECT name FROM (
       SELECT * FROM t WHERE age > 18
   ) WHERE name = 'Tom'

Optimized Plan:
   Project [name]
     Filter (age > 18 AND name = 'Tom')   ← 谓词合并 + 下推
       Relation t [name#0]                ← 列裁剪:只读 name 列(注:还需 age 列做 filter)
```

- **谓词下推 (PushDownPredicate)**:把 `Filter` 推到尽量靠近 `Relation` 的位置,Parquet/ORC 可在文件层 skip block。
- **列裁剪 (ColumnPruning)**:只读 SELECT/Filter 用到的列。
- **常量折叠 (ConstantFolding)**:`1+2` 编译期算成 `3`。
- **常量传播 (ConstantPropagation)**:`WHERE a=1 AND b=a` → `WHERE a=1 AND b=1`。

---

## 七、Join 物理实现选择(JoinSelection)

```
                                  ┌─ 任一侧 size ≤ spark.sql.autoBroadcastJoinThreshold (默认 10MB)
                                  │     → BroadcastHashJoin (无 Shuffle)
                                  │
                                  ├─ 等值 Join 且 key 可排序 + 数据量大
JoinSelection 按优先级选择   ─────►│     → SortMergeJoin (Shuffle + Sort)
                                  │
                                  ├─ 等值 Join 且小表能放下 hash(右侧 < threshold * shuffle 分区数)
                                  │     → ShuffledHashJoin
                                  │
                                  └─ 非等值(>/</between)
                                        → BroadcastNestedLoopJoin / CartesianProduct
```

**⚠ 原笔记纠错(关键点 5):** 原文写"SparkSQL 默认 BroadcastJoin"。**不严谨:**
默认会**优先**考虑 Broadcast(若一侧 ≤ 阈值),但**不是任何 join 都广播**。可手动 `broadcast(df)` hint 强制。Spark 3.0 的 AQE 还支持运行时切换:发现大小表实际大小后,把 SortMergeJoin 动态降级为 BroadcastJoin。

---

## 八、AQE(Adaptive Query Execution,Spark 3.0+)

`spark.sql.adaptive.enabled=true` 启用后,运行时根据真实 stats 动态调整:

```
Stage 1 完成 → 收集 Map 输出的真实统计(每个 partition 大小)
        ▼
   ┌────────────────────────────────────────────────┐
   │ 1. 动态合并小分区  Coalesce shuffle partitions  │
   │ 2. 倾斜 Join 自动拆分大分区   Skew join         │
   │ 3. SortMergeJoin → BroadcastJoin (若可)         │
   └────────────────────────────────────────────────┘
        ▼
   下一个 Stage 用调整后的 plan 执行
```

原笔记基本没提 AQE,这是 SparkSQL 3.x 最重要的特性。

---

## 九、Hive 整合

```
SparkSession.builder()
  .enableHiveSupport()    // ← 关键,使用 HiveExternalCatalog
  .config("spark.sql.warehouse.dir", "/user/hive/warehouse")
  // hive-site.xml 放在 classpath:resources 下
  .getOrCreate()

启动方式:
  spark-sql        : 命令行,直接执行 SQL
  thriftserver     : start-thriftserver.sh → JDBC/ODBC 端口(默认 10000)
                     用 beeline 连接,行为类似 HiveServer2
```

**⚠ 原笔记纠错(关键点 6):** 原文说"SparkSQL 自带 metastore 数据库,启用 Hive 后是替换 metastore"。**不准确:**
- 不开启 Hive 时,Spark 用 `InMemoryCatalog`,**进程内**,关掉就丢;
- 开启 `enableHiveSupport()` 后用 `HiveExternalCatalog`,共享 Hive 的 metastore_db / MySQL metastore;
- 不是"替换",而是 catalog 实现切换。

---

## 十、关键纠错清单(汇总)

| # | 原笔记表述 | 正确表述 |
| --- | --- | --- |
| 1 | "SparkSession 是 SparkContext 的子类" | 是**组合关系**,SparkSession 内持有 SparkContext |
| 2 | "DataFrame 是 Dataset 的子类" | `type DataFrame = Dataset[Row]`,是**类型别名** |
| 3 | "强类型 UDAF 不能用 SQL" | Spark 3.0+ 用 `functions.udaf(Aggregator)` 即可在 SQL 中使用 |
| 4 | "SparkPlanner 输出唯一物理计划" | 返回 `Iterator[SparkPlan]`,有多个候选,按规则/CBO 选择 |
| 5 | "默认就是 BroadcastJoin" | 仅当一侧 ≤ `autoBroadcastJoinThreshold` 才广播;否则 SortMergeJoin |
| 6 | "启用 Hive = 替换 metastore" | 切换 catalog 实现(In-Memory → HiveExternalCatalog) |
| 7 | 未提 AQE | Spark 3.x 关键特性:动态合并分区、动态 Join 切换、倾斜处理 |
| 8 | "弱类型 UDAF 是首选" | 自 Spark 3.0 `UserDefinedAggregateFunction` 已 deprecated,推荐 Aggregator |

---
