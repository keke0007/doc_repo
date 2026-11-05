---
author: "meow"
handle: "@meow2149"
source: "https://x.com/meow2149/status/2117448048150548729"
date: 2026-11-03T02:45:29.000Z
type: thread
likes: 24
reposts: 2
replies: 8
bookmarks: 33
views: 4601
---
# meow (@meow2149)

每天介绍一个开发者工具，今天是 Orca。  
  
前两期介绍了 Herdr 和 Magpie，分别用于管理多个 AI Coding Agent，以及它们使用的模型和账号。  
  
今天介绍的 Orca，同样支持 Claude Code、Codex、OpenCode 等 Agent，但它更接近一款完整的 Agent IDE。  
  
除了集中运行多个 Agent，还把 Git Worktree、代码编辑、Diff 审查和浏览器整合到了一起，重点解决多个 Agent 并行开发时的代码隔离与管理问题。  
  
Orca 由 Stably AI 开发，项目采用 MIT 协议，提供 macOS、Windows 和 Linux 桌面客户端，并有 iOS、Android 移动端配套应用。  
  
它最核心的功能是 Git Worktree 隔离。  
  
假设同时让三个 Agent 开发同一个项目：  
  
▪︎ Claude Code 负责重构登录模块  
▪︎ Codex 负责修复支付接口  
▪︎ OpenCode 负责补充单元测试  
Orca 可以为每个任务创建独立的 Git Worktree，让不同 Agent 在各自的工作目录和分支中修改代码，避免直接覆盖彼此的文件。  
也可以让多个 Agent 分别尝试解决同一个问题，再比较它们的代码修改，选择合适的方案合并。  
  
不过，Worktree 只是隔离工作目录，并不意味着最终合并时不会出现 Git 冲突。  
  
代码审查也是 Orca 的重点。  
  
它内置基于 Monaco 的编辑器，可以直接查看 Agent 修改的文件和 Git Diff。  
  
其中 Annotate AI Diff 功能允许直接在具体代码行上添加审查意见。  
  
例如 Codex 修改了一个函数，但缺少异常处理和单元测试。  
  
可以直接在对应代码行添加评论，再通过 Send to agent 将这些意见及代码位置一起发送给 Codex，让它继续修改。  
  
不需要手动复制代码、整理行号，再重新描述问题。  
  
Orca 还支持 Git 暂存、提交、推送、创建 PR，以及在三方对比界面中处理合并冲突。  
  
对于前端开发，Orca 还提供了 Design Mode。  
  
它集成了 Chromium 浏览器，可以直接预览本地运行的网站。  
  
开启 Design Mode 后，点击页面上的按钮、文字或其他元素，就能提取对应的 DOM 信息、CSS 样式和局部截图。  
  
如果 Source Map 可用，还能关联到具体源码位置。  
  
这些信息可以直接发送给 Coding Agent。  
  
例如选中登录按钮，要求 Agent 调整高度、颜色和移动端布局，就不需要再手动查找对应组件。  
  
多个 Agent 的状态也可以集中查看。  
  
Orca 可以显示 Agent 正在工作、等待输入、任务完成或处于空闲状态。  
  
实验性的 Agent Dashboard 还提供了任务看板，方便同时跟踪多个工作区。  
  
此外，Orca 提供了 Orchestration 功能，可以将工作拆分成不同任务，由多个 Agent 协作完成。  
  
例如让一个 Agent 编写代码，另一个 Agent 负责审查，再根据审查结果安排后续修改。  
  
不过，官方目前仍将这套编排功能标为实验性，需要手动开启。  
  
远程开发也在支持范围内。  
  
Orca 可以通过 SSH 管理远程机器上的工作区和 Agent，也支持连接自建远程运行时。  
  
对于需要长时间执行的构建、测试和开发任务，可以放到远程服务器运行，再通过本地客户端查看。  
  
移动端则可以用来查看 Agent 状态、读取终端输出、回复确认请求，以及浏览代码修改。  
  
另外，Orca 提供了 CLI，Agent 可以通过命令创建 Worktree、操作终端、获取浏览器快照和执行页面交互。  
  
这意味着 Agent 不仅可以在 Orca 中运行，也能够调用 Orca 提供的开发环境能力。  
  
它和前两期介绍的 Herdr、Magpie 有什么区别？  
  
▪︎ Herdr 侧重 Agent 的进程管理、终端会话和跨 Agent 通信，可以通过 hc 命令让不同 Agent 互相调用、传递任务和返回结果。  
  
▪︎ Magpie 侧重模型和账号管理，通过本地网关实现协议转换、模型切换、账号路由与额度管理。  
  
▪︎ Orca 侧重完整的开发工作流，将 Agent 运行、Worktree 隔离、代码编辑、Diff 审查和 Git 操作整合在一个 IDE 中。  
  
如果只需要集中管理多个终端，Herdr 就能覆盖不少需求；如果需要切换模型和订阅账号，Magpie 更有针对性。  
  
而 Orca 更适合经常并行开发多个功能、需要检查和合并不同 Agent 代码的开发者。  
  
使用前还有几个细节需要注意：  
  
▪︎ Git Worktree 不是安全沙箱，不会自动隔离系统权限和本机凭据。  
  
▪︎ 部分 Agent 的默认启动参数会跳过权限审批，建议在设置中检查权限模式，避免 Agent 执行未经确认的高风险操作。  
  
▪︎ 关闭桌面客户端后，后台运行时可以继续管理任务，但电脑重启或运行时终止后，正在执行的进程不会自动恢复。  
  
▪︎ 不同 Agent 的状态检测、额度统计等集成能力存在差异，并非所有 CLI 都能获得完整支持。  
  
▪︎ Orca 本身免费开源，但 Claude Code、Codex 等工具的订阅和 API 费用仍需自行承担。  
  
对于同时使用多个 Coding Agent 的开发者，Orca 提供的不只是统一的终端界面，还包括从任务隔离、代码修改、人工审查到最终提交的一整套开发工作流。  
  
官网：[onorca.dev](https://onorca.dev)  
GitHub：[github.com/stablyai/orca](https://github.com/stablyai/orca)  
文档：[onorca.dev/docs](https://onorca.dev/docs)  
  
#AI #CodingAgent #OpenSource

> ![Link card preview](<meow%20(@meow2149).assets/image-01.jpg>)
> 
> 🔗 [**Orca — The agent development environment**](https://t.co/4XxqaXfQgq)
> 
> _From onorca.dev_

---

上一期在这里：

> **meow (@meow2149)**
> 
> 每天介绍一个开发者工具，今天是 Magpie。  
>   
> 上一期介绍了 Herdr，用来管理多个 AI Coding Agent 的终端和会话。今天介绍的 Magpie，管的是这些 Agent 使用的模型和账号。  
>   
> 比如，你已经习惯 Claude Code 的操作方式，但想试试 Kimi 写代码；或者想继续用 Codex，同时接入 DeepSeek。Magpie

---

下一期在这里：

> **meow (@meow2149)**
> 
> 每天介绍一个开发者工具，今天是 T3 Code。  
>   
> 前面三期介绍了 Herdr、Magpie 和 Orca，分别侧重多个 AI Coding Agent 的终端管理、模型与账号管理，以及包含 Git Worktree 和代码审查的完整开发工作流。  
>   
> 今天介绍的 T3 Code，同样能够管理 Claude Code、Codex、OpenCode 等

---

AI Coding Agent 相关的工具汇总：

> **meow (@meow2149)**
> 
> 最近 AI Coding Agent 相关的工具真的越来越多了，感觉每周都能冒出来好几个新项目，而且迭代速度一个比一个快。  
>   
> 前面已经陆续介绍了 Herdr、Magpie、Orca 和 T3 Code，最近又关注了不少类似的工具，干脆把这些项目放在一起整理一下：  
>   
> ▪︎ Herdr — Can Celik  
> Agent 终端管理工具，支持多 Agent