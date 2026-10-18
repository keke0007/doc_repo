---
author: "Livid"
handle: "@Livid"
source: "https://x.com/Livid/status/2088813490920391025"
date: 2026-08-16T02:21:59.000Z
type: article
likes: 133
reposts: 15
replies: 12
bookmarks: 179
views: 54191
---
# 这是我写了自用的 Harness 环境，长这个样子

*By Livid (@Livid)*

![Banner](<这是我写了自用的%20Harness%20环境，长这个样子.assets/image-01.jpg>)

- 部署时是单个 go 二进制文件
- 提供 web 界面（端口 7777）及 ssh 界面（端口 2222），web 界面风格是 Mac OS 9 Platinum
- 用系统上最原生的方式启动 VM：在 macOS 上是 Virtualization.framework，Linux 上是 Firecracker，Windows 上是 QEMU on WHPX
- 然后可以直接用 LLM 和启动的 VM 聊天，LLM 后端目前支持 Ollama 或者是 Codex 订阅登录
- 在 VM 里做的 vibe apps，可以直接通过 Cloudflare Tunnel 发布成 https 域名
- 每个节点有一个自己的 Workspace 文件夹，可以多个节点之间用 Mesh 方式同步 Workspace
- 环境里自带了一些常用 apps，比如 Notes 和 Todo，数据也是在多个节点之间 Mesh 同步

![Image](<这是我写了自用的%20Harness%20环境，长这个样子.assets/image-02.jpg>)

---

比如这是一个完全用这个环境构建和发布的应用——Ollama Cloud 的性能监控：

[ocperf.v2core.com/](https://ocperf.v2core.com/)

---

项目链接 [github.com/livid/exe](https://github.com/livid/exe)

这个时代，只要有 idea 和决心，每个人都可以把自己最想要的软件做出来。

这个软件或许不一定会满足一百万人的需求，要维护那种需要满足一百万人需求的软件也会很累。

这种 personal software 就很像画画。

每次有灵感（需求）时，就去涂抹两笔，最后就会越来越接近自己心中想要的。