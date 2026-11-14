---
author: "victor-wu.eth"
handle: "@victor_wu"
source: "https://x.com/victor_wu/status/2120119142846001583"
date: 2026-11-10T11:39:28.000Z
type: tweet
---
# victor-wu.eth (@victor_wu)

最近把 agent 使用心力降低了 80%，终于不用被 agent 一直拖着走了。  
  
1. 给 agent 配一个 leader。基于 herdr core，你只和 leader 对话，leader 负责任务的调度、思考和传话。干活丢给 worker agent 干活。和 deepseek harness 认为 leader agent 只是个传话 agent 不同，leader agent 依然需要负责【翻译人类的需求】，以及对 worker agent 进行判断给出决策，必然不可以是一个低智能模型。  
  
另外实践过程中发现，最好列个表或者直接告知 worker 其他 worker 是负责什么的，避免 leader agent 沦为传声筒（worker 互相通信这点有点早先 cc agent team 的感觉）。当然你用 matt 的 skill 或者在 herdr 自建一个 mod 也行，主要是能时刻确保知道 worker 的状态和任务。这样也不用担心 leader agent 压缩，它是一个永不关闭的窗口（感谢 claude code tui 的自动压缩机制，一小时不用自动压缩）。此方法和 claude code 的 project 模式类似，只是借由 herdr core 可以用在任何模型上。  
  
2. 以结果为导向，通过控制论进行自动优化。说人话就是你给 agent 一个可以量化和确认的目标，让 agent 基于【假设-执行-验证-修改变量】的 loop 方式来执行。我个人喜欢卡帕斯的 auto-research 的方法来驱动，不过我在此基础上会加入 A/B test：先做出若干个版本，挑一到两个进行 auto-research 优化。  
  
那人在这里干嘛呢？作为 eval 的执行者，疯狂裁决裁决裁决。在 agent 先对 eval 裁决的基础上做出终审判决。并对 agent 的结果做出决策并解释缘由，整理成 ADR 用以日后参考。否则 agent 会不断地踏进同一条河流。  
  
3. 记得用 Claude code，伟大无须多言。我们用的是 opus 、fable ，你用的是 什么 sol 、astra，你不行就得怪模型。

![Image](<victor-wu.eth%20(@victor_wu).assets/image-01.png>)