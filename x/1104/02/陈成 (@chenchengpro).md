---
author: "陈成"
handle: "@chenchengpro"
source: "https://x.com/chenchengpro/status/2117845238786252885"
date: 2026-11-03T21:31:28.000Z
type: tweet
likes: 20
reposts: 2
replies: 3
bookmarks: 20
views: 3902
---
# 陈成 (@chenchengpro)

recommended reading.  
  
作者是 Jack Cable，Cognition 安全产品负责人，前 CISA、Corridor。总结最近两周的安全事件，再给一份「马上做」和「30 天内做」的清单。  
  
1/ 两周里发生的事：  
  
- Binance 被盗 63.4 亿美元，有记录以来最大的盗窃案，FBI 归因朝鲜。起点是开发者机器：几个工程师的笔记本通过假面试和被篡改的安装包中招，其中一台上有部署 token，能直接摸到钱包签名服务。  
- Cursor 上的 Claude Code 扩展被投毒（10/29 披露），恶意版本会装持久后门，让攻击者能 SSH 进开发者机器。扩展自动更新，用户什么都不用做就会中招，不到 5,000 人装了。恶意版本和 Open VSX 上的不一致，说明是 Open VSX 下游的发布流程被攻破了。  
- PHP2Shell：PHP 运行时的内存安全漏洞，某些配置下一个未认证请求就能在应用代码执行前触发，几个国家的政府网站直接下线。  
- 还有 GitLab 未认证 RCE；一批 VPN、防火墙等边界设备漏洞；CA eMudhra 的私钥被设成数字 4；KVM 虚拟机逃逸（Vercel Sandbox 赏金计划发现）；Linux MPTCP 远程提权 CVE-2026-98432，Go 1.24+ 默认开启 MPTCP。  
  
2/ 公司环境。马上做：高风险账号（Google Workspace、GitHub、云厂商、IdP）上 passkey 或硬件密钥这类防钓鱼 MFA，并且只允许受管设备登录；让 coding agent 扫一遍对外暴露面；全员用密码管理器；全员装广告拦截（如 uBlock Origin），Binance 的链条就是从搜 AI 编程工具点进恶意广告下载开始的。30 天内：所有设备上 MDM + EDR；浏览器和编辑器扩展走白名单，并设置自动更新延迟；按「一台开发机被攻破」这类场景审一遍整个攻击面，保证单台设备进不了生产。  
  
3/ 开发环境。这是全文重点。开发机上跑着大量第三方代码：依赖包、编辑器扩展、MCP server，现在还有能自己执行命令的本地 agent。任何一个被攻破，攻击者就继承了这个开发者能碰到的一切。马上做：用 TruffleHog 扫开发机上的密钥（home 目录、shell 历史、.env）；盘点团队装的扩展、插件、MCP server，重点看能碰文件系统、shell、凭证的。30 天内：密钥全部迁到 secrets manager 并轮换；转向云端 coding agent 和远程开发环境，把会出事的东西整个移出笔记本。  
  
4/ 代码。马上做：用前沿模型扫代码库和关键开源依赖，从对外服务、认证、支付开始；Docker 镜像和构建产物也要扫密钥。30 天内：规划大规模重构，比如迁到内存安全语言、全面改用参数化查询；扫描时把威胁模型告诉 harness（怎么部署、哪些对外、最该保护什么），通用 prompt 噪音太大。  
  
5/ 依赖。马上做：npm、PyPI 等设置最小包龄至少 7 天，在包管理器配置或前置代理里做都行，蠕虫式投毒（如 Shai-Hulud）基本在这个窗口内就会被发现下架；锁版本、提交 lockfile，CI 从 lockfile 装。30 天内：清理不用的依赖，看紧没人维护的。  
  
6/ 监控。确认需要的日志真的存在（IdP 登录、云审计、源码和 CI 活动、生产数据访问），并且放在拿到生产权限的攻击者也删不掉的地方；对新增管理员、MFA 变更、新设备或新地点登录、大量数据导出、异常读取密钥这些事件告警。  
  
原文 [x.com/jackhcable/sta…](https://x.com/jackhcable/status/2117731409246978489)  
博客版 [cognition.com/blog/practical…](https://cognition.com/blog/practical-guide-to-security-november-2026)  
相关调研 [research.sorrycc.dev/@sorrycc/2026-…](https://research.sorrycc.dev/@sorrycc/2026-10-security-incidents)

> **Jack Cable (@jackhcable)**
> 
> ![Article cover](<陈成%20(@chenchengpro).assets/image-01.jpg>)
> 
> 📝 **A Practical Guide to Security in November 2026**
> 
> I wouldn’t blame you if you’re having trouble keeping up with all of the security incidents over the last few weeks.
> Over a two-week period, we saw the largest theft in recorded history, a PHP...
> 
> It's been a rough few weeks for security.  
>   
> I pulled together a guide of the highest-impact actions you can take today: