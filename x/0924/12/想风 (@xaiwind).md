---
author: "想风"
handle: "@xaiwind"
source: "https://x.com/xaiwind/status/2102419733601783872"
date: 2026-09-21T17:35:33.000Z
type: tweet
---
# 想风 (@xaiwind)

有人晒了自己的 agentic coding 工作流：先花 10 到 30 分钟写一份设计文档，交给一个协调者 agent（顶配模型、推理拉满），让它把活拆开、并行分给下面的 agent。4.5 万浏览。  
  
看完最大的感受是顺序对了。大多数人（包括以前的我）是先让 agent 写，写崩了再回头补需求，来回烧 token。他反过来：人只做那 20 分钟最贵的思考，剩下全是体力活。  
  
我这两天照这个改了下：写一个 md，写清目标、边界、验收标准、不许碰的文件，然后让协调者带两个子 agent 并行改，我只审 diff。返工次数肉眼可见地少。  
  
今天就挑个小项目试一次。写文档花掉的半小时，一般能省下一整个下午。  
  
反过来说，如果你根本写不出这份文档，说明这事还没想清楚；这时候交给 agent，只会更快地跑偏。

> **Andreas Kling (@awesomekling)**
> 
> My current agentic coding workflow for larger projects (simplified):  
>   
> 1. Spend 10-30 minutes crafting a design document.  
> 2. Hand it to a coordinator agent (frontier model with maxed-out reasoning)  
> 3. Tell coordinator to parallelize the work over 4 separate computers via SSH