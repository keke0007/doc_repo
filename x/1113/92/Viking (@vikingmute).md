---
author: "Viking"
handle: "@vikingmute"
source: "https://x.com/vikingmute/status/2119675476725924171"
date: 2026-11-08T12:27:18.000Z
type: tweet
likes: 84
reposts: 9
replies: 18
bookmarks: 84
views: 7740
---
# Viking (@vikingmute)

几天文章中很好的最佳实践，有些在他之前的文章中说过，我觉得再看一遍还有收获：  
  
1 让 AI 有明确的验证机制：明确算是验证通过，不要相信 agent 自己说做完了，让它提供新的 E2E case 的通过流程，截图甚至是录屏，让它自己能跑检查，直到通过。  
  
2 设计完全自动化流程，不需要任何的人类的干预，设定这样一个循环：  
2-1 定义什么叫做完了  
2-2 明确自验证机制（和上一条正好呼应）  
2-3 确定边界  
2-4 确定约束（比如运行多少轮，最大花费的 token，时长等等）  
2-5 确定 review 流程  
  
3 把重复的错误积累到文件中，这个非常简单但是很实用，上下文一长我发现 agent 经常会出现同样的问题，假如你手动纠正它两次，那么就把对应的问题持久化到 agents.md 文件中，下次就不用手动去纠正了。

> **Viking (@vikingmute)**
> 
> Addy Osmani 的 新文章《Your Coding agents made you a manager》 认真读完了  
>   
> 现在工程师做的事情已经慢慢像管理转变了，要使用好 Agents ，和当一个好的工程经理几乎是一样，曾经对 manager 那些抱怨，现在回到了我们自己身上，在 AI x.com/addyosmani/sta…