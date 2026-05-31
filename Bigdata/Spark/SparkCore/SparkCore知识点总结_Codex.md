# SparkCore 知识点总结

> 来源：`SparkCore .md`  
> 目标：把 SparkCore 的核心知识压缩成一份便于复习、面试回顾和建立知识框架的学习笔记。

## 1. Spark 基础

### 1.1 Spark 是什么

Spark 是一个快速、通用、可扩展的分布式大数据计算引擎。

核心定位：

- 和 MapReduce 一样，Spark 也是分布式计算引擎。
- Spark 不只支持离线批处理，还支持 SQL、流处理、机器学习、图计算等场景。
- Spark 的核心抽象是 RDD，后续又发展出 DataFrame、Dataset 等更高层 API。

### 1.2 Spark 的特点

| 特点 | 说明 |
|---|---|
| Speed 快速 | 支持内存计算，减少 HDFS 读写；使用 DAG 调度优化执行计划 |
| Ease of Use 易用 | 支持 Scala、Java、Python、R；提供大量高阶算子和 Spark Shell |
| Generality 通用 | 统一支持批处理、SQL、流处理、机器学习、图计算 |
| Runs Everywhere | 可运行在 Standalone、YARN、Mesos、Kubernetes 等资源调度系统上 |

### 1.3 Spark 与 MapReduce 对比

| 对比项 | MapReduce | Spark |
|---|---|---|
| 计算模型 | Map + Reduce，模型相对单一 | 基于 RDD/DAG，算子丰富 |
| 中间结果 | 通常写入 HDFS | 可写本地磁盘，也可缓存到内存 |
| 执行效率 | 多个 MR 串联时频繁读写 HDFS，效率低 | DAG 优化 + 内存计算，效率更高 |
| 适用场景 | 主要离线批处理 | 批处理、交互式查询、流处理、机器学习等 |
| 编程体验 | API 相对笨重 | Scala/Python 等 API 更简洁 |

面试要点：

- MapReduce 的复杂任务通常要拆成多个 MR 作业，中间结果落 HDFS，网络 IO 和磁盘 IO 开销大。
- Spark 会把多个转换逻辑构造成 DAG，并基于依赖关系优化执行计划。
- Spark Shuffle 时也会落本地磁盘，并不是所有数据都一直在内存中。

## 2. Spark 架构体系

### 2.1 常见部署模式

| 模式 | 说明 |
|---|---|
| Local | 本地运行，常用于开发、调试 |
| Standalone | Spark 自带资源调度模式，部署简单 |
| Spark on YARN | 运行在 Hadoop YARN 上，生产中常见 |
| Kubernetes | 容器化资源调度场景 |

### 2.2 Standalone Client 与 Cluster 的区别

本质区别：Driver 运行在哪里。

| 模式 | Driver 位置 | 特点 |
|---|---|---|
| client | 提交任务的 SparkSubmit 进程中 | 日志查看方便；客户端断开可能影响任务 |
| cluster | 集群 Worker 启动的 DriverWrapper 进程中 | 更适合生产；Driver 在集群内部运行 |

注意：

- `spark-shell` 只能以 client 模式运行，因为交互式命令行必须和 SparkContext 在同一个进程中。
- cluster 模式可以更灵活地设置 Driver 的内存和 CPU，例如 `--driver-memory`、`--driver-cores`。

### 2.3 Spark 重要角色

| 角色 | 职责 |
|---|---|
| Driver | 创建 SparkContext，生成 DAG，划分 Stage，封装 Task，调度任务 |
| Master | Standalone 模式下负责资源调度、接收 Worker 注册和心跳 |
| Worker | Standalone 模式下的工作节点，负责启动 Executor |
| Executor | 运行在 Worker/NodeManager 上的进程，负责执行 Task 和缓存数据 |
| Task | Spark 最小执行单元，运行在 Executor 线程池中 |

### 2.4 Spark 与 YARN 角色对应

| Spark | YARN 中的近似角色 |
|---|---|
| Driver | ApplicationMaster 或客户端 Driver，取决于 deploy-mode |
| Executor | Container 中运行的执行进程 |
| Worker | NodeManager 所在节点的工作能力 |
| Master | ResourceManager 的资源调度能力 |

## 3. Spark 程序运行入口

### 3.1 SparkContext

SparkContext 是 Spark 应用程序的入口。

主要作用：

- 连接集群资源管理器。
- 创建 RDD。
- 触发 Job。
- 维护调度相关组件，例如 DAGScheduler、TaskScheduler。

可以简单理解为：SparkContext 在哪里创建，Driver 就在哪里。

### 3.2 Spark Shell

Spark Shell 是交互式开发工具，适合学习、测试算子和调试逻辑。

常见特点：

- 启动后默认创建 `sc`，即 SparkContext。
- Scala Shell 中也常见 `spark`，即 SparkSession。
- 适合快速验证 RDD 转换和 Action。

### 3.3 WordCount 基本流程

典型 Scala 写法：

```scala
sc.textFile(input)
  .flatMap(_.split(" "))
  .map((_, 1))
  .reduceByKey(_ + _)
  .saveAsTextFile(output)
```

执行逻辑：

1. `textFile` 创建 HadoopRDD。
2. `flatMap` 拆分单词。
3. `map` 转成 `(word, 1)`。
4. `reduceByKey` 按 key 聚合，会产生 Shuffle。
5. `saveAsTextFile` 是 Action，触发整个 Job 执行。

## 4. RDD 核心

### 4.1 RDD 是什么

RDD，全称 Resilient Distributed Dataset，弹性分布式数据集。

关键理解：

- RDD 是 Spark 最基础的数据抽象。
- RDD 不真正存储业务数据，而是记录数据来源、分区、依赖关系和计算逻辑。
- RDD 具备血统关系 Lineage，可用于失败恢复。

### 4.2 RDD 的特点

| 特点 | 说明 |
|---|---|
| 分区 | 数据被拆成多个 partition，可并行计算 |
| 只读 | RDD 不可变，转换会生成新的 RDD |
| 依赖关系 | RDD 之间有窄依赖或宽依赖 |
| 容错 | 可根据血统重新计算丢失分区 |
| 延迟执行 | Transformation 不立即执行，Action 才触发计算 |

### 4.3 创建 RDD 的方式

常见方式：

- 从集合创建：`sc.parallelize(Seq(...))`
- 从外部存储创建：`sc.textFile(path)`
- 由已有 RDD 转换生成：`map`、`flatMap`、`filter` 等

### 4.4 Transformation 与 Action

| 类型 | 特点 | 示例 |
|---|---|---|
| Transformation | 懒执行，返回新的 RDD | `map`、`flatMap`、`filter`、`reduceByKey` |
| Action | 触发 Job，返回结果或写外部系统 | `collect`、`count`、`saveAsTextFile` |

判断方法：

- 返回 RDD 的一般是 Transformation。
- 返回普通值、集合，或写出数据的一般是 Action。

## 5. RDD 常用算子

### 5.1 常见 Transformation 算子

| 算子 | 作用 | 是否常见 Shuffle |
|---|---|---|
| `map` | 一进一出转换 | 否 |
| `flatMap` | 一进多出并压平 | 否 |
| `filter` | 过滤数据 | 否 |
| `mapPartitions` | 按分区批量处理 | 否 |
| `mapPartitionsWithIndex` | 处理时获取分区编号 | 否 |
| `union` | 合并两个 RDD | 否 |
| `distinct` | 去重 | 是 |
| `reduceByKey` | 按 key 聚合 | 通常是 |
| `groupByKey` | 按 key 分组 | 通常是 |
| `combineByKey` | 更底层的按 key 聚合 | 通常是 |
| `foldByKey` | 带初始值的按 key 聚合 | 通常是 |
| `aggregateByKey` | 分区内和分区间聚合逻辑可不同 | 通常是 |
| `partitionBy` | 按分区器重新分区 | 通常是 |
| `repartition` | 增加或调整分区 | 是 |
| `coalesce` | 减少分区时可避免 Shuffle | 可选 |
| `sortBy` | 排序 | 是 |
| `sortByKey` | 按 key 排序 | 是 |
| `join` | 两个 PairRDD 按 key 连接 | 通常是 |
| `leftOuterJoin` | 左外连接 | 通常是 |
| `rightOuterJoin` | 右外连接 | 通常是 |
| `fullOuterJoin` | 全外连接 | 通常是 |
| `intersection` | 交集 | 通常是 |
| `subtract` | 差集 | 通常是 |
| `cartesian` | 笛卡尔积 | 数据量风险大 |

### 5.2 常见 Action 算子

| 算子 | 作用 |
|---|---|
| `collect` | 拉取全部数据到 Driver，数据大时慎用 |
| `count` | 统计元素个数 |
| `first` | 获取第一个元素 |
| `take(n)` | 获取前 n 个元素 |
| `top(n)` | 获取最大的 n 个元素 |
| `takeOrdered(n)` | 获取排序后的前 n 个元素 |
| `reduce` | 全局归约 |
| `fold` | 带初始值的全局归约 |
| `aggregate` | 分区内和分区间可使用不同聚合逻辑 |
| `sum` | 求和 |
| `min` / `max` | 最小值 / 最大值 |
| `foreach` | 对每条数据执行逻辑 |
| `foreachPartition` | 对每个分区执行逻辑 |
| `saveAsTextFile` | 保存为文本文件 |

### 5.3 高频算子对比

#### reduceByKey 与 groupByKey

| 算子 | 特点 |
|---|---|
| `groupByKey` | 先把同 key 的所有 value 拉到一起，再处理；数据量大时压力大 |
| `reduceByKey` | map 端可预聚合，减少 Shuffle 数据量 |

结论：能用 `reduceByKey` 时优先用 `reduceByKey`，避免直接 `groupByKey` 后再聚合。

#### repartition 与 coalesce

| 算子 | 说明 |
|---|---|
| `repartition(n)` | 本质通常是 `coalesce(n, shuffle = true)`，会 Shuffle |
| `coalesce(n)` | 常用于减少分区，默认不 Shuffle |

经验：

- 增加分区通常用 `repartition`。
- 减少分区通常用 `coalesce`。
- 如果减少分区后数据倾斜严重，可考虑开启 Shuffle。

#### foreach 与 foreachPartition

| 算子 | 适用场景 |
|---|---|
| `foreach` | 每条数据执行一次逻辑 |
| `foreachPartition` | 每个分区执行一次逻辑，适合批量创建连接、批量写库 |

写外部系统时，通常优先考虑 `foreachPartition`，避免每条数据都创建连接。

## 6. 缓存与 Checkpoint

### 6.1 cache / persist

作用：把 RDD 的计算结果缓存起来，避免后续 Action 重复计算。

区别：

- `cache()` 是 `persist()` 的简化形式。
- `persist()` 可以指定存储级别，例如内存、磁盘、序列化等。

适用场景：

- 一个 RDD 被多个 Action 复用。
- 某个 RDD 计算链路很长、代价高。

注意：

- cache/persist 是懒执行，也需要 Action 触发。
- 缓存不切断血统关系。

### 6.2 checkpoint

作用：把 RDD 数据写入可靠存储，切断血统关系。

适用场景：

- RDD 血统链过长。
- 计算链路复杂，失败恢复成本高。
- 流式或迭代计算中需要增强容错。

对比：

| 项目 | cache/persist | checkpoint |
|---|---|---|
| 存储位置 | 内存或本地磁盘为主 | 可靠文件系统，如 HDFS |
| 是否切断血统 | 否 | 是 |
| 主要目的 | 性能优化 | 容错和截断依赖链 |

## 7. Spark 重要概念

| 概念 | 解释 |
|---|---|
| Application | 一次 Spark 应用提交，对应一个 SparkContext |
| Job | 每触发一次 Action 通常生成一个 Job |
| DAG | RDD 转换逻辑和依赖关系形成的有向无环图 |
| Stage | Job 按 Shuffle/宽依赖切分出的执行阶段 |
| Task | Spark 最小执行单元，处理一个分区的数据 |
| TaskSet | 同一个 Stage 中多个 Task 的集合 |
| Driver | 负责程序入口、DAG 构建、Stage 划分和任务调度 |
| Executor | 执行 Task、缓存数据的工作进程 |

关系链：

```text
Application -> 多个 Job -> 多个 Stage -> 多个 Task
```

关键点：

- 一个 Action 触发一个 Job。
- 一个 Job 根据宽依赖切分为多个 Stage。
- 一个 Stage 对应一个 TaskSet。
- Task 数量通常等于该 Stage 最后一个 RDD 的分区数。

## 8. 依赖关系、Stage 与 Task

### 8.1 窄依赖

父 RDD 的一个分区最多被子 RDD 的一个分区使用。

特点：

- 不需要 Shuffle。
- 可以在同一个 Stage 中流水线执行。
- 常见算子：`map`、`flatMap`、`filter`。

### 8.2 宽依赖

父 RDD 的一个分区会被多个子 RDD 分区使用。

特点：

- 通常需要 Shuffle。
- 是 Stage 划分的依据。
- 常见算子：`reduceByKey`、`groupByKey`、`join`、`distinct`、`sortByKey`。

### 8.3 Stage 类型

| Stage 类型 | 说明 |
|---|---|
| ShuffleMapStage | 中间阶段，生成 ShuffleMapTask，为下游 Shuffle Read 准备数据 |
| ResultStage | 最后阶段，生成 ResultTask，负责输出结果或返回 Driver |

### 8.4 Task 类型

| Task 类型 | 说明 |
|---|---|
| ShuffleMapTask | 通常在非最终 Stage 中执行，产生 Shuffle Write 数据 |
| ResultTask | 最后一个 Stage 的 Task，产生最终结果 |

## 9. Spark 执行流程

完整流程：

1. 通过 `spark-submit` 启动 SparkSubmit 进程。
2. 反射调用用户指定的 `main` 方法。
3. 创建 SparkContext，向资源管理器申请资源。
4. Worker/NodeManager 启动 Executor。
5. Executor 向 Driver 反向注册。
6. Driver 创建 RDD，记录转换关系，构建 DAG。
7. Action 触发 Job。
8. DAGScheduler 根据宽依赖切分 Stage。
9. 每个 Stage 生成 TaskSet。
10. TaskScheduler 将 Task 调度到 Executor。
11. Executor 反序列化 Task，放入线程池执行。
12. Task 执行迭代器链，处理对应分区数据。
13. 结果写入外部系统或返回 Driver。

### 9.1 DAGScheduler 与 TaskScheduler

| 组件 | 职责 |
|---|---|
| DAGScheduler | 把 DAG 按宽依赖切分 Stage，生成 TaskSet |
| TaskScheduler | 根据 Executor 资源把 Task 发送到具体 Executor 执行 |

## 10. Shuffle 深入理解

### 10.1 Shuffle 是什么

Shuffle 是按照分区器规则，把数据重新分配到不同分区的过程。

关键理解：

- Shuffle 通常伴随网络传输和磁盘 IO。
- 有网络传输不一定是 Shuffle。
- Shuffle 不是上游 Task 主动推给下游 Task，而是下游 Task 到上游拉取属于自己分区的数据。

### 10.2 Shuffle Write 与 Shuffle Read

| 阶段 | 说明 |
|---|---|
| Shuffle Write | 上游 ShuffleMapTask 按分区器把数据写到本地磁盘 |
| Shuffle Read | 下游 Task 拉取自己分区的数据进行计算 |

### 10.3 reduceByKey 一定会 Shuffle 吗

不一定。

如果 RDD 已经使用相同分区器分区，并且分区数没有改变，再调用 `reduceByKey` 时可能不再产生新的 Shuffle。

### 10.4 join 一定会 Shuffle 吗

不一定。

如果两个 RDD：

- 都已经按照相同分区器分区；
- 分区数量相同；
- join 时分区器未改变；

那么 join 可能避免 Shuffle。

### 10.5 Shuffle 数据复用

Shuffle 中间结果会保存在 Executor 所在机器的本地磁盘。

只要 Application 还在运行，并且 Shuffle 数据没有丢失，后续 Action 如果复用到同一份 Shuffle 结果，可以跳过前面部分 Stage，直接读取 Shuffle 中间数据。

如果机器宕机或磁盘数据丢失，Spark 会根据 RDD 血统重新计算丢失的分区。

### 10.6 ShuffleManager 演进

| 阶段 | 说明 |
|---|---|
| HashShuffleManager | 老版本方式，容易产生大量小文件 |
| HashShuffle + File Consolidation | 同一个 core 上的多个 Task 可复用输出文件，减少文件数 |
| SortShuffleManager | Spark 2.x 后主流方式，统一输出数据文件和索引文件 |

### 10.7 SortShuffleManager 三种 Writer

| Writer | 使用条件 / 特点 |
|---|---|
| BypassMergeSortShuffleWriter | 无 map 端聚合，分区数小于阈值，适合数据量不大场景 |
| UnsafeShuffleWriter | 使用序列化后排序，要求序列化器等条件满足 |
| SortShuffleWriter | 通用方式，内存不足会 spill 到磁盘，最后合并为数据文件和索引文件 |

## 11. 广播变量

### 11.1 使用场景

适合大表关联小表场景。

如果一个 RDD 很大，另一个数据集较小，直接 join 会产生 Shuffle。可以把小数据集广播到每个 Executor，在 map 端完成关联，避免 Shuffle。

### 11.2 特点

- 广播变量由 Driver 创建。
- 广播后分发到每个 Executor。
- Executor 中多个 Task 共享一份广播变量。
- 广播变量只读，不能在 Executor 中修改。
- Spark 广播变量底层使用 TorrentBroadcast，类似 BT 分发方式。

### 11.3 使用思路

```scala
val bc = sc.broadcast(smallMap)

largeRDD.map { record =>
  val map = bc.value
  // 使用 map 进行本地关联
}
```

## 12. 序列化问题

### 12.1 常见场景

Spark 程序中常见两类序列化问题：

| 场景 | 原因 |
|---|---|
| Bean 未实现序列化接口 | 数据对象需要网络传输或落盘 |
| 闭包引用不可序列化对象 | Task 发送到 Executor 时需要序列化闭包 |

### 12.2 闭包问题

如果算子函数中引用了 Driver 端不可序列化的对象，该对象会随着函数闭包一起序列化到 Executor，可能报错。

解决思路：

- 让被引用对象实现 `Serializable`。
- 避免在算子中引用不可序列化的外部对象。
- 在 Executor 内部创建连接或对象。
- 使用 `mapPartitions` 在每个分区内创建一次对象，减少开销。

## 13. Task 线程安全问题

核心原因：

- Executor 是进程。
- Task 是线程。
- 一个 Executor 内可以同时运行多个 Task。
- 如果多个 Task 共享同一个非线程安全对象，就可能出现并发问题。

典型风险：

- 多个 Task 共用同一个连接对象。
- 多个 Task 修改同一个可变集合。
- Driver 端对象被闭包带到 Executor 后产生并发访问。

处理建议：

- 不要让多个 Task 共享非线程安全对象。
- 在 `mapPartitions` 内部为每个分区创建独立对象。
- 必要时使用线程安全数据结构，但优先从设计上避免共享可变状态。

## 14. 累加器

### 14.1 作用

累加器用于在分布式计算过程中做全局计数或指标统计。

常见场景：

- 统计脏数据数量。
- 统计某类业务指标。
- 统计过滤掉的数据量。

### 14.2 为什么需要累加器

如果不用累加器，为了统计额外指标，可能需要触发多次 Action，导致 RDD 被重复计算。

使用累加器可以在一次 Action 中顺便完成指标统计。

### 14.3 使用示例

```scala
val acc = sc.longAccumulator("even-acc")

val rdd2 = rdd1.map { e =>
  if (e % 2 == 0) acc.add(1)
  e * 10
}

rdd2.saveAsTextFile(output)
println(acc.value)
```

注意：

- 累加器在 Executor 端累加，在 Driver 端读取结果。
- 不建议在 Transformation 中多次触发或依赖累加器结果做业务判断。
- 任务失败重试时要注意累加器可能出现重复累加风险。

## 15. Spark on YARN

### 15.1 YARN 角色回顾

| 角色 | 职责 |
|---|---|
| ResourceManager | 全局资源管理和调度 |
| ApplicationMaster | 单个应用的资源申请、任务管理和状态监控 |
| NodeManager | 单节点资源和 Container 管理 |
| Container | YARN 分配资源的基本单位 |

### 15.2 Spark on YARN Cluster 模式

特点：

- Driver 运行在 YARN 集群中的 ApplicationMaster 内。
- 客户端提交后可以退出。
- 更适合生产任务。

提交示例：

```shell
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --executor-memory 1g \
  --executor-cores 2 \
  --num-executors 3 \
  --class xxx.WordCount \
  app.jar input output
```

### 15.3 Spark on YARN Client 模式

特点：

- Driver 运行在客户端。
- ApplicationMaster 主要负责申请 Executor。
- 适合开发调试，日志查看方便。

### 15.4 资源分配要点

Executor 实际占用内存：

```text
executor-memory + memoryOverhead
```

其中：

```text
memoryOverhead = max(executorMemory * 0.1, 384MB)
```

例子：

- `--executor-memory 1g`，实际向 YARN 申请约 `1024MB + 384MB = 1408MB`。
- YARN Container 资源通常会按最小分配单位向上取整。

## 16. Spark 内存管理

### 16.1 Driver 与 Executor 内存

Spark 应用主要包含两类 JVM 进程：

| 进程 | 作用 |
|---|---|
| Driver | 调度、生成 DAG、提交 Job、协调 Executor |
| Executor | 执行 Task、缓存 RDD、处理 Shuffle |

Spark 内存管理重点通常指 Executor 内存管理。

### 16.2 堆内与堆外内存

| 类型 | 说明 |
|---|---|
| On-heap 堆内内存 | JVM 管理，由 `--executor-memory` 控制 |
| Off-heap 堆外内存 | 直接向操作系统申请，减少 GC，但需要 Spark 自己管理 |

### 16.3 堆内内存分区

Executor 堆内内存主要分为：

| 区域 | 作用 |
|---|---|
| Execution Memory | Shuffle、Join、Sort、Aggregation 等计算临时数据 |
| Storage Memory | RDD cache、Broadcast、unroll 数据 |
| User Memory | 用户代码和 RDD 转换相关对象 |
| Reserved Memory | Spark 内部预留内存 |

### 16.4 统一内存管理

Spark 现代版本采用统一内存管理，Execution 和 Storage 可以动态借用对方空间。

规则要点：

- Execution 不足时，可以借用 Storage 空闲空间。
- Storage 不足时，也可以借用 Execution 空闲空间。
- Execution 可以让 Storage 归还借用空间，必要时把缓存块淘汰到磁盘。
- Storage 通常不能强制让 Execution 归还空间，因为 Shuffle 计算过程更关键。
- 借用只发生在同类型内存之间，堆内不能借堆外，堆外也不能借堆内。

## 17. 高频面试题速记

### 17.1 SparkContext 在哪一端生成？

Driver 端。

SparkContext 是 Driver 的核心对象，Driver 还包含 DAGScheduler、TaskScheduler、ShuffleManager、BroadcastManager 等组件。

### 17.2 DAG 在哪一端构建？

Driver 端。

RDD 的转换关系和依赖关系都由 Driver 记录，Action 触发后形成完整 DAG。

### 17.3 RDD 在哪一端创建？

Driver 端。

RDD 本身不保存真实业务数据，而是保存数据来源、分区、依赖和计算逻辑。

### 17.4 Transformation 和 Action 在哪一端调用？

Driver 端调用。

### 17.5 算子中传入的函数在哪一端执行？

函数在 Driver 端定义和传入，但真正的业务逻辑在 Executor 中的 Task 内执行。

### 17.6 一个 Action 会发生什么？

一个 Action 通常触发一个 Job，Driver 根据 RDD 依赖构建 DAG，DAGScheduler 按宽依赖切分 Stage，TaskScheduler 把 Task 分发到 Executor 执行。

### 17.7 Stage 如何划分？

按宽依赖，也就是 ShuffleDependency 划分。

窄依赖可以放在同一个 Stage 中流水线执行；遇到宽依赖就切分 Stage。

### 17.8 Task 数量由什么决定？

通常由该 Stage 最后一个 RDD 的分区数决定。

### 17.9 reduceByKey 和 groupByKey 如何选择？

优先使用 `reduceByKey`。

原因：`reduceByKey` 可以在 map 端预聚合，减少 Shuffle 数据量；`groupByKey` 会把同 key 的所有 value 拉到一起，数据量大时容易造成网络、内存压力。

### 17.10 collect 有什么风险？

`collect` 会把所有分区数据拉回 Driver。数据量大时容易导致 Driver OOM。

### 17.11 广播变量解决什么问题？

解决大表关联小表时 Shuffle 成本高的问题。

将小表广播到 Executor 后，可以在 map 端本地关联，实现类似 map-side join。

### 17.12 累加器解决什么问题？

在分布式计算中统计全局指标，避免为了额外统计反复触发 Action。

### 17.13 cache 和 checkpoint 的区别？

`cache` 主要用于性能优化，不切断血统；`checkpoint` 主要用于容错和截断血统，会写入可靠存储。

### 17.14 Shuffle 一定发生网络传输吗？

Shuffle 通常涉及网络传输，但有网络传输不一定是 Shuffle。

Shuffle 的本质是按照分区器规则重新分配数据。

### 17.15 join 一定会 Shuffle 吗？

不一定。

如果两个 RDD 已经使用相同分区器，并且分区数相同，join 时可能避免 Shuffle。

## 18. 学习建议

建议按以下顺序掌握：

1. 先理解 Spark 比 MapReduce 快在哪里：DAG、内存计算、减少 HDFS 中间落盘。
2. 掌握 Driver、Executor、Job、Stage、Task 的层级关系。
3. 熟悉 RDD 的 Transformation、Action 和延迟执行。
4. 重点理解宽依赖、窄依赖、Stage 划分和 Shuffle。
5. 掌握 `reduceByKey`、`groupByKey`、`join`、`repartition`、`coalesce` 等高频算子区别。
6. 再学习广播变量、累加器、序列化、线程安全这些生产问题。
7. 最后理解 Spark on YARN、资源参数和 Executor 内存管理。

一条主线记忆：

```text
写 Spark 程序
-> 创建 SparkContext
-> 创建 RDD 和转换链
-> Action 触发 Job
-> DAG 按 Shuffle 切 Stage
-> Stage 生成 TaskSet
-> Task 调度到 Executor
-> Executor 执行并输出结果
```
