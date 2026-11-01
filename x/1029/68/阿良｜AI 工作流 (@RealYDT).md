---
author: "阿良｜AI 工作流"
handle: "@RealYDT"
source: "https://x.com/RealYDT/status/2114382577721319622"
date: 2026-10-25T15:44:24.000Z
type: tweet
---
# 阿良｜AI 工作流 (@RealYDT)

Claude Code的advisor是真的，但那条刷屏帖说过头了  
  
这两天有条帖子在传，说Anthropic放出了Claude Code最好用的技巧之一，一行 claude --model sonnet --advisor opus，Sonnet写代码，Haiku并行翻文件，Opus只在三个关键时刻出来把关。  
  
我对着官方文档挨条核了一遍，功能本身没问题，问题出在后面那套分工。  
  
先说对的。--advisor是官方参数，只是没写进claude --help，会话里打 /advisor opus 也能开，桌面版一样能用。Sonnet主跑、Opus当顾问也是官方推荐的搭配，顾问能看到整段对话，每次工具调用和结果都在里面。  
  
再说过头的。Opus什么时候介入，文档写得很清楚，是模型自己决定的，原话是model-driven rather than rule-based，计划前、同一个错出现两次、收工前，这些只是它的倾向，也没有开关能强制。  
  
Haiku并行探索更是跟这个参数没关系，内置的Explore子代理默认跟主会话用同一个模型，想让它跑在Haiku上，得自己定义一个叫Explore的子代理，写上model: haiku。  
  
更绝的是结尾那段JEV路由，Anthropic文档里压根没有，Claude Code也没有把工具选择、重试交给外部模型的接口，那条提示词的第4步照着粘进去是落不了地的。  
  
帖子还漏了一笔成本，顾问每次被叫都会把整段对话从头读一遍，这部分不走缓存，会话越长每次越贵。子代理还会继承顾问设置，你以为在用便宜的Haiku，它也可能转头去叫Opus。  
  
这就像给实习生配了个导师，导师确实有用，但什么时候敲门是实习生自己说了算，而且导师每次进门都要把整本项目日志重新翻一遍。  
  
所以写代码的长任务，Sonnet加Opus顾问值得一试；要是每一步都要强模型判断，比如写稿、做决策，直接让Opus当主模型反而省心。  
  
这种一行命令的神技巧恐怕还会越来越多，粘进去之前先翻一眼官方文档。  
行，功能我认，分工那段别信。

> ![Link card preview](<阿良｜AI%20工作流%20(@RealYDT).assets/image-01.jpg>)
> 
> 🔗 [**Escalate hard decisions with the advisor tool - Claude Code Docs**](https://t.co/Hq94vgKi7S)
> 
> _From code.claude.com_