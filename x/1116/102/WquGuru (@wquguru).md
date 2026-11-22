---
author: "WquGuru"
handle: "@wquguru"
source: "https://x.com/wquguru/status/2121562681069211693"
date: 2026-11-14T11:15:34.000Z
type: thread
likes: 84
reposts: 5
replies: 20
bookmarks: 123
views: 20434
---
# WquGuru (@wquguru)

🎉 X 用 5 个工程师半年重做了整个Web端。  
  
今天下午很多朋友发现 X 的 Web 端有了很大的变化，这是开将近六个月的工作成果。  
  
5 个左右的工程师配合 coding agent，不到 6 个月做出了 [x.com](https://x.com/) 新 Web 端的第一版，并且把三套旧服务合并成一套👇  
  
每个数据中心省了一半机器，旧的 rweb SSR 关停直接删掉近 36,000 行代码。在网速慢+ 4 倍 CPU 降速的实验室测试里，打开从旧版 25 秒缩到 1.7 秒，快了 12.8 倍  
  
这个最佳实践可以直接抄进我们自己的本地仓库：  
  
第一，根目录放一个 AGENTS.md 文件定规矩  
  
里面写明强制具名导出、禁用 barrel 文件、在信任边界校验外部数据、哪些测试值得写，并且要求 lint、类型检查和相关测试全过才能提交。  
  
第二，在仓库里内置 25 个以上的 skill  
  
把写 commit、开 PR 和堆叠 PR、做代码 review、盯 PR 过 CI、改依赖、加翻译、写 Relay（Meta 的 GraphQL 客户端框架）和路由全部做成标准动作，不用每次手敲长提示词。  
  
第三，写专门抓 AI 坏习惯的静态检查插件  
  
这一步中，用 oxlint（基于 Rust 的极速前端代码检查工具）写了自定义插件查业务规范，还专门写了一个 anti-slop 插件，直接在语法树层面拒绝 agent 最爱写的偷懒写法，比如连环类型断言、把参数标成 unknown、条件展开空对象。再配合 tsrs（TypeScript 编译器的 Rust 移植版），在 pre-commit 阶段强制跑格式化、lint 和类型检查，pre-push 阶段检查引用方包。AI 写出烂代码连本地提交都过不去。  
  
第四，拿旧系统当可运行的规格说明  
  
重构最适合发挥 agent 的能力，让 agent 去读旧服务器的行为，移植过来，写测试钉死。他们还写了 parity 脚本，伪装成已验证的 Googlebot 同时请求新旧两边，自动比对状态码、重定向、标题、OG 卡片和结构化数据。  
  
主创工程师说，他们写代码的时间变少了，更多时间花在决定系统该做什么，然后去真实浏览器里验收。

![Image](<WquGuru%20(@wquguru).assets/image-01.jpg>)

---

X 官方工程博客原文在这，里面还有大量关于 React 流式 SSR、Node.js 多 worker 线程调优细节🫱

> ![Link card preview](<WquGuru%20(@wquguru).assets/image-02.png>)
> 
> 🔗 [**Rebuilding the global town square from scratch**](https://t.co/9HKr5geS0G)
> 
> _From blog.x.com_