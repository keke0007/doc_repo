---
author: "老白（每日 AI 干货）"
handle: "@laobaishare"
source: "https://x.com/laobaishare/status/2055181393404293543"
date: 2026-05-15T07:00:02.000Z
type: thread
likes: 785
reposts: 129
replies: 51
bookmarks: 1033
views: 146352
---
# 老白（每日 AI 干货） (@laobaishare)

牛 x 啊，一个印度独立开发者，  
  
做出了一个被程序员忽视了 20 年的终端工具。  
  
名字叫 witr。  
  
它能回答一个你的操作系统始终拒绝回答的问题：  
  
为什么这个进程正在运行？

![Image](<老白（每日%20AI%20干货）%20(@laobaishare).assets/image-01.jpg>)

---

你看到一个奇怪的进程在偷你的 RAM。  
  
你看到一个本不该开放的端口。  
  
你看到一个你完全不记得装过的服务。  
  
每一个工具都告诉你"它存在"。但没有一个工具告诉你"它为什么存在"。

---

ps 给你看进程。lsof 给你看端口。systemctl 给你看服务。  
  
但没有一个工具，能给你看完整的因果链。  
  
从生成 supervisor 的那个 shell、再到拉起守护进程的 supervisor、再到打开 socket 的那一环——这整条链子，没有人帮你串起来。  
  
witr 可以。

---

你敲一条命令，它就帮你把整条因果关系追下去。  
  
从内核 → PID → 父进程 → 拉起它的那个 launchd 任务或 systemd 单元；从一个开放端口 → 反向追到绑定它的二进制；从一个服务 → 追到触发它的那个用户 shell。  
  
操作系统藏起来的东西，witr 用人话告诉你。

---

和之前每一个"进程查看器"的本质区别：  
  
· 追完整因果链，不只是 PID 和端口  
· 交互式 TUI 仪表盘，不是一堵纯文本墙  
· 单个 Go 静态二进制，5 秒装完  
· 原生支持 Linux、macOS、Windows、FreeBSD  
· 已经被打包到 brew、conda、AUR、winget、npm、scoop、chocolatey、FreeBSD ports 等几乎所有发行渠道

---

从此消失的东西：  
  
每一个"这是什么进程"的 Stack Overflow 兔子洞、每一条"我 Mac 为什么在跑这玩意"的 Reddit 提问、每一段你从 2014 年的论坛复制下来的 PowerShell 一行命令——全部不需要了。

---

5 个月，15104 颗星，401 个 fork，34 位贡献者，18 个版本。Apache 2.0 协议。  
  
Pranshu Parmar 一个人，用一台笔记本就把它做出来了。  
  
没有 VC，没有团队，没有加速器。  
  
你的操作系统，终于愿意跟你说实话了。  
  
仓库：

> 🔗 [**GitHub - pranshuparmar/witr: Why is this running? Trace any process, port, container, or file back...**](https://t.co/QjihIgnEic)
> 
> _From github.com_