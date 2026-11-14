---
author: "袋鼠帝"
handle: "@aikangarooking"
source: "https://x.com/aikangarooking/status/2119977223142899882"
date: 2026-11-09T09:05:07.000Z
type: tweet
---
# 袋鼠帝 (@aikangarooking)

卧槽，本地知识库的真神来了！SAG2让我看完浑身起鸡皮疙瘩。  
或许它将彻底颠覆RAG，因为建库阶段不调大模型、不做 Embedding、不要向量库、不要 GPU，只对原文建 BM25 关键词索引。同样 11,656 段文本，一代建库 4.57 小时，SAG2 只要 2.36 秒，快约 7000 倍。  
  
语义理解交给 JEV：BM25 先粗筛，JEV 判断哪些是有用证据，再从证据里挑出新关键词接着搜。线索靠原文里的关键词和段落串起来，不用提前抽实体。  
  
效果非常🐂🍺，适合任何规模数据的知识库：  
MuSiQue 多跳检索 Recall@5 达到 80.68%，比 HippoRAG 2 高 15.55 个百分点。  
51 万份文档的 EnterpriseRAG-Bench 上，LightRAG、HippoRAG 2 这些图谱方案没跑完全量，SAG2 建库 8 分 32 秒，Recall@10 达到 81.10%。  
  
落地工具 SAG Wiki：Windows 版约 13MB，macOS 版约 18MB。添加一个本地文件夹就是知识库，文件不用动，它全自动。  
[github.com/Zleap-AI/SAG-W…](https://github.com/Zleap-AI/SAG-Wiki)

> **袋鼠帝 (@aikangarooking)**
> 
> ![Article cover](<袋鼠帝%20(@aikangarooking).assets/image-01.jpg>)
> 
> 📝 **RAG这次真死了！SAG2才是本地知识库的神，又快又准又便宜～**
> 
> 大家好，我是一直在跟知识库打交道的袋鼠帝。
> 在 AI 圈，每天都有新东西冒出来，但已经好久没有一个东西，能让我看完浑身起鸡皮疙瘩了。...