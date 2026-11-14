---
author: "铁锤人"
handle: "@lxfater"
source: "https://x.com/lxfater/status/2118943523915317709"
date: 2026-11-06T19:44:31.000Z
type: tweet
---
# 铁锤人 (@lxfater)

AI 做逆向，现在也有成套 Skill 了！！  
  
我整理了 10 个逆向方向的 Agent Skill 和集成方案，从拆安卓包、追前端签名、看二进制，再到分析产品功能，你可以直接按手上的目标来挑：  
  
1. REA：顺着应用行为往下查，把 JS、Electron、原生二进制等分析工具接进 Agent，给出证据和分析局限  
  
2. reverse-skill：先判断手上目标属于什么场景，再挑对应的分析方法、工具链和报告流程  
  
3. android-reverse-engineering-skill：反编译安卓安装包，还原混淆的 Kotlin 类名，把接口、网络调用和代码结构理出来  
  
4. replica-skill：从公开页面和操作流程分析产品，再接上架构设计、独立开发、测试和差异对比  
  
5. leanrea：不启动 MCP 服务，只用一个 8 KB 的轻量 Skill 指挥 AI 直接调用本机逆向工具  
  
6. ctf-skills：按逆向、二进制、密码、取证等 CTF 题型，配好对应的解题思路和工具依赖  
  
7. reverse-skills：配合反编译结果分析函数符号、重建数据结构，还能接上动态插桩和模拟调试  
  
8. hello_js_reverse_skill：串起前端 JS 混淆分析、签名追踪、动态 Cookie 归因、浏览器取证和对照验证  
  
9. Open-ReverseLab：把多平台逆向知识库、上百个 MCP 工具调用和证据报告放进同一个工作区  
  
10. js-reverse-automation--skill：连接浏览器 MCP、JSRPC、Flask 和 Burp，跑通前端 JS 签名定位和自动化分析流程  
  
这么多套摆在一起，是不是不知道先装哪个？  
  
其实不用一次全装，研究产品功能先看 replica-skill，看安卓接口看 android-reverse-engineering-skill，本机已经有工具、想省掉复杂配置直接调命令行就看 leanrea  
  
人人都可以逆向的时代来了

![Image](<铁锤人%20(@lxfater).assets/image-01.jpg>)

> **铁锤人 (@lxfater)**
> 
> 你还在用 AI 猛猛写代码，别人已经在用 AI 自动逆向你的代码和产品了！！  
>   
> 现在大家用 Claude Code、Codex 一天搓几个站，你以为自己项目小，黑客就看不上？  
>   
> 但现在逆向和找漏洞早就不靠人手搓了，AI 能自己翻代码找隐患，还能自己写程序验证攻击  
>   
> 在 CyberGym Level 1 真实软件代码库评测里，@GCSAHK x.com/171476994/stat…
> 
> ![Image](<铁锤人%20(@lxfater).assets/image-02.jpg>)
> ![Image](<铁锤人%20(@lxfater).assets/image-03.jpg>)