---
author: "Xudong Han"
handle: "@Xudong07452910"
source: "https://x.com/Xudong07452910/status/2113800139005460534"
date: 2026-10-24T01:10:00.000Z
type: tweet
likes: 2179
reposts: 325
replies: 92
bookmarks: 3171
views: 489710
---
# Xudong Han (@Xudong07452910)

Anthropic 最近开始建议企业「少用点 Subagent」，甚至把一些专家 Agent 重新合并回主 Agent Loop。  
  
他们在多个企业项目中发现，单 Agent + Skills 的效果经常比复杂的多 Agent 架构更好。有个客户改成单 Loop 后，运行速度甚至快了约 2 倍。  
  
原因很容易理解。  
  
多个 Agent 之间反复交接任务，每次都要传递和总结 Context，信息容易丢失，延迟和成本也跟着增加。  
  
Anthropic 现在更推荐让一个主 Agent 负责规划和执行，需要时再动态加载 Skills 和 Tools，把业务流程和领域知识留在可复用的组件里。  
  
不过文章里有个细节值得关注。  
  
他们曾把客服系统里的 Subagent 合并，延迟确实降低了，却让一个原本能完成的任务失败了，因为主 Agent 漏掉了向物流专家确认送达时间这一步。  
  
所以作者强调，每合并一个 Subagent，都要用端到端 Eval 验证，尤其要检查哪些原本通过的任务突然失败了。  
  
当然，并行研究、独立验证、Context 隔离这些场景，Subagent 依然有价值。  
  
这样看来以后设计 Multi-Agent，可能得先问问自己，每个 Subagent 真的需要独立的 Context 和执行循环吗？  
  
Anthropic 还开源了一套迁移工具，可以帮你分析现有架构，逐步合并 Subagent，并比较改动前后的质量、成本和延迟。很推荐做 Agent 工程的朋友试试。  
  
原文：

> ![Link card preview](<Xudong%20Han%20(@Xudong07452910).assets/image-01.png>)
> 
> 🔗 [**Building agents: When to fold subagents into your main agent loop**](https://t.co/0CPabUlmF7)
> 
> _From claude.com_