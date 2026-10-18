---
author: "佐仔"
handle: "@huangjinbo"
source: "https://x.com/huangjinbo/status/2102954598050992388"
date: 2026-09-24T02:53:41.000Z
type: article
---
# Paseo -- 它成了我的主力开发工具

*By 佐仔 (@huangjinbo)*

![Banner](<Paseo%20--%20它成了我的主力开发工具.assets/image-01.jpg>)

[#Paseo](https://x.com/search?q=%23Paseo&src=hashtag_click) 是一个开源的本地 AI 编程 Agent 运行与多端编排监控平台，一个统一的控制台和调度守护进程（Daemon），把你在本机安装和授权的各大编程 Agent CLI（如 Claude Code、Codex、OpenCode、Pi、Cursor、Gemini CLI 等 30+ 种）接管起来，并提供跨设备（桌面端、Web 端、移动端 iOS/Android、终端 CLI）的监控、操作与编排体验。

Paseo 的底层原理是什么？

Paseo 的核心设计思想是 **“Daemon-Client” 架构（守护进程 + 多端客户端）**：

1. **核心守护进程（Paseo Daemon）**

- 运行在你真实的开发机（或云主机、Docker 容器）上，默认监听 WebSocket 接口（如 ws://127.0.0.1:6767/ws）。

- **子进程管理**：负责实际拉起宿主机上已有的 Agent CLI（以本地子进程运行），接管它们的输入输出（I/O 流）、生命周期和状态持久化。

- **协议适配（Providers）**：

- **原生适配器（Native）**：深度绑定 Claude Code、Codex、OpenCode、Pi 等主流工具。

- **ACP（Agent Client Protocol）通用适配器**：对接任何遵循 ACP 标准的编程代理。

- **环境隔离**：支持 Git Worktrees 隔离工作区，不同 Agent 可以分配独立的本地分支和开发服务端口，互不污染。

2. **多端客户端（Clients）与端到端中继（Relay）**- 原生应用覆盖 macOS、Windows、Linux、iOS、Android 以及 Web 界面。

- 配合端到端加密的 Relay（中继）服务，你在手机或外出笔记本上无需暴露公网 IP 或配置复杂内网穿透，即可直连家里的开发机调度 Agent。

3. **模型与凭据机制**

- **完全自带（Bring Your Own Account / Keys）**：直接沿用你在本机 CLI 登录的官方订阅（如 Claude 订阅、OpenAI 账户）或本地模型（如 Ollama），Paseo 自身不代理、不对外转发生存代码和模型 Token。

Paseo 有什么好处？

1. **统一工作台，打破单一 Agent 绑架**

- 不再受限于只能用某一家工具。你可以在同一套项目工作区内，让 Codex 负责写测试、Claude 负责架构设计、Pi 负责日常脚本，多 Agent 并行或协作。

2. **移动端与远程掌控（真正的“口袋里的开发机”）**

- 在手机/平板上即可查看 Agent 实时运行日志、审查生成的 Git Diff、批准终端权限并直接提交 PR，无需守在电脑前。

3. **数据与代码隐私留存本地**

- 代码不进入第三方云端容器，完全在你的设备上运行；无私有代码遥测。

4. **多任务并行与隔离（Worktrees）**

- 配合 Git Worktree，避免多个 Agent 同时修改同一份代码发生文件锁或 Git 冲突。

5. **强大的编排与自动化能力**

- 带有内置 MCP 服务、定时任务（Schedules / Cron）、Heartbeat 唤醒机制，还能通过 Hub 响应 GitHub Issue/PR 或 Slack 触发。

请求的是 SDK 还是 CLI？

**Paseo 两种都提供，底层都是向同一个 Paseo Daemon 发送指令：**

形式 载体 适用场景 说明 **CLI** paseo（如 paseo run, paseo ls, paseo wait） **日常操作、Shell 脚本、快速排错** 类似 Docker 风格的终端命令。当你在终端或通过其他 Agent（如当前会话环境）调度 Paseo 时，通常优先调用 CLI。 **SDK** [@getpaseo/client](https://x.com/@getpaseo/client) (TypeScript SDK) **工程化集成、Webhooks、业务系统二次开发** 通过 WebSocket (ws://127.0.0.1:6767/ws) 与 Daemon 建立长连接，提供 client.agents.create()、agent.waitForFinish() 等高阶 API，适合在 Node.js/TypeScript 应用中深度集成。

Paseo 的底层是一个由你完全掌控的 **Agent 守护进程**。无论通过 **CLI**、**TypeScript SDK** 还是 **Desktop/Mobile 界面**，本质上都是连接到本机的 Paseo Daemon，让它在后台以子进程方式驱动你本机的各家编程 Agent。