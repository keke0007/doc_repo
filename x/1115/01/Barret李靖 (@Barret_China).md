---
author: "Barret李靖"
handle: "@Barret_China"
source: "https://x.com/Barret_China/status/2121949262942720331"
date: 2026-11-15T12:51:42.000Z
type: tweet
likes: 321
reposts: 51
replies: 50
bookmarks: 419
views: 30447
---
# Barret李靖 (@Barret_China)

近期在研究 DeepSeek Harness，也研究了一些能够让 Agent 在自主性和可调试性上获得更大提升的插件。挑了几个觉得还不错的，列出来，大家也可以试一试。  
  
1）dsh-market：DSH 的插件市场，支持插件搜索、安装和更新。建议优先安装，后续探索插件会方便很多。  
  
2）dsh-context：个人比较推荐。可以直观地看到 Context 的变化、Token 消耗和多 Agent 的运行状态。对于分析长任务中的决策偏差很有帮助，Agent Debug 正在从 Function Profiling 走向 Decision Profiling。  
  
3）dsh-ops：优化 Shell、文件操作和工具调用，减少 Token 消耗与执行开销。Agent 的性能优化，很多时候还需要从工具接口和执行路径入手。  
  
4）dsh-agent-teams：支持多 Agent 协作、任务依赖和状态持久化。比较值得研究的是任务调度与异常接管机制，多个 Agent 如何高效协作，仍然是个很有挑战的问题。  
  
5）Hindsight：提供 Retain、Recall、Reflect 三类能力，让 Agent 能够积累和复用历史经验。长期记忆的价值在于让过去的经验影响未来的决策。另外，billion-context 也有相似的上下文延续能力，只不过它更侧重超长会话的分层压缩与历史信息找回，适合长程任务。  
  
6）Ouroboros：通过 MCP 接入 DSH，围绕目标、执行和评测持续迭代。比较有意思的是，它开始尝试让 Agent 根据反馈调整自己的执行方式，向更深层次的 RSI 探索。  
  
这几个项目从执行调试、效率优化、多 Agent 协作，到长程任务、长期记忆和自我改进，覆盖了 Agent 工程中几个比较重要的方向。  
  
不过，我觉得目前还缺少一个比较成熟的任务管理插件，能够统一承接日常 TodoList、聊天和其他渠道产生的任务，管理优先级，并根据任务的复杂度、风险和 AI 的执行能力进行分级处理。简单任务自主完成，复杂任务持续推进，涉及关键决策和敏感操作时再交给人。  
  
最近我也在写一个插件，尝试解决这块的问题。希望把任务管理、主动调度、分级自主执行和人工接管整合起来，让 Agent 能够真正承担更多日常工作。  
  
有其他好用的插件，也欢迎推荐。👍