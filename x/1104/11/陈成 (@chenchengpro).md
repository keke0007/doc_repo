---
author: "陈成"
handle: "@chenchengpro"
source: "https://x.com/chenchengpro/status/2117452147822673956"
date: 2026-11-03T03:01:47.000Z
type: tweet
likes: 90
reposts: 11
replies: 9
bookmarks: 104
views: 8447
---
# 陈成 (@chenchengpro)

recommended reading.  
  
不错的 code mode 科普文，就是「只剩一个 Tool」有点标题党了。  
  
tool call 的演进过程是这样的。早期只给几个工具，因为模型在大目录里选不准；后来几十个工具加一堆 MCP server，上下文撑不住，于是有了 tool search 和渐进披露。剩下的问题是一轮基本只能做一件事（最多一批互不依赖的调用），每个结果都要回到 history 里，模型再想一次才能走下一步。Code Mode 把所有工具收成一个  exec(code) ，模型写一段 TS 或 Python，在沙箱里把调用串起来跑。  
  
不过文章没有讲失败恢复，评论区聊的比较多的也是这个。感觉 Pi Durable 的确定性重放可能是其中一个解。

> ![Link card preview](<陈成%20(@chenchengpro).assets/image-01.jpg>)
> 
> 🔗 [**Why Agents Are Soon Only Going to Have 1 Tool**](https://t.co/0Uv2bdjQ8u)
> 
> _From philschmid.de_