---
author: "纳米AI"
handle: "@antiAIvo"
source: "https://x.com/antiAIvo/status/2116158775694647609"
date: 2026-10-30T13:22:23.000Z
type: tweet
likes: 43
reposts: 4
replies: 6
bookmarks: 23
views: 4127
---
# 纳米AI (@antiAIvo)

给「从零手写 Agent」这个项目  
做了一个本地网页端，接入Deepseek API key  
用一个小例子演示一下  
  
输入提示词： "在工作区写一个 [hello.py](http://hello.py)，打印一句话，然后运行它"  
  
Agent 自己走完了三步：  
1. 调用 write_file 写好 [hello.py](http://hello.py)，弹窗等我确认  
2. 调用 run_shell 运行 python3 [hello.py](http://hello.py)，再次弹窗确认  
3. 拿到输出"Hello, 纳米!"，汇报结果  
  
页面显示设计分左右两块：  
左边是agent实时每一步做了什么  
右边是每次发给模型的完整上下文  
  
[github.com/aa1143/NamiAge…](http://github.com/aa1143/NamiAgent)

![🎥 Video](<纳米AI%20(@antiAIvo).assets/image-01.jpg>)