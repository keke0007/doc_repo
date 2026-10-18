---
author: "Vince 聊开发"
handle: "@vincemask"
source: "https://x.com/vincemask/status/2057788920327520305"
date: 2026-05-13T07:04:45.000Z
type: tweet
---
# Vince 聊开发 (@vincemask)

很多人不知道什么时候该用 Hooks。  
我的判断很简单：凡是你需要反复提醒 Claude 的事，都应该考虑从 prompt 里拿出来，交给 Hook。  
  
比如：  
  
1、每次改完代码都要格式化  
2、每次提交前都要跑 lint / test  
3、禁止修改某些目录或配置文件  
4、生成代码后自动检查类型错误  
5、操作高风险文件前先拦截确认  
6、会话开始时自动注入项目上下文  
7、任务结束时自动记录变更摘要  
  
Prompt 适合表达意图，Hook 适合固化规则。越是稳定、重复、容易忘的流程，越应该交给环境默认执行。

> **Vince 聊开发 (@vincemask)**
> 
> ![Article cover](<Vince%20聊开发%20(@vincemask).assets/image-01.jpg>)
> 
> 📝 **从 0 开始：用 Hooks 打造自动化 Claude Code 工作流**
> 
> 你让 Claude Code 写功能、补测试、格式化文件，最后顺手提交。
> 它功能写了，测试也补了大半，格式化跑了两个文件，但偏偏漏了一个。然后它很自然地告诉你：“搞完了。”
> 你一看：没提交，格式也没完全对齐。...