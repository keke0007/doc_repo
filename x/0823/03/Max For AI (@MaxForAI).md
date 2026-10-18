---
author: "Max For AI"
handle: "@MaxForAI"
source: "https://x.com/MaxForAI/status/2091056022551187579"
date: 2026-08-22T06:53:00.000Z
type: thread
likes: 2917
reposts: 383
replies: 141
bookmarks: 3183
views: 409532
---
# Max For AI (@MaxForAI)

爆了，Pi 居然在他们新的开发笔记里把其他家的Agent都锐评了一下🤣  
  
@pidotdev 官方表示，DSH、Claude Code、OpenCode 这一类方案，它关注的已经不是谁多几个 Tool、谁的 Demo 更炫，而是一个更麻烦的问题：  
  
如果一个 Agent 连续跑 50 个小时，它还知道自己到底干过什么吗？  
  
他们先点了 DeepSeek Harness。  
  
Pi 直接从 DSH 的 Context 管理开刀。  
DSH 有一个 tool result pruner，工具输出太长以后，会保留前面一部分和后面一部分，中间直接裁掉。  
  
Pi 那边给的评价很不客气：permanently-lossy pruner。  
  
因为删掉的内容以后找不回来了。  
比如一个 Agent 跑测试吐出 3 万字符日志，真正的错误刚好在第 12000 个字符，压缩完以后这段已经消失。后面模型再聪明，也没法从空气里重新读出来。  
  
所以 Pi 做的版本是 prune + spill。  
  
Context 里依然只放一小段，完整 Tool Result 同时落到磁盘，Context 留文件路径和 offset。模型需要的时候自己 grep、sed、read 回去。  
他们拿 GLM-5.3 和 DeepSeek V4 Flash 跑了 19 个真实 Session，Context 占用降低 26%～35%，uncached prefill 降低 72%～88%，信息还能恢复。  
PR 里甚至直接写strictly better than dsh’s permanently-lossy pruner😂。  
  
然后又点了 Claude Code。  
  
Pi 表示有一个挺有意思的发现：  
新一代 Claude 在第三方 Harness 上，有时候反而更容易把 Tool 调错。  
最典型的情况是，Pi 给模型的 edit tool schema 明明写得很清楚，模型却会自己生成 requireUnique、matchCase、oldText2 这种 Pi 根本没有的参数。  
一个很合理的猜测是，Claude 的后训练越来越适配 Claude Code 自己的 Harness 和 Tool Schema。  
模型已经学会了 Claude Code 里某个工具应该长什么样。  
然后你把它放进 Pi，它脑子里还是那一套。  
所以现在模型能力越强，模型和 Harness 的绑定可能也越深。  
以后评价一个 Coding Model，单独看模型可能越来越没意义。  
Claude + Claude Code、DeepSeek + DSH、GPT + Codex，很可能逐渐变成一个整体。  
  
讲到这里 Pi 表示他们最近重写了 Harness，开始把一次 Agent Run 当成接近数据库事务的东西来设计。  
  
Tool Call 执行之前，先写 intent。  
  
执行结束，再写结果。  
  
进程崩溃以后，系统得知道这个 Tool 到底执行过没有。  
  
Session Tree、运行状态、Lane、Operation Log 分开保存。  
  
一个 Agent 可以有多个 Lane 并行跑，每个 Lane 都知道自己的当前位置、队列和未完成操作。  
  
Context 允许压缩，原始执行历史继续保留。  
  
甚至连进程刚好死在 Tool 执行到一半这种恶心情况，他们都专门设计了 recovery。  
  
你再回头看现在很多 Agent Framework，会发现大家以前真的挺勇的：  
  
一个 while loop，塞一个模型，塞几个 Tool，Context 快满了就总结一下，然后祈祷它一直别挂。  
  
跑十分钟当然没什么。  
  
Agent 开始跑几小时、几天以后，问题全出来了。  
  
这些已经越来越像操作系统和数据库问题了。  
  
有意思的是Pi 甚至对 Extension 的态度也在变。  
  
现在大家都喜欢说「一切皆插件」，DSH 的 Cordis 就走得很远。  
  
Pi 这边最近却越来越强调边界：  
  
哪些是 Conversation，哪些是 Runtime State，哪些是 UI，哪些是 Extension，哪些东西能持久化，哪些东西只能观察。  
  
因为插件能力一旦无限扩大，Agent 又长期运行，状态会变得非常难推理。  
  
OpenCode、Claude Code、DSH、Pi 现在看着都是 Coding Agent，底层其实已经开始走不同路线了。  
  
Harness 这东西，开始越来越像 AI 时代的操作系统了🤔

![Image](<Max%20For%20AI%20(@MaxForAI).assets/image-01.jpg>)

---

原文，挺值得一读的