---
author: "meng shao"
handle: "@shao__meng"
source: "https://x.com/shao__meng/status/2111261981735067819"
date: 2026-10-16T16:32:04.000Z
type: tweet
likes: 124
reposts: 27
replies: 22
bookmarks: 161
views: 11120
---
# meng shao (@shao__meng)

怎么才能更信任你的 Agent 的工作呢？  
  
Grok Bot 团队 @poteto 提出了一套如何信任 AI Agent 工作成果的四层“倒金字塔”框架。她认为，与其花精力“管教” Agent，不如花精力改造代码库本身！  
  
# 四层倒金字塔 (精力投入从上到下递减)  
  
第一层：代码库改进 (投入最多) 更好的架构、更强的类型系统与不变量、清晰的模块边界。  
  
Lauren 认为这本质上“就是好的软件工程”。理由在于 Agent 的天性：Agent 倾向于复制已有代码、走最短路径；所以与其对抗这个天性，不如让最短路径本身就是正确路径。  
  
第二层：静态分析 Lint 规则、测试、CI、形式化验证(如 TLA+)。这是性价比很高的第二道防线，用极低的 token 成本在问题早期就拦截它，出错反馈是确定性的，不依赖 Agent “理解”了什么。  
  
第三层：文字性指导 (最少使用) 代码注释、规则文件 (如 AGENTS.md)、Skills 描述等。这一层的共同弱点是无法强制执行，它要求 Agent 自行判断是否遵守，而 Agent 的判断并不总是可靠，所以只能作为补充，用在确实需要主观判断的场合。  
  
第四层：人工审查 (最难规模化) 人的精力有限，不应消耗在逐行检查 Agent 产出上，而应投向更上游的事：设计出让 Agent 默认就做对的环境。  
  
为什么金字塔是“倒置”的？  
传统上团队把大量精力放在金字塔底部 (写规范、人工审查)，而 Lauren 主张反转：  
· 越往上层，手段越是确定性的 (代码结构直接决定 Agent 能走什么捷径) 且token 高效的(不需要反复沟通纠偏)；  
· 越往下层，越依赖不可靠的理解与不可扩展的人力。  
  
这和业界近年 “从 prompt engineering 转向 context/engineering 工程” 的共识是同一方向；但 Lauren 把它推到了更根本的层面：连提示词本身都只是第三等手段，真正的杠杆在代码库里。

![Image](<meng%20shao%20(@shao__meng).assets/image-01.jpg>)

> **lauren (@poteto)**
> 
> how do you trust your agents' work more? copy this post/link and give it to your @bot or agent!  
>   
> 1. codebase improvements: this is just good software engineering - better architecture, stronger types and invariants, clear boundaries. agents love to copy what exists and take the
> 
> ![Image](<meng%20shao%20(@shao__meng).assets/image-02.jpg>)