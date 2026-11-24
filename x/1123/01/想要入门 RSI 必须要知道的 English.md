---
author: "Chasen"
handle: "@chasen_liao"
source: "https://x.com/chasen_liao/status/2124712191152763101"
date: 2026-11-23T03:50:36.000Z
type: article
likes: 166
reposts: 23
replies: 14
bookmarks: 182
views: 18181
---
# 想要入门 RSI 必须要知道的 English

*By Chasen (@chasen_liao)*

![Banner](<想要入门%20RSI%20必须要知道的%20English.assets/image-01.jpg>)

最近要开始学习 RSI 了，但是 AI 领域就是这样，上来就是一大堆英文，看晕我了😂 我收集了一波 ，给大家用大白话一口气讲清楚来 👇 

先把这十个词认全，后面的分组会反复用到它们的区别：**RSI**、**Harness**、**Agent**、**Self-modification**、**Recursion**、**Trace** / **Trajectory**、**Evaluator**、**Benchmark** / **Held-out**、**Overfitting**、**Snapshot** / **Rollback**

下面按七组过一遍，每条我尽量说清它和旁边那个词差在哪

## 基础概念

**RSI**（Recursive Self-Improvement），总概念，系统用自己的输出和评测结果改进自己，并让新版本成为下一轮的起点

它不要求更新模型权重，改 prompt、改 skill、改工具或者改整个运行框架都算，只要形成改完再测、测完再改的循环，就是系统层的 RSI

失败之后换一种说法再试一遍，那是重试，还不是递归自我改进

**Self-improvement**、**Self-modification**、**Reflection**、**Recursion**，四个词挨着，但不在同一层

Self-improvement 说的是结果，表现变好了，不一定动过自身结构

Self-modification 更具体，指改了自己的指令、代码、工具或流程

Reflection 只是事后总结，本身不等于改进，总结如果没有写成可复用的改动、也没有经过验收，就还停在反思

Recursion 在这里不是函数自己调自己那么窄，它指的是改进后的系统会作为下一轮改进的起点，没有把新版本接回去再跑，就谈不上递归

**Model**、**Agent**、**Harness**、**Runtime**、**Scaffolding**，这一组要分清谁是模型、谁是外围

Model 是模型本身，负责生成，权重可以冻结

Agent 是被组织起来去完成任务的行动者，通常包括模型、指令、工具和一个循环

Harness 是套在外面的运行系统，管怎么调用工具、怎么管理上下文、怎么验收、怎么进入下一轮

Runtime 偏实现，指实际跑起来的那个运行时

Scaffolding 常和 Harness 近义，强调搭在模型外面的支架代码，STOP 里改进的就是这类支架，不是模型权重

**Inference**、**Training**、**Fine-tuning**、**RL**，区别在权重动没动

Inference 用已有模型做推理，不改权重，Training 和 Fine-tuning 用数据更新权重，RL（强化学习）按奖励信号调整策略，常见做法也是改权重

系统层 RSI 大多发生在 inference 外围，模型可以不动，改的是 Harness，所以看到一个方案说「训练」，先分清它是在训模型，还是在优化 prompt 和 skill

## Harness 组件

**Prompt**、**Skill**、**Tool**、**Hook**，这是 Harness 里最常被改的四类对象

Prompt 是当下给模型的指令或上下文，Skill 是可复用的专项能力说明，比如一份技能文档，Tool 是模型可调用的外部能力，Hook 是在特定时机自动插入的逻辑，例如任务开始前做检查、结束后记日志

有些系统会冻住模型，只进化这四类对象，再用 traces、benchmark、scoreboard、snapshots 留底，PenguinHarness 是这个思路的一个例子

**Memory**、**Context** 容易混成同一件事，Context 是这一轮模型实际看到的窗口内容，Memory 是跨任务保存下来、需要时再读入 context 的信息

上下文满了不等于有记忆，有记忆也不等于每次都会被用上

**Trace**、**Trajectory**、**State**，这一组讲的是系统留下了什么

Trace 是能被保存下来的执行记录，包括工具调用、返回结果、写出来的步骤和最终答案，它不代表模型内部的思考，也不是内心独白

Trajectory 常和 trace 混用，更强调按时间排列的整条行动轨迹

State 是某一时刻系统的配置和进度，含当前 skill、当前工具和任务状态

没有可靠的 trace，外层优化器就只能盲改

**Policy** 这个词有两套用法，不要混

传统强化学习里，policy 是「在某状态下采取某行动」的策略，常常对应模型或控制器的参数；在 Agent 指令的语境里，它也可能只是一套规则或工作方式，比如先搜索再下结论

读到这个词，先看它改的是权重，还是文本规则

**Token**、**Budget**、**Latency**，前一个是计费和窗口单位，第二个是成本上限，可以是 token、钱或者调用次数，第三个是时延

评测时这三项最好和正确率一起看，否则一个版本分数更高、但又慢又贵，不一定算改进

这几项上涨属于成本问题，和 reward hacking 是两回事

## 改进循环

**Optimizer**、**Proposer**、**Evaluator** 是三个角色，Optimizer 是推动改进的外层机制，Proposer 根据轨迹提出改动方案，Evaluator 负责打分和验收

好的设计会让提议和验收分开，避免自己出题、自己改答案、自己打高分

**Candidate**、**Mutation**、**Selection**、**Archive** 是一条流水线，Candidate 是待测的新版本，Mutation 是在旧版本上做的改动，Selection 按分数和约束决定留下谁、淘汰谁，Archive 是保留下来的版本库或者经验库，供以后比较、再组合

snapshot 和 scoreboard 就是在给这几样东西留底

**Inner loop**、**Outer loop**、**Meta loop** 是三层，Inner loop 是 Agent 在做任务本身，Outer loop 是在任务之外改进 Agent 或 Harness，Meta loop 更外一层，有时指连怎么提出下一轮改进的方式也一起改

先分清内环做任务、外环改 skill，不必一上来就改「如何改进」本身

**Objective**、**Metric**、**Fitness** 说的是要什么和怎么量，Objective 是到底要优化什么，Metric 是具体怎么量，Fitness 借自进化计算，常等于综合适应度分数

目标含糊时，系统很容易只去优化一个好刷的数字

**Snapshot**、**Rollback**、**Sandbox** 是三项刹车，Snapshot 是某个版本的完整快照，Rollback 是效果不好时退回上一版，Sandbox 是隔离执行环境，限制它能改什么、能碰到什么评测脚本

系统层自我改进要能安全地做下去，基本靠这三项

## 评测泛化

**Benchmark** 不只是一套题目，它还包括任务本身、运行环境，以及怎么打分的协议，换了环境或者换了评分办法，分数就不能直接拿来比

**Train** 用来提出和筛选改动，**Validation** 用来调选择标准，防止只盯着训练集

**Held-out** 是改进过程中尽量不看、最后才用的保留集，如果它在多轮改进里被反复拿来挑选版本，就会被污染，不再适合当最后的独立验收集，所以选型、调标准用 train 和 validation，held-out 尽量留到最后看一次

**OOD**（out-of-distribution）是和训练分布明显不同的样本，用来看系统是不是只会做见过的题

用来改技能的题，和最终验收的题，必须分开

**Overfitting** 是在可见评测上涨分，换一批类似任务就失效，说明可能只是在背题；**Leakage** 是验收信息不该出现却漏进了改进过程，比如 held-out 题目、标准答案或者评测脚本被 Agent 读到

分数突然很好时，先查这两项

**Baseline** 是比较起点，通常是改之前的版本；**Ablation** 是去掉某个改动再测，用来看它到底有没有贡献；**Regression** 是回归，指某些原来能做好的任务变差了

只报平均分、不看回归，很容易把这边涨、那边塌误当成全面进步

## 成本安全

**Reward hacking** 指系统没有真正变好，只是钻了评分规则的空子，例如输出更长、格式更像答案，或者专门讨好打分模型

它和费用上涨不是一回事，token、耗时、调用次数变多是成本问题，质量是另一条线，两条要分开记

**Evaluator tampering** 比钻规则更进一步，改评测脚本、放宽标准、删掉难题，甚至改分数记录，所以验收脚本和 held-out 数据要放在它写不到的地方

前面的 **Sandbox**、**Snapshot**、**Rollback** 在这里正好接上，隔离决定它能碰什么，快照决定能不能退回去，成本和质量分开记录，才知道一次改动值不值

## 算法速查

这几个是不同方法，不要互相当成别名，有的只改 prompt，有的改支架代码，有的重点在防过拟合，各用一两句分清楚就够

- **STOP**（Self-Taught Optimizer），让会调用模型的支架程序去改进「改进器」自身，模型权重不变，作者也写明这还不是完整意义上的 RSI
- **DGM**（Darwin Gödel Machine），Agent 修改自己的工具和执行代码，再用编程基准筛选能留下来的版本
- **GEPA**（Genetic-Pareto），从执行轨迹里做自然语言反思，用遗传式搜索和 Pareto 前沿来进化 prompt，不改模型权重
- **OPRO**（Optimization by PROmpting），把优化过程写成 prompt，让模型根据历史分数直接提出更好的指令
- **APE**（Automatic Prompt Engineer），自动生成一批候选指令，再按评测选出更好的 prompt
- **ACE**（Agentic Context Engineering），从训练轨迹里提炼可复用规则，写进 Agent 每次任务前会阅读的 playbook
- **RRSI**（Regularized Recursive Self-Improvement），在自我改进上加正则和约束，重点防过拟合、防信息泄漏，也防复杂度和成本失控

最容易混的几条收在这里，RSI 不等于必须改模型权重，重试和反思不等于递归自我改进，policy 有强化学习和文本规则两套含义，上面这些算法各走各的路，STOP 的作者自己也说那还不是完整的 RSI

## 用 Pi 改一个 skill 的假设例子

下面是假设例子，不是已经跑过的结果，只用来把流程走一遍

先把一件事说清楚，把活派给 Pi 或 GAP，只是委派执行，这本身不是 RSI，这里要练的是「用评测决定是否保留对 skill 的改动」这件事，属于系统层自我改进的流程

1. 只让 Pi 改进某一份 skill，而不是改整个 Pi
2. 先写清 objective 和 metric，例如在一组固定任务上提高正确率，同时 token 和耗时不明显变差，这是举例口径，不是成绩
3. 把题目分成 train 和 held-out，前者给外环看，用来提出 mutation，后者只给独立 evaluator 用，也不要一轮轮拿 held-out 去选版本
4. 内环是 Pi 按当前 skill 做事，并留下 traces，也就是执行记录，不是内部思考
5. 外环的 proposer 阅读这些轨迹，生成一份 candidate skill
6. 先在 train 上和 baseline 比，必要时做 ablation，看涨分来自哪一处改动
7. 通过之后再在它改不动的 sandbox 里跑 held-out，评测脚本、分数记录和保留集都放在 Pi 写不到的位置，防 leakage 和 evaluator tampering
8. 如果 held-out 也更好，并且没有明显 regression，就打一份 snapshot，再启用新 skill
9. 如果只是训练集涨分、保留集没动，就当成 overfitting；如果只是更贵、更慢，就当成成本问题，不要写成 reward hacking；这些情况都 rollback 回旧版