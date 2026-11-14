---
author: "工业聚"
handle: "@guyingjie129"
source: "https://x.com/guyingjie129/status/2119063224272433423"
date: 2026-11-07T13:43:37.000Z
type: tweet
---
# 工业聚 (@guyingjie129)

多 Agent 系统/多 Agent Sessions 协作，有很多容易掉入的陷阱。  
  
众所周知的是模拟人类团队协作里的岗位、角色、拟人等。比如写手、程序员、研究员、设计师、运营等Agent，显然的结果是——  
  
常常变成 Agent to Agent 的聊天室，大量算力沉没在沟通对齐而非交付推进上。  
  
更可怕的是，对齐是不可能严丝合缝的。误差/偏差的概率性发生及其持续累积，将在每一次 Agent to Agent 的局部上下文传递/压缩中，不可避免地滋生。  
  
那么，编排 Agents/Sessions 的原则到底是什么？  
  
想明白之后，其实很简单，很显然，很自然。  
  
因为核心不变量（指标）只有一个——让交付效能最大化。  
  
第一个重点是区分 Work 和 Meta Work，Work 是最扎实的工作，Meta work 是让 Work 更好而做的 Work，所以叫元工作。  
  
第二个重点是在 DDD 里也有体现，对 Work 分类：  
  
- Core：核心的，跟推进目标、交付结果直接关联  
- Support：支撑性的，非直接关联，但必不可少  
- Generic：通用性的，不只是跟这次目标有关  
  
Work 集中在 Core 范围，Meta-Work 集中在 Support/Generic 区域。  
  
不存在一个通用的 Agents/Sessions 拆分，因为 Goal 不同，Work/Meta-Work 和 Core/Support/Generic 的分类方式也不同。  
  
给定一个目标，多 Agents/Seesions 的拆分就是让 Work Agents/Sessions 的时间最大化投入在 Core Work 部分。  
  
以搭建最简循环工程为例，最小只需要两个 Sessions，一个是【交付会话】，一个是【协调会话】。  
  
- 交付会话：聚焦目标推进  
- 协调会话（定时任务）：  
* 周期性搬运目标关联的上下文到本地队列（从文档、IM、连接器等来源）  
* 将阶段性交付验收材料搬运到指定位置（IM/文档等）  
* 按需唤醒【交付会话】推进工作  
  
产品、设计、开发、测试不是 4 个 Agent 或 4 个 Session，而是一个端到端交付 Agent/Session 的 4 个工作阶段，4个阶段在 Loop 里也会随上下文输入/输出而反复横跳。  
  
不需要引入额外工具，现在你便可直接搭建上述 Sessions 协作，结合目标可验收性和 Browser-use/Computer-use 等验收工具，获得 10x 效能提升。  
  
协调会话帮交付会话分担事务性工作，让交付会话不必亲自去来回搬运。  
  
当你想要继续拓展时，【最大化交付原则】可以继续发挥指导作用，比如围绕【协调会话】，聚焦它的 Goal，找到它的 Core/Support/Generic 细分，很快发现——  
  
【协调会话】最重要的是保活【交付会话】，如果它长时间沉浸在搬运上下文这种事务性工作中，它就无法及时唤醒交付会话，也无法及时响应来自 IM 中的消息。  
  
所以【协调会话】可引入 Supporting Subagents/Sessions，将文档操作、消息响应等支撑性工作外包给它们。  
  
Gall's Law：一个运转正常的复杂系统，总是从一个运转正常的简单系统演化而来的。”  
  
Work/Meta-Work + Core/Support/Generic 两类拆解方式，配合【最大化 Work 交付】目标，可以帮助我们从简单系统出发，不断增加系统复杂度而更不容易丢失核心交付能力。

![Image](<工业聚%20(@guyingjie129).assets/image-01.jpg>)