---
author: "ThirdBro"
handle: "@dngzsn37461"
source: "https://x.com/dngzsn37461/status/2122239403796185300"
date: 2026-11-16T08:04:37.000Z
type: tweet
likes: 28
reposts: 3
replies: 2
bookmarks: 41
views: 9706
---
# ThirdBro (@dngzsn37461)

同一个星期，agent harness 赛道上的两个选手都更新了。  
  
DeepSeek 的 DSH，Earendil 的 Pi。  
  
讲一个两个框架的不同点，上下文记忆。  
  
Pi 把会话存成一棵树。每条消息都是树上的节点，随时可以回到任何一个节点分叉重试。被你放弃的分支不删，切走的时候它还会把那条分支总结成摘要，贴在你现在这条上。上下文快满了自动压缩，旧历史变摘要，原文永远留着。  
  
DSH 走的是事件溯源。所有东西进一条只追加的 JSONL 日志，默认 zstd 压缩带校验和。最狠的是崩溃恢复。进程跑到一半死了，它不扔残轮，而是给没答完的调用补一个错误结果，重启接着干。