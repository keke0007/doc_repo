---
author: "Chasen"
handle: "@chasen_liao"
source: "https://x.com/chasen_liao/status/2112174241093226846"
date: 2026-10-19T13:29:16.000Z
type: article
likes: 99
reposts: 116
replies: 90
bookmarks: 53
views: 26117
---
# 我把 EverOS 装进了 Pi，找了很久的记忆方案终于用上了

*By Chasen (@chasen_liao)*

![Banner](<我把%20EverOS%20装进了%20Pi，找了很久的记忆方案终于用上了.assets/image-01.jpg>)

MCP 接上了，工具也能调，结果模型压根不主动去查记忆。。。还得我在对话里提醒一句"去查一下 EverOS"

所以我干脆给 Pi 写了个插件，叫 pi-everos

起因是最近看到 EverOS Cloud，它能把 Agent 的记忆放到云端，通过 MCP 或 API 接进现有工作流。我就想把它接进 Pi，把开发时做过的决定留下来，以后需要再查

pi-everos 主要做了 3 件事👇

1️⃣ 任务开始前先提醒模型  
用 Pi 的 ***before_agent_start*** ***hook***，在模型开始处理任务前注入一小段提示词，告诉它有哪些记忆工具、项目资料和个人偏好分别存在哪、什么时候该查、什么时候该存。同时把 MCP 工具设成 direct，让模型能直接看到工具说明

没改 Pi 源码，接入和提示都放在扩展里

2️⃣ 项目和全局分开存  
插件注册了两条连接：everos 放当前项目的资料，everos-global 放跨项目的偏好

像"这个项目的 key 只能从环境变量读"，就该留在项目侧；真正通用的写作习惯，才适合放全局

3️⃣ /everos doctor 自检  
检查连接、作用域和提示词有没有开，确认当前到底接到了哪份记忆

![Image](<我把%20EverOS%20装进了%20Pi，找了很久的记忆方案终于用上了.assets/image-02.jpg>)

## 写这个插件的过程，刚好也成了我的实测

开发时陆续定了 5 个决定：  
1️⃣ 默认用 uvx everos-mcp 加 API key 接入  
2️⃣ key 不写进设置文件  
3️⃣ 不自动修改已有的 mcp.json  
4️⃣ plan 模式下拦截记忆写入  
5️⃣ 忘记会话前要先确认

这些后来都能从 EverOS 里查回来

![Image](<我把%20EverOS%20装进了%20Pi，找了很久的记忆方案终于用上了.assets/image-03.jpg>)

我让它"先把项目和全局两边的记忆都列出来"，它分别调了两边的工具，项目侧列出了 3 条 Episode

里面有最早的 v0.1.0 决策，有准备发 npm / GitHub 时对 README 的要求，还记下了 npm 发布遇到的 401 / 404，以及后来 pi-everos@0.1.1 发布完成的结果

选过什么方案、哪些配置不能改、发布卡在哪，这些具体的事都留下来了，要用的时候能回查

后台的 Memory Activity 里也能看到这些记录，能看到项目决策和发布过程的摘要。截图是当时后台显示的总数：Profiles 5 条，Episodes 7 条。前面那 3 条是只查项目侧的结果，所以数字对不上是正常的

![Image](<我把%20EverOS%20装进了%20Pi，找了很久的记忆方案终于用上了.assets/image-04.jpg>)

## 说说我为什么选 EverOS

最吸引我的就是记忆放在云上，存储、抽取和检索都在服务端跑，我本地只维护插件和连接，不用自己再搭一套向量库和索引服务

而且它存的不是聊天记录，是整理过的记忆，我要的正是"做过哪些决定、当时为什么这么定"。它还支持按项目分作用域，跟我项目 / 全局分开的用法刚好对上。以后换客户端或设备，身份和作用域对齐后也能接同一份记忆

按官方文档，它的技术路线大概是这样👇

1️⃣ 接入：MCP 或 API / SDK 都行，Cloud 和自托管的开源版用的是同一套 API  
2️⃣ 抽取：服务端把对话切成一段段事件，整理成不同类型的记忆。Episode 存事件叙述，里面带拆出来的原子事实；Profile 存持续的属性和偏好；Agent 跑任务的过程记成 Case，反复出现的经验再提炼成 Skill  
3️⃣ 整合：后台自己跑，重复的会合并，新旧矛盾的会处理，画像也跟着更新  
4️⃣ 检索：支持 keyword、向量、混合和 agentic 几种查法，记忆按作用域隔开，查询不会跨作用域

目前最直接的体验，就是查这些项目记录返回得挺干脆，拿到后能接着聊

## 目前的局限也说清楚：

这次是我明确要求查记忆，模型选了对应工具；查到的都在项目侧，全局还没存内容

global 只是约定的一个作用域名字，查询时还是要分别访问

默认接法仍然需要 uv、MCP 进程和 API key

写入、抽取和索引之间有延迟，刚存进去的不一定立刻能搜到

没有单独计时，也没做本地方案的同条件对比，换设备还没验证

Case → Skill、长历史和复杂任务的表现，我还要继续测

对我这种一直改同一个项目的人来说，最实在的就是：前面定下来的决定，后面的任务能从 EverOS 里查回来接着用

pi-everos 已经放到 GitHub 和 npm 了，安装用的 Agent 提示词放在第一条评论，相关链接在第二条

想试的话，开通 Pro，享半年 3 倍月度额度

传送门 👉 [everos.evermind.ai](https://everos.evermind.ai)

[#EverMind](https://x.com/search?q=%23EverMind&src=hashtag_click) [#EverOS](https://x.com/search?q=%23EverOS&src=hashtag_click)