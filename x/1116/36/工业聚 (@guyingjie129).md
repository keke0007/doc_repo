---
author: "工业聚"
handle: "@guyingjie129"
source: "https://x.com/guyingjie129/status/2122267919795237372"
date: 2026-11-16T05:34:37.000Z
type: thread
---
# 工业聚 (@guyingjie129)

我将智能的价值来源分为两类——过程智能/结果智能。  
  
其中，结果智能是最重要的，它是指智能服务于最终结果的质量提升。  
  
过程智能则是指，智能服务于人的体验过程（各类智能驱动的自动化操作）。  
  
没有结果智能，产品缺乏价值根基，无法长存。缺乏过程智能，则拖慢结果的产能（效率属性），大量时间耗费在用户跟产品在使用上的搏斗中。  
  
用上述两个维度考察看板产品，你会发现，它两不沾。  
  
既对结果智能没有任何贡献，又没有帮助人们更好地在自动管理过程。  
  
不仅如此，情况还可能更遭。因为任务来源是可以 24小时持续工作的 Agent，它将让看板里堆积的任务 x10 增加；让任务管理过程更加痛苦。  
  
看板产品陷入了两难：  
  
- 增加过程智能，则不需要面向人类的看板，任务管理是 Agent 内部过程；  
- 增加任务的颗粒度，以减少看板里的任务规模，则每个任务的完成时长将 10x 增长，使得看板更新频率低到不用去看，而是去收件箱里看结果；  
  
前沿模型已经足够智能了，但大部分 agent 产品没有从第一性原理出发，去设计它们的组织协作模式。  
  
下一次，可以试试从【结果智能/过程智能】两个维度审视你的产品设计。

![Image](<工业聚%20(@guyingjie129).assets/image-01.jpg>)

> **Theo - t3.gg (@theo)**
> 
> Every single time someone says T3 Code isn’t “optimal for real work” they show the most useless 2025-era kanban slop that wastes over half the screen real estate for in-progress work.  
>   
> This is the most poorly optimized UI for real productivity that I think I’ve seen in my life. x.com/agent_wrapper/…

---

分享最近几个比较有意思的 Session Management 的操作案例——  
  
1、带教模式->自我迭代  
  
第一阶段：带教模式  
  
让一个 Session in Codex App 去驱动另一个 Session in Codex CLI in Harness T，并根据使用情况改善该 Harness T。  
  
第二阶段：自我迭代  
  
Harness T 是一个桌面应用，让它在当前版本构建下一个版本的自己，并驱动下一个版本的 Harness T 接管当前版本的所有 Sessions 后，把自己杀死、删掉。让它不断开发所需的新特性给自己使用。  
  
2、跨机器全链路协作闭环验证  
  
两台机器通过 @Tutti_global 协作验证桌面软件的下载、配置、授权和使用的全流程闭环。  
  
机器 A 开发 Harness T，打包后，浏览器操纵 Tutti 房间里的机器 B 的 Session 背后的 Codex 去下载、安装、使用和反馈。  
  
Machine_A(Session_A) <-> Tutti(Machine_B(Session_B)) <-> HarnessT(Session_C)  
  
---  
  
当我们让 Agent 24小时工作+长程任务循环工程，其实我们可以接受更慢的对接方式和响应节奏。效率慢的部分被工作时间的延长缓解了，又被多个并行循环工程增强了。  
  
更重要的是，我们总是采用最前沿的 Model/Hanress 去保障【结果智能】。  
  
在这个背景下，那些主要是优化对接效率的 Agent 产品，其优势或许难以持续。

![Image](<工业聚%20(@guyingjie129).assets/image-02.jpg>)
![Image](<工业聚%20(@guyingjie129).assets/image-03.jpg>)
![Image](<工业聚%20(@guyingjie129).assets/image-04.jpg>)

> **工业聚 (@guyingjie129)**
> 
> 你不需要额外的产品才能体验 Super Agent, Always-on Agents 等 Session Management 实践模式。  
>   
> Codex/Claud-Code 等 Harness 的 Meta Sessions 功能已经很灵活了。你可以直接用它们搭一个可用的，并且你最好是这样做的——用最前沿和强大的 Harness 去运转 Sessions 协同。  
>   
> 你也不需要真的全自动的
> 
> ![Image](<工业聚%20(@guyingjie129).assets/image-05.jpg>)