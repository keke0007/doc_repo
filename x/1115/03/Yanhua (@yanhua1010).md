---
author: "Yanhua"
handle: "@yanhua1010"
source: "https://x.com/yanhua1010/status/2081727890912317614"
date: 2026-07-27T13:06:20.000Z
type: thread
likes: 648
reposts: 61
replies: 51
bookmarks: 660
views: 62065
---
# Yanhua (@yanhua1010)

pi 应该是目前最好的 agent 学习案例：78k star，TypeScript，架构分层极清晰，整个 agent loop 就一个文件 793 行。  
  
我准备开一个“Agent 内核拆解”系列，把 pi / codex / grok-build 三个工业级 coding agent 的内核拆干净，顺便从零写一个能跑的 mini-agent。  
  
欢迎关注。

---

三个仓库地址:  
[github.com/earendil-works…](https://github.com/earendil-works/pi)  
[github.com/xai-org/grok-b…](https://github.com/xai-org/grok-build)  
[github.com/openai/codex](https://github.com/openai/codex)

> ![Link card preview](<Yanhua%20(@yanhua1010).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - earendil-works/pi: AI agent toolkit: unified LLM API, agent loop, TUI, coding agent CLI**](https://t.co/E6VOaQVhCK)
> 
> _From github.com_

---

Pi拆解系列第一篇

> **Yanhua (@yanhua1010)**
> 
> ![Article cover](<Yanhua%20(@yanhua1010).assets/image-02.jpg>)
> 
> 📝 **Agent 的本质是一个 while 循环：拆解 pi 的 792 行核心源码**
> 
> 这是【Agent 内核拆解】系列文章的第 1 篇。这个系列我会对照三个开源 coding agent：OpenAI codex（101k star）、xAI grok-build（23k star）、earendil pi（78k...