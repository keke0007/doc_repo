---
author: "meng shao"
handle: "@shao__meng"
source: "https://x.com/shao__meng/status/2126252963383627969"
date: 2026-09-03T01:58:07.000Z
type: tweet
likes: 36
reposts: 10
replies: 10
bookmarks: 56
views: 3286
---
# meng shao (@shao__meng)

如何评估 AI 智能体：斯坦福大学 CS329Z 第 12 讲，SWE-bench 共同创建者 John Yang 拆解基准设计与好基准的七项标准  
  
CS329Z: Engineering AI Agents 第 12 讲里，SWE-bench 共同创建者、斯坦福计算机科学博士生 John Yang 把问题收回到一句很基本的话：前面已经讲了怎么搭智能体、怎么用数据去训练，可“变好了”到底是相对什么而言？视频发布在 Stanford Online 频道，专门谈评估与基准设计。  
  
视频地址：  
[youtube.com/watch?v=_iVIug…](https://youtube.com/watch?v=_iVIugwpiUI)  
CS329Z 课程网站  
[cs329z.stanford.edu](https://cs329z.stanford.edu)  
  
# 基准是公共语言，也容易被过度优化  
  
John 先提醒：基准是比较的公共地面，也拉动研究方向，并出现在实验室的模型卡里。考试型基准从 SQuAD、MMLU 一路加难，评测怎么跑也会改分数——允不允许模型先写一长段推理，结果往往不一样，所以作者不但要出题，还要把可辩护的运行说明写清楚。伯克利 SkyLab 的 Chatbot Arena（后来发展为 LM Arena）用盲测对战和 Elo 计分，模型表现取决于对手有多强，不像固定考题那样容易被做满。可一旦分数被当成营销数字，就可能出现他所说的 benchmaxing：模型在 SWE-bench Verified 上可以很高，你跟它打招呼，它却去查 GitHub issue；也有模型给代码套上大量 try-except，把错误藏起来，测试能跑过，却不是人们真正想用的模型。  
  
# 智能体基准要同时写清任务和评测协议  
  
智能体基准要求模型和环境多轮交互、调用工具。John 把它分成两半。  
  
一半是任务表述：一开始给模型什么问题，环境里有哪些资产，最后用什么方式验收。问题可以很封闭，比如早期 WebShop 要求买到一双 30 美元以内的红色篮球鞋，满足约束就得 1 分；也可以像 SWE-bench 只陈述 issue，并不指定改哪一行；再到 CodeClash 或售货机经营，往往没有唯一正确答案，比的是相对表现。环境可以是终端、Docker、真实网站或模拟站点，还要考虑可复现和别人能不能跑起来。验收可以是成败二分，也可以是连续分数。哈佛与 Google 的 Jeffrey Ma 等人做的 SWE-Efficiency，任务是把代码加速；若人类参考方案是 50 倍、模型做到 20 倍，既可以规定必须达到 50 倍才算过，也可以把 20/50 记成 0.4。  
  
另一半是评测协议：温度、轮次还是费用上限、用哪套 harness、允许尝试几次、智能体如何声明自己做完。这些看起来像半页后勤，却决定了分数能不能并排比较。SWE-bench 刚发表时仍是用 BM25 检索再让模型改文件的工作流，还称不上智能体基准；直到 SWE-agent、Devin、OpenHands 把模型放进终端，它才变成智能体评测。同一模型配不同 harness，分数也会变。有的基准要求必须用指定脚手架，例如比较精简的 mini-SWE-agent；SWE-bench 和 Terminal-Bench 的排行榜则允许提交不同 harness 的结果。设计者没有唯一正确答案，但必须把选择讲清楚。  
  
# 好基准的七项标准，以及数据污染只能被推迟  
  
他给出的好基准清单有七项：足够难、分得出模型高下、贴近真实、容易跑、容易验收、结果可解释，而且尽量经得起时间。几项之间常有张力。把 200 个 GitHub issue 塞进同一个环境，确实更难，却未必更自然。在 WebArena、Sierra 的 τ-bench、Terminal-Bench、港大 Tao Yu 课题组的 OSWorld，以及他自己参与的 CodeClash 里，环境越做越完整，验证也可以从单元测试改成对照数据库终态，或改成两套代码对打。可一旦任务偏离真实工作，分数再好看也难让人在意。CodeClash 用编程游戏对打很有观赏性，他自己也反问：人们为什么要关心模型能不能写出更好的 BattleSnake？SWE-bench 头四五个月并不受欢迎，一个原因就是验收太重：数百个实例、每个再跑几十上百个测试，又不方便并行。基准说到底是给人用的，跑不起来就没人接着往上做。  
  
关于训练数据污染，他提到三条常见做法：SWE-bench Pro 保留私有测试集，由主办方代跑；DeepSWE 请人新写任务，发布时相对干净；SWE-bench Live、LiveCodeBench 则定期纳入模型训练集截止之后的新 issue。这些办法都能推迟污染，并不能一劳永逸。

![Image](<meng%20shao%20(@shao__meng).assets/image-01.jpg>)

> **meng shao (@shao__meng)**
> 
> CS 329Z: Engineering AI Agents  
> Stanford / Fall 2026 @stanfordnlp  
> https://cs329z.stanford.edu  
>   
> 课程定位：从"模型"到"系统"的工程学  
> 覆盖：简单 LLM 流水线 → 复合 AI 系统 → 自主 Agent。三位讲师的背景也高度互补：  
> @Diyi_Yang（斯坦福 NLP 教授，人机交互与社会计算方向）
> 
> ![Image](<meng%20shao%20(@shao__meng).assets/image-02.jpg>)