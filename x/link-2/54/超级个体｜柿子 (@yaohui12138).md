---
author: "超级个体｜柿子"
handle: "@yaohui12138"
source: "https://x.com/yaohui12138/status/2056195833830539768"
date: 2026-05-18T02:11:03.000Z
type: tweet
---
# 超级个体｜柿子 (@yaohui12138)

Karpathy 发布了一个github开源项目，狠狠让我惊艳到了  
  
这个项目叫 andrej-karpathy-skills,GitHub 13 万+ star,我愿称之为2026 最有用的 AI 工程项目  
  
它解决的问题极其精准：让 Claude Code 不再瞎写代码  
  
这个项目到底有多厉害？  
  
先说结论：一个 4KB 的文本文件，让 AI 写代码的错误率暴降 90%  
  
Karpathy 自己说，他现在 80% 的代码都让 Claude 写，但 AI 经常犯几个典型错误：  
  
不问就瞎猜需求  
过度设计，写一堆用不上的抽象  
改 A 顺手把 B、C、D 也重构了  
代码能跑就行，不管目标达成没有  
  
这个项目就是专门给 Claude Code 戴上guardrails，用 4 条行为准则约束 AI 的编码行为  
  
核心亮点：4 条准则改变一切  
  
整个项目就是一个 CLAUDE.md 文件，里面只有 4 条规则，但每一条都直击 AI 编码的痛点  
  
1. Think Before Coding - 先思考再动手  
  
AI 最大的问题是“太听话”，你说啥它就写啥，从不质疑  
  
这条准则要求：明确说明假设、权衡 tradeoffs，不确定就直接问  
  
不再是“我猜你想要这个”，而是“我理解你的需求是 A，但 B 方案可能更合适，你要哪个？”  
  
2. Simplicity First - 极简实现优先  
  
AI 天生爱炫技，你要一个登录功能，它给你写个完整的 OAuth 2.0 + JWT + 刷新令牌 + 权限系统  
  
这条准则强制：只写刚好能解决当前问题的最小代码  
  
不搞 speculative abstractions，不写未来功能，不过度工程  
  
一个用户反馈：用了这条规则后，代码 diff 从动辄几百行缩减到几十行，review 轻松太多  
  
3. Surgical Changes - 手术式精准修改  
  
这是我最爱的一条  
  
AI 有个恶习：你让它改个 bug，它顺手把整个文件的命名风格、缩进、注释全优化了  
  
这条准则要求：只改用户要求的部分，严格匹配原有代码风格  
  
不碰无关文件，不顺手 refactor，不加“看起来更好”的改动  
  
有开发者实测：启用这条后，git diff 从“满屏红绿”变成“3 行精准修改”  
  
4. Goal-Driven Execution - 目标驱动执行  
  
AI 经常写完代码就交差，但代码能跑 ≠ 任务完成  
  
这条准则要求：把任务转化为可验证的目标/测试/成功标准，然后 loop 执行、验证、迭代  
  
直到真正达成目标才停止  
  
这让 AI 从“代码生成器”变成“问题解决者”  
  
真实效果：社区反馈炸裂  
  
X 上这个项目刷屏了，开发者反馈高度一致：  
  
代码质量飞跃：diff 更紧凑、干净，overbuild 和 side changes 大幅减少  
  
错误率暴降：有人实测从 41% 掉到 11%，继续优化后低至 3%  
  
中文社区评价：“必备 skills”“Claude/Cursor 实用技能 Top1”“直接扔项目里就完事了”  
  
很多人直接 @ 朋友推荐：“把这个 CLAUDE.md 放进去，Claude 立刻像换了个人，写代码更靠谱、不乱改、不瞎猜“  
  
使用方式：简单到离谱  
  
这是我见过最简单的 AI 工程优化方案：  
  
方法一：直接 curl 把 CLAUDE.md 下载到项目根目录  
  
curl -o CLAUDE.md [raw.githubusercontent.com/forrestchang/a…](https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md)  
  
方法二：用 Claude Code / Cursor 的 plugin 一键安装  
  
支持 Claude Code、Cursor 等主流 AI coding 工具  
  
完全开源（MIT 协议），拿来就用  
  
作为产品经理出身的开发者，我看到的不只是 4 条规则，而是对 AI 编码行为的深刻洞察  
  
Karpathy 做的事情本质上是：给 AI 建立编码的第一性原理，他把他对于AI编程的理解写入了文件中  
  
不是教 AI 怎么写代码（它已经会了），而是教 AI 什么时候该问、什么时候该停、什么时候该简化  
  
这 4 条准则就像产品经理给开发团队定的 PRD 原则：  
  
需求不清楚？ 先问  
功能够用就行？ 别过度设计  
改需求？ 只改需求相关的  
做完了？ 先验证目标达成没有  
  
它能帮到我们什么？  
  
如果你是独立开发者或小团队，这个项目能直接提升你的 AI 协作效率：  
  
减少返工：AI 不再瞎猜需求，写出来的代码更符合预期  
  
降低 review 成本：改动精准，不用在一堆无关修改里找真正的变更  
  
提升代码质量：极简实现意味着更少的 bug、更好的可维护性  
  
加速迭代：目标驱动让 AI 真正解决问题，而不是生成代码  
  
对于中大型项目，这是让 Claude Code 真正“生产可用”的关键一步  
  
我的使用体会  
  
我在自己的几个项目里部署了这个 CLAUDE.md，最直观的感受是：  
  
AI 变聪明了  
  
以前它是个听话的实习生，你说啥它做啥，经常做错  
  
现在它像个有经验的同事，会主动问“你确定要这么做吗？ 我有个更简单的方案”  
  
代码 diff 变干净了  
  
以前一个小需求能改几十个文件，现在精准到只改 3-5 行  
  
我的工作重心变了  
  
以前 60% 时间在 review AI 的代码、修 bug  
  
现在 80% 时间在思考产品逻辑，AI 真正成了生产力工具  
  
最后  
  
这个项目被誉为 2026 年 AI coding 领域的“现象级”黑魔法工具  
  
小文件，大作用  
  
如果你在用 Claude Code / Cursor 写代码，强烈建议直接把这个 CLAUDE.md 扔进项目根目录  
  
GitHub 地址：[github.com/forrestchang/a…](https://github.com/forrestchang/andrej-karpathy-skills)  
  
试过的人基本都是“已全项目部署”的状态  
  
作为一个天天和 AI 协作的开发者，我的建议是：别犹豫，直接用