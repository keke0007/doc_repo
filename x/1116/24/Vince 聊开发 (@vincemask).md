---
author: "Vince 聊开发"
handle: "@vincemask"
source: "https://x.com/vincemask/status/2122307598787760587"
date: 2026-11-10T21:03:02.000Z
type: tweet
likes: 64
reposts: 5
replies: 3
bookmarks: 73
views: 5410
---
# Vince 聊开发 (@vincemask)

个人多仓库 Agent 工作流：Home Repo 方案。  
  
这个思路来自 Matt Pocock 分享的 Home Repo 模式：用一个独立仓库集中管理 Agent 指令、Skills、工具和项目知识，让 Agent 能够理解多个代码仓库并完成跨项目任务。  
  
一个统一入口，多个独立仓库，两种工作模式。  
  
1. 日常单仓库开发：直接进入业务仓库运行 Agent。  
  
2. 跨仓库需求：从 Home Repo 启动 Agent，统一分析和协调修改。  
  
3. 个人规范和通用 Skills：集中管理。  
  
4. 项目架构、构建命令和技术约束：仍然保留在业务仓库。  
  
5. Git、依赖和发布流程：各仓库独立维护。  
  
这样既能实现跨仓库协作，也能避免所有 Agent 都获得过大的修改范围。

![Image](<Vince%20聊开发%20(@vincemask).assets/image-01.png>)

> **Matt Pocock (@mattpocockuk)**
> 
> A lot of folks ask me "how do you work with agents in multiple repos at once?"  
>   
> I have a very simple solution - the Home Repo  
>   
> I have a single repo (the home) where all of my skills, steering files, wiki etc live.  
>   
> I run my agent in the home repo, and then inside AGENTS.md I have
> 
> ![Image](<Vince%20聊开发%20(@vincemask).assets/image-02.png>)