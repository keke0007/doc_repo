---
author: "Larry  Orbi"
handle: "@xqliu"
source: "https://x.com/xqliu/status/2120900927573860367"
date: 2026-11-12T15:26:00.000Z
type: thread
likes: 28
reposts: 1
replies: 4
bookmarks: 14
views: 1360
---
# Larry  Orbi (@xqliu)

你是不是也想让 agent 记住项目里那些事：某个决定当初为什么这么定、哪条路试过走不通、服务部署在哪？  
  
有人 11 月 8 号开了个仓库 Brain4Agent，专门给 agent 做长期记忆，中日韩文优先。才四天就有 142 星。  
  
记忆全存在一个 SQLite 文件里，按会话取回。几处设计：  
1中文按相邻两个字切开建索引，不用词典，也不用分词模型，装上就能搜中文  
2检索核心是 BM25，不调模型，也不要 API key。作者说每次查询只要几毫秒  
3搜完不给置信度，只告诉你问题里哪些词整个库里从没出现过、哪个会话缺了哪些词，算不算命中由你判断  
  
第三条最实在。光给一个分数，agent 很难判断这条记忆该不该信；直接告诉它「这几个词从来没记过」，它就有了实打实的依据。  
  
要 Python 3.11 以上：  
  
pip install "Brain4Agent[precise]"  
  
这个版本会多装一个 jieba，作者测的首条命中率是 0.950。不想加依赖就去掉 [precise]，首条命中率是 0.875。

---

Brain4Agent 仓库，README 有中日韩文档和跑分

> ![Link card preview](<Larry%20Orbi%20(@xqliu).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - SparkofSpike/Brain4Agent: Committed to building the world’s most cutting-edge, CJK-first...**](https://t.co/m33qsyAGdS)
> 
> _From github.com_

---

让 agent 记住项目里的事，我在 Orbi 里是放进 Issue。在 Orbi Cloud 的「写需求」页，用对话把要做的事讲清楚，Orbi 先读一遍仓库再起草 Issue。不清楚的地方它会停下来问，你确认后才提交。

> ![Link card preview](<Larry%20Orbi%20(@xqliu).assets/image-02.jpg>)
> 
> 🔗 [**Orbi — 软件黑灯工厂，开源（AGPL-3.0）：提个 Issue，收个版本**](https://t.co/wUl2wKatWW)
> 
> _From orbi.build_