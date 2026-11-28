---
author: "riba2534"
handle: "@riba2534"
source: "https://x.com/riba2534/status/2126061183929368676"
date: 2026-11-26T21:11:01.000Z
type: article
likes: 95
reposts: 14
replies: 13
bookmarks: 162
views: 11719
---
# Agent 工作台产品设计与系统架构调研

*By riba2534 (@riba2534)*

![Banner](<Agent%20工作台产品设计与系统架构调研.assets/image-01.jpg>)

当 Claude Code、Codex 这类 coding agent 能独立干上几十分钟甚至几个小时，人的角色就从「和 agent 对话的人」变成了「给一群 agent 派活、放行、验收的人」。为这个新角色重新设计的指挥位，本文统称 **Agent 工作台**。这份调研沿**产品设计**与**系统架构**两条主线拆解五款代表产品：它们想让谁、在什么场景、用什么方式工作，又是怎么把 agent 接进来、跑起来、连出去的。

# 项目介绍

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-02.jpg>)

本文调研了 6 个开源仓库：`stablyai/orca`、`pingdotgg/t3code`、`makecindy/cindy`、`getpaseo/paseo`、`herdrdev/herdr-core`、`herdrdev/herdr-app`。其中 herdr-core（运行时）与 herdr-app（桌面窗口）是同一产品家族拆出来的两层，下文合称 **Herdr**，所以实际是五款产品。两条主线分别是：**产品设计**（定位与目标用户、功能全景、交互取舍、商业模式，以及每个设计在解决什么问题）和**系统架构**（进程与部署模型、Agent 接入方式、状态与持久化、远程与安全、编排与扩展）。用户使用是贯穿两者的检验视角：每款产品都用一个真实任务从发起到交付的完整链路，来验证设计与架构是否成立。

## 核心结论

1. **工作台不是新的 agent，而是 agent 的指挥位**。五款产品都不自己实现 agent 循环，而是驱动用户已经登录的 Claude Code、Codex、Pi 等 CLI，直接复用用户的订阅。只有 Cindy 额外售卖官方模型额度。
2. **最大的分野是「运行时切在哪里」**。Herdr 把运行时（core + holder）和窗口彻底分开，窗口只负责画；Paseo 和 T3 Code 是「daemon / server + 瘦客户端」；Orca 是 Electron 主进程加一个跨升级存活的终端 daemon，远程时换成无界面的 orcad；Cindy 是胖客户端，编排内核就跑在 Electron 主进程里。运行时越独立，「关窗不断、多端同构、远程同构」越容易做到。
3. **Agent 接入分成两派**。终端原生派用 PTY 包装任意 CLI（Herdr、Orca 终端模式），零适配但只拿到字节流；结构化派走 Agent SDK、Codex app-server、ACP、rpc（T3 Code、Paseo、Cindy、Orca 的 Chat 模式），能统一审批、渲染富界面，但要逐家适配。状态判断随之分为「程序自报」（OSC 7501、hooks）和「结构化事件」两路。
4. **「人在回路」被重做成一条注意力管线**。状态灯 → 需要你 → 在场判定 → 推送 → 审批卡 → 回到运行。五款产品都在想办法少打扰人，而不是让人逐条点批准；T3 Code 和 Orca 的新任务甚至默认全自主，把安全交给 worktree 隔离、checkpoint 回滚和评审。
5. **远程是标配，但信任模型差别很大**。Paseo（Noise NK）和 Orca（X25519）的中继是端到端加密的；T3 Code 走 Cloudflare 隧道，relay 只管身份和推送、不在数据路径上；Cindy 的 V1 中继在服务端终止 TLS，V2 Agent-Net 改为 Noise IK 端到端加密但仍是 Beta；Herdr 只用 SSH。
6. **多 Agent 编排收敛成四种模式**。协调者—工人（Orca、Cindy）、父子进程（Herdr、Paseo）、交接与委派（T3 Code、Paseo、Cindy）、评审与会商（Cindy、Paseo、Orca）。
7. **开源客户端是入口，收费点在云**。Orca Cloud 的 bot 连接器、与 T3 Chat 订阅挂钩的 T3 Connect、Paseo Hub、规划中的 Herdr Cloud、Cindy 的官方模型额度，五家的商业化都落在「连接与托管」上，而不是客户端本身。

## 五款产品速览

| 产品 | 一句话定位 | 使用形态 | Agent 接入 | 远程方式 | 许可 · Star · 版本 |
| --- | --- | --- | --- | --- | --- |
| Orca | IDE 里的 Bot 舰队：worktree 并行、评审、编排，外加从 Slack / Linear / GitHub 接活的 bot | 桌面（macOS / Windows / Linux）、手机伴侣、Remote Orca Server、SSH、云 VM | 终端 44 种 TUI agent；结构化 22 种（Agent SDK、app-server、19 个 ACP、pi rpc） | 端到端加密中继、LAN / Tailscale 直连、SSH 上部署 orcad | MIT · 138,807 · v1.4.277 |
| T3 Code | 编码 agent 的控制平面：一台 server，桌面、Web、手机、CLI 共享同一批线程 | 桌面、app.t3.codes、iOS / Android、t3 CLI | 11 个内置 driver + 任意 ACP agent | T3 Connect（Cloudflare 隧道）、LAN、Tailscale、SSH | MIT · 79,887 · v0.1.12 |
| Cindy | 开箱即用的通用 Agent 伙伴：账号下所有设备是同一个 Cindy | macOS / Windows / Linux / iOS / Android，另有 IM 机器人 | Claude Code、Codex、Pi 三个 Harness（ACP 插件开发中） | 官方 device-link 中继；Agent-Net（Beta）；自建中转 | Apache-2.0 · 10,656 · v0.1.146 |
| Paseo | 自托管的多 agent 统一界面：一个 daemon，所有终端 | 桌面、iOS / Android、Web、CLI、无头 daemon | Claude、Codex、Copilot、OpenCode、Pi，另经 ACP 与插件扩展 | Noise 端到端加密中继、TCP / Tailscale / SSH 直连 | Apache-2.0 · 25,528 · v0.17.4 |
| Herdr | agent 常驻的终端运行时 + 原生窗口：窗口只是客户端 | hc CLI、herdr-app 桌面、官方手机端（经 SSH）、社区客户端 | 任意程序（PTY），状态只认 OSC 7501 | SSH（Herdr Cloud 规划中） | Apache-2.0 · 52,929（Herdr 1）/ 3,506（app）/ 923（core） |

规模上，Orca 仓库有 35,352 个受控文件（TypeScript 约 3.3 万个），T3 Code 去掉内嵌的第三方子树后约 9,100 个，Cindy 23,582 个，Paseo 7,909 个，herdr-core 只有 189 个、herdr-app 1,498 个（含大量 vendor 与 mod 资源）。仓库体积和功能面大致成正比：Orca 与 Cindy 是「大而全」，Herdr 是「小而硬」。

## 术语速查

| 术语 | 释义 |
| --- | --- |
| Agent 工作台 | 本文对这一类产品的统称：不自己做 agent，而是同时驱动多个编码 agent，提供并行、多端、远程、审批、评审与编排能力。英文社区多称 ADE（Agentic Development Environment）或 agent orchestrator。 |
| Coding agent / Harness | 真正执行「模型调用 + 工具循环」的程序，如 Claude Code、Codex、Pi。Cindy 称之为 Harness（执行引擎）。 |
| Provider / Driver | T3 Code 与 Paseo 对被驱动 agent 的称呼；driver 是某一类 provider 的接入实现，同一 driver 可以有多份配置（多账号）。 |
| PTY（伪终端） | 操作系统提供的虚拟终端。工作台在 PTY 里启动 agent CLI，就能像人一样「敲命令、看屏幕」，对任何 CLI 都有效。 |
| Agent SDK | Anthropic 的 Claude Agent SDK。工作台用它的 query() 拉起用户本机的 claude CLI，拿到结构化消息与权限回调 canUseTool。 |
| Codex app-server | Codex 的服务模式：codex app-server 在标准输入输出上跑 JSON-RPC，暴露线程、回合、审批等接口。 |
| ACP | Agent Client Protocol，编辑器与 agent 之间的开放协议（JSON-RPC over stdio）。Copilot、Gemini CLI、Kiro 等都支持。 |
| MCP | Model Context Protocol。工作台常用它把「创建 agent、查看机器」等工具注入给 agent，让 agent 能指挥 agent。 |
| OSC 7501 | 由 Ghostty 发起的「程序状态协议」：程序通过一条终端转义序列自报 idle / working / done / blocked / error。Herdr 只认它，Orca 把它作为第二状态来源。 |
| git worktree | 同一仓库的另一份独立检出。工作台给每个任务一个 worktree，多个 agent 并行改代码而互不覆盖。 |
| daemon / server / core | 常驻后台、持有 agent 进程与状态的运行时。界面关闭后它仍在运行，各端都连接它。 |
| 中继（relay）与 E2EE | 手机与家里电脑都向外连同一台中继，从而无需开端口。E2EE（端到端加密）指中继只转发密文，看不到内容。 |
| 协调者—工人 | 多 Agent 编排模式：一个协调者（coordinator / Lead）拆任务、派活、验收，多个工人（worker）各自执行。 |
| 需要你 | agent 停下来等人（权限、提问、认证）的聚合状态。Orca 叫 Needs you，Herdr 叫 Blocked，T3 Code 叫 waiting for approval。 |
| 交接（handoff） | 把任务连同上下文交给另一个 agent 或模型。T3 Code 的 portable handoff、Cindy 的跨 Harness 交接、Paseo 的 /paseo-handoff 都属于此类。 |

# 为什么需要 Agent 工作台

终端里的 coding agent 是为「一人一会话、同步对话」设计的：你输入，它干活，你盯着屏幕，它问你要权限，你点确认。当 agent 能连续工作半小时以上，这种一对一同步模式就成了瓶颈。人想同时开五个任务，想离开电脑吃饭，想换一个更擅长的模型接着干，想让 agent 把子任务派给别的 agent。下图把裸用 CLI agent 时最常见的 9 个痛点，对应到工作台的设计回应，以及做了这件事的产品。

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-03.jpg>)

这 9 个痛点可以归成三类，五款产品的大部分设计都在回答这三个问题：

- **并行**：怎么同时跑多个 agent 而不互相踩？答案是「一个任务一个隔离环境」（git worktree、space），加上一个能一眼看出谁在干活、谁在等人的收件箱。
- **离开**：人不在电脑前，agent 怎么继续？答案是把运行时从界面里拆出去（daemon、server、holder），再加上远程连接、手机推送和审批卡。
- **指挥**：人怎么高效地验收、纠偏和分派？答案是 diff 行批注成批回传、失败检查一键交回，以及让 agent 能启动、等待、评审别的 agent 的编排原语。

## 一年里的演进

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-04.jpg>)

Paseo 是五款中最早的（2025-10），定位一直是「在自己机器上跑 agent，手机上也能推进」。T3 Code 在 2026-02 出现，README 直言受到 Codex 桌面版、Conductor、Claude Desktop、Cursor Glass 启发，但「都没达到我们的标准」，于是主打性能、远程与开源。Orca 在 2026-03 发布后增长最快，日均新增 Star 约 547 个，并从「多 agent 并行的 IDE」逐步扩展到「IDE 里的 bot 队友」。Herdr 同月以 TUI 终端复用器起步，2026-10 把自己拆成无界面的运行时 herdr-core 与 GPUI 桌面窗口 herdr-app。Cindy 最晚（2026-07），也最不像编码工具：它面向大众用户，把 Claude Code、Codex、Pi 当作可替换的执行引擎。

**❗ 重要**  
**趋势判断（推断）**：2026-10 前后有两件事值得注意。一是 Ghostty 发起 OSC 7501 程序状态协议，Herdr 随即把状态检测从「读屏 + 厂商 hook」改为「只认程序自报」，Orca 也把它接成第二状态来源；二是 Herdr 把运行时与窗口拆开，并提出「coding agents are becoming runtimes」。两者都说明：工作台的竞争正在从界面转向运行时和协议。

# 一张参考架构：工作台由哪六层组成

把五款产品的共性抽出来，一个 Agent 工作台大致由六层组成：人从哪里进来（入口层）、在哪里看和操作（交互层）、谁持有状态和进程（控制面运行时）、怎么驱动 agent（接入层）、agent 在哪里干活（执行环境）、怎么连到远处（连接与云）。agent 本体在工作台之外，由用户自带。

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-05.jpg>)

这六层里，真正决定产品性格的是中间两层。**控制面运行时**决定了「关掉窗口后还剩什么」以及「手机和桌面看到的是不是同一个世界」；**Agent 接入层**决定了能接多少种 agent、能拿到多结构化的信息、审批能不能统一。下表是五款产品在每一层的具体实现。

| 层 | Orca | T3 Code | Cindy | Paseo | Herdr |
| --- | --- | --- | --- | --- | --- |
| 入口 | 桌面、手机、Web 客户端、orca CLI、Slack / Linear / GitHub bot | 桌面、托管 Web、本地 Web、手机、t3 CLI、外部 agent（MCP） | 五端 App、IM 机器人、Local Agent API | 桌面、手机、Web、paseo CLI、SDK、Hub | herdr-app、hc CLI、官方手机端、社区客户端 |
| 交互 | worktree 侧栏 + 分屏；Chat / 终端双形态；diff 批注；内置浏览器与 Design Mode | 线程侧栏（Working / Settled）；Composer 的 Queue / Steer；四种 diff；内置浏览器 | 任务收件箱；伙伴与群聊；富回答（图表、控件、HTML）；远程桌面 | 按项目分组的会话；diff 评审；终端；语音模式 | space / tab / pane；agents 侧栏四态；peek；mods 绘制的面板 |
| 控制面 | Electron main 内的 OrcaRuntime + detached 终端 daemon；远程为 orcad | 独立 server 进程：Effect 服务 + 事件溯源 + outbox | Electron Main 内的 maker-core（胖客户端） | daemon：Session + AgentManager 状态机 | core daemon + holder 进程 |
| 接入 | PTY + 结构化双路；状态 hooks > OSC 7501 > 标题 > 规则 | ProviderAdapterV2 统一接口；按 capabilities 降级 | 每个 Harness 一个 translator，统一成 17 种事件 | 各家原生协议包成统一会话；权限 allow / deny | 只有 PTY；状态只认 OSC 7501 |
| 执行环境 | worktree；本机 / SSH / Orca Server / 云 VM / WSL；浏览器、模拟器、computer use | 每线程可选 worktree；多机负载；浏览器、模拟器画面 | Worker 可用独立 worktree；同账号其他电脑；SSH；云实例 | worktree；connected hosts；terminal worker | space 按机器与目录；远程机器经 SSH 并入 |
| 连接与云 | E2EE 中继、推送网关、bot 连接器（闭源） | T3 Connect：身份、隧道、推送；relay 不在数据路径 | 账号、device-link 中继、伙伴对话层、官方模型、SkillHub | 可选 E2EE 中继、Expo 推送、Hub | 无（SSH）；Herdr Cloud 规划中 |

再把五款产品放到两个维度上看：横轴是「只写代码」还是「通用工作」，纵轴是「终端原生 / 运行时」还是「结构化图形界面」。

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-06.jpg>)

四款编码向产品挤在左侧，差别在纵轴：Herdr 在最底部，产品本体是运行时；Paseo 偏下，daemon 与 CLI 优先；Orca 居中，终端和 Chat 两种形态并存；T3 Code 偏上，是打磨最细的图形控制台。Cindy 独占右上角，它的竞争对手更像通用 AI 助手，而不是 IDE。

# Orca：把 Bot 舰队装进 Agent IDE

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-07.jpg>)

## 定位与商业模式

Orca 最早的定位是「多 agent 并行的桌面 IDE」：每个任务一个真实的 git worktree，配一个 agent 终端和一个内置浏览器。最新 README 的口号改成了 **Bots inside your agent IDE**，在并行、评审、远程这些能力之上，又加了一层常驻 bot。bot 有自己的人格（`SOUL.md`）和记忆（`MEMORY.md`），能从 Slack、Linear、GitHub 接活，最后交回一个测试过的 PR。中文 README 和文档首页还停在旧口号「面向 100x 构建者的 AI 编排器」，说明这次转向发生得很近。

目标用户是「以写代码为生、把 AI 当杠杆」的人。文档的典型场景有四个：让三个 agent 同时赛跑修同一个 bug；认真评审 AI 写的 diff；已经为多家 CLI 付费、想统一编排；想在远程机器上跑 agent。文档明确说不做 no-code。

商业上，Orca 的客户端按 MIT 开源，不卖模型、不卖 VPS，仓库里找不到任何价格或套餐文案。唯一的托管部分是 Orca Cloud：中继、推送网关的代码公开，但接收 Slack / Linear / GitHub 事件的连接器闭源托管，部署配置在私有仓库 `stablyai/orca-cloud`。使用中继和 bot 需要登录 Orca Cloud。

**❗ 重要**  
**推断**：bot 依赖闭源连接器和 Cloud 登录，未来的收费点很可能落在 Orca Cloud（连接器、中继、bot 分享、团队功能）。目前仓库内没有任何价格证据。另外 `package.json` 写的版本是 1.4.214，而 GitHub Release 已到 v1.4.277（2026-11-26），版本字段并非每次发版都前移。

## 概念模型与功能全景

Orca 的所有对象都围绕两件事组织：**工作区**（agent 在哪里干活）和 **agent 状态**（谁在等人）。容器从大到小是：Host（本机、SSH 主机、Orca Server 或云 VM）→ Project（仓库或项目组）→ Workspace（git worktree 或普通文件夹，可以父子嵌套）→ Pane / Split → Tab（终端、Chat、编辑器、浏览器、diff、PR）→ Agent session。每个 worktree 记住自己的分屏布局，切换 worktree 时整套布局一起换。编排和 bot 是建在这套容器之上的两组对象。

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-08.jpg>)

| 功能域 | 代表功能 |
| --- | --- |
| 工作区 | 每个任务一个 worktree，后台创建可重试；可从 ref、SHA 或 PR 起步；Shared Paths 与 .worktreeinclude 补齐依赖和 .env；orca.yaml 配 setup 脚本与默认 tab；父子嵌套、批量休眠与删除 |
| Agent 会话 | README 点名 34 款 CLI agent（代码注册 44 种）；状态灯与 Agent 看板；默认以全自主参数启动；历史会话恢复；空闲 30 分钟休眠、回来自动恢复；全会话搜索 |
| Chat 与终端 | 同一 agent 可用结构化 Chat 或原始终端运行，⌘⇧J 切换视图；「Continue in…」把对话复制到另一种形态；AskUserQuestion 渲染成卡片；拖文件和图片进 prompt |
| 评审与交付 | 合并 diff（含图片 diff 与三方合并）；行批注成批回传；AI 行归因；AI 写 commit 与 PR；Fix broken checks、Fix with AI、Resolve with AI；stacked PR |
| 任务源 | GitHub、GitLab、Bitbucket、Azure DevOps、Gitea；Linear 抽屉（沿用 Linear 分支名）；Jira；从 issue 一键建工作区 |
| 浏览器与设备 | 每个 worktree 一个 Chromium；Design Mode 点选元素交给 agent；视口模拟；CLI 驱动浏览器；iOS / Android 模拟器；computer use |
| 编排与自动化 | orca CLI（人能做的 agent 都能做）；Run / Task / Dispatch 与决策门；在别的机器上起 worker；Automations（cron / RRULE / 事件触发） |
| Bots | 24 个模板；SOUL 与 MEMORY，每晚整理记忆；连接 Slack、Linear、GitHub、Google、Composio；Items 清单；多 bot 群聊；从 OpenClaw、Hermes 导入 |
| 远程与多端 | SSH worktree；Remote Orca Server；Cloud VM recipe（用户自带云）；手机经中继或局域网配对，可看 Source Control、建工作区、切账号 |
| 用量与效率 | 状态栏显示 9 家服务商用量、80% 预警；Claude / Codex 多账号隔离与热切换；离线听写；⌘J 跨 worktree 跳转；插件（Alpha） |

## 进程与部署架构

Orca 的关键架构决定是：**界面和运行时可以分开**。终端由一个 detached 的终端 daemon 持有，它跨 App 升级存活，所以 daemon 协议按版本门控（当前协议版本 62）；每个会话在 daemon 里有一个 headless xterm 仿真器，用于重新挂上时回放屏幕。macOS 上 daemon 由 launchd 拉起一个签名的 `Orca [Agents.app](https://Agents.app)`（约 124 MiB），目的是让文件访问授权（TCC）在 App 升级后不失效。同一套运行时去掉 Electron 就是 `orcad`，它可以跑在 SSH 主机、自建服务器、WSL 和云 VM 上。

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-09.jpg>)

所有客户端调的是同一套 runtime RPC：自定义 NDJSON 信封，加一个 24 字节随机 token。传输分三种：本机 Unix socket / 命名管道（权限 0600、最多 32 个连接）、WebSocket（默认端口 6768，手机在局域网或 Tailscale 下直连）、Orca Cloud 中继（手机出站 WSS，帧用 X25519 + HKDF + secretbox 做端到端加密）。推送走独立的推送网关到 APNs / FCM，手机端在直连和中继之间切换时要满足「3 次成功、30 秒观察、60 秒驻留」，以防抖动。

Agent 接入走两条路。终端路线在 PTY 里起登录 shell，看到就绪标记后再把 agent 命令敲进去，所以「任何 CLI 都能跑」，代码里注册了 44 种 TUI agent。结构化路线共 22 个：Claude 走 Agent SDK（指向用户自己装的 CLI），Codex 走 `codex app-server`，19 个走 ACP，pi 走 rpc。两条路之间靠 agent 自己的 session id 加 resume / fork 互转，同一会话同一时刻只允许一个写入方。状态判断叠了四路来源，优先级从高到低是：hooks（22 个 agent 用 curl 向 127.0.0.1 上报）、OSC 7501（hooks 沉默 2 分钟后接管）、终端标题、每 4 小时热更新一次的状态规则。

## 一次请求的端到端链路

下图追踪一个最典型的使用场景：在桌面新建 worktree 启动 Claude Code，人离开电脑，agent 请求权限，手机收到推送并回复，agent 跑完后评审 diff、开 PR。

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-10.jpg>)

这条链路里有两个细节体现了 Orca 的取舍。第一，Claude 的权限请求 hook 可以挂起最多 122 秒，等 Orca 给出裁决，这让桌面或手机有机会在 hook 超时前作答（手机端走这个窗口是本文推断，已确认挂起机制，未追到手机侧调用）。第二，done 状态要静默 1.5 秒才确认，避免 agent 在两个工具调用之间短暂停顿时误报完成。

## Bot：从 Slack 消息到 PR

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-11.jpg>)

bot 是 Orca 最新、也最能说明它想往哪走的功能。bot 的本体是一个藏在隐藏项目组里的 home 工作区（`~/.orca/bots/<folder>`），里面有 `SOUL.md`（它是谁、怎么工作、绝不做什么）、`MEMORY.md`（每行一条事实，只加载前 200 行并标注「不是指令」）和 skills。bot 不自己写代码：它把代码工作交给自己作为协调者启动的 worker，worker 开在子工作区里，并对 PR 负责到 CI 变绿。

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-12.jpg>)

bot 在用户自己的机器上执行（桌面主进程或 orcad），用的是用户自己的 CLI 订阅。云端只接收 Slack / Linear / GitHub 事件，再通过一个 Durable Object 发「现在来拉」的唤醒，桌面用 HTTPS 长轮询拉取（每次最多 20 条、超时 70 秒，离线机器最多等 24 小时）。这样云端从不接触用户代码，算力成本也留在用户自己的订阅上。

## 关键设计与它解决的问题

| 设计 | 解决的问题 | 证据 |
| --- | --- | --- |
| 每个任务一个 worktree，默认全自主启动 | 多 agent 并行互相踩文件；逐个工具弹审批打断心流 | docs/model/worktrees、agents-sessions |
| detached 终端 daemon + 协议版本门控 | 关闭或升级 App 不杀正在跑的 agent | daemon-protocol-version.ts（v62） |
| shell 就绪标记后再敲命令 | 任何 CLI 都能接入，不用逐个适配 | daemon-shell-ready-marker.ts |
| hooks > OSC 7501 > 标题 > 热更新规则 | 准确判断「需要你」；不发版就能修状态误判 | agent-hooks/README.md |
| 批注成批回传，而不是逐条发送 | 逐条反馈让 agent 来回摇摆 | docs/review/annotate-ai-diff |
| Fix with AI 只给修复提示，不许绕过 hook | agent 为了过关跳过 pre-commit 检查 | docs/review/commit-push |
| Chat 与终端互转时复制会话、单写者 | 想要可读视图，又不想丢掉原始 CLI 或写坏会话 | agent-resume-argv.ts |
| bot 交给子工作区 worker，watch-pr 最多自动修 2 次 | bot 主对话被代码细节淹没；无人值守时陷入修复死循环 | bot-preamble.ts |
| Who can ask 名单 + 外部消息只当数据 | 公共频道里谁都能使唤 bot；prompt 注入 | bot-types.ts、bots.md |
| loading 延迟约 200ms 再显示 | SSH 远程延迟下要么闪烁、要么没反馈 | docs/styleguide/ux-rules.md |

## 亮点与代价

**亮点**

- 功能面最广：并行、评审、浏览器、模拟器、computer use、编排、bot、自动化都在一个产品里。
- 「人能做的，agent 都能做」：`orca` CLI + 11 个 skill 让 agent 能建 worktree、起 worker、驱动浏览器。
- 评审回路做得最细：行批注、元素批注、失败检查、hook 失败、合并冲突都能一键交回 agent。
- UX 规范克制且可执行：文案不夸大、反馈按耗时分四档、后退动作保持安静，并要求每个改动同时考虑跨平台、远程、手机和文件夹工作区。

**代价**

- 复杂度高：35k 个文件、36 个主进程构建入口、67 个 CI workflow，新人很难建立全局认识。
- 默认全自主，把安全边界交给 worktree 隔离和评审，设置里保留了 Manual 档给不接受的用户。
- bot 依赖闭源连接器与 Cloud 登录，「Bring your own everything」在 bot 场景下打了折扣。
- 文档与代码有漂移：SSH 文档仍写「安装一个小 relay」，代码已换成 orcad；版本号字段落后于 Release。

# T3 Code：编码 Agent 的控制平面

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-13.jpg>)

## 定位与商业模式

T3 Code 把自己定义为 coding agent 的 **控制平面**（control plane），而不是 harness：它没有自己的 agent 循环，只驱动宿主机上已经登录的 Claude Code、Codex 等 CLI。部署形态是「一台 server，多个客户端」：server 跑在拥有代码的那台机器上（T3 Code 称之为 environment），桌面、浏览器、手机和 `t3` CLI 都只是连上它的界面。代码和 prompt 从宿主机直连模型服务商，T3 Connect 的中继只负责牵线，不代理流量。

README 对商业模式的回答是「Nothing」：不转卖 token，也不设额度。仓库里能看到的唯一付费点，是 T3 Connect 的主机数与 T3 Chat 订阅挂钩：免费账号最多连 3 台主机，关联 T3 Chat Pro 提到 10 台，Premier 提到 100 台。代码 MIT 开源，但商标从严：fork 必须换掉名称、图标并声明非官方，冒充官方「将被依法追究」。外部贡献只收 bug 修复，新功能、新 provider、UI 或架构重构都不收。项目自述用户超过 50 万（来自 AGENTS.md，未核实）。

T3 Code 有两个鲜明的产品主张。一是**远程优先**：T3 Connect、LAN、Tailscale、SSH 多条路由自动切换，配合手机推送和审批卡，人离开电脑 agent 也能往下走。二是**性能即功能**：README 说「掉帧就是 bug」，AGENTS.md 把 WebSocket 数据过量、CSS 动画导致的 GPU 尖峰、长列表渲染列为不可妥协项，CI 里有传输预算检查。

## 概念模型与功能全景

T3 Code 的术语表很严谨。Environment 是一台运行中的 server 及其机器、凭据、工作区和状态；Project 是环境里以某目录为根的工作区；Thread 是持久保存的对话和工作历史，provider 进程退出后仍然存在；Turn 是一次用户与 agent 的往返，每个 turn 产生一个 checkpoint（存在隐藏的 Git ref 里，不污染用户分支）。Provider 是被驱动的 agent 运行时，driver 是某一类 provider 的接入实现，同一个 driver 可以配多份 instance，多账号就靠这个。Session 是挂在线程上的 provider 运行时，停掉再恢复不会删线程。

| 功能域 | 代表功能 |
| --- | --- |
| 线程 | 一个任务一个线程，可单独开 worktree；不挂项目的 scratch 线程；按住 Shift 多选模型，同一 prompt 给每个模型各开一个线程；Settle / Snooze / Archive；/side 侧聊不污染主线程 |
| Composer | agent 运行中追加消息，可选 Queue（排队）或 Steer（插入当前回合）；@ 引用线程或模型、# 引用 PR；/goal 跨回合干到目标达成；Edit from here 连文件一起回滚 |
| 评审与 Git | 四种 diff 来源（本回合 / 未提交 / 对比基线 / 逐 commit）；选行评论汇入 composer；commit、push、开 PR 一键完成；Review guide；让 agent 盯 PR；支持 9 类代码托管和 Linear |
| 多 agent | 10 个内置 provider + 任意 ACP agent；线程中途换模型；delegate_task 派子代理（可指定 provider、worktree 或另一台机器）；Workflows 多阶段流水线；定时任务与 webhook |
| 远程与多端 | T3 Connect；LAN / Tailscale 配对与路由切换；SSH 托管环境；新设备审批；推送、Live Activity、小组件；手机离线消息排队 |
| 预览与工具 | 内置浏览器（元素拾取、标注、录屏，人可 Take control）；HTML 可视化回复与 MCP Apps；iOS 模拟器与 Android 画面；集成终端 |
| 协作与外部接入 | 外部 agent 经 MCP 接入（t3_thread_launch / wait）；Guest link 让访客旁观（消息需主人审批）；Prompt link 只预填草稿 |
| 安全治理 | 四级权限模式；推送会触发 Git hook 时每次确认；配对 token 限定 scope；密钥用私密卡片收集；企业策略统一下发上限 |
| 运维与用量 | 跨 provider 汇总 token、成本与订阅额度；一键更新所有机器；重启后线程自动续跑；首次启动可导入 Claude、Codex、Pi 的历史会话 |

## 分层架构：一台 server，多个瘦客户端

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-14.jpg>)

server 内核是一个「有界事件溯源 + outbox」的结构。Orchestrator 按线程加锁，先做不含 I/O 的决策；EventSink 用一个 SQLite 事务同时写事件、投影、命令回执和 outbox；EffectWorker 在事务提交之后才执行副作用（调 provider、截 checkpoint），结果再以命令的形式回灌。所以一条命令被确认，只代表「意图已经写下」，副作用不一定做完。投影才是真相源，事件日志只保留约 1 天，用作客户端重连时的重放窗口。

整个 server 用 Effect 4 组织：非测试代码里有 238 个 `Context.Service`、534 个 `Schema.TaggedError`。规则是「能力是服务上的方法，传输层的 handler 只做解码、调用、映射错误三件事」，这样 WebSocket、MCP 工具、定时任务和 CLI 调的是同一个方法，逻辑不会只存在于某一个入口。contracts 包定义了 261 个 RPC 和 18 个授权 scope，每个 RPC 调用都要过 scope 校验，握手成功不等于拿到全部权限。

所有 provider 收敛到 `ProviderAdapterV2` 这一个接口：能力声明、会话打开、开始 / 插入 / 中断回合、回复审批、回滚、分叉、注入历史。各家协议差异关在适配器里：Claude 用 Agent SDK，Codex 用 app-server 的 JSON-RPC，Cursor 用自家 SDK 跑在 worker 子进程，OpenCode 用 HTTP + SSE，Grok / Kiro / fx / Antigravity 是通用 ACP 引擎上的 flavor，Pi 用 rpc，Muse 用 SDK。编排层只读 capabilities，从不判断 provider 的名字；provider 不支持某个权限模式时回落到最严格的一档，并且明确禁止「用自动回答审批来假装支持」。

## 手机批准一条命令的完整链路

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-15.jpg>)

这条链路说明了 T3 Connect 的信任模型：relay 是一个可信的 broker，负责身份（Clerk）、为每个环境建 Cloudflare Tunnel、铸造与客户端设备密钥（DPoP）绑定的一次性凭据、以及推送；但客户端拿到凭据后直接连隧道主机名，relay 不在数据路径上，也看不到 session token。需要注意两点：一是隧道的 TLS 在 Cloudflare 边缘终止，Cloudflare 理论上可见明文（推断，仓库里没有找到端到端加密）；二是推送凭据只放在 relay 上，所以只靠 LAN 或 Tailscale 直连收不到后台推送。另外，手机通知里没有找到直接批准的入口，批准必须进 App 完成。

## 交付闭环与权限阶梯

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-16.jpg>)

T3 Code 把「跑 agent → 看 diff → 反馈 → 提交 → 盯 PR → 合并 → 清理」做成了一个闭环，每一环都在线程内完成，并且有两条回路：评审回路（选行评论合成一条消息回给 agent）和 CI 回路（每 2 分钟轮询 PR，CI 失败或有新评论就唤醒 agent）。

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-17.jpg>)

权限模式按线程设置，从严到宽四级：Supervised（命令和改文件都要批）、Auto-accept edits（改文件自动批）、Auto（交给 provider 自己的风险判断）、Full access（全不问）。新线程默认 Full access，兜底靠 worktree 隔离、checkpoint 回滚和企业策略上限。实际生效的权限层层收紧：企业策略的上限 ≥ 线程的权限模式 ≥ 子代理的权限；外部 agent 和链接的机器也不能超过授予它们的那一档。

换 provider 时，`provider.switch` 会给出五种结果之一：复用、会话内换模型、重启并恢复、带交接新建、拒绝（有回合正在运行时直接拒绝）。跨 provider 的上下文靠一份按预算挑选的可移植 transcript：优先保留首条请求、最近几轮对话和命令结果，默认 16,000（可调 1,024～64,000），被省略的部分附上提示，agent 可以用 `t3_thread_read` 按需读回。Codex 用原生的 `thread/inject_items` 注入，其他 provider 以前缀形式拼进首条消息。

## 关键设计与它解决的问题

| 设计 | 解决的问题 | 证据 |
| --- | --- | --- |
| 驱动已登录的 CLI，不卖 token、不做多账号轮换 | 已付订阅不想再给中间商付费，也怕被服务商封号 | README；docs/user/faq.md |
| 一台 server 服务多个 client | 换一台设备，会话就接不上 | docs/internals/01_architecture.md |
| 事件 + 投影 + outbox 同一事务 | 副作用与状态不一致；崩溃后重复执行副作用 | 01_architecture.md |
| 快照 + 序列号增量 + 有界重放（128 事件 / 1 MiB） | 弱网手机重连被大量重放拖垮 | ThreadStream.ts |
| 命令输出走旁路，只推给展开该行的客户端 | 忙碌的命令刷爆事件存储，拖慢所有线程 | CommandOutputHub.ts |
| 审批统一成 runtime request + 5 种决策，不伪造权限模式 | 任何 provider、任何客户端看到同一种审批卡，安全承诺与实际一致 | providerPolicy.ts；03_providers.md |
| 按预算迁移上下文（portable handoff） | 换 provider 后上下文丢失或撑爆窗口 | handoffBudget.ts |
| relay 不在数据路径，凭据绑定 DPoP | 云端掌握会话 token 和流量，信任面与带宽成本都大 | 06_t3-connect.md |
| launcher 试运行 + SQLite 快照再切换 | 远程自更新失败后无法回滚 | 07_server-updates.md |
| 本回合 diff 排除 merge 带入的改动 | 评审时被无关改动淹没 | docs/user/source-control.md |

## 亮点与代价

**亮点**

- 架构最规整：server / client 边界清楚，Effect 服务化，事件溯源 + outbox，事务语义清晰。
- provider 抽象最完整：11 个 driver 统一接口，按 capabilities 声明降级，并且有 176 份真实协议录制用于回放测试。
- 工程约束机器化：36 条自定义 lint 规则、A/B 交替的性能基准与 A/A 噪声地板、传输预算 CI。
- 远程体验开箱即用：托管隧道免去端口转发与公网 IP。

**代价**

- 桌面托管模式下关掉最后一个窗口 server 即停，常驻需要另装后台服务。
- T3 Connect 依赖 Cloudflare 与 Clerk，TLS 在边缘终止，不是端到端加密（推断）。
- 不开放第三方插件，外部贡献只收 bug 修复，生态扩展主要靠官方。
- 新线程默认 Full access，对谨慎用户需要手动收紧。

# Cindy：面向大众的 Agent 伙伴

Cindy 是五款里唯一不把「写代码」当作主线的产品。它的口号是「想到，就能做到」（Consider it done），主视觉列出的卖点是 Claude Code + Codex、More Models、Open Source、Zero Config。它和前两款的根本区别在于：主入口不是代码仓库和 diff，而是「伙伴」和任务收件箱；它覆盖 IM、邮件、Office 文档、语音通话、远程桌面这些非编码场景，Claude Code、Codex、Pi 只是可以替换的执行引擎（Cindy 称之为 Harness）。

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-18.jpg>)

## 定位、版本与商业模式

「开箱即用」有三层意思：登录后自带有赠送余额的官方模型 Cindy AI，不需要开通；自动检测本机已登录的订阅；Claude Code、Codex、Pi 已有的技能和 MCP 直接生效，不需要导入。目标用户是大众用户，产品原则写明「功能的价值以实际成果衡量，不以问答、能力展示或功能数量衡量」，同时也保留了开发者需要的 SSH、worktree、PR 状态和 `/review`。

Cindy 按构建分为两个版本：Global（国际版）是产品本体，中国大陆版是为当地法规单独构建的版本。大陆版以手机号为账号锚点、用人民币计价、支付宝支付，连接模型时优先显示智谱、Kimi、MiniMax、百炼、火山方舟等国内供应商；国际版以邮箱为锚点、用美元、Stripe 支付，优先显示订阅登录。服务器按区域部署，伙伴数据不跨境。

商业模式是「免费客户端 + 官方模型额度」：客户端以 Apache-2.0 开源；用户自带的订阅、API Key 和本地模型不占 Cindy 额度；收入来自官方模型服务 Cindy AI 的套餐与充值。官网价格（2026-11-26 抓取，仓库内没有）：大陆版 PLUS ¥100、SUPER ¥500、ULTRA ¥1000 每月，国际版 $20、$100、$200 每月，每月额度等于售价，未用完可滚存一部分。

**❗ 重要**  
**推断**：主视觉小字「XD_INC — PROJECT CINDY」、应用标识 `com.xd.cindy`、使用 TapTap 的 TapDB 做统计，开发方可能是心动网络，仓库内未正式说明。另外 README 主动披露了一个隐私缺陷：移动端企业 SSO 可能把未明确同意的用户标记为已同意统计。这种「把已知问题写进 README」的做法在五款产品里很少见。

## 概念模型：账号下所有设备是同一个 Cindy

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-19.jpg>)

Cindy 的对象模型以**账号**为根。一个账号下有多台设备（电脑、手机、SSH 主机、云实例），手机只负责看和说，不运行 agent。任务属于一台执行电脑，其他设备打开它时只是镜像；任务 → 对话 → 消息三级，「轮次」不单列。每个任务有一条运行路由，由引擎、供应商、模型、推理强度和 Fast 共同决定，供应商可以来自本机、同账号的其他电脑、别人的分享、供应商组或模型组。产品规则把 session 叫「任务」而不是「对话」，就是为了让用户一眼看出列表里每一条都能并行推进。

伙伴属于账号而不属于某台电脑，在它的主力电脑上运行；云端只保存对话层和资料，工作目录、凭证、权限档和完整运行记录都留在主力电脑。主力电脑离线约 15 秒后，界面出现「在这台电脑继续」，伙伴整体搬到当前电脑，不丢对话。两种「共享」容易混淆：共享任务借出的是任务和 agent，文件在房主电脑上；供应商分享借出的是模型额度，文件留在受邀者自己的电脑上。

| 功能域 | 代表功能 |
| --- | --- |
| 伙伴 | 默认伙伴 Cindy；自建伙伴可设名字、头像、性格（SOUL）和指令；每个伙伴是一个带规则、记忆、技能、自动化的项目；1～5 个模型候选自动顺延；可从 OpenClaw、Hermes 导入 |
| 任务 | 项目任务与对话任务；收件箱按待处理 / 进行中 / 稍后 / 完成分区；12 个预置标签；共享任务凭链接加入；自动关联 GitHub PR |
| 多设备 | 在另一台设备打开同一任务；复制任务到另一台电脑（含未提交改动快照）；远程调用同账号其他电脑的供应商；供应商组自动换机；迁移助理；Agent-Net 端到端加密与自建中继 |
| 引擎与模型 | Claude Code / Codex / Pi 可切换；Cindy AI；同一供应商多账号互相接替；本地模型按内存推荐；模型组与自动路由；撞上 5 小时或每周上限后到点自动继续 |
| 多 Agent | 协同（Lead + Worker，可分屏）；目标模式 /goal；/review 起独立只读 Reviewer；伙伴群聊里 @ 接力，可跨账号 |
| 工具与现场 | 文件浏览；Agent 专用浏览器、ADB 操作 Android、操作电脑；截图标注；密码由用户代填不经模型；SSH 远程工作区；list_machines 让 agent 看到所有机器资源 |
| 富内容 | 回答里流式出现图表和交互控件；Mermaid、公式、图片、视频；生成 PDF / Word / PPT / Excel；MCP Apps 内嵌 HTML |
| 自动化与 IM | 定时、手动、Webhook 触发；个人 IM 机器人（飞书、钉钉、企业微信、微信、Telegram、Discord、iMessage）；授权请求在 IM 与 App 同时出现，谁先确认算谁 |
| 扩展与沉淀 | SkillHub 市场与组织分发；/learn 从任务提炼 Skill；.cindy 沙箱插件；MCP；主题包；Local Agent API |
| 随身与提醒 | 桌面灵动岛；手机 Live Activity 与额度小组件；锁屏或 Apple Watch 上直接作答；语音输入与实时语音通话；隐私屏 |

## 桌面端架构：胖客户端与 Harness 翻译层

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-20.jpg>)

Cindy 是典型的胖客户端：任务在本机执行，云端只做中继和账号。编排内核 maker-core 跑在 Electron Main 进程里（本身不依赖 Electron），Renderer 被当作不可信环境，按窗口拆出 28 个最小权限的 preload。三个 Harness 都是 Main 的子进程：Claude Code 由 Agent SDK 的 `query()` 拉起，Codex 每个 agent 只起一个 `app-server`、多个会话按 thread 复用，Pi 每个会话起一个 `pi --mode rpc`。每个 Harness 配一个 translator，把原生事件统一成 17 种 AgentEvent（文本、思考、工具调用、交互请求、轮次 diff、状态、完成、错误等），规则要求翻译不丢事件、不乱序。这样一套界面、手机端和 IM 渲染就能适配所有 Harness。

两个细节体现了 Cindy 对成本与体验的在意。一是**换 Harness 不靠 LLM 摘要**：代码重放数据库里的历史，生成有上限的交接文本（≤100K tokens），旧引擎的原生会话先「停泊」，切回来时只补离开期间的增量，prompt cache 不会丢。二是**模型层的回环代理**：Claude Code 用非 Anthropic 模型时，`ANTHROPIC_BASE_URL` 指向本机 127.0.0.1 上的兼容代理，再经三个协议桥转换到 OpenAI Responses、Chat 等格式；订阅会话是例外，直接连 Anthropic。模型注册表有 333 个模型、39 个预设。

## 多设备与模型共享

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-21.jpg>)

多设备有两代实现。V1 device-link 已在生产：官方 WSS 哑中继，握手时用 JWT 固化身份、按用户隔离；被控端 monkey-patch 了 `ipcMain.handle`，所以 300 多个现有 handler 不改就能被远程调用，同时由「允许被控」开关和默认拒绝的白名单守住边界。它的代价是 TLS 在中继处终止，官方中继可见明文，附件走 OSS 预签名也没有客户端加密。V2 Agent-Net 目前是 Beta、默认关闭：自研 Rust 可靠 UDP（思路来自 KCP，明确不用 QUIC、libp2p、Tailscale），Noise IK 握手端到端加密，STUN 打洞失败依次退回 UDP 中继和 443 WSS，桌面端编成 WASM、手机端用原生库。基准测试中，大文件传输进行时小请求的最长等待从 V1 的 0.6～4.7 秒降到 35～41 毫秒。

「模型共享」不是 API 代理，而是 **Remote Agent**：任务、文件和命令留在电脑 A，agent 进程跑在有订阅的电脑 B 上，用 B 的登录调用模型；agent 需要读写文件或执行命令时，请求经 `maker:remote-agent:v1` 反向回到 A 执行，A 按游标拉取 B 的事件。跨账号分享走「一次性链接 + 双方确认」：分享者发出 5 分钟内有效的一次性链接，受邀者申请后看到一个 4 位配对码，分享者核对配对码一致再同意；之后 B 用路由令牌只放行被分享的那一家供应商，成员预算按 7 个自然日内的 token 占比计算。

## 带队：Lead、Worker 与 Reviewer

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-22.jpg>)

Cindy 的协同（内部代号 Orca）里，Lead 和 Worker 都是完整会话，不是一次性的 subagent，三种 Harness 可以混用，Worker 还可以跑在 SSH 远端或同账号的另一台电脑上。Lead 通过进程内 MCP 用 22 个工具派活，Worker 用 8 个工具回报；Worker 不回报时系统自动转发结果。验收没有独立的状态机，靠的是提示词纪律加「欠回报」计数：Lead 宣布完成前，必须用执行这个任务的同一个通道核实终态，原生 subagent 的结果不算；只要还有 Worker 没回报，Lead 就保持「运行中」。产品原则是「宿主只暴露情况，不替模型做资源决策」，所以 Worker 上限只给软 / 硬两档默认值（5 / 8），最多可调到 20。

独立复核是另一条线：`/review` 新建一个没有开发记忆的只读 Reviewer，给出带证据和优先级的问题清单，有发现时结论自动回给来源任务去修。产品规则对它的定位是「避免开发 agent 自己给自己打分」。

## 关键设计与它解决的问题

| 设计 | 解决的问题 | 证据 |
| --- | --- | --- |
| 伙伴属于账号，有一台主力电脑 | 多台设备上出现互相分裂的同名助手；项目文件不必上云 | product-rules/cloud-companions.md |
| 手机只做控制端，被控端是唯一真相 | 不用在手机上同步文件和数据库，多端状态一致 | device-link README；architecture-invariants.md |
| Remote Agent（模型共享） | 订阅只装在一台电脑上，又不想把项目文件搬过去 | device-link/src/remoteAgent.ts |
| 17 种统一事件 + 每个 Harness 一个 translator | 一套界面、手机端和 IM 渲染适配所有 Harness | maker-core/src/types/events.ts |
| 代码生成交接文本 + 会话停泊 | 换 Harness 不花摘要费，切回原引擎还能命中缓存 | sessionAgentSwitchHandler.ts |
| monkey-patch ipcMain + 默认拒绝白名单 | 300 多个现有能力不改就能远程调用，同时守住权限边界 | invoke-registry.ts；allowlist.ts |
| Agent-Net 多流 UDP + Noise | V1 小请求被大传输堵住；中继可见明文 | docs/agent-net-plan.md；benchmarks |
| 只读 Reviewer、同通道终态核验 | 开发 agent 给自己打分；Lead 拿 subagent 冒充 Worker 交差 | review-product-direction.md；orca-bridge-prompt.ts |
| 用户代填密码、隐私屏 | 密码进入模型和日志；远程操作时旁人看到屏幕 | input-handoff.md；privacy-screen.md |
| 撞额度上限到点自动继续 | 遇到 5 小时或每周上限要守着手动续跑 | usage-limit-auto-continue.md |

## 亮点与代价

**亮点**

- 多设备做得最深：换屏不换任务、跨电脑借用订阅、跨账号分享额度、手机远程桌面，都有完整的产品规则和协议实现。
- 面向大众的体验打磨：锁屏与手表作答、隐私屏、代填密码、灵动岛、IM 与 App 同步授权。
- Harness 翻译层与交接设计兼顾成本：不花摘要费、保住 prompt cache。
- 规则体系完整：dev-rules 336 篇、product-rules 58 篇、design-rules 23 篇，按路径生成规则索引，改动前先查规则。

**代价**

- 依赖官方云：账号、中继、伙伴对话层、官方模型都在不开源的服务端；跳过登录后远程与 IM hook 不可用。
- V1 中继可见明文，端到端加密的 Agent-Net 仍是 Beta、默认关闭。
- 胖客户端：编排内核在 Electron Main，无头的 apps/host 仍处于阶段 0。
- 迭代极快：近 7 天 2,770 个提交、941 个合并，伙伴 V1 待办、Cindy Make 等功能已先后下线。

# Paseo：一个 daemon，所有终端

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-23.jpg>)

## 定位与商业模式

Paseo 是五款中最早出现的一款（仓库创建于 2025-10-13），也是唯一由一个人维护的项目（维护者 Mohamed Boudra，6,838 个提交里 5,428 个出自他本人，784 个来自 bot）。README 的定位是「Claude Code、Codex、Copilot、OpenCode 和 Pi agents 的统一界面」，核心主张有五条：自托管（agent 在你自己的机器上用完整的本地环境运行）、多提供商、语音控制、跨设备（iOS、Android、桌面、Web、CLI）、隐私优先（无遥测、无追踪、不强制登录，中继可选且端到端加密）。

它的结构和 Docker 很像：一个叫 daemon 的本地服务管理所有 agent，桌面 App、手机 App、Web 和 CLI 都是连到 daemon 的客户端。CLI 的用法也是 Docker 风格：`paseo run`、`paseo ls`、`paseo attach`、`paseo send`，加 `--host` 就能操作远程 daemon。文档把用户分成两类：不懂服务器和 VPN、装桌面版扫码即用的人，以及有家用服务器、自建网络、做自动化的人。

许可证是 Apache-2.0（GitHub API 显示 NOASSERTION，是因为 LICENSE 文件开头先声明了第三方组件的例外）。收入来自 GitHub Sponsors、公司月度赞助位和 Paseo Hub：Hub 是一个开源的「自动化中心」，可以按 GitHub issue 标签等事件触发 agent 在连接的机器上干活，自托管免费，托管版 Free 每月 50 次运行，Pro 每席位每月 $15（取自仓库内的价格样例，线上档位可能不同）。维护者称拒绝过融资，靠赞助和 Hub 养活开发。

## 架构：像 Docker 一样的 daemon + 多客户端

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-24.jpg>)

daemon（`packages/server`）为每个连接的客户端建一个 Session，所有 agent 由 AgentManager 统一管理，它既是状态机，也是唯一事实来源。工具目录与传输方式解耦，MCP 只是其中一种适配：daemon 在 `/mcp/agents` 提供约 40 个管理工具，注入给每个 agent，于是「agent 编排 agent」成了一等能力。终端跑在独立的 worker 进程里，daemon 升级或崩溃时终端不断。持久化很轻：daemon 端是 Zod 校验、原子写入的 JSON 文件，没有迁移框架；时间线只在内存里，持久的事实来源是 provider 自己保存的会话历史。

接入方式和 T3 Code 类似：Claude 用 Agent SDK（权限请求走 `canUseTool` 回调），Codex 用 app-server JSON-RPC，Copilot、Cursor、Kimi、Kiro 走 ACP，OpenCode 用 `opencode serve` 加 SDK，Pi 用 rpc。各家权限请求统一成 tool / plan / question / mode 四种，回复只有 allow 或 deny；父 agent 可以代子 agent 回复。自定义 provider 只需在配置里用 `extends` 继承内置 provider 或 `"acp"`，就能换可执行文件、换 API 端点（如 [Z.AI](https://Z.AI)、通义）。

Paseo 有一个其他产品少见的工程约束：**App 和 daemon 版本经常不一致**（App 走应用商店更新，daemon 由用户自己决定何时升级），所以协议定了两份契约。协议契约要求新老版本必须能互相解析：新字段一律可选，不收窄类型，不删字段。功能契约要求客户端只在握手时看一次 `server_info.features`，daemon 不支持就提示升级，不写降级路径。所有兼容垫片都打上「COMPAT…remove after 日期」标签，非测试代码里约有 676 处。

## 远程：中继被当作不可信

![Image](<Agent%20工作台产品设计与系统架构调研.assets/image-25.jpg>)

Paseo 的远程设计以「不信任中继」为前提。daemon 首次启动生成 Curve25519 密钥对，配对链接（形如 `[app.paseo.sh/#offer=](https://app.paseo.sh/#offer=)…`）把公钥、serverId 和中继地址放在 URL 的 `#` 片段里，不会发到任何服务器，二维码就是信任锚。之后每个客户端连接都做一次 Noise_NK_25519_ChaChaPoly_SHA256 握手，每帧带计数器加密以防重放，密码也在加密通道内传输。中继能看到 IP、时序、消息大小、session ID 和公钥，读不到内容，也无法伪造或重放。中继默认关闭，用户同意配对后才开启；也可以走 TCP、Tailscale 或 SSH 隧道直连，暴露到外网时需要自己设密码。生产中继是另一个仓库里的 Elixir 分布式服务 `getpaseo/paseo-relay`。

**🛑 小心**  
**注意（推断）**：推送正文带有回复或权限请求的摘要，经 Expo 推送服务器（[exp.host](https://exp.host)）转到 APNs / FCM，这部分不在端到端加密范围内，和「代码不离开本机」的说法有出入。依据是 `push-service.ts` 与 `agent-attention-notification.ts` 的实现。

## 编排 skills、语音与通知

Paseo 的多 agent 能力不靠专门的编排引擎，而是三个 Markdown skill 加 daemon 注入的 MCP 工具：agent 先调 `list_profiles` 选配置，按需 `create_workspace`（可建 worktree），再 `create_agent`。新 agent 自动成为子 agent，完成时默认通知父 agent，父 agent 只等通知、不轮询；向子 agent 投递消息有 queue / steer / interrupt 三种方式。三个 skill 的区别在于父 agent 交出任务后的姿态：

| skill | 做法 | 典型用法 |
| --- | --- | --- |
| /paseo-handoff | 把任务连同上下文交给另一个 agent，交出后不再等结果 | 先和 Claude 规划，再交给 Codex 实现 |
| /paseo-advisor | 启动一个 agent 当顾问，等它给意见，并要求它不改文件 | 拿第二意见，但不委托工作 |
| /paseo-committee | 并行启动两个不同家族的 agent，父 agent 在两者之间互相转述论点，直到收敛 | 退一步做根因分析并产出计划 |

语音模式默认在本地处理：sherpa-onnx 跑 parakeet 识别、kokoro 合成，Silero VAD 判断用户是否说完；理解与执行复用本机已登录的 claude、codex 或 opencode，以隐藏会话运行，这个语音 agent 只有「说话」工具会被自动放行。通知按「在场」分级：用户正在看这个 agent 时不打扰；180 秒内有活跃客户端只发应用内提醒；否则才经 Expo 发系统推送。

## 关键设计与它解决的问题

| 设计 | 解决的问题 | 证据 |
| --- | --- | --- |
| daemon 与客户端分离，共用一套协议 | 换设备不用重建环境，各端看到同一批 agent | docs/architecture.md |
| 包装原生 CLI，不保管 provider 密钥 | 复用用户已有的订阅、登录和配置 | architecture.md；SECURITY.md |
| 不信任中继：Noise NK + 二维码配对 | 不开端口也能远程访问，中继读不到代码 | SECURITY.md；relay/src/noise |
| 协议契约与功能契约分开 | App 与 daemon 版本不一致时互相解析失败或行为不确定 | docs/protocol-compatibility.md |
| 终端放在独立 worker 进程 | daemon 升级或崩溃导致终端中断 | docs/terminal-worker.md |
| 工具目录与传输解耦，经 MCP 注入 | 跨 provider 让 agent 编排 agent | docs/providers.md |
| 完成订阅 + 三种投递方式 | 父 agent 不用轮询，子 agent 也不被打断 | docs/agent-lifecycle.md |
| 按在场决定推送 | 多端重复提醒 | agent-attention-policy.ts |
| 分阶段灰度（5% → 18 小时全量） | 发布频繁时坏版本影响面过大 | docs/release.md |

## 亮点与代价

**亮点**

- 隐私与信任模型最干净：无遥测、不强制账号、端到端加密中继，二维码作信任锚。
- 可脚本化：Docker 风格的 CLI 和 SDK，能在服务器上无头运行，也能被其他 agent 调用。
- 编排做得轻：三个 skill 就覆盖了交接、顾问、会商三种协作姿态。
- 单人维护却节奏极快：自 2026-02 以来 148 个版本，11 月 3～23 日发了 12 个；PR 不附 QA 证据会被直接关闭。

**代价**

- 巴士因子为 1：项目高度依赖一名维护者，README 也提醒 Issue 不一定及时处理。
- 刻意不做沙箱：agent、终端共用 workspace 权限，插件不隔离，安全边界只在「谁能连上 daemon」。
- 推送正文经 Expo，不在端到端加密范围内（推断）。
- daemon 端持久化很轻（JSON 文件、无迁移框架），时间线依赖 provider 自己的会话历史。

# Herdr：让终端比窗口活得更久

[🖼️ Herdr：让终端比窗口活得更久](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-25.png)

## 从 TUI 到「运行时 + 窗口」

Herdr 的前身是 herdr（本文称 Herdr 1）：一个终端复用器风格的 TUI，加一个后台 server，有 prefix 键和插件市场，重启后能恢复布局并「恢复受支持 agent 的会话」。它 2026-03 创建，半年多拿到 52.9k Star，末版 v0.9.5 发布于 2026-10-14。两天后团队建了两个新仓库，2026-10-23 首发：

- **herdr-core**（命令 `hc`）：无界面的终端运行时。README 第一句是「Core draws nothing」：它让程序和 coding agent 运行在你离开之后仍然存活的终端里，agent 可以启动别的 agent、给它们派活、等它们回报，你可以随时接入任何一个终端并接管。
- **herdr-app**：Rust + GPUI 写的原生桌面窗口，只是 herdr-core 的一个客户端。README 的原话是「窗口负责排布终端，core 负责运行它们」。

拆分有三条明确的原则：core 是运行时而不是复用器，只在有布局时才有意义的功能不进 core；core 必须是新写的代码，不得从 Herdr 1 拷贝；状态只认程序自报，不做屏幕检测。拆分动因来自团队博客：读屏检测每逢 agent 界面改版就失效，每次修复都要发版；有些想法在终端里很难实现，需要更多种类的客户端。现在官方有手机端（iPhone、iPad、Android，经 SSH 直连 core，免费、无账号），社区也已经出现 hc-tui、rodeo、herdr-mobile 等客户端。代价是 TUI 与新 app 不共享 agent，用户一度分不清手机读的是哪个运行时。

商业上，Herdr 全部 Apache-2.0，免费、无账号，团队承诺「runtime stays free」。公司入选 YC F26，完成 600 万美元种子轮，下一步是 Herdr Cloud：连接各台机器的端到端加密中间层，已开放候补名单。herdr-app 默认开启遥测（每天一次，不含终端内容和路径），core 不采集任何数据。

## 分层与进程模型

[🖼️ 分层与进程模型](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-26.jpg)

herdr-core 的进程链是：短命的 `hc` CLI → 每个 session 一个的 core daemon → holder 进程。**holder 是整个设计的支点**：它持有全部 PTY，是所有程序的父进程，自身不解析字节、不做任何策略；它为每个终端保留约 1 MB 的输出尾巴，core 崩溃后据此重建屏幕，并负责重启崩溃的 core。core 为每个活终端维护一份 libghostty-vt 屏幕（取自 Ghostty 上游，用 Zig 编译），用于代答终端查询、给新查看者发首帧、`hc read` 和解析 OSC 7501。

更新时终端不断，靠的是 holder 的交接协议：最新发来 Hello 的 core 接管 holder；新终端交给新版本的 holder，旧 holder 收到 DRAIN 后不再接新终端，等最后一个终端结束再退出。CI 每次都跑「从上一版升级」测试。`--keep` 的恢复语义很克制：程序丢失（崩溃、重启、信号致死）后，在原目录起一个新 shell、地址不变，**原命令绝不重跑**，因为重跑一个有副作用的任务比丢掉它更危险。

对外接口只有一套：OpenRPC 描述的 36 个方法（只增不减，用已发布版本的 schema 拦截破坏性变更），通过本地 socket 或 `hc connect` 承载。`hc connect` 在标准输入输出上跑帧协议，所以 SSH 天然就是传输层：本地 core 为每台远程机器保持一条帧连接，把远程终端并入同一个列表，地址形如 `vps/t7`，程序、历史和文件都留在远端。窗口的布局也存在 core 的文档里（带版本号做 CAS），已读和置顶存在终端 label 上，所以桌面和手机看到的是同一个世界，只有焦点和缩放留在各自屏幕。

## 状态只认程序自报：OSC 7501

Herdr 最有辨识度的决定是：**agent 状态只来自 OSC 7501**。这是 Ghostty 作者 Mitchell Hashimoto 在 2026-10-06 发起的「程序状态协议」，程序用一条终端转义序列自报 state（idle / working / done / blocked / error）、kind（permission / question / auth）、进度、标题与消息。herdr-core 的 AGENTS.md 写得很绝对：「No screen detection, no agent registry」。core 不读屏、不识别 agent 品牌，不报状态的程序就显示「无状态」，而不是「猜错状态」。

| 维度 | 读屏 + 厂商 hook（Herdr 1、多数工作台） | 程序自报 OSC 7501（Herdr core） |
| --- | --- | --- |
| 准确性 | 依赖屏幕文本模式，agent 改版就可能误判 | 程序自己说，语义明确 |
| 维护成本 | 每个 agent 维护一套规则，修复要发版或热更新 | 零规则，运行时不认识任何 agent 品牌 |
| 覆盖面 | 不改 agent 就能用 | 依赖 agent 采纳协议；未采纳的显示「无状态」 |
| 跨机器 | hook 通常回连本机端口，远程需要额外转发 | 随字节流带内传输，经 SSH 多跳天然可用（tmux 内无法透传） |

Orca 的做法是两者兼顾：hooks 优先，hooks 沉默 2 分钟后由 OSC 7501 接管，再往下是终端标题和热更新规则。这代表了两种产品哲学：Herdr 押注协议，宁可暂时少显示；Orca 押注覆盖面，愿意承担维护成本。

## agent 派 agent：父子原语

[🖼️ agent 派 agent：父子原语](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-27.jpg)

多代理协作是 core 的一等原语。template 是一个 Markdown 文件，frontmatter 只写命令、就绪条件、提交方式等少数几项，正文加上任务就合成首条消息，默认等子 agent 首次报告状态后才发送。从 hc 终端里启动的程序自动成为子 agent，默认随父 agent 一起结束（`--outlive-parent` 例外），父 agent 用 `hc wait --children` 等待。子 agent「自己说不出口」的情况（blocked、出错、退出）由 core 代为通知父 agent，格式是一条 `<hc>…</hc>` 消息直接打进父 agent 的输入。

这套原语里有两个对「人」的保护值得一提。一是 **draft wait**：send 和 note 都会等用户写到一半的草稿，最长从最后一次按键起 60 秒，避免 agent 的消息插进人正在输入的内容。二是 **blocked 保护**：目标处于 blocked 时，`hc send` 默认拒绝发送，防止父 agent 替人回答授权请求（文档也明确说这不是访问边界）。

## herdr-app：界面即文件

[🖼️ herdr-app：界面即文件](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-28.png)

herdr-app 的模型是 space（一台机器上的一处工作）→ tab → pane。agents 侧栏把所有 space 里的 agent（包括别的 agent 启动的 worker）汇成一个列表，四种状态按优先级排列：Blocked（等你）> Done（在你没看时完成）> Working > Idle，界面分区名叫「Needs you」。没有 pane 的子 agent 点开后以 peek 形式浮在当前 tab 上，可以「Keep」成独立 tab；终端文本里的地址（`t7`、`vps/t7`）按住 Cmd / Ctrl 点击就能直接 peek。

它的扩展方式很激进：**界面即文件**。布局是 JSON，面板、菜单和对话框（包括默认侧栏）都由 mod 绘制，mod 是跑在 QuickJS 里的小 JavaScript 程序，用 JS 描述 GPUI 元素、由 Rust 绘制，没有 DOM。改文件即热重载，新版本坏了就保留上一个好版本。内置 3 个布局、21 个 mod，官网画廊里已有 312 个布局、576 个 mod。app 核心加功能前要先过「三问」，否则就做成 mod；`herdr-app --skill` 输出给 agent 的界面驾驶手册，让 agent 能帮用户改界面。代价是 mod 为受信代码而非沙箱，安装别人的 mod 前需要读安全说明。

## 关键设计与它解决的问题

| 设计 | 解决的问题 | 证据 |
| --- | --- | --- |
| holder 持有全部 PTY | 关窗口、core 崩溃或更新都不丢 agent | herdr-core holder.rs |
| --keep 只开新 shell，不重跑 | 重启后误重跑有副作用的任务 | herdr-core README |
| 状态只认 OSC 7501 | 读屏检测随 agent 改版失效，修复要发版 | herdr-core AGENTS.md；Herdr 博客 |
| parent note | 子 agent 卡在授权弹窗，父 agent 空等 | notes.rs |
| draft wait | agent 消息打断人正在输入的内容 | README；input.rs |
| blocked 时拒绝 send | 父 agent 替人回答授权 | skills/herdr-core/SKILL.md |
| 布局存 core 文档、已读存 label | 桌面和手机的布局、已读状态不一致 | herdr-app docs/labels.md |
| hc connect + SSH，OpenRPC 只增不减 | 第三方客户端与远程机器无需额外服务端；客户端随 core 升级失效 | schema/README.md |
| 界面即文件 + mods | 用户想按自己的方式排布窗口，甚至让 agent 帮忙改界面 | defaults/AGENTS.md |

## 亮点与代价

**亮点**

- 运行时最硬：holder 交接让 core 崩溃、更新、关窗都不影响终端，恢复语义绝不重跑任务。
- 协议最开放：OSC 7501 不绑定任何 agent，OpenRPC + `hc connect` 让任何人都能写客户端。
- 体积最小：herdr-core 只有 189 个文件，功能边界非常清楚。
- 原生性能：GPUI（Metal / Vulkan / D3D11）+ libghostty-vt，自带与 Ghostty 的对比基准。

**代价**

- 状态覆盖依赖 agent 采纳 OSC 7501，短期内能显示状态的 agent 可能比 Herdr 1 少（推断）。
- 只有终端，没有结构化 Chat、diff 评审、浏览器等「工作台」功能，需要靠 mods 与外部工具补。
- 远程只有 SSH，没有中继与推送（Herdr Cloud 规划中）。
- mods 是受信代码、app 默认开遥测；新旧两代产品并存，用户需要理解迁移关系。

# 横向对比：同一组问题的五种答案

前面五章分别讲了每款产品。这一章换一个方向，按问题逐个对比：怎么接 agent、运行时切在哪里、「需要你」怎么流转、怎么并行、怎么远程、怎么编排、怎么扩展、用什么技术、靠什么赚钱、有多热。

## Agent 接入与状态来源

[🖼️ Agent 接入与状态来源](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-29.jpg)

接入方式决定了工作台「知道多少」。PTY 包装只拿到字节流，工作台要么靠 agent 主动上报（hooks、OSC 7501），要么靠读屏猜测；结构化协议直接给出消息、工具调用和审批请求，工作台能渲染富界面、统一审批卡，代价是每接一家都要写适配器。五款产品里只有 Orca 两条路都走：终端模式保证「任何 CLI 都能跑」，Chat 模式给主流 agent 更好的体验，两者靠 session id 互转。T3 Code、Paseo、Cindy 选择了结构化路线，并都把 Claude（Agent SDK）和 Codex（app-server）作为一等公民；Herdr 只做 PTY，把「知道多少」完全交给 OSC 7501 协议。

## 运行时切在哪里

[🖼️ 运行时切在哪里](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-30.jpg)

这是五款产品之间最根本的架构差异。把运行时做成独立于界面的进程，换来三件事：关掉窗口 agent 不停；桌面、手机、CLI 连的是同一个运行时，看到同一个世界；把运行时放到远程机器上，体验和本地同构。代价是多了一个常驻进程要管理、升级和兼容。Herdr 走得最远，连「终端由谁持有」都单独拆成了 holder；Paseo 和 T3 Code 是经典的 daemon / server 模式；Orca 用一个跨升级存活的终端 daemon 保住 agent，运行时本身在 Electron 主进程里，远程时才换成 orcad；Cindy 的编排内核就在 Electron Main 里，是唯一的胖客户端。

## 「需要你」怎么流转

[🖼️ 「需要你」怎么流转](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-31.jpg)

五款产品对「人在回路」的理解高度一致：人的注意力是最稀缺的资源，系统要做的是在对的时间、用对的渠道把人叫回来，而不是让人一直盯着。差别在细节：Orca 桌面空闲 3 分钟后才推手机；Paseo 正在看就不打扰、180 秒内有活跃客户端只发应用内提醒；Herdr 只在 blocked 时一定提醒、完成只在你没在看时提醒；Cindy 让人在锁屏和手表上直接作答，但只有完整显示操作内容时才给「批准」按钮；T3 Code 的批准必须进 App 完成，但被 Snooze 的线程遇到审批会被唤醒。

## 并行与隔离

| 产品 | 隔离单位 | 环境准备 | 回滚与清理 |
| --- | --- | --- | --- |
| Orca | 每个任务一个 git worktree（或普通文件夹），可父子嵌套；执行位置可选本机 / SSH / 远程 Orca / 云 VM | orca.yaml setup 脚本；Shared Paths、.worktreeinclude 复制 .env 与依赖；可让 agent 等 setup 完成再启动 | 一键删除落选 worktree 与分支；可选闲置 30 天自动归档；空闲休眠 |
| T3 Code | 每个线程可选一个 worktree，一个 worktree 可被多个线程共用 | t3.json 的 runOnWorktreeCreate | 每个回合一个 checkpoint（隐藏 Git ref），可连文件一起回滚；PR 合并后自动 settle 并按规则清理 |
| Cindy | 任务属于一台电脑；协同的 Worker 可用独立 worktree | 复制任务到另一台电脑时带上未提交改动快照 | 副本权限回到需确认模式；任务归档 |
| Paseo | workspace = 某 daemon 上的一个 cwd；可从分支或 PR 新建或复用 worktree | setup 脚本 | 回退、Fork；归档可撤销，连带归档子 agent |
| Herdr | space = 一台机器上的一处工作；core 本身不管理 git worktree | template 规定 agent 的启动命令与角色 | --keep 只恢复 shell 不重跑；社区插件补 worktree 管理 |

四款编码向产品都把「一个任务一个 git worktree」作为默认答案，这已经是这一类产品的共识。值得注意的是 T3 Code 的 checkpoint：它把每个回合的工作区快照存在隐藏的 Git ref 里，既不往用户分支加 commit，又能精确回滚到任意回合，这让「默认全自主」变得可以接受。

## 远程与多端

[🖼️ 远程与多端](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-32.jpg)

远程链路的差别归根结底是信任模型的差别。Paseo 和 Orca 把中继当作不可信的管道，用端到端加密保证中继只见元数据；T3 Code 让 relay 当可信 broker，但把它挡在数据路径之外，数据走 Cloudflare 隧道；Cindy V1 的中继终止 TLS、可见明文，V2 才补上端到端加密；Herdr 干脆不要中继，只用 SSH。对团队选型来说，如果代码敏感、又需要手机远程，Paseo 和 Orca 的模型更让人放心；如果更在意开箱即用、不想管网络，T3 Connect 最省心。

## 多 Agent 编排

[🖼️ 多 Agent 编排](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-33.jpg)

四种模式并不互斥，成熟的产品往往同时提供几种。它们共同的设计要点有三个：子 agent 的结果要有明确的回报通道，父 agent 不轮询（Orca 的 `worker_done` 恰好一次、Paseo 的完成订阅、Herdr 的 parent note、Cindy 的欠回报计数）；子 agent 的权限不能超过父 agent，也不能替人回答授权（T3 Code 子代理权限上限、Herdr 的 blocked 保护）；无人值守时要有循环上限（Orca 的 watch-pr 最多自动修 2 次）。

## 扩展生态

[🖼️ 扩展生态](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-34.jpg)

扩展的取向反映了产品的开放程度。Herdr 最开放：连默认侧栏都是 mod，API 只增不减，任何人都能写客户端，代价是 mod 无沙箱。Orca 和 Cindy 走「插件 + 沙箱 + 市场」的应用商店路线，Orca 的四档执行级别和断网 webview 是五款里隔离最严的。Paseo 插件能力强但不隔离。T3 Code 不开放第三方插件，扩展点集中在 MCP 和 CLI 上，让外部 agent 来驱动它，而不是让插件住进它。

## 技术栈与工程体系

| 产品 | 桌面与界面 | 运行时与存储 | 移动端与云 | 工程体系 |
| --- | --- | --- | --- | --- |
| Orca | Electron 45.4.1；React 19.2 + zustand；Tailwind 4 + shadcn；xterm 6（WebGL）；Monaco | Node 24；node-pty + headless xterm；node:sqlite；claude-agent-sdk 0.3.330 | Expo 58 / RN 0.88.1；Hono、Cloud Run、Cloudflare Worker / DO；Swift、Rust 原生组件 | 67 个 workflow；1,280 个单测、10 条 golden E2E、76 个性能契约；每小时构建 |
| T3 Code | Electron 44.10.1；React 19.3 + React Compiler、TanStack Router；libghostty-vt WASM 终端 | Node 26；Effect 4.1.5；node:sqlite（WAL，150 个迁移）；Node SEA 单文件 | Expo 58 / RN 0.88.1；Cloudflare（Alchemy 部署）、Clerk、PlanetScale | 36 条自定义 lint 规则；176 份协议录制回放；2,311 个测试文件；A/B 性能基准 |
| Cindy | Electron 42.11.12；React 19；Vite 6；Electron Forge；Tauri 写的 Windows 更新器 | Node ≥22.13；better-sqlite3 + drizzle（166 个迁移）；Rust 编为 WASM 的 Agent-Net | Expo 58 / RN 0.88.1；服务端不开源 | dev / product / design 三套规则文档；设计证据与验收目录；DCO；Claude Code 辅助评审 |
| Paseo | Electron 44.2.0；Expo Web；fork 的 enriched-markdown 渲染 | Node 22.20；Express + ws + zod；JSON 文件存储；sherpa-onnx 本地语音 | Expo 57 / RN 0.86.3；Elixir 中继；Expo 推送 | lefthook + knip；约 1,786 个测试文件；Playwright 8 片 + Maestro；PR 必须附 QA 证据；分阶段灰度 |
| Herdr | Rust 1.95；GPUI（Zed 快照）+ taffy；Metal / wgpu-Vulkan / D3D11；QuickJS 跑 mods | tokio；libghostty-vt（Zig 0.16）；holder 进程 | 官方手机端经 SSH；无云（规划中） | just；生成式文档 + docs-check；每次 CI 跑升级测试；cargo-deny；与 Ghostty 对比基准 |

技术选型呈现两个阵营。四款 TypeScript 产品高度同构：Electron 桌面 + Expo / React Native 手机 + Node 运行时 + SQLite，并且都直接使用 Claude Agent SDK，说明「用 TS 写一个工作台」的技术路线已经成熟，差异化只能来自产品与架构。Herdr 是唯一的 Rust 原生方案，用 GPUI 和 Ghostty 的终端内核换取性能与常驻进程的稳定性。工程上，五家都在把约束机器化：T3 Code 的自定义 lint 规则、Orca 的性能契约、Cindy 的规则索引、Paseo 的兼容标签、Herdr 的生成式文档，背后都是同一个现实——这些仓库的大量代码由 agent 编写，规则必须能被机器检查。

## 商业与开源

| 产品 | 许可 | 收费点 | 依赖的云 | 遥测 |
| --- | --- | --- | --- | --- |
| Orca | MIT | 暂无；推测在 Orca Cloud | 中继、推送网关、bot 连接器（闭源） | PostHog，默认开启，可关；DO_NOT_TRACK |
| T3 Code | MIT + 严格商标政策 | T3 Connect 主机数与 T3 Chat 订阅挂钩 | T3 Connect（Cloudflare、Clerk） | 由 server 统一上报，可关 |
| Cindy | Apache-2.0（客户端） | 官方模型 Cindy AI 套餐与充值 | 账号、中继、伙伴对话层、官方模型、SkillHub | TapDB，默认开启，可关 |
| Paseo | Apache-2.0 | 赞助；Paseo Hub 托管版 | 可选：E2EE 中继、Expo 推送、Hub | 无 |
| Herdr | Apache-2.0 | 暂无；Herdr Cloud 规划中 | 无 | app 默认开启；core 不采集 |

## 规模与活跃度

[🖼️ 规模与活跃度](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-35.jpg)

Orca 在热度上遥遥领先，日均新增 Star 是第二名的两倍；T3 Code 的 Fork 数最多（14,112），这和它「不满意就 fork」的开源姿态以及严格的贡献政策相符：想要新功能的人更倾向于自己 fork。Paseo 起步最早但增长平稳，Cindy 起步最晚、增速和 herdr-app 相当，并在 2026-10-26 登上 GitHub Trending 第一。Herdr 的热度主要留在旧的 TUI 仓库，新的 core 与 app 刚发布一个多月。

# 评价与选型

## 各产品的长板、短板与适合的人

| 产品 | 最强的地方 | 主要风险 | 适合 |
| --- | --- | --- | --- |
| Orca | 功能面最广；评审回路最细；终端与 Chat 双形态；bot 让 agent 从 Slack / Linear 接活 | 复杂度高；默认全自主；bot 依赖闭源连接器；文档与代码有漂移 | 重度编码、想要一站式 IDE，并希望把 agent 接进团队协作流的人 |
| T3 Code | 架构最规整；provider 抽象最完整；性能与工程约束最严；远程开箱即用 | 桌面托管模式关窗即停；隧道非端到端加密（推断）；不开放第三方插件 | 同时用多家 agent、在意性能与体验、需要随时随地远程放行的开发者 |
| Cindy | 多设备最深；面向大众的体验；跨电脑借订阅、跨账号分额度；IM 与中文生态 | 依赖官方云；V1 中继可见明文；胖客户端；迭代快、功能常有下线 | 不只写代码、有多台设备、希望「开箱即用」的个人与小团队 |
| Paseo | 隐私与信任模型最干净；可脚本化 CLI 与 SDK；编排 skills 轻巧；发布节奏快 | 单人维护；不做沙箱；推送正文不在 E2EE 范围（推断） | 重视隐私、喜欢自托管和命令行、愿意接受个人项目风险的开发者 |
| Herdr | 运行时最硬；协议最开放；体积最小；原生性能 | 状态覆盖依赖 OSC 7501 普及；缺少结构化 Chat、评审等工作台功能；远程只有 SSH | 终端重度用户、想自己组合工具链或开发自己客户端的人 |

## 我该选哪一款

[🖼️ 我该选哪一款](https://img.riba2534.cn/images/2026/11/agent-workbench-20261127-36.jpg)

决策树只给出起点，实际上很多人会组合使用：例如用 Herdr 或 Paseo 的 daemon 跑常驻 agent，用 Orca 或 T3 Code 做评审和交付。五款产品都驱动用户自己的 CLI 与订阅，切换成本主要在习惯与工作流，而不在数据迁移。

## 可以借鉴的设计判断

如果要自己做一个 Agent 工作台，或者在团队内部工具里引入类似能力，下面这些判断在五款产品里被反复验证过：

1. **先把运行时从界面里拆出来**。agent 动辄运行几十分钟，界面的生命周期远短于任务。Herdr 的 holder、Orca 的终端 daemon、Paseo 的 terminal worker 都在回答同一个问题：谁对进程负责。
2. **状态是一等公民，优先让程序自报**。读屏猜状态是技术债，OSC 7501 与 hooks 才是长期解；同时要有「没有状态」的诚实表达，而不是猜一个。
3. **审批要统一，但不能伪造**。T3 Code 的「不用自动回答审批来假装支持某个权限模式」和 Herdr 的「blocked 时拒绝代答」，都是为了让安全承诺与实际行为一致。
4. **把人的注意力当最稀缺资源**。在场判定、空闲后才推送、批注成批回传、按「现在该谁动」分区，都是在减少打扰次数而不是加快打扰速度。
5. **回报必须有通道，循环必须有上限**。编排里最常见的两个故障是「父 agent 空等」和「无人值守死循环」，分别用完成通知和自动修复次数上限解决。
6. **中继默认不可信**。二维码作信任锚、配对信息放 URL 片段、Noise 握手，这套做法成本不高，却把云端从信任链里拿掉了。
7. **协议只增不减，兼容写成契约**。手机 App 与桌面运行时的版本必然不一致，Paseo 的两份契约和 Herdr 的 OpenRPC 冻结策略值得照搬。
8. **交接用代码而不是模型**。Cindy 用代码重放历史生成交接文本、T3 Code 按预算挑选上下文，既省钱又可预测。

**❗ 重要**  
**趋势判断（推断）**：一是**运行时化**，工作台的核心正从界面转向常驻运行时与开放协议（OSC 7501、ACP、OpenRPC），界面会越来越多样，甚至由 agent 自己改；二是**bot 化**，Orca Bots、Cindy 伙伴、Paseo Hub 都在让 agent 从 IM、工单、定时任务里直接接活，人从「发起者」变成「审批者」；三是**云端收费**，客户端开源免费成为标配，商业化集中在连接、托管与团队协作。