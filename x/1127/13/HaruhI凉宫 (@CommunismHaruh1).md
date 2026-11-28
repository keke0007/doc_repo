---
author: "HaruhI凉宫"
handle: "@CommunismHaruh1"
source: "https://x.com/CommunismHaruh1/status/2126322823522357315"
date: 2026-11-27T14:30:40.000Z
type: thread
likes: 560
reposts: 75
replies: 53
bookmarks: 604
views: 22992
---
# HaruhI凉宫 (@CommunismHaruh1)

卧槽！！！终于让我搞出来了！！！  
  
继 OmniCAD 之后，我开发的第二款工程自动化 Skill——OmniSolidWorks V1.0 正式开源了！  
  
这几天是真的快把我累死了😭  
  
从最初的简单零件建模，到复杂机械装配，再到原生爆炸图和工程图，从 V0.1 一路迭代到 V0.9，最后又经历了三轮 RC 候选版的开发与验收。  
  
SolidWorks 崩溃、接口调用失败、模型重建异常、装配引用丢失……各种问题层出不穷，好在终于走到了今天。  
  
那么，这个 Skill 到底能干什么？  
  
· 原生参数化三维零件建模  
· 多级机械装配与配合  
· 原生爆炸视图与拆装步骤  
· 二维工程图、剖视图与 BOM  
· 实体干涉检测  
· 参数修改与模型重建  
· 工程规范校核  
· 自动验收与故障恢复  
  
与普通的 AI 建模方案不同，我并不希望 AI 仅仅生成一个看起来很漂亮的模型。  
  
所以这次依然延续了 OmniCAD 的核心设计理念——证据驱动（Evidence-Driven）。  
  
通过 SolidWorks 原生 API 读取实际几何数据，对模型体积、质心、装配关系、实体干涉及参数修改后的重建结果进行验证。  
  
甚至还会主动注入错误，测试系统能否识别问题并拒绝不合格的工程成果。  
  
目前已经完成的测试包括：  
  
· 108 个零件实例的多级机械装配  
· 6 个原生子装配  
· 49 个实例的减速器案例  
· 独立三工位夹爪机构建模  
· 24 项单元测试  
· 独立安装及原生文件重开验证  
  
开发过程中也参考了 SolidCraft 等优秀开源项目的技术思想，吸收其优点，并结合 SolidWorks 的实际运行环境进行独立实现。  
  
顺带一提，我们的国标制图 Skill 也已经取得了一些开发进展，后续会逐步融入整个 Omni 系列，这个以后再单独聊。  
  
当然，V1.0 并不意味着完美，部分高级爆炸功能、连续碰撞检测以及不同版本的兼容性仍需要进一步完善。  
  
实际验收记录和已知限制都已经整理在仓库中，没实现的东西咱也不瞎吹。  
  
说实话，作为一个土木工程专业的大二学生，跨到机械领域折腾这些东西，难度确实远超我的预期。  
  
但这段时间认识了很多专业人士，也收到了不少热心网友的建议和帮助，甚至还有朋友专门为我的项目制作了二创视频。  
  
真的非常感谢大家！！！  
  
从 OmniCAD 到 OmniSolidWorks，从二维工程制图到三维机械建模，距离我最初设想的多工程软件自动化融合，又近了一步。  
  
至于接下来的 OmniRevit……  
  
先让我歇歇吧，真的要累死了哈哈哈哈。  
  
项目现已采用 MIT 许可证开源，感兴趣的朋友可以免费拿去折腾！  
  
也欢迎各位机械行业的大佬参与测试、提出建议，帮助我继续完善这个项目。  
  
安装方式很简单  
👇

![Image](<HaruhI凉宫%20(@CommunismHaruh1).assets/image-01.jpg>)
![Image](<HaruhI凉宫%20(@CommunismHaruh1).assets/image-02.jpg>)
![Image](<HaruhI凉宫%20(@CommunismHaruh1).assets/image-03.jpg>)
![Image](<HaruhI凉宫%20(@CommunismHaruh1).assets/image-04.jpg>)

---

懒人安装方法：  
  
直接把下面的 GitHub 仓库链接发给 Codex，让它阅读 SKILL.md 和安装文档，检查本地环境并完成安装即可。  
  
🔗 [github.com/Haruhi764/Omni…](https://github.com/Haruhi764/OmniSolidWorks-Skill)  
  
目前实机验收环境为 SolidWorks 2014 x64 SP0，欢迎其他版本的用户参与兼容性测试！  
  
另外，为了方便大家交流和反馈问题，我也建立了 Omni 系列官方技术交流群。  
  
QQ群：313181842  
  
完全免费开源，欢迎 Star，也欢迎各位大佬参与后续开发！

> ![Link card preview](<HaruhI凉宫%20(@CommunismHaruh1).assets/image-05.jpg>)
> 
> 🔗 [**GitHub - Haruhi764/OmniSolidWorks-Skill: Evidence-driven native SolidWorks automation for paramet...**](https://t.co/3YGgB8L2cl)
> 
> _From github.com_

---

兄弟们，我是真的要累死了