---
author: "cxuan"
handle: "@criscxuan"
source: "https://x.com/criscxuan/status/2114379305266659792"
date: 2026-10-25T15:31:24.000Z
type: tweet
---
# cxuan (@criscxuan)

A 社这篇文章得读一下：《构建智能体：何时把子智能体合并回主智能体循环》  
  
文里有个观点说到：很多公司的 Agent 架构，不知不觉就编程成了内部组织架构图的翻版。  
  
做客服或企业中台时，市场、财务、订单各自搞一个独立子智能体，上面再挂一个总调度，看着分工清晰，其实像在复刻部门墙。  
  
问题也跟着来了：跨部门需求要来回传话，上下文在交接里丢细节；每个子智能体又是一轮嵌套推理，延迟和成本一起涨。有人实测端到端成功率从大约 55% 提到 82%，靠的就是把一堆子智能体收成「单主循环 + 几个 Skill」。  
  
Anthropic 的建议很直接：能收进主循环的就收。流程规则做成按需加载的 Skills，API 做成 MCP 工具；写计划和执行计划的是同一个模型，中途还能改计划。Claude Code 自己也把系统提示砍掉八成以上，改成用到再加载。  
  
子智能体留给这几类情况：上下文真的会爆、任务要并行且用完即扔、或者必须单独跟用户多轮对话。其余多数「伪子 Agent」，最终都会被模型能力冲掉，或者折回单循环。

> ![Link card preview](<cxuan%20(@criscxuan).assets/image-01.png>)
> 
> 🔗 [**Building agents: When to fold subagents into your main agent loop**](https://t.co/o1JZf75q02)
> 
> _From claude.com_