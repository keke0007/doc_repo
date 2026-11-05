---
author: "KunKun折腾手记"
handle: "@Karadokuy"
source: "https://x.com/Karadokuy/status/2116475425916694997"
date: 2026-10-31T10:20:38.000Z
type: tweet
likes: 174
reposts: 23
replies: 16
bookmarks: 214
views: 18531
---
# KunKun折腾手记 (@Karadokuy)

昨晚看了 @poteto 的推文，她有一句话说得很好：代码不再一滴一滴地来，而是一条连续奔涌的河。哀悼写代码的时代早就过去了，现在要做的是设计系统和护栏，让 Agent 在里面长好。  
  
具体到我自己的实践，认知是这样的：  
  
1. 先把测试条件给 AI 补齐。Agent 只能在它现有的条件下往前推，很多时候没做好，是我们没把条件给够。今晚 Switch 2 23.01 被破解，工程师做的第一件事也是把物理和软件环境全部搭好。做软件也一样，所有端、所有用户场景的设备和测试（E2E、功能测试）都要先配齐。  
  
2. 建一套防腐化的上下文系统。AI 写代码天然是熵增的，便宜的模型尤其抓不住意图，容易走弯路、走进死胡同。要有一个往前盯着的监控，再加上 MCP、skill、hook，把上下文管住。这样仓库被 AI 提交成百上千次 PR 之后，还能稳、不出错、不膨胀。  
  
非常推荐我自己一直在用的开源项目，@yetone 的 magpie。它是怎么做成一套自我递归、自我成长的系统的？也是这个思路：把条件给够，护栏建好，剩下的交给用户。每次修改之前，他都先让 AI 在沙盒里用真实账号把问题详细复现一遍，测试条件齐了才动手。所以它现在已经跑了 2000 多个版本号，每天都在飞快迭代，整套系统却越跑越稳，没有出现 Agent 腐化。

> ![Link card preview](<KunKun折腾手记%20(@Karadokuy).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - yetone/magpie: Every agent's model. One place. Codex on DeepSeek, Claude Code on Kimi,...**](https://t.co/G8IJmoiDGB)
> 
> _From github.com_