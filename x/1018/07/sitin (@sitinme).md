---
author: "sitin"
handle: "@sitinme"
source: "https://x.com/sitinme/status/2111425240492281976"
date: 2026-10-15T01:09:54.000Z
type: thread
---
# sitin (@sitinme)

看到一篇文章感觉有点在理：这篇文章的核心观点是：多智能体系统不能把所有消息都塞进一个聊天框，而应该按通信关系拆成不同频道。  
  
在多 Agent 协作中，真正混乱的往往不是模型，而是负责监督的人。  
  
一个聊天流里同时出现任务要求、子 Agent 汇报、同级 Agent 协调、人工修改意见、CI 报错和系统通知，时间一长很难追溯“谁对谁说了什么”。  
  
所以多 Agent Harness 至少应该把消息分成五类：  
  
上游任务要求、下游 Agent 汇报、和人类同事的协作、同级 Agent 之间的协调，以及系统警报。  
  
它们可以共享同一个工作上下文，但在界面和权限上应该分开。  
  
比如，上游负责目标和验收，人类同事可以直接纠正具体实现，同级 Agent 只能协调，系统警报则应该有自己的优先级和处理规则。  
  
这件事看起来像是界面设计，实际上关系到整个系统的可靠性。  
  
消息来源不清楚，Agent 可能把建议当命令，把警报当新任务，或者多个 Agent 同时修改同一个文件。出了问题以后，也很难追溯到底是谁做了什么决定。  
  
现在很多多 Agent 系统，本质上还是在用“一个人聊天框”模拟一个团队。  
  
随着 Agent 数量增加，真正重要的可能不只是模型有多聪明，而是 Harness 能不能把不同关系、不同权限、不同紧急程度的消息管理清楚。

![Image](<sitin%20(@sitinme).assets/image-01.jpg>)

> **sitin (@sitinme)**
> 
> 看了一篇挺有意思的文章：一群开发者用 Claude Code 和 Codex，花了大约 3 个月，让一群 AI Agent 自动逆向还原一款经典第一人称射击游戏，累计消耗估计 6000 亿到 7000 亿 Token。  
>   
> 整个过程很有意思。他们一开始安排了 4 个 Agent，3 个负责写代码，1 个负责审核。第一个月就还原了大约 80% x.com/sitinme/status…
> 
> ![Image](<sitin%20(@sitinme).assets/image-02.png>)
> ![Image](<sitin%20(@sitinme).assets/image-03.jpg>)

---

如果消息混在一起，一定会带来几类问题：  
  
1.权限和权威不清楚：子 Agent 的建议看起来可能和人类主管的命令一样重要。  
  
2.回复发错地方：本来应该向上级汇报，却被写进了给下游 Agent 的对话。  
  
3.同级 Agent 互相冲突：可能同时修改同一个文件，或重复修复同一个问题。

> ![Link card preview](<sitin%20(@sitinme).assets/image-04.jpg>)
> 
> 🔗 [**Multi-agent harnesses need separate message channels · Harshal Gajjar / blog**](https://t.co/jzZaBbzDSX)
> 
> _From harshalgajjar.com_