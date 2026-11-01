---
author: "南晚Nanwan"
handle: "@snwiki238337"
source: "https://x.com/snwiki238337/status/2115358658447970489"
date: 2026-10-28T08:23:00.000Z
type: tweet
likes: 161
reposts: 23
replies: 13
bookmarks: 147
views: 8256
---
# 南晚Nanwan (@snwiki238337)

LangChain 工程团队在官方博客发的一篇关于 agent 工程构建的工程复盘可以看看，题为《How We Built Our Design Agent》。  
  
主要是最近他们在内部做设计智能体 Content Studio 的过程，他们把整套代码和设计底座在 GitHub 开源了。  
  
其实日常中各位有做用大模型做设计，应该是对 ai 做设计的审美有这认知的。  
  
Stripe 的设计总监把它叫做“僵尸界面（Zombie UI）”，也就是颜色、字体和排版挑不出大毛病，但放在一起千篇一律、极其平庸。  
  
LangChain 作为一家高频推产品的创业公司，内部设计资源长期严重吃紧，于是他们让设计团队和工程团队坐在一起，在 Slack 里搭了一个常驻的设计 Agent。  
  
据他们说在全公司面前频繁翻车，经过四个月的打磨，目前已经在内部交付了超过 3000 个真实设计物料，包括 80 多套完整的演讲 Keynote，75% 的提需求者是产品经理和程序员，中位数单次生成成本从 8 美元被压缩到了 0.5 美元。  
很多人好奇他们是怎么把所谓的设计品味教给大模型的，其实核心在于他们坚决不让 AI 直接生成拍死的不可编辑图片，而是把所有物料全部强制用代码来编写。  
  
简单来说的话其实就是给 Agent 配了一台真实设计师工具的虚拟电脑。  
  
他们最早也走过弯路，为了防止模型自由发挥搞砸，团队硬编码了 115 个现成的死板 UI 组件和密密麻麻的条规，结果模型完全失去了灵活性。  
  
直到后来换上更聪明的基座模型，他们才领悟了设计领域的苦涩教训。  
  
于是他们删掉了绝大部分琐碎限制，只保留高层的品牌规范和设计原则，转而在执行层给它装上专门的虚拟画布工具，让人类能直接在画面上打点圈选提意见，主 Agent 则是一个调度多线程渲染。  
  
他们的这个流程总结下来说的话其实就是给模型一套接近真实的执行沙箱，加上一个能保留长期审美的东西，AI 是可以做到在试错中逼近专业设计师的直觉和审美的。  
  
这一点也是模型在真实世界的模拟的体现。  
  
GitHub：

> ![Link card preview](<南晚Nanwan%20(@snwiki238337).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - langchain-ai/design-agent: Open-source design agent: on-brand stills, decks, diagrams,...**](https://t.co/yF9GFduHXb)
> 
> _From github.com_