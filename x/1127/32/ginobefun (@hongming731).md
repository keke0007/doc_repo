---
author: "ginobefun"
handle: "@hongming731"
source: "https://x.com/hongming731/status/2125940741402988784"
date: 2026-11-26T13:12:25.000Z
type: thread
likes: 18
reposts: 2
replies: 4
bookmarks: 27
views: 1581
---
# ginobefun (@hongming731)

阿里把内部智能体安全治理的实践经验进行总结，并共同起草了 Deferred Token Response 规范，目前该规范草案已被 OAuth 工作组正式采纳，值得一读。  
  
主要包含以下 6 项核心能力：  
  
1. 智能体需要独立身份才能解决责任追溯问题  
智能体虽然不是法律主体，但作为独立行动单元，必须拥有区别于用户和传统应用的唯一标识。只有独立身份才能明确责任主体，使运营者、委托者和智能体之间的责任关系能够厘清，事后追责才有依据。  
  
2. 动态令牌确保授权在特定任务和时间范围内有效  
每次执行任务时，智能体应获得匹配当前任务的动态令牌。令牌绑定代表谁、执行者、权限范围、任务目的和有效期，防止长期凭证被滥用。高风险操作还需绑定关键参数，确保授权内容在执行前不被篡改。  
  
3. 最小授权原则要求按任务授予最小权限  
传统基于角色的授权不适用于智能体的自主规划。应从「角色通常能做什么」转向「完成此次任务最少需要做什么」。权限边界落实到资源、动作、条件和时间四个维度，防止 Agent 越权或滥用权限。  
  
4. 链式委托需可追溯且防止权限放大  
多跳委托要求每一跳验证授权来源，权限不能超过上一级拥有的范围，也不能超过当前子任务所需。时间、金额、用途等约束必须随委托传递。若需要新增权限，必须返回原始委托方授权。  
  
5. 审计存证需要形成可验证的证据链  
智能体行为审计应从记录日志升级为形成证据链。完整证据链包含发起、授权、委托、确认、执行和结果六类证据，通过数字签名、摘要和统一关联标识实现可验证、可追溯。  
  
6. 运行隔离为智能体提供受控的私密空间  
智能体运行环境必须与用户凭证、系统权限等隔离。隔离应覆盖模型、数据、凭证、工具和进程五个方面。通过沙箱等方式限制 Agent 访问宿主进程和无关数据，防止行为漂移导致链路风险。

![Image](<ginobefun%20(@hongming731).assets/image-01.jpg>)

---

文章链接：[bestblogs.dev/article/5a0c0a…](https://bestblogs.dev/article/5a0c0a9159)  
  
Deferred Token Response（DTR），IETF OAuth工作组互联网草案： [datatracker.ietf.org/doc/html/draft…](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-deferred-token-response-01)

> ![Link card preview](<ginobefun%20(@hongming731).assets/image-02.jpg>)
> 
> 🔗 [**智能体身份与授权：智能体生态的信任底座 | BestBlogs.dev**](https://t.co/XN30eUvqGN)
> 
> _From bestblogs.dev_