---
author: "meow"
handle: "@meow2149"
source: "https://x.com/meow2149/status/2122019853284659443"
date: 2026-11-15T17:32:12.000Z
type: thread
likes: 93
reposts: 12
replies: 7
bookmarks: 98
views: 9097
---
# meow (@meow2149)

pi和dsh都支持通过插件深度定制agent。  
  
具体怎么选，可以从底层机制、同类插件、生态规模和使用习惯这四个方面看。  
  
一、底层机制  
  
▪︎ Pi  
  
pi是一个精简、可扩展的agent底座，主要通过TypeScript扩展增加能力。  
  
扩展可以注册工具、添加命令、监听生命周期事件、拦截工具调用、调整上下文处理，也能定制终端界面。  
  
扩展、skills、提示词模板和主题，可以打包成Pi Package，通过npm或Git安装。除了终端交互，pi还提供SDK和RPC，方便接入自己的程序。  
  
▪︎ DSH  
  
dsh是DeepSeek Harness，底层使用Cordis组织插件，核心思路是“Everything is a Plugin”。  
  
模型适配、工具注册、会话日志，甚至agent循环本身，都是插件，可以通过配置替换。  
  
插件也能扩展网页界面，加入配置页、任务面板和新的交互入口。整套运行环境由插件组合起来，同时提供Web、无界面运行和SDK等形式。  
  
二、同类插件对标  
  
下面按照相同的核心用途对照，具体功能各有侧重。  
  
▪︎ 多agent分工：pi-subagents ↔ dsh-agent-teams  
  
两者都能把任务分配给多个agent，再把结果汇总回来。  
  
pi-subagents支持专门的子agent、并行或串联工作流，以及后台任务，适合代码搜索、功能实现和代码审查。  
  
dsh-agent-teams提供队长、成员、任务依赖、成员通信和调度，并通过可视化面板展示团队进度。  
  
▪︎ 联网搜索与网页读取：pi-web-access ↔ dsh-free-search  
  
两者都能让agent搜索网络、获取网页内容，并支持多个搜索来源。  
  
pi-web-access还扩展了GitHub仓库处理和部分视频内容分析。  
  
这里的dsh-free-search指DDDMUC维护的项目，提供引擎切换、密钥配置、回退顺序和连通性测试界面，也支持免密钥的搜索路径。  
  
▪︎ MCP服务接入与管理：pi-mcp-adapter ↔ dsh-mcp-connector  
  
两者都能接入MCP服务，让agent发现并调用外部工具。  
  
pi-mcp-adapter进一步提供服务按需启动、工具按需发现，以及上下文占用控制。  
  
dsh-mcp-connector提供连接器目录、授权管理、跨连接工具搜索和连接排障界面。  
  
Pi现在已经内置MCP，DSH也有官方MCP客户端。这两个社区插件提供的是进一步增强。  
  
三、插件生态规模  
  
截至11月16日，pi官方包目录收录5,346项，包括扩展、skills、提示词模板和主题。  
  
dsh-market社区目录收录11,137项，其中5,601项有npm下载数据。  
  
按这两份公开目录的收录量，dsh领先。  
  
但统计口径不同：pi目录统计npm包，dsh目录同时收录npm和GitHub分发的插件。pi也支持Git安装，所以不能直接说dsh的实际插件数量是pi的两倍。  
  
pi的核心插件也已经有很高的下载活跃度：pi-mcp-adapter月下载约210万，pi-web-access约140万，pi-subagents约97万。下载量包含自动化流量，不等于用户数。  
  
插件管理方面，dsh-market已经支持在界面里搜索、安装、升级、卸载插件和切换主题。  
  
把公开目录规模和图形化分发放在一起评，我认为dsh目前的插件生态更有优势。  
  
四、如何选择适合自己的  
  
▪︎ 主要在终端里工作，希望自己调整工具、命令和上下文处理，或者通过SDK把agent接进自己的程序，可以优先选pi。  
  
▪︎ 希望通过图形界面安装和管理扩展，把搜索、连接器、多agent任务等能力组织进一个工作台，可以优先选dsh。  
  
如果从零开始，主要依据是现成插件供给和图形化管理，我会先试dsh。  
  
如果已经围绕终端建立了开发流程，需要灵活组合扩展，pi依然很合适。

![Image](<meow%20(@meow2149).assets/image-01.jpg>)

---

下一期介绍 Pi 和 OMP 明明是同源的两个 Coding Agent，为什么却走出了两条不同的路线？

> **meow (@meow2149)**
> 
> Pi 和 OMP 明明是同源的两个 Coding Agent，现在却走出了两条不同的路线。  
>   
> 很多人可能不知道，OMP（Oh My Pi）并不是从零开发的竞品，而是从 Mario Zechner 的 Pi 项目分叉出来的。  
>   
> 两者都保留了终端交互、TypeScript 扩展、会话管理和多模型支持，但后续的发展方向明显不同。  
>   
> Pi
> 
> ![Image](<meow%20(@meow2149).assets/image-02.jpg>)