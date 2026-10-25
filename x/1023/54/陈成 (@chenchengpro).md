---
author: "陈成"
handle: "@chenchengpro"
source: "https://x.com/chenchengpro/status/2113243170788278767"
date: 2026-10-22T12:16:48.000Z
type: tweet
---
# 陈成 (@chenchengpro)

读了 @poteto 的 pstack，记一些点。  
  
1. pstack 不是 56 个单独的 skill，而是一套方法论和工作流。先做验证 skill 和 feature map，让 agent 能自己跑起并操作应用。再用 /poteto-mode 按任务类型走固定流程。最后才轮到并行和自动合并。第一步没做好，后面多开 agent 只会更贵，不会更可信。她演讲里的分法是，验证管"对不对"，pstack 管"好不好"。  
  
2. 建验证的方法值得花时间。先把应用做成 CLI，命令能组合，破坏性操作带 --dry-run，输出 JSON，报错时告诉 agent 下一步怎么做。再建 feature map，每个功能写清从哪进、怎么用、CLI 怎么驱动、有哪些坑。她管这个叫"物化的记忆"。我准备先找个 demo 产品试一下，照着她的 Atlas 示例来。  
  
3. 技术栈影响生产力，这点我深有体会。那 agent 友好的技术栈是什么？我目前的答案是轻、快、可调试、可控制。Electron 和 Web 有 CDP，iOS 有模拟器。运行时越难调、越难控，agent 越难用。她说值得为此自建调试工具，甚至换栈。  
  
4. 补上下文的几个 skill 也值得一看。/teach 会调 /how 追运行机制，调 /why 去 git、PR 评论、Linear、Slack、Sentry 里查当初为什么这么写。/recall 从历史会话里捞上下文。还有 /in-your-own-words，让 agent 先用自己的话复述问题，理解偏了能早点发现。  
  
5. 她反对对抽象计划做对抗式评审。原因是 agent 会臆想风险，给不会发生的边界情况加防护。她的做法是先做两三个可切换的原型，用验证 skill 截图测量。然后让几个模型家族各出一版类型签名级的设计，互相评判，挑一个再实现。我之前一直用对抗式评审，确实能想到很多 edge case，但也有不少是多余的。下次试试拿原型换掉一部分评审，看会漏掉多少。  
  
6. 56 个 skill 里有 55 个禁了模型调用，Claude Code 的 Skill 工具会直接拒绝。镜像版建了一张映射表，让 agent 去读对应的 SKILL.md 照着做。能用，但 skill 之间每调一次都多一步，也多吃上下文。这方案比很多产品直接给你塞一堆模型可调用的 skill 好要多了。  
  
7. 她视频里讲的外循环不在 pstack 里。Grok Bot 接了 Slack、Datadog、Sentry，订阅 Slack 线程和 Sentry 告警，自动派云 agent 去复现 bug、开 PR。她一个月 2,500 个 PR，主要靠这一层和验证，pstack 只占一小块。这层得自己搭。  
  
8. 把反复纠正变成结构的四级顺序也值得一看。用架构或数据结构根除 > lint 或测试让 CI 拦 > skill 或规则 > 人工 review。第四级她直接标成 ngmi。她的 Dune 框架就是第一级的例子。每件事只留一种写法，看到坏模式先写 lint 止血，再让 agent 清理，让代码库一直保持"agent 照抄也不会错"。我打算挑一个 agent 老犯的错，用 /correct 试一次。  
  
详细 research 见 [research.sorrycc.dev/@sorrycc/pstack](https://research.sorrycc.dev/@sorrycc/pstack) 。