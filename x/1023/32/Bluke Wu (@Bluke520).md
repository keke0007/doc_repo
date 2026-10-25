---
author: "Bluke Wu"
handle: "@Bluke520"
source: "https://x.com/Bluke520/status/2113836298167611588"
date: 2026-10-24T03:33:41.000Z
type: tweet
likes: 176
reposts: 29
replies: 10
bookmarks: 348
views: 28681
---
# Bluke Wu (@Bluke520)

VPN显示美国IP，网站为什么还是知道你在中国🇨🇳  
  
这篇讲网站查的6个信号、怎么自查、怎么减少泄漏。  
  
一、先说清  
  
1：VPN 只改网络出口 IP，不改手机定位、浏览器信息、账户历史，网站会通过多个信号发现异常。  
  
二、网站查的 6 个信号  
  
信号1：VPN 不改 GPS，授权过的网页和 App 可能拿到真实坐标。  
  
信号2：IP 在美国、时区是上海，不一致可作异常参考，但不能单独证明人在中国。  
  
信号3：语言偏好中文，不等于人在中国，可成为设备指纹的一部分。  
  
信号4：某些浏览器和 VPN 配置可能暴露额外网络地址，但现代浏览器和正确配置的VPN已有不少防护，不是一定泄漏。  
  
信号5：VPN 只代理部分流量时，DNS 可能走本地或其他解析器，影响查询隐私，不代表网站一定知道真实位置。  
  
信号6：Cookie 和历史记录：注册国家、绑定号码、支付资料，换 VPN 也不会消失。  
  
三、怎么自查  
  
自查工具：  
  
[browserleaks.com](http://browserleaks.com) 查IP、DNS、WebRTC  
[coveryourtracks.eff.org](http://coveryourtracks.eff.org) 查浏览器指纹  
[whoer.net](http://whoer.net) 查 IP 和环境  
  
自查重点：  
  
1、出口IP是否显示境外IP  
2、不能出现任何的中国IP  
3、DNS有无非预期路径  
4、WebRTC有无暴露不希望公开的地址  
5、时区、语言、定位权限是否符合预期  
  
四、怎么减少泄漏  
  
1、小火箭或Clash的配置文件一定要选有DNS泄漏防护、仿WebRTC 暴露、减少Cookie关联的设置  
2、时区、语言按实际需要设，不必特意改成境外的  
  
最后：可以直接让GPT帮你生成一份专属你的配置文件，然后自己导入进去就行了。

> 🔗 [**Browserleaks - Check your browser for privacy leaks**](https://t.co/PlzkrBh4k7)
> 
> BrowserLeaks is a suite of tools that offers a range of tests to evaluate the security and privacy of your web browser. These tests focus on identifying ways in which websites may leak your real IP...
> 
> _From browserleaks.com_