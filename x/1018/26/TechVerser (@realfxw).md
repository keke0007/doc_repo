---
author: "TechVerser"
handle: "@realfxw"
source: "https://x.com/realfxw/status/2111110544769859586"
date: 2026-10-16T15:02:31.000Z
type: tweet
likes: 26
reposts: 4
replies: 5
bookmarks: 21
views: 2026
---
# TechVerser (@realfxw)

当 AI 写了团队 90% 的代码，人类该如何做 Code Review？🤖🔍  
  
Anthropic 最新发布的工程博客《How the Claude Code team does code review》干货极多。在主仓库日均变更从 5 月的 2,500 次飙升到 9 月 30,000+ 次的极限压力下，Claude Code 团队不仅让 PR 合并量翻倍，还把代码回滚率（Revert Rate）压低了 40% 以上！  
  
他们的核心逻辑非常清晰：人类的注意力和时间才是最稀缺的资源。 审查的标准不再是“有没有人类读过这行代码”，而是“这个改动可能出什么问题，应该由机器还是人来拦截？”  
  
以下是他们跑通的 AI 原生代码审查体系核心亮点：  
  
1️⃣ 审查前置：PR 诞生前就已完成自检代码写完后，Claude 必须先自己跑通测试、构建和 Linter，甚至对比 UI 截图，随后拉起一个全新上下文的 Subagent 审查 Diff。对于反复出现的老问题，团队绝不靠人肉提醒，而是直接写进 CLAUDE.md（全局规范）和由模块负责人维护的 REVIEW.md（明确该模块“什么才算真正的故障”以及忽略哪些细枝末节）。  
  
2️⃣ 多 Agent 并行找 Bug + 投票过滤噪音每个 PR 都会触发多 Agent 并行扫描，随后由另一组独立的验证 Agent 对发现的问题进行投票——只有高票通过的真问题才会变成评论，并标记为 Important、Nit 或 Pre-existing（历史遗留）。这让实质性审查评论覆盖率从 16% 跃升至 54%，同时最大程度避免了误报噪音。  
  
3️⃣ 设置“收敛机制”，防止 AI 陷入死循环当作者是 Agent、审查员也是 Agent 时，很容易互相死磕——团队曾有个 PR 一晚上被两个 AI 来回改了 50 多轮！为此他们引入了收敛策略：最多 3 轮审查上限（超限需人类介入重新思考方案）、几轮后仅做增量 Diff 审查、按同类 Bug 报错而非逐行挑刺。改造后，PR 单轮通过率直接从 65% 提升到了 78%。  
  
4️⃣ 按风险分级审批，迈向“零人工合并”系统通过分类器根据代码改动本身的风险（而非作者身份）来决定需要几个真人审批：  
  
低风险（文档、内部功能、Feature Flag 后的增量代码）：只要自动化测试、安全扫描和 AI 审查全绿，受信任的作者无需其他人类审批即可合并。  
中高风险（计费、自动更新器、系统提示词、CI 工作流）：强制要求 1～2 名人类审批。 同时严格执行 “You land it, you own it”——用 Prompt 指挥 Agent 写代码的人，必须对最终结果负全责。  
  
5️⃣ 把每一次线上事故变成每日 Eval任何逃逸到线上的故障或回滚，都会被提取为测试用例加入评测集（Eval Set）。每当调整审查 Agent 的提示词、工具或底层模型时，都会用这套真实故障集每日跑分验证，实现审查能力的飞轮闭环。  
💡 一个反直觉的洞察：如果机器接管了读代码，工程师怎么熟悉代码库？传统观念认为读队友的 Diff 是熟悉项目的最佳方式。但 Claude Code 团队坦言：没人能在脑子里装下整个代码库，看一行改文案的 Diff 也学不到任何东西。现在当工程师要改某个模块时，直接让 Claude 针对那一块代码生成实时解析或可视化 Artifact，几分钟就能精准掌握最新架构。  
  
那么人类留下来做什么？做机器不擅长的 5 件事：决定这功能该不该做（方向与品味）、把关高危代码、设计底层架构、查验测试证据，以及为最终结果担责。  
  
从“人肉逐行看代码”进化到“多智能体层层设防 + 人类聚焦高维决策”，这就是 Agent 时代软件工程的全新范式。🚀  
  
原文：[claude.dev/blog/solving-c…](https://claude.dev/blog/solving-code-review-at-anthropic/)  
  
#ClaudeCode #Anthropic #AIEngineering #CodeReview #AgenticWorkflow #软件工程

> ![Link card preview](<TechVerser%20(@realfxw).assets/image-01.png>)
> 
> 🔗 [**How the Claude Code team does code review / claude.dev Blog**](https://t.co/buI1OjaxL5)
> 
> _From claude.dev_