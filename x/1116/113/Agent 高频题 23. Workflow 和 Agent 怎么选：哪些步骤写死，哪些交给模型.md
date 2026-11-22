---
author: "码良"
handle: "@cxjwin"
source: "https://x.com/cxjwin/status/2121443670566322342"
date: 2026-11-14T03:22:40.000Z
type: article
---
# Agent 高频题 23. Workflow 和 Agent 怎么选：哪些步骤写死，哪些交给模型

*By 码良 (@cxjwin)*

![Banner](<Agent%20高频题%2023.%20Workflow%20和%20Agent%20怎么选：哪些步骤写死，哪些交给模型.assets/image-01.jpg>)

千问一场 AI 研发一面，记下来的第一问是：你理解的 Agent 是什么？什么样的场景适合用 Agent，什么样的场景适合固定的 Workflow？第二问里接着问，哪些部分做固定流程，哪些交给模型决策？阿里 Token Foundry 一场 AI 应用研发二面，候选人讲项目时用了「受控 Agent Workflow」这个说法，面试官追问：具体受控在哪里？

拿一张退款申请来说。把分类、查订单、按规则判断、生成回复写成几步依次调用，这是 Workflow。给模型查订单、查政策、追问用户、发起退款几个工具，让它自己定下一步，这是 Agent。样本里被追问的项目，有几个是两样拼在一起的，追问也就落在哪几步写死、哪几步交给模型、各自为什么。

这是 Agent 高频面试题单第 23 题。原题单记了 17 位作者、23 场面试。这次重读了库里留有链接、还能看到正文的 515 篇面经，其中 26 位作者在 37 场面试里遇到了这道题。下表按作者数排序，一场面试可以问到几个方向。

![图像](<Agent%20高频题%2023.%20Workflow%20和%20Agent%20怎么选：哪些步骤写死，哪些交给模型.assets/image-02.jpg>)

第四行的 8 场里，有 6 场是同一位候选人带着同一段改造经历去不同公司面的，这一行按 3 位作者看。

## 一、同一张退款申请，两种写法

假设用户发来一句「订单 A1024 的耳机想退」。店铺的退款政策有两条：签收 7 天内没拆封的，直接退；有质量问题的，15 天内凭照片或视频退。

Workflow 的写法，路径在代码里：

```python
def handle_refund(ticket):
    kind = classify(ticket.text)                 # 模型：是不是退款申请
    if kind != "refund":
        return route(kind)
    order = get_order(ticket.order_id)           # 工具：查订单
    ok = order.days_since_signed <= 7 and order.unopened   # 规则：按政策判断
    if ok:
        issue_refund(order.id)                   # 工具：发起退款
    return write_reply(ticket, order, ok)        # 模型：生成回复
```

Agent 的写法，代码里只有一个循环，走哪条路每一步由模型定：

```python
tools = [get_order, get_refund_policy, ask_user, issue_refund]
messages = [SYSTEM_PROMPT, ticket.text]
while True:
    resp = llm(messages, tools)                  # 模型：定下一步
    if not resp.tool_calls:                      # 不再调工具，结束
        break
    messages += run(resp.tool_calls)             # 执行工具，结果放回上下文
```

![图像](<Agent%20高频题%2023.%20Workflow%20和%20Agent%20怎么选：哪些步骤写死，哪些交给模型.assets/image-03.jpg>)

订单在 7 天内、没拆封，两种写法结果一样。差别出在例外上。

用户写的是「用了十天，左耳有杂音」。Workflow 按第一条判了不能退，第二条政策还没写成分支。要支持它，得有人加分支、测试、发版，在那之前这类单子只能转人工。Agent 调 get_refund_policy 读到第二条，接着用 ask_user 向用户要照片。

开发补上了质量问题的分支：要照片，等用户上传。用户发完照片又说「算了，换一副行吗」。这条分支只认「收到照片」和「超时」，换货得再加一条。Agent 把这句话当成新的输入接着走，前提是它手里有换货的工具。

反过来，Agent 手里拿着 issue_refund。同一张申请跑两次，可能一次先查政策，一次先查订单。照片还没到就发起退款，也没有哪行代码拦着。Workflow 里退款前面的条件是代码判的，不满足就走不到那一行。

Workflow 没写到的情况，落在开发和人工客服身上。Agent 能接住没写到的情况，它判断错了由谁拦，要另外设计，第四节讲。

## 二、区别在哪：下一步由谁定

字节一场 AI Agent 开发一面问，你怎么理解 Agent 和 Workflow 的区别？B 站一场 Agent 二面问的是 Workflow 和一个整体的 Agent 有什么区别。

Anthropic 2024 年 12 月的文章「Building effective agents」把两者都归在 agentic systems 下面，再按架构分开：Workflow 是用预先定义的代码路径把模型和工具编排起来的系统；Agent 是模型动态地指挥自己的流程和工具使用、对怎么完成任务保有控制权的系统。LangGraph 文档的说法差不多：Workflow 有预先确定的代码路径，按一定的顺序运行；Agent 是动态的，自己定流程和工具的用法。

放回退款的例子。Workflow 里「先分类，再查订单，再判断」这个顺序写在 handle_refund 里，是写代码的人定的。Agent 里先查什么、要不要追问、什么时候结束，是模型每一步看着上一步的结果现定的。第 14 题第一节按「下一步什么时候定、谁定」排过一张表，Workflow 和状态机在第一行：写代码的时候，人定。

Workflow 里模型也在做决定。Anthropic 那篇文章列了五种 Workflow 模式：提示链（prompt chaining）、路由（routing）、并行（parallelization）、编排者与工作者（orchestrator-workers）、评估者与优化者（evaluator-optimizer）。路由是让模型或者传统分类器选一条分支。编排者与工作者这一种，子任务有几个、各是什么，由负责编排的模型按输入现拆。这些决定都发生在代码留好的位置上：分支就那几条，拆完派出去、收回来再汇总，这个骨架不变。Agent 的循环里，调哪个工具、调几次、什么时候停，没有事先画好的线。

字节电商一场日常一面追问：用户是在和 Agent 对话，还是直接和大模型对话？你的系统到底属于 Agent、Workflow 还是 Skill？（Skill 是写给模型看的做事说明，三者的关系第六节讲。）小红书一场 Product Engineer 一面的问法是：现在更像是把 LLM 当一个功能点使用，还是 Agent？

判断自己的系统算哪一种，可以去找决定下一步的那段代码。是 if/else 或者固定的调用顺序，这一段是 Workflow。是「模型返回了工具调用就执行，没有就结束」的循环，这一段是 Agent。只调一次模型、输出直接给用户，两种都算不上。OpenAI 的 Agent 构建指南说，接了模型但不用它来控制流程执行的应用，比如简单聊天机器人、单轮调用、情感分类器，不算 agent。一个系统里可以几种都有，按段说。

这份指南里 workflow 一词的用法和 Anthropic 不同，指的是为了达成用户目标要执行的那串步骤，agent 是用模型来管这串步骤怎么走的系统。面试里对方说的 Workflow 是哪个意思，拿不准可以先确认。

这题只讲路径由谁定。Agent 这个词怎么定义在第 21 题，Agent 内部是每步现定还是先列计划在第 14 题，多 Agent 怎么拆在第 3 题，入口怎么分流在第 17 题，LangGraph 怎么把流程画成图、拖拽式平台搭的流程算哪一种在第 10 题。

## 三、怎么选

美团一场 AI 后端一面和淘天一场 AI 一面，都把区别和适用场景放在一句里问：两者有什么区别，分别适合哪些场景？

先问能不能更简单。Anthropic 那篇文章建议找尽可能简单的方案，只在需要时加复杂度，这可能意味着根本不做 agentic 系统。它的理由是这类系统常常拿延迟和成本去换任务表现，而很多应用把单次模型调用配上检索和示例优化好，通常就够了。

确实需要更复杂时，它的说法是：定义清楚的任务，Workflow 带来可预测性和一致性；需要大规模的灵活性、要由模型来做决策时，Agent 更合适。Agent 可以用在开放式的问题上，也就是很难或者没法预测要走多少步、没法把路径写死的问题，代价是成本更高，错误可能累积。

OpenAI 的指南列了三类适合做成 agent 的流程：要做复杂决策的，里面有细微的判断、例外和依赖上下文的决定，举的例子是客服流程里的退款审批；规则集又多又绕、改一次成本高或者容易出错的；重度依赖非结构化数据的，要读自然语言、从文档里提取含义、和用户对话。指南接着说，动手之前先确认用例能清楚地满足这些标准，否则确定性的方案可能就够了。

合成一张表，是我的归纳：

![图像](<Agent%20高频题%2023.%20Workflow%20和%20Agent%20怎么选：哪些步骤写死，哪些交给模型.assets/image-04.jpg>)

退款这件事拆开看，两边都占。7 天内没拆封这一类，步骤列得出来，量大，用户在线等，适合写死。「左耳有杂音」「少发了充电线」这一类，说法各不相同，要读聊天记录和照片，适合交给模型判断。钱退出去撤不回，不管前面是谁判的，退款这一步都留在代码里。

同一位候选人在两场面试里被从两头问过。小鹏汽车一场端侧 Agent 一面先问，同一个问题多次运行，工具调用的先后顺序会不会不一样？接着问，为什么不用固定代码的 Workflow，而是用 prompt 描述流程？美团一场 Agent 一面先问，两个 QA 同时输入相同的查询，Agent 给出的报告是否一致？接着问，整体流程用的是 Workflow，为什么这个场景适合 Workflow？

两场都是先问结果稳不稳定，再问为什么这么选。答的时候把「一致」说细。Workflow 能保证的是步骤和顺序不变。分支由模型选的地方、节点里由模型生成的文字，两次仍然可能不一样。业务要的如果是两个人查同一件事拿到同样的结论，固定步骤再加上结构化的结论字段更容易做到。只要求最后答案对、不在乎中间怎么查的，可以接受 Agent 每次路径不同，用评测盯结果（第 4 题）。

蚂蚁一场 AI Agent 开发一面问，项目为什么采用 Agent Teams 架构，相比 Workflow 编排有什么优势？下一问是，线上要求全流程自动化、不依赖人工审核，怎么保证系统稳定、结果可靠？选了更自主的一边，接下来要回答的就是靠什么兜住。

## 四、两样怎么拼，「受控」在哪

字节一场全栈二面先问：你说 Agent 应该自主判断、决策，实际又用固定的 Workflow 编排，这是怎么考虑的？是否希望整个流程的骨架和功能点固定下来？隔了几问又问：冲突处理、失败处理这些链路，是提前用 Workflow 编排好的，还是部分交给大模型判断，只在关键节点做约束？

拼法有两种，阿里优酷一场二面的一问把两种都点到了：Workflow 是由程序直接触发，还是封装成工具交给 Agent 调用？

第一种，外层是 Workflow，某个节点里放一个 Agent。退款可以这么写：

```python
def handle_refund(ticket):
    order = get_order(ticket.user_id, ticket.order_id)   # 代码：每张单都查，顺带核对订单归属
    verdict = refund_agent.run(ticket, order)            # 模型：读申请、查政策、缺材料就追问
    verdict = validate(verdict)                          # 代码：格式或证据不合格，改成转人工
    if verdict.decision == "approve" and order.amount <= 500:   # 代码：金额门
        issue_refund(order.id)                           # 代码：退款
        return reply(ticket, verdict)
    if verdict.decision == "reject":
        return reply(ticket, verdict)
    return to_human(ticket, verdict)                     # 大额、存疑和其他情况都转人工
```

追问要等用户回话，流程得能停下来再接上，中断和恢复在第 10 题第三节。

第二种，外层是 Agent，把写死的流程当成一个工具。客服 Agent 和用户聊着，判断出这是退款，就调 start_refund(order_id, reason)，工具内部是上面那段代码。Agent 只决定调不调、参数填什么。可以这样分：入口是表单、工单类型已知的，用第一种；入口是自由对话的，用第二种。

哪些固定、哪些交给模型，千问那场问的就是这句。按退款的例子，我的分法是：

- 固定的：顺序本身就是规定的步骤，比如先核对订单归属，再谈退不退；写操作和它的前置条件，比如退款和金额门；便宜、每张单都要做的事，比如查订单；对模型交回来的结论做的校验。
- 交给模型的：读懂用户说了什么；缺什么材料、该怎么问；政策里哪一条适用；回复怎么写。

Anthropic 2026 年 10 月有一篇讲把子 Agent 并回主循环的文章，分法相近：顺序和领域步骤写进 Skill，硬限制（校验、审批门）放进工具；便宜而且几乎每次都用得上的加载，仍然可以用代码做。千问那一问的前半句是模型幻觉怎么解决，走错了链路怎么强控在第 19 题。

回到 Token Foundry 那一问，受控在哪里。落到 refund_agent 身上有五处：

```yaml
# refund_agent 的限制（示意，不对应某个框架的配置项）
tools: [get_refund_policy, ask_user]   # 查政策加追问，没有 issue_refund
max_steps: 8                   # 步数上限
max_total_tokens: 30000        # 花费上限
on_limit: escalate             # 到了上限转人工
output:                        # 只能交回这个结构
  decision: approve | reject | escalate
  reason: string
  evidence: [policy_clause, photo_id]
```

1. 给它哪些工具。它拿到的是查政策和追问两个工具，发起退款的工具不在表里。LangGraph 的文档在讲 Agent 比 Workflow 更自主的那一段里也写了，仍然可以限定可用的工具集和行为准则。OpenAI 的指南建议给每个工具评低、中、高风险，依据是只读还是写、能不能撤回、需要什么账号权限、对钱有多大影响。
2. 最多跑多久。步数和花费有上限，到了就停，转人工。Anthropic 2024 年那篇说，加停止条件来保持控制是常见做法，比如最大迭代次数。2026 年那篇把轮数预算、成本预算和写操作的闸门都设在 harness 里，也就是包在模型外面、负责跑循环和管权限的那层程序（第 9 题）。循环怎么停在第 28 题。
3. 写操作谁放行。退款在 Agent 外面，由代码按金额决定直接办还是转人工。OpenAI 的指南说，敏感、不可逆或者利害大的动作应当触发人工监督，直到对 Agent 的可靠性更有把握，举的例子里有批准大额退款。
4. 交回什么。只收固定结构的结论，代码校验字段和证据，不合格的改成转人工。Anthropic 讲提示链时提到可以在中间步骤加程序化的检查，这里是同一个做法。结构化输出怎么保证在第 22 题。
5. 之后能往哪走。外层只认通过、拒绝、转人工三种结论，每种后面接什么是写死的，别的值一律转人工。

![图像](<Agent%20高频题%2023.%20Workflow%20和%20Agent%20怎么选：哪些步骤写死，哪些交给模型.assets/image-05.jpg>)

这五处都由代码执行。提示词里写一句「超过 15 天不要退」也是约束，但它是说明，模型多数时候照做，合规场景里不够（第 7 题第六节）。被问到受控在哪，先讲代码执行的这几处，再讲提示词里写了什么。

## 五、Workflow 改成 Agent 加 Skill，什么时候再写回去

这个方向只有 3 位作者。百度一场 Agent 后端三面问：原来的 Workflow 是什么样，后来为什么改成 Agent 和 Skill？同一位候选人在快手一场 AI 全栈一面被要求把 Workflow 和 Skill 的优缺点各说一遍，在度小满一场 AI 全栈二面被问，两种方式哪一种回答更准确？另外两位，无锡赫玛科技一场面试问的是，为什么后期取消了独立的分类器，改成把各个分类沉淀为独立的 Skill 动态挂载；帆软一场后端二面问的是，从 Workflow 改造成 LangGraph 中心调度 Agent，思路是什么。

为什么改。设想退款的 Workflow 上线半年，质量问题、少发配件、签收人不对、过了 7 天但物流延误，每种例外一条分支，改一条要把别的分支重测一遍。这是 OpenAI 指南里规则难维护的那一类。改法是把退款怎么判写成一份 Skill，政策条款、要哪些证据、什么情况转人工都写在里面，模型按情况走。Anthropic 2026 年那篇文章提到，他们常见到的方案里有意图分类器和固定的多步推理流水线，得到的反馈往往是，这些结构大多在替模型做它已经不需要别人代劳的事。无锡赫玛那一问里去掉分类器的做法，第 17 题第七节讲过。

收益和代价在第 7 题第四节列过：改流程只改文档、不发版，模型能跳过用不上的步骤，一份 Skill 可以复用；代价是不保证顺序，延迟和 token 比固定链路高，得有评测才知道改完是变好还是变坏。

度小满那一问，哪种更准确，要拿数据答。同一批历史工单两边各跑一遍，常规单和例外单分开统计：常规单看 Agent 加 Skill 有没有比原来差，例外单看它接住了多少、又新错了多少。没跑过这个对比，就说没跑过。百度那场还追问，如果多花约一分钟能把准确率提高 20 到 30 个百分点，质量和性能怎么权衡？可以按单子分：用户在线等的常规单走快的路径；本来就要人工复核的疑难单，多等一分钟换准确率。

什么时候再写回去。阿里优酷那场二面问过：为什么现阶段还需要较多人工交互？如果流程已经稳定，是不是该进一步提高自动化程度？这一问说的是少让人参与。流程稳定以后，同样的话也可以对着模型问：这几步还需要它每次现定吗？

翻 Agent 的运行记录。某一类单子每次都是同样的工具顺序、没有追问，比如 7 天内没拆封的，这条路径可以写成代码：每一步少一次模型调用，延迟和成本降下来，步骤也能复现。做法是在入口加一个判断，命中的走写死的路径，其余的留给 Agent，这又回到了路由（第 17 题）。除了路径反复一样，还有几个信号：这类输入量大，省下的延迟和成本算得出来；出过一次不该发生的写操作；合规要求每步留痕。这一段是我的判断，手头几份资料没有给过固化的标准。

![图像](<Agent%20高频题%2023.%20Workflow%20和%20Agent%20怎么选：哪些步骤写死，哪些交给模型.assets/image-06.jpg>)

## 六、Skill、Workflow、Agent 是什么关系

字节一场后端一面问，Skill、Workflow、Agent 三者的联系和区别是什么？懂车帝一场 AI Agent 一面把 Tool 也加进来，问四样东西各自承担什么职责、有什么区别。

第 7 题第四节的答法是三个词不在一层：系统是 Agent，因为有循环在跑；Skill 是它的说明书；Workflow 是其中某几段写死的链路。放到退款的例子里，Agent 是读消息、调工具、看结果的那个循环，Skill 是退款怎么判的那份说明，Workflow 是从校验结论到退款的那段代码，Tool 是 get_order 这样的单个动作。字节电商那场问系统到底属于哪一种，可以按这个答：三样都有，各在哪一段。

懂车帝那场接着问：什么情况下更适合用 Skill，什么情况下需要设计 Workflow？有哪些业务场景必须依赖 Workflow 才能实现？Skill 和 Workflow 写的都是一件事按什么步骤做，差别在谁来执行这些步骤。Skill 由模型读了照着做，能跳过用不上的步骤，能应付没写到的情况，也可能漏掉一步。Workflow 由代码执行，一步不多，一步不少。步骤是经验和建议、允许看情况调整的，写成 Skill。顺序本身是规定、漏一步要赔钱或者要追责的，写成 Workflow，比如退款前的校验和审批、每步都要留痕的合规流程。

字节后端那场还问，项目为什么要用一个 Agent，而不是直接写一个 Skill 嵌进去？Skill 自己不会跑，要有一个循环来读它、用它，第 7 题第四节展开过。

## 七、面试怎么答

**两分钟主线**

1. Workflow 和 Agent 的区别在下一步由谁定。Workflow 的路径写在代码里，模型在留好的节点上干活。Agent 是一个循环，调哪个工具、调几次、什么时候停，由模型看着上一步的结果定。
2. 选之前先问能不能更简单，一次模型调用加检索够用，就两样都不上。步骤列得出来、错一步代价高、要快、要逐步留痕的，写成 Workflow。步数说不准、例外多、输入是自然语言的，交给 Agent，同时接受更高的延迟和成本，以及错误会累积。
3. 落地时两样可以拼。拿退款说，查订单、金额门、退款写死，缺什么材料、适用哪条政策交给模型。可以在流程里放一个 Agent 节点，也可以让 Agent 把写死的流程当工具调。
4. 受控要说到具体位置：给它哪些工具，最多跑几步、花多少，写操作谁放行，交回什么格式，外层只认哪几种结论。这几处由代码执行，提示词里的要求另算。
5. 把 Workflow 改成 Agent 加 Skill，要讲得出原来卡在哪，比如分支多到改不动，并且拿同一批样本两边都跑过。Agent 跑稳以后，重复出现的路径可以再写回代码，省延迟和成本。
6. Skill、Workflow、Agent 不在一层：Agent 是在跑的循环，Skill 是给它看的说明，Workflow 是其中写死的几段。

**准备自己的项目**

画一张自己系统的流程图，把每个决定下一步的位置标出来：这里是代码定的还是模型定的，为什么。再准备一次流程没覆盖到的输入，讲当时是谁接住的，是加了分支、转了人工，还是模型自己处理了，后来有没有因此改设计。做过 Workflow 改 Agent 或者反过来的，带上改造前后在同一批样本上的对比，准确率、延迟、单次成本各是多少，以及改完以后新出现的一类错误。没测过的数就说没测。

**容易答错的地方**

- 用了大模型就说是 Agent，或者把流程固定说成落后。
- 说 Workflow 的输出每次都一样。步骤和顺序不变，模型选的分支和生成的文字仍然会变。
- 被问受控在哪，只答提示词里写了限制。
- 讲改造只讲 Agent 更灵活，不提延迟、成本和评测。
- 把 Skill、Workflow、Agent 当成三选一。

![图像](<Agent%20高频题%2023.%20Workflow%20和%20Agent%20怎么选：哪些步骤写死，哪些交给模型.assets/image-07.jpg>)

## 出处

面经按发帖人记述转述。原题单和本次复核的样本范围、收录条件有差别。

- 两者的区别：[淘天 AI 一面](https://www.nowcoder.com/feed/main/detail/8e5fb9efdc154b9e913a2c8bde2a2a0b) · [B 站 Agent 二面](https://www.nowcoder.com/discuss/919630524673458176) · [小红书 Agent 开发二面](https://www.nowcoder.com/feed/main/detail/9f7361c709f4413396988b4f334a0d6f) · [名创优品 Agent 开发实习一面](https://www.nowcoder.com/feed/main/detail/f844a4ac20be44bc9b3f756bd0ebb84c) · [美团 AI 后端一面](https://www.nowcoder.com/discuss/927579574932705280) · [字节抖音电商客户端一面](https://www.nowcoder.com/feed/main/detail/9d9259986b304b749bc8ab07c20365c9) · [字节电商日常一面（中国交易与广告）](https://www.nowcoder.com/feed/main/detail/45e715fbedcb4c1c93a968647823214c) · [小红书 Product Engineer 一面](https://www.nowcoder.com/feed/main/detail/70ad5ead2f784676b0b399eab877d3d4) · [美团 Agent 一面](https://www.nowcoder.com/feed/main/detail/18fa4134edb14e8c901787ee5cd33ace) · [字节业务中台 AI 全栈日常实习](https://www.nowcoder.com/feed/main/detail/6ac5ba31705a4b64a407b087f39d5447) · [字节 AI Agent 开发一面](https://www.nowcoder.com/feed/main/detail/9cd459d87a754e07a490ba7f1c03b495) · [网易 Agent 开发一面](https://www.nowcoder.com/feed/main/detail/bad72b1323274ad5b172039f48bfa4b1)。
- 选型和适用场景：[淘天 AI 一面](https://www.nowcoder.com/feed/main/detail/8e5fb9efdc154b9e913a2c8bde2a2a0b) · [虾皮 Agent 开发一面](https://www.nowcoder.com/feed/main/detail/79fe3fc8f8cb4c4d9beca478f4279e3a) · [小红书与百度合帖（百度部分）](https://www.nowcoder.com/feed/main/detail/e319aadc79a9479397a6661a7f5ca088) · [蚂蚁 AI Agent 开发一面](https://www.nowcoder.com/feed/main/detail/6490526c86124f92b79a993e171f7222) · [懂车帝 AI Agent 一面](https://www.nowcoder.com/feed/main/detail/69e209d77a4647b7896ef3fd835239b6) · [阿里 Token Foundry AI 应用研发二面](https://www.nowcoder.com/feed/main/detail/ed25d2f60ddc4436b0139a7c52e62a61) · [阿里优酷二面](https://www.nowcoder.com/feed/main/detail/13bdf306b95f4cc9b6fafeec2e74ad70) · [美团 AI 后端一面](https://www.nowcoder.com/discuss/927579574932705280) · [小鹏汽车端侧 Agent 一面](https://www.nowcoder.com/feed/main/detail/ac25d49b0692473c8f65654adda82b9b) · [千问 AI 研发一面](https://www.nowcoder.com/feed/main/detail/ebf0ec03e5ee4fa8a140cc8d247b1e20) · [经纬恒润 AI 应用开发一面](https://www.nowcoder.com/feed/main/detail/32d60ac358504e86aab2d634a15b996b) · [美团 Agent 一面](https://www.nowcoder.com/feed/main/detail/18fa4134edb14e8c901787ee5cd33ace) · [字节业务中台 AI 全栈日常实习](https://www.nowcoder.com/feed/main/detail/6ac5ba31705a4b64a407b087f39d5447) · [字节全栈二面](https://www.nowcoder.com/feed/main/detail/3cac2a5686f5478f849d7e64a5a2e124)。
- 哪些固定、哪些交给模型：[百度后端 Go/Agent 一面](https://www.nowcoder.com/discuss/922907945363349504) · [阿里国际 Agent 开发三面](https://www.nowcoder.com/feed/main/detail/747f07e71f4448bebdce6ada5de800cd) · [百度一面（9.1）](https://www.nowcoder.com/feed/main/detail/22468da481974784bb19116f5b0134c1) · [抖音电商后端开发一面](https://www.nowcoder.com/discuss/925065999378829312) · [阿里 Token Foundry AI 应用研发二面](https://www.nowcoder.com/feed/main/detail/ed25d2f60ddc4436b0139a7c52e62a61) · [阿里优酷二面](https://www.nowcoder.com/feed/main/detail/13bdf306b95f4cc9b6fafeec2e74ad70) · [千问 AI 研发一面](https://www.nowcoder.com/feed/main/detail/ebf0ec03e5ee4fa8a140cc8d247b1e20) · [字节抖音电商 Agent 开发一二三面（二面）](https://www.nowcoder.com/discuss/937314680270311424) · [百度前端面试](https://www.nowcoder.com/feed/main/detail/fc061fe55af84effb8014ed6c60db92d) · [字节全栈二面](https://www.nowcoder.com/feed/main/detail/3cac2a5686f5478f849d7e64a5a2e124)。
- 从 Workflow 改成 Agent 或 Skill：[快手 AI 全栈一面](https://www.nowcoder.com/feed/main/detail/a30242712e8d456c839ff4223470f491) · [百度后端一面](https://www.nowcoder.com/discuss/924730985210458112) · [快手 AI 全栈二面](https://www.nowcoder.com/discuss/926528416512315392) · [帆软后端二面](https://www.nowcoder.com/feed/main/detail/2eb0fa4d90a04b598d58d6d303de5010) · [美团 AI 后端二面](https://www.nowcoder.com/discuss/929087871779110912) · [百度 Agent 后端三面](https://www.nowcoder.com/discuss/931476137576845312) · [度小满 AI 全栈二面](https://www.nowcoder.com/discuss/931952631340077056) · [无锡赫玛科技](https://www.nowcoder.com/feed/main/detail/24a0ee8b5d2c4d65aaf5fc1e18bfcc23)。
- Skill、Workflow、Agent 的关系：[懂车帝 AI Agent 一面](https://www.nowcoder.com/feed/main/detail/69e209d77a4647b7896ef3fd835239b6) · [字节电商日常一面（中国交易与广告）](https://www.nowcoder.com/feed/main/detail/45e715fbedcb4c1c93a968647823214c) · [字节后端一面](https://www.nowcoder.com/feed/main/detail/6b68d99d9ff74e4f920d7527a8fb27c0)。

外部资料按用途分组。

- 两个词的定义、五种 Workflow 模式、什么时候先不上 Agent。[Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)（Anthropic，2024-12-19）· [Workflows and agents](https://docs.langchain.com/oss/python/langgraph/workflows-agents)（LangGraph 文档）。
- 哪类流程适合做成 agent，工具风险分级，什么动作要转人工。[A practical guide to building agents](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf)（OpenAI，PDF）。
- 把分类器、流水线和子 Agent 并回一个循环，顺序进 Skill、硬限制进工具。[Building agents: When to fold subagents into your main agent loop](https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop)（Anthropic，2026-10-22）。