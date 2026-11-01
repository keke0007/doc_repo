---
author: "丁洪Liam"
handle: "@ding_liam"
source: "https://x.com/ding_liam/status/2114562663145443567"
date: 2026-10-25T15:58:12.000Z
type: tweet
---
# 丁洪Liam (@ding_liam)

Amp 的工程师 Thorsten Ball 分享的《How I work with Agents》，里面最反直觉的一条：  
别跟 Agent 来回试错，让它自己造迭代的工具。不是「试试这个……不行，再试那个」，而是「生成 20 个变体，再做个小页面让我组合测试」。  
  
对于非开发者用户来说，我挑了 7 条大家能直接照做的：  
  
1. 如果你钱包够厚，只用最新最强的模型。模型错 10% 的时候，你不知道是哪 10%，只能多试几次。  
  
2. 提示词写「要什么」，不写「怎么做」。他的例子：让字节码编译器和 VM 尽可能快，不到 2 倍不要停  
  
3. 让 Agent 看见整个研发流程：跑自己写的东西、读线上日志、部署、盯上线、发版、开 feature flag、分诊 bug、给客户发邮件。他的说法是，Agent 看不见的东西，它就会编  
  
4. 要「无可辩驳的证据」：端到端演示、录屏、前后对比数字。他说这是最大的改变，还建议在 AGENTS.md 里写上「this is not evidence」  
  
5. 别等 Agent，让 Agent 等你。给更大的任务，批量交代，少一来一回  
  
6. 定时、唤醒、心跳是杀手级功能：「2 小时后看日志」「上线了告诉我」「每天 9 点检查」  
  
7. 只 review 爆炸半径大的代码：不到 5 个核心文件，加上数据库迁移。变量名、重复代码他已经不管了，只看能不能跑、架构层面看不看得懂、Agent 有没有端到端测过  
  
从产品角度看，这 7 条其实指向同一件事：  
人从操作 Agent 的人变成验收的人。瓶颈不再是 Agent 写得快不快，而是人能不能说清楚什么算做完，以及拿到什么证据才算做好。

![Image](<丁洪Liam%20(@ding_liam).assets/image-01.jpg>)

> **Thorsten Ball (@thorstenball)**
> 
> ![Article cover](<丁洪Liam%20(@ding_liam).assets/image-02.jpg>)
> 
> 📝 **How I work with Agents**
> 
> ... as of October 2026.
> Use the latest and greatest models. Everything else is potentially a waste of time. If a model is wrong 10% of the time, you don't know which 10%, so you end up making multiple...
> 
> Finally summarized what I've been sending to friends in DMs