---
author: "陈成"
handle: "@chenchengpro"
source: "https://x.com/chenchengpro/status/2102782934189252662"
date: 2026-09-23T15:31:33.000Z
type: tweet
---
# 陈成 (@chenchengpro)

Opus 5.5 用了一天，攒了 24 条 tips，分五块。  
  
一、怎么用  
  
1. 整个任务直接扔给它。说清什么算做完，什么时候要回来找你。  
2. 别写「think carefully」，它本来就会先想。思考模式也关不掉了。  
3. 长任务跑完，问它「还缺什么才能继续」，别急着自己接手。  
4. 长任务跑一半停下来汇报是前沿模型的通病。Anthropic 给了一段专门的提示词压住这个毛病。  
5. 默认 medium 就够。智能跟 Fable 5.1 差不多但更快，输出 token 比官方建议还少两成。  
6. 别开 max。有人两次测试都在思考阶段撞上 128K 输出上限，一个结果没出，每次 2.56 美元、20 分钟。Fable 5.1 的 max 反而没事。  
7. 简单活 medium，难活切 high。Claude Code 2.1.280 之后，会话中途切强度不再打掉缓存。  
  
二、把旧提示词清一遍  
  
8. 跑 /claude-api prompt-audit。skills、CLAUDE.md、AGENTS.md 里那些给旧模型写的保姆规则，现在只会拖累 5.5。  
9. 该删的典型句式：全大写 IMPORTANT、一刀切的 be concise、always ask、always summarize、be thorough / don't stop。  
10. 每次模型升级都重跑一遍，拿几个代表任务验证，只留有效果的改动。  
11. 反模式不只在 CLAUDE.md，工具描述里也有。有人把 12 个工具砍到 4 个，效果比改任何提示词都大。  
12. 让 audit 解释每一条删除理由。有些看着奇怪的行，是绕某个 bug 的 workaround。  
  
三、账单  
  
13. 几乎所有事都能用 Opus 5.5 干，Fable 那 50% 的上限可以不管了。只有「给我点惊喜」那种模糊的创意原型，我还是回去找 Fable。  
14. 多 agent 编排里，coordinator 和 worker 都换成 5.5，额度掉得明显慢了。  
15. 缓存读取从 0.5 降到 0.2 美元。编程会话缓存命中率一般 95% 以上，实际省的比标价多。  
16. 三家里只有 Anthropic 不对长上下文加价，到 1M 单价不变。OpenAI 过 272K 翻倍，Grok 过 200K 翻倍。  
17. banked reset 不会挪动每周重置时间。留到周期后半段额度见底再用。  
  
四、顾问模式  
  
18. /advisor fable 把 advisor 设成 Fable。Fable 出方案，5.5 执行，做完让 advisor 验收，按反馈返工到通过为止。配 /goal 更顺。  
19. Opus 和 Fable 的真正差距在找开关。Opus 被 auto-mode 拦住会让你手动粘命令，Fable 会去 settings.json 里翻出 autoMode.allow。所以 Opus 当主力，Fable 别关。  
  
五、它变了什么  
  
20. 只在该长的时候长。「Claude 腔」和破折号没了，结论放最前面。  
21. 「honest mistake」循环没了。以前是先下结论、走错、道歉、再花几轮修，现在一次做对。  
22. 会自己遵守 skill，不用你提醒。  
23. 3D、视觉、Blender、设计品味都明显强了。视频、Minecraft、3D 网页一次成型。  
24. 网络安全和生物任务会被自动转给 Opus 4.8。2026-08-31 后注册的 API 账号拿不到推理过程。输出带文本水印。