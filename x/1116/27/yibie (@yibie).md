---
author: "yibie"
handle: "@yibie"
source: "https://x.com/yibie/status/2122307976321171900"
date: 2026-11-16T12:37:06.000Z
type: thread
---
# yibie (@yibie)

作者亲自解读 Pi Durable：怎么做到断点恢复的  
  
终端里的 Pi，崩了没关系，敲一句 continue 就行。但如果 Agent 跑在 Slack、内存受限的手机上、或部署到会被回收的 serverless 环境后，旁边就没人敲 continue 了。它得能被重新拉起，从断点恢复，还不能忘掉做过什么。  
  
Pi Durable 把会话里每个动作都表达成带阶段的任务：先记录接下来要做什么，执行这个 effect，把结果持久化，再进下一阶段。所有状态变化按顺序串行提交，提交后才算数；模型一次回复发起的多个工具调用各自成为任务，可以并行跑。  
  
它刻意不用事件溯源。常见 harness 把每个 token、每次工具调用都写成事件，几千条记录的编码会话够用，但一个要在 Slack 频道活五年、几千人参与的 bot 撑不住。  
  
Pi Durable 改存 JSON 文档：监听者先拿整份快照，之后只收 delta，文档带版本号、可迁移，演讲里说这是为跑十年的 agent 准备的。存储只是一个简单接口，SQLite、Postgres、Cloudflare Durable Object、JSONL 都能接。  
  
进程任意时刻被杀，重启后打开同一个存储说 resume，没完成的任务会被读出来续跑。这里有个讲究：bash 这种不可重放的工具不会再执行，而是在转录里标记成因进程消失被中断，把结果交给模型让它收尾；只有搜索这种天然幂等、或带幂等键的工具才会重放。用户输入也带幂等键，Slack 把同一次 mention 重复投递时不会触发两次运行。  
  
工具和运行的机器分开：read、write、edit、bash 只依赖访问文件系统和起进程两个接口，同一套工具能跑笔记本、安卓 Termux、QEMU 沙箱或 Cloudflare，运行中还能热插拔。一个 harness 配置一次就能并行承载多个会话，Slack 子线程可从主会话某点 fork。  
  
两个现场演示挺说明问题。一是放进 Cloudflare Durable Object，它跑满 15 分钟就被回收，设个闹钟到期前重启，resume 后像什么都没发生。二是整套 harness 跑在他的安卓手机上，现场 51 个人同时网页围观。代码在 GitHub 的 earendil 仓库里。  
  
原文链接在评论区  
#原理

![Image](<yibie%20(@yibie).assets/image-01.jpg>)

---

原文：

> ![Link card preview](<yibie%20(@yibie).assets/image-02.jpg>)
> 
> 🔗 **Pi Durable - Mario Zechner**
> 
> This is the recording of Mario's talk at our meetup on 2026-11-10. ...
> 
> _From youtube.com_