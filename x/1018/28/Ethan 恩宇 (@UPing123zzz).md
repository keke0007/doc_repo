---
author: "Ethan 恩宇"
handle: "@UPing123zzz"
source: "https://x.com/UPing123zzz/status/2086483086473109953"
date: 2026-08-09T16:01:47.000Z
type: tweet
likes: 518
reposts: 87
replies: 51
bookmarks: 662
views: 53406
---
# Ethan 恩宇 (@UPing123zzz)

我最近把 RAG 从头到尾重新研究了一遍，才发现很多人对 RAG 的理解其实还停留在：  
  
把 PDF 切成 Chunk，Embedding 一下，丢进向量数据库，然后接个大模型。  
这确实能做出 Demo，但离真正的企业知识库还差得很远。  
  
RAG 的原理其实特别简单：用户问一个问题，系统先从企业内部找到相关资料，再把资料交给 LLM 回答。所以 RAG 最重要的可能根本不是最后那个 LLM，而是前面的 Retrieval。如果检索出来的东西就是错的，模型越强，只会把错误答案说得越像真的。  
  
真正给企业做 RAG，我现在会把它理解成两条流水线。  
  
第一条是知识入库：  
  
PDF / Word / 网页 → 文档解析 → OCR/表格/结构恢复 → Chunk → Metadata → Embedding → 索引。  
  
第二条是用户提问：  
  
Question → Query 理解 → 权限过滤 → Hybrid Search → Rerank → Context → LLM → Answer + 引用来源。  
  
这里面最容易被低估的是第一条。比如我们前面聊的 PDF，人看到的是章节、表格和上下文，但机器拿到的可能只是坐标和文字。如果这里解析错了，后面的 Embedding、向量数据库、Rerank 全部建立在错误数据上。现在一些 Document AI 工具已经开始把 PDF 恢复成包含章节、表格、图片、页眉页脚和来源信息的结构化文档，而不是单纯抽文本。  
  
第二个容易踩的坑是：不要迷信纯向量搜索。  
  
企业知识里有大量 SKU、产品型号、合同编号、人名和数字。语义搜索很强，但精确关键词同样重要。所以真正成熟一点的 RAG，通常会开始做 Dense + Keyword/Sparse 的 Hybrid Search，再把召回结果交给 Reranker 排一次。  
  
但企业 RAG 真正复杂的地方还不是这些，而是：旧文档怎么办？新旧政策冲突怎么办？不同部门权限怎么办？知识什么时候更新？回答错了怎么发现？  
  
所以我现在越来越觉得，企业 RAG 首先是知识工程，然后才是大模型工程。  
  
而 AI Coding 又把这件事情改变了一次。  
  
以前做这套东西，你还要花大量时间写 Parser、API、Vector DB、后台、Docker。现在这些工程代码完全可以大量交给 Codex、Claude Code 之类的 Coding Agent 去实现。  
  
于是人的价值反而往上移动了。  
  
你真正需要知道的是：为什么这个 PDF 要结构化解析？为什么这里要 Hybrid Search？为什么这里需要 Rerank？为什么这个问题应该查知识库，而另一个问题应该直接查 SQL？  
  
AI 可以帮你写 RAG，但你必须知道什么才是一套正确的 RAG。  
  
所以如果今天让我重新理解企业 AI，我不会再把 RAG 看成一个“知识库聊天机器人”。  
  
我会把它看成一层新的企业基础设施：  
  
把散落在 PDF、Word、网页、数据库和员工脑子里的知识，变成 AI 可以检索、理解、引用，并且有权限地使用的知识层。  
  
等这一层真正建立起来，上面接的就不只是 Chatbot 了。  
  
还可以是客服 Agent、销售 Agent、内部 Copilot、内容 Agent、决策 Agent……  
  
RAG 真正有价值的地方，从来不是“让 AI 可以读 PDF”。  
  
而是：  
  
让一个企业第一次拥有一层，可以被 AI 调用的知识系统。  
  
这可能才是企业 RAG 真正值得做的地方。