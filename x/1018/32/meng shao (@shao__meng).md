---
author: "meng shao"
handle: "@shao__meng"
source: "https://x.com/shao__meng/status/2110922224547082692"
date: 2026-10-09T02:28:27.000Z
type: tweet
likes: 103
reposts: 30
replies: 13
bookmarks: 119
views: 4940
---
# meng shao (@shao__meng)

密歇根大学最新 AI 编程课「Applied Agentic Software Engineering」  
  
第 14 讲「Subagents」讲义刚刚公开，围绕：为什么需要 subagents、机制是什么、关键取舍、以及它和相邻概念的区别 四个部分展开。  
  
讲义：[github.com/eecs498-aase/m…](https://github.com/eecs498-aase/materials/tree/main/lectures/L14)  
  
Part 1：为什么需要 subagents？  
因为长对话必然劣化，四个原因：  
· 上下文有限，内容只进不出  
· 被否决的想法仍在场（history bleed），模型会捡回来；这就是“变差”的来源  
· 每次请求重发全部历史，caching 省钱不省窗口  
· 单对话串行，读代码时无法并行比较方案  
  
解法：主对话只留请求和计划，各部分的阅读留在各自的 subagent 里；只回流结果，垃圾全部丢弃。  
  
Part 2：subagents 机制  
· Agent 工具对模型就是普通工具；prompt（即 brief）是 subagent 信息的唯一来源  
· harness 立即回 "launched"（默认后台），开出第二对话：更短的系统提示词、更少的工具（Explore 无 Edit/Write/Agent，物理上只读）、同样的 CLAUDE.md、没有主对话历史  
· 结束必须调 SubagentHandback，否则内容不送达  
· 报告回流时的防护：标记为模型输出而非用户（无用户权限）、整体缩进（伪造帧线可辨）；用纯格式守住对话边界  
· brief 是模型写给模型的提示词：位置、验收标准、报告形状都要写清；模糊的 brief 得到模糊的搜索  
  
Part 3：Fresh 还是 Forked（最大取舍）  
harness 自己列出的风险：  
· 细节双向丢失：它只知 brief，你只见报告  
· 错误与正确发现同样自信  
· 最危险：拿到你的假设，倾向于证实着送回来；要第二意见，给它代码，不给结论  
  
Fresh：起点「仅 brief」、适合「第二意见、独立审查」、成本「更贵」  
Forked：起点「全部历史 + brief」、适合「需要全部积累的侧任务」、成本「更便宜（复用 prompt cache）」  
  
Fresh 不是便宜选项，是独立选项。 L10 的 Goldfish 必须是 fresh：fork 出来的会用记忆填补文档缺口，然后说文档没问题。  
  
Part 4：辨析与工程  
三个问题区分一切形态：几个对话、谁和谁说话、谁决定下一步。  
· Subagents：每段新对话，只与父对话通信，父对话掌权  
· Team：多对话 + 互发消息 + 共享任务列表（带文件锁），你可直连任意队友  
· Handoff：一段对话，换个 agent 接管（换指令换工具、保历史），不回头；与 subagent 恰好相反  
· Swarm = 机制 × 规模（Kimi K2.5：100 并发；OpenAI Navier–Stokes：约 10,000 agents、1,300 亿 token）  
· 并行编辑靠 worktree 隔离：每个 subagent 独立 checkout + 分支，改完留分支等合并

![Image](<meng%20shao%20(@shao__meng).assets/image-01.jpg>)

> **meng shao (@shao__meng)**
> 
> ![Article cover](<meng%20shao%20(@shao__meng).assets/image-02.jpg>)
> 
> 📝 **密歇根大学最新 AI 编程课把方法论做成了五个 Skill：「Applied Agentic Software Engineering」深度解读**
> 
> 密歇根 EECS 498 AASE：一门把“AI 编程”教成工程学科的课程
>  
> Michigan EECS 498 AASE: Applied Agentic Software Engineering
> https://eecs498-aase.github.io/
> 2026...