---
author: "kayce"
handle: "@fttt007"
source: "https://x.com/fttt007/status/2119631060623581276"
date: 2026-11-09T03:20:00.000Z
type: tweet
likes: 51
reposts: 6
replies: 9
bookmarks: 43
views: 3854
---
# kayce (@fttt007)

学习 Pi Durable：Per-Conversation Agent。  
  
先理解三个核心概念：  
  
Session：整个会话运行环境，可以包含多个 Conversation。  
Conversation：一个独立的聊天对话，保存自己的对话记录。  
Agent：负责执行任务的 AI 助手，拥有自己的模型、工具、指令和工作目录。  
  
Pi 的设计很有意思：每个 Conversation 都可以拥有自己的 pi.agent 配置。  
  
例如，同一个 Session 中：  
Conversation A 是编程助手，可以读取文件、执行命令。  
Conversation B 是翻译助手，只负责翻译。  
Conversation C 是研究助手，拥有自己的模型和工具。  
  
它们互不干扰，各自工作。  
  
其中：  
configure()：修改当前 Conversation 的 Agent 配置。  
agent(context)：获取最终生效的 Agent 配置。  
未单独设置的配置，继续使用 Host 默认值。  
  
另外，还要区分 Provider Session ID：它是用于关联模型提供商侧会话的标识，不等于 Pi 自己的 Session。  
  
一句话总结：Session 管理整个会话环境，Conversation 管理独立对话，而 Per-Conversation Agent 让每个对话拥有自己的 AI 工作方式。  
  
这就是 Pi Durable 的核心设计思想之一。