---
author: "meng shao"
handle: "@shao__meng"
source: "https://x.com/shao__meng/status/2113095303973028269"
date: 2026-10-21T13:21:06.000Z
type: tweet
---
# meng shao (@shao__meng)

Claude Mods 深读：当 Harness 从“管机器”转向“管人的注意力”  
  
来自 @daniel_mac8 发布在 @latentspacepod 的文章，他认为：Claude Mods 让用户和 Claude 一起用代码改造 Claude Code 面向人的那一层，决定何时打断你、给你看什么、让你做哪些决定。Dan 认为这是 “注意力界面” 的早期形态，Harness 也可能由此演变成别人可以在上面做产品的平台。  
[latent.space/p/claude-mods](https://latent.space/p/claude-mods)  
  
Mods 是什么  
· 本质上是 Claude Code 插件：一组 TS/JS 函数，监听提示词、工具调用、界面渲染等事件，可以观察、改写渲染，或者替代默认行为。作者称之为“Harness 的中间件”，CLI 和桌面端都能用。  
· Claude Code 内置写 Mod 的 skill，用自然语言描述需求，Claude 就能帮你写出来。  
· 官方已经内置 6 个 Mod：agents-md、diff、plugin-authoring、sec-default、telemetry、you-should-know。也就是说，Claude Code 自身的一部分功能已经改用 Mod 实现。  
  
Dan 推荐的两个 Mod  
· You Should Know（官方内置，默认关闭）：一个旁路 agent 盯着主 agent 工作，把埋在执行记录里、但对结果影响大的信息标出来。实例是：主 agent 改文件时，它提醒作者有约 45 个文件存在未提交的改动，如果之后丢弃这次的演示修改，会连带把之前的工作一起丢掉。  
· Next Steps Supervisor（Dan 开源的 ModSmith 插件里的一个）：某一轮改了文件、跑了命令或调用工具 8 次以上时触发，列出需要关注的点，并把建议的下一步做成按钮，点一下就发给 Claude。  
  
与另外两家的对比：核心区别在于谁来设计注意力界面  
· ChatGPT Intelligent UI：每次回答由模型从 OpenAI 的组件库里挑选生成界面。零配置，但可控性很低。  
· Cursor PR Routing & Approval：团队写策略，固定流程来执行，包括按风险评分自动批准低风险 PR。有一条设计值得注意：如果某个 PR 修改了审批策略，系统按基础分支上的旧策略来审这个 PR，防止 agent 改规则给自己放行。  
· Claude Mods：用户和 agent 一起用代码搭界面本身，可以挂到循环的任意节点上。自由度最高，代价是界面好不好用要自己负责。  
  
Dan 自己指出的问题  
· 噪音：10 月 15 日装上之后，111 轮里有 99 轮触发了 Supervisor。提醒太频繁，人就开始跳过不看。  
· 成本：每次检查都要分叉会话、重读上下文。按 README 的估算，50k token 的会话每次检查约消耗 5–6k 等效输入 token，缓存过期后更多。  
· 信任：Mod 以用户权限运行，没有沙箱，能读写文件、看到所有提示词、消耗额度。内置 Mod 在 --safe-mode 下也照样运行。  
  
判断标准与方向  
Dan 衡量一个 Mod 有没有用，看三点：是否找出了你本会漏掉的东西，是否让决策更容易，是否让你能直接行动而不被更多输出分心。下一步的设想是让 Harness 自己读会话记录，发现你反复纠正 agent 或反复手动检查的地方，然后主动建议写一个 Mod。

> **Dan McAteer (@daniel_mac8)**
> 
> There's not enough buzz around Claude Mods.  
>   
> Read this Claude Mods article if you want to:  
>   
> - Understand Claude Mods basics  
> - Install and use the most useful mods today  
> - Ponder where the agent-human interface will go  
>   
> For @latentspacepod.