---
author: "yibie"
handle: "@yibie"
source: "https://x.com/yibie/status/2121154006978621742"
date: 2026-11-13T08:11:39.000Z
type: thread
likes: 21
reposts: 2
replies: 4
bookmarks: 24
views: 1033
---
# yibie (@yibie)

把大模型的本事写成 Skill 给小模型用  
  
想让便宜的小模型替代昂贵的大模型，通常的办法是微调，要准备训练数据、再跑一次训练，换模型还得重来。Zep 发布的 Agent Skills 换了一个做法：不动模型，把大模型跑通任务的步骤编译成一个 SKILL.md 文件，小模型照着做。  
  
具体是五步。先用 Opus 5.5 当老师，跑通一个小模型自己做不对的多步任务。然后只保留被外部验证过成功的运行，比如工具确认或评估系统打分，agent 自己说成功不算。编译器再把这些运行整理成 SKILL.md，每条说明都指回具体的运行事件，账户名这类实例数据替换成占位符。小模型运行时搜索并加载这个文件，照着执行。之后每一次成功和失败都会编译成新版本，验证过的用法提高检索排名。  
  
Zep 在自己的 EnterpriseOps-Gym 基准上测的结果是，Haiku 5.5 加 Skill 后任务完成率从 33.9% 升到 45.3%，DeepSeek V4.1 Flash 从 48.3% 升到 56.8%，反而高于 Opus 5.5 全力跑的 52.8%。在长任务较多的 τ³-bench banking 上，每通过一个任务的 token 用量最多减少 45%。这些都是 Zep 自测的数字。  
  
适用范围也有限。编译器只从验证过的成功里学，还没有任何 agent 解决过的任务不会产生 Skill；模型本来就做得好的任务，加上去收益也小。比较合适的，是那种 agent 会反复跑、小模型单独跑又老出错的多步流程。  
  
原文链接在评论区  
#小技巧

![Image](<yibie%20(@yibie).assets/image-01.jpg>)

---

原文：  
[getzep.com/blog/introduci…](https://getzep.com/blog/introducing-zep-agent-skills)

> **Santiago (@svpino)**
> 
> How to replace a large, expensive, and slow model with a small, cheaper, fast model and still get the same results.  
>   
> 7x faster and 50% fewer tokens.  
>   
> This is like fine-tuning, but without touching the model.  
>   
> Here, we teach Haiku, using Zep Agent Skills, to do what Opus 5.5 does
> 
> ![🎥 Video](<yibie%20(@yibie).assets/image-02.jpg>)