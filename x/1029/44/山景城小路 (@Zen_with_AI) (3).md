---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2114752264674201841"
date: 2026-10-26T16:13:24.000Z
type: tweet
likes: 50
reposts: 5
replies: 7
bookmarks: 34
views: 4675
---
# 山景城小路 (@Zen_with_AI)

Claude Code 团队怎么做代码评审 --这是所有程序员都需要学习的  
  
Claude Code 团队公开了他们怎么做代码评审，值得所有用 agent 写代码的人看。  
  
先说规模：Claude 写了 Claude Code 约 90% 的代码；单体仓每天合并的改动从 5 月的 2500 个涨到 9 月的 3 万多个，而需要回滚的比例反而降了 40% 以上。  
  
他们的做法是把评审拆成六层：  
PR 前先让 Claude 跑测试、截图验证，再由子 agent 换全新上下文审一遍；  
每个 PR 必过 AI 评审和安全评审；  
人的审批按风险分级，低风险可以只留一个人在环里。  
衡量标准也变了：不看有没有人读过代码，看什么检查能抓住这个改动可能出的问题。