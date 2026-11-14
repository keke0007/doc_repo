---
author: "AYi"
handle: "@AYi_AInotes"
source: "https://x.com/AYi_AInotes/status/2119462929682579610"
date: 2026-11-08T07:32:13.000Z
type: thread
likes: 71
reposts: 14
replies: 10
bookmarks: 111
views: 10443
---
# AYi (@AYi_AInotes)

Anthropic 官方 Agent 管理学来了！值得连看3遍学习和品味！  
Anthropic Claude Code 的开发主管Addy Osmani详解管理 Agent 的 5 条高杠杆铁律非常干货，强烈推荐给大家！  
很多程序员老哥走纯技术路线就是为了不当经理，就是为了远离冗长的例会、复杂的协作和管理人的琐事，只想安安静静写代码，  
结果用上 Coding Agent 的第一天，就被逼着当上了五人团队的技术主管，因为工程师的核心工作已经从写代码变成了让好代码发生，  
Addy Osmani刚发的这篇文章戳破了一个无法回避的现实： 当你每天开始在后台挂着三五个 Coding Agent 帮你跑任务时，你其实已经被系统直接推上了技术经理的位置。  
你每天做的事情早已不是敲击键盘，而是在选定任务、交代背景、制定验收标准、审查代码变更，并决定哪些东西可以正式上线。 工程师的核心工作，已经从亲自写代码，彻底变成了让好代码发生。  
  
Addy Osmani 把 Anthropic 内部管理 Agent 的高杠杆经验全盘摊开，提炼出五条最值钱的执行铁律：  
  
建立自检闭环，绝不当人肉测试机 这是最高杠杆的一步。不要自己一遍遍肉眼找 Bug，而是给 Agent 提供明确的测试用例、构建命令和代码规范，让它自己去跑、自己看失败报错、自己修复。没有自检护栏的人，最后自己会沦为系统的最大瓶颈。  
  
早干预与两次重置原则 看到模型走偏立刻按 Esc 叫停。如果连续纠偏两次模型依然理解不了，不要尝试第三次辩论，果断清空会话重新写更清晰的提示词。第三次纠错往往只是在给充满噪音的历史上下文添乱。  
  
警惕隐形的调度税 同时跑的 Agent 绝不是越多越好。超过三五个之后，每一个新增的智能体都在指数级掠夺你的审查注意力。用自动化评审接管监控，把有限的注意力留给最终的架构决策。  
  
认真审查代码，而不是给代码盖章 不要因为测试全绿、文件看起来合理就顺手批准大段代码变更。不理解逻辑的批准不是审查，而是认知层面的投降。看变更时必须假设这个逻辑漏洞上线后会半夜把你叫醒。  
  
让每一次纠错变成制度记忆 同一个错误在不同任务里出现两次以上，必须立刻固化进 CLAUDE.md 或者封装成专属 Skill。把个人脑子里的纠偏，变成所有智能体都能终身继承的系统规范。  
  
以前管理技能只有带团队的主管才被迫去学；现在每一个用 Agent 的工程师，都会在日常开发里被逼着学会这一整套高维心智。  
  
Addy Osmani 深度长文原文与 Agent 管理行动清单整理在一楼。

![Image](<AYi%20(@AYi_AInotes).assets/image-01.jpg>)

> **Addy Osmani (@addyosmani)**
> 
> ![Article cover](<AYi%20(@AYi_AInotes).assets/image-02.jpg>)
> 
> 📝 **Your coding agents made you a manager**
> 
> The skills for managing agents are pretty much the skills for managing a team, and you don't need to be a manager to learn them.
> If you're spending all day with a coding agent, you are managing....

---

→ 开发者高效驾驭 Agent 核心行动清单：  
→ 提供自验证脚本：每次派发任务附带测试用例、构建命令或截图验收标准，要求 Agent 提供验证通过的证据  
  
→ 上下文精简：只交代代码仓库之外的信息（业务意图、边界约束、不做什么），不重复代码库内已有细节  
  
→ 两次失败硬重置：若两次纠偏未能解决问题，果断使用 Esc 终止并开启新会话重写更精炼的 Prompt  
  
→ 制度化记忆沉淀：重复出现的规范写入 CLAUDE.md，复杂高频流程封装为独立 Skill  
  
→ 严格区分建议与硬约束：提示词与文档属于模型考量的软建议，Hook 与 Deny Rules 才是物理层面绝对执行的硬性闸门