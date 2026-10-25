---
author: "灰狐"
handle: "@huihoo"
source: "https://x.com/huihoo/status/2113960750771479000"
date: 2026-10-24T11:48:13.000Z
type: tweet
likes: 51
reposts: 6
replies: 1
bookmarks: 54
views: 3074
---
# 灰狐 (@huihoo)

PostgreSQL 正在吞噬整个数据库世界  
坚实底座 + 万物皆可插件  
PostgreSQL 坚实数据库底座 + 插件生态（Extensibility）= 多模态数据平台  
Rust 语言加持，随着 pgrx（Build Postgres Extensions with Rust）成熟  
进一步加速生态繁荣  
[github.com/pgcentralfound…](https://github.com/pgcentralfoundation/pgrx)  
PG 采用类似 MIT 的 PostgreSQL License，允许任何人闭源商业化  
  
以前企业架构：MySQL（业务）+ Elasticsearch（搜索）+ Redis（缓存）+ InfluxDB（时序）+ Milvus（向量）  
现在企业架构：一套 PostgreSQL + 对应插件，这极大地降低了数据同步延迟、运维成本和团队学习曲线  
  
支撑多模态的三个底层关键：  
可编程的索引架构（可插拔索引）  
PG 不仅有 B-Tree 索引  
它提供了 GIN（倒排索引，干掉 ES 的核心）、GIST/SP-GIST（空间索引，PostGIS 的基石）、BRIN（块区间索引，时序海量数据的克星）以及 HNSW（pgvector 向量索引）  
开发者不需要改动 PG 内核，就能自己实现一种全新的数据结构索引  
  
强大的外表扩展（FDW - Foreign Data Wrappers）  
PG 的 FDW 允许它直接把外部的 Redis、MySQL、ClickHouse 甚至 S3 上的 Parquet 文件“映射”成 PG 里的一张表  
哪怕数据不在 PG 内部，你也能用一条标准的 SQL 实现跨库、跨文件的联邦查询（Federated Query）  
  
高阶数据类型与内置函数支持  
在处理半结构化数据时，JSONB 的读写性能和检索效率在很多场景下甚至超越了专门的文档数据库 MongoDB  
这让业务在面对频繁变动的字段时，拥有了极高的灵活性

![Image](<灰狐%20(@huihoo).assets/image-01.jpg>)