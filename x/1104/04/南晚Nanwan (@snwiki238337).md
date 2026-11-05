---
author: "南晚Nanwan"
handle: "@snwiki238337"
source: "https://x.com/snwiki238337/status/2117445911794061742"
date: 2026-11-03T02:37:00.000Z
type: tweet
likes: 5
reposts: 1
replies: 5
bookmarks: 5
views: 489
---
# 南晚Nanwan (@snwiki238337)

LangChain 工程团队最近在官方博客发了一篇非常有实操价值的技术文章，题为Browser Use in Managed Deep Agents。  
  
切中了现在做智能体落地的一个问题就是世界交互。  
  
现实世界里绝大多数高频的业务场景根本没有现成的开放 API 可调，比如退订服务、预订机票或者在没有提供接口的企业内网系统里办事。  
  
LangChain 这次直接在他们托管的 Managed Deep Agents 里内置了一套完整的云端浏览器运行环境。  
  
简单来说的话其实就是给每个 Agent 配了一个独立的远程沙箱浏览器，让智能体能够像真人一样去识别网页渲染出的真实内容。  
  
此外LangChain 在底层的设计非常讨巧，他们把浏览器放在一个独立的远端沙箱里。  
  
工具层基于 Browserbase 开源的 Stagehand 做了深度调优，不传整页 DOM 树，而是通过无障碍树给交互元素动态标上数字 ID，并支持通过一次批处理代码调用把一连串点击和填表全部打包跑完。  
  
在真实的 WebVoyager 基准测试里做完 100 个日常任务只花了 1.98 美元，通过率达到了 90%。  
  
针对登录难关，他们做了一个被称为人机交接的机制。  
  
当 Agent 撞见复杂的双重验证或登录界面时，它会自动在 Slack 或调试界面里弹出一张内嵌卡片把控制权抛给人类。  
  
人类在里面手动输完密码点个交回，Agent 就能在已经登录好的会话里继续执行，甚至支持按用户维度复用 Cookie，避免每次开新任务都得重新认证。  
  
原文：

> ![Link card preview](<南晚Nanwan%20(@snwiki238337).assets/image-01.jpg>)
> 
> 🔗 [**Browser Use in Managed Deep Agents**](https://t.co/ZSaBgoEIPW)
> 
> _From langchain.com_