---
author: "kayce"
handle: "@fttt007"
source: "https://x.com/fttt007/status/2105561984536089047"
date: 2026-10-01T07:34:31.000Z
type: tweet
likes: 225
reposts: 37
replies: 11
bookmarks: 249
views: 16497
---
# kayce (@fttt007)

理解 Pi 的 Session 机制后，我对 Agent 的运行方式有了一个新的认识。  
  
很多人理解 Session，只想到“聊天记录”。  
  
但 Pi 的 Session 更像一个 Agent 的运行事件日志（event log）。  
  
它不是简单保存用户和 AI 的对话，而是记录一次 Agent 运行过程中发生的各种事件：  
  
用户消息  
  
Assistant 回复  
模型切换  
Tool Call  
Tool Result  
Token 使用情况  
Context Compaction  
Extension 状态  
Branch 分支  
底层使用 JSONL 保存：  
  
一个 Session 对应一个 JSONL 文件。  
  
文件中的每一行都是一个 entry，通过 id 和 parentId 连接成一棵历史树，而不是简单的一条消息列表。  
  
一次 Agent 运行过程中：  
  
用户输入后，模型可能产生多个 content block：  
  
thinking  
text  
toolCall  
  
其中 toolCall 表示模型决定执行某个动作。  
工具执行完成后产生 toolResult。  
然后模型继续基于新的上下文生成下一步结果。  
  
所以一个 Agent 任务可能包含多次模型调用，每一次调用都有自己的 assistant message。  
  
而 Branch 机制让 Session 不再是一条线。  
  
当你回到历史某个节点重新探索时，Pi 不需要创建新的 Session，而是在同一个 Session 中产生新的分支。  
  
这和 Git 的 commit 历史很像：  
  
保留过去的路径，同时允许从某个节点发展新的方向。  
  
但需要注意：  
  
Session 保存的是 Agent 的状态，不是整个项目环境。  
  
它可以恢复：  
  
✅ 对话历史  
✅ Agent 运行状态  
✅ 工具调用记录  
✅ 模型状态变化  
✅ Context 压缩记录  
  
但不能保证恢复：  
  
❌ 项目文件  
❌ 代码版本  
❌ 运行环境  
  
所以一个可靠的 Coding Agent 系统，需要 Session、Git、项目环境和 Sandbox 等多个部分共同配合。  
理解 Session，其实是在理解 Agent 如何实现：  
可恢复（durable）  
可分支（branchable）  
可持续运行（long-running）  
这也是未来 Agent 系统区别于普通 Chatbot 的重要基础。  
#AI #Agent #LLM

![Image](<kayce%20(@fttt007).assets/image-01.jpg>)