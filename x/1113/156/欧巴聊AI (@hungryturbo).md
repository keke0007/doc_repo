---
author: "欧巴聊AI"
handle: "@hungryturbo"
source: "https://x.com/hungryturbo/status/2121456182183551028"
date: 2026-11-14T04:12:23.000Z
type: tweet
---
# 欧巴聊AI (@hungryturbo)

AI 时代的 e2e 测试该怎么做?  
  
最近 GitHub 斩获 13.8k Star，刚拿了 800 万美金融资的开源 AI 测试框架 e2e，其创始团队技术布道师 Kacper Kapuściak 写了一篇非常扎实的工程实践，下面总结几条核心内容：  
  
1、测用户意图目标，而非前端实现，越过 DOM 节点与 test-id 绑定，直接用自然语言向 Agent 下达具体，收敛的业务意图。由 AI 自主感知界面完成选日期，点生成等交互，哪怕前端重构组件库，测试照样稳定。  
  
2、废弃拖慢 CI 流水线的固定 Sleep 读秒，改用带超时上限的 AI 动态条件等待。忽略Loading 消失这类弱的 UI 信号，直接等上月报表已打开可查看等强业务状态，并将负责异步加载的动态等待，与只抓拍当前画面的瞬时断言严格解耦。  
  
3、执行 Agent 汇报点击完成，不能代表报表没空数据或串数据，更不能让大模型盲猜什么才是正确的画面。必须先从后端 API 查出真实任务列表作为唯一真值源，再调用独立的 AI Judge，严格比对屏幕渲染明细与 API 底账。  
  
4、严禁测试用例之间产生顺序依赖，开跑前通过可复用的 Fixture 调接口独立构造专属工作区与测试数据，跑完后在生命周期钩子中自动清理还原，确保单测重试或并发执行绝对幂等。  
  
5、将“生成报表，下载 CSV，发邮件”拆成命名清晰的独立原子测试。一旦断言失败，框架自动打包故障追踪轨迹，涵盖前置步骤、应用日志、Agent 推理上下文与屏幕快照，不仅人类能秒级定位，更能直接喂给 Coding Agent 自动修 Bug。  
  
传送门：[e2e.com/blog/5-tips-fo…](http://e2e.com/blog/5-tips-for-writing-good-agentic-tests)

![Image](<欧巴聊AI%20(@hungryturbo).assets/image-01.jpg>)