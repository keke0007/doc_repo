---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2115791511354827183"
date: 2026-10-29T13:03:00.000Z
type: tweet
---
# 山景城小路 (@Zen_with_AI)

小路带你看大厂架构｜Netflix 每天 10 万次失败，AI 怎么帮工程师少翻日志  
  
Netflix 数据平台每天跑数百万个 workload（Titus、Spark、Trino、Druid），高峰时一天约 10 万次失败。旧的 Pensive 靠正则加手写 Java 规则，能让约一半失败自助化，但三个月约 700 万次分类里，约一半还是落入 unknown，工程师照样得翻日志。  
  
新一代 Pensive AI 的思路是：AI 只做「错误提取」和「按需调试」，包在确定性流程外面，不直接当裁判。最漂亮的一招是输出格式：日志先逐行编号，模型只返回错误片段的起止行号，也就是两个整数，正文由系统从自己的日志副本重建。省 token、低延迟，还少幻觉。  
  
硬约束同样真实：必须在 Maestro 90 秒的后处理超时内返回，于是逐日志并行、共享 deadline、迟到直接剔除。如今 AI 提取已覆盖 100% 的失败，调试结果按 step attempt 缓存，条件写抢占、90 天过期，SSE 再转 WebSocket 流式推送。  
  
评估也值得学：旧规则冻结下来当标签和回归底线，日志归档到 S3 解决 7 到 30 天过期，Repeated noise 指标把出现超 30% 的行列入排除清单，实际只有 3 行。观测数据说，没用 Pensive AI 的失败尝试，手动翻日志动作多 321%，不过这是观测性数据，不能当成因果结论。  
  
LLM 嵌进流水线的正确姿势，是把它限定成一个输出格式极窄的提取器。

> ![Link card preview](<山景城小路%20(@Zen_with_AI).assets/image-01.jpg>)
> 
> 🔗 [**Making Failure Diagnosis Reliable with Pensive AI**](https://t.co/ZkfuLL91AU)
> 
> _From netflixtechblog.com_