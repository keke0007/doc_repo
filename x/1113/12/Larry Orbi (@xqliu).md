---
author: "Larry  Orbi"
handle: "@xqliu"
source: "https://x.com/xqliu/status/2121186811510690228"
date: 2026-11-13T10:22:00.000Z
type: thread
likes: 4
replies: 5
bookmarks: 3
views: 847
---
# Larry  Orbi (@xqliu)

家里装了 Home Assistant 的，现在可以把设备配置和自动化交给 Claude、ChatGPT 这类 AI 助手了。有人专门写了个 MCP 服务，叫 ha-mcp。  
  
仓库去年 9 月建的，11 月 11 号刚发新版，今天一天涨了 714 星，总数 7687。  
README 徽章上标着 87 个工具。看这个数量，作者想交给 AI 的应该是整套配置和自动化，不只是开关几盏灯。演示是在 Claude Desktop 里跑的。  
  
已经在用的，升级前留意：v7.3.0 有个不兼容改动，直接写 YAML 配置的那个工具挪进了 beta。之前靠它改配置的，升级后先去翻一下 beta 文档。  
  
项目用 Python 写，基于 FastMCP，MIT 协议，仓库里还挂着端到端测试的徽章。  
  
家里自动化规则越攒越多的，挑一条现成的，让 AI 改改看。

---

ha-mcp 仓库，README 里有演示和文档入口

> ![Link card preview](<Larry%20Orbi%20(@xqliu).assets/image-01.png>)
> 
> 🔗 [**GitHub - homeassistant-ai/ha-mcp: Configure and automate Home Assistant with Claude, ChatGPT, and...**](https://t.co/e3NdYbvHCz)
> 
> _From github.com_

---

想让 AI 动手又说不清要改什么时，Orbi Cloud 有个「写需求」页：你用对话描述，Orbi 先读仓库，帮你起草 Issue，没说清的地方会停下来问你，你确认了才提交。

> ![Link card preview](<Larry%20Orbi%20(@xqliu).assets/image-02.jpg>)
> 
> 🔗 [**Orbi — 软件黑灯工厂，开源（AGPL-3.0）：提个 Issue，收个版本**](https://t.co/RmKzfTCZOz)
> 
> _From orbi.build_