---
author: "XiaoAnLab"
handle: "@xiaoanlab"
source: "https://x.com/xiaoanlab/status/2118963036484690305"
date: 2026-11-07T07:05:30.000Z
type: thread
---
# XiaoAnLab (@xiaoanlab)

每天处理一堆PDF和Word的你，查一条要翻3个文件夹，本地搭RAG知识库能让资料开口回答。  
  
它解决资料散、格式杂、问不到的问题。  
  
先解析成文本，再向量入库，最后检索生成带出处的答案。  
  
第 1 步 · 文档解析工具：[github.com/CatchTheTornad…](http://github.com/CatchTheTornado/text-extract-api) —— 把PDF、Word等解析成干净文本。  
  
第 2 步 · 向量数据库：[github.com/alibaba/zvec](http://github.com/alibaba/zvec) —— 把文本转成向量存入本地。  
  
第 3 步 · RAG 检索问答框架：[github.com/truefoundry/co…](http://github.com/truefoundry/cognita) —— 检索片段并生成带引用的答案。  
  
适合已有本地文档想离线问答的人 / 不适合只有少量笔记、想即开即用的人。  
  
先从一类文档试起，确认解析效果再扩到全部资料。  
  
你最想先让它回答哪类文档里的问题？  
  
#工具推荐 #开源项目

> ![Link card preview](<XiaoAnLab%20(@xiaoanlab).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - CatchTheTornado/text-extract-api: Document (PDF, Word, PPTX ...) extraction and parse API...**](https://t.co/3zY1eDBFIU)
> 
> _From github.com_

---

ggml 4 年后发了 v1.0.0，本地大模型跑得快不快，底子就在这儿。  
  
本地跑大模型先过张量运算这关：量化、SIMD、跨后端，自己写又重又难，还得兼顾不同硬件。  
  
ggml 就是干这个的——纯 C/C++、零依赖，x86、ARM、RISC-V 都能跑。1.0.0 砍了废弃 API，加了 FP8 和可配置 YaRN 截断，各后端都做了性能优化。  
  
适合自己搭本地推理、或把模型嵌进产品的人；不适合只想开箱聊天的人，那用 ollama 更快。它是个库，得自己编译集成。  
  
说白了，llama.cpp 的底子就是 ggml。先看 1.0.0 发布说明，再跑 examples/simple 矩阵乘例子。  
  
你本地跑模型用的是哪个工具？  
  
#开源 #大模型

> ![Link card preview](<XiaoAnLab%20(@xiaoanlab).assets/image-02.jpg>)
> 
> 🔗 [**GitHub - ggml-org/ggml: Tensor library for machine learning**](https://t.co/FII17rIhjn)
> 
> _From github.com_