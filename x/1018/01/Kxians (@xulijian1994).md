---
author: "Kxians"
handle: "@xulijian1994"
source: "https://x.com/xulijian1994/status/2111754647245607112"
date: 2026-10-18T09:41:57.000Z
type: tweet
likes: 1
replies: 3
bookmarks: 2
views: 333
---
# Kxians (@xulijian1994)

最近看了不少关于 Agent 开发的技术文章，发现一个挺有意思的现象：很多人还在研究 Prompt 怎么写，但真正决定 Agent 能力上限的，已经不只是提示词了。  
  
如果你准备系统学习大模型应用开发，我建议重点搞懂三个核心概念：Agent Loop、Context Engineering、Harness Engineering。  
  
这三个东西基本构成了现代 Agent 的技术骨架。  
  
1. Agent Loop：让模型真正执行任务  
  
Agent 的核心并不复杂，本质就是一个循环：  
  
用户输入 → LLM 推理 → Tool Call → 工具执行 → 结果反馈 → LLM 继续推理。  
  
直到任务完成，或者触发终止条件。  
  
理解这个循环，比上来就研究 LangChain、LangGraph 的各种封装更重要。  
  
建议自己实现一个最小 Agent Loop，支持工具调用、错误处理、最大迭代次数和任务终止。  
  
2. Context Engineering：决定模型每一步能看到什么  
  
Agent 能不能完成任务，很大程度取决于每次调用模型时，提供了什么上下文。  
  
常见的上下文包括：  
  
System Prompt、用户输入、会话历史、工具定义、外部知识、执行结果和长期记忆。  
  
真正麻烦的是长任务。  
  
随着执行次数增加，上下文会不断膨胀，出现信息污染、目标漂移、历史冲突等问题。  
  
所以需要掌握几个关键技术：  
  
• Context Compaction：上下文压缩  
• Progressive Disclosure：渐进式加载  
• Memory Management：长期记忆管理  
• Context Retrieval：动态检索  
• Context Orchestration：上下文编排  
  
尤其是渐进式加载，不要一开始就把所有知识塞给模型，而是让 Agent 在需要的时候主动读取。  
  
3. Harness Engineering：让 Agent 稳定运行  
  
这是我认为非常值得深入研究的方向。  
  
模型本身只负责推理，Harness 则负责给模型创造一个可以持续工作的运行环境。  
  
包括工具系统、执行环境、权限控制、任务调度、状态管理、错误恢复、反馈机制等。  
  
以 Coding Agent 为例，Read、Write、Edit、Glob、Grep、Bash 这些基础工具，就能组合出相当强大的能力。  
  
特别是 Bash，能力非常强，但也带来了命令执行安全、超时控制、输出截断和环境隔离等问题。  
  
这些才是真正需要工程能力解决的地方。  
  
4. RAG 不再是唯一的知识获取方式  
  
以前做知识库，很多人第一反应就是 Embedding + Vector Database + RAG。  
  
但现在的 Coding Agent 已经证明，基于文件系统和搜索工具的动态检索同样有效。  
  
比如通过 Glob 定位文件、Grep 搜索内容、Read 按需读取。  
  
当然，这不代表 RAG 过时了。  
  
结构化知识、语义检索、大规模文档库仍然有 RAG 的优势。  
  
关键是根据任务选择检索方式，而不是为了使用向量数据库而使用向量数据库。  
  
5. 多 Agent 不一定比单 Agent 更好  
  
我现在更倾向于先把单 Agent 做好，再根据需要引入 SubAgent。  
  
多 Agent 虽然可以拆分任务，但同时会引入上下文同步、任务交接、状态一致性和协作成本。  
  
很多场景，一个主 Agent 加少量专用 SubAgent 就够用了。  
  
不要为了架构复杂而复杂。  
  
6. Agent Evals：最容易被忽略的一环  
  
这可能是整套技术体系里最值得重视的部分。  
  
很多 Agent 项目都是先开发，觉得效果不好就改 Prompt，改完再人工试几次。  
  
这种方式很难持续优化。  
  
更合理的做法是先构建评估数据集，再开发 Agent。  
  
至少关注几个维度：  
  
• Task Success Rate：任务成功率  
• Tool Call Accuracy：工具调用准确性  
• Token Cost：执行成本  
• Latency：任务耗时  
• Recovery Rate：异常恢复能力  
• Trajectory：执行轨迹是否合理  
  
每次调整 Prompt、Context 或工具系统，都通过相同的数据集进行回归评估。  
  
这样才能知道一次优化到底是真的提升了效果，还是只改善了几个特定案例。  
  
7. 学习资源推荐  
  
如果想系统学习大模型应用开发，下面这些资源值得收藏。  
  
① 通往 AGI 之路（飞书知识库）  
[waytoagi.feishu.cn/wiki/QPe5w5g7U…](https://waytoagi.feishu.cn/wiki/QPe5w5g7UisbEkkow8XcDmOpn8e)  
  
② Anthropic Engineering（工程实践）  
[anthropic.com/engineering](https://anthropic.com/engineering)  
  
③ Claude Blog（Claude 团队博客）  
[claude.com/blog](https://claude.com/blog)  
  
④ Anthropic Research（前沿研究与论文）  
[anthropic.com/research](https://anthropic.com/research)  
  
⑤ Cursor Blog（Coding Agent 工程实践）  
[cursor.com/cn/blog](https://cursor.com/cn/blog)  
  
⑥ Lilian Weng Blog（LLM、Agent、强化学习等技术原理）  
[lilianweng.github.io](https://lilianweng.github.io)  
  
⑦ Claude Code Docs（工具、Hooks、Skills、Agent 等）  
[code.claude.com/docs/en/overvi…](https://code.claude.com/docs/en/overview)  
  
⑧ Lenny's Podcast（AI 产品、工程与行业访谈）  
[youtube.com/@LennysPodcast](https://youtube.com/@LennysPodcast)  
  
⑨ Pi（Agent Toolkit，适合研究 Agent Loop 和底层实现）  
  
源码：  
[github.com/earendil-works…](https://github.com/earendil-works/pi)  
  
文档：  
[pi.dev/docs/latest](https://pi.dev/docs/latest)  
  
最后说一下我的理解。  
  
大模型应用开发正在从 Prompt Engineering，逐渐走向 Context Engineering 和 Harness Engineering。  
  
未来真正有竞争力的 Agent 开发者，不一定是最会写提示词的人，而是能够把模型能力、上下文、工具系统和评估体系组合起来的人。  
  
与其不停追逐新的 Agent 框架，不如先把这些底层原理真正吃透。