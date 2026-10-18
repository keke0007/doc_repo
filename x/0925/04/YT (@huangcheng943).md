---
author: "YT"
handle: "@huangcheng943"
source: "https://x.com/huangcheng943/status/2102695088401047920"
date: 2026-09-23T06:47:28.000Z
type: thread
---
# YT (@huangcheng943)

写一个小功能时，我以前常常先把实现堆出来，再补测试。速度看着快，出了边界 Bug 才发现，补上的测试全是照着现有代码猜答案。  
  
test-driven-development Skill 的顺序很死板，但确实有用：  
1. 先写一个只描述单一行为的测试，名字说人话；  
2. 运行，亲眼确认它因为功能缺失而失败，不是拼写错误；  
3. 只写让它转绿的最小实现；  
4. 重构之后再跑测试，仍然绿才开始下一个用例。  
  
修 Bug 时也一样：先让测试复现问题，然后再修。这样过几周回来看，这个 case 还在替你守着。  
  
调用模板：  
$test-driven-development 用 TDD 实现 <功能/修复>：先定义一个可观察行为，写最小失败测试；确认失败原因后再写最小实现并复跑全部相关测试。  
  
免费范围：这是流程型 Skill，本地测试可免费；CI、云构建或外部 API 仍会产生原本成本。第三方 Skill 先看 SKILL.md，别把可执行脚本和读取权限当成小事。  
  
接上上一帖：  
[https://x.com/huangcheng943/status/2102651043209203760…](/huangcheng943/status/2102651043209203760)  
  
#TDD #Testing #Codex #开发效率  
  
官方来源：[developers.openai.com/plugins/build/…](https://developers.openai.com/plugins/build/skills)

> **YT (@huangcheng943)**
> 
> Skill 用多了以后，最浪费时间的不是写流程，而是每次都把同一套约束重新讲一遍。  
>   
> 我准备把“发帖前核查事实来源、链接、字数与敏感信息”做成自己的 Skill：目录里先放 SKILL.md；开头只写 name 和 description。description x.com/huangcheng943/…

---

动手做新功能前，需求只有一句“做个待办清单”，怎样避免 AI 自行补出一堆你不想要的功能？obra（Jesse Vincent）维护的 Superpowers「brainstorming」是一份独立 SKILL.md：先弄清使用场景和成功标准，再复述理解，让人确认设计后才动手。适合把模糊想法练成可执行需求。  
  
Codex App 安装：侧边栏

> **YT (@huangcheng943)**
> 
> 测试全绿后，开发分支该合并、提 PR，还是先保留？obra（Jesse Vincent）维护的 Superpowers「finishing-a-development-branch」是一份独立 SKILL.md，专门把收尾决策变成可检查的步骤：重新跑完整测试、辨认普通仓库或 x.com/huangcheng943/…