分布式检索引擎 ElasticSearch

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/fa0765de9dce09da7e21da5280067c95be998d564e3e3556591c2fc31ddef68c.jpg)

# elasticsearch

# 概述

# Elasticsearch 是什么

Elasticsearch（简称ES）是一个基于Apache Lucene(TM)的开源搜索引擎

Elasticsearch 是一个高伸缩的开源全文搜索和分析引擎，是一个基于JSON的分布式搜索和分析引擎，基于restful web接口，Elasticsearch是用Java语言开发的，基于Apache协议的开源项目，是目前最受企业欢迎的搜索引擎。

它可以快速地、近实时的存储，搜索和分析大规模的数据，一般被用作底层引擎/技术，为具有复杂搜索功能和要求的应用提供强有力的支撑。

# ElasticSearch特点

Elasticsearch是实时的分布式搜索分析引擎，内部使用Lucene做索引与搜索，有以下特点

- 近实时性：新增到 ES 中的数据在 1 秒后就可以被检索到，这种新增数据对搜索的可见性称为“近实时搜索”

- 全文检索：将全文检索、数据分析以及分布式技术，合并在了一起，才形成了独一无二的ES

- 分布式：意味着可以动态调整集群规模，弹性扩容

- 集群规模：可以扩展到上百台服务器，处理PB级结构化或非结构化数据

- 开箱即用：对用户而言，是开箱即用的，非常简单，作为中小型的应用，直接3分钟部署一下ES

- 不支持事务：数据库的功能面对很多领域是不够用的，事务，还有各种联机事务型的操作

# 使用场景

ElasticSearch广泛应用于各行业领域，比如维基百科，GitHub的代码搜索，电商网站的大数据日志统计分析，BI系统报表统计分析等。

# 记录和日志分析

ELK结合使用可以实现日志数据的收集整理

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/c1c29386090435e86a19f76f4dac5cb53ea556bc102a5a40f98c98d705ca56ce.jpg)

围绕Elasticsearch建立的生态系统使其成为实施和扩展日志记录解决方案最简单的系统之一，利用这一点将日志添加到他们的主要用例中，或者纯粹将我们用于日志记录。

从Beats到Logstash，再到Ingest Nodes，Elasticsearch为您提供了很多选择，可以随时随地获取数据并将其编入索引，从那里，像Kibana这样的工具使您能够创建丰富的仪表板和分析。

# 搜集和合并公共数据

像日志数据一样，Elastic Stack有很多工具可以使远程数据的获取和索引编制变得容易

而且，像大多数文档存储一样，缺乏严格的架构也使Elasticsearch可以灵活地接收多种不同的数据源，并且仍然可以使所有数据源易于管理和搜索。

# 全文检索

全文检索作为Elasticsearch的核心功能，得到了广泛的应用，远远超出了传统的企业搜索或电子商务

从欺诈检测/安全性到协作及其他方面，Elasticsearch的搜索功能强大，灵活，并且包含许多工具，可以使搜索变得更加容易，Elasticsearch拥有自己的查询DSL以及内置的功能。

# 事件数据和指标

Elasticsearch在如指标和应用程序事件之类的时间序列数据上也能很好地运行

这是巨大的Beats生态系统允许您轻松获取常见应用程序数据的另一个区域，无论您使用哪种技术，Elasticsearch都有很大的机会拥有可以立即获取指标和事件的组件。

# 可视化数据

Kibana拥有大量图表选项，用于地理数据的图块服务以及用于时间序列数据的TimeLion，是功能强大且易于使用的可视化工具。

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/b6251f7bedfb5f05f43e6db942671eba55e2c493529e3e05ec5d99509313ad77.jpg)

对于上述每个用例，Kibana都会处理一些可视组件，熟悉了各种数据提取工具后，您会发现Elasticsearch + Kibana将成为您可视化试图包裹数据的必备工具。

# 能做什么

要使用ElasticSearch我们需要知道ElasticSearch能够做什么

# 提供快速查询

试想一下，当你打开一个博客网站，搜索一篇博客的时候，等待了一分钟才有搜索结果，那将会是一个极差的体验

可想而知，这个博客网站肯定没有使用搜索引擎处理搜索的请求，而是使用了传统的关系型数据库查询，在庞大的数据面前，关系型数据库的查询就显得力不从心，相当耗时，Elasticsearch在这个时候可以帮上忙，使用博客数据建立索引库，依赖倒排索引的优势，为用户快速的呈现搜索的相关结果。

# 确保结果的相关性

接下来有一个难题: 如何将真正描述选举的帖子排序在前呢?有了 Elasticsearch，就可以使用几个算法来计算相关性的得分(relevancy score)，然后根据分数来将结果逐个排序。

默认情况下，计算文档相关性得分的算法是TF-IDF (term frequency-inverse document frequency)，词频逆文档频率，我们将在后面讨论这个概念，除了选择算法，Elasticsearch还提供了很多其他内置的功能来计算概相关性得分，以满足定制需求。

# 处理错误的拼写

当我们在使用搜索时，会出现英文拼写错误，中文错别字等情况时有发生

我们可以通过配置让Elasticsearch容忍一些错误，而不仅仅只是查找精确匹配，如我们输入“book”的时候由于手误输入了“bok”，如果搜索引擎能够意识到这一错误并且在搜索时帮我们修正这个错误，那么搜索会更快让人满意。

# 给予自动提示

当用户开始输入时，你可以帮助他们发现主流的查询和结果。

还可以通过自动提示技术预测他们所要输入的内容，就像 Web 上很多搜索引擎做的那样，你同样可以展示主流的结果，通过特殊的查询类型来匹配前缀、通配符或正则表达式。

# 使用统计信息

当用户不太清楚具体要搜索什么的时候，可以通过几种方式来协助他们。

一种方法是聚集统计数据，聚集是在搜索结果里得到一些统计数据，如每个分类有多少议题、每个分类中“赞”和“分享”的平均数量。

假想一下，进入博客时，用户会在右侧看见最近流行的议题，其中之一是自行车，对其感兴趣的读者会点击这个标题，进一步缩小范围，然后可能还有另外的聚集方式，将自行车相关的帖子分为“自行车鉴赏”“自行车大事件”等。

# ElasticSearch的发展

| 395 systems in ranking, August 2022 |          |          |                        |                            |          |        |        |
| ----------------------------------- | -------- | -------- | ---------------------- | -------------------------- | -------- | ------ | ------ |
|                                     | Rank     | DBMS     | Database Model         | Score                      |          |        |        |
| Aug 2022                            | Jul 2022 | Aug 2021 | Aug 2022               | Jul 2022                   | Aug 2021 |        |        |
| 1.                                  | 1.       | 1.       | Oracle +               | Relational, Multi-model    | 1260.80  | -19.50 | -8.46  |
| 2.                                  | 2.       | 2.       | MySQL +                | Relational, Multi-model    | 1202.85  | +7.98  | -35.37 |
| 3.                                  | 3.       | 3.       | Microsoft SQL Server + | Relational, Multi-model    | 944.96   | +2.83  | -28.39 |
| 4.                                  | 4.       | 4.       | PostgreSQL +           | Relational, Multi-model    | 618.00   | +2.13  | +40.95 |
| 5.                                  | 5.       | 5.       | MongoDB +              | Document, Multi-model      | 477.66   | +4.68  | -18.88 |
| 6.                                  | 6.       | 6.       | Redis +                | Key-value, Multi-model     | 176.39   | +2.77  | +6.51  |
| 7.                                  | 7.       | 7.       | IBM Db2                | Relational, Multi-model    | 157.23   | -3.99  | -8.24  |
| 8.                                  | 8.       | 8.       | Elasticsearch          | Search engine, Multi-model | 155.08   | +0.75  | -2.01  |
| 9.                                  | 9.       | ↑10.     | Microsoft Access       | Relational                 | 146.50   | +1.41  | +31.66 |
| 10.                                 | 10.      | ↓9.      | SQLite +               | Relational                 | 138.87   | +2.20  | +9.06  |

# 起源Lucene

Lucene 是一个用 Java 编写的非常古老的搜索引擎工具包，用来构建倒排索引（一种数据结构）和对这些索引进行检索，从而实现全文检索功能。

# 缺点

Lucene，必须使用Java来作为开发语言并将其直接集成到你的应用中，并且Lucene的配置及使用非常复杂，你需要深入了解检索的相关知识来理解它是如何工作的，有以下缺点

- 只能在Java项目中使用，并且要以jar包的方式直接集成项目中

- 使用非常复杂-创建索引和搜索索引代码繁杂

- 不支持集群环境-索引数据不同步（不支持大型项目）

- 索引数据如果太多就不行，索引库和应用所在同一个服务器，共同占用硬盘，共用空间少。

# 诞生

ElasticSearch的创始人期初是为了能够为妻子开发一个菜谱搜索应用而接触的Lucene

它本身不是一个应用程序无法直接提供用户使用，同样对其他语言不友好的，那么ElastiSearch的开发者在使用过程中遇到的一系列问题，他就在Lucene的基础上对之进行不断的优化形成了自己的一套应用程序'Compass'。

后来它自己在工作中同样遇到了一个需要高性能，分布式的搜索服务，所以他就在'Compass'的基础之上重新构建起了ElasticSearch，从设计之初的目标就是打造成分布式、高性能、基于JSON、Restful的易用性可易用与其他语言的独立服务。

# 发展

围绕ElasticSearch后来成立一家公司(Elastic公司)全面围绕ElasticSearch或者说是数据生态进行发展，该公司已经在去年上市(ESTC)，上市当天暴涨

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/6a48617bd3e41e282394797011ee2af99fec9d11ecfce1b9c11bff60852159eb.jpg)

ElasticSearch当前已经可以与多种客户端进行集成Python、PHP、.NET、Java等，当前同样支持与Hadoop、Spark等大数据分析平台进行集成。

ElasticSearch衍生出一系列的开源项目，例如业内较火的ELK Stack，ELK Stack是负责数据检索服务的ElasticSearch、数据采集解析服务的Logstash和负责数据可视化服务的Kibana的简称，Logstash是由Java语言编写的，同时负责数据的采集与解析工作，会导致服务的CPU与内存资源占用过高，后来ELastic又推出采用Go语言编写的Beats家族

# 基本概念

下面我们启动一下ES来看一下ES的相关概念，并结合案例来讲一下

# 启动服务

通过下面的命令可以快速启动ES服务

```
#切换到ElasticSearch用户
su elasticsearch
#启动Elasticsearch
sh bin/elasticsearch -d
#启动kibana
nohup sh bin/kibana >/dev/null 2>&1 &
```

# 索引类型

我们常见的索引包括正排索引和倒排索引

# 正排索引

正排索引是以文档的ID为关键字，表中记录文档中每个字段的位置信息，查找时扫描表中每个文档中字段的信息直到找出所有包含查询关键字的文档

# 正排索引说明

拿MySQL Innodb的聚簇索引来说，如下图所示，

一个极简版（无页属性）的B+树索引结构大概是这样，叶子节点存放完整数据，非叶子节点存放建立对应聚簇索引对应的字段（主键），一条可以使用到聚簇索引的SQL，会依次从上到下进行B+树的查找直到字段一致；

```
CREATE TABLE user_info (
    id int,
    name varchar(16),
    hobby varchar(256)
); 
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/7c5bbe571f1370cfc373f211d2fe32acaf36a37287a81d6fc74ada1cc48bf621.jpg)

# 索引查询

而对应非聚簇索引只是叶子节点的内容存放的是该表的主键信息，查询的顺序则是先通过非聚簇索引的字段找到叶子节点中一致的单个或者多个主键id，再使用这些主键id进行回表，最终获得对应的完整实体数据。

# 全表扫描

如果我们看上面在mysql中表的hobby爱好字段，如果我们有业务需求：根据用户爱好关键字如“篮球”去查询对应用户列表，我们怎么做，只能是写个字符串的like sql，全表扫描的逻辑。

```
SELECT *
FROM user_info
WHERE hobby LIKE '%篮球%';
```

即使我们对hobby字段创建了普通索引，在Innodb引擎下，在查询中想使用字符串类型的索引也只能走最左前缀索引的逻辑，即LIKE‘篮球%’。

# 倒排索引

倒排索引源于实际应用中需要根据属性的值来查找记录，也就是说，不是由记录来确定属性值，而是由属性值来确定记录，因而称为倒排索引

相比B+树的正排索引，如果我们对hobby字段建立了索引，他的倒排索引极简的数据格式如下。

创建倒排索引的field，会通过分词器根据语义将字段中的field分成一个一个对应的分词索引（term index），构成该类型数据的全部词索引集合，如“喜欢篮球、唱歌”会被分成“篮球”和“唱歌”两个term index;

第二列是含有这些term index对应的文档Id，这个数据可以帮助我们最终溯源到完整实体数据；

第三列则是对应term index在该文档字段中的位置，0表示在开头的位置，这个可以帮助标注检索出来数据的高亮信息。

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/841cb046991638d9d50581e9b0d341575b653b51bb5b5b72fa03c46472142b3d.jpg)

# 两种索引查找顺序

# 正排索引查找过程

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/30d76e26ca50cdab4aa792e26d4b959fc596934bea4459f0d1ac561bc843eb47.jpg)

# 倒排索引查找过程

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/33ea175bb220310649dc7e546984004fd1684f24457c363f5ca83f0bc49fc32c.jpg)

# 逻辑概念

假设我们在一个业务系统中选择MySQL做数据存储

那么我们需要先创建一个database，再创建一组相关的table，Elasticsearch同样具有这样的概念，使用index和mapping来组织数据，下面是Elasticsearch的一些基本概念

| ES中的概念          | 关系型数据库     | 说明                                                         |
| ------------------- | ---------------- | ------------------------------------------------------------ |
| 索引库 (indices)    | Databases 数据库 | indices是index的复数,代表许多的索引                          |
| 类型 (type)         | Table 数据表     | 类型是模拟mysql中的table概念,一个索引库下可以有不同类型的索引,比如商品索引,订单索引,其数据格式不同,不过这会导致索引库混乱,因此未来版本中会移除这个概念 |
| 文档 (document)     | Row 行           | 存入索引库原始的数据,比如每一条商品信息,就是一个文档         |
| 字段 (field)        | Columns列        | 文档中的属性                                                 |
| 映射配置 (mappings) | 表结构           | 字段的数据类型、属性、是否索引、是否存储等特性               |

# 索引 (Index)

一个索引由一个名字来标识（必须全部是小写字母），并且当我们要对这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/ca2f1d964a5ef50425f879ac7c79cfedf1499b2e4df57ff1eb32d4ae836cefa7.jpg)

一个索引相当于数据库，是多个相似文档的集合，必须通过索引才能进行搜索，使用索引能够极大的提升查询速度，类似于词典里面的目录。

当然在底层，肯定用到了倒排索引，最基本的结构就是“keyword”和“PostingList”，Postinglist就是一个int的数组，存储了所有符合某个term的文档id，另外这个倒排索引相比特定词项出现过的文档列表，会包含更多其它信息。

它会保存每一个词项出现过的文档总数，在对应的文档中一个具体词项出现的总次数，词项在文档中的顺序，每个文档的长度，所有文档的平均长度等等相关信息。

# 类型 (Type)

一个类型过去是索引的逻辑类别/分区，允许你在同一索引中存储不同类型的文档

例如，一种类型用于用户，另一种类型用于博客文章，在索引中创建多个类型不再可能，类型的整个概念将在稍后的版本中删除，相当于sql领域中表的概念。

# 类型的变化

在不同的elasticsearch中，类型发生了不同的变化

| 版本 | Type                                        |
| ---- | ------------------------------------------- |
| 5.x  | 支持多种Type                                |
| 6.x  | 只有一种Type                                |
| 7.x  | 默认不在支持自定义的索引类型,默认类型为_doc |

# 文档 (Document)

一个文档是可以被索引的一个基本单元，相当于数据库中的一条数据，索引和搜索数据的最小单位是文档

# 文档结构

结合上面的倒排索引可以知道，倒排索引存储包含了索引文件以及数据文件

在我们查询的时候，一般使用term词项先去反向索引库中查询到文档_id，然后根据_id去文档库中找到最初上传上去的所有文档原始内容_source，当然，前提是上传的时候有保存原始数据

# 倒排索引库

倒排索引库存放的是所有的倒排索引字典数据以及文档id等，查询的时候先根据倒排索引查询到文档ID然后再到文档库查询具体的文档

```
term词项 _id 词频、位置
term词项 _id 词频、位置
term词项 _id 词频、位置
...
```

# 文档库

文档库就是我们所说的数据文件，ES根据id来文档库查询文档数据

```
_index
_type
_id _version _source
_id _version _source
_id _version _source
... 
```

# 字段 (Field)

相当于数据库表的字段，每个字段有不同的类型

# 映射 (Mapping)

Mapping是对处理数据时的方式和规则作出一定的限制，如字段的类型、默认值、分析器、是否被索引等，映射定义了每个字段的类型、字段所使用的分词器等。

可以显式映射，由我们在索引映射中进行预先定义，也可以动态映射，在添加文档的时候，由es自动添加到索引，这个过程不需要事先在索引进行字段数据类型匹配等等，es会自己推断数据类型。

get itheima/_mapping

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/b03bd80c4dded7449744626d01e4abca056954dbdfa9f0801adbff3ba9b8834b.jpg)

# 物理概念

Elasticsearch是一个分布式系统，其数据会分散存储到不同的节点上，为了实现这一点，需要将每个index中的数据划分到不同的块中，然后将这些数据块分配到不同的节点上存储

# 集群 (cluster)

一个集群就是由一个或多个节点组织在一起，它们共同持有整个的数据，并一起提供索引和搜索功能

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/f271676862b31147f7bdd8124420d479d614ab4b745b58344bcdfe0a89b58a62.jpg)

集群（cluster）是一个或多个节点（node)的集合，这些节点将共同拥有完整的数据，并跨节点提供联合索引、搜索和分析功能。

一个集群由一个唯一的名字标识，这个名字默认就是“elasticsearch”，这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群。

ES集群是一个 P2P 类型(使用 gossip 协议)的分布式系统，除了集群状态管理以外，其他所有的请求都可以发送到集群内任意一台节点上，这个节点可以自己找到需要转发给哪些节点，并且直接跟这些节点通信，所以从网络架构及服务配置上来说，构建集群所需要的配置极其简单

集群中节点数量没有限制，一般大于等于2个节点就可以看做是集群了，一般处于高性能及高可用方面来考虑一般集群中的节点数量都是3个及3个以上。

# 节点 (node)

一个节点是集群中的一个服务器，作为集群的一部分，它存储数据，参与集群的索引和搜索功能

和集群类似，一个节点也是由一个名字来标识的，默认情况下，这个名字是一个随机的漫威漫画角色的名字，这个名字会在启动的时候赋予节点

一个节点可以通过配置集群名称的方式来加入一个指定的集群，默认情况下，每个节点都会被安排加入到一个叫做“elasticsearch”的集群中，这意味着，如果你在你的网络中启动了若干个节点，并假定它们能够相互发现彼此，它们将会自动地形成并加入到一个叫做“elasticsearch”的集群中。

# 分片 (Shards)

分片的存在是为了解决单个索引大量文档的存储问题、以及搜索是响应慢等问题。

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/4f6e4b3cad66973b8c5f0c633d0a27a3f1d6011cc0d8ad213af17a28ab0500a2.jpg)

比如，一个具有10亿文档的索引占据1TB的磁盘空间，而任一节点都没有这样大的磁盘空间，或者单个节点处理搜索请求，响应太慢，为了解决这个问题，Elasticsearch提供了将索引划分成多份的能力，这些份就叫做分片。

将一个索引划分成了多份，每一份就称之为分片，每个分片也是一个功能完善的“索引”，这个“索引”可以被放置到集群的任意节点上，通过"分"的思想，可以突破单机在存储空间和处理性能上的限制，这是分布式系统的核心目的

至于一个分片怎样分布，它的文档怎样聚合回搜索请求，是完全由Elasticsearch管理的，对于作为用户的你来说，这些都是透明的。

# 副本 (Replicas)

而对于分布式存储而言，还有一个重要特性是"冗余"，因为分布式的前提是：接受系统中某个节点因为某些故障退出，为了保证在故障节点退出后数据不丢失，同一份数据需要拷贝多份存在不同节点上

在一个网络 / 云的环境里，失败随时都可能发生，在某个分片/节点不知怎么的就处于离线状态，或者由于任何原因消失了，这种情况下，有一个故障转移机制是非常有用并且是强烈推荐的，为此目的，Elasticsearch 允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片(副本)。

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/41e180f5fc527f2450aee994ea70b05c751dab3eb4d740b0bb92f6f1c9f07e06.jpg)

# 副本的作用

复制之所以重要，有两个主要原因：

在分片/节点故障的情况下，提供了高可用性，因为这个原因，注意副本分片从不与主分片置于同一节点上这一点非常重要的，扩展你的搜索量/吞吐量，因为搜索可以在所有的复制上并行运行，总之每个索引可以被分成多个分片。

一个索引也可以被复制0次（意思是没有复制）或多次，一旦复制了，每个索引就有了主分片（作为复制源的原来的分片）和复制分片（主分片的拷贝）之别，分片和复制的数量可以在索引创建的时候指定，在索引创建之后，你可以在任何时候动态地改变复制的数量，但是你事后不能改变分片的数量。

# 段 (segment)

segment来自于lucene，因为ES底层就是使用的lucene，一个shard包含一组segment, segment是最小的数据单元

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/5abe48d9d9ea5da4d2e1c75eef1a1fa1ef336e88e8b8e8f91263c848cd9298f7.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/a590406900cb16737d8c2603a1990a7c788ec996dba9573994fe5732d9a17967.jpg)

Elasticsearch每隔一段时间产生一个新的segment，里面包含了新写入的数据，lucene的数据写入会先写如到缓存（buffer）中，当达到一定数量以后，会flush成文一个segment，写入到磁盘当中，每个segment有自己独立的索引，可以单独查询。

segment不会被修改，数据的的写入都是进行批量的追加，避免了随机写的存在，提高了吞吐量，segment可以被删除，但也不是修改segment文件，而是由另外的文件记录需要被删除的documentId。

index的查询是对多个segment文件的查询，其中也包含了处理被删除文件的处理，并对查询结果进行合并，为了进行查询优化，lucene有策略对多个segment进行优化。

# 集群搭建

集群部署以及数据同步参考《分布式检索引擎-异构数据同步》

# 集群角色

一个Elasticsearch实例代表了一个ES节点，如果不通过 node.roles 设置节点的角色，一个ES节点默认的节点角色有很多个不同的角色

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/6433059b296822f73c6268b7410913c1a1f18adb365bf1b4bc422f4e2a381b0d.jpg)

每个节点既可以是候选主节点也可以是数据节点，通过在配置文件 ../config/elasticsearch.yml 中设置即可，默认都为 true，ES节点有如下角色：

# Master节点

其实这个是master准确的来说是具有成为master节点资格的节点，即 master-eligible node

# 主要职责

Master角色的主要职责是负责集群层面的相关操作，管理集群变更，如创建或删除索引，跟踪哪些节点是群集的一部分，并决定哪些分片分配给相关的节点。

拥有一个稳定的主节点对集群非常重要，候选主节点可以通过节点选举过程被选举为主节点，主节点最好是专用的，不和其他角色共用，以免其他的操作对master节点负载造成影响，导致集群不可用

# 角色介绍

主节点负责轻量级集群范围的操作，任何不是仅投票节点的合格节点都可以通过选举成为主节点

主节点必须有一个 path.data 目录，其内容在重启后仍然存在，就像数据节点一样，因为这是存储集群元数据的地方，集群元数据描述了如何读取存储在数据节点上的数据，因此如果丢失，则无法读取存储在数据节点上的数据。

# 仅投票节点

只能参与主节点的投票选举环节，但是自己不能被选举为master

# 主要职责

仅投票节点用来凑数的，如果只部署了两个候选主节点，当一个节点挂掉后集群将会不可用，加入了仅投票节点则不一样，有了仅投票节点可以帮助快速选择一个主节点出来，并且仅投票节点不会选为主节点，不存储数据，所以消耗的资源也很小。

# 角色介绍

高可用性 (HA) 集群需要至少三个符合主节点的节点，其中至少两个不是仅投票节点，这样即使其中一个节点发生故障，集群也能够选举出一个主节点。

# 数据节点

负责数据的存储和相关的操作，例如对数据进行增、删、改、查和聚合等操作

# 主要职责

数据节点主要是存储索引数据的节点，执行数据相关操作：CRUD、搜索，聚合操作等。

数据节点对cpu，内存，I/O要求较高，在优化的时候需要监控数据节点的状态，当资源不够的时候，需要在集群中添加新的节点。

# 角色介绍

保存包含已编入索引的文档的分片，数据节点处理数据相关操作，如 CRUD、搜索和聚合这些操作是 I/O 密集型、内存密集型以及 CPU 密集型的，监控这些资源并在它们过载时添加更多数据节点非常重要

# 预处理节点

这是从5.0版本开始引入的概念，预处理节点可以执行由一个或多个摄取处理器组成的预处理管道

# 主要职责

预处理操作运行在索引文档之前，即写入数据之前，通过事先定义好的一系列processors(处理器)和pipeline（管道），对数据进行某种转换、富化

# 角色介绍

能执行预处理管道，有自己独立的任务要执行，在索引数据之前可以先对数据做预处理操作，不负责数据存储也不负责集群相关的事务，类似于 logstash 中 filter 的作用，功能相当强大。

在实际文档索引发生之前，使用Ingest节点预处理文档，Ingest节点拦截批量和索引请求，它应用转换，然后将文档传递回索引，在数据被索引之前，通过预定义好的处理管道对数据进行预处理。

# 仅协调节点

如果您取消了候选主节点的职责、保存数据和预处理文档的能力，那么您就剩下一个只能路由请求、处理搜索减少阶段和分发批量索引的协调节点

# 只要职责

协调节点将请求转发给保存数据的数据节点，每个数据节点在本地执行请求，并将结果返回给协调节点。

协调节点收集完数据合，将每个数据节点的结果合并为单个全局结果，对结果收集和排序的过程可能需要很多CPU和内存资源。

# 角色介绍

本质上，仅协调节点的行为就像智能负载均衡器，通过从数据和符合主节点的节点卸载协调节点角色，仅协调节点可以使大型集群受益，他们加入集群并接收完整的集群状态，就像其他每个节点一样，他们使用集群状态将请求直接路由到适当的地方。

# 节点配置方式

以下是个个节点的配置方式

| 节点类型          | 配置参数    | 默认值                                             |
| ----------------- | ----------- | -------------------------------------------------- |
| master eligible   | node.master | true                                               |
| data              | node.data   | true                                               |
| ingest            | node.ingest | true                                               |
| Coordinating only | 无          | 每个节点默认都是 Coordinating,设置其他类型为 false |
| machine learning  | node.ml     | true (需要 enable x-pack)                          |

# 集群脑裂问题

脑裂是因为集群中的节点失联导致的

# 脑裂分析

例如一个集群中，主节点与其它节点失联：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/0372592795681e0ba716cf777fefcf0c69dd6f01c1d9e4bcc062ea2ad3359679.jpg)

# 重新选主

此时，node2和node3认为node1宕机，就会重新选主：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/3c1bbe1a95c88bf90635328f0dd55dde6dc272f363fc5a0b662ddd6599c2b366.jpg)

网络阻塞

# 出现脑裂

当node3当选后，集群继续对外提供服务，node2和node3自成集群，node1自成集群，两个集群数据不同步，出现数据差异，当网络恢复后，因为集群中有两个master节点，集群状态的不一致，出现脑裂的情况：

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/ec122052a131166339d57adae7ad2267abee9d31b56836ed93c9e1d00257f0a4.jpg)

# 解决方案

解决脑裂的方案是，要求选票超过（候选主节点数量 + 1）/2 才能当选为主

因此候选主节点数量最好是奇数，对应配置项是 discovery.zen.minimum_master_nodes，在 es7.0 以后，已经成为默认配置，因此一般不会发生脑裂问题

例如：3个节点形成的集群，选票必须超过 (3 + 1)/2 ，也就是2票，node3得到node2和node3的选票，当选为主，node1只有自己1票，没有当选，集群中依然只有1个主节点，没有出现脑裂。

# 数据库设计

在MySQL中数据库设计非常重要，同样在ES中数据库设计也是非常重要的

# 索引设计

我们创建索引就像创建表结构一样，必须非常慎重的，索引如果创建不好后面会出现各种各样的问题

# 索引设计的重要性

首先索引创建后，索引的分片只能通过 _split 和 _shrink 接口对其进行成倍的增加和缩减，主要是因为 es 的数据是通过 _routing 分配到各个分片上面的，所以本质上是不推荐去改变索引的分片数量的，因为这样都会对数据进行重新的移动。

还有就是索引只能新增字段，不能对字段进行修改和删除，缺乏灵活性，所以每次都只能通过_reindex重建索引了，还有就是一个分片的大小以及所以分片数量的多少严重影响到了索引的查询和写入性能，所以可想而知，设计一个好的索引能够减少后期的运维管理和提高不少性能，所以前期对索引的设计是相当的重要的。

# 基于时间的Index设计

Index设计时要考虑的第一件事，就是基于时间对Index进行分割，即每隔一段时间产生一个新的Index

# 这样设计的目的

因为现实世界的数据是随着时间的变化而不断产生的，切分管理可以获得足够的灵活性和更好的性能

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/2f695a52d9cc3e3d91645444e5563884cb02cc94a5a0a85ea1fc52f7953f1ab0.jpg)

One Index

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/8aadd4c0466b5aa2109f31a4d5f36633198cda5fff789ed5da6c5a72179a75c6.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/70903946a8ea02e9707922f70a6474288a492e5496a8bfd6e2d67408a94d53c9.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/724ec17e879cec15b8c125d8f378ce2a3327151f9f8d0ba43ce4e20b73515322.jpg)

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/7144d8c1459085e72a1177e3c4e06c5dec2dc6b92b2f02b54ebff2bdf5ebbccd.jpg)

Time-based Index

如果数据都存储在一个Index中，很难进行扩展和调整，因为Elasticsearch中Index的某些设置在创建时就设定好了，是不能更改的，比如Primary Shard的个数。

而根据时间来切分Index，则可以实现一定的灵活性，既可以在数据量过大时及时调整Shard个数，也可以及时响应新的业务需求。

大多数业务场景下，客户对数据的请求都会命中在最近一段时间上，通过切分Index，可以尽可能的避免扫描不必要的数据，提高性能。

# 时间间隔

根据上面的分析，自然是时间越短越能保持灵活性，但是这样做就会导致产生大量的Index，而每个Index都会消耗资源来维护其元信息的，因此需要在灵活性、资源和性能上做权衡

- 常见的间隔有小时、天、周和月：先考虑总共要存储多久的数据，然后选一个既不会产生大量Index又能够满足一定灵活性的间隔，比如你需要存储6个月的数据，那么一开始选择“周”这个间隔就会比较合适。

- 考虑业务增长速度：假如业务增长的特别快，比如上周产生了1亿数据，这周就增长到了10亿，那么就需要调低这个间隔来保证有足够的弹性能应对变化。

# 如何实现分割

切分行为是由客户端（数据的写入端）发起的，根据时间间隔与数据产生时间将数据写入不同的Index中，为了易于区分，会在Index的名字中加上对应的时间标识

创建新时期这件事，可以是客户端主动发起一个创建的请求，带上具体的Settings、Mappings等信息，但是可能会有一个时间错位，即有新数据写入时新的Index还没有建好，Elasticsearch提供了更优雅的方式来实现这个动作，即 Index Template

# 分片设计

所谓分片设计，就是如何设定主分片的个数

看上去只是一个数字而已，也许在很多场景下，即使不设定也不会有问题（ES7默认是1个主分片一个副本分片），但是如果不提前考虑，一旦出问题就可能导致系统性能下降、不可访问、甚至无法恢复，换句话说，即使使用默认值，也应该是通过足够的评估后作出的决定，而非拍脑袋定的。

# 限制分片大小

单个Shard的存储大小不超过30GB

Elastic专家根据经验总结出来大家普遍认为30GB是个合适的上限值，实践中发现单个Shard过大（超过30GB）会导致系统不稳定。

其次，为什么不能超过30GB？主要是考虑Shard Relocate过程的负载，我们知道，如果Shard不均衡或者部分节点故障，Elasticsearch会做Shard Relocate，在这个过程中会搬移Shard，如果单个Shard过大，会导致CPU、IO负载过高进而影响系统性能与稳定性。

# 评估分片数量

单个Index的 Primary Shard 个数 = k * 数据节点个数

在保证第一点的前提下，单个Index的Primary Shard个数不宜过多，否则相关的元信息与缓存会消耗过多的系统资源，这里的k，为一个较小的整数值，建议取值为1,2等，整数倍的关系可以让Shard更好地均匀分布，可以充分的将请求分散到不同节点上。

# 小索引设计

对于很小的Index，可以只分配1~2个Primary Shard的

有些情况下，Index很小，也许只有几十、几百MB左右，那么就不用按照第二点来分配了，只分配1~2个Primary Shard是可以，不用纠结。

# 使用索引模板

就是把已经创建好的某个索引的参数设置(settings)和索引映射(mapping)保存下来作为模板，在创建新索引时，指定要使用的模板名，就可以直接重用已经定义好的模板中的设置和映射

Elasticsearch基于与索引名称匹配的通配符模式将模板应用于新索引，也就是说通过索引进行匹配，看看新建的索引是否符合索引模板，如果符合，就将索引模板的相关设置应用到新的索引，如果同时符合多个索引模板呢，这里需要对参数priority进行比较，这样会选择priority大的那个模板进行创建索引。

在创建索引模板时，如果匹配有包含的关系，或者相同，则必须设置priority为不同的值，否则会报错，索引模板也是只有在新创建的时候起到作用，修改索引模板对现有的索引没有影响，同样如果在索引中设置了一些设置或者mapping都会覆盖索引模板中相同的设置或者mapping

索引模板的用途

索引模板一般用在时间序列相关的索引中。

也就是说, 如果你需要每间隔一定的时间就建立一次索引, 你只需要配置好索引模板, 以后就可以直接使用这个模板中的设置, 不用每次都设置settings和mappings.

创建索引模板

```
PUT _index_template/logstash-village
{
    "index_patterns": [
    "logstash-village-" // 可以通过"logstash-village-"来适配创建的索引
    ],
    "template": {
    "settings": {
    "number_of_shards": "3", //指定模板分片数量
    "number_of_replicas": "2" //指定模板副本数量
    },
    "aliases": {
    "logstash-village": {} //指定模板索引别名
    },
    "mappings": { //设置映射
    "dynamic": "strict", //禁用动态映射
    "properties": {
    "@timestamp": {
    "type": "date",
    "format": "strict_date_optional_time||epoch_millis||yyyy-MM-dd"
    HH:mm:ss"
    },
    "@version": {
    "doc_values": false,
    "index": "false",
    "type": "integer"
    },
    "name": {
    "type": "keyword"
    },
    "province": {
    "type": "keyword"
    },
    "city": {
    "type": "keyword"
    },
    "area": {
    "type": "keyword"
    },
    "addr": {
    "type": "text",
    "analyzer": "ik_smart"
    },
    "location": {
    "type": "geo_point"
    },
    "property_type": {
    "type": "keyword"
    }
}
},
"property_company": {
    "type": "text",
    "analyzer": "ik_smart"
},
"property_cost": {
    "type": "float"
},
"floorage": {
    "type": "float"
},
"houses": {
    "type": "integer"
},
"built_year": {
    "type": "integer"
},
"parkings": {
    "type": "integer"
},
"volume": {
    "type": "float"
},
"greening": {
    "type": "float"
},
"producer": {
    "type": "keyword"
},
"school": {
    "type": "keyword"
},
"info": {
    "type": "text",
    "analyzer": "ik_smart"
}
} 
```

# 模板参数

下面是创建索引模板的一些参数

| 参数名称       | 参数介绍                                                     |
| -------------- | ------------------------------------------------------------ |
| index_patterns | 必须配置,用于在创建期间匹配索引名称的通配符(*)表达式数组     |
| template       | 可选配置,可以选择包括别名、映射或设置配置                    |
| composed_of    | 可选配置,组件模板名称的有序列表。组件模板按指定的顺序合并,这意味着最后指定的组件模板具有最高的优先级 |
| priority       | 可选配置,创建新索引时确定索引模板优先级的优先级。选择具有最高优先级的索引模板。如果未指定优先级,则将模板视为优先级为0(最低优先级) |
| version        | 可选配置,用于外部管理索引模板的版本号                        |
| _meta          | 可选配置,关于索引模板的可选用户元数据,可能有任何内容         |

# 映射配置

上面我们配置了映射模板，但是我们用到了映射，下面我们说下映射

# 什么是映射

在创建索引时，可以预先定义字段的类型（映射类型）及相关属性

数据库建表的时候，我们DDL依据一般都会指定每个字段的存储类型，例如：varchar、int、datetime等，目的很明确，就是更精确的存储数据，防止数据类型格式混乱，在Elasticsearch中也是这样，创建索引的时候一般也需要指定索引的字段类型，这种方式称为映射（Mapping）

# 被动创建（动态映射）

此时字段和映射类型不需要事先定义，只需要存在文档的索引，当向此索引添加数据的时候当遇到不存在的映射字段，ES会根据数据内容自动添加映射字段定义。

# 动态映射规则

使用动态映射的时候，根据传递请求数据的不同会创建对应的数据类型

| 数据类型      | Elasticsearch 数据类型                                       |
| ------------- | ------------------------------------------------------------ |
| null          | 不添加任何字段                                               |
| true或者false | boolean类型                                                  |
| 浮点数据      | float类型                                                    |
| integer数据   | long类型                                                     |
| object        | object类型                                                   |
| array         | 取决于数组中的第一个非空值的类型。                           |
| string        | 如果此内容通过了日期格式检测,则会被认为是date数据类型如果此值通过了数值类型检测则被认为是double或者long数据类型带有关键字子字段会被认为一个text字段 |

# 禁止动态映射

一般生产环境下需要禁用动态映射，使用动态映射可能出现以下问题

1. 造成集群元数据一直变更，导致不稳定；

1. 可能造成数据类型与实际类型不一致；

1. 对于一些异常字段或者是扫描类的字段，也会频繁的修改mapping，导致业务不可控。

如何禁用动态映射，动态 mapping 的 dynamic 字段进行配置，可选值及含义如下

- true：支持动态扩展，新增数据有新的字段属性时，自动添加对于的mapping，数据写入成功

- false：不支持动态扩展，新增数据有新的字段属性时，直接忽略，数据写入成功

- strict：不支持动态扩展，新增数据有新的字段时，报错，数据写入失败

# 主动创建（显示映射）

动态映射只能保证最基础的数据结构的映射

所以很多时候我们需要对字段除了数据结构定义更多的限制的时候，动态映射创建的内容很可能不符合我们的需求，所以可以使用 PUT {index}/mapping 来更新指定索引的映射内容。

# 映射类型

我们要创建映射必须还要知道映射类型，否则就会走默认的映射类型，下面我们看看常用的映射类型

# 准备工作

我们先创建一个用于测试映射类型的索引

```
PUT mapping_demo 
```

# 字符串类型

字符串类型是我们最常用的类型之一，我们操作的时候字符串类型可以被设置为以下几种类型

text

当一个字段是要被全文搜索的，比如Email内容、产品描述，应该使用text类型，text类型会被分词

设置text类型以后，字段内容会被分词，在生成倒排索引以前，字符串会被分析器分成一个一个词项，text类型的字段不用于排序，很少用于聚合

keyword

keyword类型不会被分词，常用于关键字搜索，比如姓名、email地址、主机名、状态码和标签等

如果字段需要进行过滤(比如查姓名是张三发布的博客)、排序、聚合，keyword类型的字段只能通过精确值搜索到，常常被用来过滤、排序和聚合

# 两者区别

它们的区别在于 text 会对字段进行分词处理而 keyword 则不会进行分词

也就是说如果字段是 text 类型，存入的数据会先进行分词，然后将分完词的词组存入索引，而 keyword 则不会进行分词，直接存储，这样划分数据更加节省内存。

数字类型

数字类型也是我们最常用的类型之一，下面我们看下数字类型的使用

| 类型         | 取值范围                      |
| ------------ | ----------------------------- |
| long         | -263 ~ 263                    |
| integer      | -231 ~ 231                    |
| short        | -215 ~ 215                    |
| byte         | -27 ~ 27                      |
| double       | 64位的双精度 IEEE754 浮点类型 |
| float        | 32位的双精度 IEEE754 浮点类型 |
| half_float   | 16位的双精度 IEEE754 浮点类型 |
| scaled_float | 缩放类型的浮点类型            |

# 注意事项

- 在满足需求的情况下，优先使用范围小的字段，字段长度越小，索引和搜索的效率越高。

# 使用案例

我们先创建一个映射，name是keyword类型，描述是text类型的

```
PUT mapping_demo/_mapping
{
    "properties": {
    "name": {
    "type": "keyword"
    },
    "city": {
    "type": "text",
    "analyzer": "ik_smart"
    }
    }
} 
```

插入数据

```
PUT mapping_demo/_doc/1
{
    "name": "北京小区",
    "city": "北京市昌平区回龙观街道"
}
```

对于keyword的name字段进行精确查询

```
GET mapping_demo/_search
{
    "query": {
    "term": {
    "name": "北京小区"
    }
    }
}
```

对于text的city进行模糊查询

```
GET mapping_demo/_search
{
    "query": {
    "term": {
    "city": "北京市"
    }
    }
}
```

# 日期类型

# JSON表示日期

JSON没有表达日期的数据类型，所以在ES里面日期只能是下面其中之一

- 格式化的日期字符串，比如："2015-01-01" or "2015/01/01 12:10:30"

- 用数字表示的从新纪元开始的毫秒数

- 用数字表示的从新纪元开始的秒数 (epoch_second)

注意点：毫秒数的值是不能为负数的，如果时间在1970年以前，需要使用格式化的日期表达

# ES如何处理日期

在ES的内部，时间会被转换为UTC时间（如果声明了时区）并使用从新纪元开始的毫秒数的长整形数字类型的进行存储，在日期字段上的查询，内部将会转换为使用长整形的毫秒进行范围查询，根据与字段关联的日期格式，聚合和存储字段的结果将转换回字符串

注意点：日期最终都会作为字符串呈现，即使最开始初始化的时候是利用JSON文档的long声明的

# 默认日期格式

日期的格式可以被定制化的，如果没有声明日期的格式，它将会使用默认的格式：

```
"strict_date_optional_time||epoch_millis" 
```

这意味着它将会接收带时间戳的日期，它将遵守strict_date_optional_time限定的格式（yyyy-MM-dd'T'HH:mm:ss.SSSZ 或者 yyyy-MM-dd）或者毫秒数

# 日期格式示例

```
PUT mapping_demo/_mapping
{
    "properties": {
    "datetime": {
    "type": "date", 
"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
}
}
# 添加数据
PUT mapping_demo/_doc/2
{
    "name": "河北区",
    "city": "河北省小区",
    "datetime": "2022-02-21 11:35:42"
}
```

# 日期类型参数

下面表格里的参数可以用在date字段上面

| 参数             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| doc_values       | 该字段是否按照列式存储在磁盘上以便于后续进行排序、聚合和脚本操作,可配置 true(默认)或 false |
| format           | 日期的格式                                                   |
| locale           | 解析日期中时使用了本地语言表示月份时的名称和/或缩写,默认是 ROOT locale |
| ignore_malformed | 如果设置为true,则奇怪的数字就会被忽略,如果是false(默认)奇怪的数字就会导致异常并且该文档将会被拒绝写入。需要注意的是,如果在脚本参数中使用则该属性不能被设置 |
| index            | 该字段是否能快速的被查询,默认是true。date类型的字段只有在doc_values设置为true时才能被查询,尽管很慢。 |
| null_value       | 替代null的值,默认是null                                      |
| on_script_error  | 定义在脚本中如何处理抛出的异常,fail(默认)则整个文档会被拒绝索引,continue:继续索引 |
| script           | 如果该字段被设置,则字段的值将会使用该脚本产生,而不是直接从source里面读取。 |
| store            | true or false(默认)是否在_source之外在独立存储一份           |

# 布尔类型

boolean类型用于存储文档中的true/false

# 范围类型

顾名思义，范围类型字段中存储的内容就是一段范围，例如年龄30-55岁，日期在2020-12-28到2021-01-01之间等

类型范围

es中有六种范围类型：

```
- integer_range
- float_range
- long_range
- double_range
- date_range
- ip_range 
```

使用实例

```
PUT mapping_demo/_mapping
{
    "properties": {
    "age_range": {
    "type": "integer_range"
    }
    }
}

# 指定年龄范围，可以使用 gt、gte、lt、lte。
PUT mapping_demo/_doc/3
{
    "name": "张三",
    "age_range": {
    "gt": 20,
    "lt": 30
    }
}
```

# 分词器

# 什么是分词器

分词器的主要作用将用户输入的一段文本，按照一定逻辑，分析成多个词语的一种工具

顾名思义，文本分析就是把全文本转换成一系列单词（term/token）的过程，也叫分词，在ES中，Analysis是通过分词器（Analyzer）来实现的，可使用ES内置的分析器或者按需定制化分析器。

举一个分词简单的例子：比如你输入 Mastering Elasticsearch，会自动帮你分成两个单词，一个是 mastering，另一个是 elasticsearch，可以看出单词也被转化成了小写的。

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/92d7b26893424bb0f6aff05364437978651c3c54153ac271c6564a3cca259dcd.jpg)

# 分词器构成

分词器是专门处理分词的组件，分词器由以下三部分组成：

# character filter

接收原字符流，通过添加、删除或者替换操作改变原字符流

例如：去除文本中的html标签，或者将罗马数字转换成阿拉伯数字等，一个字符过滤器可以有零个或者多个

# tokenizer

简单的说就是将一整段文本拆分成一个个的词

例如拆分英文，通过空格能将句子拆分成一个个的词，但是对于中文来说，无法使用这种方式来实现，在一个分词器中，有且只有一个 tokenizer

# token filters

将切分的单词添加、删除或者改变

例如将所有英文单词小写，或者将英文中的停词 a 删除等，在 token filters 中，不允许将 token（分出的词）的 position 或者 offset 改变。同时，在一个分词器中，可以有零个或者多个 token filters。

# 分词顺序

Java is the best language in the world.

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/52dbcf8f03ae0ce94cc580cc94e3fe4c28aeb5f17251c7b549c25bf6d9a0a88d.jpg)

[java,is,the,best,language,in,the,world]

同时 Analyzer 三个部分也是有顺序的，从图中可以看出，从上到下依次经过 Character Filters, Tokenizer 以及 Token Filters，这个顺序比较好理解，一个文本进来肯定要先对文本数据进行处理，再去分词，最后对分词的结果进行过滤。

# 测试分词

可以通过 _analyzer API 来测试分词的效果，我们使用下面的 html 过滤分词

```
POST _analyze
{
    "text":"<b>hello world<b>" # 输入的文本
    "char_filter":["html_strip"], # 过滤html标签
    "tokenizer":"keyword", # 原样输出
}
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/9e1c5f7b15e30c5bc5d4d51ffcb36e29b0194e55751b29c51a136f670fce3841.jpg)

# 什么时候分词

文本分词会发生在两个地方：

- 创建索引：当索引文档字符类型为 text 时，在建立索引时将会对该字段进行分词。

- 搜索：当对一个 text 类型的字段进行全文检索时，会对用户输入的文本进行分词。

# 创建索引时指定分词器

如果设置手动设置了分词器，ES将按照下面顺序来确定使用哪个分词器

- 先判断字段是否有设置分词器，如果有，则使用字段属性上的分词器设置

- 如果设置了 analysis.analyzer.default，则使用该设置的分词器

- 如果上面两个都未设置，则使用默认的 standard 分词器

# 字段指定分词器

为addr属性指定分词器，这里我们使用的是中文分词器

```
PUT my_index
{
    "mappings": {
    "properties": {
    "info": {
    "type": "text",
    "analyzer": "ik_smart"
    }
    }
} 
```

设置默认分词器

```
PUT my_index
{
    "settings": {
    "analysis": {
    "analyzer": {
    "default": {
    "type": "simple"
    }
    }
    }
    }
} 
```

# 搜索时指定分词器

在搜索时，通过下面参数依次检查搜索时使用的分词器，这样我们的搜索语句就会先分词，然后再来进行搜索

- 搜索时指定 analyzer 参数

- 创建mapping时指定字段的 search_analyzer 属性

- 创建索引时指定 setting 的 analysis.analyzer.default_search

- 查看创建索引时字段指定的 analyzer 属性

- 如果上面几种都未设置，则使用默认的 standard 分词器。

# 指定analyzer

搜索时指定analyzer查询参数

```
GET my_index/_search
{
    "query": {
    "match": {
    "message": {
    "query": "Quick foxes",
    "analyzer": "stop"
    }
    }
    }
} 
```

指定字段analyzer

```
PUT my_index
{
    "mappings": {
    "properties": {
    "title": {
    "type": "text",
    "analyzer": "whitespace",
    "search_analyzer": "simple"
    }
    }
} 
```

指定默认default_seach

```
PUT my_index
{
    "settings": {
    "analysis": {
    "analyzer": {
    "default": {
    "type": "simple"
    },
    "default_seach": {
    "type": "whitespace"
    }
    }
    }
} 
```

# 内置分词器

es在索引文档时，会通过各种类型 Analyzer 对text类型字段做分析

不同的 Analyzer 会有不同的分词结果，内置的分词器有以下几种，基本上内置的 Analyzer 包括 Language Analyzers 在内，对中文的分词都不够友好，中文分词需要安装其它 Analyzer

| 分析器                                                       | 描述                                                         | 分词对象                                                 | 结果                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------ |
| standard                                                     | 标准分析器是默认的分析器,如果没有指定,则使用该分析器。它提供了基于文法的标记化(基于 Unicode 文本分割算法,如 Unicode 标准附件 # 29 所规定),并且对大多数语言都有效。 | The 2 QUICK Brown-Foxes jumped over the lazy dog's bone. | [ the, 2, quick, brown, foxes, jumped, over, the, lazy, dog's, bone ] |
| simple                                                       | 简单分析器将文本分解为任何非字母字符的标记,如数字、空格、连字符和撇号、放弃非字母字符,并将大写字母更改为小写字母。 | The 2 QUICK Brown-Foxes jumped over the lazy dog's bone. | [ the, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone ] |
| whitespace                                                   | 空格分析器在遇到空白字符时将文本分解为术语                   | The 2 QUICK Brown-Foxes jumped over the lazy dog's bone. | [ The, 2, QUICK, Brown-Foxes, jumped, over, the, lazy, dog's, bone.] |
| stop                                                         | 停止分析器与简单分析器相同,但增加了删除停止字的支持。默认使用的是 _english_ 停止词。 | The 2 QUICK Brown-Foxes jumped over the lazy dog's bone. | [ quick, brown, foxes, jumped, over, lazy, dog, s, bone ]    |
| keyword                                                      | 不分词,把整个字段当做一个整体返回                            | The 2 QUICK Brown-Foxes jumped over the lazy dog's bone. | [The 2 QUICK Brown-Foxes jumped over the lazy dog's bone.]   |
| pattern                                                      | 模式分析器使用正则表达式将文本拆分为术语。正则表达式应该匹配令牌分隔符,而不是令牌本身。正则表达式默认为 w+ (或所有非单词字符)。 | The 2 QUICK Brown-Foxes jumped over the lazy dog's bone. | [ the, 2, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone ] |
| 多种西语系 arabic, armenian, basque, bengali, brazilian, bulgarian, catalan, cjk, czech, danish, dutch, english等等 | 一组旨在分析特定语言文本的分析程序。                         |                                                          |                                                              |

# IK中文分词器

# IKAnalyzer

IKAnalyzer是一个开源的，基于java的语言开发的轻量级的中文分词工具包

从2006年12月推出1.0版开始，IKAnalyzer已经推出了3个大版本，在2012版本中，IK实现了简单的分词歧义排除算法，标志着IK分词器从单纯的词典分词向模拟语义分词衍化

# 中文分词器算法

中文分词器最简单的是ik分词器，还有jieba分词，哈工大分词器等

| 分词器      | 描述                                           | 分词对象              | 结果                                                         |
| ----------- | ---------------------------------------------- | --------------------- | ------------------------------------------------------------ |
| ik_smart    | ik分词器中的简单分词器,支持自定义字典,远程字典 | 学如逆水行舟,不进则退 | [学如逆水行舟,不进则退]                                      |
| ik_max_word | ik_分词器的全量分词器,支持自定义字典,远程字典  | 学如逆水行舟,不进则退 | [学如逆水行舟,学如逆水,逆水行舟,逆水,行舟,不进则退,不进,则,退] |

# ik_smart

# 原始内容

传智教育的教学质量是杠杠的

# 测试分词

```
GET _analyze
{
    "analyzer": "ik_smart",
    "text": "传智教育的教学质量是杠杠的"
}
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/9ef4c245d241d655f6b285a91a1cc675e222b380d1fc9ba0ed2a35375cb18701.jpg)

ik_max_word

原始内容

传智教育的教学质量是杠杠的

测试分词

```
GET _analyze
{
    "analyzer": "ik_max_word",
    "text": "传智教育的教学质量是杠杠的"
}
```

![image](https://cdn-mineru.openxlab.org.cn/result/2026-05-09/4113b092-9727-437f-90c8-f833abb146a2/62b21afd49c50fe91198baaa7c2f31360985a11bab96547c67fa154af083c296.jpg)