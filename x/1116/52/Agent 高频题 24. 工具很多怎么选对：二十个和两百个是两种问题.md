---
author: "码良"
handle: "@cxjwin"
source: "https://x.com/cxjwin/status/2121798967440871436"
date: 2026-11-15T02:54:29.000Z
type: article
likes: 20
reposts: 1
replies: 6
bookmarks: 20
views: 2474
---
# Agent 高频题 24. 工具很多怎么选对：二十个和两百个是两种问题

*By 码良 (@cxjwin)*

![Banner](<Agent%20高频题%2024.%20工具很多怎么选对：二十个和两百个是两种问题.assets/image-01.jpg>)

百度一场 AIGC 多模态智能体二面，先问现在大约十个工具时怎么识别出正确的 Tool，接着问扩到几百或几千个，怎么保证命中率。深信服一场 Agent 开发实习二面问，现在 100 多个工具，AI 想调用的时候能找到自己想要的接口吗？怎么保障？

假设一个企业助手接了代码托管、工单、聊天、日历、文档五个 MCP Server，加起来 120 个工具。用户说「把昨天那个 bug 的进展同步到群里」。助手得先查到这个 bug，可是名字里带 search 的工具有四个。查到以后要发消息，发到群和发给个人又是两个工具。工具只有十来个时，难处在这种相近的工具之间挑对。到了 120 个，先要处理的是这些定义还放不放得进上下文。

这是 Agent 高频面试题单第 24 题。原题单记了 16 位作者、18 场面试。这次重读了库里留有链接、还能看到正文的 515 篇面经，其中 18 位作者在 21 场面试里遇到了这道题。下表按作者数排序，一场面试可以问到几个方向。

![Image](<Agent%20高频题%2024.%20工具很多怎么选对：二十个和两百个是两种问题.assets/image-02.jpg>)

__XPOSTER_v1f0x_TABLE_1__

「选得对不对怎么评测」的 3 场里，有 2 场来自同一位候选人，问的都是这位候选人做过的 Skill 路由评测。

## 一、120 个工具，出的是两种问题

第一种是装不下。每个工具的名字、描述和参数定义都随请求发给模型，算输入 token。Anthropic 2025 年 11 月介绍工具搜索的文章列过一组数：接 GitHub、Slack、Sentry、Grafana、Splunk 五个 MCP Server，共 58 个工具，对话还没开始，工具定义就占了约 5.5 万 token。再接一个 Jira 又是约 1.7 万。他们内部见过优化前工具定义占 13.4 万的。按这组数平均，一个工具约 950 token。照这个均值粗算，120 个工具要 11 万多，两百个约 19 万，一个 20 万 token 的窗口就差不多占满了。

第二种是分不清。同一篇文章写，最常见的失败是选错工具和填错参数，工具名字相近时尤其多，举的例子是 notification-send-user 和 notification-send-channel。Claude 的工具搜索文档给了一个经验值：可用工具超过 30 到 50 个，Claude 选对工具的能力会下降。OpenAI 的 Function Calling 指南建议，一轮开始时可用的函数尽量少于 20 个，并注明这只是软性建议。OpenAI 讲怎么构建 Agent 的那份指南说，问题不只在数量，还在工具之间是否相似、重叠：有的系统能用好 15 个以上界限清楚的工具，有的不到 10 个互相重叠的工具就已经吃力（第 3 题引过）。

所以二十个工具和两百个工具是两种问题。二十个装得下，要处理的是相近工具之间选错。百度一场 Agent 开发一面问的就是这个：Agent 有 20 个 Tool，怎么避免 LLM 选错 Tool？到了两百个，光定义就接近一个窗口，先要决定哪些定义进上下文，进来的那一小批仍然可能相近。

办法按代价从低到高排，下面三节依次讲。

__XPOSTER_v1f0x_TABLE_2__

__XPOSTER_v1f0x_IMAGE_3__

## 二、先把工具本身收拾好

网易一场 Agent 开发一面问，怎么设计 Tool 的 description 来提升调用准确率？写描述之前还有两步：先减，再起名。

第一步是减。Anthropic 讲工具写法的文章说，一个常见错误是把现成的接口原样一个个包成工具。例子里，工单 Server 把 ticket_list、ticket_get、ticket_list_comments 各包成一个工具，而助手查 bug 进展时三个总是连着调。可以合成一个 ticket_get_progress，一次返回状态、负责人和最近几条评论。那篇文章自己的例子是用一个 schedule_event 代替 list_users、list_events、create_event。Claude 的工具定义文档给了另一种合并方式：create_pr、review_pr、merge_pr 合成一个工具，用 action 参数区分，理由是工具少而全，选的时候歧义就少。助手用不到的管理接口，比如改权限、删项目，就不接进来。假设这样清完，120 个剩下 50 个。

第二步是起名。四个搜索工具如果分别叫 search、query、find_items、lookup，模型只能靠描述猜。Anthropic 建议加命名空间：按服务加前缀，服务里再按资源分，比如 asana_search、jira_search，再细到 asana_projects_search、asana_users_search。例子里就是 ticket_search、repo_issue_search、chat_message_search、docs_search。他们还发现，用前缀还是后缀对工具评测的结果有不小的影响，而且因模型而异，所以要拿自己的评测集来定。

第三步才是描述。Claude 的排障文档里，想让它调 B、它却调了 A 这一行，写的原因是描述有歧义，改法是按「什么时候用」来区分工具，不要只写「做什么」。工具定义文档建议每个工具至少写三四句：做什么，什么时候该用、什么时候不该用，有什么限制。例子里的难点是「昨天那个 bug」记在哪个系统。假设这家公司内部的 bug 都记在工单系统，代码托管上的 issue 只给开源项目用。老员工都知道，模型不知道。Anthropic 的说法是，像给新来的同事介绍工具那样写，把自己默认知道的背景写明白。ticket_search 的描述可以写成：

```text
搜索工单系统里的缺陷单和需求单。公司内部的 bug 都记在这里，
查 bug 的状态、负责人和进展时用。
不搜代码仓库的 issue（用 repo_issue_search），也不搜聊天记录。
```

chat_send_channel_message 也一样，写明「往群里发消息，发给个人用 chat_send_direct_message」。OpenAI 的指南还建议把每个函数什么时候用、什么时候不用写进系统提示词。

参数名也在给模型提示，Anthropic 举的例子是用 user_id 代替 user。两个相近的工具还可以各配一两个调用示例，他们提到示例对 create_ticket 和 create_incident 这类相近工具有用，代价是定义更长。参数怎么少出错，第 11 题讲过。

## 三、装不下：延迟加载，先搜再用

快手电商一场后端实习一面，先问模型怎么判断要调哪些工具（原理在第 27 题），接着问：如果 Agent 有几千个工具，怎么避免把所有工具描述都放进上下文？

做法是只让少数工具常驻，其余的先不进上下文，模型要用时先搜。以 Claude API 为例，请求里照样带上全部工具定义，不常用的加 defer_loading: true，再放一个搜索工具。

```json
{
  "tools": [
    {"type": "tool_search_tool_bm25_20251119", "name": "tool_search_tool_bm25"},
    {
      "name": "ticket_search",
      "description": "搜索工单系统里的缺陷单和需求单……",
      "input_schema": {"type": "object", "properties": {"query": {"type": "string"}}, "required": ["query"]}
    },
    {
      "name": "chat_send_channel_message",
      "description": "往群里发消息。发给个人用 chat_send_direct_message。",
      "input_schema": {"type": "object", "properties": {"channel_id": {"type": "string"}, "text": {"type": "string"}}, "required": ["channel_id", "text"]},
      "defer_loading": true
    }
  ]
}
```

模型一开始只看到搜索工具和没有延迟的 ticket_search。轮到发消息，它先搜一次。API 在全部定义的名字、描述、参数名和参数描述里匹配，默认最多返回 5 个，再把它们展开成完整定义。

```json
{
  "type": "tool_search_tool_result",
  "tool_use_id": "srvtoolu_01",
  "content": {
    "type": "tool_search_tool_search_result",
    "tool_references": [
      {"type": "tool_reference", "tool_name": "chat_send_channel_message"},
      {"type": "tool_reference", "tool_name": "chat_send_direct_message"}
    ]
  }
}
```

文档建议留三到五个最常用的工具不延迟，一次请求最多带一万个延迟工具。搜索有两种，一种由模型写正则，一种由模型写自然语言查询、按 BM25 做关键词匹配。

Anthropic 那篇文章里还有一张对比图，条件和 58 个工具那组不同。全量加载时，50 多个 MCP 工具的定义约 7.2 万 token，开工前上下文总消耗约 7.7 万。换成工具搜索后，常驻的只有搜索工具，约 500 token，用到时找出的三到五个工具约 3000，总消耗约 8700。图是按 20 万 token 的窗口画的。工具搜索文档的说法是，这类开销通常能减掉 85% 以上。文章还给了内部 MCP 评测的结果：工具库很大时，开了搜索，Opus 4 的准确率从 49% 到 74%，Opus 4.5 从 79.5% 到 88.1%。

__XPOSTER_v1f0x_IMAGE_7__

代价是多一步搜索，会加延迟。文档的建议是，工具有 10 个以上、定义超过 1 万 token、工具越加选得越差，或者聚合了多个 MCP Server，值得开；不到 10 个、每个请求都会用到全部工具，直接加载更合适。Claude Code 默认对 MCP 工具延迟加载，会话开始只带工具名和各 Server 的说明。OpenAI 的 API 也有 tool_search，它建议把延迟的函数按领域放进命名空间：模型一开始只看到命名空间的名字和描述，每个命名空间最好少于 10 个函数。两家都把新找到的工具追加在上下文后面，前面的内容不动，提示词缓存不会因此失效（第 12 题）。

用的平台没有这个能力，或者工具清单要按租户、按项目现查，就得自己做检索。深信服那场二面后面还问了一句：工具检索是怎么做的？

做法是给每个工具的名字、描述和参数名建索引，按查询取前几个候选交给模型。触发方式有两种：应用在调模型之前，拿用户这句话去检索；或者把检索包成一个工具，让模型自己决定什么时候搜、搜什么。后一种两家的 API 都支持，Claude 允许自定义的搜索工具返回工具引用、由 API 展开，OpenAI 有由应用执行查找的模式。

坑有四个，在「把昨天那个 bug 的进展同步到群里」这句话上都能看到。

1. 搜不到就选不到。模型只能在候选里选，检索的召回率就是选对率的上限。用户说「同步到群里」，工具描述写的是「向指定频道发送消息」，关键词对不上。描述同时是索引。Claude 文档的建议是，描述里要有用户描述任务时会用的词，找不到工具时先往描述里补常用关键词。Claude Code 的文档还提醒，匹配时只读描述靠前的部分，要被搜到的词得放在前面。
2. 一句话要用几个工具，一次检索不够。应用拿整句话去检索，前五个可能全是聊天工具，查工单的那个进不来。让模型自己搜，它可以先搜工单，查到 bug 以后再搜发消息的工具。
3. 搜空了不报错。Claude 的搜索工具没有匹配时返回空列表，不算错误，模型可能就此认为没有这个能力。文档的做法是在系统提示词里写明有哪几类工具可以搜，给的例句是「You can search for tools to interact with Slack, GitHub, and Jira.」，并且观察模型实际搜到了哪些工具，据此改描述。深信服那场二面还问，当前给的工具解决不了问题怎么办？可以让模型换个说法再搜一次，确实没有，就告诉用户缺哪种能力，不拿相近的工具硬凑。
4. 候选回来还是相近。上面那次搜索同时返回了发到群和发给个人两个工具，模型还得靠第二节写的名字和描述来分。

## 四、还分不清：按领域分层，或者换一种接口

顺丰一场 AI 全栈二面问，工具数量多的时候，你是怎么控制大模型选哪个工具的，有没有做控制？

能由代码决定的先由代码决定。流程走到某一步只该用某几个工具，就只开放那几个。OpenAI 的 allowed_tools 可以不改工具列表，只把模型能调的范围限制在一个子集里，文档说这样做是为了保住提示词缓存。某一步必须调某个工具，两家都能用 tool_choice 指定，Claude 的文档注明不是每个模型和设置都支持强制调用。

再往上是按领域分层：先判断请求属于哪个领域，再只给这个领域的工具。问题出在路由这一步。例子里的请求同时要工单和聊天两个领域，只选一个领域的路由一定缺另一个领域的工具。路由判错了，对的工具模型根本看不到。所以路由要能选多个领域。OpenAI 的命名空间加工具搜索是把这一步交给模型：它先看到各领域的名字和描述，需要哪个再加载哪个。意图路由怎么做、怎么量准确率，在第 17 题。

再重一点是拆给子 Agent。工单一个 Agent，聊天一个 Agent，各带自己的工具，主 Agent 只负责派活。每个 Agent 面对的工具少了，代价在交接上。Anthropic 今年 10 月有一篇讲子 Agent 何时并回主循环的文章，列了几条：每次调用交回来的是有损的摘要；每次调用是一个嵌套的模型循环，会加延迟；按问题类型拆出来的 Agent 来回传话，每传一次丢一点。他们比较过几个企业部署，带 Skill 的单 Agent 经常在质量上好过子 Agent 的设计，单任务的成本和延迟往往也更低。文章说，很多时候更有效的做法是渐进加载，也就是 Skill 用到才加载正文、工具用搜索延迟加载。它保留子 Agent 的三个条件里，有一条是单个 Agent 选不好工具，或者系统提示词里的指令互相冲突。OpenAI 那份指南的顺序也一样：先把名字、参数和描述写清楚，没有改善再用多 Agent。什么时候拆、什么时候并回来，第 3 题讲过。

最后一种是换接口：不把工具定义交给模型，让模型写代码去调。Anthropic 讲代码执行配 MCP 的文章里，工具按 Server 排成一棵目录树，一个工具一个文件，模型先列目录，再只读要用的几个文件。文章里的例子，token 用量从 15 万降到 2000。代价是要有沙箱、资源限制和监控，来跑模型写的代码。MCP 和 CLI 怎么选，第 7 题讲过。

## 五、Skill 很多时怎么办

字节业务中台一场 AI 全栈日常实习面问，有 100 个 Skill 和 100 个 MCP 该怎么处理，怎么让 Agent 精确匹配到你写的 MCP？一家小厂的 AI Agent 工程师一面出了道开放题：runtime 里有 500 到 800 个 Skill，怎么管理、保证上下文健康、一次调用命中该命中的？字节泛抖音一场 Agent 全栈一面问，一个项目里最多能加载多少个 Skill？

Skill 常驻在上下文里的只有名字和描述，正文用到才加载（第 8 题）。数量上去以后，碰到的还是那两种问题。以 Claude Code 为例，文档没有给「最多多少个」的数，给的是清单的预算：按模型上下文窗口的 1% 算，超了就从最少调用的 Skill 开始丢描述，只留名字。每条的描述加 when_to_use 最多 1,536 个字符。名字都还在，模型用来匹配请求的关键词少了。

办法和工具是同一个顺序。先清理：Claude Code 的 /skill-doctor 会列出每个 Skill 占多少上下文、被调用过几次，从没用过又占得多的先关。再合并相近的，在描述里写明什么时候不用。然后按目录、项目或角色挂载，不相关的不让模型看到。到了几百个，和工具一样，先检索候选再让模型选。度小满一场一面问，系统有几十个 Skill，每次调用都把它们放进上下文，Token 消耗大、命中率下降，怎么优化？如果放进去的是全文，先改成只常驻名字和描述，再做上面的前两步。几十个还用不到检索。

美团一场 AI 后端二面连着问了四句。怎么路由、冲突怎么处理、怎么持续提升准确率这三句，和清单预算、换模型后路由会变，都在第 17 题第八节。第四句是 Skill 增多导致上下文膨胀怎么办，答的就是上面的预算和清理。三层治理和两阶段检索的细节在第 8 题。

至于「精确匹配到你写的 MCP」，回到第二节：工具名加服务前缀，描述写清什么时候用。Claude Code 里还可以写 Server 说明，告诉模型这个 Server 管哪类任务、什么时候该来搜它的工具。

## 六、怎么知道选对了

科大讯飞一场 AI 应用一面问，Skill 路由评测主要评什么、怎么做？路由差是不是只改 description，Skill 很多或功能重叠时怎么办？同一位候选人在东方财富的 AI 应用一面被问：除了描述，误路由还有什么原因，Skill 数量有影响吗？路由怎么量化，什么阈值放行？百度那场 Agent 开发一面问，Agent 的 Tool Calling 应该怎么评估？Skill 的触发评测怎么配正反例，第 8 题讲过，这里讲工具这一侧。

评测集从真实请求里取，每条标上期望调用的工具。例子那句话期望的是 ticket_search、ticket_get_progress、chat_send_channel_message。还要配上长得像、答案不同的请求，比如「把那个开源库 issue 的进展发给小王」，期望的是 repo_issue_search 和 chat_send_direct_message。Anthropic 提醒，一个任务可能有几条都对的路径，期望工具别标得太死。

跑完分两层看。

第一层，对的工具在不在候选里。用了延迟加载或自己做检索，先看它有没有被搜出来。没搜出来，先改描述里的词或检索方式，这时去调两个工具的边界没有用。

第二层，在候选里，模型选没选对。把「期望的工具」和「实际调的工具」成对计数，次数最多的几对就是要改的地方。如果 ticket_search 被 repo_issue_search 抢走得最多，就改这两个工具的边界，别的不用动。

__XPOSTER_v1f0x_IMAGE_8__

参数填得对不对在第 11 题，任务最终完成没有在第 4 题。

所以「只改 description 吗」要先看错在哪一层。对的工具不在候选里，是检索或清单预算的问题。候选太多太像，要删、合并或分作用域，「数量有没有影响」指的就是这里。用户的话本身缺信息，比如没说哪个群，该反问（第 17 题）。换了模型，同一份描述的结果也可能变。

阈值没有通用的数，可以这样定。改之前先跑一遍当基线。改描述只对着一部分用例改，留一部分不看，Anthropic 优化自己的工具时就用留出的测试集防过拟合。每条多跑几次，他们讲子 Agent 的那篇文章里，有个回归只在两次运行中的一次出现。除了总分，还要看哪些用例从对变成了错。放行的条件可以定成：留出的用例不比基线差，要治的那一对混淆确实少了。发消息、改数据这类有副作用的工具，选错的代价高，光靠准确率不够，要在执行前加确认和拦截（第 11 题第七节）。

## 七、面试怎么答

**两分钟主线**

1. 工具多了有两种问题。一是装不下：工具定义算输入 token，Anthropic 举过 58 个工具占约 5.5 万 token 的例子。二是分不清：名字和描述相近的工具之间选错。二十个工具主要是后一种，两百个先是前一种。
2. 先收拾工具本身：用不到的删掉，总是连着调的合并，名字按服务加前缀，描述写清什么时候用、什么时候不用，把模型不知道的业务背景写进去。
3. 还装不下就延迟加载：最常用的三五个常驻，其余的让模型先搜再用。搜索按名字、描述和参数匹配，描述里要有用户会说的词。
4. 自己做检索要防四件事：搜不到就选不到；一句话要几个工具时一次检索不够；搜空了模型会以为没有这个能力；候选回来还是相近。
5. 再不够才分层：按流程和领域限制可用的工具，或者拆给子 Agent。路由会错，交接会丢信息，所以排在后面。
6. 用评测集验证：每条标期望工具，分开看「候选里有没有」和「有了以后选没选对」，统计最常混淆的是哪两个工具，改完在留出的用例上复测。

**准备自己的项目**

带一份脱敏的工具清单和一组你统计过的数：一共多少个工具，定义占多少 token，最常被混淆的是哪两个。讲一次具体的改动：改之前模型把哪个请求派给了哪个工具，你改的是名字、描述，还是合并了工具，改完同一批用例的结果怎么变。如果做过工具检索，准备一条搜不到的查询，说清描述里缺的是哪个词。

**容易答错的地方**

- 只答「优化 description」，不提先删和合并，也不说怎么验证。
- 把装不下和分不清当成一个问题，一开口就是向量检索。
- 讲检索只讲怎么建索引，不讲搜不到时会怎样。
- 工具一多就拆子 Agent，不提交接丢信息和延迟。
- 报一个准确率，却说不出评测集怎么来、最常错的是哪一对。

__XPOSTER_v1f0x_IMAGE_9__

## 出处

面经按发帖人记述转述。原题单和本次复核的样本范围、收录条件有差别。

- 工具多了怎么选对：[快手一面](https://www.nowcoder.com/feed/main/detail/8b684580bd464a19b45bcf600e004f82) · [深信服 Agent 开发实习一二面](https://www.nowcoder.com/feed/main/detail/14b2c379ae434062a009aefea9fc5df9) · [金山数娱 AI 应用开发一面](https://www.nowcoder.com/feed/main/detail/8f0f005a1f7a48958ac5f07bea3c5d88) · [拼多多 Agent 一面](https://www.nowcoder.com/feed/main/detail/4dd4b979b1f74625aaef3e7b975142ba) · [去哪儿一面](https://www.nowcoder.com/discuss/931958559204470784) · [金 9 银 10 真的是 28 的天堂吗](https://www.nowcoder.com/feed/main/detail/c7bb3d5f19104b4b8d2c0866ca9655b4) · [百度 AIGC 多模态智能体二面](https://www.nowcoder.com/feed/main/detail/2d033fd6daa04e2aaad2282b75531bbc) · [字节业务中台 AI 全栈日常实习](https://www.nowcoder.com/feed/main/detail/6ac5ba31705a4b64a407b087f39d5447) · [顺丰 AI 全栈二面](https://www.nowcoder.com/discuss/939846943163826176)。
- Skill 多了怎么管、怎么路由：[快手一面](https://www.nowcoder.com/feed/main/detail/8b684580bd464a19b45bcf600e004f82) · [小厂 AI Agent 工程师一面](https://www.nowcoder.com/discuss/927944523437674496) · [美团 AI 后端二面](https://www.nowcoder.com/discuss/929087871779110912) · [度小满一面](https://www.nowcoder.com/feed/main/detail/41baab8c631647568203ebfaf3898574) · [去哪儿一面](https://www.nowcoder.com/discuss/931958559204470784) · [科大讯飞 AI 应用一面](https://www.nowcoder.com/feed/main/detail/aece93373f9c47ea95c831aef6471886) · [东方财富 AI 应用一面](https://www.nowcoder.com/feed/main/detail/d406314e733c4fc8945637ce5d06cdcf) · [字节业务中台 AI 全栈日常实习](https://www.nowcoder.com/feed/main/detail/6ac5ba31705a4b64a407b087f39d5447) · [字节 Agent 全栈一面（泛抖音）](https://www.nowcoder.com/feed/main/detail/3d554c6d99e74b93a673fa9c39e9d113) · [快手 AI 全栈一面（10-14）](https://www.nowcoder.com/discuss/939918937486016512)。
- 定义占上下文和按需加载：[B 站 AI 应用 Agent 开发一面](https://www.nowcoder.com/feed/main/detail/0df78025653d492d86cc7b7273c2a0c2) · [九号公司 Agent 开发实习一面](https://www.nowcoder.com/feed/main/detail/c0b53fdd38e041d2bc779753d04ee2ce) · [深信服 Agent 开发实习一二面](https://www.nowcoder.com/feed/main/detail/14b2c379ae434062a009aefea9fc5df9) · [快手电商后端实习一面](https://www.nowcoder.com/feed/main/detail/78fc3b815b4a4b3bb27d642fb53da16b) · [小厂 AI Agent 工程师一面](https://www.nowcoder.com/discuss/927944523437674496) · [美团 AI 后端二面](https://www.nowcoder.com/discuss/929087871779110912) · [度小满一面](https://www.nowcoder.com/feed/main/detail/41baab8c631647568203ebfaf3898574) · [百度后端开发一面](https://www.nowcoder.com/discuss/932313667943616512) · [网易 Agent 开发一面](https://www.nowcoder.com/feed/main/detail/bad72b1323274ad5b172039f48bfa4b1)。
- 描述怎么写，重叠冲突怎么办：[美团 AI 后端二面](https://www.nowcoder.com/discuss/929087871779110912) · [去哪儿一面](https://www.nowcoder.com/discuss/931958559204470784) · [科大讯飞 AI 应用一面](https://www.nowcoder.com/feed/main/detail/aece93373f9c47ea95c831aef6471886) · [东方财富 AI 应用一面](https://www.nowcoder.com/feed/main/detail/d406314e733c4fc8945637ce5d06cdcf) · [快手 AI 全栈一面（10-14）](https://www.nowcoder.com/discuss/939918937486016512) · [网易 Agent 开发一面](https://www.nowcoder.com/feed/main/detail/bad72b1323274ad5b172039f48bfa4b1)。
- 选得对不对怎么评测：[科大讯飞 AI 应用一面](https://www.nowcoder.com/feed/main/detail/aece93373f9c47ea95c831aef6471886) · [金 9 银 10 真的是 28 的天堂吗](https://www.nowcoder.com/feed/main/detail/c7bb3d5f19104b4b8d2c0866ca9655b4) · [东方财富 AI 应用一面](https://www.nowcoder.com/feed/main/detail/d406314e733c4fc8945637ce5d06cdcf)。

外部资料按正文用到的顺序分组。文中的数字都带着原文的条件，换一组工具就不是这个数。

- 工具定义占多少、选错在哪、工具搜索的效果。[Introducing advanced tool use](https://www.anthropic.com/engineering/advanced-tool-use)（Anthropic，2025-11-24）· [Tool search tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool) · [Function calling](https://developers.openai.com/api/docs/guides/function-calling)（OpenAI）· [A practical guide to building agents](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf)（OpenAI）。
- 删、合并、命名和描述。[Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents)（Anthropic，2025-09-11）· [Define tools](https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools) · [Troubleshooting tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/troubleshooting-tool-use)。
- 延迟加载在几处的实现和缓存。[Tool use with prompt caching](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-use-with-prompt-caching) · [Claude Code 的 MCP 文档](https://code.claude.com/docs/en/mcp) · [Tool search](https://developers.openai.com/api/docs/guides/tools-tool-search)（OpenAI）。
- 分层、子 Agent 和代码执行。[Building agents: When to fold subagents into your main agent loop](https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop)（Anthropic，2026-10-22）· [Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp)（Anthropic，2025-11-04）。
- Skill 清单的预算和清理。[Claude Code 的 Skills 文档](https://code.claude.com/docs/en/skills)。