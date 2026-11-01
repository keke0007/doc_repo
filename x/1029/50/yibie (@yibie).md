---
author: "yibie"
handle: "@yibie"
source: "https://x.com/yibie/status/2114712183095624012"
date: 2026-10-26T13:34:08.000Z
type: thread
---
# yibie (@yibie)

Claude Code、Cursor 这类 agent 要替你干活，就得拿到 GitHub PAT、Stripe key、数据库密码。常见做法是把值直接写进 MCP 配置或 .env。  
  
agent 的上下文里从此躺着一串明文。  
  
mcp-warden 改的是这一步：密钥存进系统钥匙串，agent 只报得出一个预设名。  
  
一条请求怎么走。你在命令行把密钥写进钥匙串，值从 stdin 读，不进 argv。预设是一个具名作用域，等于基础 URL 加允许方法加整条路径正则。  
  
agent 要访问 API 时调 proxy_http，报预设名和路径；mcp-warden 先拿请求去过允许清单，过了才从钥匙串取密钥、注入、发往上游，只把响应还给 agent。全部跑在本机，Rust 写，Apache-2.0。  
  
所以提示注入能做的事变少了。它可以把结果带出去，但抄不出一串自己没见过的 token。会话记录贴出来，也没有密钥可漏。  
  
安全边界落在预设上，钥匙串只负责存放。README 自己指出的坑：预设不写 --allowed-paths 就等于放开 base_url 下所有路径，配经典 PAT 时 agent 能读这个 token 看得见的每个私有库。路径规则按整条路径锚定，不做前缀匹配，所以 ssh 那条写成 git pull.* 就等于放行 git pull; rm -rf /。  
  
审批和审计是配套的。默认策略是 prompt，要设 auto 必须在建预设时显式加 --confirm-auto。macOS 和 Windows 的桌面通知只负责提醒，真正的批准发生在 stderr 里的 TUI 弹窗。每个动作落进本机 SQLite，审计接口没有 delete / clear。按窗口放行的调用记成 windowed，事后能看出哪几条没让你再点一次。  
  
我的判断：它防的是提示注入骗 agent 把密钥发出去，agent 自己去翻钥匙串它不管。威胁模型 N2 写明，agent 没被沙箱隔离时，可以起子进程跑 security find-generic-password。所以它是凭证边界，要真隔离，还得把 agent 关进容器或虚拟机。另一处窄口是响应不做脱敏，放行 GET /me 就等于把邮箱这类信息交给 agent。  
  
要上手的话，preset scaffold 里有 8 个内置模板，或者用 preset from-openapi 从 OpenAPI 3.x 规格生成一个。先只配读操作，把批准窗口留在 10 分钟以内，隔几天跑一次 mcp-warden audit，比一次配齐写权限稳妥。  
  
原文链接在评论区  
  
#MCP #Agent #安全

![Image](<yibie%20(@yibie).assets/image-01.jpg>)

---

原文：  
[news.ycombinator.com/item?id=502414…](https://news.ycombinator.com/item?id=50241410)  
[github.com/ermoluk/mcp-wa…](https://github.com/ermoluk/mcp-warden)  
[github.com/ermoluk/mcp-wa…](https://github.com/ermoluk/mcp-warden/blob/main/docs/threat-model.md)