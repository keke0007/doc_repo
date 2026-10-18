---
author: "yibie"
handle: "@yibie"
source: "https://x.com/yibie/status/2105816871132119255"
date: 2026-10-02T00:27:20.000Z
type: article
likes: 82
reposts: 12
replies: 3
bookmarks: 166
views: 15250
---
# Pi 1.0

*By yibie (@yibie)*

![Banner](<Pi%201.0.assets/image-01.jpg>)

《Pi 1.0》

---

Pi 1.0

出处：Earendil（2026-10-01）

---

今天的 Pi 1.0 是一个「加固过的、极简的、可扩展的」agent harness。全世界有几十万人在每周使用 Pi。

而这次发布里最值得讲的，是它对 MCP 的态度掉了个头。

如果你以前去过 [pi.dev](https://pi.dev)，你会看到一句骄傲的声明：Pi 不支持 MCP。如果你听过我们聊 Pi 的播客，你会听到不止一句对 MCP 的轻蔑表态。结果升级之后你会发现，MCP 现在是受支持的核心功能。

到底发生了什么

「世界不是静止的。我们过去一年一直在关注 MCP，今天的 MCP 不是去年的 MCP。」——但光靠这个理由，不足以把 MCP 放进核心。Pi 有很好的扩展生态，MCP 完全可以当一个扩展。

真正的原因是：他们发现在把 MCP 并进核心的过程中，需要做的那些改动本身是有普遍价值的。 原文的说法是：

最终 Pi 需要的和 MCP 需要的非常相似：一个能玩的沙箱，一个解释器（a sandbox to play with in the form of an interpreter）。

举个例子：为了 MCP 做的那些改动，同时也让 Jev 在 Pi 里更容易用。

CodeMode 到底是什么

这是整篇发布里最需要解释清楚的概念。原文先给了它一句话的定义：

CodeMode 是一个小小的沙箱，用来把工具调用组合起来（a neat little sandbox to allow composing of tool calls）。

然后是更重要的那段机制解释。一个 harness 执行工具时，基本有两侧：可以在 bash 跑的那侧执行，也可以在 harness 的 agent loop 跑的那侧执行。这两侧的信任级别差别很大——harness loop 通常跑在受信任的环境里，而它调用的工具往往跑在一个并不那么受信任的沙箱里。

CodeMode 的特殊之处是：它跑在 harness 运行的那一侧。

所以它最准确的理解方式是：它是一套用来编排和协调工具调用的机制。它是一个沙箱，让 agent 用更灵活的顺序去发出那些工具调用，并且允许它用 JavaScript 把它们组合起来。

因为 CodeMode 也跑在 harness 这一侧，它的状态是作为会话记录的一部分保存的，而不是存在文件系统里。

为什么是 JavaScript？原文给了个实际理由：

理论上任何语言都行，但 JavaScript 相当有吸引力——因为小体积的 JavaScript 可以被作为 WASM 二进制文件分发，从而提供合理的保护级别。

在 Pi 里，配置了 MCP 时 CodeMode 会自动加载，也可以作为默认工具加进配置。原文甚至说：直接让 Pi 自己重新配置来启用 CodeMode 就行。

而它不只是给 MCP 用的。 这是我认为最能说明它价值的一个例子——当你用着一个提供 Jev 的供应商时，可以直接这么写：

Use typesafe/jev via codemode to find the 20 most frustrated

commenters on our issue tracker

Pi 会自己组合 Linear 的 MCP 和 Jev 来完成这个分析。它实际生成的 CodeMode 脚本大致是这样（节选）：

const { issues } = await tools.mcp__linear__list_issues({

team: "Pi", state: "open", limit: 250,

});

const jev = await models.getModelOfType(

"classifier", "cloudflare-workers-ai", "typesafe/jev",

);

const questions = {

frustration: {

type: "choice",

instructions: "Judge ONLY the emotional tone of the people. " +

"Ignore how severe the bug is.",

criteria: {

none: "Neutral, factual, or friendly, even about a serious bug",

mild: "Explicit annoyance, impatience, or disappointment",

high: "Clearly angry, exasperated, sarcastic, or fed up",

},

},

};

const results = [];

let next = 0;

async function worker() {

while (next < issues.length) {

const issue = issues[next++];

const { comments } = await tools.mcp__linear__list_comments({

issueId: issue.identifier,

});

const c = await models.classify(jev, {

state: { ...issue, comments }, questions,

});

results.push({ id: issue.identifier, title: issue.title,

...c.answers.frustration });

}

}

请注意这段脚本里同时出现了 tools.mcp__linear__*（工具调用）和 models.classify(jev, ...)（一个非 LLM 的 Jev 分类模型）。 这就是原文那句「原生支持 MCP，以及 Jev、图像模型这类非 LLM 模型」的实际含义——CodeMode 的沙箱里，工具和模型是同一等公民，可以用 JavaScript 自由编排。

MCP 现在的问题（他们的判断）

原文没回避：MCP 最大的问题依然是难以组合（hard to compose）。但它把责任指向了别处：

在这个节点上，那与其说是 MCP 的问题，不如说是外面那些 MCP 服务器、以及各家 harness 与之协作方式的问题。

很多 MCP 服务器仍然是为那种「把工具直接倒进上下文」的 harness 造的，并在自己那侧通过返回文本来优化 token 效率。

他们希望 MCP 变成什么样？

我们认为 MCP 现在应该更接近 OpenAPI 加上智能的工具发现。也就是说，工具应该返回结构化数据，而且应该能通过它们的文档和描述被找到。

另一个动机是诚实说出的：「我们相信积极影响一件事的最好方式是拥抱它。」 他们认为现代 MCP 比过去好得多，但服务器和模式仍有改进空间，所以他们想进入这场对话，帮助塑造出在小 harness 里也能跑得好的做法，而不是站在场边看。

其余的主要变化

上面这些是 CodeMode 的故事，1.0 还带了这些：

· 虚拟模型的扩展支持（Extension support for virtual models）

· 延迟工具加载（Deferred tool loading）

· Anthropic 模型的缓存预热（Cache warming for anthropic models）

· 对话中途的系统消息（Mid-conversation system messages，也就是「感知对话记录（transcript-aware）的 prompt 与工具变更」）

· 新的 TUI 主题

· 默认全屏模式

原文对这些的态度值得注意：

这些功能我们想了好几个月。它们是被扔到墙上然后粘住的。从墙上掉下来的东西，那张清单长得多。

当我们用带着全部这些新功能的 Pi 时，感觉像是迈进了一大步，但它仍然感觉很简洁，像 Pi。

关于「延迟工具加载」为什么可能被需要

原文在解释为什么不能只做 CodeMode 而不做 MCP 时，交代了一条关键的设计约束，我认为这是理解这次发布的技术核心：

我们近几个月做了很多工作，让 Pi 能在支持延迟工具加载、对话中途系统消息和推理等级变更的新模型上有效运作。但我们还没有升级自己的工具装配（tool loadout），让它更好地适配这些新能力。

在 CodeMode 的世界里，你必须决定一个工具是对 LLM 可见，还是只对 LLM 的 CodeMode 部分可见。而一个普通的 MCP 扩展，从 Pi 的工具装配里拿不到足够多的元数据来把这个体验做好。 所以我们需要确保工具可以被配置成「仅延迟」或者「CodeMode 专属」。

另一个交付物：Pi Durable

原文用了相当篇幅交代一个方向性判断：

除了持续打磨 Pi 之外，我们也开始意识到 Pi 有些方面，并不契合很多人想使用它的方式。 在 Earendil，我们想把 Pi 的极简主义、以及它让你在编码 agent 之外、终端之外驾驭 AI 的方式带出去。它需要能从不同的界面触达，并支持更长时间的对话和任务。简而言之，Pi 必须变得更耐久（more durable）。

我们没有背离自己的极简根基、把 Pi 弄成它不是的东西，而是把这项工作合并成一个新的实验性包：Pi Durable。 它是构建长时间运行的 agentic 应用的新基座，让这些应用的构建者和使用者能以极大的灵活性驾驭和引导底层智能。

Pi Durable 与 Pi 共享两个核心原则——极简主义和超级可塑性——但把这两条原则延伸到了新的维度。

我读到的三件事

一、这次发布的真正内容是「一次重新思考」，不是「一次功能添加」。

MCP 进核心这件事，如果只看结果会觉得很怪：一个公开说过「不支持 MCP」的项目突然支持了。但原文给出的理由是：为了支持 MCP 所必须做的那些改动（沙箱、解释器、工具元数据、延迟加载），恰好也是 Pi 支持 Jev 这类非 LLM 模型所必须的。

换句话说，是「为了 A 做的重构让 B 也变得可能」，而不是「我们改主意了」。这也是为什么 CodeMode 同时出现在了 MCP 的故事和 Jev 的故事里。

二、CodeMode 的定位是「编排层」，不是「执行层」。

这点原文明说得很清楚，也是我认为最容易被误解的地方。它跑在 harness 侧（受信任侧），而不是工具侧（沙箱侧）。它不负责执行工具，它负责决定这些工具调用以什么顺序、如何组合地发生——并且因为它在 harness 侧，它的状态进的是会话记录，不是文件系统。

用 JavaScript 而不是别的语言，理由也很实在：小体积 JS 可以作为 WASM 分发，从而拿到一个合理的隔离级别。

三、他们对「极简」的理解是「等它证明自己」。

原文那句话我认为是整篇的价值主张：

agentic 工具链每周都在变，而很多变化活不下来。Pi 不是那样运作的。我们等到某样东西证明了它自己，然后才考虑采纳——把它真正的功能和它固有的、增加出来的复杂度放在一起称。

这一条解释了这次发布里一个看起来矛盾的地方：一边说「MCP 很难组合、外面很多 MCP 服务器是错的」，一边把 MCP 并进核心。 因为「证明了自己」的标准，在 Pi 那里不是「MCP 已经变好了」，而是「为了 MCP 要做的改动本身值得做」。

边界

· 这是厂商发布文，所有判断都是 Earendil 自己的；上面我尽量把「他们说的」和「我读到的」分开标注了

· CodeMode 的完整定义来自同周的姐妹篇《"You Said No MCP!"》（2026-09-29），不是 1.0 发布文本身——发布文里只有一句话带过，这也是为什么这一篇需要两篇合起来读

· 原文提到的 Pi Durable 有独立的技术文（作者 Mario），我没有读，本文只交代了它的定位

· Pi 是本账号日常使用的 harness，这构成了一个立场问题，你自己判断

链接

Pi 1.0 发布文：[earendil.com/posts/pi-1-0/](https://earendil.com/posts/pi-1-0/)

CodeMode 的定义出处：[earendil.com/posts/you-said-no-mcp/](https://earendil.com/posts/you-said-no-mcp/)

文档：[pi.dev](https://pi.dev)

代码（MIT）：[github.com/earendil-works/pi](https://github.com/earendil-works/pi)

[#Agent](https://x.com/search?q=%23Agent&src=hashtag_click) Harness [#MCP](https://x.com/search?q=%23MCP&src=hashtag_click) [#CodeMode](https://x.com/search?q=%23CodeMode&src=hashtag_click)