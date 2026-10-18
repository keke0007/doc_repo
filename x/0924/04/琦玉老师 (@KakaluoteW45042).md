---
author: "琦玉老师"
handle: "@KakaluoteW45042"
source: "https://x.com/KakaluoteW45042/status/2102711712172630100"
date: 2026-09-23T02:10:00.000Z
type: thread
---
# 琦玉老师 (@KakaluoteW45042)

把专家经验变成可测试、可版本控制的代码，本质是把 org memory 当 codebase 管：CI、review、回归测试都得配齐。真正难的不是存储，是 elicitation——把老工程师脑子里的 tacit knowledge 抠成 assert，还有谁 own 这份 repo、人走了怎么防 drift 🤔

> **lifcc (@mylifcc)**
> 
> Meta 刚公开了一个我觉得会改变 Agent Memory 方向的系统：Organizational Second Brain。  
>   
> 它做了一件很疯狂的事，就是蒸馏公司专家：  
>   
> 把公司专家脑子里的经验，变成 Agent 可以持续学习、修改、测试、版本控制的代码。
> 
> ![Image](<琦玉老师%20(@KakaluoteW45042).assets/image-01.jpg>)

---

SPACE 值得看的是它把信任边界做成了默认架构。沙箱不是防 agent 乱跑，是给 agent 划一块能放开手脚的地方。每任务一个 microVM 加凭据外置，agent 该有动作半径，凭据不该。60 毫秒的中位启动让按任务开 VM 划得来账，这是安全问题被做成了经济学问题的样板。

> **梦莱加德 (@EuneJjj73857)**
> 
> Perplexity 把自家跑 Agent 的沙箱开源了，项目叫 SPACE。  
>   
> 它解决的是个很实际的问题：Agent 要执行代码，但你不敢让它在自己机器上跑。  
>   
> SPACE 的做法是每个任务起一个独立的 Firecracker microVM，凭据和出网流量都放在 VM 外面控制。  
>   
> 官方说中位启动时间大约 60 毫秒，支持暂停、恢复和快照。
> 
> ![Image](<琦玉老师%20(@KakaluoteW45042).assets/image-02.jpg>)