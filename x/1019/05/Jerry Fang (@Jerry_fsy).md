---
author: "Jerry Fang"
handle: "@Jerry_fsy"
source: "https://x.com/Jerry_fsy/status/2112149107863392401"
date: 2026-10-19T11:49:23.000Z
type: tweet
likes: 8
reposts: 3
replies: 2
bookmarks: 12
views: 459
---
# Jerry Fang (@Jerry_fsy)

如果你正在做 Agent，或者准备 Agent 相关岗位的面试，RAG 和记忆系统都是绕不开的知识点。很多人觉得 RAG 已经落后了，没有人使用了，但实际上，它是一种上下文管理策略，在很多现实的业务中，会将Agent与RAG结合，从而满足业务的需求。同时，在Agent的使用中，我们还需要考虑记忆的存储、检索和更新。  
  
对于RAG来说，它不单单是"向量库 + 检索"，在真正落地过程中，问题往往出在切块、召回、排序和记忆管理上。  
  
我最近梳理了一下这块的知识，整理了一份 RAG 与 Agent 记忆系统的学习路线，从"检索"到"记忆"，欢迎收藏，保存，我也会继续分享在这个领域的学习心得。  
  
完整路线：  
1.RAG 基础：理解“文档加载→分块→embedding→向量存储→检索→注入上下文”的完整流程。理论部分可以先看 2020 年 Lewis 等人那篇 RAG 原始论文（[arxiv.org/abs/2005.11401](https://arxiv.org/abs/2005.11401)），重点带着下面三个问题来看这篇论文：  
•Introduction：为什么需要 RAG？传统参数化语言模型有哪些局限？  
•Methods：RAG 如何将 Retriever 和 Generator 结合起来？  
•Experiments：如何验证检索增强确实改善了知识密集型任务的效果？  
  
2.文档检索：理解什么样的文档格式适合RAG，理解为什么代码检索不适合使用RAG。  
  
3.分块：知道为什么需要分块，以及几种主流的分块策略（包括固定大小分块，递归分块，语义分块等）  
推荐看一下一篇分块相关的文章：[premai.io/blog/rag-chunk…](https://premai.io/blog/rag-chunking-strategies-the-2026-benchmark-guide/)  
  
4.Embedding 与向量检索：理解 Embedding 模型、余弦相似度，以及 ANN 近似检索（比如 HNSW 索引）。  
  
5.存储（向量数据库的选型）：了解 sqlite-vec、Milvus、pgvector 这类工具的区别。知道在当前场景下，哪一款向量数据库最合适。  
  
6.混合检索与 Rerank：理解“语义相似不等于任务相关“。向量检索 + BM25 关键词检索结合，再用 Reranker（Cross-Encoder）重新排序。要时刻记住：检索不是"搜到就行"，而是"搜到对的"。  
  
7.索引更新：理解“全量重建”和“增量更新”两种策略，知道每一种策略适用于什么样的场景。  
  
8.短期记忆：也就是 Context 管理，包括对话历史裁剪、摘要压缩，以及在上下文窗口有限的情况下保留关键信息。  
  
9.长期记忆：跨 Session 保存用户偏好、事实和经验。  
  
10.记忆的写入、更新与遗忘：什么时候该写入记忆，怎么处理冲突和过时的信息。可以看[arxiv.org/abs/2304.03442](https://arxiv.org/abs/2304.03442)这篇文章。

> ![Link card preview](<Jerry%20Fang%20(@Jerry_fsy).assets/image-01.jpg>)
> 
> 🔗 [**Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks**](https://t.co/iN7Rc3K67m)
> 
> Large pre-trained language models have been shown to store factual knowledge in their parameters, and achieve state-of-the-art results when fine-tuned on downstream NLP tasks. However, their...
> 
> _From arxiv.org_