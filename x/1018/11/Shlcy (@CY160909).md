---
author: "Shlcy"
handle: "@CY160909"
source: "https://x.com/CY160909/status/2111240271190982890"
date: 2026-10-16T23:38:00.000Z
type: tweet
---
# Shlcy (@CY160909)

精心整理了 7 个让 AI Agent 拥有长期记忆的开源项目。  
  
用 Codex、Claude Code，最麻烦的就是：换个会话，又得重新交代项目背景、之前踩过的坑和已经做过的决定。  
  
这 7 个项目都在解决 Agent 的记忆问题，但实现思路各不相同。  
  
分享给大家👇  
  
① memU｜跨 Agent 共享记忆  
  
将不同 Agent 的历史信息整理成共享 Wiki，让 Codex、Claude Code、Cursor 等工具在配置后复用相关记忆，还能从历史任务中提炼可复用的 Skill。  
  
[github.com/NevaMind-AI/me…](https://github.com/NevaMind-AI/memU)  
  
② Hindsight｜让 Agent 从经历中学习  
  
提供 Retain、Recall、Reflect 三种核心操作。不仅能保存和检索过去的信息，还能分析多次任务经历，形成新的观察和经验。  
  
[github.com/vectorize-io/h…](https://github.com/vectorize-io/hindsight)  
  
③ Graphiti｜记住信息是怎么变化的  
  
通过时间知识图谱管理记忆。  
  
比如项目最初决定用方案 A，后来改成 B，它能记录事实和关系随时间发生的变化，适合长期迭代的项目。  
  
[github.com/getzep/graphiti](https://github.com/getzep/graphiti)  
  
④ Letta｜有持续状态的 AI Agent  
  
更像一个完整的 Agent 运行平台。Agent 能维护自己的记忆、身份和状态，在不同会话中延续工作。  
  
[github.com/letta-ai/letta…](https://github.com/letta-ai/letta-code)  
  
⑤ Mem0｜给 AI 应用添加记忆层  
  
可以从对话中提取用户偏好、重要事实和历史信息。  
  
适合给自己开发的 AI 助手、客服机器人增加跨会话记忆。  
  
[github.com/mem0ai/mem0](https://github.com/mem0ai/mem0)  
  
⑥ Cognee｜把资料变成关联记忆  
  
将文档、代码和对话组织成可检索的知识图谱，让 Agent 不只能找到信息，还能利用其中的关系。  
  
支持本地部署。  
  
[github.com/topoteretes/co…](https://github.com/topoteretes/cognee)  
  
⑦ LangMem｜让 Agent 持续更新记忆  
  
LangChain 推出的开源工具。  
  
能够从历史交互中提取、整理和更新长期记忆，也能利用反馈优化 Agent 的提示词与行为，特别适合 LangGraph 开发者。  
  
[github.com/langchain-ai/l…](https://github.com/langchain-ai/langmem)

> ![Link card preview](<Shlcy%20(@CY160909).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - NevaMind-AI/memU: Personal memory across agents**](https://t.co/vUJka040jV)
> 
> _From github.com_