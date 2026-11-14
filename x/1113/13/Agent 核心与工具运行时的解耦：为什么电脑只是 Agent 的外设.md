---
author: "Todd Zheng"
handle: "@ShiZheng24517"
source: "https://x.com/ShiZheng24517/status/2121005874693239092"
date: 2026-11-12T22:23:01.000Z
type: article
likes: 15
bookmarks: 9
views: 707
---
# Agent 核心与工具运行时的解耦：为什么电脑只是 Agent 的外设

*By Todd Zheng (@ShiZheng24517)*

![Banner](<Agent%20核心与工具运行时的解耦：为什么电脑只是%20Agent%20的外设.assets/image-01.jpg>)

最近在看OpenAI的Agents API的protocol设计。抛开外面包着的那些概念不谈，我觉得这个protocol底下藏着一个很重要的趋势：Agent核心（Agent Core）和工具运行时（Tool Runtime，或者说多个Tool Runtimes）的解耦。这意味着我们可以有一个带持久记忆的agent，同时去操作N台不同的电脑，而不再需要让不同电脑上的agent之间去搞什么A2A通信。

## 一个被默认绑死的公式

2023年Lilian Weng写过一篇很出名的博客《LLM Powered Autonomous Agents》，里面提出了一个后来被无数论文和演讲反复引用的公式：Agent = LLM (Model) + Memory (Context) + Planning + Tool Use。后来OpenAI和Anthropic在各自的工程指南里也给出了类似的定义，大体上就是Model加上Context，再加上Control Loop和Tools。这个公式本身只定义了逻辑上的组件，从来没有规定说这些组件必须跑在同一台物理机（bare metal）上。

事实上，第一步解耦从第一天起就已经发生了：因为千亿参数的模型根本塞不进个人的电脑里，大家天然就接受了Model跑在云端的GPU集群上，通过HTTP API去调用。但是剩下的三样东西：Context、Control Loop和工具运行时，却因为工程上的图省事和历史惯性，被默认塞进了同一台机器的同一个进程里。

自从Claude Code和openclaw这类本地agent火了之后，这种绑定就更被当成理所当然了。为了能最方便地拿到本地的文件系统和bash，开发者顺手把agent的Control Loop（或者叫Harness）、会话上下文、甚至是长期记忆（比如躺在本地目录里的CLAUDE.md或者各种memory文件）全都放进了同一个本地进程。大家不知不觉中接受了一个隐含的假设：工具运行时在哪台电脑上，agent就必须住在哪台电脑上。

![Image](<Agent%20核心与工具运行时的解耦：为什么电脑只是%20Agent%20的外设.assets/image-02.jpg>)

## 跨机器A2A：为架构缺陷买单

只要你不止有一台电脑，这个假设就会立刻给你制造麻烦。比如我现在手头有一台随身带的MacBook，家里有一台台式机，云上还开着两台跑训练和编译的Linux开发机。按照现在的部署方式，我得在这四台机器上分别跑四个agent。它们各自守着自己那台机器的bash和文件系统，有着各自独立、且互相完全不通的记忆。我在MacBook上教会了agent我的代码风格和某个内部工具的用法，切到开发机上的时候，那边的agent对这一切一无所知，还得从头再教一遍。

你可能会说，我可以在MacBook上跑一个主agent，然后让它通过SSH或者远程MCP去连另外几台开发机，这不也能操作多台机器吗？问题在于，这样只是把你的MacBook变成了那个单点中心：一旦你把笔记本盖子合上走出办公室，整个Control Loop就跟着休眠挂死了；而且当你回到家坐在台式机前的时候，你白天积累的上下文和记忆还锁在公司那台合着盖的MacBook里。

更荒谬的事情发生在需要跨机器协作的时候。比如我想让agent看一眼我MacBook上改了一半的本地代码，然后去云端开发机上跑一下压测，最后把结果整理到家里的机器上。因为每个agent都和自己所在的物理机绑在一起，为了串起这个流程，现在业界开始流行让不同电脑上的agent之间通过A2A（Agent-to-Agent）协议互相发消息。在我看来，同一个人名下的多台电脑之间搞A2A，完全是在为架构缺陷买单。

任何系统里，信号在跨边界传递过程中的损耗都是不能忽视的：每一次跨边界的交接，本质上都是一个0到1的系数相乘，链条越长，最后漏掉的东西就越多。A2A就是强行在系统里插入了一次甚至多次有损的自然语言转译。电脑A上的agent明明已经把报错堆栈、上下文细节和推理过程（reasoning tokens）都装在脑子里了，只因为它调不到电脑B的bash，就必须把这些精确的中间状态压缩成一段自然语言，像传话筒一样发给电脑B上的agent；电脑B上的agent收到这段话，再去重新理解、重新推理、重新规划，执行完之后再把结果用自然语言传回来。这不仅多烧了成倍的token、增加了延迟，更麻烦的是中间的信息丢失：很多关键细节在两次自然语言转述之后就漏光了。

![Image](<Agent%20核心与工具运行时的解耦：为什么电脑只是%20Agent%20的外设.assets/image-03.jpg>)

真正的A2A应该只发生在跨信任边界或者跨人的时候，比如我的个人agent去和你的个人agent协商一个会议时间；或者是为了主动的上下文隔离，比如派一个临时的subagent去读几百兆的脏日志，读完只把结论带回来，避免污染主上下文。仅仅因为两块硬盘、两个bash不在同一台物理机上，就要跑两个独立的agent去发消息沟通，这显然是搞错了系统边界。

## OpenAI Agents API 的解法

回过头来看OpenAI新推出的Agents API（v1/agents）的protocol设计，你会发现它在协议层面把这个结解开了。

在Agents API的设计里，原本绑死在本地进程里的东西被明确拆成了两层原语：一层是托管在服务端的Agent核心（在API里体现为维护了持久化记忆、自动上下文压缩和后台控制循环的Session），另一层则是纯粹代表工具运行时的Environment和远程MCP。至于API表层暴露成Session这个概念本身合理不合理，我觉得对用户来说最好根本就是sessionless的，这个话题是个大坑，我们留到下一篇blog再聊。但至少在底层拓扑上，它把Agent核心和工具运行时彻底分开了。

在这个协议下，如果你想让云端的Agent核心操作你自己的电脑或私有服务器，协议提供了两条通道：一条是把Environment设置成self_hosted，在机器上跑一个轻量级的执行器（codex exec-server）来接管本地的PTY终端和文件读写；另一条是用它的Secure MCP Tunnel（tunnel-client），把不同机器上的本地MCP服务反向暴露给同一个Agent核心。无论走哪条通道，你在本地电脑上跑的都不再是一个完整的agent，而只是一个非常薄的executor。它主动向云端发起一条出站的WebSocket或HTTPS长连接（因此不需要公网IP），平时什么都不想，只负责听从云端Agent核心的指令：在本地跑一条shell命令、读写一下文件、或者转发本地工具调用，然后把stdout/stderr原样推回去。

这其实和我之前在《[Skill + CLI Considered Harmful](https://toddzheng.net/zh/blog/skill-cli-considered-harmful/)》里说的“薄客户端”是一个道理：每台电脑不再跑完整的agent，而是退化成了纯粹的工具运行时。

## 电脑只是外设：走向真正的 Personal Agent

一旦想明白这一点，整个架构就顺了：电脑其实只是agent的热插拔外设。就像我们用一台主机接三个显示器、两个键盘一样，一个持久化的Agent核心完全可以同时挂载N个不同的工具运行时。我的MacBook是一个工具运行时，我的云端GPU开发机是另一个工具运行时，我树莓派上的家庭服务器是第三个工具运行时。在同一个推理循环（Turn）里，这同一个agent可以先调MacBook上的工具读一把本地文件，紧接着在同一段思考过程里直接调Linux开发机上的bash把任务跑起来。中间没有任何A2A的传话损耗，所有的长期记忆、个人偏好和推理上下文都完整地留在那唯一一个Agent核心里。甚至当我把MacBook合上装进包里走出办公室的时候，这个agent也不会因为笔记本休眠而停掉，它只是感知到MacBook这个外设断开了，手头在云端开发机上的活儿照样可以继续往下跑。

![Image](<Agent%20核心与工具运行时的解耦：为什么电脑只是%20Agent%20的外设.assets/image-04.jpg>)

仔细想想，这其实跟人自己工作的方式是一模一样的。人身上是没有长着tools的，我们都是走到有tools的地方去操作tools：在办公室用工位上的台式机，在路上掏出笔记本，回到家切到家里的电脑。工具留在原地，但做思考、带着记忆的始终是同一个我。我们从来不会因为在办公室和家里各放了一台电脑，就把自己劈成两个人，下班回家还要先给办公室的自己打个电话交接一下白天干了什么。

通过这样的架构，我们才能有真正的Personal Agent：一个完全如影随形的个人助手。

## 写在最后

过去两年，我们造agent的思路一直是“Compute-Attached”的：先有一台电脑，再往这台电脑里塞一个agent，让它替我操作这台电脑。但随着OpenAI Agents API这类把控制循环和执行环境解耦的protocol成熟，接下来的趋势一定会走向“Identity-Attached”：Agent核心跟着人走，而我们手头的N台电脑、容器、手机，都只是按需接入这个核心的工具运行时。

当然，把所有设备的工具运行时都挂在同一个Agent核心下面，也会带来新的工程考题：比如某台边缘机器读了脏数据被Prompt Injection之后，如何做跨机器的权限隔离来控制安全爆炸半径；再比如飞机上断网时的离线降级。但方向是清楚的：别再给你每台电脑上都装一个独立的agent再让它们互相发微信了，你的电脑需要的只是一个薄薄的runtime daemon。

*原文首发于个人博客：*[*https://toddzheng.net/zh/blog/agents-api-and-decouple/*](https://toddzheng.net/zh/blog/agents-api-and-decouple/)