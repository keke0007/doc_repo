---
author: "旺旺"
handle: "@TiagerBao"
source: "https://x.com/TiagerBao/status/2120041783568875653"
date: 2026-11-10T06:32:04.000Z
type: tweet
---
# 旺旺 (@TiagerBao)

Coding Agent 最浪费 Token 的地方，可能根本不是写代码。  
  
最近读了一份《Jev Engineering for Coding Agents》，整理了 TypeSafe 创始人 Diogo Almeida 对 Agent 架构的思考。  
  
其中几个观点值得关注：  
  
写代码只占约 4%～10% 的 Token，更多消耗在读取、搜索和命令输出上  
小模型不一定省钱，模型切换后的上下文重建可能更贵  
MCP 工具不该一次性加载全部 Schema  
Compaction 不该盲目压缩，而应按当前任务提取上下文  
AGENTS.md 应按条件加载，避免无关规则长期占用上下文  
子 Agent 应共享检索结果，减少重复读取  
  
核心思路：上下文应该按任务动态组装，而不是让聊天记录无限累积。  
  
这是一份第三方汇编的设计笔记，不是已经落地的完整方案，但很值得做 Agent 的开发者参考。  
  
原文 PDF：  
[github.com/yibie/jev-engi…](https://github.com/yibie/jev-engineering-zh/blob/main/assets/original.pdf)  
  
你们觉得 Coding Agent 下一阶段最大的优化空间，是模型能力，还是上下文管理和任务调度？  
  
#Agent #ClaudeCode #Codex #AI

> ![Link card preview](<旺旺%20(@TiagerBao).assets/image-01.jpg>)
> 
> 🔗 [**jev-engineering-zh/assets/original.pdf at main · yibie/jev-engineering-zh**](https://t.co/tGs1s7tC48)
> 
> _From github.com_