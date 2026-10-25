---
author: "meng shao"
handle: "@shao__meng"
source: "https://x.com/shao__meng/status/2112371729276563825"
date: 2026-10-20T02:34:00.000Z
type: tweet
likes: 224
reposts: 58
replies: 32
bookmarks: 379
views: 21391
---
# meng shao (@shao__meng)

Meta 工程团队 @Meta_Engineers 发布了 Agent 架构实践:一个让 Agent 更可靠的架构选择「Thin Orchestrator」  
  
一个中央主 Agent “知道得越多，表现得越差”；这不是提示词问题，是架构问题！  
Meta 团队最初用单个“全能 Agent”自动化推荐系统的数据列上线流程：它同时装载了所有领域知识、schema、代码生成规则、路由逻辑和校验要求。结果在演示中正常，在生产环境中开始产生幻觉。具体表现为知识交叉污染：把 A 子系统的命名规范套用到 B 上，把适用于某数据源的代码生成模式用错对象。  
  
这一现象有实证支撑：他们引用 Mount Sinai 的研究，多 Agent 编排系统在 80 任务批次下保持 65.3% 准确率，而单 Agent 仅 16.6%，token 消耗最多降低 65 倍。原因在于任务隔离阻止了上下文互相干扰，上下文超过某个阈值后，更多知识反而降低性能。  
  
解决方案：两层架构 + 严格的职责边界  
第一层：确定性的瘦编排器（约 1300 行代码），零领域工作。 它把 20+ 工作流路由到 14 个专业 Agent，自己只做三件事：意图分类、收集必要字段、执行可靠性检查。它从不推理代码、不生成配置、不触碰代码库。  
  
第二层：14 个上下文隔离的专业 Agent。 如 schema 管理 Agent、population Agent（负责 10 种数据源的代码生成）、校验 Agent 等。关键设计是，没有任何一个 Agent 看到全貌，这是刻意为之。  
  
每个请求走固定的 7 步协议：读取子 Agent 指令文件 → 查重 → 收集用户字段 → 自动推断（repo 类型等）→ 可靠性门禁检查 → 委派子 Agent → 处理结果。其中第 1–5 步和第 7 步全部是确定性逻辑，只有第 6 步涉及 LLM 推理，且推理发生在只装载自身领域知识的隔离环境中。  
  
最有工程价值的洞察：模板复用而非新增 Agent  
他们把“做什么”（workflow 模板）与“每步怎么做”（子 Agent）分离。模板是确定性状态机，例如：  
· 列上线：Schema → 校验 → Population → 测试  
· 事件上线：Schema → Population → 校验 → 测试 → 事件处理  
· 管道迁移：评估 → 规划 → Population → 校验 → 监控  
  
支持管道迁移这个全新能力时，只需新增一个模板重新排列现有 Agent，“零新增 Agent 代码，完整生产能力”。所有子 Agent 结构统一（知识源、预收集清单、工作流步骤、错误-解决方案映射），第 15 个 Agent 与第一个在结构上完全一致，新增能力变成模板化工作。  
  
知识在运行时按需加载而非内嵌：25+ 知识文件按领域组织，每份通常几千 token，更新后在下一次会话自动生效，且同一份文件可服务多个平台实现。  
  
博客地址  
[engineering.fb.com/2026/10/19/ml-…](https://engineering.fb.com/2026/10/19/ml-applications/thin-orchestrator-ai-agent-architecture/)

![Image](<meng%20shao%20(@shao__meng).assets/image-01.jpg>)