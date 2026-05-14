# 1.手机与电脑投屏软件

https://github.com/viarotel-org/escrcpy

2.用100% 开源的一体化 ERP 系统，把企业的账务、库存、生产、资产、项目管理全管起来，不用各个模块分别买软件

https://github.com/frappe/erpnext

# 3.本周 GitHub 涨星最快的开源项目

```
1.mattpocock/skills (+30.9K stars） 
真正的工程师技能库。直接摘自我的 .claude 目录

2.forrestchang/andrej-karpathy-skills(+23.1K stars)
仅用一个 CLAUDE.md 文件就能优化 Claude Code 的表现，灵感来源于 Andrej Karpathy 对大模型编程常见陷阱的深入观察

3.Alishahryar1/free-claude-code (+14.7K stars)
让你在终端、VSCode 插件或类似 OpenClaw 的 Discord 机器人中免费使用 claude-code（支持语音输入）

4.Z4nzu/hackingtool (+8.7K stars)
专为黑客打造的 All-in-One 一站式黑客工具包

5.TauricResearch/TradingAgents (+6.0K stars）TradingAgents：基于大语言模型多智能体（Multi-Agents）的金融交易框架

6.huggingface/ml-intern (+5.7K stars)
ml-intern：一个开源的 AI 机器学习工程师，能全自动帮你读论文、训练模型并完成部署

7.abhigyanpatwari/GitNexus (+5.2K stars)
GitNexus：零服务器的代码智能引擎。它是一个完全在你的浏览器中运行的纯客户端知识图谱生成器。只需拖入一个 GitHub 仓库链接或 ZIP 压缩包，就能生成带有内置 Graph RAG 智能体的交互式知识图谱。简直是代码探索的神器

8.lsdefine/GenericAgent (+2.4K stars)
 自进化智能体：从仅 3300 行代码的种子起步，自行生长出技能树，用原来六分之一的 token 消耗就能实现对系统的完整控制

9.mksglu/context-mode (+2.3K stars)
专为 AI 编程助手设计的上下文窗口优化工具。通过对工具输出进行沙盒化处理，直接将上下文占用锐减 98%。目前已支持 14 个平台

10.AIDC-AI/Pixelle-Video (+2.1K stars)
AI 全自动短视频引擎

本周趋势总结： claude.md 配置文件和智能体技能树正在成为 GitHub 上新的技术护城河
```

# 4.做数据分析或者训练模型数据集

```
做数据分析或者训练模型，卡在哪里？不是算法，不是框架，是找数据这一步就能把你搞死。

今天刷 GitHub 发现一个宝藏项目：Awesome Public Datasets，74k+ Star，不是那种没人维护的烂仓库，是真的在持续更新的。

覆盖领域直接列给你看：

🌾 农业
🧬 生物
💰 经济金融
🌍 气候环境
📚 教育
🏥 医疗健康

……几十个方向全有，基本你能想到的场景都覆盖到了。

最省心的地方是啥？

1️⃣ 每个数据集都标了可用状态，不用点进去才发现链接已经挂了

2️⃣ 附了元数据链接，来源清楚，引用起来不心虚

3️⃣ 标明数据格式：CSV、JSON、数据库……你用啥都能找到对应的

4️⃣ 体量和更新频率也都写好了，一眼看完直接决策

不用再一个个去 Kaggle 翻，不用靠运气碰数据，这一份清单基本能解决你80%的找数据需求。

建议直接收藏，下次要用的时候直接翻，省出来的时间去做真正有价值的事。

🔗 https://github.com/awesomedata/awesome-public-datasets
🔗 https://awesomedataworld.slack.com
```

# 5.Linux管理工具

```
https://github.com/matthart1983/syswatch
```

# 6.如何为AI造一个项目大脑

```
如何为AI造一个项目大脑？

第一件事，在你最重要的项目根目录建一个CLAUDE.md。

下次AI犯错的时候，先不要手动修，而是问自己一句：CLAUDE.md里缺了什么？

第二件事，把每天重复做的事改造成skill。

如果你每天做某件事超过一次，把它变成skill或command。
Code review、生成commit message、写发布说明、修一类重复的bug，这些都该是skill，不该是每天手敲提示词。

第三件事，在容易踩坑的地方加一个hook。

Hook是98.4%里最有杠杆的那部分。它不依赖AI变聪明，它依赖确定性代码做强制检查。这是把人类工程师的判断力翻译成机器可读约束的过程。

未来五年，工程师的能力曲线正在从「我能写多少行代码」转向「我能为AI设计多严格的工作环境」。
```

# 7.给 Claude Code 装了个记忆层

```
给 Claude Code 装了个记忆层，token 省了 71.5 倍。
graphify 做的事很简单：在你问 AI 任何问题之前，先把你的整个项目消化成一张知识图谱。代码走 AST 本地提取（25 种语言），文档/论文/截图/视频走 Claude 并行提取，全部合并、聚类，导出交互式 HTML + JSON + 自然语言报告。
之后每次对话，AI 先读图，再回答。不用每次都把文件翻一遍。

为什么 26 天 4 万星？
因为它解决了一个所有人的共同痛点：AI 编程助手的上下文是"一次性"的。每次新对话，它忘光光。graphify 把这个变成了"持久化记忆"。

一个斜杠命令 /graphify .，把 AI 助手变成了有记忆的搭档。
```

# 8.OpenAI开源Symphony：给每一个任务配一个永不下班的 AI员工

```
OpenAI 最近开源了一个叫 Symphony 的项目。
https://github.com/openai/symphony
感觉是给AI Agent用的任务管理系统，OpenAI 内部与Linear整合，大大提升了人管理Agent的能力，目前已经有1.8w Star。

好像跟一个X友做的产品很像？让AI翻译介绍下：
从一个激进的实验说起
六个月前，OpenAI 内部一个团队做了个当时看起来很激进的决定：仓库里不允许有任何人类写的代码。
每一行，都必须由 Codex 生成。
Codex 是 OpenAI 的 AI 编程助手，可以理解需求、读懂代码库、自主完成编程任务。
他们重新设计了整个工程流程，大量投入自动化测试和防护机制，把 Codex 当成真正的团队成员。
他们把这套方法叫做"harness engineering"（脚手架工程），并专门写了一篇博客记录这段历程。
结果确实跑通了。
但随即撞上了下一个瓶颈：上下文切换。
真正的瓶颈是人的注意力
每个工程师同时开几个 Codex 会话，分配任务，审查输出，调整方向，循环往复。
实际操作下来，大多数人同时管理三到五个会话还算舒适，超过这个数字，效率就开始下降。
忘了哪个会话在做什么，在几个终端之间来回跳，调试卡在一半的长任务……
AI 跑得很快，但系统的瓶颈是人的注意力。
他们意识到，自己其实是雇了一批极其能干的初级工程师，然后让人类工程师去微观管理他们。
这显然没法规模化。
换一个视角
问题出在思路上。
他们一直在优化"编程会话"和"合并 PR"，但这些只是手段。
PR（Pull Request）：工程师完成一段代码后，向主代码库提交合并请求，等待审查和合入。
软件开发真正围绕的是可交付物：issues（问题单）、任务、里程碑。
所以他们问了自己一个问题：如果不直接监督 AI，而是让 AI 自己从任务追踪系统里拉取工作，会怎样？
这个想法变成了 Symphony。
Symphony 是什么
一句话：把项目管理看板变成 AI 编码代理的控制中枢。
他们用的是 Linear，一款工程团队常用的任务管理工具。
每一个打开的任务，都会自动分配一个 AI 代理。
代理持续运行，直到任务完成。人类只需要审查结果。
具体来说，每个 Linear issue 对应一个独立的Agent工作空间。
Symphony 持续监视任务看板，确保每个活跃任务都有Agent在跑。
Agent崩溃了，自动重启；有新任务进来，自动接手。
整个工作流用 Linear 的状态来驱动，像一台状态机：
Todo（待办）→ In Progress（进行中）→ Human Review（人工审查）→ Done（完成）
AI 代理在这些状态之间流转，人类在"Human Review"节点介入。
几个让人印象深刻的细节
任务粒度可以很大
不再局限于"改一个函数"这种小粒度。
可以让代理先分析整个代码库、Slack 记录或 Notion 文档，产出实现方案，再自动拆解成一棵任务树，按依赖关系并行执行。
他们用了一个词叫 DAG（有向无环图，Directed Acyclic Graph），本质就是一张"哪些任务依赖哪些任务"的执行顺序图，确保代理不会乱序执行。
比如他们做过一个真实案例：先完成从 Webpack 到 Vite 的迁移，再升级 React。
Agent自己识别了这个依赖关系，等 Vite 迁移完成后才开始升级 React，完全符合预期。
Agent会自己创建任务
在实现过程中，Agent如果发现了性能问题、重构机会或者更好的架构方案，会直接在 Linear 里开新 ticket，供人类评估和排期。
很多后续任务也会被代理接手执行。
从手机上也能工作
因为编排器跑在开发服务器（devbox）上，从不睡觉，有个工程师在信号很差的小屋里，用手机 Linear App 提了三个重要改动，Agent照样接手执行了。
数据很直接
部分团队在前三周，合并的 PR 数量增长了 500%。
Linear 创始人 Karri Saarinen 也公开提到，Symphony 发布后，Linear 上新建工作区的数量出现了明显峰值。
它的核心是一个 Markdown 文件
这是 Symphony 最有意思的设计决策之一。
打开 Symphony 的代码仓库，会发现它本质上就是一个 SPEC.md，一份对问题和解决方案的定义文档，而不是一个复杂的监控系统。
他们定义好问题，给出高层次的指引，然后把这份规范扔给 Codex，让 Codex 来实现它。
参考实现选了 Elixir，一门相对小众的编程语言，但在并发（同时处理大量任务）和进程监督方面有非常好的原语（基础构建块）。
选它的理由也很直接：当代码成本趋近于零，终于可以为了语言的优势本身来选语言，而不是为了招人方便。
Codex 一次性就把 Elixir 实现写出来了。
为了打磨规范本身，他们又让 Codex 用 TypeScript、Go、Rust、Java、Python 各实现了一遍，用这些实现来发现规范里的歧义和可以简化的地方。
每种语言都成功了。
工作流也被文档化了
这里有个值得单独说的转变。
以前，工程师们有一套隐性的工作流程：接到任务，切出分支，把任务标记为进行中，提 PR，移到 Review 状态，附上演示视频……这些步骤人人都懂，但从来没有被正式写下来。
现在，这套流程被写进了 WORKFLOW.md，Symphony 确保 AI 代理遵循它。
以前是人类遵循隐性规范，现在是把规范显式化，让 AI 来遵循。
这个文件还有一个重要特性：热重载。
修改 WORKFLOW.md 后，Symphony 会自动检测变化，无需重启，直接把新配置应用到后续任务上。
如果以后想让代理在完成工作后附上自我反思，只需要在 WORKFLOW.md 里加一行，Symphony 就会引导Agent执行这一步。
Symphony 的技术架构（不想看可以跳过）
Symphony 的内部由几个核心组件构成，理解它们有助于明白整个系统为什么可靠：
Orchestrator（编排器）：整个系统的大脑，唯一有权修改调度状态的组件。
它负责轮询任务、决定哪些任务该启动、重试或停止，并追踪所有正在运行的代理状态。
Workspace Manager（工作空间管理器）：每个任务都有自己独立的文件目录，Agent 只能在自己的目录里操作，不会互相干扰。这是一个重要的安全边界。
Agent Runner（执行器）：负责启动 Codex 进程，把任务提示词传给它，然后把执行结果反馈给编排器。
Issue Tracker Client（任务追踪客户端）：负责和 Linear 通信，拉取任务列表，同步状态变化。
整个系统的并发控制也很细致，可以设置全局最大并发代理数（默认 10 个），也可以针对特定状态的任务单独限制并发数。
重试机制用的是指数退避（exponential backoff）：第一次失败等 10 秒，第二次等 20 秒，第三次等 40 秒，以此类推，最长不超过 5 分钟。
正常完成后的续跑检查只等 1 秒。
一个重要的架构选择：App Server 模式
Symphony 使用了 Codex 的 App Server 模式，一种内置的无头（headless）运行模式。
无头（headless）：没有图形界面，完全通过程序接口控制，适合自动化场景。
这种模式通过 JSON-RPC（一种轻量级的远程调用协议，用 JSON 格式传递指令和结果）以编程方式控制 Codex，比如启动一个对话线程、触发一个执行轮次、读取执行结果。
比通过 CLI 命令行或 tmux 会话操控 Codex 方便和可扩展得多。
另一个安全细节：为了避免把 Linear 的访问令牌（API token，相当于访问密码）直接暴露给Sub Agent，他们用动态工具调用（dynamic tool calls）的方式，封装了一个叫 linear_graphql 的函数。
代理可以通过这个函数对 Linear 执行任意查询，但永远接触不到原始 token。
遇到的新问题
当然，这种工作方式也有代价，他们没有回避这一点。
从实时干预Agent，变成在任务层面分配工作，意味着失去了随时纠偏的能力。
有时候Agent会完全跑偏，产出的东西完全不对路。
但他们的应对方式很有意思：不是手动修补结果，而是补充防护机制和技能，让Agent下次能自己成功。
这倒逼他们持续完善系统，加入了端到端测试、通过 Chrome DevTools 驱动浏览器、管理 QA 冒烟测试等新能力，还大幅改善了文档质量。
还有一个认知上的转变：不能把Agent当成状态机里的僵硬节点。
早期版本只让 Codex 实现任务，这太局限了。
Codex 完全有能力同时管理多个 PR、读取 CI（持续集成，自动化测试和构建流程）日志、处理代码审查反馈。
CI（Continuous Integration，持续集成）：每次代码提交后自动运行测试，确保新代码不破坏已有功能。
所以他们最终的方向是：给Agent目标，而不是给它严格的状态转换规则。
就像一个好的管理者，给直接下属分配目标，而不是每一步都手把手指导。
给它工具，给它上下文，让它自己想办法。
不是所有任务都适合 Symphony 的工作方式。
涉及模糊问题或需要强判断力的工作，工程师还是会直接用交互式 Codex 会话。
实际上，这些往往也是工程师最感兴趣、最享受的任务。
用 Symphony 来构建 Symphony
这个细节值得单独说一下。
Symphony 基本功能跑通之后，他们就开始用 Symphony 来开发 Symphony 本身。
当他们在内部演示这个系统，看到它自主管理任务、并附上功能演示视频作为工作证明时，反应非常热烈。Symphony 的内部项目频道迅速增长，各个团队开始自发使用它。
在 OpenAI，内部产品市场契合度（PMF）是对外发布的前提条件。
基于内部的使用情况，他们决定把 Symphony 分享给外部世界。
OpenAI 不打算把它做成产品
这个项目开源后，三周内获得了超过 15,000 个 GitHub Star。
社区已经有人做了各种移植版本：
有人用 Go 语言加上 Charm CLI 的终端 UI 做了一个版本
有人把它改造成支持 Anthropic 的 Claude Code，并支持 GitHub Issues，还做成了 Homebrew 可以直接安装
有人用 Claude Code 重新实现了整套规范，取名 hatice
但 OpenAI 明确说了：不打算把 Symphony 作为独立产品来维护。
它是一个参考实现，一个演示 Codex App Server 能力的例子。
核心思路很简单：
对每一个打开的任务，保证有一个Agent在它自己的工作空间里持续运行。
他们希望大家把自己喜欢的编码代理指向这份规范，构建适合自己环境的版本。
门槛其实出奇地低，直接把规范扔给 Codex，让它帮你实现一个就行。
值得思考的地方
Symphony 解决的问题，表面上是"怎么让更多 AI 并行工作"，但更深层的变化是：当代码的边际成本趋近于零，整个软件开发的经济学都变了。
每次改动的感知成本下降，意味着大家开始愿意做以前觉得"不值得"的事：试一个想法，探索一次重构，验证一个假设，不满意就扔掉。
参与工作的人也变了。
产品经理和设计师可以直接向 Symphony 提需求，不需要懂代码，不需要管理 AI 会话，描述功能，然后收到一个包含视频演示的审查包。
在大型 monorepo（单一代码仓库，把所有项目代码放在一个仓库里管理）里，Symphony 还承担了"最后一公里"的工作：监视 CI 状态，需要时自动 rebase（同步最新代码），解决冲突，重试不稳定的检查项，把改动一路护送进主分支，不需要人类盯着。
随着模型越来越强，能解决的问题越来越大，其他公司的瓶颈也会从"写代码"转向"管理 AI 工作"。
Symphony 提供的，是一种思路：不要管理Agent，管理任务就够了。
官方原文：https://openai.com/index/open-source-codex-orchestration-symphony/
```

# 9.借助ChatGPT英语学习

```
英语学习党狂喜！
这个开源项目每周自动更新英语杂志免费下载合集 

awesome-english-ebooks 直接拉满：
收录《经济学人》（含音频）、《纽约客》、《卫报》、《连线》、《大西洋月刊》等主流英语杂志  
每期都提供 epub + mobi + pdf 三种格式，Kindle、手机、平板、电脑全适配  
每周自动更新，最新一期点开就能下，下载即用

再也不用花钱订阅或者到处找资源了！
想练阅读、听力、提升英文思维，这仓库简直是神器。
GitHub直达：
 https://github.com/hehonghui/awesome-english-ebooks
快去收藏！
---------------------------
每天一个超级实用网站！！
第7期！LetMeEnglish！
官网直达： https://letmeenglish.com/?utm_source
想当web3大使，实习生，入住币安吗？
进入这些最主要的条件是会说流利的英语，这个网站是一个可以系统性学习英语的优质网站
-英文语法（从基础到进阶）
-常用词汇 + 易混词解析
-口语句型（母语者常用表达）
-日常英语场景
-而且是完全免费 + 不用注册
-每个知识点讲的通俗易懂，没有长篇大论后面还加了练习题，可以巩固知识点
有想学英语的可以收藏一下 避免丢失
（仅个人分享，没有任何广告！没有邀请码！）
关注我！每天获取超级雕炸天的实用网站！
```

# 10.MCP 从入门到精通

```
Hugging Face 官方上线了免费的《MCP 从入门到精通》课程（MCP Course），一套课把 MCP 的原理、开发到上线部署全打通，学完还能拿到官方认证证书。

课程地址：http://huggingface.co/learn/mcp-course

课程覆盖三大核心模块：理论基础｜实战开发｜部署应用
主要大纲如下：

- 入门指南：快速熟悉所需工具与平台，开学即上手
- MCP 基础 / 架构 / 核心概念：讲透核心概念、架构与组件，并用简单用例带你跑通思路
- 端到端实战：从 0 构建一个完整的 MCP 应用
- 部署实战：做出可上线的 MCP 应用，学习生产环境部署流程

学习节奏也很友好：每章 1 周，每周约 3–4 小时。

适合已有一定 AI 基础与编程能力的同学。完成后，你将能熟练使用 MCP SDK 与框架进行开发，并具备落地与部署能力。

```

# 11.PyTorch 免费入门教程

```
分享一份 GitHub 上口碑不错、讲得清晰又不绕的 PyTorch 免费入门教程：PyTorch Fundamentals。

从张量创建起步，覆盖矩阵运算、索引与重塑等高频操作，把 PyTorch 最核心的概念一口气梳理到位。

GitHub：http://github.com/analyticalrohit/pytorch_fundamentals

你将学到：

- 张量的核心概念与多种初始化方式
- 常用数学运算与比较操作的实战用法
- 矩阵乘法与批处理（batch）操作技巧
- 索引、切片、reshape 等形状操作指南
- NumPy 与 Tensor 的互转方法与注意点
- 广播机制及一系列实用小技巧

附带完整的 Jupyter notebook 和配套博客讲解，非常适合深度学习初学者打基础、快速上手。

```

# 12.入门 Codex 最好的视频

```
这绝对是入门 Codex 最好的视频，OpenAI 官方都认可！

中文字幕版和视频时间戳放在评论区

这是课程目录概览：
第一部分：Codex 基础
1. 安装与界面熟悉
2. 高效 Prompt、内置搜索功能的使用方法
3. 项目管理：创建项目、电子表格、文件存储与引用、搜索、文件夹组织
4. Skill 和插件
5. 创建自己的自定义 Skill（通过 API 实现）
6. 使用已创建的技能示例
7. 如何创建自动化工作流，让 AI 自动执行任务

第二部分：使用 Codex 多任务处理
1. 同时推进 6 个项目：
iOS App 设计和开发、落地页、投资者演示文稿、启动视频、社交媒体自动化发帖

2. 具体技能学习：
- 移动端设计 Skill 使用与 iOS App 完整搭建
- 生成 App 图标、在真机运行 App
- 用 Remotion 制作专业启动视频（时间线编辑、网格线、加音乐）
- Fork 聊天分支并行制作 Investor Deck
- 搭建收集用户信息的落地页 + Tally 表单
- Vercel 插件一键部署 Web App
- Typefully 自动化发 X（Twitter）帖
3. 高级操作技巧：
- 聊天窗口组织、重命名、多窗口并行
- Steering vs Queueing 代理
- 在 Codex 终端里调用 Claude Code
- 项目记忆管理、上下文切换
```

# 13.机器学习

```
https://github.com/harvard-edge/cs249r_book
---------------
写的非常详细的入门教程
RL 很恶心的点就在于你什么都要会：pretrain、inference、theory、infra
现在我也有了可以推荐给别人入门的读物，比我当时东看一点西看一点好太多了🥺
很多现代 LLM 的教学其实跳过了相当一部分的基础知识，比如 CNN、Q-Learning 这一类经典内容。我感觉我也是填鸭式地在学习这些内容，需要耐下心来慢慢地去看那些最旧最老最传统的东西
很感谢苏剑林的 blog 带我入门，我也想能写出一些带别人入门的东西，只不过即使是用 LLM 辅助写 notes 也相当折磨人，然后知乎上还没什么人看
当然还是有很多人喜欢看我写的垃圾 notes 的，还被行业杰青大佬订阅了，这种事跟 paper 被人关注了一样开心
Website: http://walkinglabs.github.io/hands-on-modern-rl

```

# 14.Agentic Design Patterns知识

```
https://adp.xindoo.xyz/
https://github.com/xindoo/agentic-design-patterns
```

# 15 Skill知识

```
https://github.com/TencentCloud/CubeSandbox
https://github.com/addyosmani/agent-skills
https://github.com/luzhenhua/NCE-Flow
https://github.com/alchaincyf/huashu-design
https://github.com/midudev/autoskills

https://github.com/win4r/skill-creator
https://github.com/anthropics/skills/tree/main/skills/skill-creator
https://github.com/win4r/skill-creator

我现在越来越觉得，打造 AI 管家这件事，核心就两样：知识 Markdown 化，能力 Skill 化。

所谓知识 Markdown 化，就是把你脑子里的东西、你看过的东西、你学到的东西，全部整理成 Markdown 文档，沉淀在知识库里。所谓能力 Skill 化，就是把你反复要做的事情，封装成一个个技能，让智能体来执行。

这两件事加在一起，智能体就能通过读文档来获取你的知识，通过跑技能来帮你处理事务。它不再是一个每次都从零开始的工具，而是真正了解你、能替你干活的管家。

我自己现在工作目录下面已经有 900 多个 Markdown 文件了，涵盖了我几乎所有的知识积累，而且还在快速增长。技能方面也有 23 个，也在持续增加。

说真的，做到这个程度以后，感觉 AI 跟以前完全不一样了。它越来越像是一个真正懂我的助手，而不只是一个聪明的搜索引擎。

这就是我现在最专注在做的一件事：一点一点地，把自己的知识和能力，迁移到这个系统里去。

-----------------
Skill其实就是分类学。
最近也不知道为啥，看到大家对skill的热情高涨到了一种有点离谱的程度。
感觉万物都可以蒸馏，万物都可以封装成skill。
我看了好几个朋友的skill库，有的装了六七十个，最离谱的甚至都破百了。

昨天正好发了Harness的文章后，提到了我觉得skill就是分类学这个事，那一段出乎意料的被很多朋友转发。

所以我就再展开说一下。

一个好的skill，我觉得他的核心就两个词：
分类和触发。

一个多月前Claude还更新过一次他们的Skills生成器，我当时还专门写过一篇，新版本最重要的动作，就是怎么用反馈去不断优化一个skill的触发条件。

skill怎么触发、能不能正确触发、触发以后能干什么，才是最核心的事。

之前有一篇论文发过实验数据，就是当Skills数量在20个以下时，准确率保持在90%以上，几乎不会错。超过30个准确率就不行了。到了200个的时候，准确率就剩20%了，而且速度极慢，Token消耗还爆炸。

跟我自己体感差不多，我自己的的Skills常年保持在30个以下。

我举个自己的例子，我之前想把NanoBanana的API封装成能被Agnet调用的skill，因为我平时有很多生图的需求，比如公众号封面图、小红书封面图、PPT配图等等。

那这些应该每一个需求单独做一个skill，还是应该合成一个skill呢？

我的做法就是只有一个图片生成的Skill，这个skill内部写了我的几个主流场景，在Agent触发这个skill后，根据我的上下文进行二次分析，再调用内部具体的分支画图场景，同时也能用这个skill，覆盖我其他的通用生图需求。

这其实就是分类学的核心理念。
从来不是只分的越细越好，是找到最合适的颗粒度。

界、门、纲、目、科、属、种，生物学就是如此，一层一层穿透下去。
你要是把前面全抹了，只给你留个种，你可以想象一下这个世界有多灾难。

封面图和PPT配图之间的差异，不值得在最顶层各自占一个独立的skill，它们只是图片生成这个类别内部的变异。
但图片生成和服务器管理之间的差异，那是真的大到需要各自占一个独立的skill。

我自己判断一个skill值不值得存在，标准就三条：
1. 它对应的场景有没有明确的边界。
2. 它对应的场景是不是会高频复现。
3. 它能不能归属进已有的skill里。

以及，奥卡姆剃刀原则，如无必要，勿增实体。
翻译过来就是，你用不到的skill你就别装。

但最重要的，还是需要你设计自己的分类系统，哪些是CLAUDE.md能处理的，哪些是skill该处理的。

分类学如此。
你的skill，也应如此。
-----------------
Top Claude Code GitHub Repositories to Boost Your Next Project's Efficiency 10x:
1. Supabase CLI  
https://github.com/supabase/cli  
2. Skill Creator  
https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md  
3. Get Sh*t Done  
https://github.com/gsd-build/get-shit-done  
4. NotebookLM (Python)  
https://github.com/teng-lin/notebooklm-py  
5. Obsidian  
https://github.com/obsidianmd  
6. Continue  
https://github.com/continuedev/continue  
7. Open Interpreter  
https://github.com/OpenInterpreter/open-interpreter  
8. AutoGen  
https://github.com/microsoft/autogen  
9. LangChain  
https://github.com/langchain-ai/langchain  
10. Flowise  
https://github.com/FlowiseAI/Flowise  
11. Boltdotnew (clone)  
https://github.com/stackblitz/bolt.new  
12. Awesome Claude Code  
https://github.com/hesreallyhim/awesome-claude-code  
13. Prompt Engineering Guide  
https://github.com/dair-ai/Prompt-Engineering-Guide  
14. Everything Claude Code
https://github.com/zhuyansen/skill-blue-book
--------------------------------------
今天给 skills 更新了一下。借鉴了 
@garrytan
 在 gstack 里面的Orchestrator模式，会从 vc，客户，运营，和杠精的角度来讨论方案。
同时增加了自学习 /money-learn，周复盘 /money-retro，流程 skill 化 /money-skillify 等技能，迭代了分析技能 /money-diagnose
现在完整的技能库一共增加到了 25 个。
---------------------------------
解决真正工程问题的 Skills：Skills For Real Engineers

作者 
@mattpocockuk
 公开了自己 .claude/ 目录中每天在用的 Agent Skills 集合，目标读者是在做真正工程的人们，解决真正的工程问题。

# 真正想解决的四类失败模式

1. Agent 没做对你想要的事 —— 沟通鸿沟
引用 The Pragmatic Programmer："没人确切知道自己想要什么。" 修复：在动工前先被 Agent 反向拷问。
· /grill-me：通用版逼问
· /grill-with-docs：工程版逼问，同时维护项目术语表与 ADR
这是作者明说的"最受欢迎的两个 Skill"。

2. Agent 太啰嗦 —— 缺少共享语言
引用 Eric Evans 的 DDD：领域专家与开发者一开始说的就不是同一种语言，Agent 也一样。 修复：项目根目录维护一份 CONTEXT.md（领域词典）+ docs/adr/（架构决策记录）。 作者举了自己 course-video-manager 仓库的例子：
· Before："a lesson inside a section of a course is made 'real' …"
· After："problem with the materialization cascade"
共享语言带来的连锁收益：命名一致 → 代码可导航 → 思考 token 更少。这一条被作者称为"整个 repo 里最酷的技术"。

3. 代码跑不通 —— 反馈回路缺失
引 Pragmatic Programmer："反馈速率就是你的速度上限。" 修复：把静态类型 / 浏览器 / 自动化测试的反馈接回来。
· /tdd：强制 red-green-refactor，并明确反对"horizontal slicing"（先把所有测试写完再实现）——只能 vertical slice，一次一个 tracer bullet
· /diagnose：固定的"复现 → 最小化 → 假设 → 插桩 → 修 → 回归测试"诊断循环

4. 系统变成屎山 —— Agent 加速软件熵增
引 Kent Beck 与 John Ousterhout：每天投资设计，深模块（窄接口、厚实现）优先。 修复：
· /to-prd 在写 PRD 前会问"这个改动到底碰哪些模块"；
· /zoom-out 强制 Agent 把局部代码放回系统全景里讲；
· /improve-codebase-architecture 是"周期性救火"——作者建议每隔几天对代码库跑一次。

# Skill 清单结构
仓库分三类，命名上都是 Slash Command 风格：

Engineering（日常代码工作） grill-with-docs、tdd、diagnose、to-prd、to-issues、triage、improve-codebase-architecture、zoom-out、setup-matt-pocock-skills

Productivity（通用工作流） grill-me、caveman（极简通信模式，省 ~75% token）、write-a-skill

Misc（不常用工具） git-guardrails-claude-code、migrate-to-shoehorn、scaffold-exercises、setup-pre-commit

各 Skill 之间不是孤立的，而是一条从对话到落地的流水线：
对齐与设计              落地与守护
──────              ────────
grill-with-docs   →     tdd
     ↓                   ↓
  to-prd            diagnose
     ↓                   ↓
 to-issues          zoom-out
     ↓                   ↓
   triage   ───→   improve-architecture
     ↑                   ↓
     └──── CONTEXT.md / ADR ──┘

项目地址
https://github.com/mattpocock/skills
--------------------------------------
```

# 16.AI知识

```
https://github.com/echohive42/AI-reads-books-page-by-page
https://github.com/echonoshy/cgft-llm
-------------------
人们已经开始用 NotebookLM，
几分钟内批量生成专属 Claude Skills。  

它的核心思路很简单：  
把精选资料丢进去， 
让 NotebookLM 先理解、整理、提炼， 
再转成可复用的 skill.md 文件。  

这样一来，Claude 不再需要你反复写提示词， 
而是可以直接像一个“被训练过的垂直专家”一样工作。
--------------------
```

# 17.Harness Engineering

```
Harness Engineering 在 AI 开发中的落地实践，强调不能仅依赖提示词，而应构建一套稳定、可控且可校验的系统。通过实际项目案例，将核心框架拆解为约束流程、结果反馈、知识索引和系统进化四块拼图。重点介绍了如何通过多角色 Agent 协作和标准操作手册（Skill）规范 AI 行为，并利用自动化脚本（Scripts）建立硬性的验收门禁。此外，推荐利用 dev-map 和任务看板 为 AI 提供全局视野，确保项目在持续迭代中不重复造轮子。最终，这种方法论旨在将 AI 从简单的辅助工具转化为受制度化管理、能持续产出正确结果的专业研发团队。
https://x.com/freeman1266/status/2047229036741763383
```

# 18.本地跑AI

```
本地跑AI的人注意了，这份小模型清单可能是你今年装机最值的参考

整理了一套能在Mac Mini或普通笔记本上流畅跑的小模型组合，上下文够用、不卡机、不烧显卡，适合真实日常场景。

先说结论：如果只能选一个，直接上 Qwen3.5 9B（GGUF / Q4_K_M），聊天、起草、研究、翻译全能扛，日常驱动没毛病。

然后按场景拆开说——

日常使用

1⃣ Qwen3.5 9B — 全能日常主力
2⃣ DeepSeek-R1 Distill Qwen 7B — 数学逻辑推理专用，慢但真的在思考

专业工作

3⃣ Qwen2.5 Coder 7B — 代码补全、重构、调试，比通用模型强
4⃣ Llama 3.1 8B — 长上下文场景、RAG、文档问答，输出质量一般但上下文能扛
5⃣ Phi-4 Mini Reasoning — 结构化推理、短代码，缺点是上下文窗口小

轻量高效

6⃣ Gemma 4 E4B — 轻量全能，写作聊天搞智能体都行
7⃣ Phi-3.5 Mini — 摘要提取文档聊天，配合大模型当副手很顺手
8⃣ Qwen3.5 2B — 改写标记摘要，轻量任务够用

微型模型（跑路由和分类用）

9⃣ Qwen3.5 0.8B — 分类、关键词路由、二元判断
🔟 Gemma 4 E2B-it — 快速问答、轻量智能体

如果预算只够两个模型，组合这样选：
想编码 → Qwen3.5 9B + Qwen2.5 Coder 7B
想省力 → Qwen3.5 9B + Phi-3.5 Mini

本地部署玩起来，云端订阅费能省一大截。
```

# 19.AI使用注意事项

```
你的AI是不是越聊越傻？教你一招解决“上下文污染”，让Gemini智商永远在线！🧠
这是一个门槛极低，但异常暴利的“纯净上下文”玩法。今天拿Google Gemini的新功能拆解：

1⃣ 痛点：上下文污染 (Context Pollution)
当你和AI多轮对话（比如写方案）时，前面的纠错、废话、无效尝试都会变成“噪音”。聊到第5、6轮，AI就会因为噪音太多而“降智”，逻辑混乱。
2⃣ 解决方案：Gemini Notebook (笔记本)
别再手动复制粘贴总结给新对话了！Gemini现在集成了NotebookLM。
第一步： 在当前对话聊出高质量结果后，点击右侧菜单 -> “添加至笔记本”。
第二步： 新建一个对话，点击“工具” -> 选择刚才的“Notebook”。
第三步： 直接下达新指令。
3⃣ 核心逻辑
你给AI提供了一个“没有废话的纯净知识库”。AI只读取你筛选过的高质量信息，忽略之前的噪音。

别再去报那些几千块的Prompt课了，核心逻辑是“降噪”。
以前是人肉复制粘贴，现在是AI自动读取纯净库。这才是高级玩家的用法。
```

# 20.Tw93大佬

```
你不知道的 Agent：原理、架构与工程实践
https://www.youtube.com/watch?v=Z5If1L3eFtw&t=4s
```

# 21.github有趣的项目

```
1.生成语音电子书
https://github.com/santinic/audiblez
2.学习opencode
https://learnopencode.com/
3.有趣的skill
https://github.com/yaojingang/yao-open-skills/tree/main/skills%2Fyao-bayesian-skill
4.画图工具
https://github.com/nicejade/markdown-online-editor
5.英语学习
https://github.com/byoungd/English-level-up-tips
6.windows常用app
https://github.com/stackia/best-windows-apps
7.ocr工具
https://github.com/datalab-to/chandra
8.llm
https://github.com/Jason2Brownlee/awesome-llm-books
https://github.com/microsoft/ai-agents-for-beginners
https://github.com/openai/openai-cs-agents-demo
https://github.com/mattpocock/skills/blob/main/improve-codebase-architecture%2FLANGUAGE.md
https://github.com/coreyhaines31/marketingskills
-------
Windows 上的离线语音听写和助手，本地跑 Whisper 识别和 Ollama 解析，不用云端服务就能语音打字、管日程。
https://github.com/benmaster82/writher
WritHer 跑在 Windows 托盘里，AltGr 听写直接粘贴到当前窗口，Ctrl+R 语音管理笔记和提醒。识别用 faster-whisper，理解用 Ollama，数据全在本地 SQLite。
----------------------------------------
偶然刷到一本被反复安利的电子书，却没时间通读？想先抓住核心观点和关键结论。
试试开源工具 ebook-to-mindmap：一键把电子书变成思维导图和文字总结，几分钟快速摸清全书脉络。
支持 EPUB / PDF，提供三种输出方式：文字总结、章节思维导图、整书思维导图。
GitHub：http://github.com/SSShooter/ebook-to-mindmap
主要特性：
- 解析 EPUB / PDF 电子书
- 三种 AI 模式：总结、章节导图、整书导图
- 书籍类型可选：社科类、小说类
- 智能识别章节，自动跳过前言、目录等非正文内容
- 同时支持 Google Gemini 与 OpenAI
- 缓存机制避免重复处理，省时也省成本
- 交互式导图：支持缩放、拖拽、节点展开
本地部署后配置 API Key 即可使用，有需要可以直接上手试试。
---------------------------------------------
真的太炸裂了！
Github 直接拿下81万超高星标收藏！
这个开源仓库把各行各业的职业角色都做成专属AI员工。
覆盖20多种职能大类、140多个细分岗位，每一个都对标行业专业水准。
只要给AI Agent设定好对应的职业身份，立马就能拥有一位专属专家助手。
还能同时启用多个AI代理分工协作，各做各的任务，效率直接拉满。
http://github.com/msitarzewski/agency-agents
------------------------------------------
面试前不摸清公司底细，进去就是送人头！
说真的，现在找工作最怕什么？不是简历写得烂，是对公司一问三不知，坐在HR对面脑子空空。天眼查是方便，但要看完整信息？掏钱吧。自己手动搜？一两个小时没了，信息还东一块西一块。
今天推荐一个开源神器：Agentic Company Researcher。输入公司名，一键出一份结构完整、信息量拉满的公司研究报告，直接拿去面试用。
它怎么跑的？后台是多个AI智能体分工干活——
1️⃣ 多源自动采集：官网、新闻、财报一网打尽，不用你一个个翻
2️⃣ Tavily内容筛选：专门过滤垃圾信息，只留高质量相关内容
3️⃣ 双模型协同作战：Gemini负责大量吞吐整理，GPT-4.1负责精排成稿，分工明确
4️⃣ 流式进度展示：边跑边看，不是黑盒子，进展随时掌握
5️⃣ React现代前端：实时刷新，报告生成完直接下载走人
6️⃣ 模块化智能体设计：想加什么能力自己扩展，不被框死
用起来也不复杂，克隆到本地，把API Key配上，直接开跑。
面试季来了，信息差就是竞争力。别人两眼一抹黑去面试，你已经把对方公司摸透了，这局谁赢还用说？
🔗 https://github.com/guy-hartstein/company-research-agent
----------------------
把文章、课程笔记或提纲整理成一组统一风格的中文手绘技术解释 PPT 风格整页 PNG 图。
https://github.com/helloianneo/ian-handdrawn-ppt
---------------------------
学习线性代数时，抽象概念一多、公式一密，想真正吃透常常很费劲。
《The Little Book of Linear Algebra》就是为初学者写的核心概念入门书：讲得清楚、不绕弯，帮你把难点拆开、逐个攻破。
它用循序渐进的方式，从向量与矩阵起步，逐步带你走到线性变换、特征值分解，再落到真实场景的应用。
GitHub：http://github.com/the-litte-book-of/linear-algebra
你将学到：
- 向量空间基础：从标量、向量到内积、正交，一次讲明白
- 矩阵运算：加法、乘法、转置、逆矩阵，系统梳理不零散
- 线性方程组：高斯消元、矩阵的秩、齐次方程组的关键思路
- 线性变换：核空间、像空间、基变换，抓住本质不只背公式
- 特征值与对角化：特征多项式、谱定理、二次型的典型用法
- 实战案例：计算机图形学、数据科学、机器学习中的常见应用
每章配有大量例题与练习，并提供 PDF、EPUB、LaTeX 三种格式，方便下载、随时开学。
---------------------------------
给 auth2api 增加了 OpenAI (Codex) 和 Cursor 的授权登陆。auth2api可能是目前唯一一个可以中转 Cursor 账号的项目了。
https://github.com/AmazingAng/auth2api
----------------------------------------
让 Agent 自主搞定金融研究：拆问题、拉数据、自查、出结论，全程不用人盯。
https://github.com/virattt/dexter
Dexter 是个金融研究 Agent，拿到问题后自己拆步骤、挑工具、拉实时市场数据，做完自己查一遍，不行就再来一轮，直到结果靠谱。
------------------------------
发现一个出行神器 👉 http://luodianyun.com
高速公路实时监控，全国各省都能查。
点开地图就能看到每个摄像头的实时画面，收费站堵不堵、服务区人多不多，一目了然。
还能专门筛选服务区和收费站的监控，出门前看一眼，避开拥堵太香了！
节假日出行必备，建议收藏 ！
---------------------------------------
本周 GitHub 涨星最快的开源项目：

1.mattpocock/skills (+30.9K stars） 
真正的工程师技能库。直接摘自我的 .claude 目录

2.forrestchang/andrej-karpathy-skills(+23.1K stars)
仅用一个 CLAUDE.md 文件就能优化 Claude Code 的表现，灵感来源于 Andrej Karpathy 对大模型编程常见陷阱的深入观察

3.Alishahryar1/free-claude-code (+14.7K stars)
让你在终端、VSCode 插件或类似 OpenClaw 的 Discord 机器人中免费使用 claude-code（支持语音输入）

4.Z4nzu/hackingtool (+8.7K stars)
专为黑客打造的 All-in-One 一站式黑客工具包

5.TauricResearch/TradingAgents (+6.0K stars）TradingAgents：基于大语言模型多智能体（Multi-Agents）的金融交易框架

6.huggingface/ml-intern (+5.7K stars)
ml-intern：一个开源的 AI 机器学习工程师，能全自动帮你读论文、训练模型并完成部署

7.abhigyanpatwari/GitNexus (+5.2K stars)
GitNexus：零服务器的代码智能引擎。它是一个完全在你的浏览器中运行的纯客户端知识图谱生成器。只需拖入一个 GitHub 仓库链接或 ZIP 压缩包，就能生成带有内置 Graph RAG 智能体的交互式知识图谱。简直是代码探索的神器

8.lsdefine/GenericAgent (+2.4K stars)
 自进化智能体：从仅 3300 行代码的种子起步，自行生长出技能树，用原来六分之一的 token 消耗就能实现对系统的完整控制

9.mksglu/context-mode (+2.3K stars)
专为 AI 编程助手设计的上下文窗口优化工具。通过对工具输出进行沙盒化处理，直接将上下文占用锐减 98%。目前已支持 14 个平台

10.AIDC-AI/Pixelle-Video (+2.1K stars)
AI 全自动短视频引擎

本周趋势总结： claude.md 配置文件和智能体技能树正在成为 GitHub 上新的技术护城河
--------------------------------------
做数据分析或者训练模型，卡在哪里？不是算法，不是框架，是找数据这一步就能把你搞死。
今天刷 GitHub 发现一个宝藏项目：Awesome Public Datasets，74k+ Star，不是那种没人维护的烂仓库，是真的在持续更新的。
覆盖领域直接列给你看：
🌾 农业
🧬 生物
💰 经济金融
🌍 气候环境
📚 教育
🏥 医疗健康
……几十个方向全有，基本你能想到的场景都覆盖到了。
最省心的地方是啥？
1️⃣ 每个数据集都标了可用状态，不用点进去才发现链接已经挂了
2️⃣ 附了元数据链接，来源清楚，引用起来不心虚
3️⃣ 标明数据格式：CSV、JSON、数据库……你用啥都能找到对应的
4️⃣ 体量和更新频率也都写好了，一眼看完直接决策
不用再一个个去 Kaggle 翻，不用靠运气碰数据，这一份清单基本能解决你80%的找数据需求。
建议直接收藏，下次要用的时候直接翻，省出来的时间去做真正有价值的事。
🔗 https://github.com/awesomedata/awesome-public-datasets
🔗 https://awesomedataworld.slack.com
-----------------------------------------
linux系统资源管理
https://github.com/matthart1983/syswatch
-----------------------------------------
```

# 22.构建企业 AI Agent 

```
在构建企业 AI Agent 的时候，工作上下文（Context）是不可或缺的元素，那什么是好的 Context，又如何构建好的 Context？
好的 Context，是一套能让 AI 理解“此刻该如何行动”的组织记忆系统，它至少包含四层：
1）情境记忆。也就是发生过什么、谁说过什么、在哪个时间点做过什么决定。这对应 Endel Tulving 在 1972 年提出的 episodic memory，对具体事件和经历的记忆。对企业来说，聊天、会议、文档、项目流、审批、工单，都是情境记忆。它的价值在于保留现场，让 AI 在面对结论时，同时能够理解当时的路径和判断过程。
2）语义记忆。也就是从大量情境中抽象出来的稳定知识，例如规则、术语、流程、产品定义、组织共识、经验方法。Tulving 把 semantic memory 作为一种不依赖具体经历的知识系统。知识库真正产生价值的地方，在于把零散材料逐渐沉淀成可以反复使用的结构。
3）程序化记忆。也就是“遇到某类问题应该怎么做”。后续的记忆研究中，procedural memory 常被单独拿出来看。映射到 AI 系统里，就是 SOP、模板、工作流、工具调用策略、Agent Skill。它会直接影响系统停留在建议层，还是能够进一步进入执行层。
4）工作记忆。也就是当前任务窗口里，AI 临时需要的那一小块高相关信息。像 MemGPT 这样的工作，会把 LLM 的上下文窗口当成一种稀缺资源，通过分层管理来调用更大的长期记忆。这个视角很关键，Context 的核心在匹配程度，是否刚好支撑当前任务。
那 Context 如何被有效地组织起来，让它变成真正有价值的 Agent 语料呢？不同的场景，需要不同的处理策略。
例如在复杂项目推进、多人协作决策、跨周期目标管理中，对上下文的处理，适用于递归式记忆蒸馏与回注机制（Recursive Distillation & Grounding）。在认知科学里，它更像是从情境记忆不断压缩到语义记忆，再反向投射回情境的一种循环。
它有两个同时发生的动作：1）一条是向上抽象，日报 → 周报 → 月报，本质是在做信息压缩，把大量具体事件提炼成模式、趋势和判断；2）另一条是向下穿透，周报和月报反过来影响日报，让后续记录逐渐带上结构和重点，减少无序堆积。

这两条链路形成一个闭环：经历不会直接沉没，而是不断被压缩、再利用、再强化。这和 Endel Tulving 提出的记忆转化过程是高度一致的：经历会逐渐抽象为知识，知识也会进一步参与后续行为的生成。
类似的探索，在工作场景中，还有一些常见的组织模式：
1）情境重构机制（Context Reframing），适用于问题推进卡住、讨论反复震荡的阶段。很多时候限制来自问题所处的框架本身。通过调整问题的边界、目标或观察视角，再把已有记录重新放进去看，会发现原本难以推进的讨论开始出现新的路径。同一批信息，在不同结构下会导向完全不同的判断，这种能力更像是在主动切换解空间。
2）记忆遗忘与权重衰减机制（Forgetting & Decay），适用于信息持续累积、系统开始变慢或噪声变多的阶段。信息如果被一视同仁地保留，会逐渐拖慢判断节奏。更有效的方式，是让信息在使用中自然分层，低频、过期、无效的内容逐渐退出核心上下文，高频被引用、对关键决策有贡献的内容则持续被强化。时间拉长之后，系统会变得更轻，也更准。
3）任务驱动的 Context 编排机制（Context Assembly），适用于多任务并行或 AI 执行复杂流程的场景。上下文围绕当前目标展开，挑选出最相关的一小部分信息，并按照任务需要组织起来。不同任务对应不同的上下文切片，这种按需组装的方式，可以在有限空间内保持信息的高相关性，让执行过程更稳定，也更可控。
Context 是生长出来的，需要逐步清洗、过滤和沉淀，形成对个体和团队分别有效的上下文。
从当下开始，去构建自己工作/生活/学习的上下文，逐步让 AI 进来参与决策，AI 会帮助我们慢慢沉淀出一套稳定的认知结构，直接影响判断的质量与方向。
或许，这也是让自己从繁琐的事务中解脱出来的必要路径。😄
https://pbs.twimg.com/media/HGt9LWxaAAAo4BN?format=jpg&name=small
```

# 23.Agent组件方法论

```
发现一个给 AI Agent 加“长期记忆”的纯本地方案：Mnemosyne（为 Hermes Agent 打造）。底层用 SQLite，查询延迟 <1ms，全程离线运行，零成本、零依赖、隐私自己掌控。

GitHub：https://github.com/plastic-labs/mnemosyne

官网：https://mnemosyne.site

核心亮点：

- 速度快：写入 0.81ms、读取 0.076ms、搜索 1.2ms，比 Honcho / Zep / Mem0 快 43–500 倍
- 纯本地：数据不出机，离线可用，无 API 调用，隐私可控
- 上手快：pip install 即用，不要 Docker、不配数据库、不写配置、不用注册/付费
- BEAM 三层记忆：working_memory（热上下文）+ episodic_memory（长期存储 + 向量/全文搜索）+ scratchpad（临时推理）
- 混合检索：向量 + 全文 + 重要性评分，LongMemEval 98.9% 召回率
- 自动整理：sleep() 压缩旧记忆，把工作记忆平滑转入长期记忆
- 迁移省心：一条命令从 Zep / Mem0 / Honcho / Hindsight 导入数据

适合本地 Agent 开发、重视数据主权、追求低延迟、需要离线运行的场景。强项是速度、隐私和成本；短板是缺少 Web 管理界面与团队协作能力。做本地 Agent 的话，值得上手试一把。

--------------------------------------

本周AI agent领域悄然发生了一个有意思的现象。

DeepMind、Anthropic、Alibaba等顶级实验室的最新论文集体指向同一个方向：智能体不再是简单调用工具的“聊天机器人”，而是正在变成可工程化、可审计、可规模化的真正生产力系统。

先看Agentic Harness Engineering——它把目前最头疼的“智能体支架”从手工调优、试错进化的黑箱，变成了可观测、可证伪的工程闭环。

系统被拆成三层：可版本回滚的组件文件、从百万轨迹token中提炼的结构化经验证据、以及可验证的决策预测。

每一次修改都变成可审计的契约。

结果？

Terminal-Bench Pass@1从69.7%提升到77.0%，超越人类设计的Codex-CLI，还节省12% token。

更重要的是，这个框架的优化能跨模型迁移，证明它抓到了结构本质而非特定模型的过拟合。

再看Alibaba的AgenticQwen-30B-A3B—一个只有30B参数的MoE模型，激活参数仅3B，却在真实工具使用任务上接近235B级别的Qwen3表现。

秘诀是两个并行强化学习飞轮：一个从自身失败中挖掘更难的推理问题，另一个用模拟用户不断制造误导场景来进化多分支行为树。

这套方法让开源实验室第一次在极低激活参数下实现了高性能工具使用，成本曲线被彻底改变。

还有RecursiveMAS，它直接挑战了多智能体通信的传统方式：不再让每个agent用文本消息互相喊话，而是通过潜在空间的递归计算传递状态。

结果是token消耗降低34.6%-75.6%，推理速度提升1.2-2.4倍，同时准确率平均提高8.3%。

OneManCompany则把多智能体团队从固定组织图，变成了动态“人才市场”：每个agent都是可招聘的Talent，任务时实时匹配，最优组合，失败后还能自动迭代。

这些论文共同勾勒出一个清晰趋势：agent系统正在从“实验玩具”走向“生产级工程”。

当我们还在讨论模型参数谁更大的时候，真正决定落地胜负的，可能已经是“谁先把智能体工程化”这件事。

你觉得agent工程会成为下一波AI红利的主战场吗？
-----------------------------------------
你花钱买 AI 课的时候 Google 在偷偷把同一套东西免费给出来

推上 
@RoyAmal
 整理了一份 Google 官方 10 门免费 AI 课的清单
路径都在 http://skills.google（Google Cloud Skills Boost）上
注册 Google 账号就能学 一分钱不要

我把 10 门课整理一版中文索引 链接在每条下面

1）Generative AI Basics（生成式 AI 基础）
入门第一课 讲清楚什么是 GenAI 应用场景有哪些
https://cloudskillsboost.google/course_templates/536

2）Large Language Models（大语言模型）
ChatGPT 这类系统底层怎么跑的
https://cloudskillsboost.google/course_templates/539

3）Responsible AI（负责任的 AI）
伦理 偏见 真实世界的风险
这门很多人会跳 但它其实是 AI 工程师的护城河
https://cloudskillsboost.google/course_templates/554

4）Full Learning Path（完整学习路径）
Google 官方排好顺序的 AI 学习路线 step by step
不知道从哪开始就直接走这条
https://cloudskillsboost.google/paths/118

5）Encoder-Decoder（编码器-解码器）
模型怎么把输入转成输出 翻译类模型的底层
https://cloudskillsboost.google/course_templates/543

6）Attention Mechanism（注意力机制）
Transformer 的核心 现代所有大模型的根基
这门是分水岭 听懂了再往后走 轻松一倍
https://cloudskillsboost.google/course_templates/537

7）Image Generation（图像生成）
文生图系统的原理 Stable Diffusion / Midjourney 那一套底层
https://cloudskillsboost.google/course_templates/541

8）GenAI Tools（生成式 AI 工具）
动手实验课 不光看视频
https://cloudskillsboost.google/course_templates/552

9）AI Applications（AI 应用）
真实业务场景里 AI 怎么落地
https://cloudskillsboost.google/course_templates/556

10）Vertex AI
Google Cloud 上怎么把 AI 模型 build + deploy 上线
这门最实用 直接对应工作技能
https://cloudskillsboost.google/course_templates/723

总入口 https://skills.google
所有课程集合 也可以从第 4 条「Full Learning Path」那门进去 一条线学下来

@RoyAmal
 在原推里的那句金句

大多数人的学习是 watch → forget（看完就忘）
这套路径给你的是 learn → practice → build（学 → 练 → 造东西）

如果你真打算认真学 AI，这是你免费版的 roadmap
-----------------------------------------
偶然刷到一个开源项目 AGENTS Book Rules，把 13 本经典编程书籍的核心原则，整理成了可以直接喂给 AI 编码工具的规则文件。

涵盖《代码整洁之道》、《领域驱动设计》、《重构》等十几本经典软件工程著作的核心思想。

GitHub：http://github.com/ciembor/agent-rules-books

每本书提供三个版本，完整版用于参考；精简版适合日常使用；极简版应对上下文窗口紧张的场景。

规则按任务类型分类，日常代码质量、架构设计、遗留代码处理都有对应推荐组合。

支持 Claude Code、Codex 和 Cursor，复制对应的规则文件到项目里就能用。
----------------------------------
我跟模型的交互，基本从 ChatGPT / Claude 转到 codex / claude code 上了。

所以我把 ChatGPT 里这三年的聊天记录全导出来，总共 2G，然后丢进 codex 里让它自己分析，重建一套对我的理解。
如果你也有类似的困扰，这一步可以做一下，哪怕只是本地备份也值。
方法很简单：在设置里点 export data，先会收到一封导出启动邮件，差不多 24 小时内会再来一封导出完成的，直接下载就行。
别问我为什么不导 Claude 的，老号没了，新号也没啥记录。
-----------------------------------------------
CLAUDE.md 终于有人把最全用法讲清楚了

原来很多人用 Claude 这么久，其实只用了它20%的能力。

作者直接把 CLAUDE.md 的核心玩法摊开了——一个简单文件，就能让 Claude 每次打开会话都不再从零开始，记住你的偏好、规则和上下文。

CLAUDE.md 不是只有程序员才需要的东西。写手用它锁死自己的语气，营销人用它定义受众，研究者用它固定输出结构，老板用它把公司背景一次性喂给 Claude。
没有它，你每次都要重复解释自己；有了它，Claude 就像一个越来越懂你的长期同事。
作者分享的核心玩法很简单：
在项目文件夹里新建一个叫 CLAUDE.md 的文件（大写，无空格），然后把规则直接写进去就行。Claude Code 每次启动会自动读取它，不用额外操作。
里面最实用的几条（我挑我觉得最立刻能用的）：
•  干掉所有“Great question!”之类的废话，开场直接给答案
•  大动作前先列2-3个方案，让你选方向再执行
•  不确定的时候主动说“我不确定”，别自信满满地编
•  长度匹配任务需求，别简单问题写四段，长任务却只给骨架
•  改东西前先确认，别擅自大改文件
•  专注用户问的问题，别顺手“顺便优化”其他地方
•  每次结束时总结改了什么、没改什么
•  永远不替你做外部动作（发邮件、发帖、改数据库）除非你明确说 yes
作者还提到 Andrej Karpathy 那版爆火的 CLAUDE.md 里4条核心规则，基本可以直接抄进去。
我看完后的感觉是，这东西就像给 Claude 装了一个永久记忆体和行为守则。设置一次，后面每次使用都省一大堆重复劳动。
尤其是对重度用户来说，真的值得现在就建一个试试。
AI 工具越来越强，但真正决定你能用到什么程度的，是你愿不愿意把这些“幕后规则”先搭好。
CLAUDE.md 就是其中最简单、回报最高的一个。
----------------------------------------
想转行做 AI 工程师？你大概率会先被两样东西劝退：碎片化教程，以及一眼就知道是 AI 拼出来的长文，既不成体系也不落地。
我在 GitHub 上挖到一个更靠谱的：AI Engineering Field Guide。它不是“经验贴”，而是基于 1765 份真实职位描述 + 实际面试经历整理出来的开源指南，数据驱动，直接对准求职和上手。
它把 AI 工程师这份工作的关键信息拆得很清楚：岗位在招什么、技能怎么配、面试怎么走、常见题型和真实案例长什么样。更重要的是，还按数据/后端/前端等不同背景，给出了可执行的转型学习路径。
GitHub：http://github.com/alexeygrigorev/ai-engineering-field-guide
内容覆盖角色定位与技能地图、完整面试准备清单、精选学习资源合集，以及真实的市场数据与项目案例；还收录了 OpenAI、Anthropic、Google、Meta 等 51 家公司的面试流程与经验整理，并附带 17 个 take-home 作业的真实案例分析。
如果你在认真考虑转型 AI 工程师，或想系统搞清楚这岗位到底在做什么、怎么准备，这份指南值得直接收藏。
--------------------------------
一个 CLAUDE.md 文件登上了GitHub热门榜第一！！
去研究了一下，到底有什么特别的。
现在的 LLM 写代码有四个坏毛病——
❶ 遇到模糊需求，不问，直接猜 ；
❷ 解决一个小问题，顺手写出一个"框架" ；
❸ 你只改一行，它把周围三个函数也动了 ；
❹ 没有权衡，没有澄清，直接跑；
它自作聪明以为在帮你，实际上是制造了一个又一个 bug。
这个热榜第一的 CLAUDE.md 文件，就是靶向四个坏习惯，用 Karpathy 的 4 条原则来强制约束。
1、先思考，不确定就问，不允许自选解释然后硬跑，遇到歧义要停下来。
2、最小可用代码没人要的抽象层？不写。"以后可能用到"的灵活性？不加。
3、像外科手术一样精准修改任务要求的代码，不顺手"优化"旁边那段。
4、把模糊指令变成可验证目标"加个验证"→"先写失败测试，再让它通过"。
使用方式极其简单：
把这个文件丢进项目根目录，Claude Code 从第一个任务起就遵守这套规则。
一个文件、零配置、零依赖、完全开源。
10万多Star，看来实际用下来效果还不错。

所以模型的问题，不一定是要用更好的模型来解决。
提示词工程的本质，是给 AI 设边界，而不是给它更多自由。
------------------------
闲鱼上卖899的Claudecode高阶使用技巧
跟普通创建sub agent不一样 
Agent Team模式帮你创建一个完整的开发团队，team中的Agent之间能互相沟通，真正实现一人团队💪
现在免费分享出来👇🏻
----------------------------
说个暴论，PM这个岗位，正在被AI一点点拆碎重写。

Marcus用Claude Code加一个自定义插件，一个人跑完了传统PM团队的完整交付流程。

他说的一句话直接击穿了本质：The conversation is the work。
不是比喻，就是是字面意思。

他的工作流现在是这样的，

1️⃣策略阶段，输入/ce-strategy，
代理会对话式采访你，直到生成完整的strategy.md。

2️⃣规划阶段，/ce-ideate /ce-brainstorm /ce-plan，自动生成所有票据直接推到Linear。

3️⃣每日监控，早上八点自动收到单页产品脉搏报告，数据异常会自动标注。

以前PM80%的时间，都在协调跨部门，写用户故事，追进度，刷仪表盘。
现在这些工作被压缩到了几乎为零，
剩下的20%，战略，用户洞察，判断力，反而被放大了一百倍。

兄弟们，这就不只是什么效率提升那么简单了，简直就是工作性质的彻底改变。

从我做OD的视角看，
这才是AI对组织最根本的冲击。

过去一百年，我们设计的所有组织架构，本质上都是为了解决信息传递和执行协调的问题。

PM这个岗位本身，就是这个体系的中间节点。
而现在，AI直接把这个节点给吃掉了。

所有的执行在AI时代变得无限廉价，
真正稀缺的，是定义什么值得做的能力。
是能从用户的只言片语里摸到真实需求的直觉。
能在无数个选项里做出正确取舍的判断力。
能把模糊的愿景变成清晰方向的战略思考。
Marcus仍然坚持每周花15分钟和真人用户通话，这件事他没有交给AI，因为他知道这才是所有答案的核心。
我觉得未来不会有那么多PM了，
但会有极少数真正的产品人，带着一支AI Agent组成的军队，做出以前整个团队才能做出来的产品。
我相信所有知识工作，最终都会走向这个结局。
#产品经理 #AI #组织发展 #ClaudeCode #职场
-----------------------------------
在 Agent 时代，你分享思路，别人让各自的 Agent 去定制化搭建就行了。
Karpathy 之前提过一个很有意思的想法：以后每个人都应该有一个自己的 LLM Wiki。
不是那种传统 RAG，每次提问都从一堆原始文档里临时检索，而是让 LLM 先帮你把资料“消化”一遍，整理成结构化的 Markdown 知识库。
它会生成页面、建立链接、标注冲突、持续更新，像一个长期帮你维护知识库的知识工程师。
现在，这个想法已经有人做成开源桌面应用了，项目叫 LLM Wiki
核心逻辑是：你把 PDF、Word、Markdown、网页文章等资料丢进去，系统先分析里面的关键概念、实体和观点，再生成互相关联的 Wiki 页面。
一个文档进去，可能会牵动十几个页面更新。没改过的文件会自动跳过，失败了还能重试，整个流程比手动整理知识库省太多事。
它不只是“生成笔记”，还做了知识图谱。你可以看到不同页面之间的关系，哪些内容连接紧密，哪些主题形成了聚类，哪些知识点是孤岛。
甚至还能自动发现“意外关联”和“知识缺口”，然后通过深度研究去网上搜索资料，把缺口补上。
还配了 Chrome 剪藏扩展。看到好文章，点一下就能提取正文、转成 Markdown，然后自动进入本地 Wiki 流程。生成的知识库也兼容 Obsidian，你可以用 Obsidian 查看，用 LLM Wiki 负责加工和维护。
--------------------------------
```

# 24.Codex使用

```
Codex越用越卡？别忍了！

用久了本地垃圾堆成山——聊天记录、worktree残留、日志文件、过期配置全压在一起，能不慢吗？

这个工具专门干这事：

🧹 清理积累的聊天缓存
🌿 删除废弃的worktree
📋 清空堆积日志
⚙️ 移除过期配置

一键瘦身，让Codex回到出厂状态的丝滑感。

🔗 http://github.com/vibeforge1111/keep-codex-fast
---------------------------------------
还在被Codex的英文文档折磨？这份保姆级中文教程救了我

说真的，Codex这东西上手门槛不低。安装配置搞不定、英文文档看不懂、AGENTS.md是什么鬼……很多人卡在起步阶段就放弃了。

今天给大家挖到一个宝藏资源，专门为国内用户整理的Codex全套中文教程，从零到能用，跟着走就行。

里面涵盖的内容相当硬核：

① 国内网络环境下的一键安装配置，不用自己踩坑

② Codex APP、CLI、Desktop App、VS Code扩展，四种形态全覆盖，配有截图，不是那种让你自己对着英文猜的教程

③ AGENTS.md模板怎么写、MCP Server怎么配，有实战案例直接抄作业

④ Skills使用案例加高效工作流技巧，不只是告诉你能做什么，还告诉你怎么做得更快

⑤ 常见报错排查和优化建议，踩过的坑都帮你填好了

新手看完能直接跑起来，老手也能查漏补缺，内容是以清晰步骤和截图为主，不是那种东拼西凑的水货。

现在Codex还在快速迭代，早点玩熟比什么都强。建议先收藏，用到的时候随时翻。

评论区聊聊你们用Codex都在做什么，互相交流一下~

🔗 https://github.com/xianyu110/gpt-codex
-------------------------
Codex 新手必看！5 分钟上手保姆级中文教程
很多朋友想用 Codex，但安装配置、英文文档、AGENTS.md 这些常常让人头疼
今天分享一个专为小白用户整理的详细中文资源
主要内容包括：
- 国内网络环境下一键安装配置指南
- Codex APP、CLI、Desktop App、VS Code 扩展的全形态保姆级图文教程
- AGENTS.md 模板编写方法 + MCP Server 配置实战
- Skills 使用案例与高效工作流技巧
- 常见问题排查和优化建议
内容以清晰的步骤和截图为主，从零基础到进阶都能快速跟上。特别适合新手快速入门，也方便有经验的用户查漏补缺
地址：https://github.com/xianyu110/gpt-codex
如果你正在学习或使用 Codex，建议收藏，方便以后随时参考
欢迎在评论区分享你的使用心得，一起交流～
--------------------------
好多人不知道如何开启 Codex 的 /goal，步骤如下：
1. 升级 codex 至 0.128.0版本
2.编辑  config.toml
3. 在 [features] 字段处，添加  goals = true
示例：
[features]  
goals = true

4. 重启 Codex
5. 输入 /goal
------------------------------
我给Codex做了个skill，让GPT模仿Opus的说话风格，专门治 GPT 不说人话的问题
https://github.com/0xsakura666/opus-style-output
这个skill的作用总结就是：
（1）分析和总结带表格
（2）说人话
（3）说话带emoji
（4）ui前端描述不带上开发思路，不要句子结构（这个贼恶心）

再用上最新的GPT5.5模型，vibe coding的舒适度大大提升，不用再花2-5倍的成本去用Opus了，现在我的codex be like：
```

# 25.吴恩达提示词

```
吴恩达 2026 年新课《AI Prompting for Everyone》21 节看完，提炼最值得抄的 6 条：
1️⃣ 新手和高手差 5-10 倍产出，差在 4 个维度：问题难度、上下文、是否引导、写作流程
2️⃣ 信息获取分 3 层：pretrained / web search / deep research
复杂任务用 deep research 比手刷网页快几十倍
3️⃣ Context 窗口能塞 75 万字（≈ 哈利波特前 4-5 本）
换话题就开新对话防污染
4️⃣ 忘掉 "Let's think step by step"，现在直接说 think hard 或 ultrathink
模型自己知道展开多少推理
5️⃣ ChatGPT 同意你的频率比不同意高 10 倍
反 sycophancy 4 招：中性提问 / 给评分卡 / 别埋偏见 / 列双方案
6️⃣ AI slop 4 大特征：滥用破折号、delve/nuanced、三人组排比、空洞 not X but Y
写作走渐进式大纲（出大纲 → bullet → 正文），不要让 AI 直接写正文
完整课程免费：http://learn.deeplearning.ai/courses/ai-prompting-for-everyone
---------------------------------------
如何快速判断一个人使用AI的水平？
Level 1：
完全不了解AI，从未主动使用过任何AI工具。
Level 2：
刚接触AI，好奇尝试但仅浅尝辄止，很快放弃。
Level 3：
把AI当作智能搜索引擎或作文机，直接复制简单输出。
Level 4：
掌握基础提示，能完成日常简单任务，但很少迭代。
Level 5：
学会思维链和角色扮演，用于常规工作，但复杂问题仍易卡住。
Level 6：
系统化设计提示，多轮迭代，像指挥AI一样工作，输出开始个性化。
Level 7：
AI深度嵌入个人工作流，能自优化提示并自动化重复任务。
Level 8：
构建端到端AI系统和多代理工作流，成为小团队放大器。
Level 9：
AI与认知深度融合，开发创新工具与治理框架，成果高度原创。
Level 10：
AI融入决策底层，创造新型人机共创，是战略创新者和贡献者。
```

# 26.如何在 YouTube 上找到 AI 行业的一手信息？

```
很多人学 AI，只盯着二手总结、热点解读和碎片化教程。
但真正值得长期关注的，其实是那些直接来自行业一线的内容：
创业者、研究员、投资人、产品负责人、工程师，他们在访谈、播客、发布会、大会分享里讲出来的东西，往往比整理过的文章更接近真实现场。
我一般会从这 5 类 YouTube 内容入手：
1. 访谈大佬的视频播客
想理解 AI 行业正在发生什么，最直接的方式就是听一线的人怎么聊。
推荐关注：
Lenny's Podcast
Peter Yang
AI and I by Every
Unsupervised Learning by Redpoint Capital
Training Data by Sequoia Capital
Minus One by South Park Commons
Google DeepMind: The Podcast
No Priors
AI + a16z
Latent Space
The AI Daily Brief
Lightcone Podcast by Y Combinator
Lex Fridman
Dwarkesh Podcast
这些内容不只是“新闻”，更像是行业内部人的思考记录。
他们会聊产品判断、模型能力边界、AI Agent、创业机会、组织变化、未来趋势。
很多时候，一个访谈里随口提到的一句话，可能就是下一个方向的线索。
2. 大佬在活动 / 大会上的分享录像
如果说播客更像深度聊天，那大会分享更像阶段性总结。
可以重点看这些：
YC Startup School
AI Engineer World’s Fair
Sequoia AI Ascent
Stripe Sessions
Figma Config
这些活动里，经常会有创始人、AI 工程师、投资人分享他们正在做什么、为什么这样做、遇到了什么问题。
比起只看别人整理的“大会重点”，我更建议直接看原视频。
因为真正有价值的东西，往往藏在案例、语气和上下文里。
3. OpenAI / Anthropic 等官方频道
官方频道一定要看。
尤其是 OpenAI、Anthropic、Google DeepMind 这类公司。
不是只看发布会，而是看：
产品演示
研究员访谈
开发者活动
技术讲解
API / Agent / Coding 相关更新
官方视频能帮助你判断一个很重要的问题：
这个公司到底希望开发者怎么用它的模型？
很多产品方向、开发范式、生态机会，其实都能从官方内容里提前看到。
4. 手把手教你用 AI 工具的干货教程
行业趋势很重要，但实际动手也很重要。
可以关注：
Riley Brown
Greg Isenberg
Ras Mic
Mckay Wrigley
这类频道通常会直接展示怎么用 AI 工具做产品、自动化工作流、Agent、内容生产、MVP 验证。
适合用来建立“AI 应用感”。
不要只停留在看概念，而是要观察别人怎么把模型能力接到真实任务里。
5. Andrej Karpathy
单独把 Karpathy 拎出来，是因为他的内容非常适合建立底层理解。
不管是神经网络、LLM、Token、训练过程，还是对 AI 产品形态的判断，他的表达都很清晰。
如果你想从“会用 AI 工具”进一步走向“理解 AI 系统”，Karpathy 的内容非常值得反复看。
最后说一下我的使用方法：
不要把 YouTube 当成娱乐信息流刷。
可以把它当成一个 AI 行业的一手信息库。
看到好的访谈，记下里面反复出现的关键词。
看到大会分享，留意他们强调的新问题。
看到官方演示，思考背后的产品方向。
看到工具教程，尝试自己复现一遍。
长期这样看，你会慢慢形成自己的判断：
什么是真趋势，什么只是包装；
什么是可以落地的机会，什么只是概念；
什么能力正在变得重要，什么技能正在被替代。
AI 行业变化太快，二手信息当然可以看。
但真正想建立认知差，还是要多看一手信息。
```



