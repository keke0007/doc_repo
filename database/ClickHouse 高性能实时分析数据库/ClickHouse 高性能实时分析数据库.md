# **课程概述 &#x20;**

本课程旨在系统性地介绍当今最快的开源在线分析处理（OLAP）数据库 ClickHouse。学员将从零开始，深入理解其核心架构、独特设计和强大功能。课程将通过大量的实践操作、真实案例分析和性能调优技巧，使学员不仅能“用上”ClickHouse，更能“用好”ClickHouse，从容应对海量数据的实时分析挑战。

课程特色

* **官方对标：** 内容紧密结合 Clickhouse.com 官方文档、博客和最佳实践，确保知识的权威性和前沿性。

* **实践驱动：** 每个章节都配有精心设计的动手实验，从环境搭建到复杂查询，再到集群管理，学以致用。

* **深度解析：** 不止于“如何用”，更深入讲解“为什么”，剖析 `MergeTree` 引擎、列式存储等核心原理，培养学员解决问题的底层能力。

* **全景视角：** 覆盖从单机部署到分布式集群，从开源版到 ClickHouse Cloud，从数据ETL到BI可视化集成的完整技术栈。

目标学员

* 数据工程师、后端开发工程师

* 数据库管理员（DBA）、系统运维工程师（SRE）

* 数据分析师、BI 工程师

* 对海量数据实时分析技术感兴趣的技术爱好者

# 第一部分：入门与核心概念 (Foundation)

## 第1章：ClickHouse 世界初探 (Introduction to ClickHouse)

### 1.1. **什么是 OLAP？**

**OLAP** 的全称是 **Online Analytical Processing**，即 **联机分析处理**。它是一种让你能够从不同角度、快速地对大量数据进行分析、查询和报告的技术。它的核心目标不是处理单笔交易，而是支持复杂的分析操作，帮助用户发现数据中的趋势、模式和洞察。

它的核心价值在于：

* **快速响应**：让复杂的分析查询能在几秒或几分钟内返回结果。

* **多维分析**：让用户能自由地从不同业务维度探索数据。

* **赋能业务**：让不懂技术的业务人员也能通过简单的拖拽（下钻、旋转等）进行自助式数据分析，从而做出更明智的业务决策。

***

* OLAP vs. OLTP 场景对比 (交易处理 vs. 分析处理)

* 现代数据分析的挑战：海量、高速、多维

![](<images/ClickHouse 高性能实时分析数据库-image-14.png>)

> **总结一下它们的区别：**

现代数据分析的挑战就是，**数据量越来越大（TB/PB级）、分析维度越来越多、对实时性的要求越来越高。**&#x4F20;统的解决方案（比如在 MySQL 上跑复杂 `GROUP BY`）已经力不从心，这就是 ClickHouse 等 OLAP 数据库大展身手的舞台。

### &#xA;1.2. **ClickHouse 闪亮登场**

```sql
* ClickHouse 官方定义：“Blazingly Fast, Open Source, Column-Oriented SQL Database”
* 核心特性：列式存储、向量化执行、数据压缩、MPP 架构
* 典型应用场景：用户行为分析、日志与指标监控、BI报表、广告与推荐系统
```

既然 OLAP 这么重要，那 ClickHouse 又是如何成为这个领域的佼佼者的呢？它的官方口号是：“**Blazingly Fast, Open Source, Column-Oriented SQL Database**”。我们来拆解一下它的“超能力”。

1. **列式存储 (Column-Oriented): 核心秘密武器**
   想象一下，我们有一张用户表，包含 `用户ID`, `姓名`, `城市`, `年龄`。

> * **行式存储 (如 MySQL):** 数据是按行存放在磁盘上的，像一本名册。
>   `[1, '张三', '北京', 30], [2, '李四', '上海', 25], [3, '王五', '北京', 35]`
>
> * **列式存储 (ClickHouse):** 数据是按列存放在一起的。
>   `[1, 2, 3], ['张三', '李四', '王五'], ['北京', '上海', '北京'], [30, 25, 35]`
>
> **问题：** 如果我们要计算所有用户的平均年龄，哪种存储方式更快？
> **答案：** 显然是列式存储！它只需要读取“年龄”那一列的数据，而行式存储必须把每一行所有的数据都读到内存里，再把不需要的字段丢掉，浪费了大量的 I/O。

![](<images/ClickHouse 高性能实时分析数据库-image.png>)

* **向量化执行 (Vectorized Execution):**
  这就像一个高效的工厂流水线。传统数据库是一个个地处理数据（标量执行），而 ClickHouse 是一批批地处理（向量化执行）。<span style="color: inherit; background-color: rgb(247,105,100)">它一次性从列中取出一整块数据（一个向量）</span>，然后在这个数据块上执行计算，极大地减少了函数调用开销，充分利用了 CPU 的 SIMD 指令。

* **超高的数据压缩率:**
  因为同一列的数据类型相同，具有相似性，所以更容易被压缩。比如一列都是城市名，很多重复值。ClickHouse 默认使用 **`LZ4`&#x20;**&#x538B;缩算法，可以达到很高的压缩比，进一步减少 I/O。

* **MPP 架构:**
  在集群模式下，一个查询会被打散到多台机器（**分片**）上并行执行，最后再汇总结果。人多力量大！

**典型应用场景:**

> * **用户行为分析:** 网站/App 的点击流分析、漏斗分析、留存分析。
>
> * **日志与指标监控:** 服务器、应用日志的实时查询与聚合。
>
> * **BI 报表:** 为 Tableau, Superset, Grafana 等 BI 工具提供高速后端。
>
> * **广告与推荐:** 广告曝光、点击数据的实时分析。


1.3. **ClickHouse 架构概览**

#### &#xA;1.3.1. 单节点架构 vs. 分布式集群架构

ClickHouse 可以单兵作战，也可以集团军出击。

1. **单节点架构 (Single Node):**
   这是最简单的模式，所有数据存储和计算都在一台服务器上完成。非常适合学习、开发或中小型数据量的场景。

![](<images/ClickHouse 高性能实时分析数据库-image-1.png>)

* **分布式集群架构 (Distributed Cluster):**


  这是生产环境的标配。它通过**分片 (Sharding)** 和 **副本 (Replication)** 来实现高可用和水平扩展。

  * **分片 (Shard):** 将数据水平切分，分布在不同的节点上，以提高查询性能。（把一本厚书撕成几份，让几个人同时看）

  * **副本 (Replica):** 每个分片的数据都有一个或多个备份，保证在一个节点宕机时，服务依然可用。（给每个人看的书都复印一份，防止有人弄丢了）

  * **ZooKeeper:** 在集群中扮演着“协调者”的角色，负责管理副本之间的数据同步和一致性。

![](<images/ClickHouse 高性能实时分析数据库-image-2.png>)


1.3.2. 客户端/服务器模型

想象一下去一家高级餐厅吃饭的过程。

> * **你 (Client/客户端):** 你是顾客，你想点一道“澳洲和牛配黑松露”。你不需要知道牛是怎么养的，也不需要知道黑松露是怎么挖的，你只需要用菜单（SQL语言）下达你的指令。
>
> * **餐厅厨房 (Server/服务器):** 这是 ClickHouse Server。它接收你的订单（SQL 查询），然后厨师们（ClickHouse 核心引擎）开始忙碌：从冷库（磁盘）里取出食材（数据），经过一系列复杂的烹饪（计算、聚合、排序），最后把一道精美的菜肴（查询结果）端给你。
>
> 这就是 **客户端/服务器 (C/S)** 模型。我们用来与 ClickHouse 交互的任何工具，无论是命令行工具 `clickhouse-client`、图形化界面的 DBeaver，还是我们程序代码中的一个库，都扮演着“客户端”的角色。它们通过网络，使用特定的“语言”（协议）与 ClickHouse 服务器进行通信。
>
> **主要通信协议:**
>
> * **TCP 协议 (Native Protocol):** 这是 `clickhouse-client` 和大多数驱动程序使用的默认方式，性能最高，就像餐厅的内部专用通道，效率极快。
>
> * **HTTP/HTTPS 协议:** ClickHouse 也提供了一个像网站一样的 HTTP 接口，你可以用任何能发网络请求的工具（比如浏览器或者 `curl`）来查询它，非常灵活，就像餐厅的外卖窗口。

![](<images/ClickHouse 高性能实时分析数据库-image-3.png>)

#### 1.3.3. ZooKeeper 的角色（在复制和分布式 DDL 中的作用）

**ZooKeeper** 对于 ClickHouse 集群来说，不是用来存储海量业务数据的，而是用来存储**元数据（Metadata）和协调任务**的。它就像是整个餐饮集团的“董事会”或者“中央计划委员会”，不参与具体做菜，但负责制定规则和同步信息。

**它的核心作用有两个：**

> 1. **数据复制 (Replication):**
>    当你在一个副本上 `INSERT` 一批新数据时，过程是这样的：
>    a. 该副本把“我插入了某某数据块”这个**操作日志**，发布到 ZooKeeper 的一个共享任务队列里。
>    b. 同一个分片下的其他副本，一直在“监听”这个队列。
>    c. 它们看到新任务后，就会从那个副本那里把对应的数据块拉过来，应用到自己身上。
>    通过这种方式，ZooKeeper 确保了所有副本的数据最终都是一致的。
>
> ![](<images/ClickHouse 高性能实时分析数据库-image-4.png>)
>
>

2. **分布式 DDL (Distributed DDL - Data Definition Language):**
   当你需要在整个集群上创建一个新表时（例如 `CREATE TABLE ... ON CLUSTER my_cluster`），你不可能手动去每个节点上都执行一遍。
   a. 你只需要在一个节点上执行这个 `ON CLUSTER` 命令。
   b. 这个节点会把“请在 my\_cluster 集群上创建这张表”这个**DDL任务**，提交到 ZooKeeper 的一个全局任务队列里。
   c. 集群里的**每一个节点**都会监听这个队列，看到新任务后，各自在本地执行这个 `CREATE TABLE` 命令。
   这样就保证了整个集群的表结构是一致的。

> **总结一下 ZooKeeper:** 它不存业务数据，但它维护了集群的“生命线”——**一致性**和**协调性**。没有它，分布式集群就是一盘散沙。

### &#xA;1.4. **横向对比：ClickHouse vs. 其他技术**

**一句话总结:**

* 当你的核心诉求是**对海量结构化数据做极速的聚合分析**时，首选 ClickHouse。

* 不要用 ClickHouse 来做高并发的单行 `UPDATE` 或 `DELETE`，这不是它的强项。

1.5. **生态版图：开源版 vs. ClickHouse Cloud**
\* 开源自建的优势与挑战
\* ClickHouse Cloud 介绍：Serverless、自动扩缩容、托管服务
\* **【实践】**: 注册并体验 ClickHouse Cloud Playground。

***

# 第二部分：基础实践与数据模型 (Hands-on & Data Modeling)

## 第2章：极速安装与基础操作 (Quick Start)



### 2.1. **环境搭建**


\* **【实践】**: 使用原生的CentOS系统,安装!最原始的安装方式

```bash

sudo yum-config-manager --add-repo https://packages.clickhouse.com/rpm/clickhouse.repo
sudo yum install -y clickhouse-server clickhouse-client
sudo yum install clickhouse-server-22.8.7.34

sudo systemctl enable clickhouse-server
sudo systemctl start clickhouse-server
sudo systemctl status clickhouse-server
clickhouse-client  -m
```

### 2.2. **客户端工具**


\* `clickhouse-client` 命令行交互

```bash
[root@linux01 ~]# 
[root@linux01 ~]# clickhouse-client    -m
ClickHouse client version 25.6.4.12 (official build).
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 25.6.4
试试第一个命令： SHOW DATABASES;
退出客户端： 输入 exit 或按 Ctrl+D。
```


\* HTTP/HTTPS 接口 (cURL, Postman)

```bash
# 发送一个简单的查询
curl 'http://localhost:8123/'  --data-binary  'select 1'
# 返回结果:                                                                                                              
# 1

示例2:
[root@linux01 ~]# curl 'http://localhost:8123' --data-binary 'SELECT number FROM system.numbers LIMIT 5 FORMAT CSV'         
0
1
2
3
4
```


\* DBeaver, DataGrip 等图形化工具连接

1. 打开 DBeaver，点击 “新建数据库连接”。

2. 在驱动列表中，搜索并选择 “ClickHouse”。

3. 在“主”设置面板中，填写以下信息：

   * **主机:** `localhost`

   * **端口:** `8123` (注意，大部分图形化工具默认走 HTTP 端口)

   * **用户:** `default`

   * **密码:** (默认是空的，不用填)

4. 点击左下角的 “测试连接”。如果看到“连接成功”的提示，就大功告成了！

5. 保存连接。现在你可以在左侧的数据库导航器中，像操作 MySQL 一样，直观地看到所有的数据库和表了。


2.3. **SQL 基础**
**【实践】**: `CREATE DATABASE`, `CREATE TABLE`

**<span style="color: rgb(216,57,49); background-color: inherit">创建数据库</span>**

```sql
linux01 :) create  database  learning;
linux01 :) use learning;
```

**<span style="color: rgb(216,57,49); background-color: inherit">创建表 </span>**

**`ENGINE `**&#x662F; ClickHouse 最最核心的操作之一。注意 `ENGINE = MergeTree()` 部分，我们将在下一章详细讲解它，现在你只需要知道这是 ClickHouse 最常用、最强大的表引擎。

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [NULL|NOT NULL] [DEFAULT|MATERIALIZED|EPHEMERAL|ALIAS expr1] [COMMENT 'comment for column'] [compression_codec] [TTL expr1],
    name2 [type2] [NULL|NOT NULL] [DEFAULT|MATERIALIZED|EPHEMERAL|ALIAS expr2] [COMMENT 'comment for column'] [compression_codec] [TTL expr2],
    ...
) ENGINE = engine
  [COMMENT 'comment for table']
```

```sql
-- 创建一张名为 't_web_hits' 的表
CREATE TABLE  t_web_hits
(
    `timestamp` DateTime ,          -- 访问时间
    `user_id`   UInt64,            -- 用户ID
    `url`       String,            -- 访问的URL
    `os`        String,            -- 操作系统
    `browser`   String,            -- 浏览器
    `duration_ms` UInt32           -- 页面停留时长(毫秒)
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp) -- 按月分区
ORDER BY (timestamp, user_id);   -- 按时间和用户ID排序
```

**(解释关键部分):**

* 我们定义了表的列名和数据类型，比如 `DateTime` 表示日期时间，`UInt64` 表示无符号64位整数。

* `ENGINE = MergeTree()`: **声明表的引擎类型，这是 ClickHouse 建表语句与标准 SQL 最大的不同！**

* `PARTITION BY toYYYYMM(timestamp)`: 按月对数据进行“物理切分”，提高查询效率。

* `ORDER BY (timestamp, user_id)`: 这是 ClickHouse 性能优化的关键！它规定了数据在磁盘上是如何物理排序的。
  \* **【实践】**: `INSERT INTO ... VALUES` 与 `INSERT INTO ... FORMAT`

```sql
-- 插入几条模拟数据
INSERT INTO learning.t_web_hits VALUES
(now(), 1001, '/page/a', 'iOS', 'Safari', 1500),
(now(), 1002, '/page/b', 'Windows', 'Chrome', 3200),
(now(), 1001, '/page/c', 'iOS', 'Safari', 5000);
```

**ClickHouse 特色插入方式 (FORMAT):**
ClickHouse 支持直接插入特定格式的数据，比如 `JSONEachRow`，这在数据导入时非常有用。

```sql
-- 插入 JSON 格式的数据
INSERT INTO learning.t_web_hits FORMAT JSONEachRow
{"timestamp": "2023-10-27 10:00:00", "user_id": 1003, "url": "/home", "os": "Android", "browser": "Chrome", "duration_ms": 800}
{"timestamp": "2023-10-27 10:01:00", "user_id": 1002, "url": "/profile", "os": "Windows", "browser": "Chrome", "duration_ms": 12000} ;
```


\* **【实践】**: `SELECT`, `WHERE`, `GROUP BY`, `ORDER BY`, `LIMIT`

```sql
-- 查询所有数据
SELECT * FROM learning.t_web_hits;

-- 案例1: 查询用户 1001 的访问记录
SELECT * FROM learning.t_web_hits WHERE user_id = 1001;

-- 案例2: 统计每个浏览器的使用次数
SELECT
    browser,
    count() AS view_count
FROM learning.t_web_hits
GROUP BY browser
ORDER BY view_count DESC;

-- 案例3: 计算每个用户的平均页面停留时长
SELECT
    user_id,
    avg(duration_ms) AS avg_duration
FROM learning.t_web_hits
GROUP BY user_id;
```


\* **【实践】**: `SHOW DATABASES`, `SHOW TABLES`, `DESCRIBE TABLE`

## 第3章：深入理解表引擎 (The Power of Table Engines)

**教学目标:**

1. 理解“表引擎”在 ClickHouse 中的核心地位和作用。

2. **深入掌握 `MergeTree` 引擎的工作原理**，包括其数据存储结构（分区、数据片段）、主键（稀疏索引）和合并过程。

3. 了解并能够区分 `ReplicatingMergeTree`、`SummingMergeTree` 和 `AggregatingMergeTree` 的应用场景。

4. 了解 Log、Integration 和 Special 引擎家族的典型成员及其用途。

5. 通过实践，能够为不同场景选择并创建合适的表引擎。

### 3.1. **表引擎简介：ClickHouse 的心脏**

**为什么需要表引擎？**
在传统数据库中，数据的存储和管理方式通常是固定的。但 ClickHouse 面对的场景千变万化：有时是海量的、需要长期保存的日志数据；有时是临时的、用完即弃的中间数据；有时数据甚至不在 ClickHouse 本身，而是在远端的 Kafka 或 S3 上。

为了应对这些多样化的需求，ClickHouse 聪明地将\*\*“数据如何存储和管理”**这个功能，做成了一个个可插拔的模块，这就是**表引擎\*\*。

**生动的比喻：不同材质的储物箱**

* **`MergeTree` 引擎:** 一个带有自动整理、压缩、打标签功能的**智能保险柜**。适合存放最重要、最庞大的核心数据。

* **`Log` 引擎:** 一个普通的**纸箱**。东西可以快速扔进去，但找起来很麻烦，也没有整理功能。适合存放临时、不重要的小批量数据。

* **`Memory` 引擎:** 一个放在桌面上的**透明托盘**。存取速度极快，但电脑一关机（服务重启），里面的东西就全没了。适合存放需要高速访问的临时数据。

* **`Kafka` 引擎:** 一个神奇的**传送带**。它本身不存储东西，而是直接连接到 Kafka 的生产线，让你能实时看到生产线上的物品。"数据联邦"

### &#xA;3.2. **王者家族：MergeTree**

大家好！在上一章，我们学会了驾驶 ClickHouse 这辆“F1赛车”，并成功地在数据赛道上跑了几圈。我们当时创建了一张表，用了一个叫做 ENGINE = MergeTree() 的东西。
大家有没有想过，为什么 ClickHouse 的 CREATE TABLE 语句里，ENGINE 是一个必填项，而在我们熟悉的 MySQL 中却可以省略（默认 InnoDB）？
这就是今天我们要探索的核心秘密：**表引擎 (Table Engine)**。如果说 ClickHouse 是一辆赛车，那么表引擎就是它的**发动机**。不同的发动机，决定了这辆车是适合跑直线加速赛，还是适合跑崎岖的山路拉力赛。
今天，我们将一起打开发动机盖，深入研究 ClickHouse 最强大、最核心的“V12涡轮增压发动机”—— **MergeTree 家族**。理解了它，你就掌握了 ClickHouse 80% 的性能奥秘！

`MergeTree`<span style="color: inherit; background-color: rgb(251,191,188)"> 及其变种，是 ClickHouse 中最先进、功能最强大的表引擎，专为海量数据的插入和高性能分析而设计。几乎所有生产环境的核心业务表都应该使用 </span>`MergeTree`<span style="color: inherit; background-color: rgb(251,191,188)"> 家族。</span>

```sql
CREATE TABLE learning.t_web_hits
( ... )
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp) -- 分区键
ORDER BY (timestamp, user_id);   -- 排序键/主键
```

#### 1. **分区 (Partition) & 数据片段 (Data Part)**

* **分区 (`PARTITION BY`):** 就像一个大书柜里，按照年份把书分成不同的格子（例如 `2022年`、`2023年`）。当你查询 `WHERE toYYYYMM(timestamp) = 202310` 时，ClickHouse 只需要打开 `202310` 这个分区格子，极大地减少了扫描范围。

* **数据片段 (`Data Part`):** 每次 `INSERT` 操作，ClickHouse 都会生成一个新的、独立的**数据片段 (Part)**，存储在对应的分区目录下。每个 Part 都是一小组**按列存储**的文件，并且内部已经按照 `ORDER BY` 的规则排好序了。

![](<images/ClickHouse 高性能实时分析数据库-image-5.png>)

#### 2. **主键/排序键 (Primary Key / `ORDER BY`) 与稀疏索引**

* **`ORDER BY` 才是真正的“指挥官”**: 它规定了每个数据片段内部的数据物理排序规则。**这是 ClickHouse 查询性能的灵魂！**

* **主键 (Primary Key):** 在 `MergeTree` 中，主键就是 `ORDER BY` 定义的键（或者其前缀）。但它和 MySQL 的主键完全不同！它不是唯一的，而是用来创建**稀疏索引 (Sparse Index)** 的。

* **稀疏索引的工作方式:** 想象一本很厚的英文字典。

  * **MySQL (密集索引):** 目录里有书中**每一个单词**及它所在的页码。

  * **ClickHouse (稀疏索引):** 目录里只有**每隔10页的第一个单词**及页码（比如 A, An, Apple...）。如果你要找 `Ant`，索引会告诉你“它在 `An` 和 `Apple` 之间”，你只需要去扫描这两页之间的少量数据，而不用翻整本字典。

  ![](<images/ClickHouse 高性能实时分析数据库-image-6.png>)

这个索引文件很小，可以常驻内存，查找极快。这就是为什么**选择正确的 `ORDER BY` 键至关重要**的原因。你应该把你最常用于 `WHERE` 条件过滤的列放在 `ORDER BY` 的最前面。

> **[<span style="color: rgb(216,57,49); background-color: rgba(183,237,177,0.8)">稀疏索引详细文章</span>](https://jvxlv7umxg.feishu.cn/docx/QbuydYGUyoXRZfxo1ZnciiMrn7f?from=from_copylink)<span style="color: rgb(216,57,49); background-color: rgba(183,237,177,0.8)">                                                                                                                                     </span>**

#### 3. **合并 (Merge Process)**


随着 `INSERT` 次数增多，小的 `Data Part` 会越来越多，这会影响查询性能（因为查询时需要聚合所有 Part 的结果）。ClickHouse 后台有一个**合并线程(手动合并)**，会定期地、智能地选择同一个分区内的一些小 Part，把它们合并成一个更大的、有序的新 Part，然后删除掉旧的 Part。

这个过程就是 `MergeTree` 名字的由来：**它在后台不断地合并（Merge）数据，形成一棵更加健康的“数据之树 (Tree)”**。

### 3.3 表引擎

#### 1. **什么是表引擎？**

* **定义**：表引擎决定了数据的存储方式、存储位置、并发访问支持（读/写）、索引支持、多线程请求以及数据复制等。它就像是表的“驱动程序”。

* **与传统数据库对比**：MySQL 有 InnoDB, MyISAM 等；ClickHouse 则提供了几十种，核心思想是“专事专办”，为不同场景提供最优解。

* **查看所有支持的引擎**：

* Generated sql

```plain&#x20;text
SHOW ENGINES;
```

**&#x20;表引擎的四大分类**

* **MergeTree 家族**：ClickHouse 的核心，用于海量数据分析。

* **Log 家族**：用于轻量级、快速写入的临时数据场景。

* **Integration 引擎**：用于与外部系统（如 MySQL, Kafka, S3）集成。

* **Special 引擎**：用于特殊用途（如 Memory, Distributed, MaterializedView）。

#### 2. log家族(轻量级数据写入)

Log 家族是 ClickHouse 中最简单的表引擎，适用于那些“一次写入、多次读取”的轻量级场景。它们的设计目标是快速地将数据追加到磁盘，结构非常简单。

##### ① 核心特点

* **不支持索引**：无法利用索引进行快速查询。

* **并发写锁定**：当有写操作时，表会被锁定，所有读写操作都会被阻塞，因此不适合高并发写入。

* **原子性写入**：数据写入是原子性的，要么成功，要么失败。

* **追加式写入**：数据总是以块的形式追加到文件末尾。

##### ② 适用场景

* 临时存储中间计算结果。

* 小批量、非核心业务的日志快速写入。

* 数据量不大（通常建议百万行以下）的配置表或元数据表。

* **警告**：绝对不要在生产环境的核心分析业务中使用 Log 家族引擎！

最简单的引擎，它将每一列数据存储在不同的压缩文件中。没有并发控制，适合单线程写入。

```sql
-- 创建一个 TinyLog 表
CREATE TABLE tiny_log_table (
    timestamp DateTime,
    level String,
    message String
) ENGINE = TinyLog;

-- 插入数据
INSERT INTO tiny_log_table VALUES (now(), 'INFO', 'User logged in') ;
INSERT INTO tiny_log_table VALUES (now(), 'WARN', 'Disk space is low') ;

-- 查询数据 (会进行全表扫描)
SELECT * FROM tiny_log_table WHERE level = 'WARN';
-rw-r-----. 1 clickhouse clickhouse 124 Jul 29 18:05 level.bin
-rw-r-----. 1 clickhouse clickhouse 174 Jul 29 18:05 message.bin
-rw-r-----. 1 clickhouse clickhouse 109 Jul 29 18:05 sizes.json
-rw-r-----. 1 clickhouse clickhouse 120 Jul 29 18:05 timestamp.bin

表的每列以文件的形式存储 , 插入数据的时候就是将数据追加到对应的列文件后面


```

<span style="color: inherit; background-color: rgba(254,212,164,0.8)">Log 和 StripeLog 是 TinyLog 的改进版。它们额外包含一个小型的元数据文件（__marks.mrk），记录了每个数据块的偏移量。这使得它们可以支持并发读取，并且在读取时可以跳过数据块，性能略好于 TinyLog。</span>

* <span style="color: inherit; background-color: rgba(254,212,164,0.8)">Log: 适合处理大量小记录。</span>

* <span style="color: inherit; background-color: rgba(254,212,164,0.8)">StripeLog: 将所有数据存储在一个文件中，更适合处理列数较多、包含大字段的表。</span>

#### 3. MergeTree家族

1. 概述 (Overview)

MergeTree（合并树）家族是 ClickHouse 中最强大、最核心的表引擎，专为海量数据的高性能在线分析（OLAP）而设计。它是生产环境的首选。

* Mermaid 图解核心原理：后台合并

MergeTree 的核心思想是将数据写入多个小的、有序的、不可变的数据部分（Data Parts），然后通过后台线程将这些小部分不断地合并（Merge）成更大的部分。

![](<images/ClickHouse 高性能实时分析数据库-image-7.png>)

* 核心概念

- **主键与排序键 (`ORDER BY`)**：**物理排序键**。数据在磁盘上严格按照 `ORDER BY` 定义的列进行排序。这是稀疏索引的基础，也是性能的关键。

- **分区 (`PARTITION BY`)**：逻辑上将表数据拆分成不同的**目录**。便于数据管理，如快速删除旧分区 (`ALTER TABLE ... DROP PARTITION`)。

- **稀疏索引 (Primary Key)**：基于排序键，每隔N行（`index_granularity`）创建一个索引标记。查询时能快速定位到可能包含目标数据的“数据块”，极大减少需要扫描的数据量。

- **`ORDER BY`必须声明 , 如果只有order by 字段 , 字段也是主键  可以使用primary key声明主键**

##### 3.1  MergeTree()

**`MergeTree`** 引擎和 **`MergeTree`** 系列的其他引擎（例如 **`ReplacingMergeTree`**、**`AggregatingMergeTree`** ）是 ClickHouse 中最常用和最健壮的表引擎。

**`MergeTree`** 系列表引擎专为高数据摄取速率和巨大数据量而设计。插入作创建表部件，这些部件由后台进程与其他表部件合并。

**`MergeTree`** 系列表引擎的主要特性。

* 表的主键确定每个表部分（聚集索引）中的排序顺序。主键也不引用单个行，而是引用称为颗粒的 8192 行块。这使得大型数据集的主键足够小，可以继续加载在主内存中，同时仍然提供对磁盘数据的快速访问。

* 可以使用任意分区表达式对表进行分区。分区修剪可确保在查询允许时从读取中省略分区。

* 数据可以跨多个集群节点复制，以实现高可用性、故障转移和零停机升级。请参阅[数据复制 ](https://clickhouse.com/docs/engines/table-engines/mergetree-family/replication)。

* **`MergeTree`** 表引擎支持多种统计类型和采样方式，帮助查询优化。

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [[NOT] NULL] [DEFAULT|MATERIALIZED|ALIAS|EPHEMERAL expr1] [COMMENT ...] [CODEC(codec1)] [STATISTICS(stat1)] [TTL expr1] [PRIMARY KEY] [SETTINGS (name = value, ...)],
    name2 [type2] [[NOT] NULL] [DEFAULT|MATERIALIZED|ALIAS|EPHEMERAL expr2] [COMMENT ...] [CODEC(codec2)] [STATISTICS(stat2)] [TTL expr2] [PRIMARY KEY] [SETTINGS (name = value, ...)],
    ...
    INDEX index_name1 expr1 TYPE type1(...) [GRANULARITY value1],
    INDEX index_name2 expr2 TYPE type2(...) [GRANULARITY value2],
    ...
    PROJECTION projection_name_1 (SELECT <COLUMN LIST EXPR> [GROUP BY] [ORDER BY]),
    PROJECTION projection_name_2 (SELECT <COLUMN LIST EXPR> [GROUP BY] [ORDER BY])
) ENGINE = MergeTree()
ORDER BY expr
[PARTITION BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr
    [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx' [, ...] ]
    [WHERE conditions]
    [GROUP BY key_expr [SET v1 = aggr_func(v1) [, v2 = aggr_func(v2) ...]] ] ]
[SETTINGS name = value, ...]
```

1）PARTITION BY **\[选填]**：分区键，用于指定表数据以何种标 准进行分区。分区键既可以是单个列字段，也可以通过元组的形式使 用多个列字段，同时它也支持使用列表达式。如果不声明分区键，则 ClickHouse会生成一个名为all的分区。合理使用数据分区，可以有效 减少查询时数据文件的扫描范围，更多关于数据分区的细节会在6.2节 介绍。

（2）ORDER B&#x59;**&#x20;\[必填]：**&#x6392;序键，用于指定在一个数据片段内， 数据以何种标准排序。默认情况下主键（PRIMARY KEY）与排序键相 同。排序键既可以是单个列字段，例如ORDER BY CounterID，也可以 通过元组的形式使用多个列字段，例如ORDER BY（CounterID,EventDate）。当使用多个列字段排序时，以ORDER BY（CounterID,EventDate）为例，在单个数据片段内，数据首先会以 CounterID排序，相同CounterID的数据再按EventDate排序。

（3）PRIMARY KEY **\[选填]**：主键，顾名思义，声明后会依照主键 字段生成**一级索引**，用于加速表查询。默认情况下，主键与排序键 (ORDER BY)相同，所以通常直接使用ORDER BY代为指定主键，无须刻 意通过PRIMARY KEY声明。所以在一般情况下，在单个数据片段内，数 据与一级索引以相同的规则升序排列。与其他数据库不同，MergeTree 主键允许存在<span style="color: inherit; background-color: rgb(247,105,100)">重复数据</span>（**ReplacingMergeTree**可以去重）。

（4）SAMPLE BY \[选填]：抽样表达式，用于声明数据以何种标准 进行采样。如果使用了此配置项，那么在主键的配置中也需要声明同 样的表达式，例如：

省略...&#x20;

> `() ENGINE = MergeTree() `
>
> `ORDER BY (CounterID, EventDate, intHash32(UserID) SAMPLE BY intHash32(UserID) `

* **SETTINGS**：index\_granularity \[选填]： index\_granularity对于MergeTree而言是一项非常重要的参数，它表 示索引的粒度，默认值为8192。也就是说，MergeTree的索引在默认情 况下，每间隔8192行数据才生成一条索引，其具体声明方式如下所 示：

> 省略...&#x20;
>
> ) ENGINE = MergeTree()&#x20;
>
> 省略...&#x20;
>
> SETTINGS index\_granularity = 8192;&#x20;

8192是一个神奇的数字，在ClickHouse中大量数值参数都有它的 影子，可以被其整除（例如最小压缩块大小 min\_compress\_block\_size:65536）。通常情况下并不需要修改此参 数，但理解它的工作原理有助于我们更好地使用MergeTree。关于索引 详细的工作原理会在后续阐述。

（6）SETTINGS：index\_granularity\_bytes \[选填]：在19.11版本之前，ClickHouse只支持固定大小的索引间隔，由 index\_granularity控制，默认为8192。在新版本中，它增加了自适应 间隔大小的特性，即根据每一批次写入数据的体量大小，动态划分间 隔大小。而数据的体量大小，正是由index\_granularity\_bytes参数控 制的，默认为10M(10×1024×1024)，设置为0表示不启动自适应功 能。

（7）SETTINGS：enable\_mixed\_granularity\_parts \[选填]：设 置是否开启自适应索引间隔的功能，默认开启。

（8）SETTINGS：merge\_with\_ttl\_timeout \[选填]：从19.6版本 开始，MergeTree提供了数据TTL的功能，关于这部分的详细介绍，将 留到第7章介绍。

（9）SETTINGS：storage\_policy \[选填]：从19.15版本开始， MergeTree提供了多路径的存储策略，关于这部分的详细介绍，同样留 到第7章介绍。

```sql
-- 创建一个标准的 MergeTree 表
CREATE TABLE website_hits (
    EventDate Date,
    CounterID UInt32,
    UserID UInt64,
    URL String,
    Income UInt32
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate) -- 按月分区
ORDER BY (CounterID, EventDate, UserID); -- 按站点ID,日期,用户ID排序

-- 插入数据
INSERT INTO website_hits VALUES ('2025-10-27', 1, 101, '/pageA', 10);
INSERT INTO website_hits VALUES ('2025-10-27', 1, 102, '/pageB', 0);

-- 高效查询 (利用了排序键)
SELECT count() FROM website_hits WHERE CounterID = 1 AND EventDate = '2025-10-27';

optimize table website_hits  ;手动合并数据 
```

MergeTree表引擎中的数据是拥有物理存储的，数据会按照分区目 录的形式保存到磁盘之上，

![](<images/ClickHouse 高性能实时分析数据库-image-8.png>)

一张数据表的完整物理结构分为3个层级，依次是数据表目录、分区目录及各分区下具体的数据文件。接下来就逐一 介绍它们的作用。

（1）**partition**：分区目录，余下各类数据文件（primary.idx、 \[Column].mrk、\[Column].bin等）都是以分区目录的形式被组织存放 的，属于相同分区的数据，最终会被合并到同一个分区目录，而不同分 区的数据，永远不会被合并在一起。

（2）**checksums**.txt：校验文件，使用二进制格式存储。它保存了余下各类文件(primary.idx、count.txt等)的size大小及size的哈希值，用于快速校验文件的完整性和正确性。

（3）**columns**.txt：列信息文件，使用明文格式存储。用于保存此数据分区下的列字段信息，例如：

（4）**count**.txt：计数文件，使用明文格式存储。用于记录当前数 据分区目录下数据的总行数

（5）**primary**.idx：一级索引文件，使用二进制格式存储。用于存放稀疏索引，一张MergeTree表只能声明一次一级索引（通过ORDER BY 或者PRIMARY KEY）。借助稀疏索引，在数据查询的时能够排除主键条 件范围之外的数据文件，从而有效减少数据扫描范围，加速查询速度。

（6）\[**Column**].bin：数据文件，使用压缩格式存储，默认为LZ4压缩格式，用于存储某一列的数据。由于MergeTree采用列式存储，所以每一个列字段都拥有独立的.bin数据文件，并以列字段名称命名（例如 CounterID.bin、EventDate.bin等）。

（7）**data**.bin  存储数据的文件 ,使用二进制进行压缩

（9）**partition**.dat与minmax\_\[Column].idx：如果使用了分区 键，例如PARTITION BY EventTime，则会额外生成partition.dat与 minmax索引文件，它们均使用二进制格式存储。partition.dat用于保 存当前分区下分区表达式最终生成的值；而minmax索引用于记录当前分 区下分区字段对应原始数据的最小和最大值。例如EventTime字段对应 的原始数据为2019-05-01、2019-05-05，分区表达式为PARTITION BY toYYYYMM(EventTime)。partition.dat中保存的值将会是2019-05，而 minmax索引中保存的值将会是2019-05-012019-05-05。

在这些分区索引的作用下，进行数据查询时能够快速跳过不必要的 数据分区目录，从而减少最终需要扫描的数据范围。

（10）**skp\_idx\_**\[Column].idx与skp\_idx\_\[Column].mrk：如果在建 表语句中声明了二级索引，则会额外生成相应的二级索引与标记文件， 它们同样也使用二进制存储。二级索引在ClickHouse中又称跳数索引， 目前拥有minmax、set、ngrambf\_v1和tokenbf\_v1四种类型。这些索引 的最终目标与一级稀疏索引相同，都是为了进一步减少所需扫描的数据 范围，以加速整个查询过程。。

##### 3.2 ReplacingMergeTree

在后台合并时，对于排序键相同的行，只保留**最新**的版本。(去重)

这个引擎是在MergeTree的基础上，添加了“处理重复数据”的功能，该引擎和MergeTree的不同之处在于它会删除具有相同主键的重复项。数据的去重只会在合并的过程中出现。合并会在未知的时间在后台进行，所以你无法预先作出计划。有一些数据可能仍未被处理。因此，ReplacingMergeTree适用于在后台清除重复的数据以节省空间，但是它不保证没有重复的数据出现.

* **机制**：可以指定一个版本列（如 `timestamp` 或 `version`），保留版本最大的行；若不指定，则保留最后插入的行。

* **注意**：去重只在合并时发生，查询时可能仍会看到重复数据，直到 `OPTIMIZE ... FINAL` 执行完毕。

```sql
-- 创建一个带版本号的 ReplacingMergeTree 表
CREATE TABLE user_profiles (
    user_id UInt64,
    profile String,
    updated_ts DateTime
    --ReplacingMergeTree() 没有指定版本 , 按照数据插入时间进行判断取舍
) ENGINE = ReplacingMergeTree(updated_ts) -- updated_ts 是版本列
ORDER BY user_id;

-- 插入旧版本数据
INSERT INTO user_profiles VALUES (1, '{"city": "Beijing"}', '2025-01-01 10:00:00');
-- 插入新版本数据
INSERT INTO user_profiles VALUES (1, '{"city": "Shanghai"}', '2025-01-01 11:00:00');

-- 此时查询可能看到两条
SELECT * FROM user_profiles;

-- 强制合并
OPTIMIZE TABLE user_profiles FINAL;

-- 再次查询，只剩新版本
SELECT * FROM user_profiles;
```

##### 3.3 SummingMergeTree

在后台合并时，对于排序键相同的行，会将所有**数值类型**的列进行累加。

```sql
-- 创建一个 SummingMergeTree 表
CREATE TABLE daily_stats (
    day Date,
    page_id UInt32,
    visits UInt64,
    clicks UInt64
) ENGINE = SummingMergeTree()
ORDER BY (day, page_id);

-- 插入多批数据
INSERT INTO daily_stats VALUES ('2025-10-27', 1001, 10, 1);
INSERT INTO daily_stats VALUES ('2025-10-27', 1001, 15, 3); 
INSERT INTO daily_stats VALUES ('2025-10-27', 1002, 8, 8); 
INSERT INTO daily_stats VALUES ('2025-10-27', 1002, 8, 8); 
INSERT INTO daily_stats VALUES ('2025-10-27', 1003, 9, 9); 
-- 同样 day 和 page_id-- 强制合并
OPTIMIZE TABLE daily_stats FINAL;

-- 查询结果是自动累加的
- visits 会变成 10 + 15 = 25
- clicks 会变成 1 + 3 = 4
SELECT * FROM daily_stats;

```

##### 3.4 AggregatingMergeTree

当需要进行更复杂的预聚合（如 `avg`, `uniq`, `quantile`）时使用。

AggregatingMergeTree就有些许**数据立方体**的意思，它能够在合并分区的时候，按照预先定义的条件聚合数据。同时，根据预先定义的聚合函数计算数据并通过**二进制的格式**存入表内。将同一分组下的多行数据聚合成一行，既减少了数据行，又降低了后续聚合查询的开销。可以说，AggregatingMergeTree 是**SummingMergeTree的升级版**，它们的许多设计思路是一致的，例如同时定义 ORDER BY与PRIMARY KEY的原因和目的。但是在使用方法上，两者存在明显差异，应该说AggregatingMergeTree的定义方式是MergeTree家族中最为特殊的一个。

* **机制**：它存储聚合函数的“中间状态”（**State**），而不是最终值。查询时需要使用 `-Merge` 函数来获取最终结果。

* **语法**：

  * 建表：列类型为 **`AggregateFunction`**`(function_name, arg_types)`

  * 插入：使用 `-State` 函数，如 **`uniqState`**`(user_id)`

  * 查询：使用 `-Merge` 函数，如 `uniqMerge(state_column)`

AggregatingMergeTree没有任何额外的设置参数，在分区合并时，在每个数据分区内，会按照ORDER BY聚合。而使用何种聚合函数，以及针对哪些列字 段计算，则是通过定义AggregateFunction数据类型实现的。在insert和select时，也有独特的写法和要求：写入时需要使用-State语法，查询时使用-Merge语法。

**AggregateFunction(arg1 , arg2) ;**

参数一 聚合函数

参数二 数据类型

sum\_cnt AggregateFunction(sum, Int64) ;

先创建原始表 ---插入数据---> 创建预先聚合表 --通过Insert的方式导入数据, 数据会按照指定的聚合函数聚合预先数据!

```sql
-- 1)建立明细表
CREATE TABLE detail_table
(id UInt8,
 ctime Date,
 money UInt64
) ENGINE = MergeTree() 
PARTITION BY toDate(ctime) 
ORDER BY id;

-- 2)插入明细数据INSERT INTO detail_table VALUES(1, '2021-08-06', 100);
INSERT INTO detail_table VALUES(1, '2025-08-06', 100);
INSERT INTO detail_table VALUES(1, '2025-08-06', 300);
INSERT INTO detail_table VALUES(2, '2025-08-07', 200);
INSERT INTO detail_table VALUES(2, '2025-08-07', 200);

-- 3)建立预先聚合表，-- 注意：其中UserID一列的类型为：AggregateFunction(uniq, UInt64)
CREATE TABLE agg_table
(id UInt8,
ctime Date,
cnt AggregateFunction(count, UInt64) ,
total_money AggregateFunction(sum, UInt64) ,
) ENGINE = AggregatingMergeTree() 
PARTITION BY  toDate(ctime) 
ORDER BY id;

-- 4) 从明细表中读取数据，插入聚合表。
-- 注意：子查询中使用的聚合函数为 uniqState， 对应于写入语法<agg>-State
INSERT INTO agg_table
select 
id, 
ctime, 
countState(id) ,
sumState(money)
from detail_table
group by id, ctime ;

-- 不能使用普通insert语句向AggregatingMergeTree中插入数据。
-- 本SQL会报错：Cannot convert UInt64 to AggregateFunction(uniq, UInt64)INSERT INTO agg_table VALUES(1, '2020-08-06', 1);

-- 5) 从聚合表中查询。
-- 注意：select中使用的聚合函数为uniqMerge，对应于查询语法<agg>-Merge
SELECT
    id,
    ctime,
    countMerge(cnt),
    sumMerge(total_money)
FROM agg_table
GROUP BY
    id,
    ctime ;
  
     ┌─id─┬──────ctime─┬─countMerge(cnt)─┬─sumMerge(total_money)─┐
1. │  1 │ 2025-08-06 │               9 │                  1500 │
2. │  2 │ 2025-08-07 │               6 │                  1200 │
   └────┴────────────┴─────────────────┴───────────────────────┘
```

总结:

> （1）用ORBER BY排序键作为聚合数据的条件Key。
>
> （2）使用AggregateFunction字段类型定义聚合函数的类型以及聚合的字 段。
>
> （3）只有在合并分区的时候才会触发聚合计算的逻辑。
>
> （4）以数据分区为单位来聚合数据。当分区合并时，同一数据分区内聚合 Key相同的数据会被合并计算，而不同分区之间的数据则不会被计算。
>
> （5）在进行数据计算时，因为分区内的数据已经基于ORBER BY排序，所以 能够找到那些相邻且拥有相同聚合Key的数据。
>
> （6）在聚合数据时，同一分区内，相同聚合Key的多行数据会合并成一 行。对于那些非主键、非AggregateFunction类型字段，则会使用第一行数据的 取值。
>
> （7）AggregateFunction类型的字段使用二进制存储，在写入数据时，需 要调用*State函数；而在查询数据时，则需要调用相应的*Merge函数。其中，\* 表示定义时使用的聚合函数。
>
> （8）AggregatingMergeTree通常作为物化视图的表引擎，与普通 MergeTree搭配使用。
>
> 该查询尝试使用\[MergeTree]系列中的表引擎初始化表的未计划的数据部分合并。\[MaterializedView和\[Buffer]引擎OPTMIZE也支持。不支持其他表引擎。
>
> 当OPTIMIZE与使用\[ReplicatedMergeTree]表引擎，ClickHouse创造了合并，并等待所有节点上执行（如果该任务replication\_alter\_partitions\_sync已启用设置）。
>
> · 如果OPTIMIZE由于任何原因未执行合并，则不会通知客户端。要启用通知，请使用\[optimize\_throw\_if\_noop]设置。
>
> · 如果指定PARTITION，则仅优化指定的分区。\[如何设置分区表达式]。
>
> · 如果指定FINAL，即使所有数据已经在一个部分中，也会执行优化。
>
> · 如果指定DEDUPLICATE，则将对完全相同的行进行重复数据删除（比较所有列），这仅对MergeTree引擎有意义。



![](<images/ClickHouse 高性能实时分析数据库-diagram.png>)



**注意: `AggregatingMergeTree`**:

* 这个引擎是专门为“增量聚合”设计的。

* 它不会直接存储像`100`这样的最终结果，而是存储一种**中间状态**（`AggregateFunction`类型）。

* 比如，对于`count`，它存储的是计数的中间状态；对于`uniq`，它存储的是去重计数的中间状态（通常是HyperLogLog算法的草图）。

* 这样做的好处是，当ClickHouse在后台合并数据片时，它可以把这些**中间状态也进行合并**，从而得到一个全局正确的聚合结果，而不会重复计算。



##### **<span style="color: rgb(216,57,49); background-color: inherit">3.5 使用物化视图同步聚合数据</span>**

**核心思想**：物化视图就像一个勤劳的、自动化的“帮厨”，它在你看不见的地方，默默地把原始数据进行预处理（聚合、转换），然后存放到一个“立即可用”的目标表中。这样，当你需要查询分析结果时，直接从这个目标表取数据，速度快得飞起！

和其他数据库（如PostgreSQL）的物化视图不同，ClickHouse的物化视图有一个非常关键的特点：**它是一个“触发器”**。

它本身不存储数据，而是像一个**数据转换的管道**。当数据被`INSERT`到原始表时，物化视图会像一个检查站一样，把这批数据“拦截”下来，按照你预设的规则（SQL查询）进行转换和聚合，然后塞入到你指定的另一张**目标表**中。

![](<images/ClickHouse 高性能实时分析数据库-image-9.png>)

**流程分解：**

1. **数据写入 (INSERT)**：你的应用程序向`原始数据表` (Source Table) 中插入一批新的数据。

2. **触发物化视图 (Trigger)**：这个`INSERT`动作会立刻触发与`原始数据表`绑定的`物化视图`。

3. **数据转换 (Transform)**：物化视图执行它内部定义的`SELECT`查询逻辑，对这批新插入的数据进行实时转换或聚合。

4. **写入目标表 (Load)**：转换后的结果被自动写入到`目标表` (Target Table) 中。

5. **数据查询 (SELECT)**：当你要查询分析结果时，你**直接查询的是目标表**，而不是物化视图本身。因为目标表里存放的已经是预计算好的、高度聚合的数据，所以查询速度极快。

**关键点总结：**

* **它是个触发器**：由对源表的`INSERT`操作触发。

* **它不存数据**：它只是一个转换规则的定义。

* **数据存在目标表**：真正的“物化”结果存储在另一张独立的表中。

* **查询目标表**：我们最终查询的是目标表，享受预计算带来的性能提升。

![](<images/ClickHouse 高性能实时分析数据库-diagram-1.png>)

**第一步创建原始表&#x20;**

```sql
-- 原始访问日志表
CREATE TABLE raw_logs (
    log_time    DateTime,      -- 访问时间
    user_id     UInt64,        -- 用户ID
    url         String         -- 访问的URL
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(log_time) -- 按月分区
ORDER BY log_time;

```

**第二步创建聚合结果表&#x20;**

```sql
-- 每日流量聚合表
CREATE TABLE daily_summary (
    summary_date Date,                         -- 统计日期
    pv           AggregateFunction(count, UInt64),     -- PV (页面浏览量)
    uv           AggregateFunction(uniq, UInt64) -- UV (独立访客数)
) ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(summary_date)
ORDER BY summary_date;
```

**第三步创建物化视图&#x20;**

```sql
-- 创建物化视图，将原始日志实时聚合到每日汇总表
CREATE MATERIALIZED VIEW mv_daily_summary TO daily_summary AS
SELECT
    toDate(log_time) AS summary_date, -- 将时间转换为日期
    countState() AS pv,               -- 计算PV的中间状态
    uniqState(user_id) AS uv          -- 计算UV的中间状态
FROM raw_logs
GROUP BY summary_date;
```

代码解读:

CREATE MATERIALIZED VIEW mv\_daily\_summary: 定义物化视图的名称。

TO daily\_summary: 这是关键！ 它明确告诉物化视图，转换后的数据应该写入到daily\_summary表中。

AS SELECT ...: 这是数据转换的逻辑。

toDate(log\_time): 从访问时间中提取出日期。

countState(): 对应AggregateFunction(count)，计算计数的状态。

uniqState(user\_id): 对应AggregateFunction(uniq, UInt64)，计算user\_id去重的状态。

GROUP BY summary\_date: 按天进行聚合。

\*State函数：这些函数（如 countState, uniqState, sumState）是专门与 AggregatingMergeTree 配合使用的，它们生成聚合的中间状态，而不是最终值。

**第四步插入数据&#x20;**

```sql
-- 插入一些今天的日志数据
INSERT INTO raw_logs VALUES
(now(), 101, '/page/a'),
(now(), 102, '/page/b'),
(now(), 101, '/page/c'), 
(now(), 103, '/page/a');
```



```sql
-- 插入一些昨天的日志数据
INSERT INTO raw_logs  VALUES
(now() - INTERVAL 1 DAY, 201, '/home'),
(now() - INTERVAL 1 DAY, 202, '/home'),
(now() - INTERVAL 1 DAY, 201, '/profile');
```



**第五步查询结果表**

```sql
 SELECT
    summary_date,
    countMerge(pv) AS total_pv, -- 从状态中合并计算出最终的PV
    uniqMerge(uv) AS total_uv   -- 从状态中合并计算出最终的UV
FROM daily_summary
GROUP BY summary_date
ORDER BY summary_date ;

   ┌─summary_date─┬─total_pv─┬─total_uv─┐
1. │   2025-07-18 │        3 │        2 │
2. │   2025-07-19 │        4 │        3 │
   └──────────────┴──────────┴──────────┘
```

> **高级话题**

1. **历史数据回填 (`POPULATE`)**
   默认情况下，物化视图只对**新插入**的数据生效。如果想让它把历史数据也处理一遍，可以在创建时加上`POPULATE`关键字：

```plain&#x20;text
CREATE MATERIALIZED VIEW mv_daily_summary TO daily_summary
POPULATE -- <<<<<<<<<<<<<<<< 加上这个AS SELECT ...
```

1. **警告**：`POPULATE`会对源表进行一次全量扫描，如果源表数据量巨大，会非常消耗资源和时间，请在业务低峰期执行。

2. **物化视图无法修改 (`ALTER`)**
   一旦创建，物化视图的`SELECT`逻辑就不能修改了。如果业务逻辑变更，你只能：

   * `DROP VIEW mv_daily_summary;` (删除旧的)

   * `CREATE MATERIALIZED VIEW ...` (创建新的)

3. **目标表引擎的选择**

   * **`AggregatingMergeTree`**: 最常用，适合需要去重（`uniq`）、计算平均值（`avg`）等复杂聚合的场景。

   * **`SummingMergeTree`**: 如果你的聚合逻辑只有求和（`sum`）和计数（`count`），可以用它，更简单高效。

   * **普通`MergeTree`**: 如果你的物化视图只是对数据做一些转换（比如字段清洗、格式化），而没有聚合，那么目标表用普通的`MergeTree`即可。

##### 3.6 CoalescingMergeTree

引擎继承自 [MergeTree](https://clickhouse.com/docs/engines/table-engines/mergetree-family/versionedcollapsingmergetree)。不同之处在于，在合并 **`CoalescingMergeTree`** 表的数据部分时，ClickHouse 将具有相同主键（或更准确地说，具有相同[排序键 ](https://clickhouse.com/docs/engines/table-engines/mergetree-family/mergetree)）的所有行替换为包含每列的最新非空值的行。如果排序键的组成方式是单个键值对应大量行，则可以显著减少存储量并加快数据选择速度。

我们建议将引擎与 **`MergeTree`** 一起使用。将完整数据存储在 **`MergeTree`** 表中，并使用 **`CoalescingMergeTree`** 进行聚合数据存储，例如，在准备报表时。这种方法将防止您因主键组合不正确而丢失有价值的数据。

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = CoalescingMergeTree([columns])
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

* **`columns`** - 一个元组，其中包含将联合值的列的名称。可选参数。列必须是数字类型，并且不能位于分区键或排序键中。

* 如果未指定 **`columns`**，则 ClickHouse 会将不在排序键中的所有列中的值合并。

**<span style="color: rgb(216,57,49); background-color: inherit">保留数字键的最新数据</span>**

```sql
CREATE TABLE test_table
(
    id UInt32,
    name String ,
    age UInt32 
    
)
ENGINE = CoalescingMergeTree()
ORDER BY id 

INSERT INTO test_table VALUES(1,'zss'  ,21),(1,'zss' , 22),(2,'ls' , 20)   ;
INSERT INTO test_table VALUES(3,2),(4,1),(3,8) ;
SELECT * FROM test_table FINAL;
```

##### 3.7 CollapsingMergeTree

“大家好，我们都知道 ClickHouse 在处理海量数据分析时速度飞快，这得益于它的列式存储和 `MergeTree` 引擎。但 `MergeTree` 主要为追加写入（Append-only）设计。如果我们遇到需要频繁变更或删除少量数据的业务场景，该怎么办？”

* **举例说明:**

  1. **用户行为追踪:** 记录用户进入和离开一个直播间。一条“进入”记录，一条“离开”记录。我们只关心当前在线的用户。

  2. **订单状态流转:** 一个订单从“已创建”变为“已支付”，再变为“已发货”。我们通常只关心订单的最终状态。

  3. **配置项管理:** 一个配置项被创建，然后可能被修改，最后被删除。

* **点出痛点:**

  * 在传统 OLTP 数据库中，我们会用 `UPDATE` 或 `DELETE`。

  * 但在 ClickHouse 中，`ALTER TABLE ... UPDATE/DELETE` 是一个非常“重”的异步操作（**Mutation**），它会重写整个数据分区，不适合高频、小范围的修改。

* **引出解决方案:**

  * “为了解决这类问题，ClickHouse 提供了一系列特殊的表引擎，它们通过一种巧妙的方式在应用层模拟了‘更新’和‘删除’。今天，我们就来深入学习其中最经典的一个——`CollapsingMergeTree`。”

> **`CollapsingMergeTree`** 引擎继承自 [MergeTree](https://clickhouse.com/docs/engines/table-engines/mergetree-family/mergetree) 并添加了在合并过程中折叠行的逻辑。 如果排序键 （**`ORDER BY`**） 中的所有字段都相同，则 **`CollapsingMergeTree`** 表引擎会异步删除（折叠）行对，但特殊字段 **`Sign`** 除外，该字段可以具有 **`1`** 或 **`-1`** 的值。保留没有一对相反&#x503C;**`符号`**&#x7684;行。
>
> CollapsingMergeTree就是一种通过**以增代删**的思路，支持行级数 据修改和删除的表引擎。它通过定义一个sign标记位字段，记录数据行 的状态。如果sign标记为1，则表示这是一行有效的数据；如果sign标 记为-1，则表示这行数据需要被删除。当CollapsingMergeTree分区合 并时，同一数据分区内，sign标记为1和-1的一组数据会被抵消删除。 这种1和-1相互抵消的操作，犹如将一张瓦楞纸折叠了一般。这种直观 的比喻，想必也正是折叠合并树（CollapsingMergeTree）名称的由来，
>
> 多行的排序相同的状态为1的数据会折叠成一行 , 保留最后一行
>
> 两行排序相同的数据, 状态为 1 和 -1 删除这两行数据
>
> ENGINE = CollapsingMergeTree(sign)

###### 核心机制

* **核心机制：`Sign` 列**

  * **定义:** `CollapsingMergeTree` 要求表中必须有一个特殊的 `Sign` 列，类型为 `Int8`，且只能存 `1` 或 `-1` 这两个值。

  * **`Sign = 1`:** 代表一个“状态行”或“常规数据行”。可以理解为一次 **“增加”** 操作。

  * **`Sign = -1`:** 代表一个“取消行”。它用于 **“抵消”** 或 **“删除”** 具有相同排序键（`ORDER BY` 字段）的 `Sign = 1` 的行。

* **折叠（Collapsing）的规则:**

  * **触发时机:** 折叠操作只在后台的数据分区合并（Merge）过程中发生。这意味着，刚写入的数据（包括 `+1` 和 `-1` 的行）是同时存在的，不会立即消失。

  * **配对规则:**

    1. 当一个 `Sign = -1` 的行和一个 `Sign = 1` 的行具有 **完全相同的排序键** 时，它们会配对。

    2. 在合并时，如果一个数据块内，相同排序键的 `Sign=1` 和 `Sign=-1` 的行数相等，则它们都会被删除。

    3. **重要:** 如果 `Sign=-1` 的行比 `Sign=1` 的行多，或者 `Sign=1` 的行比 `Sign=-1` 的行多一个，系统会保留 **最后一条 `Sign=1` 的行** 和 **第一条 `Sign=-1` 的行**。这个规则保证了即使数据乱序写入，逻辑也是健壮的。（*讲师注：这个细节比较复杂，可以简化为“成对的会被删除，未成对的会被保留”，重点强调乱序写入可能导致折叠失败*）。

```sql
// 1. 写入 User A 进入 (Sign=1)
user_id | event_time | Sign
---------------------------
'A'     | 10:00      | 1

// 2. 写入 User A 离开 (Sign=-1)
user_id | event_time | Sign
---------------------------
'A'     | 10:00      | 1
'A'     | 10:00      | -1  <-- 在合并前，两条记录都存在！

// 3. 后台 Merge 发生后
(表中无 'A' 的记录)
```

###### **示例**

```sql
-- 创建 CollapsingMergeTree 表
-- Sign 必须是最后一个字段
-- ORDER BY 定义了哪些行是“相同”的，需要一起折叠
CREATE TABLE user_sessions
(
    user_id String,
    session_id String,
    start_time DateTime,
    -- 其他一些维度或指标
    Sign Int8
)
ENGINE = CollapsingMergeTree(Sign)
ORDER BY (user_id, session_id); -- 排序键是折叠的依据！

-- 用户 'Alice' 开始了一个会话
INSERT INTO user_sessions VALUES ('Alice', 'session_1', now(), 1);

-- 此时查询，Alice 是在线的
-- 但是用普通 SELECT 会看到什么？
SELECT * FROM user_sessions;

   ┌─user_id─┬─session_id─┬──────────start_time─┬─Sign─┐
1. │ Alice   │ session_1  │ 2025-07-19 14:49:41 │    1 │
   └─────────┴────────────┴─────────────────────┴──────┘

-- 用户 'Bob' 也开始了会话
INSERT INTO user_sessions VALUES ('Bob', 'session_abc', now(), 1);

-- 用户 'Alice' 的会话结束了
-- 注意：必须插入一条与开始时排序键完全相同的记录，只是 Sign 为 -1
INSERT INTO user_sessions VALUES ('Alice', 'session_1', now(), -1);

-- 再次查询，看看发生了什么？
SELECT * FROM user_sessions ORDER BY user_id;
/*
┌─user_id─┬─session_id─┬──────────start_time─┬─Sign─┐
│ Alice   │ session_1  │ 2023-10-27 10:30:00 │    1 │
│ Alice   │ session_1  │ 2023-10-27 10:31:00 │   -1 │  <-- Alice 的两条记录都还在！
│ Bob     │ session_abc│ 2023-10-30:45       │    1 │
└─────────┴────────────┴─────────────────────┴──────┘
*/

```

&#x20;这是最容易让新手困惑的地方！\`CollapsingMergeTree\` 不会立即删除数据。我们的查询逻辑必须能处理这种情况。

**方法一：`GROUP BY ... HAVING` (推荐的通用方法)**

* “我们如何只看‘当前有效’的会话？可以通过对 `Sign` 求和来判断。”

```sql
SELECT
    user_id,
    session_id,
    max(start_time) AS latest_start_time, -- 可以取任意聚合值
    sum(Sign) AS state
FROM user_sessions
GROUP BY user_id, session_id
HAVING state > 0; -- 或者 sum(Sign) > 0
```

* **优点:** 逻辑清晰，总是返回正确的结果，性能通常不错。

* **缺点:** 写法稍显复杂。

**方法二：使用 `FINAL` 修饰符 (方便但需谨慎)**

* “ClickHouse 提供了一个‘语法糖’来简化这个过程。”

```sql
SELECT * FROM user_sessions FINAL;
```

* **优点:** 写法非常简单。

* **缺点:**

  * **性能开销:** `FINAL` 会在查询时强制在内存中进行合并计算，如果涉及的数据分区很多，会导致查询变慢，并且会消耗更多资源。

  * **并行度限制:** `FINAL` 会让查询以单线程模式执行（在数据合并阶段）。



**手动触发合并&#x20;**

* “我们可以通过 `OPTIMIZE` 命令来手动触发后台合并，模拟真实环境下的折叠过程。”

```sql
OPTIMIZE TABLE user_sessions FINAL;

-- 再次用普通 SELECT 查询
SELECT * FROM user_sessions;

现在 Alice 的记录已经物理上被清除了。这就是 CollapsingMergeTree 最终达成的效果。”
```

###### 总结

* **何时使用 `CollapsingMergeTree`？ (适用场景)**

  1. **对象状态不频繁变化:** 比如用户注册/注销、订阅开始/结束。如果一个对象状态每秒变化几十次，这个模型就不太适合。

  2. **只关心对象的最终状态:** 查询主要是为了获取“当前是什么”，而不是历史变化路径。

  3. **写入逻辑可控:** 应用层程序能够保证，每次状态变更都能正确地写入 `+1` 和 `-1` 的配对行。

  4. **软删除:** 实现数据的“软删除”是一个绝佳的场景。

* **何时避免使用 `CollapsingMergeTree`？ (不适用场景)**

  1. **需要查看历史版本:** 如果你想知道 Alice 是在什么时间进入，又在什么时间离开的，这个引擎不直接提供。

  2. **数据写入顺序无法保证:** `CollapsingMergeTree` 对行的写入顺序有一定敏感性，如果 `-1` 的行比 `+1` 的行先到，折叠可能不会按预期发生。（*补充：`VersionedCollapsingMergeTree` 解决了这个问题*）。

  3. **高频更新:** 写入量巨大且状态变化极快，会导致未合并的数据部分非常臃肿，查询性能下降。

* **最佳实践 & 对比**

  1. **查询首选 `GROUP BY ... HAVING`:** 性能更可控，更稳定。仅在小表或对性能要求不高的临时查询中使用 `FINAL`。

  2. **`ORDER BY` 键的选择:** 必须包含能够唯一标识一个“可变实体”的所有字段。

  3. **与 `ReplacingMergeTree` 对比:**

     * `ReplacingMergeTree`<span style="color: rgb(216,57,49); background-color: inherit"> </span>更简单，它只保留排序键相同的最后一条记录，**适合“更新”场景**，但不适合“删除”。

     * **`CollapsingMergeTree`&#x20;**&#x901A;过 `Sign` 列，明确地支持了“删除”或“抵消”的语义。

##### 3.8 **VersionedCollapsingMergeTree**

在上一节课中，我们学习了 `CollapsingMergeTree`，它通过 `Sign` 列（+1 和 -1）巧妙地实现了数据的‘软删除’和状态变更。但它有一个致命的弱点，大家还记得是什么吗？” (引导学员回答：对写入顺序敏感)。

* “想象一个分布式系统，比如 Kafka。由于网络延迟或分区 **rebalance**，消息的顺序可能会被打乱。”

![](<images/ClickHouse 高性能实时分析数据库-diagram-2.png>)

* **场景:** 用户 Alice 的会话。

  1. `T1` 时刻: 会话开始 (`session_id=s1, Sign=1`)。

  2. `T2` 时刻: 会话结束 (`session_id=s1, Sign=-1`)。

* **问题:** 如果“会话结束”的消息 (`Sign=-1`) 比“会话开始”的消息 (`Sign=1`) 先到达 ClickHouse，会发生什么？

* **推演 `CollapsingMergeTree` 的失败过程:**

  1. **写入:** `(s1, -1)` 先被写入。

  2. **写入:** `(s1, +1)` 后被写入。

  3. **合并:** 在后台合并时，ClickHouse 的规则是“保留第一条 `-1` 的行和最后一条 `+1` 的行”。因为这两条记录不在同一个批次，或顺序不对，它们可能都无法找到配对，最终导致两条记录都被保留下来。

  4. **查询:** `SELECT ... GROUP BY ... HAVING sum(Sign) > 0` 的结果可能是错误的，或者 `FINAL` 查询也无法正确折叠。最终 Alice 的会话可能被错误地认为是“活跃”的。

* “为了解决这个在真实世界中非常常见的问题，ClickHouse 提供了 `CollapsingMergeTree` 的增强版——`VersionedCollapsingMergeTree`。它增加了一个维度来保证无论数据何时到达，逻辑都是正确的。”

###### **核心机制：`Version` 列**

* **定义:** `VersionedCollapsingMergeTree` 除了 `Sign` 之外，还要求一个 `Version` 列。这个列必须是数值类型（如 `UInt8`, `UInt32`, **`UInt64`**）。

* **作用:** `Version` 列代表了数据的“版本号”。它可以是时间戳、序列ID或任何单调递增的数字。

* **新的折叠（Collapsing）规则:**

  * **规则1 (删除):** `Sign = -1` 的行只会取消具有 **相同排序键** 和 **相同 `Version`** 的 `Sign = 1` 的行。

  * **<span style="color: rgb(216,57,49); background-color: inherit">规则2 (更新/保留最新):</span>**<span style="color: rgb(216,57,49); background-color: inherit"> 对于具有相同排序键但 </span>`Version`<span style="color: rgb(216,57,49); background-color: inherit"> 不同的行，只有 </span>**`Version`<span style="color: rgb(216,57,49); background-color: inherit"> 最高</span>**<span style="color: rgb(216,57,49); background-color: inherit"> 的那一行（如果 </span>`Sign=1`<span style="color: rgb(216,57,49); background-color: inherit">）会被保留。</span>

  * **组合起来的逻辑:** 这个引擎同时具备了 `CollapsingMergeTree`（删除）和 `ReplacingMergeTree`（保留最新版本）的特性，但比它们都更健壮。

`ENGINE = VersionedCollapsingMergeTree(Sign, Version)`，顺序不能错。`Version` 列的数据源必须是可靠的、单调递增的。

###### 示例

* **场景设定: 一个健壮的配置管理系统。配置项可以被创建、更新和删除，并且操作指令可能从不同的地方乱序到达。**

```sql
-- Version 列必须作为引擎的第二个参数
CREATE TABLE app_configs
(
    config_key String,
    config_value String,
    -- 使用时间戳作为版本号，非常常见的做法
    Version UInt64,
    Sign Int8
)
ENGINE = VersionedCollapsingMergeTree(Sign, Version)
ORDER BY config_key; -- 排序键依然是对象的唯一标识


-- 1. 创建配置项 'timeout'
-- 假设当前时间戳是 1677610000
INSERT INTO app_configs VALUES ('timeout', '30s', 1677610000, 1);

-- 2. 创建配置项 'retries'
INSERT INTO app_configs VALUES ('retries', '3', 1677610010, 1);

-- 3. 更新配置项 'timeout' 为 '60s' (关键操作！)
-- 假设新时间戳是 1677610025
-- 必须插入两条！
INSERT INTO app_configs VALUES('timeout', '30s', 1677610000, -1),
('timeout', '60s', 1677610025, 1);  

-- 查询未合并的数据
SELECT * FROM app_configs ORDER BY config_key, Version;
```

* **核心要点总结:**

  1. **解决了什么？** 乱序写入导致的数据不一致问题。

  2. **如何解决？** 引入 `Version` 列，让抵消和替换操作有了明确的版本依据。

  3. **<span style="color: rgb(216,57,49); background-color: inherit">更新操作的铁律:</span>**<span style="color: rgb(216,57,49); background-color: inherit"> 必须是 </span>**<span style="color: rgb(216,57,49); background-color: inherit">“取消旧版 + 创建新版”</span>**<span style="color: rgb(216,57,49); background-color: inherit"> 的原子写入。应用层逻辑必须保证这一点。</span>

  4. **`Version` 的来源:** 必须是单调递增的。Unix 时间戳、数据库序列、业务事件 ID 都是很好的选择。

* **与其它引擎的终极对比:**

#### 4. 集成引擎- 打破数据孤岛，连接万物

##### **1 课程基本信息**

* **课程主题:** ClickHouse 集成引擎（Integration Engines）深度解析与应用

* **先修要求:**

  * 熟悉 ClickHouse 基础，包括 `MergeTree` 引擎。

  * 对常见的外部数据系统（如 MySQL, S3, Kafka）有基本概念。

* **目标学员:**

  * 希望将 ClickHouse 与现有数据生态（如 OLTP 数据库、数据湖、消息队列）无缝集成的数据工程师。

  * 寻求减少复杂 ETL 流程，实现**数据联邦**查询（Data Federation）的架构师。

* **课程时长:** 约 90 分钟

* **教学目标:** 学员在课程结束后，应能够：

  1. **阐述** ClickHouse 集成引擎的核心价值：**原地查询** 和 **简化数据集成**。

  2. **区分** 不同类型的集成引擎及其适用场景（数据库、文件系统、消息队列）。

  3. **实践** 使用 `MySQL` 引擎，实现对线上 OLTP 数据库的联邦查询。

  4. **实践** 使用 `S3` 引擎，直接查询存储在对象存储上的数据文件。

  5. **掌握** 使用 `Kafka` 引擎结合物化视图（Materialized View），构建强大的实时数据摄取管道。

  6. **分析** 使用集成引擎的性能考量和最佳实践。

* “在真实的企业环境中，数据往往不是集中存放在一个地方的。我们的用户信息可能在 `MySQL` 或 `PostgreSQL` 里，日志文件可能存储在 `S3` 或 `HDFS` 上，而实时的业务事件则通过 `Kafka` 传输。要把这些数据汇集到 ClickHouse 进行分析，传统的方法是什么？”

* **引导学员回答:** ETL (Extract-Transform-Load)。

* **图解传统 ETL 流程:**

  * “ETL 流程通常需要独立的工具（如 Spark, Flink, Airflow, Kettle），它有几个明显的缺点：

    1. **延迟高：** T+1 或 T+H，无法实时分析。

    2. **链路复杂：** 需要开发和维护额外的ETL任务。

    3. **存储冗余：** 同一份数据在源系统和 ClickHouse 中各存一份。”

![](<images/ClickHouse 高性能实时分析数据库-image-10.png>)

* “ClickHouse 的集成引擎提出了一种全新的思路：**为什么不让 ClickHouse 直接去查询这些外部系统呢？** 这就是数据联邦的核心思想——查询发生在数据原地，无需（或极大简化）数据移动。”

![](<images/ClickHouse 高性能实时分析数据库-image-11.png>)

##### 2 核心概念

* **定义:** ClickHouse 集成引擎是一类特殊的表引擎，它不把数据存储在 ClickHouse 本地，而是充当一个\*\*“代理”或“连接器”\*\*。当你查询一个集成引擎表时，ClickHouse 会：

  1. 解析你的 SQL 查询。

  2. 将查询转换为对外部系统的请求（如，一个 MySQL 查询、一个 S3 GET 请求或一个 Kafka Consumer 拉取）。

  3. 从外部系统获取数据。

  4. 在 ClickHouse 内部进行后续的计算（如 `GROUP BY`, `JOIN` 等）。

* **主要类别:**

  1. **数据库类:** `MySQL`, `PostgreSQL`, `JDBC`, `ODBC`。用于连接其他数据库。

  2. **文件系统/对象存储类:** `File`, `URL`, `HDFS`, `S3`。用于直接读取文件。

  3. **消息队列类:** `Kafka`, `RabbitMQ`。用于消费实时消息流。

***

##### 3 连接关系型数据库 (MySQL)

* **场景:** 我们的用户维度表存储在生产环境的 MySQL 中，我们希望在 ClickHouse 中直接用它来关联事实表，而无需每日同步。

![](<images/ClickHouse 高性能实时分析数据库-diagram-3.png>)

* **准备工作:**

  * 一个运行中的 MySQL 服务器。

  * 在 MySQL 中创建一个表并插入数据。

```sql
-- 在 MySQL 中执行:
CREATE DATABASE my_app;
USE my_app;
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    user_name VARCHAR(50),
    registration_city VARCHAR(50)
);

INSERT INTO users VALUES 
(1, 'AO', 'Beijing'), 
(2, 'BO', 'Shanghai'), 
(3, 'AB', 'HongKong');
-- 在clickhouse中建表
CREATE TABLE mysql_users
(
    user_id UInt32,
    user_name String,
    registration_city String
)
ENGINE = MySQL('linux01:3306', 'my_app', 'users', 'root', 'root');
参数...

```

注意: 在clickhouse表中可以查询对应的mysql表数据 ,**也可以对数据进行修改(插入)!**

进行关联查询

```sql
-- 先创建一张 ClickHouse 本地表
CREATE TABLE local_orders (
    order_id String,
    user_id UInt32,
    amount Float64
) ENGINE = MergeTree() ORDER BY order_id;
INSERT INTO local_orders VALUES ('order1', 1, 99.9), ('order2', 3, 45.0), ('order3', 1, 120.5);

-- 联邦查询！
SELECT
    o.order_id,
    o.amount,
    u.user_name,
    u.registration_city
FROM local_orders AS o
JOIN mysql_users AS u ON o.user_id = u.user_id;

   ┌─order_id─┬─amount─┬─user_name─┬─registration_city─┐
1. │ order1   │   99.9 │ AO        │ Beijing           │
2. │ order2   │     45 │ AB        │ HongKong          │
3. │ order3   │  120.5 │ AO        │ Beijing           │
   └──────────┴────────┴───────────┴───────────────────┘
```

![](<images/ClickHouse 高性能实时分析数据库-image-12.png>)

##### 4 HDFS 引擎

clickhouse可以直接加载HDFS中的数据(结构化数据), 来进行处理 性能低!&#x20;

![](<images/ClickHouse 高性能实时分析数据库-image-13.png>)

* **两种使用方式：**

  1. **`hdfs()` 表函数:** 用于一次性的、临时的查询。无需创建表，语法灵活。

  2. **`HDFS` 引擎表:** 用于需要频繁查询的固定路径。创建一个永久的表结构，简化后续查询。



```sql
SELECT *
FROM hdfs(
    'hdfs://your-namenode-host:9000/user/clickhouse/logs/dt=2023-11-15/log1.parquet',
    'Parquet',
    'event_time DateTime, level String, message String' -- 必须手动定义 Schema
)
LIMIT 10;
```

1. `URI`: 完整的 HDFS 文件路径。

2. `Format`: 文件格式，如 `Parquet`, `ORC`, `CSV`, `JSONEachRow` 等。

3. `Structure`: 表的结构定义，`'col1 type1, col2 type2, ...'`。

**利用路径通配符 (Globs) 查询多个文件:**

* “`HDFS` 引擎的强大之处在于支持通配符，这让我们能轻松处理分区数据。”

```sql
-- 查询 2023-11-15 这一天所有的日志
SELECT count(*)
FROM hdfs(
    'hdfs://your-namenode-host:9000/user/clickhouse/logs/dt=2023-11-15/*.parquet',
    'Parquet',
    'event_time DateTime, level String, message String'
);

-- 查询 11 月份所有日期的日志
SELECT count(*)
FROM hdfs(
    'hdfs://your-namenode-host:9000/user/clickhouse/logs/dt=2023-11-*/log*.parquet',
    'Parquet',
    '...' -- 省略结构
);

-- 查询指定两天的数据
SELECT count(*)
FROM hdfs(
    'hdfs://your-namenode-host:9000/user/clickhouse/logs/dt=2023-11-{15,16}/*.parquet',
    'Parquet',
    '...'
);
```

**创建 `HDFS` 引擎表进行持久化访问 (10分钟)**

* “如果一个 HDFS 路径我们会经常查询，可以为它创建一个永久的表定义。”

```sql
CREATE TABLE hdfs_logs
(
    event_time DateTime,
    level String,
    message String
)
ENGINE = HDFS(
    'hdfs://your-namenode-host:9000/user/clickhouse/logs/dt=*/', -- 使用通配符
    'Parquet'
);
```

##### 6 消费实时数据流 (Kafka)

ClickHouse 的 Kafka 引擎本质上是一个**数据流的适配器（Adapter）**，而不是一个存储引擎。

你需要记住的最重要的一点是：**Kafka 引擎本身不存储任何数据**。它就像一根管道，直接连接到 Kafka 的 Topic。当你查询一个 `ENGINE = Kafka` 的表时，ClickHouse 会实时地从 Kafka Topic 中拉取（Consume）消息，并根据你指定的格式（如 JSON, CSV）进行解析，然后将结果返回给你。

由于它不存储数据，所以它通常不单独使用，而是与**物化视图（Materialized View）** 结合，形成一个完整、高效的数据摄取流水线（Pipeline）。

**核心比喻**：

* **Kafka Topic**：一个源源不断流淌着“原浆数据”的河流。

* **ClickHouse Kafka 引擎**：一根直接插在河里的**智能吸管**，它只负责吸水，不负责存水。

* **ClickHouse MergeTree 表**：一个巨大无比的**蓄水池**（我们的数据仓库），水最终要存在这里。

* **物化视图**：一个**永动机水泵**，自动把吸管吸上来的水，源源不断地泵入蓄水池。

![](<images/ClickHouse 高性能实时分析数据库-image-15.png>)

**图解**：数据从各种源头生产出来，汇入 Kafka 这条大河。我们的“智能吸管”（Kafka引擎表）从河里实时吸水，然后“永动机水泵”（物化视图）立刻把水抽走，存入“蓄水池”（MergeTree表），最后数据分析师就可以在蓄水池里愉快地游泳（查询）了！

光说不练假把式！我们来亲手搭建这个系统。假设 Kafka 的 `user_actions` topic 里有如下JSON数据流：
`{"user_id": 101, "event": "login", "ts": "2023-10-27 10:00:00"}`
`{"user_id": 102, "event": "purchase", "ts": "2023-10-27 10:00:05"}`

###### 第一步：建造蓄水池 (创建 MergeTree 目标表)

我们得先有个地方存数据。这是我们的最终归宿，必须坚固耐用（性能好）。

```sql
-- 这是我们的“蓄水池”，用来存最终的数据
CREATE TABLE account_store (
    user_id UInt64,
    name String,
    city String
) ENGINE = MergeTree()
PARTITION BY city
ORDER BY (user_id);
```

###### 第二步：安装智能吸管 (创建 Kafka 引擎表)

现在，把我们的吸管插到 Kafka 河里。

```sql
-- 这是我们的“智能吸管”，它本身不存水！
drop table if exists account  ;
CREATE TABLE account (
    user_id UInt64,
    name   String,
    city String
) ENGINE = Kafka
SETTINGS
    kafka_broker_list = 'linux01:9092,linux02:9092,linux03:9092',
    kafka_topic_list = 'click_data',
    kafka_group_name = 'g1', -- 非常重要！每个流用独立组名
    kafka_format = 'JSON', -- 告诉吸管，水里的是啥味道的（数据格式）
    kafka_num_consumers = 1 ,
    kafka_handle_error_mode = 'stream'  -- 数据解析失败的处理方式
 
    ;
```

**灵魂拷问**：如果我现在 `SELECT * FROM user_actions_pipe`，会发生什么？
**答案**：你会看到 **当前 Kafka Topic 中的数据**！就像你用吸管吸了一口河水尝尝味道。但你关掉查询，数据就没了，因为它不存储。

###### 第三步：启动永动机水泵 (创建物化视图)

```sql
-- 这是我们的“永动机水泵”，连接吸管和蓄水池
CREATE MATERIALIZED VIEW user_actions_pump TO account_store AS
SELECT user_id, name, city
FROM account ;
```

**工作原理**：

* `TO  account_store `: 告诉水泵，水要泵到哪个池子。

* `AS SELECT ... FROM account `: 告诉水泵，要从哪个吸管抽水，以及抽水的方式（可以直接抽，也可以在抽的时候过滤、转换一下）。

**大功告成！** 从现在起，任何进入 account Topic 的新消息，都会被这套全自动系统捕捉，并在几秒钟内出现在 account\_store 表中，随时可以查询！



性能优化: 如果管道堵了怎么办

**关键监控指标：消费延迟 (Lag)**
Lag 指的是你的消费速度和你上游数据生产速度之间的差距。Lag 持续增大，说明你的“水泵”马力不足，水快要从河里溢出来了！

**巡查工具**：`system.kafka_consumers` 表

```sql
-- 查水表！看看我们的消费组状态
SELECT
    table,
    partition,
    last_committed_offset, -- 水泵上次汇报说“我抽到这儿了”
    current_offset,        -- 河流的最新水位
    (current_offset - last_committed_offset) AS lag, -- 水位差
    last_error             -- 水泵有没有发出警报？
FROM system.kafka_consumers
WHERE table = 'user_actions_pipe';
```

下图:所示

```sql
gantt
    title Kafka 分区消费状态
    dateFormat X
    axisFormat %s

    section "健康状态 (Lag ≈ 0)"
        已消费 :done, P1, 0, 9
        未消费 :active, P2, 9, 1
        水位/Offset :milestone, M, 10, 0s
        消费者位置 :crit, C, 9, 0s

    section "告警状态 (Lag > 0)"
        已消费 :done, P3, 0, 5
        未消费/延迟 :crit, P4, 5, 5
        水位/Offset :milestone, M2, 10, 0s
        消费者位置 :crit, C2, 5, 0s
```

> * **问题：Lag 持续增长**
>
>   * **原因**：ClickHouse写入慢（目标表结构复杂、硬件瓶颈）或消费能力不足。
>
>   * **解决**：
>
>     * 优化 `MergeTree` 表的 `ORDER BY` 键。
>
>     * 增加 `kafka_num_consumers` 数量（不能超过Topic分区数）。
>
>     * 给 ClickHouse 服务器加配置！
>
> * **问题：`last_error` 显示错误，消费停止**
>
>   * **原因**：遇到了“**毒丸消息**” (Poison Pill)！比如你的数据流里混进了一个非JSON格式的字符串，解析器直接卡住。
>
>   * **解决**：给 Kafka 引擎表加上“金刚不坏之身”。

```sql
-- 加上这个设置，遇到10个连续的坏数据就跳过，不影响大部队
ALTER TABLE user_actions_pipe MODIFY SETTING kafka_skip_broken_messages = 10;

```

#### 5. 特殊引擎

我们已经知道，`MergeTree`家族是ClickHouse的明星引擎，它非常适合存储和分析海量的时序数据。但它并非万能的。

**思考以下场景：**

1. **高并发写入**：每秒有成千上万条日志需要写入ClickHouse，如果每次写入都直接操作`MergeTree`表，会产生大量的小数据文件（parts），影响后台合并（merge）的效率，从而拖慢查询。

2. **分布式集群查询**：当数据量大到单台服务器无法存储时，我们会搭建ClickHouse集群。此时，我们如何像查询单表一样，方便地查询分布在多个节点上的数据呢？

3. **频繁的聚合查询**：有一个大屏应用，需要每秒刷新“最近一分钟的总交易额”。如果每次都从几十亿的明细表中实时计算，延迟高且资源消耗大。

4. **关联维度表**：分析时，我们经常需要将事实表（如：订单表）与维度表（如：用户信息表）进行`JOIN`。`MergeTree`之间的`JOIN`性能在处理大表时并不理想。

为了解决这些问题，ClickHouse提供了多种“特殊表引擎”，它们就像是工具箱里的专用工具，与`MergeTree`配合使用，能极大地提升系统性能和易用性。

##### 5.1 引擎精讲 -Buffer

* **核心思想**：为目标表（通常是`MergeTree`表）在内存中设置一个缓冲区。数据写入`Buffer`表时，实际上是先写入内存。当缓冲区的数据达到一定阈值（记录数、大小、时间）后，`Buffer`引擎会自动将数据“刷写”（Flush）到后端的真实表中。

* **解决问题**：解决了高频、小批量写入导致的`MergeTree`表parts过多、合并压力大的问题。它将多次小写入聚合为一次大写入。

![](<images/ClickHouse 高性能实时分析数据库-image-16.png>)

**语法演示**

```sql
-- 1. 创建目标表 (数据最终存储的地方) -- 物理表 (使用)
CREATE TABLE default.logs_mergetree (
    `timestamp` DateTime,
    `level` String,
    `message` String
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY timestamp;

-- 2. 创建 Buffer 表
-- 它将数据缓冲后写入到 default 数据库的 logs_mergetree 表
CREATE TABLE default.logs_buffer (
    `timestamp` DateTime,
    `level` String,
    `message` String
) ENGINE = Buffer(
    default,           -- 目标数据库
    logs_mergetree,    -- 目标表/物理表
    16,                -- num_layers: 并行刷写的线程数
    10,                -- min_time (seconds): 距离上次刷写的最小时间
    100,               -- max_time (seconds): 距离上次刷写的最大时间
    10000,             -- min_rows: 最小刷新行数
    1000000,           -- max_rows: 最大刷新行数
    10000000,          -- min_bytes (10MB): 最小刷新字节数
    100000000          -- max_bytes (100MB): 最大刷新字节数
);
-- 任何一个最大条件满足都会执行ffush

-- 3. 写入数据到 Buffer 表 (非常快, 因为是写内存)
INSERT INTO default.logs_buffer VALUES (now(), 'INFO', 'User logged in');
INSERT INTO default.logs_buffer VALUES (now(), 'WARN', 'Disk space is low');

-- 4. 查询数据
-- 注意：查询Buffer表只能看到尚未刷写的数据。要查询所有数据，应该查目标表。
SELECT count() FROM default.logs_buffer; -- 可能为 0, 1, 或 2，取决于是否已刷写
SELECT count() FROM default.logs_mergetree; -- 数据最终会在这里
```

* **注意事项**：

  * `Buffer`表不保证数据的<span style="color: rgb(216,57,49); background-color: inherit">原子性和持久性</span>。如果服务器异常重启，内存中尚未刷写的数据会丢失。

  * `Buffer`表不支持`ALTER`, `UPDATE`, `DELETE`, `INDEX`等操作。

  * 通常，查询操作应该直接针对后端的目标表。

  * 重要的业务数据就不要使用了  , 有可能丢失数据&#x20;

  * **用户行为日志可以使用**

##### 5.2 Distributed

* **核心思想**：`Distributed`引擎本身不存储任何数据。它是一个“虚拟”的表，充当查询的代理或路由器。当你查询一个`Distributed`表时，它会根据集群配置，将查询分发到集群中的所有分片（Shard），然后将各个分片返回的结果进行合并，最终返回给客户端。

* **解决问题**：简化分布式集群的查询操作，让用户可以像查询单表一样查询整个集群的数据。

![](<images/ClickHouse 高性能实时分析数据库-image-17.png>)

**语法与示例**：

* **前提**：你需要在ClickHouse的配置文件（如 `users.xml` 或单独的 `config.d/*.xml`）中定义一个集群。

```xml
<!-- file: /etc/clickhouse-server/config.d/my_cluster.xml -->
<clickhouse>
    <remote_servers>
        <my_cluster>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>ch_node1</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>ch_node2</host>
                    <port>9000</port>
                </replica>
            </shard>
        </my_cluster>
    </remote_servers>
</clickhouse>
```

**步骤**：

1. **在每个分片上创建本地表（`local table`）**。表结构必须完全一致。

```sql
-- 在 ch_node1 和 ch_node2 上分别执行
CREATE TABLE default.logs_local ON CLUSTER my_cluster (
    `timestamp` DateTime,
    `level` String,
    `message` String
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/logs_local', '{replica}')
PARTITION BY toYYYYMM(timestamp)
ORDER BY timestamp;
-- ON CLUSTER my_cluster 可以在所有节点上执行此语句
```

**在集群中的任一节点（或所有节点）上创建 `Distributed` 表**。

```sql
CREATE TABLE default.logs_distributed ON CLUSTER my_cluster (
    `timestamp` DateTime,
    `level` String,
    `message` String
) ENGINE = Distributed(
    my_cluster,       -- 集群名称 (与配置文件中一致)
    default,          -- 远程节点上的数据库
    logs_local,       -- 远程节点上的表名
    rand()            -- 分片键 (sharding_key)，用于写入时决定数据路由到哪个分片
);
```

**使用:&#x20;**

```sql
-- 写入数据: 数据会自动根据分片键被路由到某个分片
INSERT INTO default.logs_distributed VALUES (now(), 'ERROR', 'Payment gateway timeout');

-- 查询数据: 查询会被分发到所有分片，结果在协调器节点上聚合
SELECT level, count()
FROM default.logs_distributed
GROUP BY level;
```

##### 5.3 字典引擎(优化 join)

* **核心思想**：字典不是表引擎，而是一种将外部(内)数据源（如ClickHouse表、MySQL、文件等）加载到内存中，并以高效的键值对形式提供访问的特殊数据结构。它主要通过`dictGet`函数在查询中使用，以替代`JOIN`操作。

* **解决问题**：解决了与低基数、更新不频繁的维度表进行`JOIN`时的性能瓶瓶颈问题。

**工作原理**

![](<images/ClickHouse 高性能实时分析数据库-image-18.png>)

**语法与示例**：

* <span style="color: rgb(216,57,49); background-color: inherit">用户信息</span>数据  -  变化频次低 数据量有限

* 页面访问记录表    join  on  用户表uid和访问信息表的uid

* 映射到字典后 , 每行页面访问数据就可以根据uid获取字典数据 , 进行用户详细的填补 ,字典数据在内存中

1. **准备维度数据**（可以是一个ClickHouse表）：

```javascript
CREATE TABLE  user_info (
    `user_id` UInt64,
    `user_name` String,
    `city` String
) ENGINE = MergeTree()
ORDER BY user_id ;

INSERT INTO  user_info VALUES (1, 'Alice', 'New York'), (2, 'Bob', 'London');
```

**定义字典配置文件**（例如 `dictionaries/user_info_dict.xml`）：

```xml
<dictionaries>
  <dictionary>
    <name>user_info_dict2</name>
    <source>
      <clickhouse>
        <host>localhost</host>
        <port>9000</port>
        <user>default</user>
        <password></password>
        <db>learning</db>
        <table>user_info</table>
      </clickhouse>
    </source>
    <layout>
      <hashed /> <!-- 内存布局，hashed适合唯一key -->
    </layout>
    <structure>
      <id>
        <name>user_id</name> <!-- Key列 -->
      </id>
      <attribute>
        <name>user_name</name> <!-- 可查询的属性列 -->
        <type>String</type>
        <null_value></null_value>
      </attribute>
      <attribute>
        <name>city</name>
        <type>String</type>
        <null_value></null_value>
      </attribute>
    </structure>
    <lifetime>
      <min>300</min> <!-- 字典在内存中的刷新周期（秒） -->
      <max>360</max>
    </lifetime>
  </dictionary>
</dictionaries>
```

*确保这个xml文件被ClickHouse主配置文件`include`。*

在 config.xml中引入

```sql
<dictionaries_config>/etc/clickhouse-server/dicts/*.xml</dictionaries_config>
```

```sql
-- 验证字典创建成功否 ; 重启试试!
SELECT
    name,
    type,
    key,
    attribute.names,
    attribute.types,
    bytes_allocated,
    element_count,
    source
FROM system.dictionaries
-- WHERE name = 'user_info_dict2'
```



**在查询中使用`dictGet`函数**：

```sql
-- 假设我们有一个事实表 `page_views`
CREATE TABLE page_views (
    `view_time` DateTime,
    `user_id` UInt64,
    `url` String
) ENGINE = MergeTree() ORDER BY view_time;

INSERT INTO page_views VALUES (now(), 1, '/home'), (now(), 2, '/pricing');

-- 使用字典进行“关联”查询
SELECT
    view_time,
    user_id,
    url,
    dictGet('user_info_dict2', 'user_name', user_id) AS user_name,
    dictGet('user_info_dict2', 'city', user_id) AS user_city
FROM page_views;
```

这个查询的性能远高于 `SELECT ... FROM page_views LEFT JOIN user_info ON ...`。

##### 5.4 Memeory 内存

* **描述**: 数据只存储在内存中，以未压缩的形式存在。服务器重启后数据会丢失。

* **用途**: 用于存储临时的、数据量不大的中间结果，或者用于快速测试。读写速度极快

```sql
CREATE TABLE temp_data (id UInt64, data String) ENGINE = Memory;
INSERT INTO temp_data VALUES (1, 'some data');
SELECT * FROM temp_data;
```

##### **5.5 Null**:

* **描述**: 任何写入`Null`表的数据都会被直接丢弃。读取`Null`表总是返回空结果。

* **用途**:

  1. 与`Distributed`表结合使用，当某些分片只需要接收数据而不需要存储时。

  2. 用于性能基准测试，衡量数据传输和解析的开销，而不受磁盘I/O影响。

  3. 在开发和测试阶段，作为数据写入的终点，而不关心数据存储。

```sql
CREATE TABLE black_hole (id UInt64, data String) ENGINE = Null;
-- 这条语句会成功执行，但数据会消失
INSERT INTO black_hole VALUES (1, 'this will be discarded');
SELECT count() FROM black_hole; -- 永远返回 0
```

##### 综合实战 (Practical Scenario)

**目标**: 构建一个可扩展的、高性能的分布式日志接收和分析系统。

**架构设计**:
我们将组合使用 `Buffer`, `ReplicatedMergeTree`, 和 `Distributed` 引擎



1. **数据写入**: 应用程序将日志高速写入本地节点的`Buffer`表。

2. **数据缓冲**: `Buffer`表在内存中聚合数据，然后批量刷入本地的`ReplicatedMergeTree`表。

3. **数据存储与高可用**: `ReplicatedMergeTree`负责持久化存储数据，并通过ZooKeeper在副本间同步，保证高可用。

4. **分布式查询**: 用户通过`Distributed`表查询，可以透明地访问整个集群的所有日志数据。

![](<images/ClickHouse 高性能实时分析数据库-image-19.png>)

**实现步骤**: (假设我们有一个名为 `my_cluster` 的两分片集群)

1. **在所有节点上创建本地表 (`ReplicatedMergeTree`)**:

```javascript
CREATE TABLE default.logs_local ON CLUSTER my_cluster (
    `timestamp` DateTime,
    `hostname` String,
    `level` String,
    `message` String
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/logs_local', '{replica}')
PARTITION BY toYYYYMM(timestamp)
ORDER BY (timestamp, hostname);
```

* **在所有节点上创建`Buffer`表，指向本地表**:

```javascript
CREATE TABLE default.logs_buffer ON CLUSTER my_cluster (
    `timestamp` DateTime,
    `hostname` String,
    `level` String,
    `message` String
) ENGINE = Buffer(default, logs_local, 16, 10, 100, 10000, 1000000, 10485760, 104857600);
```

*注意：这里的写入目标是`logs_local`，即本地节点上的表。*

* **在所有节点上创建`Distributed`表，用于全局查询**:

```javascript
CREATE TABLE default.logs_distributed ON CLUSTER my_cluster (
    `timestamp` DateTime,
    `hostname` String,
    `level` String,
    `message` String
) ENGINE = Distributed(my_cluster, default, logs_local, rand());
```

**工作流程**:

* **写入**: 应用通过负载均衡器将日志发送到任意节点的`9000`端口，`INSERT`到`logs_buffer`表。

```sql
-- 这是一个应用端的操作, 写入到任一节点的 logs_buffer
INSERT INTO default.logs_buffer VALUES (now(), 'web-server-01', 'INFO', 'Request processed successfully');
```

**查询**: 分析师连接到任意节点，从`logs_distributed`表中查询。

```sql
-- 查找最近一小时内所有的ERROR日志
SELECT *
FROM default.logs_distributed
WHERE level = 'ERROR' AND timestamp >= now() - INTERVAL 1 HOUR;

-- 统计每个主机的日志数量
SELECT hostname, count()
FROM default.logs_distributed
GROUP BY hostname;
```

这个架构充分利用了多个引擎的优点，构建了一个生产级的日志解决方案。

### 3.4  数据库引擎

大家好！在之前的课程中，我们花了大量时间研究**表引擎**，比如 `MergeTree`。我们把表引擎比作功能各异的“储物箱”。我们学会了为不同的数据选择合适的储物箱。

现在，我们把视角拉高一个层次。如果说**表 (Table)** 是一个房间里的**储物箱**，那么**数据库 (Database)** 就是存放这些储物箱的**房间**。

一个很自然的问题就来了：这些“房间”本身，有没有什么不同的特性呢？比如，有的房间是防火的，有的房间是恒温的，有的房间甚至是一个“任意门”，可以直接通往另一栋大楼的房间。

这就是我们今天要探讨的主题——**数据库引擎 (Database Engine)**。它定义了一个数据库（一个“房间”）自身的行为和特性，比如它如何存储表的元数据、是否支持原子性的 DDL 操作，以及它是否能连接到外部系统。

![](<images/ClickHouse 高性能实时分析数据库-image-20.png>)

从图中可以看到，数据库引擎定义了数据库级别的行为，而每个数据库内部的表，仍然可以拥有自己独立的表引擎。

#### 1) 默认数据库引擎：`Ordinary` 与 `Atomic`&#x20;

> 当我们执行 `CREATE DATABASE my_db;` 而不指定引擎时，ClickHouse 会使用一个默认的数据库引擎。在不同的版本中，这个默认值发生了变化。
>
> * **特点:** 这是最原始、最简单的数据库引擎。
>
> * **元数据存储:** 每个表的元数据（它的 `CREATE` 语句）都存储在一个单独的 `.sql` 文件里。比如 `CREATE TABLE my_table ...`，就会在磁盘上生成一个 `my_table.sql` 文件。
>
> * **核心缺陷:** **非原子性操作。**
>   \> \* **`RENAME TABLE`** 操作不是原子的。在重命名期间，如果服务器发生故障，你可能会得到一个“半成功”的状态，导致表名不一致。
>   \* **`EXCHANGE TABLES`** 和 **`SWAP`** 等操作也不支持。
>
> * **比喻:** 一个管理松散的办公室。每个文件（表）都有自己的档案袋（.sql文件），但移动和交换文件时，可能会出错，没有一个统一的登记处来保证操作的完整性。
>
> 从 v21.1 版本开始，`Atomic` 成为了默认的数据库引擎。它解决了 `Ordinary` 的所有痛点。
>
> * **特点:** 提供了原子性的 DDL 操作。
>
> * **元数据存储:** 所有表的元数据都集中存储在 ZooKeeper（如果是集群环境）或本地的一个统一日志中。
>
> * **核心优势:** **原子性 DDL。**
>   \> \* **`RENAME TABLE`** 是原子的。它要么成功，要么失败，不会出现中间状态。
>   \> \* 支持 **`EXCHANGE TABLES`** 操作，可以原子地交换两张表，这在数据发布、灰度测试等场景中非常有用。
>
> * **比喻:** 一个管理严格的中央档案室。所有文件的操作（创建、重命名、交换）都必须在中央登记簿（ZooKeeper/本地日志）上记录，并且是事务性的。要么整个操作完成并登记，要么整个操作回滚，档案室永远不会处于混乱状态。

```sql
-- 1. 创建一个 Atomic 引擎的数据库 (在新版中 ENGINE 可省略)
CREATE DATABASE atomic_db ENGINE = Atomic;
USE atomic_db;

-- 2. 创建两张简单的表
CREATE TABLE t1 (id UInt8) ENGINE = Log;
CREATE TABLE t2 (id UInt8) ENGINE = Log;
INSERT INTO t1 VALUES (1);
INSERT INTO t2 VALUES (2);

-- 3. 体验原子性的交换操作
EXCHANGE TABLES t1 AND t2;  -- Linux kernel 3.15+ 

-- 4. 验证结果
SELECT * FROM t1; -- 结果应该是 2
SELECT * FROM t2; -- 结果应该是 1
```

这个 `EXCHANGE` 操作在 `Ordinary` 数据库中是无法执行的。这充分体现了 `Atomic` 引擎的优越性。**结论：在新项目中，始终使用 `Atomic` 数据库引擎。**

#### 2) `Lazy` 引擎：为大量小文件而生 (10分钟)

> **场景:** 想象你有海量的日志数据，每天都会生成成千上万个小的 `MergeTree` 表（例如，按小时甚至分钟分表）。当 ClickHouse 启动时，它需要加载所有表的元数据，如果表数量达到几十万甚至上百万，启动过程会变得非常缓慢。
>
> **`Lazy` 引擎的解决方案:**
>
> * **特点:** 它只在内存中保留最近一段时间内（由 `expiration_time_in_seconds` 参数定义）被访问过的表。对于长期未被访问的“冷”表，它会从内存中卸载，只保留其在磁盘上的数据。
>
> * **工作机制:** 当你查询一张已经被卸载的“冷”表时，`Lazy` 引擎会重新加载它的元数据到内存中，然后再执行查询。这会带来一点首次查询的延迟，但极大地加快了服务器的启动速度和降低了内存占用。
>
> **比喻:** `Lazy` 引擎就像一个\*\*“健忘”的图书管理员\*\*。他的办公桌上只放最近被借阅过的几本书（热表）。对于书库里成千上万本没人看的书（冷表），他根本不记在脑子里。只有当有人要借一本冷门书时，他才去书库里把它找出来（加载元数据），这会慢一点，但让他的日常工作（服务器运行）非常轻松。

**【实践】:** 创建一个 Lazy 数据库

```sql
-- 创建一个 Lazy 数据库，表在 60 秒不被访问后就会被从内存卸载
CREATE DATABASE lazy_db2 ENGINE = Lazy(60);
USE lazy_db2;

CREATE TABLE my_lazy_table (id UInt8) ENGINE = MergeTree() ORDER BY id;
INSERT INTO my_lazy_table VALUES (1);

-- 立即查询，表在内存中
SELECT * FROM my_lazy_table;

-- 等待超过 60 秒...

-- 再次查询，你会感受到一个非常微小的延迟 (在表很多时会很明显)
-- 因为 ClickHouse 需要重新加载表的元数据
SELECT * FROM my_lazy_table;
```

#### 3) 外部数据库引擎：`MySQL` & `PostgreSQL`&#x20;

> 这是数据库引擎中最令人兴奋的功能之一！它允许你把一个外部的关系型数据库（如 MySQL 或 PostgreSQL）整个“映射”到 ClickHouse 中，就像它是 ClickHouse 的一个原生数据库一样。
>
> **场景:** 你的公司核心业务跑在 MySQL (OLTP) 上，但你想对这些业务数据进行复杂的 OLAP 分析。传统做法是编写 ETL 脚本，定期把 MySQL 数据同步到 ClickHouse。这个过程很繁琐。
>
> **`MySQL` 引擎的解决方案:**
>
> * **特点:** 直接在 ClickHouse 中创建一个指向外部 MySQL 数据库的“快捷方式”。
>
> * **工作机制:**
>   \> \* 当你在 ClickHouse 中 `CREATE DATABASE ... ENGINE = MySQL(...)` 时，ClickHouse 不会创建任何物理数据。它只是连接到指定的 MySQL 服务器，并缓存其数据库的表结构信息。
>   \> \* 当你查询这个数据库下的某张表时（例如 `SELECT ... FROM mysql_db.users`），ClickHouse 会**实时地**将你的查询转换成 MySQL 能理解的 SQL，发送给 MySQL 服务器执行。
>   \> \* MySQL 返回结果给 ClickHouse，ClickHouse 再把结果返回给你。
>
> **比喻:** `MySQL` 数据库引擎就是一道\*\*“任意门”\*\*。你在 ClickHouse 的大楼里打开这扇门，直接就走进了隔壁 MySQL 大楼的房间里，可以随意查看里面的东西（数据），而无需把东西搬过来。

**【实践】:** 连接一个外部 MySQL 数据库
**前提:** 你需要有一个可访问的 MySQL 服务器，并且有一个数据库和表。
(假设 MySQL 中有数据库 `my_erp`，表 `products`)

```sql
-- 在 ClickHouse 中创建“任意门”
CREATE DATABASE mysql_bridge ENGINE = MySQL('linux01:3306', 'my_app', 'root', 'root');

-- 现在，你可以直接查看 MySQL 中的表了
SHOW TABLES FROM mysql_bridge;
-- 你会看到 MySQL 中 `my_erp` 数据库的所有表名

-- 直接查询 MySQL 中的表
SELECT * FROM mysql_bridge.products LIMIT 5;

-- 更强大的功能：跨数据库 JOIN！
-- 将 ClickHouse 中的本地大表 (例如日志) 与 MySQL 中的维度表 (例如产品信息) 进行关联查询
SELECT
    hits.url,
    products.product_name
FROM learning.t_web_hits AS hits
JOIN mysql_bridge.products AS products ON hits.product_id = products.id
...
```

**重要提示:**

* 这种方式的性能受限于外部数据库（<span style="color: rgb(216,57,49); background-color: inherit">MySQL</span>）的处理能力和网络延迟。

* 它非常适合查询**维度表**或**小批量数据**。对于海量事实表的分析，仍然推荐将数据导入 ClickHouse 本地。

* `PostgreSQL` 数据库引擎的工作方式与 `MySQL` 完全相同。

## 第4章：高性能的模式设计 (Schema Design for Performance)

### 4.1. **核心设计原则**

#### &#xA;<span style="color: rgb(216,57,49); background-color: inherit"> </span>**<span style="color: rgb(216,57,49); background-color: inherit">宽表优先，适当反范式化</span>**

忘掉你在 MySQL 等关系型数据库里学的“第三范式”！在 ClickHouse 的世界里，**JOIN 是昂贵的**。我们追求的是一次扫描，出所有结果。

* **传统做法 (慢)**：`订单表` JOIN `用户表` JOIN `商品表`...

* **ClickHouse 做法 (快)**：把 `用户名`, `商品名` 等信息直接冗余到 `订单表` 中，形成一张“宽表”。

![](<images/ClickHouse 高性能实时分析数据库-image-21.png>)

注意: 在CK中建议使用宽表 ,虽然有冗余数据, 但是对于分析计算时的性能提成很大

#### **选择正确的排序键 (ORDER BY)**：

这是 ClickHouse 最重要的性能优化点

`ORDER BY` 是 **ClickHouse 表设计中最最最重要的一个环节**！它决定了数据在磁盘上的**物理存储顺序**。

想象一下一本巨大的电话簿。如果它是按姓氏首字母排序的，你要找姓“张”的人会非常快。但如果它是乱序的，你只能一页一页翻。

* **排序键的威力**：ClickHouse 会根据 `ORDER BY` 的列创建**稀疏索引**。当你查询的 `WHERE` 条件命中了排序键的前缀时，ClickHouse 就能像翻电话簿一样，迅速跳过大量不相关的数据块。

![](<images/ClickHouse 高性能实时分析数据库-image-22.png>)

**图解**：当查询 `WHERE event_date = '2023-10-02'` 时，ClickHouse 查看索引发现，只有“数据块2”可能包含这个日期的数据，因此它会**跳过**“数据块1”和“数据块3”，只读取极少量的数据。

**法则**：将你**最常用作查询条件、范围筛选、分组**的列放在 `ORDER BY` 的最前面！

#### **合理设置分区键 (PARTITION BY)**

如果说 `ORDER BY` 是整理书架上的书，那 `PARTITION BY` 就是把图书馆分成不同的房间，比如“历史区”、“科技区”。

* **分区的好处**：当你的查询条件能命中分区键时，ClickHouse 连“房间”的门都不会打开，直接跳过整个分区目录。这对于删除、修改旧数据（`ALTER TABLE ... DROP PARTITION`）也非常高效。

**常用分区策略**：按月（`toYYYYMM(event_date)`）或按天（`toDate(event_date)`）。

![](<images/ClickHouse 高性能实时分析数据库-diagram-4.png>)

![](<images/ClickHouse 高性能实时分析数据库-image-23.png>)

![](<images/ClickHouse 高性能实时分析数据库-diagram-5.png>)

**法则**：分区粒度不宜过细（比如按秒），否则会产生海量小文件，拖垮性能。通常按月或按天是最佳实践。
**数据类型是关键**

**使用最小且最合适的数据类型 &#x20;**

用大炮打蚊子是浪费。为数据选择**最小且最合适**的类型，可以极大地减少存储空间、降低内存消耗和 I/O，从而提升查询速度。

* **错误**：用 `String` 存IP地址，用 `Int64` 存年龄。

* **正确**：用 `IPv4` 类型存IP，用 `UInt8` 存年龄（0-255岁足够了）。



1. 反范式 , 建立宽表

2. 使用合理的引擎  MergeTree优先

3. 设置合适的排序键

4. 建立合理的分区

5. 选择合适且最小的数据类型

### &#xA;4.2. **ClickHouse 的数据类型**

https://clickhouse.com/docs/sql-reference/data-types

#### 性能神器：LowCardinality(String) 详解

`LowCardinality`（低基数）是 ClickHouse 的大杀器。对于那些重复值很多的 `String` 列，它通过字典编码的方式，将长字符串替换为小整数。

![](<images/ClickHouse 高性能实时分析数据库-image-24.png>)

#### &#xA;**实践 设计一个用户行为日志表**

: ，综合运用多种数据类型和 `LowCardinality`。

&#x20;            记录用户行为，包括用户ID、事件类型、事件时间、<span style="color: rgb(216,57,49); background-color: inherit">来源国家</span>、访问的URL。

```sql
CREATE TABLE user_behavior( -- 用户行为表
    -- 排序键和分区键
    event_date          Date,
    event_time          DateTime,
    
    -- 高性能类型选择
    user_id             UInt64,
    event_type          LowCardinality(String), -- 事件类型基数低，用LowCardinality
    country_code        FixedString(2),         -- 国家代码是定长2位
    
    -- 复杂与可空类型
    url                 String,
    user_agent          Nullable(String),       -- 用户代理可能为空
    extra_params        Map(String, String)     -- 存储额外的KV参数
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY (event_date, event_type, user_id);




CREATE TABLE  person(
uid  UInt64 ,
country LowCardinality(String) 
)  engine = Log() ;
```

### 4.3 索引与数据跳过（查询的“瞬移”能力）

#### 4.3.1 主键索引 (稀疏索引) 的工作原理

1. 核心概念：稀疏索引 (Sparse Index)

与 MySQL 等数据库为每一行数据都建立索引（密集索引）不同，ClickHouse 的主键索引是**稀疏**的。它只为**每个数据颗粒（Granule）的第一行**记录一个“路标”。

* **数据颗粒 (Granule)**：ClickHouse 在存储数据时，会将表中的行分批打包，一个包就是一个 Granule。默认情况下，一个 Granule 包含 8192 行。

* **索引文件 (`primary.idx`)**：这个文件非常小，因为它只存储每个 Granule 的“路标”值。例如，如果 `ORDER BY` 是 `(event_date)`，那么索引文件里存的就是每个 Granule 的起始日期。

图示

![](<images/ClickHouse 高性能实时分析数据库-image-25.png>)

1. **查询来了**：`WHERE event_date = '2023-10-03'`。

2. **扫描索引**：ClickHouse 快速扫描内存中的 `primary.idx` 文件。

3. **定位范围**：它发现 `'2023-10-03'` 这个值介于路标2 (`'2023-10-03'`) 和路标3 (`'2023-10-05'`) 之间。这意味着，目标数据 **只可能存在于 Granule 2 中**。

4. **精确打击**：ClickHouse 直接跳过 Granule 1 和 Granule 3，只从磁盘读取 Granule 2 这一个数据块进行处理。

**结论**：稀疏索引的威力在于**大幅减少 I/O**。它不关心数据具体在哪一行，只关心数据在哪一个**数据块范围**内。

#### 主键索引的设计要点：

* **列的选择**：`ORDER BY` 的列应该是你 `WHERE` 子句中最常用的**过滤条件**，尤其是范围查询（`>`, `<`, `BETWEEN`）。

* **列的顺序**：把**基数更高**（筛选能力更强）的列放在前面。例如 `ORDER BY (event_date, user_id)` 就比 `ORDER BY (user_id, event_date)` 要好，因为日期能先过滤掉大量不相关的数据块。



我们再强调一次：ClickHouse 的主键索引是**稀疏**的。它不像 MySQL 那样为每一行都建索引。它只为每个**数据颗粒（Granule，默认8192行）** 的第一行建立一条索引记录。

**优点**：索引文件非常小，可以常驻内存。
**工作方式**：查询时，ClickHouse 在内存中快速扫描索引，定位到可能包含目标数据的 Granule 范围，然后只把这些 Granule 从磁盘加载到内存中进行精确匹配。

#### 【实践】: 为表添加跳数索引

给刚才的 `user_behavior` 表的 `url` 列添加一个布隆过滤器索引，以加速特定URL的查找。

```sql
-- 在建表时添加
CREATE TABLE user_behavior_with_index (
    -- ... 其他列定义和上面一样 ...
    url                 String,
    -- ...
    INDEX idx_url url TYPE bloom_filter() GRANULARITY 1 -- GRANULARITY表示索引的粒度  8192   2*8192
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY (event_date, event_type, user_id);

-- 查询时，ClickHouse会自动使用该索引
-- 这个查询会因为idx_url索引而变得更快
SELECT count()
FROM user_behavior_with_index
WHERE url = 'https://clickhouse.com/docs/en/';
```

#### 4.3.2 数据跳过索引 (Skipping Indexes)-Granule 的“智能标签”&#x20;



如果说主键索引是城市间的高速公路，那么数据跳过索引就是每个高速出口旁边的**信息指示牌**。它告诉你这个出口下去的区域“有什么”和“没有什么”，帮你决定是否要下高速。

数据跳过索引是附加在**每个数据颗粒 (Granule)** 上的元数据。它独立于主键索引，用于对**非主键列**进行预过滤。

除了主键，ClickHouse 还提供了额外的“跳数索引”，它们像给数据颗粒贴上的“标签”，进一步减少需要扫描的数据量。

* `minmax`: 记录每个颗粒内某列的最大最小值。如果查询 `WHERE price > 500`，而某个颗粒的 `minmax` 标签是 `[100, 400]`，则可以直接跳过。

* `set(N)`: 记录每个颗粒内某列的前N个唯一值。如果查询 `WHERE color = 'Red'`，而某个颗粒的 `set` 标签是 `{'Blue', 'Green'}`，则可以跳过。

* `bloom_filter`: 一种概率性索引。如果你查询 `WHERE has(urls, 'some_rare_url')`，布隆过滤器可以快速告诉你“这个颗粒**绝对没有**这个URL”，从而跳过。它可能会误报（说有但实际没有），但绝不会漏报。

![](<images/ClickHouse 高性能实时分析数据库-diagram-6.png>)



##### `① minmax`

* **作用**：记录每个 Granule 中某一列的最小值和最大值。

* **场景**：非常适合数值或日期类型。

* **原理**：查询 `WHERE price > 1000`。如果某个 Granule 的 `minmax` 标签是 `[100, 900]`，ClickHouse 就知道这个 Granule 内所有 `price` 都小于等于900，不可能满足条件，于是直接跳过。

![](<images/ClickHouse 高性能实时分析数据库-image-26.png>)

**图解**：查询 `price > 1000` 时，Granule 1 被直接跳过，因为它的最大值 900 都不满足条件。Granule 2 和 Granule 3 因为范围有交集，所以需要被读取。

##### `② set(N)`

* **作用**：记录每个 Granule 中某列的**前 N 个唯一值**。

* **场景**：适合基数较低的 `String` 或 `Enum` 列，用于等值查询。

* **原理**：查询 `WHERE city = 'Shanghai'`。如果某个 Granule 的 `set(3)` 标签是 `{'Beijing', 'Guangzhou', 'Shenzhen'}`，ClickHouse 就知道这个 Granule 里根本没有 'Shanghai'，直接跳过。

##### `③ bloom_filter`

* **作用**：一种概率性数据结构，可以**非常确定地判断一个元素“绝对不存在”**，但只能**概率性地判断“可能存在”**。

* **场景**：

  * 高基数的 `String` 列（如 URL，用户ID）。

  * 检查数组中是否包含某个元素 `has(array, 'value')`。

  * 检查 Map 中是否存在某个键 `mapContains(map, 'key')`。

* **原理**：它像一个“黑名单筛选器”。数据写入时，把 Granule 里的值都扔进布隆过滤器。查询时，先问布隆过滤器：“这个值在你的黑名单上吗？”

  * 如果回答“**不在**”（即绝对不存在），则**安全跳过**。

  * 如果回答“**可能在**”（有可能是误报），则需要**读取 Granule 进一步确认**。

![](<images/ClickHouse 高性能实时分析数据库-image-27.png>)

**图解**：查询 'e.com' 时，布隆过滤器 1 准确地告诉我们 Granule 1 中没有，从而避免了一次 I/O。布隆过滤器 2 提示可能存在，我们就需要去读取 Granule 2 来做最终的判断。

#### 【实践】: 为表添加跳数索引

**1. 建表时添加索引：**
假设我们有一个日志表，我们经常需要根据 `request_id`（高基数列）来追溯单条日志。

```sql
CREATE TABLE access_logs (
    event_time  DateTime,
    request_id  String,
    http_code   UInt16,
    url         String
) ENGINE = MergeTree()
ORDER BY (event_time)
SETTINGS index_granularity = 8192; -- 明确指定颗粒大小

-- 为 request_id 和 http_code 添加跳数索引
ALTER TABLE access_logs ADD INDEX idx_req_id request_id TYPE bloom_filter() GRANULARITY 4;
ALTER TABLE access_logs ADD INDEX idx_code http_code TYPE set(0) GRANULARITY 4;
```

`GRANULARITY 4`：表示这个跳数索引的粒度是主索引的 4 倍。即每 `4 * 8192` 行数据，才生成一个跳数索引块。这是一种在索引精度和大小之间的权衡。

**2. 验证索引是否生效：**
使用 `EXPLAIN` 或查询日志 `system.query_log` 是最好的方法。我们用一个更直观的方式：`trace_logging`。

```sql

INSERT INTO access_logs (event_time, request_id, http_code, url) VALUES
    (now() - 60, 'a1b2c3d4-e5f6-7890-1234-567890abcdef', 200, '/api/v1/users'),
    (now() - 55, 'b2c3d4e5-f6a7-8901-2345-67890abcdef1', 200, '/api/v1/products'),
    (now() - 43, 'c3d4e5f6-a7b8-9012-3456-7890abcdef12', 404, '/api/v1/unknown'),
    (now() - 21, 'd4e5f6a7-b8c9-0123-4567-890abcdef123', 500, '/api/v2/orders'),
    (now() - 10, 'e5f6a7b8-c9d0-1234-5678-90abcdef1234', 200, '/index.html')
    ........

-- 执行带 trace_logging 的查询
SELECT count()
FROM access_logs
WHERE request_id = 'd4e5f6a7-b8c9-0123-4567-890abcdef123'
SETTINGS log_queries=1; -- 确保查询被记录

-- 在执行查询后，立刻查看日志
-- 在 clickhouse-server.log 文件中，或者在 system.query_log 表中查找
-- 你会看到类似这样的日志：
/*
<Trace> MergeTree(Reading): Mark ranges: [0, 1]
<Trace> MergeTree(Reading): Selected 1/100 parts by partition key
<Trace> MergeTree(Reading): Selected 1/50 ranges by primary key
<Trace> MergeTree(Reading): Selected 5/20 granules by skipping indexes -- 关键！
*/
```

日志中的 `Selected ... granules by skipping indexes` 明确告诉你，数据跳过索引生效了！它帮助 ClickHouse 在主键筛选之后，又进一步排除了更多的 Granule。

#### 4.3.3 总结与最佳实践

1. **主键索引是基石**：`ORDER BY` 决定了数据的大方向，是性能优化的第一道防线。

2. **(字段)跳数索引是精细化武器**：它在主键索引筛选后的“候选范围”内，进行二次精准打击，进一步减少 I/O。

3. **按需索骥**：不要滥用索引！每个索引都会在写入时带来额外的计算开销，并占用存储空间。只为那些**真正能大幅缩小查询范围**的列创建索引。

4. **如何选择？**

   * 数值/日期范围查询 -> `minmax`

   * 低基数 `String`/`Enum` 等值查询 -> `set`

   * 高基数 `String` 等值查询或 `has()` / `mapContains()` -> `bloom_filter`

掌握了主键索引和数据跳过索引的组合拳，你就掌握了开启 ClickHouse 极致性能的钥匙。现在，去锻造你自己的“神兵利器”吧！

### &#xA;4.4. **数据导入**

INSERT 语句的最佳实践：大批量、一次性写入

记住这个口号：**“宁要胖子，不要虚胖”**。

向 ClickHouse `INSERT` 数据时，千万不要一条一条地写！这会产生大量的小文件（**Part**），后台的合并（Merge）线程会不堪重负，严重影响性能。

![](<images/ClickHouse 高性能实时分析数据库-image-28.png>)

**最佳实践**：每次 `INSERT` 至少写入几千到几万行数据。对于流式数据，先在应用层或消息队列攒批，再统一写入。

加入有一个数据文件  CSV

```sql
2023-10-27,2023-10-27 15:30:00,1001,login,CN,"https://example.com/login",,{}
2023-10-27,2023-10-27 15:31:00,1002,view,US,"https://example.com/product/1",,"{""source"":""ads""}"
... (10000 more lines) ...
```

使用 `clickhouse-client` 从文件导入：

```plain&#x20;text
# 使用命令行客户端将CSV文件内容导入到表中
clickhouse-client --query  "INSERT INTO user_behavior FORMAT CSV" < behaviors.csv
```

Use code [with caution](https://support.google.com/legal/answer/13505487).Bash

这个命令会一次性将 `behaviors.csv` 中的所有行作为一个大批次插入，这正是 ClickHouse 所喜欢的！

#### 支持的数据格式

ClickHouse 支持多种高效的数据格式导入：

* **`CSV`, `TSV`**: 简单通用。

* **`JSONEachRow`**: 非常适合日志类数据，每行一个JSON对象。

* **`Parquet`, `ORC`**: 列式存储格式，与大数据生态（Spark, Hadoop）无缝对接，导入性能极佳。

# 第三部分：高级查询与性能优化 (Advanced Querying & Optimization)

## 第5章：释放查询的全部潜力 (Advanced SQL Features)

### 5.1. **强大的聚合函数**&#xA;

聚合是数据分析的核心。除了 `count`, `sum`, `avg` 这些天下皆知的基本功，ClickHouse 还提供了许多独门秘技。

**ClickHouse 特色聚合函数**

**图一：uniq vs uniqExact 的抉择**

![](<images/ClickHouse 高性能实时分析数据库-image-29.png>)

**图解**：`uniq` 就像用眼睛估算广场上有多少人，快但不准；`uniqExact` 就像挨个数人头，准但慢。

***

**高阶函数：聚合函数组合器 (招式变种) &#x20;**

聚合函数组合器像给你的武功招式加上了“属性附魔”，让它们产生奇妙的变化。最常用的组合器是 `-If`。

* **`-If` 组合器**: 满足特定条件时才进行聚合。

**场景**：一次查询同时计算总PV和来自移动端的PV。

```sql
-- 传统写法，需要子查询或 CASE WHEN
SELECT
    page_id,
    count() AS total_pv,
    count(CASE WHEN device = 'mobile' THEN 1 ELSE NULL END) AS mobile_pv
FROM access_logs
GROUP BY page_id;

-- ClickHouse -If 组合器写法，更简洁高效！
SELECT
    page_id,
    count() AS total_pv,
    countIf(device = 'mobile') AS mobile_pv -- device='mobile'时才计数
FROM access_logs
GROUP BY page_id;
```

### 5.2 JOIN 查询 (联手合击之术)

虽然我们提倡宽表，但有时 JOIN 仍不可避免，尤其是与小型的维度表关联时。

#### &#x20;`JOIN` 的语法和类型 (`LEFT`, `INNER`, `FULL`, `CROSS`)

* `INNER JOIN` (内连接): 双方都有匹配信物的人才能配对成功，手拉手走出大门。

* `LEFT JOIN` (左连接): 左边桌子的人是主角，不管有没有配对成功，都可以走出大门。如果配对成功，就带上右边的人；如果没成功，右边的人的位置就是 `NULL`（空着）。

* `RIGHT JOIN` (右连接): 右边桌子的人是主角。

* `FULL JOIN` (全连接): 所有人都是主角，不管配没配对成功，都走出大门。

* `CROSS JOIN` (交叉连接): 不看信物，左边桌子的每个人都和右边桌子的每个人配对一次，产生笛卡尔积。



![](<images/ClickHouse 高性能实时分析数据库-image-30.png>)

#### `ALL JOIN`：多情郎模式

`ALL` 是标准 SQL 的行为。如果左表的一条记录在右表中找到了多个匹配项，它会毫不犹豫地和**每一个**匹配项都配对一次，导致左表的记录被复制。

**场景**：

* **事实表 `orders`**: `(order_id: 1, user_id: 101, amount: 99)`

* **维度表 `users_repeated` (有重复)**:

  * `(user_id: 101, name: '悟空')`

  * `(user_id: 101, name: '行者')`

```sql
-- 使用 ALL JOIN
SELECT o.order_id, o.amount, u.name
FROM orders AS o
ALL LEFT JOIN users_repeated AS u ON o.user_id = u.user_id;
```

![](<images/ClickHouse 高性能实时分析数据库-image-31.png>)

**结果分析**：原本一笔订单，现在变成了两行！如果此时计算 `sum(amount)`，结果会被错误地放大一倍。这是数据分析中非常危险的陷阱。

#### `ANY JOIN`：专一模式

`ANY` 是 ClickHouse 的特色。它同样会在右表中寻找匹配项，但一旦找到**第一个**匹配的，就立刻完成配对，**忽略后面所有其他的匹配项**。

```sql
-- 使用 ANY JOIN
SELECT o.order_id, o.amount, u.name
FROM orders AS o
ANY LEFT JOIN users_repeated AS u ON o.user_id = u.user_id;
```

![](<images/ClickHouse 高性能实时分析数据库-image-32.png>)

**结果分析**：结果只有一行，数据没有被放大，`sum(amount)` 依然是正确的。

选择困难症？听我的！

* 当你**确定**右表（维度表）的 join key 是唯一的，用 `ALL JOIN`。

* 当你**不确定**或**知道**右表 join key 可能有重复，但你只想用它来做信息补全（数据丰富化），并且不关心用的是哪个版本的信息时，**果断使用 `ANY JOIN`**。这能有效防止数据意外膨胀。

* <span style="color: rgb(216,57,49); background-color: inherit">在大多数数据仓库的场景中，</span>**`ANY JOIN`<span style="color: rgb(216,57,49); background-color: inherit"> 更常用，也更安全</span>**<span style="color: rgb(216,57,49); background-color: inherit">。</span>

#### **`GLOBAL JOIN` 在分布式查询中的应用**

我们的相亲大会现在升级了，办成了全国连锁！在 ClickHouse 集群的多个分片节点上，都有事实表（`orders`）的一部分。而维度表（`users`）通常很小，我们希望每个分会场都能人手一份“嘉宾名册”。

#### 普通 JOIN 的窘境：低效的“分发”

如果没有 `GLOBAL` 关键字，ClickHouse 会这样做：

1. 把左表（`orders`）的 join key 发送到包含右表（`users`）的节点。

2. 或者更糟，把**整个右表**从它的节点复制一份，发送到**每一个**参与查询的左表分片节点。

这就像每个分会场都要自己派人去总部复印一份嘉宾名册，网络传输开销巨大。

**【图四：普通 JOIN 在分布式环境下的低效】**

![](<images/ClickHouse 高性能实时分析数据库-image-33.png>)

`GLOBAL` 关键字告诉 ClickHouse：“请改变策略！”

1. 协调节点首先在自己这里，把完整的右表（`users`）准备好。（如果右表是子查询，会先计算出子查询的结果）。

2. 然后，将这份准备好的右表数据**一次性**<span style="color: rgb(216,57,49); background-color: inherit">广播</span>给所有参与查询的分片节点。

3. 每个分片节点在本地用自己那部分的左表数据，和收到的这份右表数据进行 JOIN。

这就像总部统一印好了名册，用快递一次性发给所有分会场，高效且节省网络资源。

**：GLOBAL JOIN 在分布式环境下的高效**

![](<images/ClickHouse 高性能实时分析数据库-image-34.png>)

```sql
-- 假设 orders_distributed 是一个分布式表
-- users 是一个存在于某个节点上的普通表

-- 强烈推荐的分布式 JOIN 写法
SELECT
    o.order_id,
    u.name
FROM orders_distributed AS o
-- 使用 GLOBAL 关键字
GLOBAL ANY LEFT JOIN users AS u ON o.user_id = u.user_id;

-- 如果右表本身也是一个需要计算的分布式表，可以用子查询
SELECT ...
FROM orders_distributed AS o
GLOBAL ANY LEFT JOIN (
    SELECT DISTINCT user_id, name FROM users_distributed -- 先对右表去重
) AS u ON o.user_id = u.user_id;
```

**总结**

1. **宽表优先，JOIN 是补充**：ClickHouse 是为宽表扫描而生的。应尽可能在数据接入层就完成数据拼装，形成宽表。`JOIN` 更适合用于关联更新不频繁、数据量较小的维度表。

2. **`ANY JOIN` 是你的安全带**：在不确定维度表数据质量时，使用 `ANY JOIN` 可以防止数据被意外放大。

3. **分布式查询，`GLOBAL` 不离手**：在集群环境中，对维表使用 `GLOBAL ... JOIN` 是提升性能、减少网络开销的标准操作。

4. **右表要小**：ClickHouse 的 `JOIN` 实现是把右表加载到内存中，构建哈希表，然后流式地与左表进行匹配。因此，**右表必须能完全放入内存**。如果右表太大，查询会失败。

**最终建议**：理解 ClickHouse `JOIN` 的特性，但不要滥用它。把 `JOIN` 当作一个有用的工具，而不是解决所有问题的银弹。真正的 ClickHouse 高手，更懂得如何通过巧妙的 Schema 设计来**避免**复杂的 `JOIN`。

### &#xA;5.3. **子查询与 CTE (Common Table Expressions)**

#### 子查询

##### `1 FROM`子句中的子查询 (派生表)

这是最常见的一种用法。子查询的结果被当作一个临时的、虚拟的表（也叫派生表），供外部主查询使用。

* **作用**：对数据进行预处理、预聚合，为主查询提供一个干净、整洁的数据源。

* **好比**：在盖房子前，先把沙子、石子和水泥搅拌成混凝土（预处理），然后再用这个混凝土来浇筑墙体（主查询）。

**场景**：计算每个用户的平均订单金额，并筛选出平均金额大于1000元的用户。

```sql
-- 步骤1: 先用子查询计算出每个用户的平均订单金额
-- 步骤2: 主查询再从这个结果中进行筛选

SELECT
    user_id,
    avg_amount
FROM ( -- <-- 这里是 FROM 子查询的开始
    SELECT
        user_id,
        avg(amount) AS avg_amount
    FROM orders
    GROUP BY user_id
) AS user_avg -- <-- 必须给子查询的结果起一个别名
WHERE avg_amount > 1000;
```

![](<images/ClickHouse 高性能实时分析数据库-image-35.png>)

##### 2. `WHERE`子句中的子查询 (过滤条件)

子查询可以返回一个值或一列值，用在 `WHERE` 子句中，与 `IN`, `NOT IN`, `=`, `>`, `<` 等操作符结合，动态生成过滤条件。

* **作用**：根据一个查询的结果来过滤另一个查询。

* **好比**：你想邀请所有“住在北京的朋友”参加派对。你得先查一下你的通讯录，列出“住在北京的朋友”的名单（子查询），然后根据这个名单发出邀请（主查询）。

**场景**：找出所有购买过“iPhone 15”的用户的**所有**订单记录。

```sql
-- 子查询找出所有购买过 'iPhone 15' 的 user_id 列表
SELECT *
FROM orders
WHERE user_id IN ( -- <-- IN 操作符与子查询结合
    SELECT DISTINCT user_id
    FROM order_items
    WHERE product_name = 'iPhone 15'
);
```

**<span style="color: rgb(216,57,49); background-color: inherit">性能提示</span>**：**在 ClickHouse 中，`IN` 子查询通常会被自动优化成 `JOIN`。对于大规模数据，`GLOBAL IN` 的性能往往优于 `GLOBAL JOIN`，因为它在网络传输和处理上更高效。**

##### 3. `SELECT`子句中的子查询 (标量子查询)

这种子查询必须只返回**单个值**（一行一列），被称为标量子查询。

* **作用**：为查询的每一行计算出一个独立的关联值。

* **好比**：在员工列表里，为每个人都附加上“他所在部门的总人数”这一列信息。

**场景**：在订单列表中，为每一笔订单都附加上该用户的总订单数。

```sql
-- 注意：这种用法在 ClickHouse 中支持有限，且通常性能不佳。
-- 很多情况下，使用窗口函数或 JOIN 是更好的选择。

SELECT
    order_id,
    user_id,
    amount,
    ( -- <-- SELECT 子查询
        SELECT count()
        FROM orders AS o2
        WHERE o2.user_id = o1.user_id
    ) AS user_total_orders
FROM orders AS o1;
```

**强烈建议**：对于上述场景，使用窗口函数是现代且高效的解决方案：

```sql
SELECT
    order_id,
    user_id,
    amount,
    count() OVER (PARTITION BY user_id) AS user_total_orders
FROM orders AS o1;
```

#### CTE (Common Table Expressions) —— 高级的“预制件”&#x20;

##### 概述

CTE，通过 `WITH` 关键字定义，可以看作是子查询的“超集”。它允许你给一个子查询（或多个子查询）命名，然后在后续的查询中像使用普通表一样多次引用它。

* **作用**：

  1. **提高可读性**：将复杂的逻辑拆分成多个独立的、有意义的步骤。

  2. **避免重复**：同一个计算逻辑可以被多次引用，无需重复编写。

  3. **支持递归**（ClickHouse 暂不支持递归CTE）：在处理层级或图状数据时非常有用。

* **好比**：在盖一座复杂的城堡时，你不是用零散的积木，而是先在工厂里预制好“窗户模块”、“墙体模块”、“屋顶模块”（CTE），然后在施工现场直接组装这些“预制件”。

**如何使用 CTE**

```sql
WITH
    cte_name_1 AS ( -- 第一个预制件
        SELECT ...
    ),
    cte_name_2 AS ( -- 第二个预制件，可以引用第一个
        SELECT ... FROM cte_name_1 ...
    )
-- 最后的组装工序
SELECT ...
FROM cte_name_1
JOIN cte_name_2 ON ...;
```

##### 实践: 使用 CTE 重构复杂分析流程

**场景**：分析一个电商网站的数据，找出每个国家销售额最高的商品品类。

**分析步骤**：

1. **步骤一 (预制件1)**：计算每个国家、每个品类的总销售额。

2. **步骤二 (预制件2)**：对上一步的结果进行排名，找出每个国家内部销售额排名第一的品类。

3. **步骤三 (最终组装)**：筛选出排名为1的记录。

```sql
WITH
    -- 预制件1：计算每个国家、每个品类的销售额
    CategorySalesByCountry AS (
        SELECT
            u.country,
            p.category,
            sum(oi.price * oi.quantity) AS total_sales
        FROM order_items AS oi
        JOIN orders AS o ON oi.order_id = o.order_id
        JOIN users AS u ON o.user_id = u.user_id
        JOIN products AS p ON oi.product_id = p.product_id
        GROUP BY u.country, p.category
    ),

    -- 预制件2：使用窗口函数对销售额进行排名
    RankedSales AS (
        SELECT
            country,
            category,
            total_sales,
            -- 对每个国家(PARTITION BY)内部，按销售额倒序排名
            row_number() OVER (PARTITION BY country ORDER BY total_sales DESC) AS rn
        FROM CategorySalesByCountry
    )

-- 最终组装：筛选出排名第一的记录
SELECT
    country,
    category,
    total_sales
FROM RankedSales
WHERE rn = 1;
```

##### **CTE 如同清晰的流程图**

![](<images/ClickHouse 高性能实时分析数据库-image-36.png>)

##### 总结

**核心建议**：

* 对于非常简单的单步逻辑（如 `WHERE id IN (SELECT ...)`），使用子查询是可接受的。

* **一旦查询逻辑超过一步，或者某个中间结果需要被多次使用，就应该毫不犹豫地使用 CTE。**

* 养成使用 CTE 的习惯，是编写高质量、可维护 SQL 代码的关键一步。它能让你的同事（以及未来的你）在读你的代码时，对你感激不尽！

ClickHouse 强大的计算引擎，配上子查询和 CTE 这两样称手的“逻辑工具”，你就可以搭建出任何你想要的数据分析模型。现在，去用这些“乐高积木”创造属于你的数据故事吧！

### 5.4. **Array/Tuple/Map 函数**

#### 第一：Array (数组) —— 数据的“有序列表” (25分钟)

数组是 ClickHouse 中最常用、功能最丰富的复杂类型。它是一个有序的元素集合，所有元素的类型必须相同。比如 `[1, 2, 3]` 是一个 `Array(UInt8)`，`['a', 'b', 'c']` 是一个 `Array(String)`。

创建与访问

* **创建**: `[val1, val2, ...]` 或 `array(val1, val2, ...)`

* **访问**: `arr[index]` (注意：ClickHouse 的数组索引**从1开始**！)

##### **1. `arrayJoin(arr)`: 王牌函数，展开利器**

* **作用**：将一行数据根据数组中的每个元素“炸裂”成多行。这是处理数组数据的**最核心、最常用**的操作。

* **好比**：一把“分身斧”，把一个怀揣一袋子苹果的人（一行数据），变成多个每人只拿一个苹果的人（多行数据）。

**<span style="color: rgb(216,57,49); background-color: inherit">场景</span>**<span style="color: rgb(216,57,49); background-color: inherit">：统计每个产品标签的热度。</span>
**表 `products`**:
\| product\_id | tags |
\|---|---|
\| 101 | `['手机', '数码', '特价']` |
\| 102 | `['家电', '特价']` |

```sql
create  table  products (
id  UInt8 ,
tags Array(String)
)engine=Log ;
insert into products values(101 ,['手机','数码','特价']) ;
insert into products values(102 ,['电脑','办公','特价']) ;    
arr[IDX-1]取值

SELECT
    tag,
    count() AS product_count
FROM products
ARRAY JOIN tags AS tag -- 使用 ARRAY JOIN 展开 tags 数组
GROUP BY tag;

|---|---|
| '特价' | 2 |
| '手机' | 1 |
| '数码' | 1 |
| '家电' | 1 |
```

**`arrayJoin` 的威力**

![](<images/ClickHouse 高性能实时分析数据库-image-37.png>)

##### **2. `arrayMap(func, arr1, ...)`: 批量加工 &#x20;**

对数组中的每个元素处理!将处理后的结果放在数组中返回

* **作用**：对数组中的每个元素应用一个 lambda 函数，返回一个结果数组。

* **好比**：一条“自动化加工流水线”，把一篮子土豆（原数组）送进去，出来一篮子薯条（新数组）。

**场景**：将一个包含 URL 的数组中的所有 URL 提取出域名。

```sql

create  table  tb_names (
id UInt8 , 
names Array(String)
)engine=Log ;
insert  into  tb_names (1 , ['a' ,'b' ,'ab' , 'ao']) ;
arrayMap
 参数1  函数  处理逻
 参数2   数组
-- `domain` 是一个提取域名的函数
SELECT
    urls,
    arrayMap(x -> domain(x), urls) AS domains
FROM (
    SELECT ['https://clickhouse.com/docs', 'https://github.com/ClickHouse'] AS urls
);

| ['https://clickhouse.com/docs', 'https://github.com/ClickHouse']
| ['clickhouse.com', 'github.com']
```

##### **3. `arrayFilter(func, arr1, ...)`: 精准筛选**

* **作用**：使用 lambda 函数过滤数组，只保留让函数返回 true 的元素。

* **好比**：一个“筛子”，从一堆沙石中只留下你想要的鹅卵石。

**场景**：从一个记录了用户每次会话访问页面时长的数组中，筛选出所有时长超过60秒的记录。

```sql
SELECT
    session_durations,
    arrayFilter(t -> t > 60, session_durations) AS long_sessions
FROM (
    SELECT [10, 120, 35, 300] AS session_durations
);
| session_durations | long_sessions |
|---|---|
| [10, 120, 35, 300] | [120, 300] |
```

##### **4. `has(arr, elem)` & `indexOf(arr, elem)`: 查找与定位**

* `has(arr, elem)`: 判断数组是否包含某个元素，返回 `1` (true) 或 `0` (false)。

* `indexOf(arr, elem)`: 返回元素在数组中首次出现的位置（从1开始），如果不存在则返回 `0`。

* **好比**：`has` 就像问“这个盒子里有苹果吗？”，而 `indexOf` 则是问“苹果在第几个位置？”。

**场景**：筛选出所有访问过“/cart”页面的用户会话。

```sql
SELECT session_id, visited_pages
FROM user_sessions
WHERE has(visited_pages, '/cart'); -- 使用 has 进行高效过滤
```



#### 第二：Tuple (元组) —— 固定的“数据包”

元组是一个有序的元素集合，但与数组不同，它的**长度固定**，且**每个元素可以有不同的类型**。它就像一个轻量级的、匿名的结构体。

创建与访问

* **创建**: `(val1, val2, ...)` 或 `tuple(val1, val2, ...)`

* **访问**: `t.index` (通过索引，从1开始) 或 `t.name` (如果创建时指定了名称)。

**场景**：函数返回多个值。比如，`minmax` 聚合函数就会返回一个包含最小值和最大值的元组。

```sql
SELECT
    group_key,
    minmax(value) AS min_max_tuple,
    min_max_tuple.1 AS min_value, -- 按索引访问
    min_max_tuple.2 AS max_value  -- 按索引访问
FROM some_table
GROUP BY group_key;
```

**元组与数组的配合**: `arrayMap` 的 lambda 函数可以操作元组，这在处理并列数组时非常有用。

**场景**：我们有两个数组，`events` 和 `timestamps`，它们是一一对应的。我们想筛选出所有 'login' 事件及其对应的时间。

```sql
SELECT
    arrayFilter(
        (evt, ts) -> evt = 'login', -- lambda函数接收元组 (event, timestamp)
        events,
        timestamps
    ) AS login_timestamps
FROM (
    SELECT
        ['view', 'login', 'purchase', 'login'] AS events,
        [100, 200, 300, 400] AS timestamps
);
```

~~**解释**：`arrayFilter` 聪明地将 `events` 和 `timestamps` 两个数组打包成 `(event, timestamp)` 元组流进行处理。~~



#### 第三：Map (映射) —— 灵活的“键值对”&#x20;

Map 类型用于存储键值对。键和值的类型需要被指定，例如 `Map(String, UInt64)`。它非常适合存储动态的、非固定的属性。

创建与访问

* **创建**: `map(key1, val1, key2, val2, ...)`

* **访问**: `map_name[key]`

**场景**：存储用户画像的动态标签，或者HTTP请求的Header。

```sql
表 user_profiles:
| user_id | properties |
|---|---|
| 1 | {'age': 30, 'level': 5} |
| 2 | {'age': 25, 'city_id': 101, 'level': 3} |
```

```sql
-- 查询用户的年龄
SELECT
    user_id,
    properties['age'] AS age
FROM user_profiles;

-- 筛选出等级大于3的用户
SELECT user_id
FROM user_profiles
WHERE properties['level'] > 3;

-- 检查是否存在某个键
SELECT user_id
FROM user_profiles
WHERE mapContains(properties, 'city_id');
```

![](<images/ClickHouse 高性能实时分析数据库-image-38.png>)

#### 第四 终极挑战

在一个记录了用户在APP内点击事件序列的表中，找出所有“先点击了商品详情页，之后又加入了购物车”的用户。

```sql
表 click_stream:
| user_id | event_names | event_timestamps |
|---|---|---|
| 'A' | ['open_app', 'view_product', 'add_to_cart'] | [100, 200, 300] |
| 'B' | ['add_to_cart', 'view_product'] | [110, 220] |
| 'C' | ['view_product', 'view_other', 'open_app']| [150, 250, 350] |
```

```sql
SELECT DISTINCT user_id
FROM click_stream
WHERE
    -- 1. 找到 'view_product' 的位置
    indexOf(event_names, 'view_product') AS view_pos
    > 0 -- 确保存在 'view_product'
AND
    -- 2. 找到 'add_to_cart' 的位置
    indexOf(event_names, 'add_to_cart') AS cart_pos
    > 0 -- 确保存在 'add_to_cart'
AND
    -- 3. 确保 'add_to_cart' 的位置在 'view_product' 之后
    cart_pos > view_pos;
```

**结果**: 只会返回 `user_id: 'A'`。这个查询完全在行内完成了复杂的时序路径分析，没有使用任何 `JOIN` 或子查询，展现了数组函数的强大威力！

**总结**：

* **Array** 是处理有序序列、进行批量操作和展开数据的利器。`arrayJoin` 是你必须掌握的王牌。

* **Tuple** 是处理固定结构、多值返回的轻量级工具，常与数组函数配合使用。

* **Map** 是处理动态键值对属性的完美选择，让你的表结构更具弹性。

掌握了这套“瑞士军刀”，你就能以一种更 ClickHouse 的方式思考和解决问题，编写出更简洁、更高效、更具表现力的查询。


5.5. **窗口函数 (Window Functions)**

在数据分析中，我们经常需要进行聚合计算，比如 `SUM()`, `AVG()`, `COUNT()`。通常，我们会使用 `GROUP BY` 来实现。

但 `GROUP BY` 有一个特点：它会**改变表的行数**。它将多行数据“压缩”成一行聚合结果。

**思考一个场景**：我想看每一笔订单的金额，**同时**，我还想看这笔订单所在部门的**总销售额**，以便于计算该订单的销售额占比。

* 如果用 `GROUP BY`，我们只能得到每个部门的总销售额，原始的每一笔订单信息就丢失了。

* 如果不用 `GROUP BY`，我们又无法得到部门的总销售额。

这时候，**窗口函数** 就闪亮登场了。

> **核心思想**：窗口函数允许你在与当前行相关的“窗口”（一组行）上执行计算，但**不改变原始表的行数**。计算结果会作为新的一列，附加到每一行上。

核心语法

```sql
FUNCTION_NAME() OVER (
    [PARTITION BY partition_expression, ... ]
    [ORDER BY sort_expression [ASC|DESC], ... ]
    [frame_clause]
)
```

1. **`FUNCTION_NAME()`**: 要执行的函数。可以是聚合函数 (`SUM`, `AVG`)，也可以是专门的窗口函数 (`RANK`, `LAG`)。

2. **`OVER()`**: 标志着这是一个窗口函数。所有窗口函数的逻辑都在 `OVER()` 的括号内定义。

3. **`PARTITION BY` (分区)**: 这是“窗口”的边界。它将数据按指定的列（如 `department_id`）分成不同的逻辑分区（或“窗口”）。函数将在每个分区内独立计算。**可以类比为 `GROUP BY`，但它不合并行**。

4. **`ORDER BY` (排序)**: 在每个分区内部，对行进行排序。这对于需要顺序的函数（如 `RANK`, `LAG`, `LEAD`）至关重要。

5. **`frame_clause` (帧)**: 这是最精细的部分，它定义了在分区内部，当前行要计算的**具体子集**（“帧”）。例如，“当前行及其前两行”。

> **`PARTITION BY` 的作用**

假设我们有一张销售数据表，`PARTITION BY department` 会像这样在逻辑上切分数据：

![](<images/ClickHouse 高性能实时分析数据库-image-39.png>)

* **解释**：`PARTITION BY` 将整个表分成了两个独立的“窗口”：销售部和技术部。后续的计算将在这两个窗口内分别进行。



> **&#x20;`ORDER BY` 的作用**

在分区的基础上，`ORDER BY amount DESC` 会在每个窗口内进行排序：

![](<images/ClickHouse 高性能实时分析数据库-image-40.png>)

* **解释**：在销售部窗口内，数据按销售额降序排列。技术部窗口内也同样如此。这个顺序对于 `RANK()` 等函数至关重要。

> **工作原理**

![](<images/ClickHouse 高性能实时分析数据库-image-41.png>)

1. **分区 (Partitioning)**: 根据 `PARTITION BY` 子句，将所有行分成多个逻辑分区。如果省略 `PARTITION BY`，则整个结果集被视为一个分区。

2. **排序 (Ordering)**: 根据 `ORDER BY` 子句，对每个分区内的行进行排序。

3. **定义帧 (Framing)**: 对于分区中的每一行，根据 `frame_clause` 定义一个计算子集（帧）。如果省略 `frame_clause`，默认行为通常是 `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`。

4. **函数计算 (Calculation)**: 将窗口函数应用于帧内的行，并为当前行计算出结果。

5. **生成结果**: 将计算结果附加到原始行的末尾，形成最终输出。



> 常用的窗口函数

让我们创建一张示例表来进行实战演练。

```sql
-- 创建销售明细表
CREATE TABLE sales
(
    `sale_date` Date,
    `category` String,
    `name` String,
    `amount` UInt32
)
ENGINE = Memory;

-- 插入示例数据
INSERT INTO sales VALUES
('2025-11-01', '服饰', 'Alice', 300),
('2025-11-01', '电器', 'Bob', 500),
('2025-11-02', '服饰', 'Charlie', 200),
('2025-11-02', '电器', 'David', 600),
('2025-11-03', '服饰', 'Alice', 350),
('2025-11-03', '电器', 'Bob', 550),
('2025-11-03', '服饰', 'Eve', 250);

INSERT INTO sales VALUES
('2025-11-03', '服饰', 'AO', 250) ;
```

在 `OVER()` 子句中时，它们就变成了窗口函数。

`SUM()`<span style="color: rgb(216,57,49); background-color: inherit">, </span>`AVG()`<span style="color: rgb(216,57,49); background-color: inherit">, </span>`COUNT()`<span style="color: rgb(216,57,49); background-color: inherit">, </span>`MAX()`<span style="color: rgb(216,57,49); background-color: inherit">, </span>`MIN()`

**需求**：计算每笔销售额，并同时展示该销售员的**个人总销售额**和所在**部门的总销售额**。

```sql
SELECT
    sale_date,
    department,
    employee,
    amount,
    -- 计算每个员工的总销售额 (窗口按员工分区)
    sum(amount) OVER (PARTITION BY name) AS employee_total_sales,
    -- 计算每个部门的总销售额 (窗口按部门分区)
    sum(amount) OVER (PARTITION BY category) AS department_total_sales
FROM sales
ORDER BY department, employee, sale_date  ;

   ┌──sale_date─┬─department─┬─employee─┬─amount─┬─employee_total_sales─┬─department_total_sales─┐
1. │ 2025-11-01 │ 服饰       │ Alice    │    300 │                  650 │                   1100 │
2. │ 2025-11-03 │ 服饰       │ Alice    │    350 │                  650 │                   1100 │
3. │ 2025-11-02 │ 服饰       │ Charlie  │    200 │                  200 │                   1100 │
4. │ 2025-11-03 │ 服饰       │ Eve      │    250 │                  250 │                   1100 │
5. │ 2025-11-01 │ 电器       │ Bob      │    500 │                 1050 │                   1650 │
6. │ 2025-11-03 │ 电器       │ Bob      │    550 │                 1050 │                   1650 │
7. │ 2025-11-02 │ 电器       │ David    │    600 │                  600 │                   1650 │
   └────────────┴────────────┴──────────┴────────┴──────────────────────┴────────────────────────┘
```

* **分析**：注意 `employee_total_sales` 列。对于 Alice 的两条记录，该值都是 650 (300+350)。`department_total_sales` 也是同理，所有服饰部门的记录，该值都为 1100。**原始行数没有改变！**

<span style="color: rgb(216,57,49); background-color: inherit">这类函数必须和 </span>`ORDER BY`<span style="color: rgb(216,57,49); background-color: inherit"> 配合使用。</span>

* `RANK()`: 排名，如果值相同，排名也相同，但后续排名会跳跃。 (1, 2, 2, 4)

* `DENSE_RANK()`: 密集排名，值相同排名相同，后续排名不跳跃。 (1, 2, 2, 3)

* `ROW_NUMBER()`: 行号，无论值是否相同，都分配一个连续的唯一编号。 (1, 2, 3, 4)

**需求**：在每个部门内，按销售额对员工业绩进行排名。

```sql
SELECT
    department,
    employee,
    amount,
    -- 在部门内按销售额降序排名
    RANK() OVER (PARTITION BY department ORDER BY amount DESC) AS rnk,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY amount DESC) AS dense_rnk,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY amount DESC) AS rn
FROM sales
ORDER BY department, rnk;

   ┌─department─┬─employee─┬─amount─┬─rnk─┬─dense_rnk─┬─rn─┐
1. │ 服饰       │ Alice    │    350 │   1 │         1 │  1 │
2. │ 服饰       │ Alice    │    300 │   2 │         2 │  2 │
3. │ 服饰       │ Eve      │    250 │   3 │         3 │  3 │
4. │ 服饰       │ Charlie  │    200 │   4 │         4 │  4 │
5. │ 电器       │ David    │    600 │   1 │         1 │  1 │
6. │ 电器       │ Bob      │    550 │   2 │         2 │  2 │
7. │ 电器       │ Bob      │    500 │   3 │         3 │  3 │
   └────────────┴──────────┴────────┴─────┴───────────┴────┘
```

* **分析**：排序函数在 `PARTITION BY department` 定义的窗口内，根据 `ORDER BY amount DESC` 计算排名。



<span style="color: rgb(216,57,49); background-color: inherit">这类函数可以获取当前行之前或之后的行的值。</span>

* `LAG(expr, offset, default)`: 获取当前行往前第 `offset` 行的 `expr` 的值。

* `LEAD(expr, offset, default)`: 获取当前行往后第 `offset` 行的 `expr` 的值。

**需求**：计算每个员工**相比于上一天**的销售额变化。

```sql
SELECT
    employee,
    sale_date,
    amount,
    -- 获取该员工上一个日期的销售额
    -- 窗口按员工分区，并按日期排序
    lag(amount, 1, 0) OVER (PARTITION BY employee ORDER BY sale_date) AS prev_day_amount,
    amount - prev_day_amount AS diff
FROM sales
ORDER BY employee, sale_date;

   ┌─employee─┬──sale_date─┬─amount─┬─prev_day_amount─┬─diff─┐
1. │ Alice    │ 2025-11-01 │    300 │               0 │  300 │
2. │ Alice    │ 2025-11-03 │    350 │             300 │   50 │
3. │ Bob      │ 2025-11-01 │    500 │               0 │  500 │
4. │ Bob      │ 2025-11-03 │    550 │             500 │   50 │
5. │ Charlie  │ 2025-11-02 │    200 │               0 │  200 │
6. │ David    │ 2025-11-02 │    600 │               0 │  600 │
7. │ Eve      │ 2025-11-03 │    250 │               0 │  250 │
   └──────────┴────────────┴────────┴─────────────────┴──────┘
```

* **分析**：`lag(amount, 1, 0)` 在按员工分区、按日期排序的窗口中，找到了上一行的 `amount` 值。对于每个员工的第一条记录，由于没有“上一行”，所以返回了默认值 `0`。

> 深入 Frame 子句 —— 移动平均计算

Frame 子句让窗口函数变得更加强大，它能定义一个“滑动”的帧。
语法：`{ROWS | RANGE} BETWEEN frame_start AND frame_end`

* `ROWS`: 基于物理行数偏移。`ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` 指的是“前一行，当前行，后一行”。

* `RANGE`: 基于值的逻辑偏移。`RANGE BETWEEN 10 PRECEDING AND CURRENT ROW` 指的是“值在 `[current_row_value - 10, current_row_value]` 区间内的所有行”。`RANGE` 依赖 `ORDER BY` 列。

* 常用边界：

  * `UNBOUNDED PRECEDING`: 帧从分区的开头开始。

  * `n PRECEDING`: 当前行之前的 `n` 行。

  * `CURRENT ROW`: 当前行。

  * `n FOLLOWING`: 当前行之后的 `n` 行。

  * `UNBOUNDED FOLLOWING`: 帧到分区的结尾结束。

**需求**：计算每个员工**最近两天（含当天）的移动平均销售额**。

```sql
SELECT
    employee,
    sale_date,
    amount,
    -- 帧定义为：当前行和它前面的一行
    avg(amount) OVER (
        PARTITION BY employee
        ORDER BY sale_date
        ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
    ) AS moving_avg_2_days
FROM sales
ORDER BY employee, sale_date;

   ┌─employee─┬──sale_date─┬─amount─┬─moving_avg_2_days─┐
1. │ Alice    │ 2025-11-01 │    300 │               300 │
2. │ Alice    │ 2025-11-03 │    350 │               325 │
3. │ Bob      │ 2025-11-01 │    500 │               500 │
4. │ Bob      │ 2025-11-03 │    550 │               525 │
5. │ Charlie  │ 2025-11-02 │    200 │               200 │
6. │ David    │ 2025-11-02 │    600 │               600 │
7. │ Eve      │ 2025-11-03 │    250 │               250 │
   └──────────┴────────────┴────────┴───────────────────┘
```

* **分析**：对于 Alice 在 11-03 的记录，其帧包含了 `1 PRECEDING` (11-01 的记录) 和 `CURRENT ROW` (11-03 的记录)，所以移动平均是 `(300 + 350) / 2 = 325`。

1. **核心区别**：窗口函数 vs `GROUP BY`。窗口函数**不减少行数**，保留明细数据，非常适合在明细旁附加聚合或排名信息。

2. **语法三要素**：`PARTITION BY` (切分窗口)、`ORDER BY` (窗口内排序)、`frame_clause` (定义计算帧)。

3. **常用场景**：

   * **聚合分析**：计算占比（行值 / 窗口总和）。

   * **排名问题**：Top N 分析（各部门销售前三名）。

   * **趋势分析**：同比/环比计算（使用 `LAG`）、移动平均计算（使用 Frame）。

4. **实践出真知**：窗口函数功能强大但初学时可能感到困惑，最好的学习方式是结合具体业务场景，多写多练。

### &#xA;5.6. **物化视图 (Materialized Views)**

在数据仓库领域，我们经常会遇到这样的场景：

1. **原始数据量巨大**：日志、事件流等数据以极高的速度写入。

2. **查询模式固定**：分析师或仪表盘（Dashboard）总是对这些原始数据进行**固定的聚合查询**，例如：

   * 每分钟的网站访问量 (PV/UV)

   * 每个商品的日销售额

   * 每个接口的平均响应时间

如果每次查询都直接扫描原始数据表，即使 ClickHouse 性能卓越，当数据量达到千亿甚至万亿级别时，查询延迟也会增加，计算资源消耗巨大。

**普通视图 (View) 能解决问题吗？**
不能。普通视图只是一个**查询别名**，它不存储任何数据。每次查询视图时，实际上还是在执行视图定义中的那个复杂查询，扫描原始表。

**物化视图 (Materialized View) 的诞生**
为了解决这个问题，物化视图应运而生。

> **核心思想**：物化视图是一种**预计算**和**持久化存储**的机制。它像一个“数据触发器”，当其监控的源表有新数据写入时，它会自动对这些新数据执行一个 `SELECT` 查询，并将结果\*\*物化（存储）\*\*到一个独立的目标表中。后续的查询可以直接访问这个小得多的、预聚合过的目标表，从而实现毫秒级的查询响应。

在 ClickHouse 中，物化视图的概念与其他数据库（如 Oracle, PostgreSQL）有本质区别。理解这一点至关重要。

**ClickHouse 物化视图 = 触发器 (Trigger)**

你可以把 ClickHouse 的物化视图理解为一个**插入触发器**。它本身不存储数据，它只是一个**数据转换和搬运的规则**。

![](<images/ClickHouse 高性能实时分析数据库-image-42.png>)

1. **数据写入**：当有 `INSERT` 操作将一批数据写入**源表** (Source Table) 时。

2. **触发执行**：该 `INSERT` 操作会**自动触发**所有监听此源表的物化视图。

3. **数据转换**：物化视图执行其 `AS SELECT ...` 子句中定义的查询，但**只针对刚刚插入的新数据块**进行计算。

4. **写入目标表**：物化视图将转换后的结果集，以 `INSERT` 的方式写入到预先定义好的**目标表** (Target Table) 中。

5. **查询加速**：用户查询时，直接查询这个已经预聚合过的、数据量小得多的**目标表**，而不是庞大的源表。

> **重要区别**：
>
> * **物化视图本身 (my\_mv)**: 是一个没有数据的**逻辑定义/触发器**。`SELECT * FROM my_mv` 通常是无意义的。
>
> * **目标表 (target\_table)**: 是一个**真实的、存储了预计算结果的物理表**。我们最终查询的是它。

> **场景**：实时统计网站每分钟的页面浏览量（PV）和独立访客数（UV）。

```sql
-- 存储原始访问日志
CREATE TABLE visits_raw
(
    `timestamp` DateTime,
    `url` String,
    `user_id` String
)
ENGINE = MergeTree()
PARTITION BY toYYYYMMDD(timestamp)
ORDER BY (timestamp, url);

INSERT INTO visits_raw (timestamp, url, user_id) VALUES
(now(), '/page/1', 'user-a'),
(now(), '/page/2', 'user-b'),
(now(), '/page/1', 'user-a'),
(now() + 2, '/page/3', 'user-c');
```

目标表用于存储预聚合的结果。这里的表引擎选择至关重要。对于聚合场景，`AggregatingMergeTree` 是最佳选择。

> **AggregatingMergeTree 引擎**：
> 它专门用于增量聚合。它会将 `SELECT` 查询中带有 `-State` 后缀的聚合函数（如 `countState`, `uniqState`）的**中间状态**存储起来。在查询时，再使用 `-Merge` 后缀的函数（如 `countMerge`, `uniqMerge`）来完成最终计算。这使得后台合并和最终查询都极为高效。

```sql
-- 存储每分钟的 PV/UV 聚合结果
CREATE TABLE visits_agg_daily
(
    `minute` DateTime, -- 聚合时间粒度：分钟
    `pv` AggregateFunction(count), -- 存储 PV 的中间状态
    `uv` AggregateFunction(uniq, String) -- 存储 UV 的中间状态
)
ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMMDD(minute)
ORDER BY minute;
```

现在，我们创建物化视图，将源表和目标表连接起来。

```sql
CREATE MATERIALIZED VIEW visits_mv TO visits_agg_daily -- 关键：指定数据流向的目标表
AS
SELECT
    toStartOfMinute(timestamp) AS minute, -- 按分钟聚合
    countState() AS pv, -- 计算 PV 的中间状态
    uniqState(user_id) AS uv -- 计算 UV 的中间状态
FROM
    visits_raw -- 关键：指定监听的源表
GROUP BY
    minute;
```

**代码解析**：

* `TO visits_agg_daily`: 明确指出，计算结果将写入 `visits_agg_daily` 表。

* `AS SELECT ... FROM visits_raw`: 定义了转换逻辑。当 `visits_raw` 插入新数据时，就执行这个 `SELECT`。

* `toStartOfMinute(timestamp)`: 将时间戳对齐到分钟级别。

* `countState()`, `uniqState(user_id)`: 使用 `-State` 函数，生成与 `AggregatingMergeTree` 兼容的聚合中间态。

* `GROUP BY minute`: 按分钟进行聚合。



正确的查询方式

```sql
SELECT
    minute,
    countMerge(pv) AS page_views,
    uniqMerge(uv) AS unique_visitors
FROM
    visits_agg_daily
GROUP BY
    minute
ORDER BY
    minute;
```

**结果**：

**至此，我们成功搭建了一个全自动的实时聚合系统！** 后续任何写入 `visits_raw` 的数据，都会被 `visits_mv` 自动处理并更新到 `visits_agg_daily` 中。分析查询只需要访问 `visits_agg_daily` 即可。

***

物化视图只对**创建之后**新插入的数据生效。如果源表在创建物化视图之前就已经存在大量数据，怎么办？

使用 `POPULATE` 关键字！它会在创建视图的同时，将源表中的存量数据也进行一次转换并插入目标表。

```sql
CREATE MATERIALIZED VIEW visits_mv TO visits_agg_daily
POPULATE -- <<<<<<<<<<<<<<<<
AS
SELECT ...
```

* **`AggregatingMergeTree`**: 最常用，适用于需要复杂聚合（如 `uniq`, `avg`, `quantile`）且要求高性能的场景。

* **`SummingMergeTree`**: 如果你的聚合需求只有“求和”与“计数”，可以使用它。它会自动合并具有相同排序键的行，将度量列相加。比 `AggregatingMergeTree` 更简单。

* **普通 `MergeTree`**: 如果你不需要预聚合，只是想对数据进行ETL（比如清洗、转换、拆分列），可以将目标表设置为普通 `MergeTree`。

* **删除物化视图**: `DROP VIEW visits_mv;`

  * 注意：删除物化视图**不会**删除目标表 `visits_agg_daily`。它只是切断了自动的数据流。目标表中的数据仍然存在。

* **暂停/恢复**:

  * **`DETACH TABLE visits_mv;`<span style="color: rgb(216,57,49); background-color: inherit"> 可以临时禁用物化视图。</span>**

  * **`ATTACH TABLE visits_mv;`<span style="color: rgb(216,57,49); background-color: inherit"> 可以重新激活它。</span>**

你可以创建一个物化视图，其目标表同时是另一个物化视图的源表，形成数据处理流水线（Pipeline）

![](<images/ClickHouse 高性能实时分析数据库-image-43.png>)

* `MV1` 从 `Source Table` 读取数据，聚合后写入 `a_min`。

* `MV2` 监听 `a_min`，当 `a_min` 有新数据时，`MV2` 将其进一步聚合到 `a_hour`。

> 总结

**何时使用物化视图？**

1. 当你有**高频的、固定的聚合查询**，且无法接受直接扫描原始大表的延迟时。

2. 当需要构建**实时数据看板**或**实时报表**系统时。

3. 当需要对写入的数据流进行**持续的ETL转换**时。

## 第6章：查询分析与性能调优 (Query Analysis & Tuning)



ClickHouse 以其惊人的查询速度闻名于世，被誉为“性能猛兽”。但即便是最快的跑车，在崎岖的道路上行驶也可能变得缓慢。当您发现 ClickHouse 查询未达到预期速度时，通常不是 ClickHouse 本身的问题，而是我们“驾驶”它的方式有待优化。

**常见性能瓶颈根源**：

1. **数据模型设计不当**：未能充分利用 ClickHouse 的核心优势。

2. **使用合适的引擎**

3. **查询语句编写糟糕**：执行了大量不必要的工作。

4. **硬件资源限制**：CPU、内存、I/O 达到瓶颈。

5. **配置不合理**：服务器参数未根据负载进行调整。



本教案将聚焦于前两点——**数据模型**和**查询优化**，这是我们作为开发者和数据分析师最能直接控制和提升的部分。

要调优，必先理解其原理。ClickHouse 的高性能秘诀主要源于其 `MergeTree` 引擎的几个关键设计。

这是 ClickHouse 与传统行式数据库（如 MySQL）最根本的区别。

> <span style="color: rgb(216,57,49); background-color: inherit">数据使用列式存储  和 索引键</span>

* **性能影响**：当查询 `SELECT avg(Age)` 时，行式数据库需要读取每一行的所有数据（ID, Name, Age），而列式数据库**只需读取 'Age' 这一列的数据**。这极大地减少了 I/O 操作。

* **调优启示**：**永远只 `SELECT` 你需要的列**，`SELECT `**`*`**<span style="color: rgb(216,57,49); background-color: inherit"> 是性能杀手。</span>

这是 ClickHouse 最核心、最需要理解的部分。

`CREATE TABLE` 语句中的 `ORDER BY` 定义了表的**物理排序键**，它也是 ClickHouse 的**一级索引**（或称主键）。

```sql
graph TD
    subgraph MergeTree 表结构与查询流程
        A[查询: WHERE UserID = 'A' AND EventDate = '2023-11-20'] --> B{1. 检查主键索引};
        
        B -- "根据 UserID 和 EventDate 找到对应的 mark (标记)" --> C[2. 读取 Sparse Index (稀疏索引)];
        
        C -- "标记指示数据在 Part 1 和 Part 3 的 Granule 5-10 之间" --> D{3. 跳过不相关的数据块 (Pruning)};
        D -- "跳过 Part 2" --> E((X));
        D -- "读取 Part 1, Part 3 的相关 Granules" --> F[4. 从磁盘读取少量数据];
        F --> G[5. 返回结果];

        subgraph 数据物理存储
            P1[Data Part 1 (UserID: A-C)]
            P2[Data Part 2 (UserID: D-F)]
            P3[Data Part 3 (UserID: A,G-H)]
        end
        C --> P1 & P3
    end
    style D fill:#9f9,stroke:#333
    style E fill:#f99,stroke:#333
```

* **数据块 (Data Parts)**: 数据被存储在多个按主键排序的“数据块”中。

* **稀疏索引 (Sparse Index)**: ClickHouse 不会对每一行建立索引。它每隔 N 行（由 `index_granularity` 定义，默认8192）才创建一个索引条目。这个条目记录了该数据粒度（Granule）的第一个主键值及其在文件中的偏移量。

* **数据裁剪 (Data Pruning)**: 当查询的 `WHERE` 条件命中了主键时，ClickHouse 利用稀疏索引可以快速定位到包含目标数据的**少数几个数据块和粒度**，跳过大量不相关的数据，从而实现极速查询。

* **调优启示**：

  1. **`ORDER BY` (主键) 的选择是 ClickHouse 性能调优的重中之重！** 应选择查询中最常用于`WHERE`过滤、基数从高到低的列（如 `(UserID, EventDate)`）。

  2. `WHERE` 条件尽量使用主键列。

> 侦察兵器 —— 查询分析工具

在优化之前，我们需要定位问题。

`EXPLAIN` 是你的第一站，用于分析查询的执行计划。

* `EXPLAIN SYNTAX query`: 检查语法是否正确，并展示优化后的查询语句。

* `EXPLAIN PLAN query`: 展示查询的执行计划步骤。

  * **关注点**：`ReadFromMergeTree` 步骤。看 `Selected parts` 和 `Selected marks` 的数量。如果这两个值很小，说明索引生效了。如果很大，接近总数，说明索引没起作用，查询正在进行全表扫描。

**示例**：

```sql
EXPLAIN PLAN SELECT count() FROM user_events WHERE UserID = 123;

-- 好的结果 (索引生效)
ReadFromMergeTree (Selected parts: 1, Selected marks: 2)

-- 坏的结果 (全表扫描)
ReadFromMergeTree (Selected parts: 50, Selected marks: 15000)
```

这是性能诊断的“神器”。它记录了每一条查询的详细性能指标。

**首先，确保它已开启**（默认开启）。

**查询慢查询日志**：

```sql
SELECT
    query,
    query_duration_ms,
    read_rows,
    read_bytes,
    memory_usage,
    result_rows,
    result_bytes,
    event_time,
    exception
FROM system.query_log
WHERE type = 'QueryFinish' -- 只看已完成的查询
  AND is_initial_query = 1 -- 只看用户发起的初始查询
ORDER BY query_duration_ms DESC
LIMIT 10;
```

* **关键指标分析**:

  * `query_duration_ms`: 查询耗时，最直接的指标。

  * `read_rows`, `read_bytes`: 读取的行数和字节数。**这两个值如果远大于 `result_rows` 和 `result_bytes`，通常意味着索引效率低下或查询语句有问题。**

  * `memory_usage`: 内存使用情况，过高可能导致 `Memory limit exceeded` 错误。

> 核心调优策略

掌握了原理和分析工具后，我们开始实战优化。

1. **选择正确的主键 (`ORDER BY`)**:

   * **原则**: 将最常用于过滤、基数较高的列放在前面。

   * **反例**: `ORDER BY EventDate`。如果大部分查询是 `WHERE UserID = ?`，那么这个主键几乎无效。

   * **正例**: `ORDER BY (UserID, EventType, EventDate)`。这样 `WHERE UserID = ?`、`WHERE UserID = ? AND EventType = ?` 都能高效利用索引。

2. **使用 `LowCardinality` 类型**:

   * 对于基数较低的字符串列（如国家、城市、枚举类型的状态），使用 `LowCardinality(String)`。

   * **原理**: 它将字符串转换为字典编码（整数），极大减少了存储空间和读取成本，并加速了 `GROUP BY` 和 `FILTER`。

   * **示例**: `ALTER TABLE my_table MODIFY COLUMN country LowCardinality(String);`

3. **使用 <span style="color: rgb(216,57,49); background-color: inherit">projections </span>(投影)**:

   * 投影是 ClickHouse 21.8 版本后引入的强大功能。它相当于一个**由 ClickHouse 自动维护的、预聚合的“物化视图”**。

   * 当查询匹配投影的定义时，ClickHouse 会**自动**从更小的投影表中读取数据，而不是原始表。

![](<images/ClickHouse 高性能实时分析数据库-image-44.png>)

示例代码

```sql
-- 为原始表添加一个按 EventType 预聚合的投影
ALTER TABLE user_events
ADD PROJECTION p_event_type (
    SELECT
        EventType,
        count()
    GROUP BY EventType
);
```

之后，任何 `SELECT EventType, count() FROM user_events GROUP BY EventType` 的查询都会自动使用这个投影加速。



> **使用 `PREWHERE` 替代 `WHERE`**:

* `PREWHERE` 是 ClickHouse 的一个特色优化。它在数据被解压缩和读取到内存**之前**进行第一轮过滤。

* **原理**: 先在压缩数据上进行粗粒度过滤，只解压可能满足条件的列，大大减少了I/O和CPU消耗。

* **适用场景**: 当过滤条件作用于非主键列时，效果尤为明显。

* **示例**:

```sql
-- 普通查询
SELECT UserID, URL FROM hits WHERE CounterID = 123 AND URL LIKE '%/some/path%';

-- 优化后
SELECT UserID, URL FROM hits
PREWHERE CounterID = 123 -- 如果 CounterID 是主键一部分，放 WHERE
WHERE URL LIKE '%/some/path%'; -- 非主键的重度过滤条件放 PREWHERE
-- ClickHouse会自动优化，通常把过滤性强的条件放到 PREWHERE
```

*最佳实践：将选择性极高（能过滤掉大量数据）的非主键条件放入 `PREWHERE`。*



> **`JOIN` 操作的优化**:

* ClickHouse 是 OLAP 数据库，不擅长处理大表之间的 `JOIN`。应尽量避免。

* **如果必须 `JOIN`**：

  * **小表驱动大表**: 确保 `JOIN` 的右表是小表。

  * **使用 `GLOBAL IN` / `GLOBAL JOIN`**: 默认的 `JOIN` 会在每个节点上独立执行，可能导致数据倾斜。`GLOBAL` 会将右表（小表）广播到所有节点，在每个节点内存中形成一个哈希表，从而避免数据重分布。

  * **示例**:

```sql
-- 右表 user_info 很小
SELECT h.UserID, u.name FROM hits AS h
GLOBAL ANY LEFT JOIN user_info AS u ON h.UserID = u.UserID;
```

> **`GROUP BY` 优化**

* 避免对高基数列进行 `GROUP BY`，这会消耗巨大内存。

* 如果确实需要，考虑使用物化视图进行预聚合。

* 如果只需要总计和小计，使用 `GROUP BY ... WITH TOTALS`，而不是 `UNION ALL` 两个查询

#### 案例分析：一步步优化慢查询

**场景**: 一个仪表盘需要展示特定活动 (`campaign_id = 'camp_xyz'`) 下，不同浏览器 (`browser`) 的用户访问次数。数据量巨大。

```sql
CREATE TABLE visits (
    EventDate Date,
    UserID UInt64,
    campaign_id String,
    browser String,
    ... -- 其它列
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY EventDate; -- <<<<<< 性能陷阱！
```

原始查询方式

```sql
SELECT browser, count()
FROM visits
WHERE campaign_id = 'camp_xyz'
GROUP BY browser;
```

> **优化之旅**:

1. **分析 (`system.query_log`)**: 发现 `read_rows` 巨大，查询耗时 30 秒。`EXPLAIN` 显示 `Selected parts` 和 `Selected marks` 数量庞大。**结论：主键索引未生效**。

2. **第一步：优化主键（治本）**

   * 最常见的过滤条件是 `campaign_id`。我们应该将它加入主键。

   * **优化后表结构**:

```sql
-- 重建表或使用新表
CREATE TABLE visits_optimized (...)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY (campaign_id, browser, EventDate); -- <<<<<< 关键优化！
```

* **效果**: 再次查询，利用了 `campaign_id` 索引，`read_rows` 大幅减少，耗时降至 2 秒。

- **第二步：使用 `LowCardinality`（锦上添花）**

* `campaign_id` 和 `browser` 的基数相对数据总量来说很低。

* **再次优化表结构**:

```sql
ORDER BY (campaign_id, browser, EventDate)
-- 列定义
`campaign_id` LowCardinality(String),
`browser` LowCardinality(String),
```

* **效果**: 存储空间减少，`GROUP BY` 速度提升，耗时降至 800 毫秒。

- **第三步：创建投影（终极加速）**

* 这个聚合查询非常固定和高频。

* **为优化后的表添加投影**:

```sql
ALTER TABLE visits_optimized ADD PROJECTION p_campaign_browser (
    SELECT
        campaign_id,
        browser,
        count()
    GROUP BY campaign_id, browser
);
```

* **效果**: 再次执行原始查询，ClickHouse 自动从投影读取数据。耗时降至 **50 毫秒**。查询性能实现了质的飞跃。

***

> 总结

**调优清单**:

**Schema 设计阶段 (治本)**

* **`ORDER BY` 是否包含了最常用的过滤列？**

* 是否使用了 `LowCardinality` 压缩低基数列？

* 日期/时间列是否使用了 `Date/DateTime` 而不是 `String`？

* 数值类型是否选择了最小的可用类型（如 `UInt8` 代替 `UInt64`）？

* 对于固定模式的聚合查询，是否考虑了 `Projection` 或物化视图？

**查询编写阶段 (治标)**

* `SELECT` 子句是否只包含了必要的列？（告别 `SELECT *`）

* `WHERE` 条件是否有效利用了主键？

* 对于过滤性强的非主键条件，是否尝试了 `PREWHERE`？

* `JOIN` 操作是否真的是必须的？能否用 `IN` 或者预先打宽的表替代？

* 如果必须 `JOIN`，右表是否是小表，并使用了 `GLOBAL JOIN`？

* `GROUP BY` 的列基数是否过高？

通过遵循这套从原理到实践的调优方法论，您将能够系统地诊断和解决 ClickHouse 中的绝大多数性能问题，真正驾驭这头性能猛兽。

# 第四部分：生产环境运维与生态集成 (Production & Ecosystem)

## 第7章：集群管理与运维 (Cluster Operations & Management)

### 7.1. 集群搭建

#### 1 概述

在2025年，一个生产级的 ClickHouse 集群不再是手动配置的节点集合，而是一个由控制器自动管理的、高可用的分布式系统。

**核心组件**:

1. **ClickHouse Nodes**: 真正执行计算和存储数据的节点。

2. **ClickHouse Keeper**: 官方出品、内置的协调服务，完全替代 ZooKeeper。负责元数据管理、DDL 任务分发、副本选举等关键任务。**它是集群的大脑**。

3. **ClickHouse Operator**: 在 Kubernetes 环境中，这是管理整个集群生命周期的“机器人管理员”。它将运维人员的“期望状态”翻译成具体的 Kubernetes 操作。

4. **Distributed Table Engine**: 这是从用户视角与集群交互的统一入口，负责将查询智能地分发到各个分片。

![](<images/ClickHouse 高性能实时分析数据库-image-45.png>)

* **解读**：运维人员通过一个 YAML 文件（`chi.yaml`）向 Operator 声明期望的集群配置（如：2个分片，每个分片2个副本）。Operator 会自动创建并维护 Keeper 集群和所有 ClickHouse 节点，并确保它们之间的连接和配置正确。

#### 2 集群设计黄金法则：分片与副本

![](<images/ClickHouse 高性能实时分析数据库-diagram-7.png>)

**教学目标**：掌握如何为业务场景设计合理的分片（Sharding）和副本（Replication）策略。

* **目的**：解决**单一节点无法存储海量数据**或**计算能力不足**的问题。数据被水平切分到不同分片上。

* **设计原则**：选择一个合适的**分片键 (Sharding Key)**。查询时，如果 `WHERE` 条件包含分片键，查询将只被路由到特定分片，性能极高。

* **分片键的选择**：通常是业务中基数非常高的列，如 `user_id`, `device_id`。

**图解数据分布**:

![](<images/ClickHouse 高性能实时分析数据库-image-46.png>)

* ：确保在**单个节点故障时，数据不丢失，服务不中断**。同一分片内的数据在多个副本间完全一致。

* **工作原理**：写入操作只需成功写入一个副本即可向客户端返回成功。ClickHouse Keeper 会确保该写入日志被同步到其他副本。

**图解副本工作**:

![](<images/ClickHouse 高性能实时分析数据库-image-47.png>)

#### 3 集群部署实战

这一部分与之前的教案完全相同，因为概念是通用的。

* **分片 (Shard)**：数据**水平扩展**，提升查询性能和存储容量。

* **副本 (Replica)**：数据**高可用**的保障，防止单点故障。

* **ZooKeeper**：集群的**协调中心**，负责元数据管理、任务队列、故障检测,分布式协同工作等。

![](<images/ClickHouse 高性能实时分析数据库-image-48.png>)

取消注释  允许远程网络请求连接

```sql
<listen_host>0.0.0.0</listen_host>
```

##### 5.1 配置ZOOKEEPER

需要在每台CK的节点上配置ZK的位置

ClickHouse使用一组zookeeper标签定义相关配置，默认情况下，在全局配置config.xml中定义即可。但是各个副本所使用的Zookeeper 配置通常是相同的，为了便于在多个节点之间复制配置文件，更常见的做法是将这一部分配置抽离出来，独立使用一个文件保存。

首先，在confi.xml文件中配置

```xml
<zookeeper> 
 <node index="1"> 
 <host>doitedu01</host>
 <port>2181</port>
 </node>
  <node index="2"> 
 <host>doitedu02</host>
 <port>2181</port>
 </node>
  <node index="3"> 
 <host>doitedu02</host>
 <port>2181</port>
 </node>
 </zookeeper>
```

将配置文件同步到其他集群节点!!

```bash
scp config.xml   linux02:$PWD
scp config.xml   linux03:$PWD 
重启服务 
systemctl  restart  clickhouse-server
systemctl  status clickhouse-server
```

##### 5.2 **创建副本表**

在创建副本表以前, 首先要启动集群中的zookeeper

首先，由于增加了数据的冗余存储，所以降低了数据丢失的风险；其次，由于副本采用了多主

架构，所以每个副本实例都可以作为数据读、写的入口，这无疑分摊了节点的负载。

在使用单使用副本功能的时候 , 我们对CK集群不需要任何的配置就可以实现数据的多副本存储!只需要在建表的时候指定engine和ZK的位置即可 ;

```bash
ENGINE = ReplicatedMergeTree('zk_path', 'replica_name')
 -- /clickhouse/tables/{shard}/table_name
 -- /clickhouse/tables/ 是约定俗成的路径固定前缀，表示存放数据表的根路径。 
```

·{shard}表示分片编号，通常用数值替代，例如01、02、03。一张数据表可以有多个分片，而每个分片都拥有自己的副本。

·table\_name表示数据表的名称，为了方便维护，通常与物理表的名字相同（虽然ClickHouse并不强制要求路径中的表名称和物理表名相同）；而replica\_name的作用是定义在ZooKeeper中创建的副本名称，该名称是区分不同副本实例的唯一标识。一种约定成俗的命名方式是使用所在服务器的域名称。

对于zk\_path而言，同一张数据表的同一个分片的不同副本，应该定义相同的路径；而对于replica\_name而言，同一张数据表的同一个分片的不同副本，应该定义不同的名称

![](<images/ClickHouse 高性能实时分析数据库-diagram-8.png>)

**一个分片 , 多个副本表**

```sql
-- lixnu01 æºå¨
create table tb_demo1 (
    id Int8 ,
    name String)engine=ReplicatedMergeTree('/clickhouse/tables/01/tb_demo1', 'doitedu01') 
order by id ;
-- lixnu02 æºå¨
create table tb_demo1 (
    id Int8 ,
    name String)engine=ReplicatedMergeTree('/clickhouse/tables/01/tb_demo1', 'doitedu02') 
order by id ;
-- lixnu03 æºå¨
create table tb_demo1 (
    id Int8 ,
    name String)engine=ReplicatedMergeTree('/clickhouse/tables/01/tb_demo1', 'doitedu03') 
order by id ;
```

查看zookeeper中的内容

```bash
[zk: localhost:2181(CONNECTED) 0] ls /
[a, zookeeper, clickhouse, DNS, datanode1, server1, hbase]
[zk: localhost:2181(CONNECTED) 1] ls /clickhouse
[tables, task_queue]
[zk: localhost:2181(CONNECTED) 2] ls /clickhouse/tables
[01]
[zk: localhost:2181(CONNECTED) 3] ls /clickhouse/tables/01
[tb_demo1]
[zk: localhost:2181(CONNECTED) 4] ls /clickhouse/tables/01/tb_demo1
[metadata, temp, mutations, log, leader_election, columns, blocks, nonincrement_block_numbers, replicas, quorum, block_numbers]
[zk: localhost:2181(CONNECTED) 5] ls /clickhouse/tables/01/tb_demo1/replicas
[linux02, linux03, linux01]
```

```sql
SELECT *
FROM system.zookeeper
WHERE path = '/' ;

[zk: localhost:2181(CONNECTED) 7] ls   /clickhouse/tables/01/tb_demo1/replicas
[doitedu01, doitedu02, doitedu03]
```

或者

```sql
-- lixnu01 æºå¨
create table tb_demo2 (
    id Int8 ,
    name String)engine=ReplicatedMergeTree('/clickhouse/tables/01/tb_demo2', 'linux01') 
order by id ;
-- lixnu02 æºå¨
create table tb_demo2 (
    id Int8 ,
    name String)engine=ReplicatedMergeTree('/clickhouse/tables/01/tb_demo2', 'linux02') 
order by id ;
-- lixnu03 æºå¨
create table tb_demo2 (
    id Int8 ,
    name String)engine=ReplicatedMergeTree('/clickhouse/tables/01/tb_demo2', 'linux03') 
order by id ;

-------------------

-- lixnu01 æºå¨
create table tb_demo2 (
    id Int8 ,
    name String)engine=ReplicatedMergeTree('/clickhouse/tables/01/tb_demo2', 'doitedu01') 
order by id ;
-- lixnu02 æºå¨
create table tb_demo2 (
    id Int8 ,
    name String)engine=ReplicatedMergeTree('/clickhouse/tables/01/tb_demo2', 'doitedu02') 
order by id ;
-- lixnu03 æºå¨
create table tb_demo2 (
    id Int8 ,
    name String)engine=ReplicatedMergeTree('/clickhouse/tables/02/tb_demo2', 'doitedu03') 
order by id ;

[zk: localhost:2181(CONNECTED) 14] ls   /clickhouse/tables/02/tb_demo2/replicas
[doitedu03]
[zk: localhost:2181(CONNECTED) 15] ls   /clickhouse/tables/01/tb_demo2/replicas
[doitedu01, doitedu02]
------------------------------
-- lixnu04 æºå¨
create table tb_demo2 (
    id Int8 ,
    name String)engine=ReplicatedMergeTree('/clickhouse/tables/02/tb_demo2', 'linux04') 
order by id ;
```

##### 5.3 分布式引擎

Distributed表引擎是分布式表的代名词，它自身不存储任何数据，而是作为数据分片的透明代理，能够自动路由数据至集群中的各个节点，所以Distributed表引擎需要和其他数据表引擎一起协同工作，

一般使用分布式表的目的有两种,

* 一种是表存储多个副本并且有大量的并发操作,我们可以使用分布式表来分摊请求压力解决并发问题

* 一种是表特别大有多个切片组成 ,并且每切片数据也可以存储数据副本

* 本地表：通常以\_local为后缀进行命名。本地表是承接数据的载体，可以使用非Distributed的任意表引擎，一张本地表对应了一个数据分片

* 分布式表：通常以\_all为后缀进行命名。分布式表只能使用Distributed表引擎，它与本地表形成一对多的映射关系，日后将通过分布式表代理操作多张本地表。

```sql
ENGINE = Distributed(cluster, database, table [,sharding_key]) 
```

* cluster：集群名称，与集群配置中的自定义名称相对应。在对分布式表执行写入和查询的过程中，它会使用集群的配置信息来找到相应的host节点。

* database和table：分别对应数据库和表的名称，分布式表使用这组配置映射到本地表。

* sharding\_key：分片键，选填参数。在数据写入的过程中，分布式表会依据分片键的规则，将数据分布到各个host节点的本地表。

##### **4 没有副本**

本示例是,使用某个集群 , 创建多分片无副本的表配置了一个集群 cluster1 集群中有三台机器ck1 ck2 ck3,没有副本,如果在这个集群上建表, 表数据会有三个切片 ,没有存储数据副本

在配置文件中配置集群详细

```xml
<clickhouse_remote_servers>
<cluster1>
<!-- éç¾¤åä¸ºcluster1 æ´ä¸ªéç¾¤ä¸­æ¯ä¸ªè¡¨æä¸ä¸ªåç,åå«å¨lx01 lx02 lx03ä¸ -->
 <shard>
 <replica>
 <host>linux01</host>
 <port>9000</port>
 </replica>
 </shard>
 <shard>
 <replica>
 <host>linux02</host>
 <port>9000</port>
 </replica>
 </shard>
 <shard>
 <replica>
 <host>linux03</host>
 <port>9000</port>
 </replica>
 </shard>
</cluster1>
 <cluster2>
<!-- éç¾¤åä¸ºcluster2 ä¸ä¸ªåç ä¸ä¸ªå¯æ¬ -->
 <shard>  
 <replica>
 <host>linux01</host>
 <port>9000</port>
 </replica>
 <replica>
 <host>linux02</host>
 <port>9000</port>
 </replica>
 <replica>
 <host>linux03</host>
 <port>9000</port>
 </replica>
 </shard>
</cluster2>
    <!--éç¾¤ä¸  å¤ä¸ªåç  ä¿çå¯æ¬ æ³¨æä¸ä¸ªä¸»æºåªä½¿ç¨ä¸æ¬¡ -->
<cluster3>
 <shard>  
 <replica>
 <host>doit01</host>
 <port>9000</port>
 </replica>
 <replica>
 <host>doit02</host>
 <port>9000</port>
 </replica>
 </shard>
  <shard>  
 <replica>
 <host>doit03</host>
 <port>9000</port>
 </replica>
 <replica>
 <host>doit04</host>
 <port>9000</port>
 </replica>
 </shard>
</cluster3>
    
</clickhouse_remote_servers>
```

![](<images/ClickHouse 高性能实时分析数据库-image-49.png>)

```sql
同步配置文件 到集群中
-- 创建本地表 
create table tb_demo3 on cluster cluster1(
id  Int8 ,name String 
)engine=MergeTree()
 order by  id ;
 -- 创建分布式表 
 create table demo3_all on cluster cluster1 
 engine=Distributed('cluster1','default','tb_demo3',id) as tb_demo3 ;
 --向分布式表中插入数据 ,数据会根据插入规则将数据插入到不同的分片中
```

有副本的配置

```xml
<!-- éç½®éç¾¤2 , éç¾¤ä¸­çè¡¨æä¸¤ä¸ªåç ,å¶ä¸­åç1 æä¸¤ä¸ªå¯æ¬ -->
<cluster2>
 <shard>
        <replica>
                <host>linux01</host>
                <port>9000</port>
        </replica>
    <replica>
                <host>linux02</host>
                <port>9000</port>
        </replica>
 </shard>
 <shard>
        <replica>
                <host>linux03</host>
                <port>9000</port>
        </replica>
 </shard>
</cluster2>
```



```python
-- 创建本地表
create table tb_demo4 on cluster cluster2(
id  Int8 ,
name String )
engine=MergeTree() order by  id ;
-- 创建分布式表
create table demo4_all on cluster cluster2
engine=Distributed('cluster2','default','tb_demo4',id) as tb_demo4 ;
```

##### 5 分布式DDL



ClickHouse支持集群模式，一个集群拥有1到多个节点。CREATE、ALTER、DROP、RENMAE及TRUNCATE这些DDL语句，都支持分布式执行。这意味着，如果在集群中任意一个节点上执行DDL语句，那么集群中的 每个节点都会以相同的顺序执行相同的语句。这项特性意义非凡，它就如同批处理命令一样，省去了需要依次去单个节点执行DDL的烦恼。将一条普通的DDL语句转换成分布式执行十分简单，只需加上ON CLUSTER cluster\_name声明即可。例如，执行下面的语句后将会对 ch\_cluster集群内的所有节点广播这条DDL语句：

```sql
-- 建表 on cluster cluster1
create table tb_demo3 on cluster cluster1(
id  Int8 ,name String )engine=MergeTree() order by  id ;
-- 删除集群中所有的本地表或者是分布式表
drop table if exists tb_demo3 on cluster cluster1;
-- 修改集群中的表结构 
alter table t3 on cluster cluster1 add column age Int8 ;-
- 删除字段 -- 删除分区 
```

#### &#xA;4 分布式协同机制 (ZooKeeper)

**【图示：从单机到集群的演进】**

* **图示说明：** 此图展示了从一个不堪重负的单节点，演进为一个可扩展、高可用的分布式集群的理念。

![](<images/ClickHouse 高性能实时分析数据库-image-50.png>)

* **1. 分片 (Shard) 和 副本 (Replica)**

  * **分片 (Shard)：** **为了扩展性**。它将数据进行水平切分。每个分片只存储数据全集的一部分。

  * **副本 (Replica)：** **为了高可用**。它是分片内数据的完整拷贝。一个分片可以有多个副本，它们的数据是完全一致的。

![](<images/ClickHouse 高性能实时分析数据库-image-51.png>)

* **2. ZooKeeper：集群的“大脑”与“协调者”**

  * **核心职责：**

    * **集群拓扑管理：** 存储集群的配置信息。

    * **任务队列：** `ON CLUSTER` DDL 操作通过 ZK 队列分发。

    * **主副本选举 (Leader Election)：** 在副本中选举一个 Leader 负责一些管理任务。

    * **数据同步日志：** 记录副本间的数据同步任务，是 Replicated 表引擎的核心依赖。

**3. 两种关键的表引擎**

* `ReplicatedMergeTree` **系列：**

  * 实现**副本**机制的基石，**必须**依赖 ZooKeeper。

  * **建表示例：**

```sql
-- 在每个节点上执行
CREATE TABLE my_table_local ON CLUSTER my_cluster (
    event_date Date, id UInt64
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/my_table', '{replica}')
PARTITION BY toYYYYMM(event_date) ORDER BY id;
```

* `/clickhouse/tables/{shard}/my_table`：ZK 路径，同一分片的所有副本**必须相同**。

* `{replica}`：副本唯一标识，同一分片内**必须不同**。

* `Distributed` **表引擎：**

  * 实现**分片**机制的逻辑表，本身**不存储数据**，像一个**路由器**。

  * **建表示例：**

```sql
-- 只需在任意一个节点执行一次
CREATE TABLE my_table_distributed ON CLUSTER my_cluster AS my_table_local
ENGINE = Distributed(
    my_cluster,      -- 集群名
    default,         -- 目标库
    my_table_local,  -- 目标本地表
    rand()           -- 分片键 (Sharding Key)
);
```

> #### **分布式写入流程图 (写时复制)**

* **图示说明：** 该序列图详细展示了数据写入的每一步。核心思想是：数据先由 `Distributed` 表路由到目标分片的**一个副本**，该副本写入成功后，在 ZooKeeper 中记录日志，同分片的其他副本监听到日志后，主动从第一个副本拉取数据，完成同步。

![](<images/ClickHouse 高性能实时分析数据库-diagram-9.png>)

* 写入数据

![](<images/ClickHouse 高性能实时分析数据库-image-52.png>)

* &#x20;“查询正好是反向过程：从一个入口进来，分散到所有分片，最后将结果聚合起来返回。”

* **【图示：分布式查询流程图 (读时聚合)】**

  * **图示说明：** 该序列图展示了查询的分发和聚合过程。`Distributed` 表将查询分发到**每个分片的一个健康副本**上，各个分片并行计算，将部分结果返回给协调节点，协调节点进行最终的合并和计算，返回给客户端。

![](<images/ClickHouse 高性能实时分析数据库-image-53.png>)

* “ZooKeeper 是幕后英雄，我们再深入看看它在两个关键场景下的作用。”

* **1. `ON CLUSTER` DDL 的执行原理**

  * **核心机制：** 利用 ZooKeeper 的共享队列。

* **【图示：ON CLUSTER DDL 执行流程】**

  * **图示说明：** 当你在一个节点上执行 `ON CLUSTER` DDL 时，该DDL语句会被提交到 ZooKeeper 的一个公共任务队列中。集群中的每个节点都在监听这个队列，一旦发现新任务，就会各自独立地去执行它，从而保证了所有节点上的表结构一致。

![](<images/ClickHouse 高性能实时分析数据库-image-54.png>)

* **故障转移（Failover）机制**

  * **写故障：** `Distributed` 表引擎在转发数据时，会自动跳过宕机的副本，选择同一分片内的其他健康副本。

  * **读故障：** 协调节点在分发查询时，同样会自动跳过宕机副本，选择分片内其他健康的副本。

由于每个分片都有副本，并且 ZooKeeper 维护着所有节点的健康状态，所以无论是读还是写，ClickHouse 都能优雅地处理单点故障，实现高可用。



#### 5 故障恢复与数据一致性

##### &#xA;5.1 . **监控与告警**

我们已经搭建了强大的集群，但如何知道它是否健康？是否在高效工作？就像开跑车需要看仪表盘一样，管理 ClickHouse 集群也需要监控。ClickHouse 非常慷慨，它通过内置的 `system` 数据库，向我们暴露了海量的内部状态信息。我们重点关注三个表。

**1. `system.metrics` - 瞬时指标**

* **用途：** 提供**当前瞬间**的指标值。它是一个快照。

* **特点：** 包含了约 200+ 个指标，值会实时变化。

* **关键指标举例：**

  * `Query`: 当前正在执行的查询数。

  * `Merge`: 当前正在进行的后台合并任务数。

  * `ReplicasMaxAbsoluteDelay`: 所有副本中，最大的绝对延迟时间（秒）。**这是衡量副本同步健康度的核心指标！**

```sql
SELECT
 metric, 
 value 
 FROM system.metrics 
 WHERE metric IN ('Query', 'Merge', 'ReplicasMaxAbsoluteDelay');
```

**2. `system.events` - 累计事件**

* **用途：** 提供自 ClickHouse 服务启动以来的**累计事件发生次数**。

* **特点：** 值是只增不减的计数器。要看“速率”，需要用当前值减去上一个时间点的值。

* **关键指标举例：**

  * `SelectQuery`: 已成功执行的 `SELECT` 查询总数。

  * `InsertQuery`: 已成功执行的 `INSERT` 查询总数。

  * `FailedQuery`: 执行失败的查询总数。

```sql
-- 查看失败查询的总数
SELECT event, value FROM system.events WHERE event = 'FailedQuery';
```

\* `system` 数据库：`system.metrics`, `system.events`, `system.merges`
\* **【实践】**: 使用 Prometheus + **Grafana** 监控 ClickHouse 集群关键指标。

**. `system.merges` - 合并任务详情**

* **用途：** 显示当前正在进行的 `MergeTree` 表的合并任务的详细信息。

* **特点：** 可以看到哪个库、哪个表的哪个分区正在合并，以及进度和耗时。对于排查 I/O 问题和写入性能瓶颈至关重要。

* **关键字段：** `database`, `table`, `elapsed` (已耗时), `progress` (进度), `is_mutation` (是否为 ALTER 操作)。

##### <span style="color: rgb(216,57,49); background-color: inherit">5.2 Prometheus + Grafana</span>

我们介绍目前最主流的 ClickHouse 监控方案：Prometheus + Grafana。这个组合拳威力巨大，我们先来理解它们是如何协同工作的。

**架构与数据流**

* **【图示：ClickHouse 监控架构图】**

  * **图示说明：** 这张图清晰地展示了从 ClickHouse 到用户告警/看板的完整数据链路。`clickhouse-exporter` 是关键的“翻译官”，它将 ClickHouse 的内部 `system` 指标翻译成 Prometheus 能看懂的格式。

![](<images/ClickHouse 高性能实时分析数据库-image-55.png>)

* **核心组件解析：**

  1. **clickhouse-exporter:** 一个独立的程序，部署在每个 ClickHouse 节点旁边。它定时查询 `system.*` 表，并将结果通过一个 HTTP 接口（通常是 `/metrics`）暴露出来。

  2. **Prometheus:** 时序数据库。它会周期性地“拉取”（Scrape）所有 exporter 暴露的指标数据，并存储起来。同时，它内置了强大的查询语言 PromQL 和告警规则引擎。

  3. **Grafana:** 数据可视化工具。它作为 Prometheus 的前端，通过 PromQL 查询数据，并以图表、仪表盘的形式展示给用户。

  4. **Alertmanager:** Prometheus 的告警处理组件，负责对告警进行去重、分组、路由，并发送到钉钉、Slack、邮件等。

**必须关注的关键指标（在 Grafana 中如何看）**

导入一个优秀的 Grafana Dashboard 模板后，你会看到琳琅满目的图表。但我们必须抓住重点，以下是关乎集群生死的几个核心指标。

![](<images/ClickHouse 高性能实时分析数据库-image-56.png>)



* **指标解读：**

  * **集群健康与可用性**

    * `clickhouse_metric_ReplicasMaxAbsoluteDelay`: **（告警必配）** 最大副本绝对延迟。如果这个值持续很高（如 > 300秒），说明副本同步出了严重问题，可能导致数据不一致或丢失。

    * `clickhouse_up`: Prometheus 检查 exporter 是否存活的指标。如果为 0，说明节点或 exporter 可能宕机。

  * **查询性能与负载**

    * `rate(clickhouse_event_SelectQuery[5m])`: 5分钟内的平均每秒查询数（QPS）。反映集群负载。

    * `rate(clickhouse_event_FailedQuery[5m])`: **（告警必配）** 5分钟内的平均每秒失败查询数。突然升高通常意味着应用端有 bug 或者有恶意查询。

    * `histogram_quantile(0.95, sum(rate(clickhouse_query_duration_ms_bucket[5m])) by (le))`: 查询耗时的 P95 分位数。反映大多数用户的查询体验。如果变高，说明整体查询性能在下降。

  * **资源与后台任务**

    * `clickhouse_metric_Merge`: 正在进行的 Merge 任务数。如果这个值持续很高，且 `parts_to_merge` 也很多，说明 I/O 压力大，写入可能即将成为瓶颈。

    * **CPU/内存使用率：** 通用指标，过高则需要扩容或优化查询。

##### &#xA;5.3 . **备份与恢复**

###### 1. `clickhouse-backup` 是什么？为什么选择它？

`clickhouse-backup` 是一个由社区（非 ClickHouse 官方）开发的、功能强大且广受欢迎的 ClickHouse 备份和恢复工具。

**为什么不用 `ALTER TABLE ... FREEZE`？**
ClickHouse 内置的 `FREEZE` 命令可以将数据分区的快照“冻结”到磁盘的 `shadow` 目录下，但这只是备份的第一步。你还需要手动去拷贝这些文件、管理备份历史、恢复时也需要手动移动文件并执行 `ATTACH`。这个过程非常繁琐且容易出错。

**`clickhouse-backup` 的核心优势：**

* **全功能:** 支持完整的备份和恢复流程。

* **增量备份:** 支持基于 `FREEZE` 的增量备份，极大节省了存储空间和备份时间。

* **云存储支持:** 原生支持将备份上传到 S3, GCS, Azure Blob, FTP, SFTP 等远程存储，实现异地容灾。

* **并发与性能:** 利用多线程进行数据拷贝，速度快。

* **灵活的表过滤:** 可以指定备份哪些表，或者排除哪些表。

* **支持 `ON CLUSTER`:** 可以在集群模式下协调所有节点的备份与恢复。

* **易于自动化:** 命令行接口清晰，非常适合集成到 Cronjob 或其他调度系统中。

###### 2. 安装 `clickhouse-backup`

我们介绍两种最常用的安装方式：下载二进制文件（推荐）和使用 Docker。

这是最简单直接的方式，适合大多数 Linux 环境。

1. **访问 GitHub Releases 页面:**
   打开 `clickhouse-backup` 的官方 [GitHub Releases 页面](https://www.google.com/url?sa=E\&q=https%3A%2F%2Fgithub.com%2FAlexAkulov%2Fclickhouse-backup%2Freleases)。

2. **选择并下载合适的版本:**
   找到最新的版本，根据你的系统架构选择对应的压缩包。对于大多数现代 Linux 服务器（如 CentOS 7, Ubuntu 20.04），选择 `clickhouse-backup-linux-amd64.tar.gz`。

```sql
    sudo rpm -ivh /path/to/package.rpm

[root@linux01 ~]# clickhouse-backup   --version
Version:         2.6.26
Git Commit:      07fcc818c8e53dcbad75f00fcbdf6cb11794af39
Build Date:      2025-07-08
```



修改配置文件

使用 `sudo vi /etc/clickhouse-backup/config.yml` 编辑。以下是针对 CentOS 7 环境的重点配置说明：

```sql

general:
  # 在本地保留多少个备份，0 表示全部保留。旧的会自动删除。
  backups_to_keep_local: 3
  # 推荐使用 tar 格式，性能更好，生成单个文件。
  backup_format: "tar" 

clickhouse:
  # ClickHouse 连接信息
  username: default
  password: "" # 如果你的 ClickHouse 设置了密码，请填写
  host: 127.0.0.1
  port: 9000
  # (!!!) CentOS 7 下通过 RPM 安装的 ClickHouse 默认路径如下
  # 如果你修改过数据存储位置，必须更新这里！
  data_path: "/var/lib/clickhouse/data"
  metadata_path: "/var/lib/clickhouse/metadata"
  timeout: 5m
  
# ... 远程存储配置 (S3, GCS, FTP 等) ...
# 按需填写，如果只做本地备份，可以忽略这部分。
# s3:
#   access_key: "YOUR_ACCESS_KEY"
#   secret_key: "YOUR_SECRET_KEY"
#   bucket: "your-clickhouse-backup-bucket"
#   endpoint: "s3.amazonaws.com"
#   ...
```

**CentOS 7 注意事项：**

* **权限问题：** `clickhouse-backup` 需要读取 ClickHouse 的数据目录。默认情况下，`/var/lib/clickhouse` 目录的属主是 `clickhouse:clickhouse`。如果你使用 `root` 用户（通过 `sudo`）来执行 `clickhouse-backup`，通常不会有权限问题。如果你使用其他用户，需要确保该用户有权限访问这些目录。

* **SELinux：** CentOS 7 默认开启 SELinux。在大多数情况下，`clickhouse-backup` 读取标准路径下的文件不会触发 SELinux 策略问题。但如果你将备份目录设置在非标准位置（如 `/srv/backups`），或者遇到莫名的 "Permission denied" 错误，可以尝试临时禁用 SELinux (`sudo setenforce 0`) 来排查是否是 SELinux 导致的问题。如果是，需要配置相应的 SELinux 上下文策略。

###### &#x20;3 核心操作：备份与恢复 (CentOS 7)

clickhouse-backup tables

这是执行任何操作前的“健康检查”，确保工具能正确连接并识别数据库

```sql
JOIN (SELECT policy_name, arrayJoin(disks) AS disk FROM system.storage_policies) AS s ON s.disk = d.name GROUP BY d.path
default.website_hits        1.95KiB  default  full
learning.t_web_hits         1.57KiB  default  full
learning.raw_logs           935B     default  full
learning.detail_table       840B     default  full
learning.agg_table          750B     default  full
default.account_store       718B     default  full
default.stripe_log_table    698B     default  full
learning.daily_summary      596B     default  full
learning.user_sessions      579B     default  full
learning.app_configs        578B     default  full
default.daily_stats         522B     default  full
default.user_profiles       474B     default  full
default.local_orders        465B     default  full
learning.test_table2        430B     default  full
learning.test_table         338B     default  full
default.hits                331B     default  full
learning.test_table3        310B     default  full
default.tb_demo3            308B     default  full
default.tb_demo2            304B     default  full
default.tb_demo1            302B     default  full
x.tiny_log_table            209B     default  full
default.tiny_log_table      209B     default  full
atomic_db.t1                43B      default  full
atomic_db.t2                43B      default  full
default.mysql_users         0B                full
default.test_hdfs1          0B                full
default.test_parquet        0B                full
default.test_parquet2       0B                full
default.json                0B                full
default.user_actions_pipe   0B                full
default.user_actions_pump   0B       default  full
default.user_actions_store  0B       default  full
default.user_behavior       0B       default  full
default.hive_users2         0B                full
default.hive_users          0B                full
default.demo3_all           0B       default  full
default.account             0B                full
default.acc_2_store         0B       default  full
default.a                   0B                full
learning.mv_daily_summary   0B       default  full
default.sales               0B                full
x.hive_users                0B                full
```

> 创建备份

```sql
sudo clickhouse-backup create
```

备份文件将存储在 `config.yml` 中定义的默认路径下，通常是 `/var/lib/clickhouse/backup`。

> **创建备份并指定名称:**

```sql
sudo clickhouse-backup create my_first_centos_backup
```

> **只备份特定的表:**

```sql
# 只备份 logs_2023 和 metrics 表
sudo clickhouse-backup create -t default.logs_2023,default.metrics specific_tables_backup
```

> **创建并上传到远程存储:**
> （前提是你已在 `config.yml` 中正确配置了 S3、FTP 等远程存储）

```sql
sudo clickhouse-backup create --upload my_remote_backup
```

> **查看本地备份:**

```sql
sudo clickhouse-backup list local
查看远端备份:
sudo clickhouse-backup list remote
```

> **警告：恢复操作会覆盖现有数据，请在测试环境中充分演练！**

```sql
# 恢复指定名称的备份
sudo clickhouse-backup restore my_first_centos_backup

# 1. 从远程存储下载备份文件到本地
sudo clickhouse-backup download my_remote_backup

# 2. 从本地恢复
sudo clickhouse-backup restore my_remote_backup
```

> **恢复单个表或仅恢复结构/数据:**

这部分操作与通用教程一致，使用 `-t`, `--schema`, `--data` 参数即可。

```sql
# 只恢复 my_table 的表结构
sudo clickhouse-backup restore --schema -t default.my_table my_first_centos_backup
```

###### 4 自动备份

编写脚本&#x20;

脚本模板如下

```bash
#!/bin/bash

# 获取当前日期，用于日志文件名
LOG_DATE=$(date +%Y-%m-%d)
LOG_FILE="/var/log/clickhouse-backup/backup-${LOG_DATE}.log"

# 创建日志目录
mkdir -p /var/log/clickhouse-backup

# 执行备份命令，并将标准输出和错误输出都重定向到日志文件
# -u 表示 --upload，如果配置了远程存储，它会执行上传
/usr/local/bin/clickhouse-backup create -u >> "${LOG_FILE}" 2>&1

# (可选) 删除旧的本地备份，以防配置中的 backups_to_keep_local 失效
# /usr/local/bin/clickhouse-backup delete local --keep 3 >> "${LOG_FILE}" 2>&1

# (可选) 删除7天前的日志文件
find /var/log/clickhouse-backup -type f -mtime +7 -delete
```

将备份脚本配置在系统定时器中  , 或这其他任务调度工具

crontab  -e

```sql
分  时 日 月 周
30 2 * * * /usr/local/sbin/backup_clickhouse.sh
```

###### 问题排查

1. CentOS 7 常见问题与排错

* **Q: 遇到 `Permission denied` 错误**

  * **A:** 首先检查文件系统权限。使用 `sudo namei -om /var/lib/clickhouse/data` 查看路径上每个目录的权限。

  * **A:** 如果权限没问题，很可能是 **SELinux** 在作祟。执行 `getenforce` 查看 SELinux 状态。如果是 `Enforcing`，尝试临时设置为 `Permissive` 模式：`sudo setenforce 0`。然后再次运行备份命令。如果成功了，说明就是 SELinux 的问题。你需要为 `clickhouse-backup` 配置正确的 SELinux 策略，或者将 SELinux 永久设置为 `Permissive` 模式（不推荐在生产环境这样做）：编辑 `/etc/selinux/config`，将 `SELINUX=enforcing` 改为 `SELINUX=permissive`，然后重启系统。

* **Q: `clickhouse-backup create` 卡住或超时**

  * **A:** 检查 ClickHouse 服务是否正常运行: `sudo systemctl status clickhouse-server`。

  * **A:** 检查 `config.yml` 中的 `host`, `port`, `username`, `password` 是否正确。

  * **A:** 检查 CentOS 7 的防火墙 (`firewalld`) 是否阻止了到 `9000` 端口的连接（如果 `clickhouse-backup` 和 `clickhouse-server` 不在同一台机器上）。

* **Q: Cronjob 任务不执行**

  * **A:** 检查 `crond` 服务是否运行。

  * **A:** 检查 `/var/log/cron` 日志，看任务是否被调度，是否有错误信息。

  * **A:** 确保脚本 (`/usr/local/sbin/backup_clickhouse.sh`) 有执行权限，并且其 shebang (`#!/bin/bash`) 是正确的。

  * **A:** 确保脚本中所有命令都使用了**绝对路径**（如 `/usr/local/bin/clickhouse-backup`），因为 `cron` 的 `PATH` 环境变量非常有限。

###### &#xA;√ 实战案例

**场景假设**

* 我们有一个数据库 `analytics`，其中有一张重要的日志表 `analytics.logs`。

* 我们的备份策略是：**每周日进行一次全量备份，之后每天进行一次增量备份。**

* 我们将模拟周日、周一、周二三天的备份与恢复过程。

> #### 准备工作

1. **创建示例表和数据**
   登录到 ClickHouse 客户端 (`clickhouse-client`)，执行以下操作：

```sql
-- 创建数据库
CREATE DATABASE IF NOT EXISTS analytics;

-- 创建一张 MergeTree 表
CREATE TABLE analytics.logs (
    event_date Date,
    event_time DateTime,
    user_id UInt64,
    message String
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY (event_date, user_id);

-- 模拟周日的数据
INSERT INTO analytics.logs VALUES 
('2023-10-22', '2023-10-22 10:00:00', 101, 'User logged in'),
('2023-10-22', '2023-10-22 11:30:00', 102, 'User viewed dashboard');
```

> 第一步：周日 - 执行全量备份 (Full Backup)

全量备份是所有后续增量备份的**基础**。它包含了指定表在执行备份那一刻的全部数据。

```sql
# 执行 create 命令
sudo clickhouse-backup create full_backup_20231022
```

**命令解读:**

* `create`: 创建备份的动作。

* `full_backup_20231022`: 我们给这个备份指定的唯一名称。

2023/10/22 14:05:10 INFO     Create backup 'full\_backup\_20231022'

2023/10/22 14:05:10 INFO     Executing `ALTER TABLE "analytics"."logs" FREEZE`

...

2023/10/22 14:05:12 INFO     Backup 'full\_backup\_20231022' created successfully

**验证一下**

```sql
sudo clickhouse-backup list local

full_backup_20231022
```

> **第二步：周一 - 执行第一次增量备份 (Incremental Backup)**

增量备份只会备份**相对于上一次备份（全量或增量）以来新产生的数据分区**。

```sql
-- 模拟周一的新增数据
INSERT INTO analytics.logs VALUES 
('2023-10-23', '2023-10-23 09:00:00', 103, 'New user registered'),
('2023-10-23', '2023-10-23 12:00:00', 101, 'User updated profile');
关键点: 这些新数据 (2023-10-23) 属于新的数据分区。
```

**<span style="color: rgb(216,57,49); background-color: inherit">执行增量备份命令</span>**
这里是核心！我们需要使用 `--diff-from` 参数来告诉 `clickhouse-backup`，这次备份是基于哪个备份的增量。

```sql
# 创建名为 inc_backup_20231023 的增量备份，其基础是 full_backup_20231022
sudo clickhouse-backup create --diff-from full_backup_20231022 inc_backup_20231023
```

**命令解读:**

* `--diff-from full_backup_20231022`: 这个参数明确指出，请只备份比 `full_backup_20231022` 多出来的部分。

* `inc_backup_20231023`: 我们为这次增量备份起的名字。

> ### 第三步：周二 - 执行第二次增量备份

```sql
-- 模拟周二的新增数据
INSERT INTO analytics.logs VALUES 
('2023-10-24', '2023-10-24 15:00:00', 104, 'API call successful'),
('2023-10-24', '2023-10-24 16:00:00', 102, 'User placed an order');
```

这次，我们的基础是**上一次的增量备份** `inc_backup_20231023`。

```sql
# 创建名为 inc_backup_20231024 的增量备份，其基础是 inc_backup_20231023
sudo clickhouse-backup create --diff-from inc_backup_20231023 inc_backup_20231024
```

**重要说明:** 你也可以选择总是基于最初的全量备份来做增量 (`--diff-from full_backup_20231022`)，这被称为“差异备份”。但通常链接到上一次备份（“增量备份”）更节省空间。

> ### 第四步：灾难恢复 - 模拟数据丢失后的恢复过程

```sql
DROP TABLE analytics.logs;
```

**恢复过程 - 必须按顺序！**

**步骤 4.1: 恢复全量备份**
这会先恢复表结构和周日的数据。

```sql
# 恢复表结构和基础数据
sudo clickhouse-backup restore full_backup_20231022
```

**步骤 4.2: 恢复第一次增量备份**

```sql
# 恢复周一的增量数据
sudo clickhouse-backup restore inc_backup_20231023
```

**步骤 4.3: 恢复第二次增量备份**

```sql
# 恢复周二的增量数据
sudo clickhouse-backup restore inc_backup_20231024
```

**最终验证**
所有恢复步骤完成后，再次登录 `clickhouse-client` 进行查询：

总结与关键点

* **全量备份是根基：** 任何增量备份策略都必须以一个全量备份开始。

* **增量备份依赖 `--diff-from`：** 这个参数是实现增量备份的核心，它指定了差异比较的基准。

* **恢复必须按顺序：** 必须严格按照 `全量 -> 增量1 -> 增量2 ...` 的顺序进行恢复，跳过任何一个环节都会导致数据不完整。

* **命名规范很重要：** 给备份起一个包含日期和类型（full/inc）的名字，能让你在恢复时清晰地知道操作顺序，避免混淆。

* **自动化脚本：** 在生产环境中，你会写一个脚本来自动判断今天是周日（执行全量）还是工作日（执行增量），并动态生成备份名称和 `--diff-from` 参数。

### 7.2. **升级与维护**

#### &#xA;版本平滑升级策略

**平滑升级 (Rolling Upgrade)** 的核心思想是：**逐个节点地、有控制地进行升级，而不是同时停止整个集群。** 在任何一个时间点，集群中总有足够多的健康节点在对外提供服务，从而实现对业务的零中断或最小化中断。

**遵循以下黄金原则：**

1. **绝不跨大版本升级：** 不要从 19.x 直接升级到 23.x。遵循官方推荐的升级路径，通常是逐个大版本升级（例如 21.x -> 22.x -> 23.x）。如果版本跨度太大，请先升级到一个中间的长期支持（LTS）版本。

2. **详细阅读发布说明 (Release Notes)：** 这是**最重要**的一步！新版本的 Release Notes 会明确指出 **向后不兼容的变更 (Backward Incompatible Changes)**、配置文件的修改、系统表的变化等关键信息。忽略它可能导致灾难。

3. **先在测试环境演练：** 永远、永远不要直接在生产环境进行升级。必须在与生产环境架构、数据量相似的测试或预发环境中完整演练一遍升级流程，并进行业务回归测试。

4. **做好备份：** 在升级前，使用 `clickhouse-backup` 对整个集群的数据和元数据进行一次完整的**全量备份**。这是你最后的“救命稻草”。

5. **从副本节点开始：** 总是先升级一个分片（Shard）中的**副本（Replica）节点**，最后升级该分片的“主”节点或写入流量集中的节点。这样可以最大程度地降低对写入操作的影响。

6. **逐个节点，耐心观察：** 每升级一个节点，都要留出足够的观察时间（例如 15-30 分钟），检查该节点的日志、监控指标，确保它稳定运行后，再进行下一个节点的升级。

**升级前的准备工作 (Checklist)**

在登录到第一台服务器前，请完成以下清单：

* **确定升级路径：** 明确当前版本和目标版本（例如，从 22.8 LTS 升级到 23.3 LTS）。

* **通读 Release Notes:** 访问 [ClickHouse Changelog](https://www.google.com/url?sa=E\&q=https%3A%2F%2Fclickhouse.com%2Fdocs%2Fen%2Fwhats-new%2Fchangelog)，仔细阅读从当前版本到目标版本之间所有版本的 Release Notes，重点关注不兼容变更。

* **准备新版本软件包：**

  * **对于 YUM/APT:** 确保你的软件源配置正确，可以拉取到目标版本的软件包。

  * **对于手动安装:** 提前下载好目标版本的 RPM 或 DEB 包（通常需要 `clickhouse-common`, `clickhouse-client`, `clickhouse-server` 这三个包）。

* **完成全量备份：** 对整个集群执行 `sudo clickhouse-backup create cluster_backup_before_upgrade_to_vXX.X`。

* **通知业务方：** 告知相关业务和开发团队即将进行的升级窗口，并说明可能存在的风险（尽管我们的目标是平滑升级）。

* **准备回滚计划：** 思考如果升级失败，如何快速回滚。回滚计划通常是：立即停止升级流程，并使用旧版本的软件包重新安装出问题的节点。

## 第8章：融入现代数据栈 (Ecosystem Integration)

8.1. **数据注入 (Ingestion)**
\* **【实践】**: 使用 ClickHouse 的 Kafka 引擎实时消费 Kafka 数据。
\* 与 Flink/Spark 的集成。
8.2. **数据湖集成 (Data Lake Integration)**
\* 使用 `s3`, `gcs`, `hdfs` 表函数直接查询云存储或 HDFS 上的 Parquet/CSV 文件。
\* **【实践】**: 不导入数据，直接查询 S3 上的 Parquet 文件。
8.3. **BI 与可视化**
\* **【实践】**: 连接 Grafana，创建动态的监控仪表盘。
\* **【实践】**: 连接 Apache Superset，进行自助式数据探索和可视化。
\* 其他工具如 Tableau, Metabase 的连接。
8.4. **编程语言客户端**
\* Python (`clickhouse-connect`, `clickhouse-driver`)
\* Java (JDBC)
\* Go (`clickhouse-go`)
\* **【代码示例】**: 展示如何用 Python 脚本查询和插入数据。





# 第五部分：综合实战 (Final Project)

## 第9章：项目案例：构建一个实时的网站流量分析平台

9.1. **项目需求分析**
\* 实时统计：PV, UV, Top N 页面, 用户来源, 平均停留时长等。
\* 技术栈：Nginx (日志生成) -> Vector/Kafka (数据管道) -> ClickHouse (存储与分析) -> Grafana/Superset (可视化)。
9.2. **系统设计**
\* 设计 ClickHouse 表结构（原始日志表、分钟/小时级聚合表）。
\* 设计物化视图用于增量聚合。
9.3. **实施步骤 (指导性)**
\* 配置并启动所有组件。
\* 编写 ClickHouse DDL 语句。
\* 配置数据管道。
\* 在 BI 工具中创建仪表盘。
9.4. **成果展示与总结**
\* 展示最终的实时仪表盘。
\* 回顾项目中的挑战和解决方案，总结 ClickHouse 在该场景下的最佳实践。

***

