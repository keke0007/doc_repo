---
author: "KC"
handle: "@ScarletKc"
source: "https://x.com/ScarletKc/status/2114225657651855608"
date: 2026-10-25T05:20:51.000Z
type: thread
---
# KC (@ScarletKc)

Anthropic 这篇新文章在劝搭 agent 的人少拆 subagent，里面有句话太真实了，很多企业的 agent 架构就是照着组织架构图抄的。每个团队交一个 subagent，平台组在最上面套个 orchestrator，康威定律又赢了  
  
思路是把 router、planner、按领域拆的 subagent，能折回主 agent 循环的都折回去，换成按需加载的 skill 和 tool  
  
他们经常看到的方案是一层层叠上去的，意图分类器、固定的多步推理流水线、用户分群、单独的推荐分支，最上面再套一层多 agent 编排。给的反馈往往是，这套结构大部分在替模型干它早就不需要别人替的活，一个写清楚的 system prompt、四五个很薄的数据 tool、加上严格的输出护栏，就够覆盖大部分场景  
  
Applied AI 团队在几家企业的部署里对比过，单 agent 加 skill 的质量经常同时赢过一个 prompt 包办一切和多 subagent 两种做法，很多时候成本和延迟也更低。有家客户拿单循环跟自己的多 agent harness 比，快了差不多一倍  
  
新的循环是一个模型、一个上下文窗口，harness 只管设上限，轮数、成本预算、写操作要过闸。模型每轮发一批 tool call，harness 并行跑完，把原始结果还给同一个模型，由它决定继续调工具、问用户还是回答。计划也是模型第一轮自己写进上下文，结果回来再改。写计划的就是执行的那个，中途随时能改，这是跟 orchestrator 那套最大的区别，以前用代码按规则表改计划的那一步可以直接删了  
  
原来各个 subagent prompt 里装的东西分到几个地方。团队做事的步骤、顺序和判断写成 skill。校验、审批这种硬限制，还有查记忆这种对话中途才要的，放进 tool。政策、指标定义、产品规格放知识库，让模型自己去查，别用代码查好了往 prompt 里粘。代码只留给便宜又几乎每次都要跑的步骤，和 tool 内部那些机械活  
  
上下文不会爆，靠的是渐进式披露。skill 和 tool 平时只露一句简介，用到才完整加载，tool 那边靠 tool search 延迟加载。以前拆 subagent 最主要的理由就是上下文塞不下，这下基本没了，Claude Code 给新模型的 system prompt 砍了 80% 以上，其中就有这块。而且所有步骤的结果都在同一个上下文里，模型能前后连起来看，比 subagent 各交回一份摘要强，也不会交接一次丢一次细节  
  
模型也撑得住了。从 Opus 4.5 开始，一个循环里能管的事就比大多数团队以为的多得多，Opus 5.5 和 Fable 5.1 的 thinking 都关不掉，还会边做边报进度边自查。帮客户迁新模型的时候，拦着新模型多干活的常常就是那套死板的多 agent 结构  
  
团队分工也跟着变。各团队不再各交一个 agent，改成交 skill 和 tool，一个很小的中央团队管 system prompt 和端到端质量，新的 skill、tool、eval 上线前都要过他们。不然就是每个团队往 system prompt 里塞一句，最后规则打架、口吻也不统一，文章管这叫 Frankenprompt  
  
各团队交上来的 tool 要按用户意图分组，别一个 API 一个 tool，定义超过 30 到 50 个，选工具就开始变差。前置条件和权限也写进 tool 里，缺了前一步的结果就返回一句模型能照做的提示，用户没权限就返回一句普通的拒绝。这样一个循环就能服务权限不同的用户，不用每个权限级别配一个 agent  
  
subagent 当然还有用，三种情况还是该拆。单循环因为上下文膨胀效果有了能测出来的下降，几件事要并行、各开一个用完就丢的干净上下文，或者单个 agent 老选错工具、system prompt 里指令打架。要拆也按上下文拆，给它一块不太依赖别处的活，让它交回精简的结果。按问题类型拆就成了传话游戏，交接一次失真一次。留下的 subagent 最好当 tool 调，主 agent 一直握着跟用户的对话  
  
迁移别想一步到位。先做 eval，10 条标准样例就够起步。从调用最多的开始折，生产流量里经常前后脚被调的两个专家 subagent，就适合先合成一个。之后一次只折一个，prompt 改成 skill，API 调用改成 MCP tool，再跟原来比  
  
比的时候别只看总分，要看哪些题翻了。他们拿一个示例客服 agent 试，折完延迟降了，但一个本来能过的流程坏了，主 agent 手里有查订单的 tool，就自己答了，没去问管物流的那个 subagent 要送达日期。这个问题跑两次只出现一次，这种最阴。所以生产上一起跑的每条 agent 链、每条不能破的规则都要有对应的题，每题多跑几遍。折回去的版本分数更低，才继续留着 subagent  
  
配套还有个迁移工具包，能先在模拟系统上跑 demo。几个 /claude-api skill 里我觉得 prompt-audit 最有意思，专门找当年为旧模型写的、现在反而让 agent 变慢变死板的规则  
  
对写 harness 的人来说最扎心的是这句，每出一代新模型，就能把更多决策从 harness 挪进模型里。我感觉 harness 以后只会越写越薄

---

> 🔗 [**Building agents: When to fold subagents into your main agent loop**](https://t.co/Fi5RRGN6N3)
> 
> _From claude.com_