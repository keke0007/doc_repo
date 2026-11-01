---
author: "porco"
handle: "@SeanchenZ"
source: "https://x.com/SeanchenZ/status/2115319389448966260"
date: 2026-10-28T05:46:57.000Z
type: thread
likes: 1
views: 92
---
# porco (@SeanchenZ)

SAP × 富士康：机器人终于开始知道自己为什么要搬东西了。  
  
10 月 27 日，SAP 发布了一项值得关注的 Physical AI 方案。  
  
富士康 Physical AI 负责人 Dr. Wang 提到：  
  
SAP EWM 中的仓储任务，可以直接编排人形机器人和自主运输系统；维护计划可以触发机器人巡检，执行结果经过确认后再回写 SAP。  
  
这件事让我挺有感触。  
  
我目前在富士康做自动化设备导入，平时接触最多的是 PLC、CCD、机器人控制器、设备联锁、生产节拍和异常处理。  
  
现场大部分自动化设备解决的是：  
  
设备能不能正确执行动作。  
  
比如传感器有没有信号、气缸有没有到位、机械臂有没有完成轨迹、物料有没有正确吸取。  
  
但还有一个问题：  
  
设备为什么现在必须执行这个动作？  
  
这批物料对应哪个订单？库存是否允许出库？任务优先级是谁决定的？执行失败以后，业务系统怎么知道？  
  
这些问题不是单靠机器人运动控制就能解决的。  
  
SAP 这次尝试打通的是一条完整的信息链路：  
  
ERP / WMS 业务任务  
↓  
任务编排与权限校验  
↓  
机器人 / 自动运输系统  
↓  
现场控制与安全联锁  
↓  
执行结果验证  
↓  
业务系统回写与审计  
  
这让我意识到，Physical AI 不只是给机器人接一个 VLA 模型，让它具备更强的视觉理解和动作规划能力。  
  
还有一条很重要的路线：让机器人执行的每一个动作，都能对应到真实的企业业务需求。  
  
当然，业务系统不能绕过 PLC、机器人控制器和独立安全系统。ERP 可以告诉机器人要搬什么、为什么搬，但现场控制系统仍然需要判断这个动作能否安全执行。  
  
目前 Intel 成都案例仍是 PoC，富士康案例也尚未披露足够的长期运行和 ROI 数据。  
  
所以我不认为这已经证明了 Physical AI 的大规模商业化。  
  
但方向值得关注。  
  
过去工业自动化主要解决机器如何正确执行动作。  
  
现在，企业开始尝试解决另一个问题：  
  
机器执行完动作以后，整个企业能不能知道它为什么做、做得怎么样，以及结果是否可信。  
  
机器人会搬东西是一回事。  
  
搬完以后，仓库账还对不对，是另一回事。

> ![Link card preview](<porco%20(@SeanchenZ).assets/image-01.jpg>)
> 
> 🔗 [**SAP Unveils Physical AI and Extends the Autonomous Enterprise into the Real World**](https://t.co/p5iVBX8qZy)
> 
> _From news.sap.com_

---

最近在研究，一个 AI Token 中转站到底是怎么搭建出来的？  
  
发现这东西的原理并不复杂，本质就是一个 AI API Gateway（API 网关）。  
  
它不是凭空生成 Token，而是把多个模型供应商的 API 接入统一管理。  
  
整个流程：  
  
用户请求 → 中转服务器 → API Key 验证 → 余额/额度检查 → 模型路由 → 上游 API → 返回结果 → Token 用量计费。  
  
如果想自己搭建，甚至不用从零开始写。  
  
GitHub 上已经有现成的开源项目：LiteLLM。  
  
技术栈可以这样选：  
  
LiteLLM：统一 API 接口、模型路由  
PostgreSQL：用户、密钥、账单数据  
Redis：缓存、限流  
Docker + Nginx：部署与 HTTPS  
虚拟 API Key：按用户分配权限和额度  
  
先跑通一个 MVP，就能实现多模型调用、用量统计、额度限制和故障切换。  
  
但如果想把它做成真正对外收费的中转站，难点就来了。  
  
上游 API 授权、计费准确性、并发控制、Key 泄露、滥用防护、服务商条款、用户数据隐私……  
  
尤其要区分：正规 API 网关和倒卖第三方账号 Session Token，根本不是一回事。  
  
转发一个 API 请求没什么技术含量。真正有技术含量的是：当有人刷爆额度、上游限流、服务异常时，你的系统还能不能守住成本和权限边界。  
  
开源项目：

> 🔗 [**GitHub - BerriAI/litellm: The fastest, litest AI Gateway. Rust core with Python SDK. Call 100+ LLM...**](https://t.co/AvkeHXu0u1)
> 
> _From github.com_