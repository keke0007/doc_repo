---
author: "Man ting"
handle: "@Manorgw"
source: "https://x.com/Manorgw/status/2120980955322044896"
date: 2026-11-12T20:44:00.000Z
type: tweet
---
# Man ting (@Manorgw)

做安全的一位朋友前阵子差点被对手扒得底朝天，等他把这套东西看完才反应过来：防人和被人防，靠的居然是同一批工具。  
  
网警平日里查人用的五个开源神器，全免费，查人、查公司、查设备，能想到的场景基本都兜住了。  
  
不过丑话得说在前头，我不是教你扒谁，是让你先拿它照照自己。连自己漏了多少都没数，还谈什么防。  
  
五个工具各管各的，别混着用，地址和用途我挨个摆明白：  
  
1️⃣ Blackbird，6300 Star，用户名和邮箱都能查，接了600多个平台。查完还能白嫖AI整一份人物画像，PDF直接导出。新手别贪，先拿这个练手，最省事。  
🔗 [github.com/antoniaci/blac…](https://github.com/antoniaci/blackbird)  
2️⃣ Maigret，33000 Star，专盯用户名查人的狠货，覆盖3000多个平台。它毒在会递归追踪，比如发现你在别的平台还挂着个小号，就顺着这个小号继续往下刨，一路刨到底。这玩意儿真能把你挖穿。  
🔗 [github.com/soxoj/maigret](https://github.com/soxoj/maigret)  
3️⃣ SpiderFoot，18000 Star，这就是门重炮，手机号、邮箱、域名、IP全吃。200多个模块同时开火，能查数据泄露、暗网、子域名，最后还甩给你一张可视化关系图，谁跟谁有关系一眼看穿。  
🔗 [github.com/smicallef/spid…](https://github.com/smicallef/spiderfoot)  
4️⃣ theHarvester，16500 Star，查公司专用。扔给它一个域名，它自己就从谷歌、必应、LinkedIn、DNS记录这40多个源里，把公司员工的邮箱、子域名、IP、URL一批一批薅出来。做背调、摸一个公司，用它。  
🔗 [github.com/laramies/theHa…](https://github.com/laramies/theHarvester)  
5️⃣ Shodan Python，2800 Star。Shodan你就当设备版的谷歌，它满网扫那些直接裸奔在公网上的摄像头、路由器、服务器、工控系统。这个库就是让你用代码去查它那个数据库，批量、自动化地拉数据。  
🔗 [github.com/achillean/shod…](https://github.com/achillean/shodan-python)  
说句实在话，很多小公司的NAS、摄像头、打印机，装上之后配置压根没动过，密码还是出厂默认那个，Shodan一搜就给你抖出来。  
你以为自己藏得挺严实，真相是你能藏住的东西，比想象中少太多。  
所以别光看热闹。自己先拿这几个工具把自己跑一遍，看看到底暴露了啥、哪个口子没关，这才是防住自己的第一步。

![Image](<Man%20ting%20(@Manorgw).assets/image-01.png>)