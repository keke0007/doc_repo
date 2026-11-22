---
author: "陈发财"
handle: "@Chenfacai0"
source: "https://x.com/Chenfacai0/status/2121489052943999226"
date: 2026-11-14T06:23:00.000Z
type: tweet
---
# 陈发财 (@Chenfacai0)

有个专门给 Skill 做质检的开源工具。  
  
如今 Skill 数量泛滥，可真正能派上用场的能有多少？  
  
NVIDIA 干脆搞了个开源项目 SkillEvaluator，专门用来检验 Skill 到底管不管用。  
  
它会让同一个 Agent 跑同一组任务：  
  
→ 一轮不挂载 Skill  
  
→ 一轮挂载 Skill  
  
接着比对任务正确率、完成质量、执行耗时以及 Token 开销，看这个 Skill 到底带来了多少增益，还是反倒拉了后腿。  
  
NVIDIA 官方拿 300 多个 Verified Skills 做了实测，正确性平均分从 46 涨到了 87（满分 100）。  
  
除了测实际效果，我也顺手用它给自己做的 Skill 过了一遍质量关。  
  
GitHub：[github.com/NVIDIA/SkillEv…](https://github.com/NVIDIA/SkillEvaluator)  
  
入门教程：[docs.nvidia.com/skills/skillev…](https://docs.nvidia.com/skills/skillevaluator/quickstart)

![Image](<陈发财%20(@Chenfacai0).assets/image-01.jpg>)