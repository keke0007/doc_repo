---
author: "Cloris"
handle: "@ClorisSignal"
source: "https://x.com/ClorisSignal/status/2121705509187358779"
date: 2026-11-14T18:36:10.000Z
type: tweet
likes: 116
reposts: 9
replies: 14
bookmarks: 159
views: 12911
---
# Cloris (@ClorisSignal)

Lauren 在 AI Engineer Code Summit 的这场演讲的核心观点是：AI 写出低质量代码，问题往往不在模型本身，而在它工作的环境。 只要把代码库约束和自动验证做扎实，就能在大幅提高产出的同时，不让质量下滑。  
  
她的几个论点很有启发：  
  
- 工程师的核心任务变了。 现在的关键是在产出大幅增加的同时守住、甚至提升代码质量，目标是从“人逐行审查 AI 代码”走向可以放心交给 AI 的“软件工厂”。  
  
- 代码库就是 AI 的记忆。 AI 会模仿代码库里已有的写法，所以临时补丁、废弃写法会像杂草一样迅速蔓延。她把工程师的工作比作打理花园。  
  
- 建立信任的优先级。 最可靠也最划算的是代码库本身（API 设计、架构、重构），其次是类型检查和 lint 等静态分析，再往后是运行时验证和形式化验证，然后才是写成文字的规则，人工审查排在最后。  
  
- 需要分寸感的写法，最容易让 AI 写出烂代码。 与其写文档教 AI 把握分寸，不如直接改 API，把容易出错的写法从根上消灭。例如她的团队在产品代码里禁用了 React 的 useEffect，并禁止 Electron 主进程和渲染进程的目录互相引用。  
  
- 注释是隐患。 AI 常用注释为偷懒的临时补丁找理由，而注释无法被程序强制检查，所以团队禁掉了绝大部分注释。  
  
- 每纠正一次 AI，就用规则把它固定下来。 不只修好这一个 bug，而是加一条 lint 规则，或者在代码层面让同类问题不可能再出现。  
  
- 自动验证比人工审查更能扩展。 AI 让大规模测试变得便宜：运行时验证靠功能地图和专门驱动应用的命令行工具，形式化验证则用 TLA+ 检查用户流程、用 Lean 证明算法。  
  
她的团队一个月合并了约 11,000 个 PR，改为每日发布，回滚率稳定在约 0.47%，启动速度、输入延迟等性能指标反而有所改善。  
  
她还展望了一个“外层循环”：由机器人调度云端 AI 去测试应用、发现 bug、提交修复并自动合并。

> **lauren (@poteto)**
> 
> my AIE talk is up! come hear how the grok @bot team ships so quickly without compromise on quality  
>   
> https://youtu.be/O_zcc6Xl12g