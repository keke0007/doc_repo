---
author: "嗯哌AI"
handle: "@NpieAI"
source: "https://x.com/NpieAI/status/2121455376604471586"
date: 2026-11-14T04:09:11.000Z
type: thread
---
# 嗯哌AI (@NpieAI)

同一个代码评审，token 从 460 万降到 50 万，时间从 12.8 分钟砍到 6.6 分钟。  
  
问题不在模型，在技能的用法。我把重复流程写成技能，agent 每次跑都要重新读一遍技能、重新决定每一步该怎么走。等于每跑一次，都要花 token 把流程从头想一遍。  
  
Swamp 的办法是反过来的：让写代码的 agent 把重复的技能一次性搭成工作流，搭好之后反复跑。  
  
每个步骤的输入和输出都是事先定好的，下一步走哪条由代码决定，不由模型临时想。模型只在真正需要判断的那几步才出场。  
  
Swamp 官方公布的一组实测是这样：同一个代码评审技能，在 Codex 里跑要花约 460 万 token、23 个 agent、12.8 分钟；改成 Swamp 工作流后，约 50 万 token、3 个 agent、6.6 分钟。  
  
省得最多的一块，是把协调这活儿从 agent 手里交给了代码。  
  
流程跑第二遍的时候，就不该再花 token 想流程——判断请模型，流程交给代码。  
  
我的做法是这样：  
第一，新任务第一次跑，照常用技能，先跑通。  
  
第二，同一个任务跑到第二遍，就让 agent 把它改成工作流。  
  
第三，改的时候先问三个问题：哪几步真的需要模型判断？上周那次跑到底发生了什么？需要判断的那一步，换个便宜模型行不行？  
  
一句话：技能是草稿，工作流才是定稿。  
  
你手里现在跑得最勤、每次都要重新讲一遍的那个流程，是哪一个？

![Image](<嗯哌AI%20(@NpieAI).assets/image-01.jpg>)
![Image](<嗯哌AI%20(@NpieAI).assets/image-02.jpg>)
![Image](<嗯哌AI%20(@NpieAI).assets/image-03.jpg>)
![Image](<嗯哌AI%20(@NpieAI).assets/image-04.jpg>)

---

源链接：

> 🔗 [**Swamp: The right way to build with AI**](https://t.co/SFZo6B9bKa)
> 
> Swamp is the right way to build with AI: compose repeatable behaviors, not token-hungry agents. Any harness, any API, secrets in vaults. Free for solo use.
> 
> _From swamp-club.com_