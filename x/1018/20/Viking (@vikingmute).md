---
author: "Viking"
handle: "@vikingmute"
source: "https://x.com/vikingmute/status/2111302797794885999"
date: 2026-10-17T03:46:27.000Z
type: tweet
likes: 344
reposts: 81
replies: 19
bookmarks: 478
views: 25027
---
# Viking (@vikingmute)

昨天还在说 Code Review 的实践，今天Anthropic就写了一篇文章详细描述它们 Code Review 的工程实践：  
[claude.dev/blog/solving-c…](https://claude.dev/blog/solving-code-review-at-anthropic/)  
  
非常受用，可以好好读一下，核心观点是：代码多到人读不过来之后，人已经不 review 代码了，而是追求花最少的时间 并且回滚和事故又不增加  
  
一个改动要过七层检查，几点也很有趣：  
  
1 让 Agent 提 PR 之前先自证 证明做完了 测试 截图 等等  
  
2 每个 PR 都有 AI review diff，对照整个仓库和 PR 的意图来审 多个 agent 同时找 bug 再由另一组投票，多数认定是真问题才发出来 纯风格类的意见直接丢掉 这个和我的 review-forge 的思路非常类似  
  
3 按风险决定要几个人批 低风险不用人批准自动合并，普通功能一人批 最高风险的路径要两个人（我怀疑这步骤真的那么有用吗？人能看过来吗）  
  
4 merge queue / feature flag / 灰度发布 先给内部员工用， 快速发现 快速回 feature flag 不用部署就能关掉  
  
里面还有一个非常实用的思路 正好解答昨天多 agent review 的问题：  
  
review 要能收敛，agent 写 agent 审，有可能可以互相折腾个没完，文中说有个 PR 一夜之间来回改了五十多轮 git blame 一看，59% 被反复标记的都是第一轮时就已经存在的老代码  
  
所以他们加了几条规则：最多三轮，超过就让作者退一步重新想方案 几轮之后只审新增的 diff 已经提过的意见不再重复。

> ![Link card preview](<Viking%20(@vikingmute).assets/image-01.png>)
> 
> 🔗 [**How the Claude Code team does code review / claude.dev Blog**](https://t.co/gQ2hY9XqIr)
> 
> _From claude.dev_