---
author: "XiaoAnLab"
handle: "@xiaoanlab"
source: "https://x.com/xiaoanlab/status/2113627415583043810"
date: 2026-10-23T13:43:39.000Z
type: article
likes: 7
bookmarks: 5
views: 403
---
# Graphify 实战教程：Claude Code 代码知识图谱与 token 节省指南

*By XiaoAnLab (@xiaoanlab)*

![Banner](<Graphify%20实战教程：Claude%20Code%20代码知识图谱与%20token%20节省指南.assets/image-01.jpg>)

先讲个场景：你接手一个八年历史的老仓库，新开会话让 Claude Code 改一个函数——它吭哧吭哧先读了四十个文件，额度先掉一大块。换个问题问，它又重读了一遍。

每次新会话都从零重读，是你 token 账单里最冤的一笔。

10 月 22 日，Graphify-Labs 的 graphify 冲上 GitHub Trending，单日涨星 1400 多，总星数突破 13 万。它干的就一件事：把整个代码库——连带文档、SQL 表结构、配置、PDF——变成一张知识图谱，让 AI 助手查图而不是翻文件。

Graphify-Labs/graphify

知识图谱听着玄乎，它比 grep 和向量检索到底强在哪？两个差别。

第一，代码部分走本地 tree-sitter AST 解析，确定性结果，不调 LLM，一个 token 都不花。只有文档、PDF、图片才走第二轮语义抽取，用的是你助手的模型或你自己配的 API key。

第二，每条边都带诚实标签：EXTRACTED 是代码里真实存在的连接，INFERRED 是模型推断出来的，AMBIGUOUS 是证据不足、存疑。调 bug 的时候只信 EXTRACTED，这种"分得清什么是猜的"的设计，比全盘相信 AI 强得多。

以及它不是向量库：没有 embedding，没有要常驻的 Neo4j 服务，整张图就是一份 graph.json 文件，跨会话复用。官方 Benchmark 页自报的数字：LOCOMO 评测 recall@10 拿 0.497（mem0 是 0.048，supermemory 是 0.149），建图消耗的 LLM 额度为 0。官方数字仅供参考，动手实测为准。

安装就两条命令（需要 Python 3.10+，先装好 uv）：

uv tool install graphifyygraphify install

包名是 graphifyy，双 y。这是社区翻车第一名：PyPI 上其他 graphify 开头的包都和它无关，手滑装错就是李鬼。装好后 graphify install 会自动识别你用的助手并注册 skill——Claude Code、Cursor、Codex、Gemini CLI，官方说支持 20 多种。

建图，在助手里敲：

/graphify .

跑完你会得到一个 graphify-out/ 目录，里面三样东西：graph.html（浏览器打开的可交互图谱，点节点、搜、按社区过滤）、GRAPH_REPORT.md（关键概念、意外的关联、建议你问的问题）、graph.json（完整的图，之后查什么都不用再读源文件）。

然后是本篇最实用的部分——查图的三个命令，终端里直接用：

graphify query "请求参数是怎么校验的"graphify path "FastAPI" "ModelField"graphify explain "APIRouter"

query 是自然语言问一个问题，返回一张裁剪好的子图；path 是找两样东西之间最短的路；explain 是解释一个概念的所有连接。官方 README 拿 FastAPI 仓库做了演示：explain "APIRouter" 返回节点在 [routing.py](https://routing.py) 第 2210 行、连接度 47、属于社区 2，还有它跟 RequestValidationError、Dependant、.get() 等 47 条连接——每条都标着是 INFERRED 还是 EXTRACTED。

想让助手养成"先查图再读文件"的习惯，再跑一次：

graphify claude install

它会装一个 hook：助手在调用搜索类工具、逐个读文件之前，会被推一把先去看图。Codex 用户注意，你们的 skill 触发写的是 [$graphify](https://x.com/search?q=%24graphify&src=cashtag_click)，不是 /graphify。

第一次上手，建议按这个顺序：先 /graphify . 建图，别急着问问题。建完先打开 GRAPH_REPORT.md，看"关键概念"和"意外关联"两节——这相当于图给你的一份陌生仓库导览。然后打开 graph.html，找度数最高的几个"神节点"，点进去看它们都连着谁。最后才针对具体需求用 query、path、explain。这个顺序走下来，一个陌生仓库的结构基本就立体了。

下面是社区里真实踩过的五个坑，按翻车频率排：

坑一：终端里敲 /graphify . 报 error: unknown command '.'。斜杠开头的形式只在助手内部有效——那是 skill，不是 shell 命令。终端里建图用的是 graphify extract .。

坑二：在终端敲 /graphify，提示只能由 Claude 调用。这是正常的。/graphify 是 Claude 内部调的命令，你不用手敲，跟 Claude 说"对这个仓库跑一下 graphify"，它自己会触发。

坑三：graphify extract --backend claude 报缺 ANTHROPIC_API_KEY。这条路走的是 Anthropic SDK 直连，不认你 Claude Code 的 OAuth 登录。要么去配独立的 key，要么直接用 /graphify .，让助手在订阅额度里做语义抽取。

坑四：.docx 文档被静默跳过。需要装 office 扩展：

uv tool install 'graphifyy[office]'

视频转写同理是 [video]，MCP 服务端模式是 [mcp]，给谁装什么按需来。

坑五：图会变旧。代码一提交图就过期，但别全量重建——/graphify . --update 只扫变更部分，代价小得多。不过有个历史教训：2026 年 5 月的 Issue #857 里，update 曾把文件静默标记为已处理，导致文档边的语义连接悄悄过期。现在是否修好我没法 100% 确认，稳妥做法是：重要结论之前，先全量重建一遍再信。

隐私方面说清楚：代码解析全本地，不出机器；但文档和媒体的语义抽取会发给你的助手模型或 API 后端。公司仓库用之前，先看一眼这条分界线。另外查询默认会记一份本地日志，不想要可以关掉。

最后说句大实话：Graphify 不是银弹。几千行的小项目没必要，grep 更快；它的主场是接手大仓库、跨会话反复问结构、新人快速 onboarding。官方还有个商业版 Graphify Cloud（YC S26 的公司在做），卖点是常驻图谱加 Sentry、Jira 连接器，开源引擎本身免费，Apache-2.0 协议。

一句话总结：给代码库画一张地图，之后每次问路，都不用重新翻街景。