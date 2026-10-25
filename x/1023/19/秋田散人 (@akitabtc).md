---
author: "秋田散人"
handle: "@akitabtc"
source: "https://x.com/akitabtc/status/2113995848975589651"
date: 2026-10-24T14:07:41.000Z
type: tweet
likes: 32
reposts: 3
replies: 16
bookmarks: 28
views: 4066
---
# 秋田散人 (@akitabtc)

Agent PR 审计无法收敛，尤其是你用 GPT/codex 这种防御性模型做审计的情况下，一定要限制代码审计的轮数，否则可能没完没了，几十轮下来，PR 越来越大不说，token 也会耗干净（如果你用 API credit 睡一觉起来就等着收恐怖账单吧）。  
  
但是限制次数就常需要人工介入，会影响到整个系统的自动化演进。所以最好的方案其实是一个软闸一个硬闸，比如 3-6 次，3 次以后主 agent 开始敦促收敛，并且作为调停方/医生介入，把一些无关紧要的找茬内容放到 follow-up 下一个 PR 之类的。  
  
无论哪个时代，控制 PR 内容的原子性，都是控制软件质量的关键。

![Image](<秋田散人%20(@akitabtc).assets/image-01.jpg>)