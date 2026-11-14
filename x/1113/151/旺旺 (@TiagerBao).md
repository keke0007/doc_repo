---
author: "旺旺"
handle: "@TiagerBao"
source: "https://x.com/TiagerBao/status/2121481428500418958"
date: 2026-11-14T05:52:42.000Z
type: tweet
---
# 旺旺 (@TiagerBao)

AI 写代码最容易埋的坑之一，就是数据库表设计错了，等上线后才发现改不动。  
  
发现一个 Claude Code 专用 Agent：Database Architect。  
  
它不只是帮你生成 SQL，而是提前参与数据库架构设计。  
  
主要能做这些事：  
数据建模：表结构、主外键、索引、约束  
技术选型：PostgreSQL、MySQL、MongoDB、向量数据库  
多租户设计：RLS、Schema 隔离、独立数据库  
性能规划：读写模式、分库分片、扩展方案  
数据迁移：生成迁移脚本和回滚方案  
  
安装命令：  
npx claude-code-templates@latest --agent database/database-architect  
  
我觉得最实用的是让 Agent 先分析业务访问模式，再设计表结构，而不是代码写完才补数据库。  
  
不过生成的 DDL 和迁移脚本仍需要人工审核，尤其是生产环境的数据变更。  
你现在用 AI 开发项目，会让它直接设计数据库，还是自己先把表结构定好？  
  
项目：[aitmpl.com/component/agen…](https://aitmpl.com/component/agents/database/database-architect)  
  
#ClaudeCode #AI #Agent #Database

> ![Link card preview](<旺旺%20(@TiagerBao).assets/image-01.png>)
> 
> 🔗 [**Database Architect — Agents for Claude Code | Claude Code Templates**](https://t.co/xrVQkEQ6iX)
> 
> _From aitmpl.com_