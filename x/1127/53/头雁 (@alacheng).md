---
author: "头雁"
handle: "@alacheng"
source: "https://x.com/alacheng/status/2125506021682491684"
date: 2026-11-25T08:25:00.000Z
type: thread
likes: 55
reposts: 7
replies: 10
bookmarks: 45
views: 4675
---
# 头雁 (@alacheng)

在湖仓里同时做好向量搜索 + 全文搜索 + 多模态存储 + 超快随机访问  
  
研究epocha ai时发现，他们的数据湖仓使用的是Lance  
  
Lance是一个面向多模态 AI 的开源湖仓（lakehouse）格式。 它的核心定位是把高性能向量搜索、全文搜索、特征工程和模型训练能力带到湖仓里，同时保留传统湖仓的 SQL 分析、ACID 事务、时间旅行等能力。  
  
Lance 的核心卖点是「在湖仓里同时做好向量搜索 + 全文搜索 + 多模态存储 + 超快随机访问」，特别适合做 AI 特征库、多模态数据湖、混合检索应用等场景。

---

1）混合搜索能力强（Hybrid Search）  
同一份数据上可以同时做：  
-向量相似度搜索  
-全文搜索（BM25）  
-SQL 过滤/分析 这些查询都有二级索引加速，属于格式规范的一部分。  
2）随机访问极快  
  
官方宣称比 Parquet / Iceberg 快约 100 倍的随机访问性能。适合 ML 推理、采样、交互式应用等需要频繁按行/按 ID 读取的场景，同时不影响扫描性能  
3）原生支持多模态数据  
  
可以把图像、视频、音频、文本、Embedding 和普通表格列存在同一个表里。大二进制对象用 Blob 编码 + 懒加载，向量存储也做了专门优化。  
4）生态集成丰富  
  
-处理/ML：Pandas、Polars、Ray、PyTorch  
-SQL 分析/分布式：DataFusion、DuckDB、Spark、Trino、Flink  
-Catalog：Apache Polaris、Unity Catalog、Gravitino、Hive Metastore 等  
[lance.org](https://lance.org)