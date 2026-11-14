---
author: "Yuepan Chao"
handle: "@smallnest"
source: "https://x.com/smallnest/status/2119315477851955546"
date: 2026-11-08T06:25:59.000Z
type: article
likes: 2
replies: 2
bookmarks: 2
views: 773
---
# deepseek superagent 实战-X

*By Yuepan Chao (@smallnest)*

![Banner](<deepseek%20superagent%20实战-X.assets/image-01.jpg>)

最近看到几个有意思的软件工程实践。一个是 xAI 的 Lauren（[@poteto](https://x.com/@poteto)）开源的一套专为 AI 编程助手（如 Cursor、Claude Code 等）打造的工程技能与工作流体系。更让我触动的是 Magpie 的作者 yetone 公开的 Magpie 项目自动化维护工作流，核心逻辑可以总结为：**Claude Code + 官方 Opus 5.5 模型 + 后台三线程 /loop 循环 + 错题本机制 + 人工晚间微调**。人们赞叹 yetone 把这套工作流运用得炉火纯青，打造出 Magpie 这个优秀的产品。

现在 DeepSeek Harness（简称 DSH）推出了 Super Agent 的一套 AI 管理工作流方法，相当于把 yetone 的这套打法形成了一套工程化的、通用的流程，更适合在实践中使用。接下来我们看看这套方法是怎么用的。

## 介绍

DeepSeek 把 DSH 定位为开源的 agent harness。官方介绍是：基于 Cordis 的「一切皆插件」架构构建，既可以当桌面应用，也可以启动成网页。这个预览版现在面向全球开放测试，同时开源。

DSH 自己带五类基础能力：日常办公（整理资料、分析表格、写文档、做汇报）、写代码（修问题、开发功能、跑测试）、查资料并标注来源、后台跑耗时任务、安装或自制插件。

2026 年 10 月 31 日，DeepSeek Harness 团队发布并开源了 Super-agent（超级智能体）的架构和研究原型（代码仓库 dsh-superagent），官方公众号上也有配套文章：[Super-agent 设计探索：迈向更自主的 Agent 系统](https://mp.weixin.qq.com/s/zoS7GlReWPhfCjENGxXCdg)。

什么是 Super-agent？

- **核心定位**：基于 DeepSeek Harness 底座搭建的主动式编程与任务助手，可常驻在代码仓库、IM 软件（如飞书群）等生产环境中。
- **7×24 小时在线**：上下文是连续的，能自己调度多个 Agent 并行干活，人不用一步步盯着、反复下指令。
- **「让 AI 管 AI」**：借用了计算机系统设计那套思路，由 Super-agent 当调度中枢，把开发或运营任务拆开，派给底下若干个专门 Agent 去做。

它的界面把工作方式讲得很清楚。页签有「概览、来源、触发、评估、执行、设置」几个，核心对象是「工单」：

- **来源**：agent 从哪些渠道收单。本文配置了两个渠道，GitHub issue 和飞书聊天。
- **触发与评估**：新工单到达后先经过筛选和评估，决定是接单还是忽略。概览页的「过去 24 小时」按「到达 → 被过滤 → 已评估 → 接单或续做」四个数字统计。
- **执行**：由执行器领取工单并驱动一个独立的 agent 会话完成工作，概览页显示「执行中 / 队列 / 等待新触发」的占用情况。

所以它和普通会话的差别不在模型，在入口：你不用守在终端前打字，团队平时用的 issue 和聊天群就可以直接喂给它，由它自己筛选、认领、执行、回话。下文简称 super agent。

## 和 Claude Code Agent Teams 的区别

其实年初的时候 Claude 推出了 Agent Teams（又叫 Agent Swarm）的概念，就是将 Agent 以团队的形式加一个 Leader 的形式完成一项任务。那么 DSH 的 super agent 和它有什么区别呢？

名字像，层级不一样。

Claude Code 的 Agent Teams 发生在一个会话内部：主会话当 lead，派生几个独立的 Claude Code 实例（teammates），共享任务列表、互相发消息，一起把同一个任务做快。

Super-agent 在会话之外。它常驻在 GitHub 和 IM 上收单，按需再拉起若干执行会话去干活。DeepSeek 博客自己也提到：Router 层和常见的 Agent Team 结构相似，但 Agent Team 侧重单个复杂任务内部的拆解，Router 层只管任务之间的分派。

对比项Claude Code Agent TeamsDeepSeek Super-agent状态实验性，需设 CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 开启原型，官方博客声明不是正式产品使用形态终端里的交互式会话常驻在 IM、GitHub 等渠道，全双工接收消息任务来源人输入的提示词环境事件（群聊、issue、PR 评论、CI 结果）组织结构lead + teammates，共享任务列表，成员之间直接通信Router + Worker，默认 Worker 之间不直接通信上下文每个 teammate 独立上下文，lead 不保留其历史Router 维护全局连续上下文，Worker 上下文隔离主动性人发起任务，Agent 执行主动观察、过滤噪音、判断是否接单，可以提前动手（投机执行）跨渠道没有同一任务可以在 IM 和 GitHub 之间接力持久性限于本机会话，会话结束即结束；不支持恢复 teammate计算与状态分离，可在云端持久在线（设计目标）一句话：Agent Teams 管的是「一个任务怎么并行做完」，Super-agent 管的是「一个团队怎么让 AI 自己接住源源不断的活」。而且 Super-agent 的 Worker 内部还可以再塞一个 Agent Team，两者不冲突。

## 实战

DSH 团队开源了 super agent 的原型：[dsh-superagent-prototype](https://github.com/deepseek-ai/dsh-superagent-prototype)。当然项目描述中也说了，它是一个研究代码：

**本原型仅供参考，并非 DeepSeek 官方产品。** 此代码源自我们团队的日常开发工作，其中大部分工程工作由编码代理完成。请将其视为研究代码，而非可用于生产环境的软件。代码可能仍存在缺陷，且编码风格和工程实践可能不一致。在使用前，请务必进行审查和调整。

但是不管怎样，我先安装上感受一下效果。

安装

手工安装是一件麻烦的事。但是在这个时代，我们只需扔一句话给 AI 就好了。首先将此项目 clone 到本地，然后在你的 Agent 输入下面一句话就好了，按照提示一步步地配置：

> Read SETUP.md in this repository and set up dsh-superagent with me.

有几个点需要你抉择和配置，以下是我的选择：

- 同时支持 GitHub 和飞书获取 ticket。
- GitHub 使用了一个单独的账号给 super agent，它的提交和 comment 都是使用这个账号，避免和你的主账号混在一起。当然这样的话你需要把这个账号加入到你的项目的协作者中。
- 提交 PR 后不自动 merge，需要手工 merge。这样我有机会人工再 review 一次代码。
- 飞书配置也很简单，扫码配置即可。

安装完毕后，在 Agent 中说「打开网站」，就打开了 http://127.0.0.1:3082/?token=，可以 web 端访问 super agent，它基本上是 DSH 的一个扩展。

![Image](<deepseek%20superagent%20实战-X.assets/image-02.jpg>)

第一个 issue：GitHub issue 方式

hyla 是我基于 kero 改造的一个终端工具，我添加了很多的功能，只好独立维护了。正好我想给它生成一个网站，这就作为我测试 DSH super agent 的第一个 issue：

![Image](<deepseek%20superagent%20实战-X.assets/image-03.jpg>)

我提交 issue 后，@ 我创建的那个和 super agent 关联的 GitHub 账号就 OK 了。super agent 会监控并获取自己的任务，创建工单开始执行：

![Image](<deepseek%20superagent%20实战-X.assets/image-04.jpg>)

接下来你就不需要管了，剩下的就是 super agent 的活了，它的 router 和 worker 后台开动起来了：

![Image](<deepseek%20superagent%20实战-X.assets/image-05.jpg>)

最终它完成了所有的工作。因为我设置的是不自动 merge，所以它提交 PR 就停止了，等待我手工 merge。

![Image](<deepseek%20superagent%20实战-X.assets/image-06.jpg>)

@@@IMG_6@@@

![Image](<deepseek%20superagent%20实战-X.assets/image-07.jpg>)

事实上我这里可以配置一个 code review，自动 code review、自动修复代码，然后自动 merge，这样就更自动化了：

![Image](<deepseek%20superagent%20实战-X.assets/image-08.jpg>)

![Image](<deepseek%20superagent%20实战-X.assets/image-09.jpg>)

飞书

上面的操作是通过 GitHub issue 作为源创建工单的。我们还可以通过飞书触发。

在配置的飞书群中输入你的工单需求：

![Image](<deepseek%20superagent%20实战-X.assets/image-10.jpg>)

super agent 就接收到了你的工单，同样地去工作了，也很方便，7×24 小时工作：

@@@IMG_10@@@

![Image](<deepseek%20superagent%20实战-X.assets/image-11.jpg>)

你可以看到任务开始执行了：

@@@IMG_11@@@

最终它提交了 PR：

@@@IMG_12@@@

整个流程都可以在飞书中查看，流程在相应的话题中：

@@@IMG_13@@@

## 总结

用下来，Super-agent 让我觉得有用的地方不在模型，在于把「什么时候该干活」从人手里交给了系统。

- **入口变了**：不用打开终端写任务描述，去 GitHub issue 或飞书群里 @ 它一句就行。接单、执行、提 PR，结果还回到原来的话题里。
- **人的位置也变了**：从盯着每一步，退到提需求和验收 PR 两头。我特意关掉自动 merge，就是想留下验收这一关。
- **边界很清楚**：一个仓库、一个专用账号、不自动 merge，风险可控。想更激进的话，也可以接上自动 review、自动修复、自动 merge。

和 yetone 的 Magpie 工作流比，思路是一样的，形态不同。Magpie 靠 /loop 把 Claude Code 循环跑起来，调度逻辑写在提示词和习惯里，是个人的实践总结；Super-agent 把调度、过滤、分发做成了系统组件（Router、Worker、Trigger、Watchdog），多任务并发、跨渠道、常驻在线这些事不用人再去维护。

它现在还是原型：只支持少数渠道，长程记忆和安全隐私，官方博客也列成了开放问题。我打算先拿它当个「数字同事」跑个人项目，稳了再考虑让它接手团队仓库的日常杂活。最近看到几个有意思的软件工程实践。一个是 xAI 的 Lauren（[@poteto](https://x.com/@poteto)）开源的一套专为 AI 编程助手（如 Cursor、Claude Code 等）打造的工程技能与工作流体系。更让我触动的是 Magpie 的作者 yetone 公开的 Magpie 项目自动化维护工作流，核心逻辑可以总结为：**Claude Code + 官方 Opus 5.5 模型 + 后台三线程 /loop 循环 + 错题本机制 + 人工晚间微调**。人们赞叹 yetone 把这套工作流运用得炉火纯青，打造出 Magpie 这个优秀的产品。

![Image](<deepseek%20superagent%20实战-X.assets/image-12.jpg>)

![Image](<deepseek%20superagent%20实战-X.assets/image-13.jpg>)

现在 DeepSeek Harness（简称 DSH）推出了 Super Agent 的一套 AI 管理工作流方法，相当于把 yetone 的这套打法形成了一套工程化的、通用的流程，更适合在实践中使用。接下来我们看看这套方法是怎么用的。

## 介绍

DeepSeek 把 DSH 定位为开源的 agent harness。官方介绍是：基于 Cordis 的「一切皆插件」架构构建，既可以当桌面应用，也可以启动成网页。这个预览版现在面向全球开放测试，同时开源。

DSH 自己带五类基础能力：日常办公（整理资料、分析表格、写文档、做汇报）、写代码（修问题、开发功能、跑测试）、查资料并标注来源、后台跑耗时任务、安装或自制插件。

2026 年 10 月 31 日，DeepSeek Harness 团队发布并开源了 Super-agent（超级智能体）的架构和研究原型（代码仓库 dsh-superagent），官方公众号上也有配套文章：[Super-agent 设计探索：迈向更自主的 Agent 系统](https://mp.weixin.qq.com/s/zoS7GlReWPhfCjENGxXCdg)。

什么是 Super-agent？

- **核心定位**：基于 DeepSeek Harness 底座搭建的主动式编程与任务助手，可常驻在代码仓库、IM 软件（如飞书群）等生产环境中。
- **7×24 小时在线**：上下文是连续的，能自己调度多个 Agent 并行干活，人不用一步步盯着、反复下指令。
- **「让 AI 管 AI」**：借用了计算机系统设计那套思路，由 Super-agent 当调度中枢，把开发或运营任务拆开，派给底下若干个专门 Agent 去做。

它的界面把工作方式讲得很清楚。页签有「概览、来源、触发、评估、执行、设置」几个，核心对象是「工单」：

- **来源**：agent 从哪些渠道收单。本文配置了两个渠道，GitHub issue 和飞书聊天。
- **触发与评估**：新工单到达后先经过筛选和评估，决定是接单还是忽略。概览页的「过去 24 小时」按「到达 → 被过滤 → 已评估 → 接单或续做」四个数字统计。
- **执行**：由执行器领取工单并驱动一个独立的 agent 会话完成工作，概览页显示「执行中 / 队列 / 等待新触发」的占用情况。

所以它和普通会话的差别不在模型，在入口：你不用守在终端前打字，团队平时用的 issue 和聊天群就可以直接喂给它，由它自己筛选、认领、执行、回话。下文简称 super agent。

## 和 Claude Code Agent Teams 的区别

其实年初的时候 Claude 推出了 Agent Teams（又叫 Agent Swarm）的概念，就是将 Agent 以团队的形式加一个 Leader 的形式完成一项任务。那么 DSH 的 super agent 和它有什么区别呢？

名字像，层级不一样。

Claude Code 的 Agent Teams 发生在一个会话内部：主会话当 lead，派生几个独立的 Claude Code 实例（teammates），共享任务列表、互相发消息，一起把同一个任务做快。

Super-agent 在会话之外。它常驻在 GitHub 和 IM 上收单，按需再拉起若干执行会话去干活。DeepSeek 博客自己也提到：Router 层和常见的 Agent Team 结构相似，但 Agent Team 侧重单个复杂任务内部的拆解，Router 层只管任务之间的分派。

对比项Claude Code Agent TeamsDeepSeek Super-agent状态实验性，需设 CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 开启原型，官方博客声明不是正式产品使用形态终端里的交互式会话常驻在 IM、GitHub 等渠道，全双工接收消息任务来源人输入的提示词环境事件（群聊、issue、PR 评论、CI 结果）组织结构lead + teammates，共享任务列表，成员之间直接通信Router + Worker，默认 Worker 之间不直接通信上下文每个 teammate 独立上下文，lead 不保留其历史Router 维护全局连续上下文，Worker 上下文隔离主动性人发起任务，Agent 执行主动观察、过滤噪音、判断是否接单，可以提前动手（投机执行）跨渠道没有同一任务可以在 IM 和 GitHub 之间接力持久性限于本机会话，会话结束即结束；不支持恢复 teammate计算与状态分离，可在云端持久在线（设计目标）一句话：Agent Teams 管的是「一个任务怎么并行做完」，Super-agent 管的是「一个团队怎么让 AI 自己接住源源不断的活」。而且 Super-agent 的 Worker 内部还可以再塞一个 Agent Team，两者不冲突。

## 实战

DSH 团队开源了 super agent 的原型：[dsh-superagent-prototype](https://github.com/deepseek-ai/dsh-superagent-prototype)。当然项目描述中也说了，它是一个研究代码：

**本原型仅供参考，并非 DeepSeek 官方产品。** 此代码源自我们团队的日常开发工作，其中大部分工程工作由编码代理完成。请将其视为研究代码，而非可用于生产环境的软件。代码可能仍存在缺陷，且编码风格和工程实践可能不一致。在使用前，请务必进行审查和调整。

但是不管怎样，我先安装上感受一下效果。

安装

手工安装是一件麻烦的事。但是在这个时代，我们只需扔一句话给 AI 就好了。首先将此项目 clone 到本地，然后在你的 Agent 输入下面一句话就好了，按照提示一步步地配置：

> Read SETUP.md in this repository and set up dsh-superagent with me.

有几个点需要你抉择和配置，以下是我的选择：

- 同时支持 GitHub 和飞书获取 ticket。
- GitHub 使用了一个单独的账号给 super agent，它的提交和 comment 都是使用这个账号，避免和你的主账号混在一起。当然这样的话你需要把这个账号加入到你的项目的协作者中。
- 提交 PR 后不自动 merge，需要手工 merge。这样我有机会人工再 review 一次代码。
- 飞书配置也很简单，扫码配置即可。

安装完毕后，在 Agent 中说「打开网站」，就打开了 http://127.0.0.1:3082/?token=，可以 web 端访问 super agent，它基本上是 DSH 的一个扩展。

![Image](<deepseek%20superagent%20实战-X.assets/image-14.jpg>)

第一个 issue：GitHub issue 方式

hyla 是我基于 kero 改造的一个终端工具，我添加了很多的功能，只好独立维护了。正好我想给它生成一个网站，这就作为我测试 DSH super agent 的第一个 issue：

![Image](<deepseek%20superagent%20实战-X.assets/image-15.jpg>)

我提交 issue 后，@ 我创建的那个和 super agent 关联的 GitHub 账号就 OK 了。super agent 会监控并获取自己的任务，创建工单开始执行：

![Image](<deepseek%20superagent%20实战-X.assets/image-16.jpg>)

接下来你就不需要管了，剩下的就是 super agent 的活了，它的 router 和 worker 后台开动起来了：

![Image](<deepseek%20superagent%20实战-X.assets/image-17.jpg>)

最终它完成了所有的工作。因为我设置的是不自动 merge，所以它提交 PR 就停止了，等待我手工 merge。

![Image](<deepseek%20superagent%20实战-X.assets/image-18.jpg>)

![Image](<deepseek%20superagent%20实战-X.assets/image-19.jpg>)

事实上我这里可以配置一个 code review，自动 code review、自动修复代码，然后自动 merge，这样就更自动化了：

![Image](<deepseek%20superagent%20实战-X.assets/image-20.jpg>)

![Image](<deepseek%20superagent%20实战-X.assets/image-21.jpg>)

飞书

上面的操作是通过 GitHub issue 作为源创建工单的。我们还可以通过飞书触发。

在配置的飞书群中输入你的工单需求：

![Image](<deepseek%20superagent%20实战-X.assets/image-22.jpg>)

super agent 就接收到了你的工单，同样地去工作了，也很方便，7×24 小时工作：

![Image](<deepseek%20superagent%20实战-X.assets/image-23.jpg>)

你可以看到任务开始执行了：

![Image](<deepseek%20superagent%20实战-X.assets/image-24.jpg>)

最终它提交了 PR：

![Image](<deepseek%20superagent%20实战-X.assets/image-25.jpg>)

整个流程都可以在飞书中查看，流程在相应的话题中：

![Image](<deepseek%20superagent%20实战-X.assets/image-26.jpg>)

## 总结

用下来，Super-agent 让我觉得有用的地方不在模型，在于把「什么时候该干活」从人手里交给了系统。

- **入口变了**：不用打开终端写任务描述，去 GitHub issue 或飞书群里 @ 它一句就行。接单、执行、提 PR，结果还回到原来的话题里。
- **人的位置也变了**：从盯着每一步，退到提需求和验收 PR 两头。我特意关掉自动 merge，就是想留下验收这一关。
- **边界很清楚**：一个仓库、一个专用账号、不自动 merge，风险可控。想更激进的话，也可以接上自动 review、自动修复、自动 merge。

和 yetone 的 Magpie 工作流比，思路是一样的，形态不同。Magpie 靠 /loop 把 Claude Code 循环跑起来，调度逻辑写在提示词和习惯里，是个人的实践总结；Super-agent 把调度、过滤、分发做成了系统组件（Router、Worker、Trigger、Watchdog），多任务并发、跨渠道、常驻在线这些事不用人再去维护。

它现在还是原型：只支持少数渠道，长程记忆和安全隐私，官方博客也列成了开放问题。我打算先拿它当个「数字同事」跑个人项目，稳了再考虑让它接手团队仓库的日常杂活。