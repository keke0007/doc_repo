---
author: "meow"
handle: "@meow2149"
source: "https://x.com/meow2149/status/2121748868413231598"
date: 2026-11-14T23:35:25.000Z
type: thread
likes: 85
reposts: 16
replies: 13
bookmarks: 194
views: 15947
---
# meow (@meow2149)

整理了7组agent工具替代方案，从模型管理到桌面自动化，按使用场景直接找。  
  
一、模型管理：CC Switch → Magpie  
  
▪︎ CC Switch：管理和切换不同agent的供应商配置，也支持本地路由。  
  
▪︎ Magpie：通过统一网关，集中管理多个agent的模型、账号、协议转换和请求路由。  
  
二、终端工作区：iTerm2 → cmux、Herdr  
  
▪︎ iTerm2：macOS终端，支持标签页和分屏，用来运行命令、管理终端窗口。  
  
▪︎ cmux：围绕agent组织工作区，整合分屏和任务通知，方便查看哪些任务需要处理。  
  
▪︎ Herdr：管理终端会话和运行状态，窗口关闭后，后台任务仍可继续运行。  
  
三、终端编程：Aider → Pi、OMP、OpenCode  
  
▪︎ Aider：终端里的AI结对编程工具，通过模型协助阅读和修改代码。  
  
▪︎ Pi：精简、可扩展的编程agent，支持自定义工具、命令和工作流。  
  
▪︎ OMP：Pi的增强分支，加入Hashline编辑、语言服务和调试能力。  
  
▪︎ OpenCode：支持多家模型供应商，提供终端、Web等交互入口。  
  
四、并行开发：Conductor → Paradev、Emdash、Parallel Code  
  
▪︎ Conductor：在Mac上通过独立工作区并行运行coding agent，集中审查代码改动。  
  
▪︎ Paradev：原名Superset，定位为Conductor的开源替代，整合终端、代码审查和浏览器预览。  
  
▪︎ Emdash：接入不同coding agent，在独立工作区中并行处理任务。  
  
▪︎ Parallel Code：为任务创建独立分支和Git Worktree，集中管理多个agent及其代码改动。  
  
五、远程访问：Happy → Paseo、Cindy  
  
▪︎ Happy：通过手机或浏览器，远程查看和控制电脑上的coding agent会话。  
  
▪︎ Paseo：跨设备访问开发工作区，查看会话、发送指令、审查代码。  
  
▪︎ Cindy：统一接入Claude Code、Codex和Pi，支持跨设备继续处理任务。  
  
六、个人助手：OpenClaw → Hermes Agent  
  
▪︎ OpenClaw：可自部署的个人agent，通过消息平台接收指令、调用工具并执行任务。  
  
▪︎ Hermes Agent：支持记忆、Skill和定时任务，也提供从OpenClaw导入部分配置、记忆和技能的迁移路径。  
  
七、桌面自动化：PyAutoGUI → agent-desktop  
  
▪︎ PyAutoGUI：用Python脚本控制鼠标键盘，通过坐标和截图定位界面。  
  
▪︎ agent-desktop：通过系统无障碍接口读取界面元素，适合支持该接口的桌面软件测试、表单填写和菜单操作。

---

pi 和 dsh 如何选？

> **meow (@meow2149)**
> 
> pi和dsh都支持通过插件深度定制agent。  
>   
> 具体怎么选，可以从底层机制、同类插件、生态规模和使用习惯这四个方面看。  
>   
> 一、底层机制  
>   
> ▪︎ Pi  
>   
> pi是一个精简、可扩展的agent底座，主要通过TypeScript扩展增加能力。
> 
> ![Image](<meow%20(@meow2149).assets/image-01.jpg>)