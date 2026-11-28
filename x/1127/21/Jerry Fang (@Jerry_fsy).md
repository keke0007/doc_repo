---
author: "Jerry Fang"
handle: "@Jerry_fsy"
source: "https://x.com/Jerry_fsy/status/2126259360150258071"
date: 2026-11-27T10:18:30.000Z
type: tweet
likes: 91
reposts: 8
replies: 13
bookmarks: 86
views: 5296
---
# Jerry Fang (@Jerry_fsy)

在Agent和Agent Infra中，会涉及到很多种存储方式，包括对象存储，向量数据库，关系型数据库，还有ClickHouse这种。我们往往需要结合特定的场景选择合适的存储方式。  
  
因此，我在这里汇总了一下，Agent运行中常常涉及到的存储方式，以及各种存储方式适用于什么样的场景：  
  
一、 核心状态与业务层（Agent 的“账本”与“存档点”）  
1关系型数据库（RDBMS / 分布式 SQL）  
◦ 代表：  
PostgreSQL、MySQL、TiDB  
◦ 存什么：  
用户/租户权限、Agent 配置、会话元数据、计费账单，以及 Durable Execution（持久化执行）的状态机 Checkpoint。  
◦ 为什么不可替代：  
Agent 执行一个复杂任务可能耗时几分钟甚至几天，中途崩溃需要断点续传，且涉及工具调用（如扣费、发邮件）时必须有严格的 ACID 事务保证。  
  
2内存型 KV 存储（In-Memory KV / Cache）  
◦ 代表：  
Redis  
◦ 存什么：  
活跃会话的短期记忆（Short-Term Memory）、流式输出（SSE）的临时状态、分布式锁、限流计数，以及 语义缓存（Semantic Cache）（相似问题直接命中缓存，省去调 LLM 的开销）。  
◦ 核心特征：  
亚毫秒级延迟，适合高频读写、允许过期（TTL）的瞬态数据。  
  
二、 知识与记忆层（Agent 的“大脑”与“图书馆”）  
3向量数据库（Vector DB）  
◦ 代表：  
Qdrant、Milvus、Pinecone、Weaviate、pgvector  
◦ 存什么：  
文档切片和长期记忆的 Embedding 向量。  
◦ 解决什么问题：  
支撑 RAG 和语义记忆（Semantic Memory），通过近似最近邻（ANN）算法实现“按意思搜”而非“按字面搜”。  
  
4全文检索引擎（Full-Text Search）  
◦ 代表：  
Elasticsearch、OpenSearch、Meilisearch  
◦ 存什么：  
倒排索引（BM25）。  
◦ 为什么需要它：  
纯向量检索对精确专有名词、错误码、SKU 编号、代码符号表现比较差。生产级 RAG 几乎都采用 向量 + 全文的混合检索（Hybrid Search）。  
  
5图数据库（Graph DB）  
◦ 代表：  
Neo4j  
◦ 存什么：  
实体（人、项目、概念）及其关联关系。  
◦ 解决什么问题：  
支撑 GraphRAG 和 情景/关系记忆（Episodic Memory）。当 Agent 需要跨多份文档做跳转推理，或追踪“用户偏好随时间的变化”时，图结构比扁平的向量切片更有效。  
  
三、 文件、产物与执行环境层（Agent 的“工作台”与“仓库”）  
6对象存储（Object Storage）  
◦ 代表：  
AWS S3、MinIO、SeaweedFS  
◦ 存什么：  
大体积非结构化数据——用户上传的 PDF/音视频、Agent 生成的报告/图表/代码包、网页抓取的 HTML 快照，以及可观测性日志中的超大 Prompt/Response Payload。  
◦ 核心特征：  
成本极低、容易扩展、无 Schema 约束，但没有查询能力，通常需要配合关系型数据库存元数据索引。  
  
7沙箱文件系统与块存储（Sandbox FS / Block Storage）  
◦ 代表：  
OverlayFS、Copy-on-Write (CoW) 快照、FUSE、EBS/NFS（如 E2B、Modal 等沙箱底层）  
◦ 存什么：  
Coding Agent 或 Computer Use Agent 在隔离容器/微虚拟机里运行代码时的工作目录、依赖包和临时文件。  
◦ 核心特征：  
需要标准的 POSIX 文件系统语义，以及毫秒级克隆、快照与回滚能力，方便 Agent 试错后重置环境。  
  
四、 可观测性、评估与异步调度层（Agent Infra 的“黑匣子”与“传送带”）  
8分析型数据库（OLAP）  
◦ 代表：  
ClickHouse  
◦ 存什么：  
海量的 Trace、Span、LLM 调用耗时、Token 消耗、成本统计以及 Eval（离线/在线评估）打分结果（例如 Langfuse 的核心存储层）。  
◦ 为什么用它：  
Agent 一次请求可能触发几十次工具调用和模型推理，日志呈指数级增长。OLAP 擅长超高吞吐追加写入，以及秒级的多维聚合分析。  
  
9消息队列与事件日志（Message Queue / Event Log）  
◦ 代表：  
Kafka、NATS、Redis Streams  
◦ 存什么：  
异步任务指令、Agent 运行事件流、待落盘的 Trace 数据。  
◦ 解决什么问题：  
虽然常被归为中间件，但在 Agent Infra 中它承担了持久化事件日志的角色——既用于异步 Run API 的任务解耦和削峰填谷，也用于把高频产生的可观测性数据平稳写入 ClickHouse 和对象存储。  
  
10时序数据库（TSDB）  
◦ 代表：  
Prometheus、VictoriaMetrics  
◦ 存什么：  
偏底层 Infra 的监控指标（GPU 利用率、推理引擎队列长度、沙箱 CPU/内存水位、API P99 延迟）。  
  
五、 推理加速层（最容易被忽略的“隐形存储”）  
11KV Cache 外部存储 / 分级缓存  
◦ 代表：  
LMCache、Mooncake（基于 DRAM / NVMe SSD / RDMA 构建的分布式缓存池）  
◦ 存什么：  
LLM 推理过程中计算好的 Attention KV Cache。  
◦ 为什么重要：  
Agent 的特点是 System Prompt 极长、多轮对话前缀高度重复、Multi-Agent 共享上下文。把 KV Cache 卸载到外部存储并跨推理节点复用，能把首字延迟（TTFT）降低一个数量级，同时大幅节省 GPU 算力。