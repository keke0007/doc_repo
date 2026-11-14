---
author: "头雁"
handle: "@alacheng"
source: "https://x.com/alacheng/status/2119738014926778535"
date: 2026-11-09T10:25:00.000Z
type: tweet
likes: 223
reposts: 42
replies: 19
bookmarks: 261
views: 12741
---
# 头雁 (@alacheng)

google deepmind 的大神 @i_amanchadha 的知识蒸馏入门blog  
  
核心内容先分别训练多个领域专家教师（比如数学专家、代码专家、工具使用专家、安全专家等），然后让学生模型自己生成轨迹，再让这些专家教师对学生实际走过的路径进行密集监督，最终把多个专家的能力整合进一个统一的模型中。它解决什么问题不同领域很难一起用一个大的混合 RL 训练（容易互相干扰、互相拉扯）  
  
传统离线蒸馏只学教师生成的轨迹，和实际推理时学生自己会走到的状态有偏差MOPD 兼顾了学生自己的状态分布+教师的密集 token 级指导。  
  
[aman.ai/primers/ai/kno…](https://aman.ai/primers/ai/knowledge-distillation/#multi-teacher-on-policy-distillation)