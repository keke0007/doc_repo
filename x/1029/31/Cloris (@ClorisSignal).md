---
author: "Cloris"
handle: "@ClorisSignal"
source: "https://x.com/ClorisSignal/status/2114876818998337696"
date: 2026-10-26T06:27:53.000Z
type: tweet
likes: 10
replies: 2
bookmarks: 9
views: 959
---
# Cloris (@ClorisSignal)

Claude Code 团队没放弃code review：先让 AI 读，人只盯有风险的，出了错马上撤。  
  
@addyosmani 和两位同事讲了团队的做法：6 到 9 月，合进去的改动翻了一倍多，要撤回的比例反降四成多。  
  
他们分 5 步:  
  
1. 写完先自查：让 Claude 跑测试、拿结果说话，再叫一个没参与写的 AI 来看  
→ Claude Code 的作者 Boris Cherny 估计，这样结果好 2 到 3 倍；没参与写的 AI 不会护着刚写的代码  
  
2. 每次提交，几个 AI 分头找 bug（程序错误），够多票同意才提  
→ 不怕乱报的话，多找 bug 很容易；可 AI 作者每条都改，报错一条就多一轮  
  
3. 安全检查报的问题没处理完、自动测试没过，都不能合并  
→ 报错的，写句理由就能关，误报就这么清  
  
4. 按风险找人：改文档这类可不用人看，普通改动 1 人，最危险的 2 人；AI 点同意不算数  
→ 叫资深工程师查小错，比机器查贵得多；每个上线的改动都得有个人负责  
  
5. 先给自己员工用，新功能留开关，出事一键撤  
→ 每道关都会漏，但全漏过去的很少，剩下的靠撤回兜底  
  
我们做code review时可以这样：  
- 提交时写清：怎么测的、有啥风险、出事怎么撤  
- 把出过事的改动攒成考题，每次调 AI 审查员就重考  
- 给 AI 来回改设上限，几轮后只看新改的部分（有个改动一夜来回了 50 多轮）

![Image](<Cloris%20(@ClorisSignal).assets/image-01.jpg>)

> **Addy Osmani (@addyosmani)**
> 
> ![Article cover](<Cloris%20(@ClorisSignal).assets/image-02.jpg>)
> 
> 📝 **How the Claude Code team does code review**
> 
> We previously mentioned that Claude writes about 90 percent of the code for Claude Code. So how do we do code review at this scale? Who reads code anyway? What happens if code goes wrong?
> This post...