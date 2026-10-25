---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2112775410228630000"
date: 2026-10-21T05:18:05.000Z
type: tweet
---
# 山景城小路 (@Zen_with_AI)

Anthropic 面试内部题库大揭秘：反复考的 5 道编程题，与最容易挂的那一轮  
  
一、流程总览：8 轮，两天 onsite，第二天取决于第一天  
  
1. Recruiter call  
2. CodeSignal OA：90 分钟，分级（levels）递进式题目  
3. Live screen：55 分钟；可查文档、可搜网页  
4. Onsite 第一天：coding + system design  
5. Onsite 第二天：hiring manager、project deep dive、values 轮——第一天表现弱，第二天可能直接取消  
  
关键规则：  
8 轮里有 4 轮要写代码或做设计，默认全部禁用 AI；电面可以查文档和搜网。  
2026 年的 loop 新增一轮「与 Claude Code 一起写代码并做 code review」，那一轮反而要求用 AI。  
  
二、高频 5 题  
  
共同特征：全是「工作中真会写的组件」，不是 LeetCode 式算法题。电面里基础题只占约 1/3 时间，剩下全是 follow-up：规模变大怎么办、IO-bound 还是 CPU-bound、怎么并行。默认 Python 手写、无 AI。  
  
1. Fix a function-cache key function, then make it durable（修函数缓存的 key 函数，再做持久化）。  
考什么：  
缓存 key 的正确性（参数规范化、哈希碰撞、不可哈希参数）+ 持久化方案（序列化、落盘、恢复、并发写）  
  
2. Concurrent same-host web crawler（同主机并发爬虫）  
考什么：BFS + 线程池/协程、URL 去重、礼貌限速；与题库里已有的「LC webcrawler 变体」互相印证  
  
3. Image transform pipeline with a wall-clock budget（带时间预算的图片变换pipeline）  
考什么：  
按 JSON recipe 批量处理图片×变换组合；本题库已有公开题面概述（3 parts / 55 min），深挖方向是大数据量与昂贵变换下的扩展性重设计  
  
4. Deduplicate files by size then content hash（先按大小、再按内容哈希去重）  
考什么：  
两级过滤降成本：大小分组 → 头部采样哈希 → 全量哈希；大文件与哈希碰撞的处理  
  
5. Reconstruct call stacks from enter and exit traces（由 enter/exit 事件重建调用栈）  
  
考什么：  
由 profiler 采样栈 diff 出 start/end 事件——找公共前缀、先结束旧帧再开启新帧，注意嵌套顺序  
  
三、次高频 3 题  
  
1. Banking system with transfers, cashback and account merges（转账、返现、账户合并的银行系统）  
考什么：面向对象 + 状态一致性：转账原子性、幂等、合并账户时的余额与返现规则  
  
2. Distributed mode and median across worker shards（跨分片求众数与中位数）  
考什么  
分布式聚合：众数用计数合并，中位数需直方图/分桶或近似算法；考的是不能把数据全拉回本地  
  
3. Greedy longest-match tokenizer（贪心最长匹配分词器）  
考什么：  
Trie/前缀匹配、词表构建、最长匹配的边界情况  
  
四、设计轮：29 题里 21 题是 AI 系统 / ML  
时长 55 分钟；可能要求把设计打字写进共享 Google Doc，而不是画图。  
评分点：API 设计、数据模型、请求流、如何扛住题目给定的负载、至少一个 failure case。  
典型题目：GPU 上的批量 LLM 推理 API、LLM 推理服务平台、分布式 AI 模型下载器、AI prompt playground。  
  
五、最容易挂的一轮没有代码：Values 轮（60 分钟）  
可能有两位面试官：一人提问、一人 shadow 观察。  
考的是你对 AI 社会影响与 Anthropic 使命的真实看法、处理伦理冲突的方式、以及会不会因为好的理由改变立场。  
追问会往下挖好几层，背好的故事很快露馅。Recruiter 可能在 onsite 前发 AI 安全阅读材料——要读，并形成自己的观点（包括你不认同的部分）。  
  
Project deep dive（第二天，60 分钟）：自己主导的项目讲约 20 分钟，剩下时间被深挖。项目要现在就选好，砍成一个经得起 40 分钟追问的故事。  
  
六、一个背景信号：为什么面试这样设计  
到 2026 年 8 月，公司内部每 1 小时员工工时对应 6 小时的交互式 AI agent 工时；hiring manager 对技术候选人 AI 工具使用的记录自 2024 年以来涨了 3 倍。原文的解读：编程轮查的是你脱离辅助能不能构建，对话轮查的是在 agent 帮你打字的岗位上，你带来的判断力。  
  
七、备考清单（对应本报告的 5 条建议）  
先练 5 个组件：纯 Python、无 AI 写出来，再练次高频 3 题。  
每个组件再练并发版：能说清 IO-bound / CPU-bound、输入放大 10 倍时改什么。  
设计题用文档练：API、数据模型、一个 failure case，全部用文字写出来；重点押 AI 基础设施方向。  
Values 轮当技术轮准备：读 Anthropic 公开的 AI 安全文章，定好自己同意与不同意的点，能扛多层追问。  
Deep dive 项目提前定：20 分钟讲完、40 分钟深挖的标准。