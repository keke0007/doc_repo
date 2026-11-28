---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2126390285508940020"
date: 2026-11-27T18:58:45.000Z
type: tweet
likes: 58
reposts: 6
replies: 9
bookmarks: 54
views: 2694
---
# 山景城小路 (@Zen_with_AI)

介绍一个好用的Gemini deep research skill;  
  
这个 skill 的核心思路是把"怎么做研究"写成流程，而不是让模型自己发挥。16 行里有五条规矩：  
  
1. 先拆角度。 开工前定 3 到 5 个研究角度，不直接上来乱搜。  
2. 并行加封顶。 每个角度派一个子 agent 去搜索和浏览，总共 8 到 12 步封顶，防止越搜越散。  
3. 只认一手来源。 财报、PDF 这类原始材料才算数，搜索结果里的片段不算证据。  
4. 卡住就停。 worker 卡住了及时取消，最多跟进一次，不在死胡同里耗。  
5. 收尾要短。 最后只给两句总结，加内联链接，来源之间有冲突就明确标出来。  
  
装法一行：npx skills add google-gemini/demolab --skill deep-research。  
  
数百次盲测里 90% 以上胜过不用 skill.