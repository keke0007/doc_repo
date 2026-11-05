---
author: "陈成"
handle: "@chenchengpro"
source: "https://x.com/chenchengpro/status/2116059979757322407"
date: 2026-10-29T17:01:00.000Z
type: tweet
likes: 69
reposts: 10
replies: 6
bookmarks: 109
views: 9147
---
# 陈成 (@chenchengpro)

recommended reading.  
  
1/ 作者把 review 分为 客观 review 和主观 review。客观 review 「管代码对不对」，包括 bug、安全漏洞、边界情况等，给好 prompt 的话，agent 做客观 review 已经比人强。主观 review 管「是不是该写这段代码」，这个只需要扫一眼，看方向对不对，不逐行读，也不找 bug。  
  
2/ 主观 review 看两个问题，scope 和 Direction。scope 是看 agent 有没有改不该改的文件，因为 agent 常常越界，让改一处，它把无关文件甚至整个项目都动了。Direction 看方向对不对，比如用现成 npm 包还是自己造轮子，比如用 mygo 还是 nsgo，都是判断题，体现人的品味。  
  
3/ 主客观的边界会移动。主观品味一旦写下来，agent 就能自己检查，也就从主观变成了客观。  
  
4/ 所以，目前人不会被移出 review。基本思路是把客观 review 交给 agent，主观 review 留给人，然后把主观能写下来就变客观，逐渐减少人的参与。

> **Hayden (@haydenbleasel)**
> 
> Two kinds of code review. https://haydenbleasel.com/writing/two-kinds-of-code-review…