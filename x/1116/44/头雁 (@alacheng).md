---
author: "头雁"
handle: "@alacheng"
source: "https://x.com/alacheng/status/2122238169345298613"
date: 2026-11-16T07:59:43.000Z
type: thread
likes: 75
reposts: 13
replies: 11
bookmarks: 64
views: 7842
---
# 头雁 (@alacheng)

从Lauren Tan 土豆姐在AI Engineer Code Summit 上的演讲里可以看到SpaceXSI团队已经开始在自动化coding里使用了形式化验证做质量保证，并且token的70%用于验证质量！  
  
土豆姐在AI Engineer Code Summit 上的演讲，标题是 《All Code Is Slop (Except Mine)》（所有代码都是垃圾，除了我的）  
  
她讲的是 SpaceXSI 的 Grok Bot 团队如何用 AI coding agents 大规模写代码，同时不牺牲质量。（看来他们已经大量的在生产环境下用形式化验证保证质量了）  
  
-有了 AI agent 后，任何人都能变成“编码机器”，产量暴涨（她自己一个月提了约 3000 个 PR，团队一个月合并约 11,000 个 PR，回滚率约 0.5%）。  
  
-真正的挑战不是“写得快”，而是在产量暴涨的同时守住甚至提升代码质量。  
  
-关键不是盲目信任 agent，而是通过改善 agent 的环境来建立“earned trust”。  
  
她用一个倒金字塔说明应该把精力放在哪里（从最重要到最次要）：  
-代码库本身（最重要）：好的 API 设计、架构、重构。Agent 会模仿代码库里的现有模式，垃圾模式会被快速复制传播。  
  
-静态分析 / 强制约束：lint 规则、类型系统、编译器、测试、CI。把容易误用的东西直接禁掉（例如他们在 Dune 框架里几乎全面禁用了 React 的 useEffect）。  
  
-验证：运行时验证 + 形式化验证（他们用了 TLA+ 和 Lean）。把 token 更多地花在验证而不是生成上（提到日常 token 使用约 70% 用于验证）。  
  
-文字规则（rules、skills、AGENTS.md 等）：最后才用，因为这些不可强制执行，容易过时。  
  
实用做法举例：  
-用文件夹结构和 lint 规则强制隔离（比如 Electron 的 main 和 renderer 不能互相导入）。  
-尽量少写注释，改用能给出明确错误信息的 lint 规则，引导 agent 修根本原因而不是打补丁。  
-让 agent 自己跑应用、用 DevTools、看截图/视频来验证，而不是只看代码。  
-先把基础打好（干净的代码库 + 强约束 + 验证），再做自动化的“software factory”，否则自动化只会更快地生产更多垃圾。  
  
演讲视频：  
[youtube.com/watch?v=O_zcc6…](https://youtube.com/watch?v=O_zcc6Xl12g)  
  
视频里提到的形式化验证工具：  
TLA+ 和 Lean 都是形式化验证工具，用来从数学层面证明“这个系统/代码在某些情况下一定是对的”，而不是只靠测试去碰运气。  
  
TLA+验证：  
主要用于模型检查（model checking）核心用户流程的状态机。  
  
gorkbot应用的典型例子：跨设备、跨平台的通知流程。  
做法是把用户行为/系统状态变化建模成状态机，然后检查在各种可能的事件顺序下，系统是否始终满足预期的性质（比如不会丢通知、不会出现不一致状态等）。适合发现“并发/时序”类问题。  
  
Lean验证：  
主要用于对具体算法做形式化证明。  
gorkbot应用的典型例子：他们自己写的一个虚拟化库（virtualization library，用于长列表只渲染可见项）。用 Lean 证明这个算法不会出现闪烁（flicker）和抖动（jitter）问题。  
  
TLA+：官方有《Specifying Systems》和一些不错的入门教程。  
Lean：可以看《Theorem Proving in Lean 4》或社区的数学形式化项目。

> ![Link card preview](<头雁%20(@alacheng).assets/image-01.jpg>)
> 
> 🔗 [**All Code Is Slop (Except Mine) — Lauren Tan, SpaceXSI**](https://t.co/s6wrMTL5va)
> 
> _From youtube.com_

---

@poteto 🥔土豆姐的x账号在这里，可以关注下。