---
author: "Ethan 恩宇"
handle: "@UPing123zzz"
source: "https://x.com/UPing123zzz/status/2121478665775272307"
date: 2026-11-13T08:06:36.000Z
type: tweet
likes: 4
replies: 2
bookmarks: 5
views: 608
---
# Ethan 恩宇 (@UPing123zzz)

一堆人天天研究 AI Startup Stack，哪个模型更新了就去试，哪个 Agent 框架火了就赶紧装，GitHub 收藏夹里几百个项目，恨不得把所有热门技术全部塞进自己的产品里。  
  
结果真让他从零设计一个能交付的 AI 产品，可能连技术栈之间到底是什么关系都没搞清楚。  
  
这份 AI Startup Stack 整理得挺不错，把 25 个工具分成了五类：LLM Models、AI Frameworks、Vector Databases、AI Infrastructure、AI Observability。  
  
但我觉得真正值得学的，根本不是这 25 个名字，而是它们背后分别解决的工程问题。  
  
LLM 负责理解、推理和生成。  
  
Framework 负责组织模型调用、工具执行、RAG 和任务流程。  
  
Vector Database 负责语义检索，让系统能够找到相关的外部知识。  
  
Infrastructure 负责提供计算资源和模型推理服务。  
  
Observability 则负责告诉你，整个系统到底是怎么运行的，哪里出错了，花了多少钱，效果究竟怎么样。  
  
这几层看起来简单，但真要做企业 AI 落地，很多人就在这里开始暴露问题。  
  
比如做一个企业 RAG。把 PDF 切块、Embedding、扔进向量数据库，再接个大模型，十几分钟可能就能跑通 Demo。  
  
但知识库更新以后怎么同步？不同用户的检索权限怎么隔离？模型回答错了，是检索的问题还是生成的问题？系统上线之后，怎么证明新版本比旧版本更好？  
  
这些问题，根本不是换个更强的 LLM 或者增加几个 Agent 就能解决的。  
  
而且更离谱的是，有些人明明一个 FastAPI、PostgreSQL 加模型 API 就能解决的问题，非要引入 LangChain、LangGraph、多个向量数据库，再部署一整套分布式架构。  
  
最后业务没复杂多少，技术栈倒是复杂得没人维护。  
  
我一直觉得，架构设计不是技术越多越好，而是你到底能不能解释清楚，每个组件为什么存在，它解决了什么问题，以及不使用它会怎样。  
  
真正做 AI Startup，最重要的也不是你用了多少先进工具。  
  
而是有没有真实的用户需求，能不能解决问题，能不能控制成本，系统出了问题能不能恢复，以及客户为什么愿意持续付费。  
  
尤其是现在 AI 编程工具越来越强，开发者可以很快生成代码、搭建前端、接入模型、完成部署。  
  
但 AI 帮你实现了技术方案，不代表这个技术方案本身就是合理的。  
  
所以我越来越觉得，AI 工程师真正应该积累的，是能够跨模型、跨框架、跨平台理解整个系统的能力。  
  
知道什么时候该使用 RAG，什么时候需要 Agent，什么时候一条简单 Workflow 就够了，什么时候应该上向量数据库，什么时候直接用 SQL 查询反而更合理。  
  
说到底，会使用工具当然重要。  
  
但真正拉开差距的，是知道什么时候该用、什么时候不该用，以及最后能不能把东西交付出去。  
  
别最后收藏了 25 个 AI 工具，技术栈比谁都复杂，结果连一个真实客户的问题都没解决。AI 创业不是技术栈收藏比赛。  
  
技术只是手段，产品和真实业务价值才是目的。

> **kritika (@vibeonX69)**
> 
> 📁 AI Startup Stack  
> ┃  
> ┣ 📂 LLM Models  
> ┃ ┣ 🤖 OpenAI  
> ┃ ┣ 🟣 Claude  
> ┃ ┣ 💎 Gemini  
> ┃ ┣ 🦙 Llama  
> ┃ ┗ 🔍 DeepSeek  
> ┃  
> ┣ 📂 AI Frameworks  
> ┃ ┣ 🦜 LangChain  
> ┃ ┣ 🦙 LlamaIndex  
> ┃ ┣ 🕸️ LangGraph  
> ┃ ┣ 🔗 Haystack  
> ┃ ┗ ⚡ DSPy  
> ┃  
> ┣ 📂 Vector Databases  
> ┃ ┣ 🟣 Pinecone  
> ┃