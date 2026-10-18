---
author: "yibie"
handle: "@yibie"
source: "https://x.com/yibie/status/2102573965470241100"
date: 2026-09-23T01:41:11.000Z
type: article
---
# 把判断层接进 harness：一次真实的两天迭代

*By yibie (@yibie)*

![Banner](<把判断层接进%20harness：一次真实的两天迭代.assets/image-01.jpg>)

作者：iamaamir（[@rimaa_nahk](https://x.com/@rimaa_nahk)，Pi 社区开发者）

Jev 那条线上我们写过很多：它是什么、怎么用、谁在替代它、怎么自己搭。但「把它接进 agent harness 当判断层」这件事，一直只有理论。

现在有实践了，而且这个实践的过程本身值得看——它被评论区改写过一次。

一、第一版：把 Jev 当「概率化的 switch 语句」

三天前，作者在 r/PiCodingAgent 发了一帖（134 赞 / 55 条评论），核心判断很锋利：

JEV 真正有意思的地方，出现在你不再把它当一个小 LLM 之后。

LLM → 生成任何东西

JEV → 在一组有界选项之间选择

 这让模型路由变得几乎显而易见：提示词 → JEV → 最好的模型。而不是用一个 LLM 去决定该调哪个 LLM，让 JEV 做那个有界判断，剩下的交给确定性代码。

也许 JEV 的杀手级用例不是 Doom。也许它是一个概率化的 switch 语句。

他发布了 PI-Bifrost，一条命令装：

pi install npm:pi-bifrost

评论区里出现了很多高质量的回复：

最高赞的回复（53 赞）很短：「直接用 von 或 Laya 就行了，那样你就不用按次付费，开源模型。」

（von 是 492 星的开源 System One 决策模型，宣称 15ms 以下、非自回归；Laya 是 121 星那个 MIT 许可的。两个都是真实存在的项目。）

然后是一条被严重低估的长评论（23 赞），列出了 Jev 在 harness 里可以用在哪——这是目前我见过最完整的一份清单：

一、Skills 钩子。 每条用户消息之后，判断该加载哪些新 skills。这会取代现在的 skill 机制——你可以有一个巨大的 skill 索引而不污染 LLM 上下文。

二、子 agent 与工作流路由。 根据当前任务选择该由哪个 workflow 步骤或哪个子 agent 来做。

三、通用工具函数。 一个接收「问题字符串 + 一组可能答案」的函数，JEV 返回选择、概率和置信度；一个发送整个对话历史的版本；一个只发用户消息历史的版本。

四、挑最好的助手回复。 一个钩子把用户提示和 AI 回复送给多个 LLM，让 JEV 挑最好的那个——带一份评分标准帮它判断。作者自己也注明了代价：这会翻倍 token 数。

五、给 LLM 回复打分。 钩子把最后一条用户消息和助手回复交给 JEV，问一个是/否：这是个好回复吗？UI 显示答案和置信分。

他还进一步说 「JEV 影响大到我认为 harness 需要一些重构」：

- 一个可选的 goal 提示，在新会话开始时给出。如果 JEV 知道你的总体目标，它能更聪明地做决策

- 重做 skills（如上）

- 重新思考「助手回复之后」的循环。 现在只有一种选项：提示用户。让 JEV 决定该做什么——继续、质量差重试、还是交给用户

- JEV.md。 类似 AGENTS.md，但专注决策而不是代码生成。它可以更大，放那些放进 AGENTS.md 太贵的东西，比如编码风格和架构

- 而且合并成一条 post-user-message 钩子做完全部，这样每条消息只往返 JEV 一次

三、作者怎么回应的：抽掉硬编码

两天后他发了第二帖（109 赞）：

几天前我在这里发过「用 Jev 当 Pi 内部的决策引擎」。评论把我指向了 Laya、von、Reflex 这些开放替代品。

一旦图景里有了多个 provider，把想法硬接在 Jev 上就说不通了。

所以我把它抽出来了。

于是有了 pi-system-one：

pi install npm:pi-system-one

export SYSTEM_ONE_BASE_URL=http://localhost:8008

给 Pi 加一个 system_one 工具，支持三种原语（choice / noul / score）。它建在一个provider 无关的 SDK 之上，所以任何兼容 System One 的 provider 都能坐在同一个接口后面。Pi-Bifrost 现在只是那个 SDK 的一个消费者。

而 TypeSafe 官方 API 只是其中一个配置：

SYSTEM_ONE_BASE_URL=[api.typesafe.ai](https://api.typesafe.ai)

SYSTEM_ONE_API_KEY=...

SYSTEM_ONE_MODEL=jev-latest

四、架构上值得记一笔

最终形态是三层：

system-one-core provider 无关的运行时（TypeScript，npm 上 v0.2.0）

├── pi-system-one 给 agent 直接暴露 system_one 工具（v1.1.2）

└── pi-bifrost 用同一个边界做路由决策（v0.4.3）

provider 边界下面可以是 Jev（TypeSafe 官方）、Reflex、任何兼容 /v1/systemone 的端点、或自定义的 SystemOneProvider。

这个分解本身是个示范：先做出能跑的东西，在评论指出「你绑死在一家」之后，把抽象抽出来而不是加适配层。

五、Bifrost 的路由设计值得单独看

它把「路由」拆成了四个独立决策，而且刻意限制了分类器的权力：

Tier（能力档） → Pool（该档允许哪些模型） → 策略（怎么从健康候选里挑一个） → 分类器（可选，只判断该用哪个档）

关键约束写在文档里：「分类器永远不获得忽略你配置的模型池或策略的权限。」

用 Jev 的时候：它收到当前提示词和你的档位判据，返回一个档位选择、概率和置信度。它永远收不到你的 provider 模型池，也永远不选择具体模型。（这部分是可选的，需要 TypeSafe 凭据。）

还有几个工程细节做得很实在：

- 失败记忆：记录模型失败，超过阈值就开路，把它从未来路由里排除，直到一次受控的恢复试验

- 不重放：「Bifrost 改变未来的路由，但不会静默重做可能已经编辑过文件、调用过工具或触碰过外部系统的工作。」 这个约束在 agent 场景里是必须的——很多朴素的路由重试会重复副作用

- 预览：/bifrost preview 在不开始生成、不激活模型的前提下解析出路由

- 缓存取舍写在明面上：逐轮切换模型会打碎 provider 的提示词缓存局部性，文档专门有一篇 A → B → A 与 N 次切换的指南，并建议「要连续性就 pin 住」

六、我的判断

这是 Jev 线里「接入 agent harness」的完整案例，而且它验证了一个我们之前只在理论上讨论的东西。

我们在写那份 Jev 工程笔记（TypeSafe 创始人 Diogo Almeida 的设计笔记）时，里面提到的用法是：用类型化决策做权限判断、做工具路由、做上下文块的可见性分级。而这个作者实际做出来的是：

- 工具选择（Bifrost 的 tier 分类器）

- 通用 system_one 工具（让 agent 自己就能问有界问题）

方向对上了，而且是从社区自下而上长出来的，不是从官方文档抄的。

另一个值得注意的点是迭代速度：第一帖到第二帖相隔两天，中间完成了「发现绑死 → 抽出 provider 抽象 → 发布 SDK → 把原工具降级为一个消费者」。评论区那条「用 von 或 Laya 就不用付费」起了作用。

当然要看清规模：这个 SDK 仓库 15 星、今天建的；npm 包版本号还在 0.x / 1.x 早期；作者是个独立开发者。这不是可以立刻上生产的东西。它的价值在于指出了形状——接口该长什么样、边界该划在哪，这里有一份可以抄的答案。

链接

第一帖（134 赞）：[www.reddit.com/r/PiCodingAgent/comments/1wlczow/i_think_i_found_the_best_use_case_for_jev_and_pi/](https://www.reddit.com/r/PiCodingAgent/comments/1wlczow/i_think_i_found_the_best_use_case_for_jev_and_pi/)

第二帖（109 赞）：[www.reddit.com/r/PiCodingAgent/comments/1wnl7da/pi_can_now_use_jev_and_more/](https://www.reddit.com/r/PiCodingAgent/comments/1wnl7da/pi_can_now_use_jev_and_more/)

pi-system-one：[github.com/iamaamir/system-one/tree/main/pi-system-one](https://github.com/iamaamir/system-one/tree/main/pi-system-one)

pi-bifrost：[iamaamir.github.io/pi-bifrost/](https://iamaamir.github.io/pi-bifrost/)

开源替代 von：[github.com/wfzyx/von](https://github.com/wfzyx/von)

开源替代 Reflex：[github.com/kshetrajna12/reflex](https://github.com/kshetrajna12/reflex)

本刊 Jev 工程笔记译文：[github.com/yibie/jev-engineering-zh](https://github.com/yibie/jev-engineering-zh)

[#Agent工程](https://x.com/search?q=%23Agent%E5%B7%A5%E7%A8%8B&src=hashtag_click) [#Pi](https://x.com/search?q=%23Pi&src=hashtag_click) [#类型化决策](https://x.com/search?q=%23%E7%B1%BB%E5%9E%8B%E5%8C%96%E5%86%B3%E7%AD%96&src=hashtag_click)