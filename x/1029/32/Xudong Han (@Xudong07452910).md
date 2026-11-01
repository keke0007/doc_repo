---
author: "Xudong Han"
handle: "@Xudong07452910"
source: "https://x.com/Xudong07452910/status/2114876984304263464"
date: 2026-10-27T00:29:00.000Z
type: tweet
likes: 79
reposts: 13
replies: 10
bookmarks: 68
views: 5678
---
# Xudong Han (@Xudong07452910)

最近 LangChain 分享了内部销售、研发和内容团队的 Agent Harness 架构，里面有不少值得借鉴的工程细节。  
  
他们发现，最棘手的问题之一是「Agent 到底以谁的身份干活」。  
  
比如销售 Agent 查询 Salesforce 可以用公司统一账号，但发邮件必须使用用户自己的授权，而且只能在私聊中操作。  
  
Coding Agent 也一样。帮你提交 PR 时，用你的 GitHub 身份；在后台自动审查代码时，则切换成独立的 GitHub App。  
  
内容 Agent 发公司 Twitter，用的是公司账号，但只有白名单用户能真正发帖。  
  
除此之外，还有几项关键设计：  
  
1. Agent 直接接入 Slack，不用员工换平台；需要时还能从工讨论里「拆」出独立线程  
2. Memory 分全局和个人两层，由 Agent 自己决定一条事实记到哪一层  
3. 代码在沙箱里执行，经安全代理访问外部服务  
4. 大量任务是定时或事件触发的，不一定等人开口。比如新 PR 自动 Review，或者你跟它说一声，它自己挂上每天销售简报  
  
Demo 阶段跑得通的 Agent，一进公司就会撞上身份和权限边界。这些细节平时讨论不多，但真落地一个都绕不过去。  
  
这些细节平时讨论得不算多，但真到落地时，一个都绕不过去。推荐做企业 Agent 的朋友读读。  
  
原文：

> ![Link card preview](<Xudong%20Han%20(@Xudong07452910).assets/image-01.png>)
> 
> 🔗 [**The five components of building a "company harness"**](https://t.co/4nUIFAJkzG)
> 
> _From langchain.com_