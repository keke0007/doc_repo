---
author: "苏茶"
handle: "@Suercha007"
source: "https://x.com/Suercha007/status/2120016866655846674"
date: 2026-11-10T04:53:03.000Z
type: tweet
---
# 苏茶 (@Suercha007)

分享一个text2sql项目， WrenAI，18K 🌟。  
  
它是一个语义层，解决的是模型不清楚公司的业务定义问题。比如你库里有一张订单表一张退款表，问上个月销售额多少。  
  
Agent 多半会写成 select sum(amount) from orders。没扣退款，测试单也算进去了。  
  
因为它只知道表结构，不知道「销售额」在你们公司指什么。WrenAI 就是把口径写成文件，放在你自己的 Git 里：销售额要扣退款、要排除测试单、按创建时间算。  
  
agent 写 SQL 前先读这个文件，就写对了。  
  
还有两个闸门， SQL 先 dry-plan 校验一遍，逻辑不通当场改。  
  
还有行数限制，防止它 select * 把你几十亿行的库拖死。

> ![Link card preview](<苏茶%20(@Suercha007).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - Canner/WrenAI: GenBI (Generative BI) for AI agents, an open-source, governed text-to-SQL...**](https://t.co/Y18a5XoYVD)
> 
> _From github.com_