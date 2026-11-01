---
author: "jolestar"
handle: "@jolestar"
source: "https://x.com/jolestar/status/2114344057975754754"
date: 2026-10-24T12:21:16.000Z
type: tweet
---
# jolestar (@jolestar)

极度赞同。之前不少模型专门优化了 subagent 的主动性，结果实际 Agent 跑起来像极了一个刚当上领导的官迷：自己啥都不干，有活就起 subagent，自己在原地干等，就差泡壶茶了。  
  
所以前几天给 @holonrun 补充委派准则时，核心原则就是“默认必须自己把活干完”，明确禁止把任务派发给 subagent 然后自己干等 relay。  
  
只有这几种真正能带来收益的场景才准起 subagent：  
  
1. 真并行：子任务有独立交付物，主 agent 必须同时推进其他不冲突的工作；  
2. 专家角色/执行环境：需要专属的 review 职责、独立运行环境，或指定 Agent 模板初始化成套技能；  
3. 上下文隔离：大日志分析、跨仓库排查等，只取结论和证据，避免污染主上下文；  
4. 独立审阅视角：比如需要一个干净、不受做题思路先入为主影响的独立 Review 视角。

> **Tak (@cherry_mx_reds)**
> 
> Anthropic is now recommending that companies REMOVE unnecessary subagents from their AI systems.  
>   
> This is a pretty significant shift in agent architecture, with their team finding that a single agent equipped with skills can frequently outperform complex multi-agent systems, and