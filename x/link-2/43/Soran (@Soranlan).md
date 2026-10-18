---
author: "Soran"
handle: "@Soranlan"
source: "https://x.com/Soranlan/status/2057997594337612128"
date: 2026-05-21T20:33:50.000Z
type: thread
---
# Soran (@Soranlan)

[https://x.com/sweexx9/status/2057560520916414628/video/1…](/sweexx9/status/2057560520916414628/video/1)  
  
这个项目肯定会火，但一定要谨慎看。  
  
有人做了个 GitHub repo，把 Claude Code 的流量重定向到 DeepSeek、Kimi 等十多个免费模型，5 分钟配置，2 万多开发者已经在用。听起来像白嫖神器，但真正值得看的不是“永久免费”四个字，而是 Claude Code 这类工具的核心价值正在被拆开：  
  
前端交互、代码工作流、Agent 执行框架、底层模型供应商、API 路由层、成本控制——每一层都可以被替换。  
  
以后开发者不会只问“哪个模型最强”，而会问“我能不能把同一套工作流，接到更便宜、更快、更适合当前任务的模型上”。这才是这个 repo 真正指向的方向。你试过这类路由方案了吗？

![🎥 Video](<Soran%20(@Soranlan).assets/image-01.jpg>)

> **Sweex (@sweexx9)**
> 
> 有人在 GitHub 上创建了一个仓库，让你能完全免费、永久运行 Claude Code。  
>   
> 将你的 Claude Code 流量重定向到 10 个免费提供商，如 DeepSeek 和 Kimi，只需 5 分钟配置，就能让超过 20,000 名开发者运行它。
> 
> ![🎥 Video](<Soran%20(@Soranlan).assets/image-01.jpg>)

---

别只盯着“免费 Claude Code”六个字，免费只是钩子，真正的杀招是它把 Claude Code 拆成了两层。  
  
上面还是你熟悉的终端体验、文件操作、代码修改流程，下面的模型供应商可以直接换  
  
NVIDIA NIM、OpenRouter、DeepSeek、Kimi、本地 LM Studio、Ollama 全都能接进去。  
  
这意味着 AI x.com/Soranlan/statu…

![Image](<Soran%20(@Soranlan).assets/image-02.jpg>)

---

这条评论我觉得很关键。  
都在吵 free-claude-code 能换哪个模型  
DeepSeek 还是 Kimi 还是接 OpenRouter  
  
但最要命的问题没人问  
  
Claude Code 真正值钱的  
不是底下跑的是哪个模型  
而是它能在一个真实项目里  
持续读文件 理解结构 追踪修改 执行命令 根据报错继续修  
  
这叫项目上下文能力 x.com/Soranlan/statu…

![Image](<Soran%20(@Soranlan).assets/image-03.png>)

---

看到一条评论  
  
我差点把 free-claude-code 最要命的坑给说漏了  
  
大部分人看 proxy 就三句话 能不能跑 能不能省 能不能换模型  
  
真正恶心的事全在后头  
  
第三方路由最怕的不是配置失败 是小 demo 跑得飞起 复杂项目一上 各种边缘问题排队找你  
  
比如工具调用格式 第三方模型根本不兼容 x.com/Soranlan/statu…

![Image](<Soran%20(@Soranlan).assets/image-04.png>)

---

纠正一下之前的一个认知，CC switch和这个比，CCswitch重很多  
实际上这俩放一起比 一开始就比错了  
  
free-claude-code 是个轻量专用代理  
只干一件事 把 Claude Code 的 Anthropic Messages API 请求 转发到其他 provider  
目标很窄 结构很薄  
CC Switch 是重型多 CLI 中控台  
管的不是一家  
Codex Gemini x.com/Soranlan/statu…

![Image](<Soran%20(@Soranlan).assets/image-05.png>)
![Image](<Soran%20(@Soranlan).assets/image-06.png>)

---

这俩从代码层面一看 路线就分岔了  
  
free-claude-code 的架构清清爽爽  
http://server.py 进 FastAPI 走 routes 到 ClaudeProxyService  
下面挂 provider registry 接 BaseProvider  
标准后端代理服务 轻 职责清楚 理解成本低  
  
CC Switch 这边完全另一条路  
Tauri 入口 进 Commands 走 Services 挂

![Image](<Soran%20(@Soranlan).assets/image-07.png>)

---

代理层这块 我本来以为 CC Switch 主要就是个配置管理  
  
翻完代码发现 想简单了 它的代理层其实相当重  
  
free-claude-code 就支持一套 Anthropic Messages API 服务 Claude Code 一个目标  
  
CC Switch 呢 Anthropic Messages OpenAI Chat/Responses Gemini 全接了 面向的是多个 CLI 多种 API 格式 x.com/Soranlan/statu…

![Image](<Soran%20(@Soranlan).assets/image-08.png>)
![Image](<Soran%20(@Soranlan).assets/image-09.png>)

---

配置管理这层 差别直接拉出两条赛道  
  
free-claude-code 简单到令人舒适  
.env 文件 加个 Admin UI 本地跑起来  
单机党个人测测 开箱即用  
  
CC Switch 一看就是奔着长期管理去的  
SQLite 存配置 settings.json 桌面 GUI  
同步方案直接给你铺满  
  
WebDAV Dropbox OneDrive iCloud  
还有 Deep Link 导入 自动备份 x.com/Soranlan/statu…

![Image](<Soran%20(@Soranlan).assets/image-10.png>)

---

扩展方式也能看出两个项目的性格。  
  
free-claude-code 扩展 provider 的方式很工程化：  
  
继承 BaseProvider。  
实现 stream_response。  
实现 list_model_ids。  
实现 cleanup。  
这对开发者很直观。  
你要接新 provider，就按接口写实现。  
CC Switch 的扩展逻辑更偏平台化。  
  
它通过 AppType、不同 CLI 的 x.com/Soranlan/statu…

![Image](<Soran%20(@Soranlan).assets/image-11.png>)