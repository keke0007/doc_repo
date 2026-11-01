---
author: "govin.eth | G哥"
handle: "@goan999999"
source: "https://x.com/goan999999/status/2113927960650776889"
date: 2026-10-24T09:37:55.000Z
type: tweet
likes: 173
reposts: 38
replies: 18
bookmarks: 309
views: 20485
---
# govin.eth | G哥 (@goan999999)

用了这么久 Codex，我终于把它摸透了！  
  
同样是用 Codex，有人一天烧掉几百万 Token，代码还全是 Bug  
  
有人却能让它自己查资料、写代码、修 Bug、跑测试，甚至同时指挥多个 Agent 干活！  
  
刚开始用 Codex，我也没少踩坑  
  
最崩溃的就是项目做到一半，上下文突然乱了，修一个 Bug，又把之前的功能改坏了  
  
好不容易调通，开个新会话，还得重新解释需求  
  
后来才发现，Codex 最大的成本不是 Token 贵，而是反复试错、重复开发和无效输出！  
  
折腾这么久，我终于摸索出一套固定工作流  
  
8 个实操经验，建议收藏，尤其是第 6、7、8 条！  
  
1、给 Codex 装个「脑子」，告别失忆  
  
以前一个项目聊几十轮，需求、Bug、废弃方案全混在一起，越聊越乱  
  
现在每完成一个阶段，我就让它在项目根目录更新 DV_STATE.md  
  
保存项目目标、技术方案、已完成任务、遗留问题和下一步计划  
  
新开会话直接复制：  
  
「读取 DV_STATE.md，结合当前代码恢复项目上下文，确认已完成任务和遗留问题，不要重复之前的工作，继续开发」  
  
不用每次把几万字聊天记录重新喂给它，长期项目尤其好用  
  
2、用 AGENTS.md 立规矩，别让它乱改代码  
  
有时候只想改个按钮，Codex 却把整个项目扫描一遍  
  
最后只改三行代码，Token 倒是烧了不少  
  
现在我会在 AGENTS.md 写清楚技术栈、项目结构、编码规范和禁止修改的文件  
  
然后告诉它：  
  
「先读取 AGENTS.md，只分析当前任务相关文件，优先复用已有代码，不要扫描无关目录，也不要擅自修改其他功能」  
  
既减少无关文件读取，也能降低误改其他模块的概率  
  
3、先去 GitHub 找轮子，别急着从零开发  
  
以前接到需求就让 Codex 开始写，后来才发现，很多功能 GitHub 上早就有人做好了  
  
现在我会先让它搜项目，再决定要不要自己开发  
  
提示词：  
  
「先不要写代码，根据需求搜索 GitHub，找出 5 个最相关的开源项目，对比功能、Star、维护情况和许可证，优先复用成熟方案，最后给出实施计划」  
  
做爬虫、管理后台、AI Agent、自动化工具，都可以先这么干  
  
别人已经做好的东西，没必要再浪费几个小时重写  
  
4、Codex + Context7，解决过时 API 问题  
  
最烦的就是代码看着没毛病，一运行接口不存在！  
  
查半天才发现，Codex 用的是旧版本 API  
  
后来我接入了 Context7，让它优先检索对应版本的技术文档  
  
GitHub：[github.com/upstash/contex…](https://github.com/upstash/context7)  
  
安装提示词：  
  
「阅读 [github.com/upstash/contex…](https://github.com/upstash/context7) ，按照官方文档安装并配置 Context7，验证 MCP 是否正常运行，编程时优先查询对应版本的官方文档」  
  
尤其是经常用新框架和 SDK 的兄弟，建议装上  
  
5、Codex + RTK，减少无效 Token  
  
以前跑一次测试，终端输出几百行日志  
  
明明只有两个报错，却把一堆无关信息全塞进上下文  
  
RTK 就是专门解决这类问题的  
  
GitHub：[github.com/rtk-ai/rtk](https://github.com/rtk-ai/rtk)  
  
可以精简常见 Git、测试、构建命令的输出，减少冗余内容  
  
直接复制：  
  
「阅读 [github.com/rtk-ai/rtk](https://github.com/rtk-ai/rtk) ，按照官方文档安装配置，验证是否生效，并对比优化前后的 Token 消耗」  
  
对于经常跑测试、查日志的项目，确实实用！  
  
6、Codex + oh-my-codex，让多个 Agent 帮你干活  
  
以前一个 Agent 负责前端、后端、数据库、测试  
  
写着写着，上下文越来越乱  
  
后来我开始让不同 Agent 分工，独立任务并行处理  
  
GitHub：[github.com/Yeachan-Heo/oh…](https://github.com/Yeachan-Heo/oh-my-codex)  
  
支持任务规划、多 Agent 协作、代码审查和质量验证  
  
安装提示词：  
  
「阅读 [github.com/Yeachan-Heo/oh…](https://github.com/Yeachan-Heo/oh-my-codex) ，检查环境兼容性，按照官方文档完成安装配置，验证多 Agent 协作和任务规划功能」  
  
复杂项目可以拆分任务，简单任务没必要强行并行  
  
7、Codex + ECC，配一套完整开发流程  
  
这项目我真的建议研究一下！  
  
GitHub：[github.com/affaan-m/ECC](https://github.com/affaan-m/ECC)  
  
它提供 Agents、Skills、规则和开发工作流  
  
最有价值的不是工具数量，而是能把整个开发流程规范起来  
  
需求分析 → 任务规划 → 编码 → 测试 → 审查 → 验证  
  
以前 Codex 写完就说完成了，结果一运行全是 Bug  
  
现在至少可以通过规则要求它完成测试和验证后再交付  
  
提示词：  
  
「阅读 [github.com/affaan-m/ECC](https://github.com/affaan-m/ECC) ，检查 Codex 环境兼容性，安装适用于 Codex 的配置，验证核心功能，并说明已启用的 Agents 和 Skills」  
  
8、让 Codex 自己找 Bug，别再手动截图了  
  
以前最烦的就是：  
  
Codex 写完 → 我运行 → 发现 Bug → 截图发给它 → 它修复 → 我再运行  
  
有时候来回折腾十几轮！  
  
现在我会直接加一条交付规则：  
  
「完成任务后，检查 Git diff，运行相关测试、类型检查和构建命令，发现错误先分析根因再修复，最后汇报实际修改内容、验证结果和遗留问题，未执行的测试必须说明，不允许把未经验证的功能标记为已完成」  
  
这条建议所有人保存下来，真的能少掉不少重复操作！

> ![Link card preview](<govin.eth%20G哥%20(@goan999999).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - upstash/context7: Context7 Platform -- Up-to-date docs for AI agents**](https://t.co/DQ28lGJw3V)
> 
> _From github.com_