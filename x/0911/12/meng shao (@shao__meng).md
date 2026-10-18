---
author: "meng shao"
handle: "@shao__meng"
source: "https://x.com/shao__meng/status/2095296258680496466"
date: 2026-09-02T15:08:00.000Z
type: tweet
---
# meng shao (@shao__meng)

CS146S: The Modern Software Developer  
Stanford University • Fall 2026  
  
斯坦福大学 @mihail_eric 开设的本科/研究生课程（Fall 2025 同样有线上课程，非常期待！），课程为期十周！  
[themodernsoftware.dev](https://themodernsoftware.dev)  
  
Eric 认为：软件开发的主体正在从"人写代码"转向"人定义意图、Agent 执行、人与 Agent 共同评估和迭代"。他这门课程，要教的就是这套新工作流所需的全部能力栈——如何给 Agent 提供上下文和工具、如何把产品需求翻译成可执行的规格、如何设计人机协同的迭代闭环。  
  
# 十周课程的层层递进  
  
第一层：理解 Agent（Week 1） 先拆开黑盒。LLM 是什么、agent loop 长什么样、read/write/edit/bash 四个核心工具如何承载任务。第一课就"用 200 行代码构建 Claude Code"，第二课深读生产级 Agent 的系统提示词。这是全课的地基：只有理解 Agent 的运作机制，后面所有"配置"才不是玄学。  
  
第二层：武装单个 Agent（Week 2–4）  
Week 2 上下文工程：高级提示技巧、RePPIT（Research→Propose→Plan→Implement→Test）的规格驱动开发框架、MCP 的完整原理与工具设计的"Agent 人机工学"。  
Week 3 Agent Skills：SKILL.md + 脚本如何把工作流编码为可复用能力，以及 CLI 工作方式。  
Week 4 定制化：CLAUDE.md / AGENTS.md 该放什么、Hooks 做 lint/测试门禁、planner/implementer/reviewer 子 agent 模式。  
这三周解决的问题是同一个：Agent 的能力上限由你给它的上下文、工具和约束决定，而非模型本身。  
  
第三层：改造环境以适配 Agent（Week 5–7）  
Week 5 Agent-Ready 代码库：结构、文档、测试、检查——什么让一个仓库对 Agent 友好，如何量化评估"就绪度"。  
Week 6 Agentic 代码审查：AI 审查擅长什么、遗漏什么，审查架构和自定义规则，如何嵌入团队 PR 流程。  
Week 7 安全：SAST/SCA、依赖与密钥泄露，以及 Agent 特有的提示词攻击面，并用 Agent 辅助分诊和修复。  
这一层的洞察是：当 Agent 产出代码的速度大幅提升，瓶颈会转移到验证、审查和安全。代码库本身必须重构为"可被 Agent 理解和检验"的形态。  
  
第四层：规模化与组织化（Week 8–10）  
Week 8 后台 Agent：异步云端委派、并行 Agent 集群管理、Issue→PR 自动流水线（Slack/Linear/GitHub 触发）。  
Week 9 AI-Native 团队：MCP portals（集中化、带权限的工具访问）、LLM gateway、模型路由与成本优化、组织级推广模式。  
Week 10 软件工厂：自运行、自改进的软件系统，部署后 Agent 的运行与安全治理，以及未来展望。  
终点是"软件工厂"——软件不再是被人工建造的产品，而是一个持续自我演进的系统。这是全课在理念上最激进的部分。  
  
# 课程两个独特的设计  
  
1. 强制向真实开源项目提交 PR（占 30% 成绩） 这是课程最有分量的创新，作者称"史无前例"。15 个 OSS 合作方（Browserbase、HeyGen、CopilotKit、Semgrep、OpenHands、Milvus、Marimo、Pi、CrewAI、Warp、Vercel、cmux、Arize Phoenix、Unsloth、Anyscale）提供支持与导师制。  
  
目的是培养"软件品味（software taste）"：当 Agent 能生成任何代码时，判断什么值得做、什么是好的、什么能被真实维护者接受，成为核心竞争力。这只能在生产级代码库中通过真实反馈习得，作业沙盒做不到。  
  
合作方名单本身也是一份 2026 年 AI 基础设施生态图谱：浏览器自动化、Agent 框架、代码安全、向量数据库、可观测性、模型微调、分布式计算——学生的贡献对象就是他们未来要用的工具链。  
  
2. 嘉宾阵容即行业一线 Lee Robinson（SpaceX/Cursor）、Boris Cherny（Claude Code 创造者）、Eno Reyes（Factory）、Silas Alberti（Cognition/Devin）、Isaac Evans（Semgrep）、Rajesh Bhatia（Cloudflare）、Amjad Masad（Replit）、Elad Gil。

![Image](<meng%20shao%20(@shao__meng).assets/image-01.jpg>)

> **Mihail Eric (@mihail_eric)**
> 
> I’m excited to finally announce the newest edition my Stanford course 𝗧𝗵𝗲 𝗠𝗼𝗱𝗲𝗿𝗻 𝗦𝗼𝗳𝘁𝘄𝗮𝗿𝗲 𝗗𝗲𝘃𝗲𝗹𝗼𝗽𝗲𝗿. It has been 9 months in the making.  
>   
> Last November, with the release of Claude Opus 4.5, coding agents experienced a step function improvement in
> 
> ![Image](<meng%20shao%20(@shao__meng).assets/image-02.jpg>)