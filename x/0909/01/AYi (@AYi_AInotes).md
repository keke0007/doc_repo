---
author: "AYi"
handle: "@AYi_AInotes"
source: "https://x.com/AYi_AInotes/status/2097857102190403825"
date: 2026-09-09T17:02:13.000Z
type: thread
likes: 329
reposts: 54
replies: 47
bookmarks: 520
views: 66905
---
# AYi (@AYi_AInotes)

前 Cursor 核心架构师、现 xAI 工程师 poteto 刚发了句暴论，直接把全网的“计划模式”抽得粉碎：  
“别再对着 Agent 抠什么长篇大论的抽象计划了。  
又长又漂亮的 Plan，除了给你制造一种‘我们推进得很高效’的虚假幻觉，本质上全是一堆没有着陆点的泡沫。”  
  
这篇长文的标题极其性感，直接戳中了 2026 年所有开发者的痛处：  
《如何管理一个比你还聪明的存在（Agent）？》  
  
她在 Meta、Netflix、Cursor 摸爬滚打了上千万行代码，给出的工业级答案刀刀见肉：  
  
很多人用 Agent 喜欢搞对抗性评审（Adversarial Review），  
让 AI 写出 10 页纸的架构预案，人类在文字层面逐条挑刺；  
但残酷的事实是：你一旦让 Agent 在抽象层面过度推演，它就会开始对着空气疯狂幻觉，编造出一堆现实中根本不会发生的伪边缘场景去自我防御。  
  
真正的顶级工程做法只有一条：  
别在计划还停留在抽象层时把它“煮得太熟”（Don't overcook abstract plans），  
直接用代码原型（Prototyping with code）去约束它。  
  
让它别光打字，直接去写最小可运行原型；  
让它自己用测试工具（Verification Skills）跑一遍，把录屏、截图和报错事实拍在桌上自证；  
人类别去当咬文嚼字的文字考官，人类唯一的职责，是给它装上那个“能判断它有没有说谎的物理地基”。  
  
从 Prompting 到 Planning，很多人以为这是终点，  
其实这两层全是在文字游戏里打转；  
真正的工业分水岭叫 Verification（确定性核验）——  
能用真实代码自证结果的 Agent，才是那台能放心地跑在生产线上的发动机。

![Image](<AYi%20(@AYi_AInotes).assets/image-01.jpg>)

> **lauren (@poteto)**
> 
> ![Article cover](<AYi%20(@AYi_AInotes).assets/image-02.jpg>)
> 
> 📝 **The Complete Guide to pstack Pt. 2**
> 
> In my last post, I showed you why verification is the foundation of everything I do with agents, and how to get started creating your own verification skills. The key lesson was that if an agent can't...
> 
> Part 2 is here! Let's learn the art of supervising someone smarter than you.

---

poteto 在长文里最值钱的一句话摘录：  
“Abstract plans only give you the illusion of progress.”（抽象的计划只会给你带来进展的虚假幻觉。）  
  
她开源的 pstack 体系里，把这套流程规范成了：  
1. 彻底禁用纯文本的计划审查，全面切入 /prototype 模式；  
2. 每一个任务在交付前，必须附带自动化测试

![Image](<AYi%20(@AYi_AInotes).assets/image-03.jpg>)