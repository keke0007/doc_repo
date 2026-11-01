---
author: "porco"
handle: "@SeanchenZ"
source: "https://x.com/SeanchenZ/status/2114939817352179839"
date: 2026-10-27T04:38:40.000Z
type: tweet
likes: 6
replies: 1
views: 149
---
# porco (@SeanchenZ)

2026 年 AI Agent 开发最常见的 30 个术语（英文全称 + 中文解释）  
  
最近从机械自动化跨到 AI Agent 开发，发现这个领域又有一整套工程术语。  
  
整理了 30 个常见概念，按实际开发流程分类。  
  
一、模型与上下文  
  
LLM（Large Language Model）：大语言模型，Agent 的推理与生成核心。  
Prompt（提示词）：定义任务、角色、输入信息和约束。  
Context Window（上下文窗口）：模型一次能处理的信息范围。  
Context Engineering（上下文工程）：决定给模型什么信息、何时给、如何组织和压缩。  
Agent（智能体）：能够根据目标调用工具、处理反馈并完成任务的系统。  
  
二、Agent 如何执行任务  
  
Tool Calling（工具调用）：让模型请求调用函数或外部工具。  
Agent Loop（智能体循环）：执行任务、观察结果、调整策略，直到完成或退出。  
Planning（任务规划）：把复杂目标拆分为可执行步骤。  
ReAct（Reasoning and Acting）：推理与行动相结合，根据环境反馈决定下一步。  
Agent Harness（智能体运行框架）：管理上下文、工具、执行循环与状态的运行控制层。  
  
三、知识、检索与记忆  
  
RAG（Retrieval-Augmented Generation）：检索增强生成，先检索资料，再结合资料生成回答。  
Embedding（向量嵌入）：把文本等信息转换成向量表示。  
Vector DB（Vector Database）：向量数据库，支持相似度检索。  
Memory（记忆机制）：保存并调用历史信息、用户偏好或任务状态。  
Checkpoint（检查点）：保存任务执行状态，支持中断恢复。  
  
四、多 Agent 协作  
  
Workflow（工作流）：按照预定义流程执行任务。  
Multi-Agent System（多智能体系统）：多个 Agent 分工协作完成复杂任务。  
Orchestration（编排）：协调任务顺序、依赖关系和执行流程。  
Handoff（任务交接）：将任务或执行控制权移交给另一个 Agent。  
Supervisor Agent（主管智能体）：负责任务分配、协调和结果汇总。  
  
五、接口、协议与能力扩展  
  
MCP（Model Context Protocol）：模型上下文协议，标准化连接外部工具与数据。  
A2A（Agent2Agent Protocol）：智能体间通信协议，支持不同 Agent 系统协作。  
Skills（技能包）：封装可复用的操作说明、脚本和资源。  
Structured Outputs（结构化输出）：让模型按约定的数据结构返回结果。  
Sandbox（沙箱）：在隔离环境中运行代码或执行操作。  
  
六、可靠性、安全与验收  
  
HITL（Human-in-the-Loop）：人在回路，关键步骤需要人工参与或审批。  
Guardrails（安全护栏）：对输入、输出和工具执行施加安全与业务约束。  
Tracing（执行追踪）：记录模型调用、工具执行和任务运行路径。  
Evals（Evaluations）：评测，通过测试集和评价标准验证 Agent 的实际能力。  
Idempotency（幂等性）：确保重复请求不会造成重复业务副作用。  
  
⸻  
  
现在做 AI Agent，我尤其关注最后五项。  
  
以前做自动化设备，机构能够正常动作，不代表整台设备通过 SAT（Site Acceptance Test，现场验收测试）。  
  
Agent 也是如此。  
  
能回答问题，不代表能正确执行任务。  
  
能调用工具，不代表能处理异常。  
  
一次 Demo 成功，更不代表系统能够长期稳定运行。  
  
这些术语没必要一次学完，但至少要分清楚：  
  
哪些技术决定 Agent 能不能跑，哪些工程能力决定它能不能真正交付。