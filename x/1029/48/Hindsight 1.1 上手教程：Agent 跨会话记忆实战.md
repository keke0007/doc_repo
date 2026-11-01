---
author: "XiaoAnLab"
handle: "@xiaoanlab"
source: "https://x.com/xiaoanlab/status/2114715979146223641"
date: 2026-10-26T13:49:13.000Z
type: article
likes: 6
reposts: 1
bookmarks: 4
views: 510
---
# Hindsight 1.1 上手教程：Agent 跨会话记忆实战

*By XiaoAnLab (@xiaoanlab)*

![Banner](<Hindsight%201.1%20上手教程：Agent%20跨会话记忆实战.assets/image-01.jpg>)

周一早上打开 Claude Code，准备继续上周那个项目，它先问你：这个项目用 pnpm 还是 npm？上周你刚跟它立过三条规矩——别碰那个 legacy 支付文件、测试只在 UTC 下跑、提交前先跑 lint——现在全得重讲一遍。

这就是编码 Agent 的日常失忆：会话一关，上下文全清，下次开工先花 20 分钟重新自我介绍。

先给判断：Hindsight 1.1 是目前给编码 Agent 接长期记忆最省事的一条路，一条命令能跑起来、一条命令能接进 Claude Code；但它只负责把规矩和经验留下来，做判断还得你来，记忆脏了它也会一本正经地按脏规矩办事。

vectorize-io/hindsight

# 存进来的，是提炼过的事实

很多人一听记忆库，第一反应是往向量库里塞聊天记录，查的时候按相似度捞几段回来。Hindsight 走的是另一条路：你让 Agent 记住一件事，它先把这件事抽成结构化事实，再做实体解析——比如“小李”和“工程部的小李”会被认成同一个人——然后挂上时间信息、生成向量、写进图里。

用起来就是三个动作：retain 存、recall 查、reflect 推理。recall 查的时候跑四路并行：语义搜索、关键词匹配、实体图遍历、时间过滤，再用交叉编码器重排一遍，捞回来的是最相关的那一小撮，沾边的原文会被排在后面。

还有一个常驻的东西叫心理模型，官方叫 mental model：相当于一页会自己更新的活文档，比如“一页写清我的技术栈偏好”，新记忆进来它就跟着改，不用每次都跑一遍完整的 reflect。

和 Claude Code 自带的记忆文件比，差别在这里：CLAUDE.md 这类文件是你手写、Agent 照读，改一次要你亲自动手；Hindsight 是 Agent 在干活过程中自己提炼、自己更新，你只负责立规矩和纠错。代价也有：recall 默认只回 4096 个 token，银行里攒的记忆多了，旧的、低相关的会被截掉。实际影响是，别指望一次查询倒出全部家底，真正关键的规矩要沉淀到心理模型里常驻，散在几百条事实里等检索，关键时刻反而捞不出来。

# 三条上线路线，先按你的机器选

路线一：Docker，一条命令起全套。 适合大多数人，API、内置数据库、本地 embedding 模型、MCP 端口一次全起来：

docker run --rm -it --pull always -p 8888:8888 -p 9999:9999  
-e HINDSIGHT_API_LLM_API_KEY=你的密钥  
-v [$HOME](https://x.com/search?q=%24HOME&src=cashtag_click)/.hindsight-docker:/home/hindsight/.pg0  
[ghcr.io/vectorize-io/hindsight:latest](https://ghcr.io/vectorize-io/hindsight:latest)

8888 是 API 和 MCP 端口，9999 是可选的管理界面。注意一个关键点：HINDSIGHT_API_LLM_API_KEY 是给 Hindsight 自己内部用的，做法术抽取、实体解析、推理，跟你 Agent 用的模型密钥是两回事，完全可以用便宜快的模型，比如 Gemini Flash 或 Groq 那一路，不用跟 Agent 抢同一个贵模型。

路线二：pip 装进自己的环境。 需要 Python 3.11 以上，官方建议内存至少 4GB：

pip install hindsight-apiexport HINDSIGHT_API_LLM_API_KEY=你的密钥hindsight-api

这条路适合想把记忆嵌进自己 Python 服务的人，后面还能直接用 Python、Node、Go 的客户端 SDK 调用，不走 MCP 也行。

路线三：不想运维，直接用云端。 注册免费账号、拿 API 密钥，MCP 地址换成云端那一头：

[https://api.hindsight.vectorize.io/mcp/你的银行名/](https://l.meta.ai/?u=https%3A%2F%2Fapi.hindsight.vectorize.io%2Fmcp%2F%25E4%25BD%25A0%25E7%259A%2584%25E9%2593%25B6%25E8%25A1%258C%25E5%2590%258D%2F&h=AUD_oo0mIrU0FiMA-1FNBqyD2B2Qff1RSqWsO3ru-fBjifI4StmyFjyDf2KV21w-5l2keiY0YQ2l-jknjs_ZGUmNtl-kHkckTSMeaZO9ewBm4np1d5kzow)

三条路没有高低之分，只有适不适合：有 Docker 就走路线一；要嵌进自己代码就走路线二；只是想先试试效果、电脑不想多跑服务，就走路线三。

# 接进 Claude Code：两条路，别混着用

第一条：官方 coding-agents 包，一键接。 这是给编码 Agent 准备的捷径：

npx [@vectorize](https://x.com/@vectorize)-io/hindsight-coding-agents install claude-code

它会按仓库自动建一个记忆银行，还会从 git 历史和过往会话里先建一批库，等于开工前已经帮你把项目背景铺好了。想全接也行，把 claude-code 换成 all，它会把检测到的 Agent 一个个接上。

第二条：手动接 MCP，控制权在你手里。 本地部署的话：

claude mcp add --transport http hindsight [http://localhost:8888/mcp/你的银行名/](https://l.meta.ai/?u=http%3A%2F%2Flocalhost%3A8888%2Fmcp%2F%25E4%25BD%25A0%25E7%259A%2584%25E9%2593%25B6%25E8%25A1%258C%25E5%2590%258D%2F&h=AUD_oo0mIrU0FiMA-1FNBqyD2B2Qff1RSqWsO3ru-fBjifI4StmyFjyDf2KV21w-5l2keiY0YQ2l-jknjs_ZGUmNtl-kHkckTSMeaZO9ewBm4np1d5kzow)

接好先验证，别直接开工。问 Agent 一句“你有哪些记忆工具”，它应该列出 retain、recall、reflect 和心理模型那组工具，列不出来就是地址或银行名写错了，先修连接再谈记忆。

银行怎么分，这里有个实测教训：一个项目一个银行，别图省事让好几个项目共用一个银行。有用户报告过，共享银行的设置下，知识页会出现空白或者串味的内容。银行在 URL 路径里，换个名字就是新银行，隔离成本几乎为零，没必要省这一步。

# 两次会话，验证它真的记住了

别一上来就往银行里倒全量历史，先跑两次会话做个小验证。

第一次会话：立规矩。 正常开工，顺手把三条项目规矩告诉 Agent：这个项目用 pnpm、别碰 legacy 支付文件、测试只在 UTC 下跑。结束前多问一句“你都记住了什么”，让它把刚存的事实复述一遍，顺手纠掉记偏的地方——记忆这东西，第一次立歪了，后面每次都会按歪的来。

第二次会话：新开一个窗口，看它还记不记得。 别先提醒，直接让它动手改一处代码，看它会不会主动按规矩来：用 pnpm 装包、绕开那个文件、测试切到 UTC。如果它还是从零问起，说明连接或者银行没对上，回去查上一节的验证步骤。

中间有个时间差要适应：retain 存完之后，事实抽取和索引是在后台跑的，刚存完立刻查可能查不到，等几秒再查。还有一个零成本的验证法：手头暂时没有 LLM 密钥也能先跑起来，官方文档里有个无模型模式，服务能启动、银行能读写、MCP 能通，先确认管线没问题，再配密钥开抽取和推理能力——学习的那部分能力是一定要 LLM 的，没密钥就是个包装得很好的本地向量加关键词库，这点官方实测文章也摊开讲了。

# 1.1 刚发 5 天，值得升的三点和一个破坏性变更

1.1.0 是 10 月 21 日发的，1.0 是 10 月 13 日发的，中间只隔一周，但有几处是日常用得着的：

第一，文档有了变更历史。 以前事实被改、被追加、被作废后，基于它重建的观察就没了痕迹，现在每份文档都能看到它经历了什么、是哪次重建造成的，记忆被改歪了能往回查。

第二，观察分组更灵活了。 支持按标签通配符分组，还能给整个银行设默认分组，不用每个客户端都传一遍参数。

第三，供应商和下载变方便了。 新增了 ChatGPT 登录、NVIDIA NIM、ONNX 重排器、Anthropic 免静态密钥这几种接入方式；知识页能直接下载成 PDF 或 Markdown，沉淀下来的项目知识终于能带走了。

但升级前先看破坏性变更：新银行的 ID 不能再以下划线开头，下划线前缀成了保留位，导入存档时 ID 不合法会直接报 400，老银行不受影响。另外升级启动时会跑 5 个数据库迁移，大库要多留点启动时间，别在赶工的时候升级。

# 四个坑，都是官方 FAQ 和用户实测里有的

坑一：Docker 没挂 volume，重启丢库。 上面那条 Docker 命令里的 -v 不是装饰，不挂它，容器一停记忆就没了。第一次跑通之后，先重启一次容器、再查一次银行，确认记忆还在，再往里面倒真东西。

坑二：刚存完立刻查，查不到。 这是上一节说的异步处理，不是 bug。复杂记忆多等几秒，简单事实也别写完立刻当场考它，流程上留个缓冲。

坑三：换 embedding 模型，旧库直接打架。 换模型意味着向量维度变了，老银行会报维度不匹配。办法只有两个：要么一个银行从头到尾固定一个 embedding 模型，要么换模型就重建向量、重导数据。别在银行里攒了几千条记忆之后才想起这件事。

坑四：Docker 重启后，任务变僵尸。 官方 FAQ 里专门写了这个：工作进程的身份默认取容器主机名，Docker 每次重启都会换，导致旧任务一直卡在处理中，银行统计里的待合并计数永远下不去。预防是在启动时固定 HINDSIGHT_API_WORKER_ID；已经卡住了，用管理命令行把死掉的工作进程退役：

hindsight-admin decommission-worker 旧工作进程的 IDhindsight-admin decommission-workers

前者清掉指定的，后者全量释放，选一个就行。

# 三类人，三种接法

每天用 Claude Code 磨同一个项目的人，走 coding-agents 一键安装加每项目独立银行，先跑一周，看它立的规矩准不准，准了再扩大到更多项目。

自己写 Agent、或者多个客户端共用记忆的人，走 MCP 或 SDK，银行按用户、按项目分开，关键规矩沉淀到心理模型里，别全靠查询时现捞。

只是想先看看记忆这事靠不靠谱的人，别一上来就倒全量历史：云端免费档或者本地无模型模式先把管线跑通，验证两次会话真的能接上，再决定配不配密钥、花不花 token。

最后提醒一句跟选型有关的实话：官方 README 里提到的 LongMemEval 最好成绩是厂商自述，独立验证还没看到；星数也一样，10 月初约 4.5 万星、10 月下旬约 5.4 万星，涨得快说明有人在用，不等于在你项目上一定好用。两次会话验证法跑一遍，比看任何榜单都管用。

你家 Agent 上次失忆，是忘了规矩、忘了人，还是忘了上次踩过的坑？