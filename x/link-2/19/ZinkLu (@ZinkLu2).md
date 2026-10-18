---
author: "ZinkLu"
handle: "@ZinkLu2"
source: "https://x.com/ZinkLu2/status/2059568352839045618"
date: 2026-05-27T08:21:42.000Z
type: tweet
likes: 172
reposts: 22
replies: 32
bookmarks: 154
views: 111097
---
# ZinkLu (@ZinkLu2)

昨天见了一个非常牛逼的Agent团队，我敢说在国内绝对是T-1的级别（之前某JS框架还找他们抄了点Agent模块）  
  
刚好聊到了这两天推上吵得非常热闹的AI产品（Agent）要不要用Python的话题  
  
他们Founder说的很直接：SB才在Agent项目里用TypeScript🤣  
  
Python适合100%Agent项目，主要有几个原因：  
  
第一，Agent最终大多时候会在后端里。  
  
不管你做的是工作流引擎、数据处理管道、模型微调、向量检索，还是离线批处理、定时任务、科学计算、爬虫、硬件控制，Python天然离这些更近。  
  
模型是Python，数据是Python，中间的tool schema、事件流、状态机都能共用同一份动态类型。  
  
如果你用TS那就会变成：  
  
模型服务在Py、数据处理在Py、后端在Py、前端在JS深坑  
一份schema要手写四次  
  
如果某个类型校验一旦松弛，你的Agent马上就给死给你看。  
  
第二，Agent很依赖稳定和简单逻辑。  
  
Agent不是边想边输出那么酷炫。  
  
它要稳定调用工具、处理异常、重试、限流、序列化、日志、测试。  
  
Python在这些场景里很顺：pytest、tenacity、asyncio、multiprocessing、pickle、shelve、logging。  
  
TS当然也能做，但你会更容易感受到「这东西本来不是为这类后端稳定性长出来的」。  
  
第三，动态类型对Agent很重要。  
  
Agent真正容易炸的地方不是「类型检查」，而是业务逻辑错、数据转换错、模型输出结构变、环境依赖炸。  
  
Python可以把很多东西轻松拾起来：  
  
tool input/output、agent state、message format、workflow node、permission object、external API response  
  
这对Agent很关键，因为Agent系统里有大量JSON对象需要动态处理，静态类型反而碍手碍脚。  
  
第四，Python更适合做「Agent runtime」。  
  
如果你做的是一个Agent框架、SDK、运行时、插件系统，Python优势更明显。  
  
因为使用者往往要把它接进：  
  
数据管道、AI推理服务、iOS/Android SDK、硬件设备、边缘盒、Gradio、Streamlit、Jupyter、MLops  
  
这些地方Python生态更统一。  
  
所以很多Agent infra选Python，不是因为TS不行，是因为它们要服务的使用场景更接近数据科学家和算法团队。  
  
第五，AI应用现在其实是拼工程。  
  
早期大家用TS，是因为AI=炫酷。  
  
现在很多AI产品已经演化到包含LLM API、tool calling、database、vector store、browser automation、workflow、UI、billing、auth、analytics  
  
这已经是研究工程了，是数据工程。  
  
数据工程的主语长期就是Python。  
  
很无聊，但世界就是这么没品😮‍💨  
  
但他也表示TS不会消失。  
  
更合理的分工其实是：  
  
TS做UI层、演示层、web端交互层、轻量脚手架、用户可见的玩具。  
  
Python做模型层、Agent编排层、数据层、eval、embedding pipeline、离线任务、实验脚本、产品核心逻辑。  
  
所以你如果做一个Agent产品，你最好：  
MVP后端+Agent orchestrator用Python。  
  
涉及前端交互、花哨UI、跨平台演示，再上TS。  
  
聊了一下午，真的学了太多了  
  
才知道自己之前对于Agent的认知到底有多浅🧎  
  
（PS：你想用啥语言就用啥语言，Claude 都会）

> **Max For AI (@MaxForAI)**
> 
> 昨天见了一个非常牛逼的Agent团队，我敢说在国内绝对是T0的级别（之前DPSK还找他们搞了点Agent数据）  
>   
> 刚好聊到了这两天推上吵得非常热闹的AI产品（Agent）要不要用Python的话题  
>   
> 他们Founder说的很直接：SB才在Agent项目里用Python🤣  
>   
> TS适合100%Agent项目，主要有几个原因：