---
author: "jiangkoumo"
handle: "@jiangkoumo_"
source: "https://x.com/jiangkoumo_/status/2113537635956449291"
date: 2026-10-23T07:46:54.000Z
type: tweet
likes: 122
reposts: 15
replies: 14
bookmarks: 155
views: 8002
---
# jiangkoumo (@jiangkoumo_)

我把 Cursor 工程师 Lauren Tan（@poteto）的工程工作流 pstack 移植成了 Pi 原生包 @jiangkoumo/pi-pstack  
  
它给 Pi 补了一套完整的工程流程：调查、修 bug、加功能、重构各有对应流程  
  
多代理模式下，代码在单独的工作目录（worktree）里生成补丁，审阅 Agent 只有只读权限。独立审阅和命令验收都通过，才应用补丁；commit 和 push 始终由人工确认  
  
并且使用 p-queue 真实 issue #177 实测，全程用 DeepSeek flash，304 个测试全过  
  
欢迎来试用一下  
  
pi install npm:@jiangkoumo/pi-pstack

![Image](<jiangkoumo%20(@jiangkoumo_).assets/image-01.jpg>)