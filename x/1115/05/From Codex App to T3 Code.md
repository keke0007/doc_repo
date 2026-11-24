---
author: "Xuanwo"
handle: "@xuanwo"
source: "https://x.com/xuanwo/status/2120758765800870128"
date: 2026-11-12T06:01:06.000Z
type: article
likes: 208
reposts: 19
replies: 36
bookmarks: 287
views: 40911
---
# From Codex App to T3 Code

*By Xuanwo (@xuanwo)*

![Banner](<From%20Codex%20App%20to%20T3%20Code.assets/image-01.jpg>)

最近我的工作流已经完全从 Codex App 迁移到 [@theo](https://x.com/@theo) 的 T3 Code，在这里简单分享一下我迁走的原因，以及对 T3 Code 的初步感受。

在我构建好我自己的 LOOP 之后，下一个课题就变成如何稳定的运行，这时候本地的 Macbook 和 Mac Mini 就成了瓶颈，我把目光转移向了在远端的服务器上运行。

我的第一考虑是沿用 Codex App，毕竟它本身就有远程的支持（虽然特别慢）。但是我在完全 setup 好之后发现，Codex App 的 thread 协调有致命的问题：Codex App 的 thread 管理能力居然是 App 本身提供的一个 MCP， 在 Remote 服务器上只提供查询和读取的能力，根本就不能创建！

如果我想要跟之前一样，让 agent 自主创建多个 thread 并编排，那我需要这样做：

- 分别查桌面端和 app-server 的项目 ID （是的，他们不一样）
- 通过 app-server RPC 的 initialize → thread/start 创建持久化线程，明确设置 full access 和 never 审批（是的，我在 codex 里面设置了没用，不继承配置）
- 通过桌面端创建 managed worktree，再把工作区附件、cwd 和运行时根目录绑定给新线程（是的，这就是 OpenAI）

为了让这一套魔鬼的东西可以正常运行，我被迫维护了一个 codex thread 技能。这一切的原因，都是因为 OpenAI 在强推他们的 Codex Cloud，他们压根就不打算让你好好的使用自己的远程服务器。

在忍受 Codex App 一段时间之后，我终于考虑换别的方案。我的第一考虑是 [@lody_ai](https://x.com/@lody_ai) ，lody 的设计感很好，实现思路也对，但是我实在接受不了所有的数据要别人的服务器上存储（公司也不允许），哪怕是有了端到端加密。我向团队提出了付费自部署的方案，希望他们能真的提供这样的选择。

我也短暂的使用过 Paseo 之类的方案，但是着实不太喜欢里面的很多设计，最后我选择了 T3 Code。

T3 Code 的很多设计思路跟 Lody 是非常像的，但是区别在于它不要求也不提供一个中心的 Server，你可以自己部署服务器和 Relay，所有的数据都不需要存在第三方的服务器上。

![Image](<From%20Codex%20App%20to%20T3%20Code.assets/image-01.jpg>)

我目前的架构大致上就是这样，相比于 Codex，T3 的跨节点的 Agent 协同做得特别好，可以非常自然的创建多个跨节点的 Thread / Subagents。

除了 Remote 能力之外，T3 的交互设计也有一点小惊喜，模拟了类似邮箱的 Inbox 结构：Inbox / Working / Settled。刚开始会有点不太适应，但是久了之后，感觉这种设计很大程度上降低大量并行任务带来的注意力损耗，我只需要关注 Inbox 里面的任务就行。

总的来说，T3 的初体验很不错，推荐大家也试试～