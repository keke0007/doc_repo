---
author: "陈成"
handle: "@chenchengpro"
source: "https://x.com/chenchengpro/status/2113567114544263591"
date: 2026-10-23T09:44:02.000Z
type: tweet
---
# 陈成 (@chenchengpro)

学习了下 OpenCode 2，记几个点。  
  
1/ 常驻后台服务。OpenCode 一直是 client/server 架构，但以前默认不用。2.0 起本机只跑一个后台进程，TUI、Web、桌面端都连它，开 5 个会话也只有这一个进程。Claude Code 就有这个问题，每个会话一个进程，开多了内存顶不住。Qoder CLI 最近加的 Daemon 模式也是解这个问题。  
  
2/ 新 API，从零重写，改动最大的是发消息。以前是发完等回复，现在是把输入持久写进会话的收件箱，然后马上返回。可以选 steer 插进当前这一轮，也可以选 queue 排队。客户端带上 id，断网重试也不会重复发。会话里发生的每件事都按顺序编号记下来，客户端断线重连时报上最后收到的编号，服务端把漏掉的补发过来。  
  
3/ Code Mode。模型写一段 JS 把工具串起来跑，只有最后 return 的结果进上下文。注意这个 code mode 在这里是给 MCP 用的，read、edit、shell 这些主力工具还是直接调。MCP 默认全走 code mode，prompt 里的工具清单控制在 2000 token 左右，没列出来的靠 search() 现查。我知道还有 Browser Use 和 Computer Use 的场景会用 code mode，OpenCode 2 自己的 browser 工具也确实放在里面。跑 JS 的是他们自己写的解释器，没用 QuickJS，也没用 V8 isolate。  
  
4/ 后台任务。子代理和 shell 都能放后台，跑完会通知主 agent。前台卡住了，按 ctrl+b 挪到后台。这个 Claude Code 和 Qoder CLI 里都早有了。  
  
5/ 插件化和热重载。80 多个内置功能都写成了内部插件，不想要哪个就关哪个。改配置、接 MCP、写插件都马上生效。模型可以当场给自己写个工具，同一轮就用上，不用重启，prompt cache 也不会失效。和 deepseek-harness 的 "Everything is a Plugin" 是一个方向。  
  
6/ SDK。OpenCode 2 的 SDK 的特点是和 CLI 用同一份代码，直接跑在你的进程里，不开端口。它能放到 Cloudflare Durable Object 里用，2.0 发布当天 Cloudflare 就出了 OpenCodeHarness。同一个目录下的不同会话还能挂不同的插件，比如每个 Slack thread 一套。API 有 Promise 和 Effect 两种写法。  
  
详细 research 见

> 🔗 [**OpenCode 2**](https://t.co/yMtrMpnSCs)
> 
> OpenCode 2 相比 v1 有哪些架构变化和亮点，社区怎么看？
> 
> _From research.sorrycc.dev_