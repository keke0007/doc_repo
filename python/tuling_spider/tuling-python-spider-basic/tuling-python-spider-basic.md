# 图灵Python爬虫基础课件

> Source: https://www.yuque.com/u41211103/nf8rlk

## Table of Contents

- 1.初识爬虫
  - 1.1.爬虫的相关概念
  - 1.2.爬虫流程以及案例演示
  - 1.3.HTTP与HTTPS协议
  - 1.4.编码
- 2.requests模块的使用
  - 2.1.requests简介与安装
  - 2.2.requests发送网络请求以及常用属性
  - 2.3.text与content方法区别
  - 2.4.下载网络图片
  - 2.5.iter_content方法
  - 2.6.携带请求头
  - 2.7.请求带有url查询参数的地址
  - 2.8.使用requests发送post请求
  - 2.9.requests处理cookie
  - 2.10.重定向与历史请求
  - 2.11.SSL证书问题
  - 2.12.请求超时
  - 2.13.retrying模块的使用
  - 2.14.发送json格式数据
  - 2.15.session会话
  - 2.16.代理
- 3.数据提取
  - 3.1.数据提取的概念和数据分类
  - 3.2.结构化数据提取-json
  - 3.3.练习：蜻蜓FM排行榜信息
  - 3.4.xpath语法
  - 3.5.使用lxml模块中的xpath语法提取非结构化数据
  - 3.6.练习：通过xpath提取豆瓣电影评论
  - 3.7.jsonpath模块
  - 3.8.练习：使用jsonpath提取数据
  - 3.9.非结构化数据提取-bs4
  - 3.10.练习：使用bs4抓取搜狗微信下的所有文章标题
- 4.正则表达式
  - 4.1.正则表达式的作用
  - 4.2.正则表达式在线验证工具
  - 4.3.常见语法
  - 4.4.常用字符串处理方式
- 5.数据存储
  - 5.1.CSV文件存储
  - 5.2.JSON文件存储
  - 5.3.MySQL数据库存储
  - 5.4.MySQL数据库连接池
  - 5.5.MongoDB数据库存储
  - 5.6.数据去重
- 6.并发爬虫
  - 6.1.asyncio结合requests完成爬虫任务
  - 6.2.使用aiohttp完成爬虫任务
  - 6.3.aiomysql的使用
  - 6.4.使用多线程完成并发爬虫
  - 6.5.使用线程池完成并发爬虫
  - 6.6.使用多进程完成并发爬虫
  - 6.7.并发爬虫实战
- 7.自动化测试框架
  - 7.1.环境配置
  - 7.2.基本使用
  - 7.3.元素定位方法
  - 7.4.selenium框架的其他方法
  - 7.5.项目实战
  - 7.6.补充 - DrissionPage框架
- 8.代理池的搭建
  - 8.1.免费代理采集脚本
  - 8.2.付费代理的设置与搭建
- 9.scrapy框架 - 基础操作
  - 9.1.scrapy项目的创建与启动
  - 9.2.parse方法中的响应对象
  - 9.3.响应对象中的属性与方法
  - 9.4.管道的基本使用
  - 9.5.scrapy框架的内置模块与执行流程
  - 9.6.多数据保存
  - 9.7.案例 - 豆瓣爬虫
  - 9.8.中间件的使用
  - 9.9.管道的详细使用
  - 9.10.数据去重
  - 9.11.增量爬虫
  - 9.12.dont_filter参数与start_requests方法
  - 9.13.发送post请求
- 10.scrapy框架 - 分布式爬虫以及爬虫部署
  - 10.1.scrapy-redis实现增量爬虫
  - 10.2.scrapy-redis实现分布式爬虫
  - 10.3.项目部署 - scrapyd
  - 10.4.项目部署 - scrapydweb
  - 10.5.项目部署 - gerapy
- 11.feapder框架
  - 11.1.feapder框架的简单使用
  - 11.2.feapder框架创建完整项目

## 1.初识爬虫

在本章节中会学习到如下内容：

- 爬虫程序的定义
- 网站数据获取的思路分析
- `http`网络协议
- 获取数据后的编码问题

数据获取的思路分析相对于代码来说更为重要，核心知识点在于如何利用浏览器自带的抓包工具获取对应数据。

在爬虫基础阶段浏览器统一使用谷歌浏览器最新版本。

> 来自: [1.初识爬虫](https://www.yuque.com/tuling_python/spider_base/xtfzkmnqs0t4gyfy)

### 1.1.爬虫的相关概念

**什么是爬虫**

网络爬虫（又被称为网页蜘蛛，网络机器人）就是模拟浏览器发送网络请求，接收请求响应，一种按照一定的规则，自动地抓取互联网信息的程序。

原则上，只要是浏览器(客户端)能做的事情，爬虫都能够做

**如何获取爬虫程序**

- 下载其他公司开发的通用爬虫(八爪鱼)
- 开发人员自己编写

**区别**

通用爬虫

可以提取大多数网站的数据，但是对于网站中某些特殊数据的提取方式没有实现

自定义爬虫

可以针对某一种网站自行开发符合要求的爬虫

**开发语言**

只要能够发送`HTTP(S)`请求的任何编程语言都是可以完成爬虫程序的，例如：`C++`、`java`、`php`、`JavaScript`等等，但是论爬虫开发效率一般都指的是`python`语言。

**爬虫分类**

根据抓取网站的数量不同，大致将爬虫分为两种：

- 通用爬虫：通常指搜索引擎的爬虫，例如：[https://www.baidu.com](https://www.baidu.com)
- 聚焦爬虫：针对特定网站的爬虫

> 来自: [1.1.爬虫的相关概念](https://www.yuque.com/tuling_python/spider_base/rofvi4mrhv0hgwky)

### 1.2.爬虫流程以及案例演示

**聚焦爬虫代码执行流程**

![](images/image-001.png)

流程说明

- 向起始地址发送请求，并获取响应
- 对响应结果进行数据提取
- 如果获取的数据是新的网站地址则继续发送请求并获取响应
- 如果获取的数据为页面需要的数据则完成数据保存

**案例：斗鱼图片**

目标

- 练习分析素材并提取素材地址的能力
- 手动下载素材

过程记录

斗鱼-颜值`URL`：[https://www.douyu.com/g_yz](https://www.douyu.com/g_yz)

分析出图片的`URL`：[https://rpic.douyucdn.cn/live-cover/roomCover/2023/09/02/003a4fd060deae496bab910340b6a165_big.png](https://rpic.douyucdn.cn/live-cover/roomCover/2023/09/02/003a4fd060deae496bab910340b6a165_big.png)

![](images/image-002.png)

在一般的网站中，图片地址都是在`html`代码的`img`标签中的，例如百度图片。但是斗鱼网站进过分析之后我们发现，图片并不在`html`代码当中。像这种网站的资源都是动态加载过来的，所以需要善于利用浏览器开发者工具进行网络抓包。基于抓包我们发现当前图片等动态信息位于：[https://www.douyu.com/wgapi/ordnc/live/web/room/yzList/1](https://www.douyu.com/wgapi/ordnc/live/web/room/yzList/1)

当前`api`返回的数据为`json`数据，在`json`数据中包含了主播封面图片地址。

**案例：抖音视频**

要求：获取抖音原视频地址

分析地址：[https://www.douyin.com/channel/300206](https://www.douyin.com/channel/300206)

![](images/image-003.png)

根据抓包分析出当前视频的`api`接口并返回`json`数据。在`json`数据中包含视频的播放地址，位于当前`api`的`url_list`节点。

**案例：淘宝评论**

要求：获取商家评论信息

分析地址：[https://item.taobao.com/item.htm?spm=a21bo.jianhua.201876.10.5af92a89LhtPtE&id=620925796742&scm=1007.40986.276750.0&pvid=a2473adf-6c80-4e9d-a1e8-84c2253bbed9](https://item.taobao.com/item.htm?spm=a21bo.jianhua.201876.10.5af92a89LhtPtE&id=620925796742&scm=1007.40986.276750.0&pvid=a2473adf-6c80-4e9d-a1e8-84c2253bbed9)

![](images/image-004.png)

根据浏览器抓包工具获取对应的评论`api`并获取响应的`json`数据。

> 来自: [1.2.爬虫流程以及案例演示](https://www.yuque.com/tuling_python/spider_base/zgiygrmqhm0f4hoa)

### 1.3.HTTP与HTTPS协议

目前大部分网站是基于`HTTP`与`HTTPS`进行网络交互的，在爬虫程序中也是发送网络协议来获取对应的网站信息，所以还是有必要了解网络协议。

**`HTTP`与`HTTPS`相关概念**

- `HTTP`

- 超文本传输协议
- 默认端口号：80

- `HTTPS`

- `HTTP` + `SSL`(安全套接字层)，即带有安全套接字层的超本文传输协议
- 默认端口号：443

`HTTPS`比`HTTP`更安全，但是性能更低。

**理解`HTTP`协议**

`HTTP`协议使用了`TCP`协议，接下来我们使用`网络调试助手`软件发送`HTTP`协议并携带`hello world`数据到浏览器。

软件下载地址：[https://soft.3dmgame.com/down/213757.html](https://soft.3dmgame.com/down/213757.html)

![](images/image-005.png)

操作步骤：

- 设置网络调试助手为`TCP Server`端
- 浏览器链接网络调试助手
- 发送`HTTP`协议到浏览器并携带数据
- 断开连接，浏览器显示相应内容

**`HTTP`协议的重要信息**

在以上案例中，我们想要给浏览器发送信息并显示，就必须要带上`HTTP`协议。`HTTP`协议中有一部分数据对爬虫程序来说非常重要。分别是请求头与响应头。

常见的请求头参数

- `Host` (主机和端口号)
- `Connection` (链接类型)
- `Upgrade-Insecure-Requests` (升级为`HTTPS`请求)
- `User-Agent`(浏览器名称)
- `Accept` (传输文件类型)
- `Referer` (页面跳转处)
- `Accept-Encoding`（文件编解码格式）
- `Cookie`（`Cookie`信息）
- `x-requested-with :XMLHttpRequest` (表示该请求是`Ajax`异步请求)

响应头参数

`Set-Cookie` （对方服务器设置`cookie`到用户浏览器的缓存）

响应状态码

- `200`：成功
- `302`：临时转移至新的`url`(一般会用`GET`，例如原本是`POST`则新的请求则是`GET`)
- `307`：临时转移至新的`url`(原本是`POST`则新的请求依然是`POST`)
- `403`：无请求权限
- `404`：找不到该页面
- `500`：服务器内部错误
- `503`：服务不可用，一般是被反爬

**浏览器发送`HTTP`请求过程**

![](images/image-006.png)

- 客户端发送网站域名到`DNS`服务器
- `DNS`服务器返回`IP`地址到客户端
- 客户端根据返回的`IP`地址访问网站后端服务器并请求网站资源
- 网站后端服务器返回对应页面资源

**`robots`协议**

网站通过`Robots`协议告诉搜索引擎哪些页面可以抓取，哪些页面不能抓取，但它仅仅是互联网中的约定而已，可以不用遵守。例如：[https://www.taobao.com/robots.txt](https://www.taobao.com/robots.txt)

在后期的`Scrapy`框架学习中，需要手动关闭`Robots`协议，现阶段了解即可。

**谷歌浏览器插件**

- XPath Helper
- Web Scraper
- Toggle JavaScript
- User-Agent Switcher for Chrome
- EditThisCookie
- SwitchySharp

插件下载地址：

- [https://extfans.com/](https://extfans.com/)
- [https://chrome.zzzmh.cn/#/index](https://chrome.zzzmh.cn/#/index)

**请求测试软件**

`PostMan`：[https://www.postman.com/downloads](https://www.postman.com/downloads)

`ApiPost`：[https://www.apipost.cn/download.html](https://www.apipost.cn/download.html)

> 来自: [1.3.HTTP与HTTPS协议](https://www.yuque.com/tuling_python/spider_base/xt23ztty15qw4fsh)

### 1.4.编码

字符是各种文字和符号的总称，包括国家文字、标点符号、图形符号、数字等等。

字符集是多个字符的集合，字符集包括：`ASCII`、`GB2312`、`Unicode`等等。`UTF-8`是`Unicode`的实现方式之一

`Python3`中的字符串

- `str`：`unicode`的呈现形式
- `bytes`：字节类型，互联网上的数据都已以二进制的方式传输的

`str`与`bytes`类型的互相转换

- `str`使用`encode`方法转换为`bytes`

```python
str_code = 'abc'
print(type(str_code))

byte_code = str_code.encode()
print(type(byte_code))
```

- `bytes`使用`decode`方法转换为`str`

```python
byte_code = b'abc'
print(type(byte_code))

str_code = byte_code.decode()
print(type(str_code))
```

注意：编码方式必须和解码方式一样，否则就会出现乱码问题。例如使用`utf-8`编码，那么就必须使用`utf-8`解码。

> 来自: [1.4.编码](https://www.yuque.com/tuling_python/spider_base/gexdqc8angdusfur)

## 2.requests模块的使用

`requests`模块在爬虫程序中的应用相当广泛，可以说一个合格的爬虫工程师是必须要掌握的知识点。

所以在本章节中我们会使用`requests`获取各个网站的页面数据。

需要掌握的核心知识点：

- `get`请求
- `post`请求
- `form`表单参数
- `params`查询字符串参数
- `json`参数
- 请求头设置
- `session`会话
- `cookie`处理
- 代理设置

> 来自: [2.requests模块的使用](https://www.yuque.com/tuling_python/spider_base/owgo9erz8env26ig)

### 2.1.requests简介与安装

作用：发送网络请求，返回响应数据。

中文文档：[https://requests.readthedocs.io/projects/cn/zh_CN/latest/](https://requests.readthedocs.io/projects/cn/zh_CN/latest/)

对于爬虫任务，使用`requests`模块基本能够解决绝大部分的数据抓取的任务。所以用好`requests`至关重要

![](images/image-007.png)

**功能特性**

- `Keep-Alive` & 连接池
- 国际化域名和`URL`
- 带持久`Cookie`的会话
- 浏览器式的`SSL`认证
- 自动内容解码
- 基本/摘要式的身份认证
- 优雅的 `key/value Cookie`
- 自动解压
- `Unicode`响应体
- `HTTP(S)`代理支持
- 文件分块上传
- 流下载
- 连接超时
- 分块请求
- 支持 `.netrc`

**模块安装**

![](images/image-008.png)

```bash
pip install requests -i https://pypi.tuna.tsinghua.edu.cn/simple
```

> 来自: [2.1.requests简介与安装](https://www.yuque.com/tuling_python/spider_base/flvnhpsymosh9z5m)

### 2.2.requests发送网络请求以及常用属性

需求：通过`requests`向百度首页发送请求，获取百度首页数据

```python
import requests

url = "https://www.baidu.com"

response = requests.get(url=url)

print("---状态码如下---")
print(response.status_code)

print("---bytes类型数据：---")
print(response.content)

print("---str类型数据---")
print(response.text)

print("---str类型数据(utf-8)---")
print(response.content.decode("utf-8"))
```

常用属性如下：

- `response.text` 响应体`str`类型
- `respones.content` 响应体`bytes`类型
- `response.status_code` 响应状态码
- `response.request.headers` 响应对应的请求头
- `response.headers` 响应头
- `response.request.headers.get('cookies')` 响应对应请求的`cookie`
- `response.cookies` 响应的`cookie`（经过了`set-cookie`动作）
- `response.url`请求的`URL`

> 来自: [2.2.requests发送网络请求以及常用属性](https://www.yuque.com/tuling_python/spider_base/rk87ynp0u39pmata)

### 2.3.text与content方法区别

- `response.text`

- 类型：`str`
- 解码类型：`requests`模块根据`HTTP`头部对响应的编码推测文本编码类型
- 修改编码方式：`response.encoding = 'gbk'`

- `response.content`

- 类型：`bytes`
- 解码类型：没有指定
- 修改解码方式：`response.content.decode('utf-8')`

获取网页源码的通用方式：

- `response.encoding = 'utf-8'`
- `response.content.decode('utf-8')`
- `response.text`

以上三种方式从前往后依次尝试，百分百可以解决网页编码问题。

```python
import requests

r = requests.get("https://www.baidu.com")

print("-----requests一般能够根据响应自动解码-----")
print(r.text)

print("-----如果不能够解析出想要的真实数据，可以通过设置解码方式-----")
r.encoding = "utf-8"
print(r.text)
```

> 来自: [2.3.text与content方法区别](https://www.yuque.com/tuling_python/spider_base/xxf61gab9okamvlc)

### 2.4.下载网络图片

需求：将百度`logo`下载到本地

思路分析：

- `logo`的`url`地址：[https://www.baidu.com/img/bd_logo1.png](https://www.baidu.com/img/bd_logo1.png)
- 利用`requests`模块发送请求并获取响应
- 使用二进制写入的方式打开文件并将`response`响应内容写入文件内

```python
import requests

# 图片的url
url = 'https://www.baidu.com/img/bd_logo1.png'

# 响应本身就是一个图片,并且是二进制类型
response = requests.get(url)

# print(r.content)

# 以二进制写入的方式打开文件
with open('baidu.png', 'wb') as f:
    # response.content: bytes二进制类型
    f.write(response.content)
```

> 来自: [2.4.下载网络图片](https://www.yuque.com/tuling_python/spider_base/ot3pin8khrcliyg5)

### 2.5.iter_content方法

如果下载一个较大的资源，例如一个视频，可能需要的下载时间较长，在这个较长的下载过程中程序是不能做别的事情的（当然可以使用多任务来解决），如果在不是多任务的情况下，想要知道下载的进度，此时就可以通过类似迭代的方式下载部分资源。

- 使用`iter_content`

在获取数据时，设置属性`stream=True`

```python
r = requests.get('https://www.baidu.com', stream=True)

with open('test.html', 'wb') as f:
    for chunk in r.iter_content(chunk_size=100):
        f.write(chunk)
```

- `stream=True`说明：

- 如果设置了`stream=True`，那么在调用`iter_content`方法时才会真正下载内容
- 如果没设置`stream`属性则调用`requests.get`就会耗费时间下载

- 显示视频下载进度

```python
# pip install tqdm

import requests
from tqdm import tqdm

def download_video(url, save_path):
    response = requests.get(url, stream=True)
    # 获取视频资源大小
    total_size = int(response.headers.get('content-length', 0))

    # 初始化下载大小
    downloaded_size = 0

    # unit: 下载单位（字节）
    # unit_scale: 自动调整单位
    # unit_divisor: 单位换算（除数为1024）
    with open(save_path, 'wb') as file, tqdm(total=total_size, unit='B', unit_scale=True, unit_divisor=1024) as bar:
        for chunk in response.iter_content(chunk_size=1024):
            if chunk:
                file.write(chunk)
                downloaded_size += len(chunk)
                bar.update(len(chunk))

    print("下载完成...")

# 调用下载函数
video_url = "你要下载的视频资源地址"
path = "video.mp4"
download_video(video_url, path)
```

> 来自: [2.5.iter_content方法](https://www.yuque.com/tuling_python/spider_base/ea7ry0a8dgt8xcas)

### 2.6.携带请求头

在请求某些网址时根据不同的浏览器会返回不同的响应内容，所以此时就需要根据需求来修改或添加请求头信息。

获取请求头与响应头

```python
import requests

response = requests.get("https://www.baidu.com")

print(response.headers)  # 响应头
print(response.request.headers)  # 请求头
```

定义`User-Agent`

```python
headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36"}
```

携带请求头访问百度

```python
import requests

url = 'https://www.baidu.com'

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36"
}

# 在请求头中带上User-Agent，模拟浏览器发送请求
response = requests.get(url, headers=headers)

# 打印请求头信息
print(response.request.headers)

# 响应内容
print(response.text)
```

> 来自: [2.6.携带请求头](https://www.yuque.com/tuling_python/spider_base/blfvfgqv3cngfk58)

### 2.7.请求带有url查询参数的地址

我们在使用百度搜索的时候经常发现`url`地址中会有一个`?`，那么该问号后边的就是请求参数，又叫做查询字符串。如果想要做到自动搜索，就应该让发送出去的`url`携带参数。

示例地址：[https://www.baidu.com/s?wd=python](https://www.baidu.com/s?wd=python)

```python
# 1. 设置需要携带的查询字符串参数
kw = {'wd': 'java'}

# 2. 发送请求
response = requests.get('https://www.baidu.com/s', params=kw)

# 3.查看发送的URL
print(response.url)
```

当前查询字符串参数可以直接写到`url`地址中：

```python
import requests

headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36"}

url = 'https://www.baidu.com/s?wd=python'

# url中包含了请求参数，所以此时无需params
response = requests.get(url, headers=headers)

print("请求的URL：", response.url)
print("响应内容如下：", response.content)
```

> 来自: [2.7.请求带有url查询参数的地址](https://www.yuque.com/tuling_python/spider_base/ug7zib1cwnuw1tyk)

### 2.8.使用requests发送post请求

在`HTTP`请求中，`GET`和`POST`是使用最为频繁的请求方式。

- `GET`：获取数据
- `POST`：提交数据

**发送`post`请求**

`requests`模块中能发送多种请求，例如：`GET`、`POST`、`PUT`、`DELETE`等等

发送`post`请求代码示例：

请求地址：[http://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse](http://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse)

```python
import requests

# 获取到url
url = 'http://www.cninfo.com.cn/new/disclosure'
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36'
}
data = {
    'column':'szse_latest',
    'pageNum':'2',
    'pageSize':'30',
    'sortName':'',
    'sortType':'',
    'clusterFlag':'true',
}

response = requests.post(url, headers=headers, data=data)
print(response.json())
```

过程分析：

- 通过浏览器开发者工具获取当前网站数据`API`
- 查询载荷选项卡中的表单数据
- 构建表单数据
- 发送`post`请求

**代码练习：百度翻译**

当前网站需要使用移动端的方式访问

请求地址： https://fanyi.baidu.com/#en/zh/

```python
import requests

headers = {
    "Accept": "*/*",
    "Accept-Language": "zh-CN,zh;q=0.9",
    "Connection": "keep-alive",
    "Content-Type": "application/x-www-form-urlencoded",
    "Origin": "https://fanyi.baidu.com",
    "Referer": "https://fanyi.baidu.com/",
    "Sec-Fetch-Dest": "empty",
    "Sec-Fetch-Mode": "cors",
    "Sec-Fetch-Site": "same-origin",
    "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 16_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/16.6 Mobile/15E148 Safari/604.1",
    "X-Requested-With": "XMLHttpRequest"
}
cookies = {
    "BIDUPSID": "E903EA6FE7E149DD08AC410F69E62C93",
    "PSTM": "1725263983",
    "BAIDUID": "E903EA6FE7E149DDF6463D0D6101FEF5:FG=1",
    "BAIDUID_BFESS": "E903EA6FE7E149DDF6463D0D6101FEF5:FG=1",
    "ZFY": "yfOO:BiZMHxBokWiBeepKmh:BIO8qDADHGAKKrtn2QjZo:C",
    "newlogin": "1",
    "H_PS_PSSID": "60677_60682_60694_60725_60360",
    "BA_HECTOR": "0h84052lag800g2k20ak8021buj90u1je2jcs1v",
    "BDORZ": "B490B5EBF6F3CD402E515D22BCDA1598",
    "Hm_lvt_afd111fa62852d1f37001d1f980b6800": "1726117944",
    "HMACCOUNT": "3A98C46FC7637597",
    "Hm_lvt_64ecd82404c51e03dc91cb9e8c025574": "1726117944",
    "Hm_lpvt_64ecd82404c51e03dc91cb9e8c025574": "1726118908",
    "Hm_lpvt_afd111fa62852d1f37001d1f980b6800": "1726118908",
    "ab_sr": "1.0.1_YTNlZGU5NjhhMDcxYTA4MmNhMzA3ZDc4NzA3MTIwMDRhNjQzMDVlZTlkMWFlY2VjZGI5NzVlZGNmMmFkMjVlNzYzMmY0MTY0M2QzNzg2NWM0OWVmMzk2YzQ5NjBiZjNmMmMxMDkwODMwN2JiOGE0OWM4NmY2Y2IxOTI0YTg0OGFjNTBmOWU2YmJjZTE3OTFmZDgzNTdmMWU3ZDkyMGFiOQ=="
}
url = "https://fanyi.baidu.com/extendtrans"
data = {
    "query": "happy",
    "from": "en",
    "to": "zh",
    "token": "4608866bf8cfe1a234c85a8e47900029",
    "sign": "221212.492333"
}
response = requests.post(url, headers=headers, cookies=cookies, data=data).json()

print(response['data']['st_tag'][0])
```

> 来自: [2.8.使用requests发送post请求](https://www.yuque.com/tuling_python/spider_base/fkglbyggftan8uuq)

### 2.9.requests处理cookie

引入

为了能够通过爬虫获取到登录后的页面，或者是解决通过`cookie`的反爬，需要使用`request`来处理`cookie`相关的请求。

爬虫中使用`cookie`的利弊

- 带上`cookie`的好处

- 能够访问登录后的页面
- 能够实现部分反反爬

- 带上`cookie`的坏处

- 一套`cookie`往往对应的是一个用户的信息，请求太频繁有更大的可能性被对方识别为爬虫
- 那么上面的问题如何解决？使用多个账号

**发送请求时添加`cookie`**

- `cookie`字符串放在`headers`中

```python
# 将cookie添加到headers中
headers = {
    "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 "
                  "(KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1",

    "Cookie": 'BIDUPSID=07953C6101318E05197E77AFF3A49007; PSTM=1695298085; '
              'ZFY=jAXoBNlaBGlHggda:BLlW8x7pEMyEhiZUIRbuQnnavss:C; '
              'APPGUIDE_10_6_5=1; REALTIME_TRANS_SWITCH=1; '
              'FANYI_WORD_SWITCH=1; HISTORY_SWITCH=1; '
              'SOUND_SPD_SWITCH=1; SOUND_PREFER_SWITCH=1; '
              'BAIDUID=37927E8274D89B902DEB6F1A024B3860:FG=1; '
              'BAIDUID_BFESS=37927E8274D89B902DEB6F1A024B3860:FG=1; '
              'RT="z=1&dm=baidu.com&si=ba30f04e-d552-4a5a-864f-1b2b222ff176&ss=lne882ji&sl=2&tt=1'
              'ju&bcn=https%3A%2F%2Ffclog.baidu.com%2Flog%2Fweirwood%3Ftype%3Dperf&ld=3pn&nu='
              '1dzl78ujc&cl=3bd&ul=61c&hd=622"; BA_HECTOR=a12g8ka22421al2k80ak21a31ihvc081o; '
              'BDORZ=B490B5EBF6F3CD402E515D22BCDA1598; '
              'Hm_lvt_64ecd82404c51e03dc91cb9e8c025574=1695476554,1696577209; '
              'Hm_lvt_afd111fa62852d1f37001d1f980b6800=1695476565,1696577271; '
              'Hm_lpvt_afd111fa62852d1f37001d1f980b6800=1696577271; Hm_lpvt_64e'
              'cd82404c51e03dc91cb9e8c025574=1696577271; ab_sr=1.0.1_MjZiYjAyZTQ4OTZkNWU0Y2M'
              '5YjQxMzZiOTE4Y2ZkOWNmMmI2MTNiMzhlOWQ0MTE4MzU0NDg5Njc5ZWU1ZDVkN2E4ZmM2Zjg3NjA5N2IwYWQ3OG'
              'I3ZDBlYWJlMmFmODM3Y2FhZmJkYzgxY2EzZmI1NWRiZDgxNWMxOTU3ZjNhZTk3NzE0ZDg1OGY1MGM4YTM2ZjA1'
              'ZTY4MGViOTI2OTlhYQ=='
}
```

`headers`中的`cookie`格式：

- 使用分号`;`隔开
- 分号两边的类似`a=b`形式的表示一条`cookie`
- `a=b`中，`a`表示键`name`，`b`表示值`value`
- 在`headers`中仅仅使用了`cookie`的`name`和`value`

- 把`cookie`字典传给请求方法的`cookies`参数接收

```python
cookies = {"cookie的name": "cookie的value"}

requests.get(url, headers=headers, cookies=cookie_dict)
```

- 使用`requests`提供的`session`模块（后面讲解）

**获取响应时提取`cookie`**

使用`requests`获取的`resposne`对象，具有`cookies`属性，能够获取对方服务器设置在本地的`cookie`，但是如何使用这些`cookie`呢？

使用`requests`模块提供的`response.cookies`方法。

- `response.cookies`是`CookieJar`类型
- 使用`requests.utils.dict_from_cookiejar`，能够实现把`cookiejar`对象转化为字典

```python
import requests

url = "https://www.baidu.com"
# 发送请求，获取resposne
response = requests.get(url)
print(type(response.cookies))

# 使用方法从cookiejar中提取数据
cookies = requests.utils.dict_from_cookiejar(response.cookies)
print(cookies)
```

> 来自: [2.9.requests处理cookie](https://www.yuque.com/tuling_python/spider_base/hb93ityee4cglgxd)

### 2.10.重定向与历史请求

测试代码

```python
import requests

headers = {
    "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"
}

r = requests.get("http://www.baidu.com", headers=headers)
print(r.url)

# 以上代码打印结果为：https://m.baidu.com/?from=844b&vit=fps
```

思考：为什么打印出来的`url`不是请求的`url`呢？

想要搞清楚这个问题，就要知道`requests`的重定向问题。

`requersts`的默认情况

默认情况下，`requests`发送的请求除了方式为`HEAD`之外，其余的请求例如`GET`、`POST`等都是能自动进行重定向的

这也就是为什么上面明明访问的是`http://www.baidu.com`而打印出来之后是`https://m.baidu.com/?from=844b&vit=fps`的原因

取消自动重定向

在发送请求的时候，可以通过如下的设置，取消`requests`模块的自动重定向功能

```python
requests.get(url, allow_redirects=False)
```

示例代码：

```python
import requests

headers = {
    "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"
}

response = requests.get("http://www.baidu.com", headers=headers, allow_redirects=False)

print(response.status_code)
print(response.url)
```

默认情况下获取历史请求

通过`response.history`可以获取到请求的历史记录

```python
import requests

headers = {
    "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"
}

response = requests.get("http://www.360buy.com", headers=headers)

print("历史请求过程信息：")
print(response.history)
for one_info in response.history:
    print(one_info.status_code, one_info.url, one_info.headers)

print("\n\n最后一次的请求信息：")
print(response.status_code, response.url, response.headers)
```

> 来自: [2.10.重定向与历史请求](https://www.yuque.com/tuling_python/spider_base/npk71lbe4lghay9w)

### 2.11.SSL证书问题

在浏览网页时，可能会遇到以下这种情况：

![](images/image-009.png)

出现这个问题的原因是：`ssl`证书不安全导致的。

在代码中发起请求的效果

```python
import requests

# 网站已无法正常访问, 可参考截图...
url = "https://chinasoftinc.com/owa"
response = requests.get(url)
print(response.text)

# 当前程序报错：ssl.CertificateError...
```

解决方案

在代码中设置`verify`参数

```python
import requests

url = "https://chinasoftinc.com/owa"
response = requests.get(url, verify=False)
print(response.text)
```

> 来自: [2.11.SSL证书问题](https://www.yuque.com/tuling_python/spider_base/gq7fgzvw3cz4ilor)

### 2.12.请求超时

在爬虫中，一个请求很久没有结果，就会让整个项目的效率变得非常低。这个时候我们就需要对请求进行强制要求，让他必须在特定的时间内返回结果，否则就报错。

超时参数的使用

```python
r = requests.get(url, timeout=3)
```

通过添加`timeout`参数，能够保证在三秒钟内返回响应，否则会报错。

```python
import requests

# url = "https://www.baidu.com"
url = "https://www.google.com"

response = requests.get(url=url, timeout=1)
```

这个方法还能够拿来检测代理`ip`（代理会在后面讲解）的质量，如果一个代理`ip`在很长时间没有响应，那么添加超时之后也会报错，对应的这个`ip`就可以从代理`ip`池中删除。

> 来自: [2.12.请求超时](https://www.yuque.com/tuling_python/spider_base/bsdlkx4sk2xyw5rt)

### 2.13.retrying模块的使用

使用超时参数能够加快我们整体的运行速度。但是在普通的生活中当我们使用浏览器访问网页时，如果发生速度很慢的情况，我们会做的选择是刷新页面。那么在代码中，我们是否也可以刷新请求呢？

在本小节中我们使用`retrying`模块来完成需求。

`retrying`模块的使用

模块地址：[https://pypi.org/project/retrying/](https://pypi.org/project/retrying/)

```bash
# 安装指令如下：
pip install retrying -i https://pypi.tuna.tsinghua.edu.cn/simple
```

作用：

- 使用`retrying`模块提供的`retry`模块
- 通过装饰器的方式使用，让被装饰的函数反复执行
- `retry`中可以传入参数`stop_max_attempt_number`,让函数报错后继续重新执行，达到最大执行次数的上限，如果每次都报错，整个函数报错，如果中间有一个成功，程序继续往后执行

```python
import time
from retrying import retry

num = 1

@retry(stop_max_attempt_number=3)
def test():
    global num
    print("num=", num)
    num += 1
    time.sleep(1)
    for i in 100:
        print("i", i)

if __name__ == '__main__':
    try:
        test()
    except Exception as ret:
        print("产生异常...")
        print(ret)
    else:
        print("没有异常")
```

`retrying`和`requests`的简单封装

实现一个发送请求的函数，每次爬虫中直接调用该函数即可实现发送请求

- 使用`timeout`实现超时报错
- 使用`retrying`模块实现重试

示例代码：

```python
import requests
from retrying import retry

num = 1

@retry(stop_max_attempt_number=3)
def _parse_url(url):
    global num
    print("第%d次尝试" % num)
    num += 1
    headers = {
        "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"
    }
    # 超时的时候会报错并重试
    response = requests.get(url, headers=headers, timeout=3)
    # 状态码不是200，也会报错并重试
    assert response.status_code == 200  # 此语句是"断言"，如果assert后面的条件为True则呈现继续运行，否则抛出异常
    return response

def parse_url(url):
    # 进行异常捕获
    try:
        response = _parse_url(url)
    except Exception as e:
        print("产生异常：", e)
        # 报错返回None
        response = None
    return response

if __name__ == '__main__':
    url = "https://chinasoftinc.com/owa"
    # url = "https://www.baidu.com"
    print("----开始----")
    r = parse_url(url=url)
    print("----结束----", "响应内容为：", r)
```

> 来自: [2.13.retrying模块的使用](https://www.yuque.com/tuling_python/spider_base/iu71xvmqg4diilcf)

### 2.14.发送json格式数据

当我们发送`POST`请求的时候，一般会携带数据，之前在学习`POST`时，可以通过给`data`赋值，从而能够完成传递`form`表单数据。

```python
requests.post(url, data={"kw": "python"})
```

但有很多时候，要向服务器发送的是`json`数据，此时应该怎么办呢？

```python
requests.post(url, json={"kw": "python"})
```

在请求方法中设置`json`参数即可。

代码示例：

```python
import requests

headers = {
    "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"
}

response = requests.post("https://fanyi.baidu.com/sug", headers=headers, json={"kw": "python"}, timeout=3)
print("请求头是：", response.request.headers)
print("请求体是：", response.request.body)
```

> 来自: [2.14.发送json格式数据](https://www.yuque.com/tuling_python/spider_base/drlqqetqt8sl5rbs)

### 2.15.session会话

当我们在爬取某些页面的时候，服务器往往会需要`cookie`，而想要得到`cookie`就需要先访问某个`URL`进行登录，服务器接收到请求之后验证用户名以及密码在登录成功的情况下会返回一个响应，这个响应的`header`中一般会有一个`set-cookie`的信息，它对应的值就是要设置的`cookie`信息。

虽然我们再之前可以通过`requests.utils.dict_from_cookiejar(r.cookies)`提取到这个响应信息中设置的新`cookie`，但在下一个请求中再携带这个数据的过程较为麻烦，所以`requests`有个高级的方式 - 会话`Session`

`Session`的作用

`Session`能够跨请求保持某些参数，也会在同一个`Session`实例发出的所有请求之间保持`cookie`

会话保持有两个内涵：

- 保存`cookie`，下一次请求会自动带上前一次的`cookie`
- 实现和服务端的长连接，加快请求速度

使用方法

```python
# 1. 创建一个session实例对象
s = requests.Session()

# 2. 使用上一步创建的对象发起请求
r = s.get(url1, headers)
r = s.get(url2, headers)
r = s.get(url3, headers)
r = s.get(url4, headers)
```

`session`对象在请求了一个网站后，对方服务器设置在本地的`cookie`会保存在`session`对象中，下一次再使用`session`对象请求对方服务器的时候，会自动带上前一次的`cookie`。

代码示例：

```python
import requests

session = requests.Session()

headers = {
    "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"
}

# 发送第一个请求
response = session.get('https://www.baidu.com', headers=headers)
print("第一次请求的请求头为:", response.request.headers)
print("响应头：", response.headers)
print("设置的cookie为:", requests.utils.dict_from_cookiejar(response.cookies))

# 发送第二个请求
response = session.get("https://www.baidu.com")
print("第二次请求的请求头为:", response.request.headers)
```

> 来自: [2.15.session会话](https://www.yuque.com/tuling_python/spider_base/sp6whe3c6laa2ihg)

### 2.16.代理

使用代理的原因

当在爬某个网站的时候，如果对方进行了封锁例如将我们电脑的公网`ip`封锁了，那么也就意味着只要是这个`ip`发送的所有请求这个网站都不会进行响应；此时我们就可以使用代理，绕过它的封锁从而实现继续爬取数据

基本原理

在当前用户电脑中连接其他区域的电脑，每台电脑因为区域不同所以分配的`ip`也不相同。使用其他区域的电脑帮助我们发送想要发送的请求。

基本使用

将代理地址与端口配置成字典并使用`proxies`参数传递

```python
proxies = {
  "http": "http://10.10.1.10:3128",
  "https": "http://10.10.1.10:1080",
}

requests.get("https://example.org", proxies=proxies)
```

如何获取代理

- 百度查询`免费代理ip`，但一般情况下都不太好用
- 付费代理：[https://www.zmhttp.com/?have_open_ok=1](https://www.zmhttp.com/?have_open_ok=1)

对于免费代理大部分都是不可用的，建议可以使用付费代理。例如：芝麻代理、快代理等等。

代理案例

```python
import requests

# http代理
ip = "127.0.0.1"
port = 7890

proxies = {
    "http": "http://%s:%d" % (ip, port),
    "https": "http://%s:%d" % (ip, port)
}

# 请求头
headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 "
                  "(KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36"
}

url = "http://httpbin.org/ip"

response = requests.get(url=url, headers=headers, proxies=proxies, timeout=10)
print(response.text)
```

> 来自: [2.16.代理](https://www.yuque.com/tuling_python/spider_base/dlurbf9gtv5pisa2)

## 3.数据提取

利用`requests`可以获取网站页面数据，但是`requests`返回的数据中包含了一些冗余数据，我们需要在这些数据集中提取自己需要的信息。所以在本章节中我们会重点讲解如何在数据集中提取自己需要的数据。

需要掌握的知识点如下：

- `json`数据提取

- `jsonpath`语法

- 静态页面数据提取

- `xpath`语法
- `bs4`模块使用

- 正则表达式的使用

> 来自: [3.数据提取](https://www.yuque.com/tuling_python/spider_base/dguypboogogkwe54)

### 3.1.数据提取的概念和数据分类

在爬虫爬取的数据中有很多不同类型的数据,我们需要了解数据的不同类型来又规律的提取和解析数据。

- 结构化数据：`json`、`xml`

- 处理方式：直接转化为`python`数据类型

- 非结构化数据：`HTML`

- 处理方式：正则表达式、`xpath`、`bs4`

**结构化数据**

`json`

![](images/image-010.png)

`xml`

![](images/image-011.png)

**非结构化数据**

![](images/image-012.png)

> 来自: [3.1.数据提取的概念和数据分类](https://www.yuque.com/tuling_python/spider_base/uda438gngv14worq)

### 3.2.结构化数据提取-json

**什么是`json`**

`JSON(JavaScript Object Notation)` 是一种轻量级的数据交换格式，它使得人们很容易的进行阅读和编写，同时也方便了机器进行解析和生成，适用于进行数据交互的场景，比如网站前端与后端之间的数据交互。

**`json`模块方法回顾**

![](images/image-013.png)

```python
# json.dumps 实现python类型转化为json字符串
# indent实现缩进格式
# ensure_ascii=False实现让中文写入的时候保持为中文
json_str = json.dumps(mydict, indent=2, ensure_ascii=False)

# json.loads 实现json字符串转化为python的数据类型
my_dict = json.loads(json_str)
```

**代码示例**

```python
import json
import requests

# 网站地址: http://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse

# 获取公告信息
url = 'http://www.cninfo.com.cn/new/disclosure'

# 定义请求头，模拟浏览器
headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36",
}

# 定义表单数据
post_data = {
    'column': 'szse_latest',
    'pageNum': 1,
    'pageSize': 30,
    'sortName': '',
    'sortType': '',
    'clusterFlag': 'true'
}

# 请求json数据
r = requests.post(url, headers=headers, data=post_data)

# 解码
json_str = r.content.decode()

# 把json格式字符串转换成python对象
json_dict = json.loads(json_str)

print(json_dict)

print("\n\n")

# 还可以使用json()方法, 免去了自己手动编解码的步骤
print(r.json())
```

> 来自: [3.2.结构化数据提取-json](https://www.yuque.com/tuling_python/spider_base/lu8kddubkgblvlnn)

### 3.3.练习：蜻蜓FM排行榜信息

蜻蜓FM的首页`url`：[https://m.qingting.fm](https://m.qingting.fm/rank)

代码示例：

```python
import requests

url = "https://webapi.qingting.fm/api/mobile/rank/hotSaleWeekly"

headers = {
    "User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1"
}

r = requests.get(url=url, headers=headers)
print(r.status_code)
print(r.json())
```

分析过程：

![](images/image-014.png)

> 来自: [3.3.练习：蜻蜓FM排行榜信息](https://www.yuque.com/tuling_python/spider_base/hdhsnsnd4ftbgaln)

### 3.4.xpath语法

**什么是`xpath`**

`XPath (XML Path Language)` 即`XML路径语言`，在最初时它主要在`xml`文档中查找需要的信息，而现在它也适用于`HTML`文档的搜索。

`W3School`官方文档：[http://www.w3school.com.cn/xpath/index.asp](http://www.w3school.com.cn/xpath/index.asp)

`xpath`可以很轻松的选择出想要的数据，提供了非常简单明了的路径选择表达式，几乎想要任何定位功能，`xpath`都可以很轻松的实现。所以在之后的静态网站数据提取中会经常使用`xpath`语法完成。

**`xpath`节点**

每个标签我们都称之为`节点`，其中最顶层的节点称为`根节点`。

![](images/image-015.png)

**辅助工具**

- `Chrome`浏览器插件： `XPath Helper`
- `Firefox`浏览器插件：`XPath Finder`

注意： 这些工具是用来学习`xpath`语法的，当熟练掌握`xpath`的语法后就可以直接在代码中编写`xpath`而不一定非要用此工具。

**语法规则**

`XPath`使用路径表达式来选取文档中的节点或者节点集。这些路径表达式和我们在常规的电脑文件系统中看到的表达式非常相似。

表达式

描述

`nodename`

选中该元素

`/`

从根节点选取、或者是元素和元素间的过渡

`//`

从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置

`.`

选取当前节点

`..`

选取当前节点的父节点

`@`

选取属性

`text()`

选取文本

路径表达式

路径表达式

结果

`/bookstore`

选取根元素`bookstore`。注释：假如路径起始于正斜杠(`/`)，则此路径始终代表到某元素的绝对路径

`bookstore/book`

选取属于`bookstore`之下的所有`book`元素

`//book`

选取所有`book`子元素，而不管它们在文档中的位置

`bookstore//book`

选择属于`bookstore`元素的后代的所有`book`元素，而不管它们位于`bookstore`之下的什么位置

`//book/title/@lang`

选择所有的`book`下面的`title`中的`lang`属性的值

`//book/title/text()`

选择所有的`book`下面的`title`的文本

查询特定节点

路径表达式

结果

`//title[@lang="eng"]`

选择`lang`属性值为`eng`的所有`title`元素

`/bookstore/book[1]`

选取属于`bookstore`子元素的第`1`个`book`元素

`/bookstore/book[last()]`

选取属于`bookstore`子元素的最后`1`个`book`元素

`/bookstore/book[last()-1]`

选取属于`bookstore`子元素的倒数第`2`个`book`元素

`/bookstore/book[position()>1]`

选择`bookstore`下面的`book`元素，从第`2`个开始选择

`/bookstore/book[position()>1 and position()
选择`bookstore`下面的`book`元素，从第`2`个开始取到第`4`个元素

`//book/title[text()='Harry Potter']`

选择所有`book`下的`title`元素，仅仅选择文本为`Harry Potter`的`title`元素

注意点: 在`xpath`中，第一个元素的位置是`1`，最后一个元素的位置是`last()`，倒数第二个是`last()-1`

**语法练习**

接下来对豆瓣电影`top250`的页面来练习上述语法：[https://movie.douban.com/top250](https://movie.douban.com/top250)

- 选择所有的`h1`下的文本

```plain
//h1/text()
```

- 获取电影信息的`href`属性

```plain
//div[@class='item']//div[@class='hd']/a/@href
```

- 获取电影的评价人数

```plain
//div[@class='star']/span[last()]/text()
```

**总结**

- `XPath`的概述：`XPath (XML Path Language)`，解析查找提取信息的语言
- `xml`是和服务器交互的数据格式和`json`的作用一致
- `html`是浏览器解析标签数据显示给用户
- `XPath`的重点语法获取任意节点:`//`
- `XPath`的重点语法根据属性获取节点:`标签[@属性='值']`
- `XPath`的获取节点属性值:`@属性值`
- `XPath`的获取节点文本值:`text()`

> 来自: [3.4.xpath语法](https://www.yuque.com/tuling_python/spider_base/yzxst29hi3lamgml)

### 3.5.使用lxml模块中的xpath语法提取非结构化数据

前面学习的`xpath`知识主要的作用是：学会怎样通过`xpath`语法找到需要的数据，想要在代码中使用`xpath`进行处理，就需要学习另外一个新的模块`lxml`

**模块安装**

```plain
pip install lxml -i https://pypi.tuna.tsinghua.edu.cn/simple
```

**`lxml`的使用**

- 使用`lxml`转化为`Element`对象

```python
from lxml import etree

text = '''

- [first item](link1.html)

- [second item](link2.html)

- [third item](link3.html)

- [fourth item](link4.html)

- [fifth item](link5.html)
          '''

# 利用etree.HTML，将字符串转化为Element对象, Element对象具有XPath的方法
html = etree.HTML(text)
print(type(html))

# 将Element对象转化为字符串
handled_html_str = etree.tostring(html).decode()
print(handled_html_str)
```
使用`lxml`中的`xpath`语法提取数据

提取`a`标签属性和文本

```python
from lxml import etree

text = '''

- [first item](link1.html)

- [second item](link2.html)

- [third item](link3.html)

- [fourth item](link4.html)

- [fifth item](link5.html)
          '''

html = etree.HTML(text)

# 获取href的列表和title的列表
href_list = html.xpath("//li[@class='item-1']/a/@href")
title_list = html.xpath("//li[@class='item-1']/a/text()")

for title, href in zip(title_list, href_list):
    item = dict()
    item["title"] = title
    item["href"] = href
    print(item)
```
以上代码必须确保标签中的数据是一一对应的，如果有些标签中不存在指定的属性或文本则会匹配混乱。
```python
from lxml import etree

text = '''
        first item

- [second item](link2.html)

- [third item](link3.html)

- [fourth item](link4.html)

- [fifth item](link5.html)
          '''

html = etree.HTML(text)

# 获取href的列表和title的列表
href_list = html.xpath("//li[@class='item-1']/a/@href")
title_list = html.xpath("//li[@class='item-1']/a/text()")

for title, href in zip(title_list, href_list):
    item = dict()
    item["title"] = title
    item["href"] = href
    print(item)
```
输出结果为：
```plain
/Users/poppies/python_envs/base/bin/python3 /Users/poppies/Documents/spider_code/1.py
{'title': 'first item', 'href': 'link2.html'}
{'title': 'second item', 'href': 'link4.html'}
```
`xpath`分次提取

前面我们取到属性，或者是文本的时候，返回字符串 但是如果我们取到的是一个节点，返回什么呢?

返回的是`element`对象，可以继续使用`xpath`方法

对此我们可以在后面的数据提取过程中：先根据某个`xpath`规则进行提取部分节点，然后再次使用`xpath`进行数据的提取

示例如下：

```python
from lxml import etree

text = '''

- first item

- [second item](link2.html)

- [third item](link3.html)

- [fourth item](link4.html)

- [fifth item](link5.html)
          '''

html = etree.HTML(text)

li_list = html.xpath("//li[@class='item-1']")
print(li_list)
```
可以发现结果是一个`element`对象，这个对象能够继续使用`xpath`方法。先根据`li`标签进行分组，之后再进行数据的提取。
```python
from lxml import etree

text = '''
        first item

- [second item](link2.html)

- [third item](link3.html)

- [fourth item](link4.html)

- [fifth item](link5.html)
          '''

html = etree.HTML(text)

li_list = html.xpath("//li[@class='item-1']")
print(li_list)

# 在每一组中继续进行数据的提取
for li in li_list:
    item = dict()
    item["href"] = li.xpath("./a/@href")[0] if len(li.xpath("./a/@href")) > 0 else None
    item["title"] = li.xpath("./a/text()")[0] if len(li.xpath("./a/text()")) > 0 else None
    print(item)
```
总结`lxml`库的安装：`pip install lxml`
- `lxml`的导包：`from lxml import etree`
- `lxml`转换解析类型的方法：`etree.HTML(text)`
- `lxml`解析数据的方法：`data.xpath("//div/text()")`
- 需要注意`lxml`提取完毕数据的数据类型都是列表类型
- 如果数据比较复杂：先提取大节点，然后再进行小节点操作

> 来自: [3.5.使用lxml模块中的xpath语法提取非结构化数据](https://www.yuque.com/tuling_python/spider_base/esp2rtirw0b92x6e)

### 3.6.练习：通过xpath提取豆瓣电影评论

要求

爬取豆瓣电影的评论，地址链接：[https://movie.douban.com/subject/1292052/comments?status=P](https://movie.douban.com/subject/1292052/comments?status=P)

提示

先在浏览器中使用插件`XPath Helper`进行`xpath`语法测试，效果如下：

![](images/image-016.png)

代码示例

```python
import requests
from lxml import etree

"""
流程分析：
1. 通过requests发送请求获取豆瓣返回的内容
2. 将返回的内容通过etree.HTML转换为Element对象
3. 对Element对象使用XPath提取数据
"""

# 1. 通过requests发送请求获取豆瓣返回的内容
url = "https://movie.douban.com/subject/1292052/comments?status=P"
headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36",
}
r = requests.get(url=url, headers=headers)

# print(r.text)

# 2. 将返回的内容通过etree.HTML转换为Element对象
html = etree.HTML(r.text)

# 3. 对Element对象使用XPath提取数据
comment_list = html.xpath('//span[@class="short"]/text()')

print("提取到的个数：", len(comment_list))

for comment in comment_list:
    print(comment)
    print('\n')
```

> 来自: [3.6.练习：通过xpath提取豆瓣电影评论](https://www.yuque.com/tuling_python/spider_base/ncpvvp3lylfvplan)

### 3.7.jsonpath模块

`JsonPath`是一种可以快速解析`json`数据的方式，`JsonPath`对于`JSON`来说，相当于`XPath`对于`XML`，`JsonPath`用来解析多层嵌套的`json`数据。

官网：[https://goessner.net/articles/JsonPath/](https://goessner.net/articles/JsonPath/)

想要在`Python`编程语言中使用`JsonPath`对`json`数据快速提取，需要安装`jsonpath`模块

```plain
pip install jsonpath -i https://pypi.tuna.tsinghua.edu.cn/simple
```

**`jsonpath`常用语法**

![](images/image-017.png)

**代码示例**

```python
import jsonpath

info = {
    "error_code": 0,
    "stu_info": [
        {
            "id": 2059,
            "name": "小白",
            "sex": "男",
            "age": 28,
            "addr": "河南省济源市北海大道xx号",
            "grade": "天蝎座",
            "phone": "1837830xxxx",
            "gold": 10896,
            "info": {
                "card": 12345678,
                "bank_name": '中国银行'
            }
        },
        {
            "id": 2067,
            "name": "小黑",
            "sex": "男",
            "age": 28,
            "addr": "河南省济源市北海大道xx号",
            "grade": "天蝎座",
            "phone": "87654321",
            "gold": 100
        }
    ]
}

"""
未使用jsonpath时，提取dict时的方式
"""

res = info["stu_info"][0]['name']  # 取某个学生姓名的原始方法:通过查找字典中的key以及list方法中的下标索引
print(res)  # 输出结果是：小白
res = info["stu_info"][1]['name']
print(res)  # 输出结果是：小黑

print("----------我是分割线----------")

"""
使用jsonpath时，提取dict时的方式
"""

res1 = jsonpath.jsonpath(info, '$.stu_info[0].name')  # $表示最外层的{}， . 表示子节点的意思
print(res1)  # 输出结果是list：['小白']
res2 = jsonpath.jsonpath(info, '$.stu_info[1].name')
print(res2)  # 输出结果是list：['小黑']

res3 = jsonpath.jsonpath(info, '$..name')  # 嵌套n层也能取到所有学生姓名信息,$表示最外层的{}，..表示模糊匹配
print(res3)  # 输出结果是list：['小白', '小黑']

res4 = jsonpath.jsonpath(info, '$..bank_name')
print(res4)  # 输出结果是list：['中国银行']
```

> 来自: [3.7.jsonpath模块](https://www.yuque.com/tuling_python/spider_base/sugd14zsaf3ggg0a)

### 3.8.练习：使用jsonpath提取数据

`jsonpath`对比`xpath`：

![](images/image-018.png)

练习代码：

```python
import jsonpath

info = {
    "store": {
        "book": [
            {"category": "reference",
             "author": "Nigel Rees",
             "title": "Sayings of the Century",
             "price": 8.95
             },
            {"category": "fiction",
             "author": "Evelyn Waugh",
             "title": "Sword of Honour",
             "price": 12.99
             },
            {"category": "fiction",
             "author": "Herman Melville",
             "title": "Moby Dick",
             "isbn": "0-553-21311-3",
             "price": 8.99
             },
            {"category": "fiction",
             "author": "J. R. R. Tolkien",
             "title": "The Lord of the Rings",
             "isbn": "0-395-19395-8",
             "price": 22.99
             }
        ],
        "bicycle": {
            "color": "red",
            "price": 19.95
        }
    }
}

# 1. 提取第1本书的title
print("\n1. 提取第1本书的title")
ret = jsonpath.jsonpath(info, "$.store.book[0].title")
print(ret)

ret = jsonpath.jsonpath(info, "$['store']['book'][0]['title']")
print(ret)

# 2. 提取2、3、4本书的标题
print("\n2. 提取2、3、4本书的标题")
ret = jsonpath.jsonpath(info, "$.store.book[1,2,3].title")
print(ret)
ret = jsonpath.jsonpath(info, "$.store.book[1,2,3]['title']")
print(ret)
ret = jsonpath.jsonpath(info, "$.store.book[1:4]['title']")
print(ret)

# 3. 提取1、3本书的标题
print("\n3. 提取1、3本书的标题")
ret = jsonpath.jsonpath(info, "$.store.book[::2].title")
print(ret)

# 4. 提取最后一本书的标题
print("\n4. 提取最后一本书的标题")
ret = jsonpath.jsonpath(info, "$.store.book[(@.length-1)].title")
print(ret)
ret = jsonpath.jsonpath(info, "$.store.book[-1:].title")
print(ret)

# 5. 提取价格小于10的书的标题
print("\n5. 提取价格小于10的书的标题")
ret = jsonpath.jsonpath(info, "$.store.book[?(@.price =5 && @.price10)]")
print(ret)

# 15. 获取所有的元素
print("\n15. 获取所有的元素")
ret = jsonpath(info, '$.*')  # 获取json本身并打印, 相当于print(info)
print(ret)

ret = jsonpath(info, '$..*')  # 获取json全部数据, 并且单独将value获取出来
for temp in res:
    print(temp)
```

> 来自: [3.8.练习：使用jsonpath提取数据](https://www.yuque.com/tuling_python/spider_base/xa8t35cnlnzk5p6t)

### 3.9.非结构化数据提取-bs4

**介绍与安装**

介绍

`BeautifulSoup4`简称`BS4`，和使用`lxml模块` 一样，`Beautiful Soup` 也是一个`HTML/XML`的解析器，主要的功能也是解析和提取`HTML/XML`数据。

`Beautiful Soup`是基于`HTML DOM`的，会载入整个文档，解析整个`DOM`树，因此时间和内存开销都会大很多，所以性能要低于`lxml模块`。

`BeautifulSoup`用来解析`HTML`比较简单，`API`非常人性化，支持`CSS`选择器、`Python`标准库中的`HTML`解析器，也支持`lxml模块`的`XML`解析器。

安装

```plain
pip install bs4 -i https://mirrors.aliyun.com/pypi/simple
```

官方文档：[http://beautifulsoup.readthedocs.io/zh_CN/v4.4.0](http://beautifulsoup.readthedocs.io/zh_CN/v4.4.0)

![](images/image-019.png)

**`bs4`基本使用示例**

```python
from bs4 import BeautifulSoup

html = """
The Dormouse's story

The Dormouse's story

Once upon a time there were three little sisters; and their names were
[Elsie](http://example.com/elsie),
[Lacie](http://example.com/lacie) and
[Tillie](http://example.com/tillie);
and they lived at the bottom of a well.

...

"""

# 创建 Beautiful Soup 对象
soup = BeautifulSoup(html, "lxml")

# 格式化输出html代码
print(soup.prettify())
```

**搜索文档树中的标签、内容、属性**

`find_all`方法中的参数

```python
def find_all(self, name=None, attrs={}, recursive=True, string=None, limit=None, **kwargs)...
```

`name`参数

当前参数可以传递标签名称字符串，根据传递的标签名称搜索对应标签

```python
# 1. 创建soup对象
soup = BeautifulSoup(html_obj, 'lxml')

# 2. 根据标签名称搜索标签
ret_1 = soup.find_all('b')
ret_2 = soup.find_all('a')

print(ret_1, ret_2)
```

除了传递标签名称字符串之外也可传递正则表达式，如果传入正则表达式作为参数，`Beautiful Soup`会通过正则表达式的 `match()`来匹配内容。下面例子中找出所有以`b`开头的标签。

```python
soup = BeautifulSoup(html_obj, 'lxml')
for tag in soup.find_all(re.compile('^b')):
    print(tag.name)
```

如果传递是一个列表，则`Beautiful Soup`会将与列表中任一元素匹配的内容返回。

```python
soup = BeautifulSoup(html_obj, 'lxml')
ret = soup.find_all(['a', 'b'])
print(ret)
```

`attrs`参数：可以根据标签属性搜索对应标签

```python
from bs4 import BeautifulSoup

html = """
The Dormouse's story

The Dormouse's story

Once upon a time there were three little sisters; and their names were
[Elsie](http://example.com/elsie),
[Lacie](http://example.com/lacie) and
[Tillie](http://example.com/tillie);
and they lived at the bottom of a well.

...

"""

soup = BeautifulSoup(html, "lxml")
ret_1 = soup.find_all(attrs={'class': 'sister'})
print(ret_1)

print('-' * 30)

# 简写方式
ret_2 = soup.find_all(class_='sister')
print(ret_2)

print('-' * 30)

# 查询id属性为link2的标签
ret_3 = soup.find_all(id='link2')
print(ret_3)
```

`string`参数：通过`string`参数可以搜索文档中的字符串内容，与`name`参数的可选值一样, `string`参数接受字符串 , 正则表达式 , 列表

```python
import re
from bs4 import BeautifulSoup

html = """
The Dormouse's story

The Dormouse's story

Once upon a time there were three little sisters; and their names were
[Elsie](http://example.com/elsie),
[Lacie](http://example.com/lacie) and
[Tillie](http://example.com/tillie);
and they lived at the bottom of a well.

...

"""

soup = BeautifulSoup(html, "lxml")

ret_1 = soup.find_all(string='Elsie')
print(ret_1)

ret_2 = soup.find_all(string=['Tillie', 'Elsie', 'Lacie'])
print(ret_2)

ret_3 = soup.find_all(string=re.compile('Dormouse'))
print(ret_3)
```

`find`方法

`find`的用法与`find_all`一样，区别在于`find`返回第一个符合匹配结果，`find_all`则返回所有匹配结果的列表

**文档搜索树中的`css`选择器**

另一种与`find_all`方法有异曲同工之妙的查找方法，也是返回所有匹配结果的列表。

`css`选择器编写注意事项：

- 标签名称不加任何修饰
- 类名前加`.`
- `id`属性名称前加`#`

`css`选择器编写方式与编写`css`样式表的语法大致相同。在`bs4`中可以直接使用`soup.select()`方法进行筛选，返回值类型是一个列表。

**标签选择器**

```python
import re
from bs4 import BeautifulSoup

html = """
The Dormouse's story

The Dormouse's story

Once upon a time there were three little sisters; and their names were
[Elsie](http://example.com/elsie),
[Lacie](http://example.com/lacie) and
[Tillie](http://example.com/tillie);
and they lived at the bottom of a well.

...

"""

soup = BeautifulSoup(html, "lxml")

print(soup.select('title'))
print(soup.select('a'))
print(soup.select('b'))
```

**类选择器**

```python
import re
from bs4 import BeautifulSoup

html = """
The Dormouse's story

The Dormouse's story

Once upon a time there were three little sisters; and their names were
[Elsie](http://example.com/elsie),
[Lacie](http://example.com/lacie) and
[Tillie](http://example.com/tillie);
and they lived at the bottom of a well.

...

"""

soup = BeautifulSoup(html, "lxml")

print(soup.select('.sister'))
```

**`id`选择器**

```python
import re
from bs4 import BeautifulSoup

html = """
The Dormouse's story

The Dormouse's story

Once upon a time there were three little sisters; and their names were
[Elsie](http://example.com/elsie),
[Lacie](http://example.com/lacie) and
[Tillie](http://example.com/tillie);
and they lived at the bottom of a well.

...

"""

soup = BeautifulSoup(html, "lxml")

print(soup.select('#link1'))
```

**层级选择器**

```python
import re
from bs4 import BeautifulSoup

html = """
The Dormouse's story

The Dormouse's story

Once upon a time there were three little sisters; and their names were
[Elsie](http://example.com/elsie),
[Lacie](http://example.com/lacie) and
[Tillie](http://example.com/tillie);
and they lived at the bottom of a well.

...

"""

soup = BeautifulSoup(html, "lxml")

print(soup.select('p #link1'))
```

**属性选择器**

```python
import re
from bs4 import BeautifulSoup

html = """
The Dormouse's story

The Dormouse's story

Once upon a time there were three little sisters; and their names were
[Elsie](http://example.com/elsie),
[Lacie](http://example.com/lacie) and
[Tillie](http://example.com/tillie);
and they lived at the bottom of a well.

...

"""

soup = BeautifulSoup(html, "lxml")

print(soup.select('a[class="sister"]'))
print('-' * 30)
print(soup.select('a[href="http://example.com/elsie"]'))
```

**`get_text()`方法：获取文本内容**

```python
import re
from bs4 import BeautifulSoup

html = """
The Dormouse's story

The Dormouse's story

Once upon a time there were three little sisters; and their names were
[Elsie](http://example.com/elsie),
[Lacie](http://example.com/lacie) and
[Tillie](http://example.com/tillie);
and they lived at the bottom of a well.

...

"""

soup = BeautifulSoup(html, "lxml")

# select返回的是列表对象, 需要使用for循环遍历列表元素再使用get_text方法获取文本数据
for title in soup.select('title'):
    print(title.get_text())
```

**`get()`方法：获取属性**

```python
import re
from bs4 import BeautifulSoup

html = """
The Dormouse's story

The Dormouse's story

Once upon a time there were three little sisters; and their names were
[Elsie](http://example.com/elsie),
[Lacie](http://example.com/lacie) and
[Tillie](http://example.com/tillie);
and they lived at the bottom of a well.

...

"""

soup = BeautifulSoup(html, "lxml")

for attr in soup.select('a'):
    print(attr.get('href'))
```

**总结**

- 安装`beautifulsoup4`:`pip install bs4`
- `beautifulsoup`导包: `from bs4 import BeautifulSoup`
- `beautifulsoup`转换类型: `BeautifulSoup(html)`
- `find`方法返回一个解析完毕的对象
- `findall`方法返回的是解析列表`list`
- `select`方法返回的是解析列表`list`
- 获取属性的方法: `get('属性名字')`
- 和获取文本的方法: `get_text()`

> 来自: [3.9.非结构化数据提取-bs4](https://www.yuque.com/tuling_python/spider_base/oux4t2930pucvkok)

### 3.10.练习：使用bs4抓取搜狗微信下的所有文章标题

```python
import requests
from bs4 import BeautifulSoup

url = "https://weixin.sogou.com/weixin?_sug_type_=1&type=2&query=python"

headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36"
}

response = requests.get(url, headers=headers).text
soup = BeautifulSoup(response, 'lxml')
ul_tag = soup.select('ul[class="news-list"]')
# print(ul_tag)

h3_list = ul_tag[0].select('h3')
for temp in h3_list:
    print(temp.select('a')[0].get_text(), temp.select('a')[0].get('href'))
    print('-' * 30)
```

> 来自: [3.10.练习：使用bs4抓取搜狗微信下的所有文章标题](https://www.yuque.com/tuling_python/spider_base/iwarsu7rmkalsc5x)

## 4.正则表达式

正则表达式`Regular Expression`，通常缩写为`RegExp`或`Regex`，是一种用于匹配、搜索和操作文本字符串的强大工具。它是由模式`pattern`和相关的匹配规则组成的表达式。

正则表达式的模式由一系列字符和特殊字符组成，用于描述所需匹配的文本模式。它可以用于各种编程语言和文本编辑器中，例如`Python`、`JavaScript`、`Perl`等。

正则表达式提供了一种灵活的方式来匹配和处理字符串。它可以用于以下情况：

- 搜索和替换：可以使用正则表达式来搜索文本中符合特定模式的字符串，并进行替换或其他操作。
- 验证数据：可以使用正则表达式来验证用户输入的数据是否符合特定的格式要求，例如验证电子邮件地址、电话号码、日期等。
- 提取信息：可以使用正则表达式从文本中提取特定的信息，例如提取`URL`、提取网页中的所有链接等。

正则表达式中的特殊字符和语法规则很多，包括字符类`character class`、量词`quantifier`、分组`grouping`、转义字符`escape character`等。这些特殊字符和规则可以组合使用，构成复杂的匹配模式。

> 来自: [4.正则表达式](https://www.yuque.com/tuling_python/spider_base/qx5ovyg23nctx8vf)

### 4.1.正则表达式的作用

**案例演示**

先给大家看一个例子，在以下文本中存储了一些职位信息：

```plain
Python3 高级开发工程师 上海互教教育科技有限公司上海-浦东新区2万/月02-18满员
测试开发工程师（C++/python） 上海墨鹍数码科技有限公司上海-浦东新区2.5万/每月02-18未满员
Python3 开发工程师 上海德拓信息技术股份有限公司上海-徐汇区1.3万/每月02-18剩余11人
测试开发工程师（Python） 赫里普（上海）信息科技有限公司上海-浦东新区1.1万/每月02-18剩余5人
Python高级开发工程师 上海行动教育科技股份有限公司上海-闵行区2.8万/月02-18剩余255人
python开发工程师 上海优似腾软件开发有限公司上海-浦东新区2.5万/每月02-18满员
```

将文本中的薪资数据提取出来，只要包含数字就可以。

代码实现

```python
import re

content = '''
Python3 高级开发工程师 上海互教教育科技有限公司上海-浦东新区2万/月02-18满员
测试开发工程师（C++/python） 上海墨鹍数码科技有限公司上海-浦东新区2.5万/每月02-18未满员
Python3 开发工程师 上海德拓信息技术股份有限公司上海-徐汇区1.3万/每月02-18剩余11人
测试开发工程师（Python） 赫里普（上海）信息科技有限公司上海-浦东新区1.1万/每月02-18剩余5人
Python高级开发工程师 上海行动教育科技股份有限公司上海-闵行区2.8万/月02-18剩余255人
python开发工程师 上海优似腾软件开发有限公司上海-浦东新区2.5万/每月02-18满员
'''

for temp in re.findall(r'([\d.]+)万/每{0,1}月', content):
    print(temp)
```

通过以上代码就可以轻松的将文本中的数字提取出来，在`find_all`方法中的字符串其实就是正则表达式。观察当前方法返回的数据我们发现是一个列表。

> 来自: [4.1.正则表达式的作用](https://www.yuque.com/tuling_python/spider_base/fqog22hv5py9s5m4)

### 4.2.正则表达式在线验证工具

工具链接地址：[https://regexr-cn.com](https://regexr-cn.com)

在这个工具中我们可以快速验证自己编写的正则表达式是否存在语法错误。

![](images/image-020.png)

> 来自: [4.2.正则表达式在线验证工具](https://www.yuque.com/tuling_python/spider_base/qn5rcdpalh5a6zny)

### 4.3.常见语法

**普通字符匹配**

可以在正则表达式中直接输入我们想要匹配的字符，如图所示：

![](images/image-021.png)

当然直接查询汉字也是可以的。但是有些特殊字符不能直接匹配，这些特殊字符有专业术语：元字符。

元字符具有特殊含义，如下所示：

```plain
. * + ? \ [] ^ $ {} | ()
```

**通配符：`.`**

在以下文本中选出所有的颜色信息：

```plain
苹果是绿色的
橙子是橙色的
香蕉是黄色的
乌鸦是黑色的
```

在文本中找到以`色`结尾，并且包括前面一个字符的信息，那么正则表达式就可以写成：

```plain
.色
```

当前`.`代表任意字符，但是字符个数只有一个。`色`这个汉字代表以这个汉字结尾。

![](images/image-022.png)

代码实现

```python
import re

content = '''
苹果是绿色的
橙子是橙色的
香蕉是黄色的
乌鸦是黑色的
'''

for temp in re.findall(r'.色', content):
    print(temp)
```

**重复匹配任意次数：`*`**

`*`表示匹配子表达式任意次，包括`0`次。

在以下文本中匹配逗号后面的字符串内容，包含逗号本身：文本中的逗号为中文。

```plain
苹果，是绿色的
橙子，是橙色的
香蕉，是黄色的
乌鸦，是黑色的
猴子，
```

表达式语法：

```plain
，.*
```

效果如下：

![](images/image-023.png)

大家注意最后一行，猴子逗号后面没有其它字符了，但是`*`表示可以匹配`0`次， 所以表达式也是成立的。

代码实现

```python
content = '''
苹果，是绿色的
橙子，是橙色的
香蕉，是黄色的
乌鸦，是黑色的
猴子，
'''

for temp in re.findall(r'，.*', content):
    print(temp)
```

`.*`在正则表达式中非常常见，表示匹配任意字符任意次数。当然，`*`前面不一定就是`.`，也可以是其他字符。

![](images/image-024.png)

**重复匹配一次或多次：`+`**

`+`表示匹配前面的子表达式一次或多次，不包括`0`次。

以之前的文本为例，匹配所有逗号的内容，包含逗号。但是如果逗号后没有内容则不匹配。

![](images/image-025.png)

表达式语法：

```plain
，.+
```

**匹配`0`次或者一次：`?`**

以之前的文本为例，在文本中匹配每行逗号后面的`1`个字符，也包含逗号本身。

![](images/image-026.png)

表达式语法：

```plain
，.?
```

**匹配执行次数：`{}`**

`{}`表示指定字符匹配的次数。

测试文本：

```plain
红彤彤，绿油油，黑乎乎，绿油油油油
```

- 表达式`油{3}`就表示匹配连续的`油`字`3`次
- 表达式`油{3,4}`就表示匹配连续的`油`字至少`3`次，至多`4`次

![](images/image-027.png)

**贪婪模式与非贪婪模式**

将以下字符串中的所有`html`标签提取出来：

```html
Title
```

根据之前所学习的内容，我们可以使用``将标签进行匹配，代码如下：

```python
import re

source = 'Title'

for temp in re.findall(r'', source):
    print(temp)
```

运行效果如下：

```plain
Title
```

当前结果除了标签之外的文本数据也一起提取了。

解决方式

在正则表达式中，`*`、`+`等元字符都是贪婪的。使用它们时会尽可能匹配多的内容，所以在``表达式中的`*`一直匹配到了字符串最后的``中的`e`，解决这个问题的方式就是将贪婪模式更改为非贪婪模式。在`*`后加上`?`。

语法如下：

```plain

```

代码实现：

```python
import re

source = 'Title'

for temp in re.findall(r'', source):
    print(temp)
```

非贪婪模式在匹配到第一个``停止，之后继续匹配下一个符合条件的字符串。

**元字符转义**

反斜杠`\`在正则表达式中有多重用途，例如在以下文本中搜索所有`.`之前的字符串，也包含`.`本身。

```plain
苹果.是绿色的
橙子.是橙色的
香蕉.是黄色的
```

如果我们将正则表达式写成：`.*.`，肯定是不正确的，因为`.`是一个元字符，具有特殊含义。直接出现在正则表达式中不能表示`.`这个字符本身。

解决方式：使用`\`转义。

`Python`程序如下：

```python
import re

content = '''
苹果.是绿色的
橙子.是橙色的
香蕉.是黄色的
'''

for temp in re.findall(r'.*\.', content):
    print(temp)
```

![](images/image-028.png)

**匹配某种字符类型**

在反斜杠后链接一些字符可以表示某种类型的一个字符。

- `\d`匹配`0-9`之间任意一个数字字符，等价于表达式：`[0-9]`
- `\D`匹配任意一个不是`0-9`之间的数字字符，等价于表达式：`[^0-9]`
- `\s`匹配任意一个空白字符，包括空格、`tab`、换行符等，等价于表达式：`[\t\n\r\f\v]`
- `\S`匹配任意一个非空白字符，等价于表达式：`[^\t\n\r\f\v]`
- `\w`匹配任意一个文字字符，包括大小写字母、数字、下划线，等价于表达式：`[a-zA-Z0-9_]`
- `\W`匹配任意一个非文字字符，等价于表达式：`[^a-zA-Z0-9_]`

反斜杠也可以用在方括号里面，比如`[\s,.]`表示匹配 ： 任何空白字符， 或者逗号，或者点

**使用中括号匹配指定字符范围：`[]`**

中括号表示要匹配的几个指定字符之一。

`[abc]`可以匹配`a、b、c`中任意一个字符，等价于：`[a-c]`

`[a-c]`中间的`-`表示范围从`a`到`c`，如果你想匹配所有的小写字母，可以使用：`[a-z]`

一些元字符在中括号内会失去特殊含义，和普通字符没有区别。例如：`[akm.]`，在当前正则中的`.`只是一个普通字符而已，并不表示匹配任意字符。

如果在中括号中使用`^`则表示非，不匹配在中括号中的字符集合。

```python
import re

content = 'a1b2c3d4e5'
for temp in re.findall(r'[^\d]', content):
    print(temp)
```

**起始结尾位置与多行模式**

起始位置

`^`表示匹配文本的开头位置。在正则表达式中可以设置单行模式与多行模式。

- 单行模式：表示匹配整个文本的开头位置
- 多行模式：表示匹配文本每行的开头位置

在下面的文本中，每行最前面的数字表示水果的编号，最后的数字表示价格。

```plain
001-苹果价格-60
002-橙子价格-70
003-香蕉价格-80
```

如果我们要提取所有的水果编号，用这样的正则表达式：`^\d+`

代码示例：

```python
import re

content = '''001-苹果价格-60
002-橙子价格-70
003-香蕉价格-80
'''

for temp in re.findall(r'^\d+', content, re.M):
    print(temp)
```

在以上代码的`findall`方法中，第三个参数`re.M`表示使用多行模式。

运行结果如下：

```plain
001
002
003
```

如果去掉第三个参数则运行结果如下：

```plain
001
```

结尾位置

`$`表示匹配文本的`结尾`位置。

将之前文本中所有的水果价格提取出来，可以使用这样的表达式：`\d+$`

注意：在结尾匹配也有单行与多行的区别。

代码示例：

```python
import re

content = '''001-苹果价格-60
002-橙子价格-70
003-香蕉价格-80
'''

for temp in re.findall(r'\d+$', content, re.M):
    print(temp)
```

运行结果如下：

```plain
60
70
80
```

如果将`re.MULTILINE`去掉则只能匹配最后一行。

结果如下：

```plain
80
```

单行模式下，`$`只会匹配整个文本的结束位置。

**匹配指定多个字符中的其中之一 - `|`**

竖线表示匹配其中之一，示例如下：

![](images/image-029.png)

特别要注意的是竖线在正则表达式的优先级是最低的，这就意味着竖线隔开的部分是一个整体。比如绿色|橙表示要匹配是 绿色或者橙 ，而不是绿色或者绿橙。

**分组 - `()`**

小括号是正则表达式的组选择。`组`就是把正则表达式匹配的内容中的一部分标记为某个组。

我们可以在正则表达式中标记多个`组`。

使用之前的文本案例，从以下文本中提取逗号前面的字符串，包含逗号。

```plain
苹果，苹果是绿色的
橙子，橙子是橙色的
香蕉，香蕉是黄色的
```

以上案例可以使用：`^.*，`来完成，但是如果要求不要包含逗号呢？可能有同学会写成：`^.*`来完成，这种写法无法满足需求，因为逗号是结尾特征。

解决方式

使用小括号分组，并去除逗号。

语法示例：

```plain
^(.*)，
```

`Python`代码如下：

```python
import re

content = '''苹果，苹果是绿色的
橙子，橙子是橙色的
香蕉，香蕉是黄色的'''

for temp in re.findall(r'^(.*)，', content, re.M):
    print(temp)
```

`分组`还可以多次使用，例如在以下文本中提取每个人的名字以及对应的手机号：

```plain
张三，手机号码15945678901
李四，手机号码13945677701
王二，手机号码13845666901
```

代码如下：

```python
import re

content = '''张三，手机号码15945678901
李四，手机号码13945677701
王二，手机号码13845666901'''

for temp in re.findall(r'^(.+)，.+(\d{11})', content, re.M):
    print(temp)
```

当有多个分组的时候可以使用`(?P...)`这样的格式给每个分组命名，这样做的好处是方便之后的代码提取每个指定分组中的内容。

```python
import re

content = '''张三，手机号码15945678901
李四，手机号码13945677701
王二，手机号码13845666901'''

for temp in re.finditer(r'^(?P.+)，.+(?P\d{11})', content, re.M):
    print(temp.group('user_name'), temp.group('mobile'))
```

**`DOTALL`参数 - 标记允许点号匹配所有字符**

`DOTALL`标记允许点号匹配所有字符，包括换行符。这对于需要处理包含换行符的文本时非常有用，因为默认情况下点号无法匹配换行符。

需求解决

在`html`代码中提取所有的职位名称：

```html

Python开发工程师

        南京
        1.5-2万/月

java开发工程师

        苏州
        1.5-2/月

```

如果直接使用表达式`class=\"t1\">.*?(.*?)`会发现匹配不上，因为`t1`和``之间有两个空行，这时就需要`.`匹配所有字符了。

代码示例：

```python
import re

content = '''

Python开发工程师

        南京
        1.5-2万/月

java开发工程师

        苏州
        1.5-2/月

'''

for temp in re.findall(r'class=\"t1\">.*?(.*?)', content, re.DOTALL):
    print(temp)
```

> 来自: [4.3.常见语法](https://www.yuque.com/tuling_python/spider_base/xlnoxb7uvfsxq2mc)

### 4.4.常用字符串处理方式

**正则字符串切割**

`Python`中的字符串切割

字符串对象中的`split`方法只适用于简单的字符串切割，有时你需要更加灵活的字符串切割。

在以下字符串中提取武将名称：

```python
names = '关羽; 张飞, 赵云,马超, 黄忠  李逵'
```

我们发现这些名字之间有些是`;`分割，有些是`,`分割，还有一些是空格分割，并且空格数量是不一致的。这时使用字符串对象中的分割方法不好处理。

正则表达式中的`split`方法

使用`re.split`方法完成字符串提取：

```python
import re

names = '关羽; 张飞, 赵云,   马超, 黄忠  李逵'

name_list = re.split(r'[;,\s]\s*', names)
print(name_list)
```

**字符串替换**

匹配模式替换

字符串对象中的`replace`方法只适应于简单的替换，有时你需要更加灵活的字符串替换。

比如我们需要在以下文本中找到所有以`av`开头的所有链接：`\avxxxxxx\`，然后将这些字符串替换为`/cn345677/`。

```python
html_obj = '''

下面是这学期要学习的课程：

点击这里，边看视频讲解，边学习以下内容
这节讲的是牛顿第2运动定律

点击这里，边看视频讲解，边学习以下内容
这节讲的是毕达哥拉斯公式

点击这里，边看视频讲解，边学习以下内容
这节讲的是切割磁力线
'''
```

被替换的内容不是固定的，所以无法使用字符串中的`replace`方法。

使用正则表达式中的`sub`方法

```python
import re

html_obj = '''

下面是这学期要学习的课程：

点击这里，边看视频讲解，边学习以下内容
这节讲的是牛顿第2运动定律

点击这里，边看视频讲解，边学习以下内容
这节讲的是毕达哥拉斯公式

点击这里，边看视频讲解，边学习以下内容
这节讲的是切割磁力线
'''

data = re.sub(r'/av\d+/', '/cn345677/', html_obj)
print(data)
```

`sub`方法是正则表达式中的替换方法，替换的内容是用正则表达式匹配出来的内容。

当前第一个参数`r'/av\d+/'`是一个正则表达式，表示以`/av`开头，后面是一串数字，在以`/`结尾的这种特征的字符串是需要被替换的。

第二个参数`/cn345677/`是替换的结果。

第三个参数是原来的字符串。

指定替换函数

在刚刚的例子中，我们用来替换的是一个固定的字符串`/cn345677/`。如果我们要求替换的内容是原来的数字加上数字`6`的结果，例如：`/av66771949/`替换为`/av66771955/`。如何实现？

这种更加复杂的替换，我们可以把`sub`中的第二个参数指定为一个函数，该函数的返回值就是用来替换的字符串。

代码如下：

```python
import re

names = '''

下面是这学期要学习的课程：

点击这里，边看视频讲解，边学习以下内容
这节讲的是牛顿第2运动定律

点击这里，边看视频讲解，边学习以下内容
这节讲的是毕达哥拉斯公式

点击这里，边看视频讲解，边学习以下内容
这节讲的是切割磁力线
'''

# 替换函数，参数是Match对象
def sub_func(match):
    # Match对象的group(0)返回的是整个匹配上的字符串，
    src = match.group(0)

    # Match对象的group(1)返回的是第一个group分组的内容
    number = int(match.group(1)) + 6
    dest = f'/av{number}/'

    print(f'{src} 替换为 {dest}')

    # 返回值就是最终替换的字符串
    return dest

data = re.sub(r'/av(\d+)/', sub_func, names)
print(data)
```

> 来自: [4.4.常用字符串处理方式](https://www.yuque.com/tuling_python/spider_base/dydvtukw7kgrdzwl)

## 5.数据存储

在爬虫项目当中，我们需要将目标站点数据数据进行持久化保存，一般数据保存的方式有两种：

- 文件保存
- 数据库保存

在数据保存的过程中需要对数据完成去重操作，所以在本章节中还需要使用`redis`中的`set`数据类型完成去重。

在本章节内容中因为需要和数据库交互，请同学们提前配置好数据库连接包：

```shell
pip install pymysql
pip install redis
pip install pymongo
pip install DBUtils  # 数据库连接池
```

> 来自: [5.数据存储](https://www.yuque.com/tuling_python/spider_base/eh8gm6i6gnkzds2g)

### 5.1.CSV文件存储

**什么是`csv`**

通俗直白的说：就是一个普通文件，里面的内容是每一行中的数据用逗号分隔，然后文件后缀为`csv`

![](images/image-030.png)

**`Python`对`csv`文件进行读写操作**

写入列表数据到`csv`文件

```python
import csv

headers = ['班级', '姓名', '性别', '手机号', 'QQ']

rows = [
    ["18级Python", '小王', '男', '13146060xx1', '123456xx1'],
    ["18级Python", '小李', '男', '13146060xx2', '123456xx2'],
    ["19级Python", '小赵', '女', '13146060xx3', '123456xx3'],
    ["19级Python", '小红', '女', '13146060xx4', '123456xx4'],
]

with open('test_1.csv', 'w') as f:
    # 创建一个csv的writer对象，这样才能够将写入csv格式数据到这个文件
    f_csv = csv.writer(f)
    # 写入一行（我们用第一行当做表头）
    f_csv.writerow(headers)
    # 写入多行（当做数据）
    f_csv.writerows(rows)
```

写入字典数据到`csv`文件

```python
import csv

rows = [
    {
        "class_name": "18级Python",
        "name": '小王',
        "gender": '男',
        "phone": '13146060xx1',
        "qq": '123456xx1'
    },
    {
        "class_name": "18级Python",
        "name": '小李',
        "gender": '男',
        "phone": '13146060xx2',
        "qq": '123456xx2'
    },
    {
        "class_name": "19级Python",
        "name": '小赵',
        "gender": '女',
        "phone": '13146060xx3',
        "qq": '123456xx3'
    },
    {
        "class_name": "19级Python",
        "name": '小红',
        "gender": '女',
        "phone": '13146060xx4',
        "qq": '123456xx4'
    },
]

with open('test_2.csv', 'w') as f:
    # 创建一个csv的DictWriter对象，这样才能够将写入csv格式数据到这个文件
    f_csv = csv.DictWriter(f, ['class_name', 'name', 'gender', 'phone', 'qq'])
    # 写入一行（我们用第一行当做表头）
    f_csv.writeheader()
    # 写入多行（当做数据）
    f_csv.writerows(rows)
```

读取`csv`文件

```python
import csv

with open('test_1.csv') as f:
    # 创建一个reader对象，迭代时能够提取到每一行（包括表头）
    f_csv = csv.reader(f)
    for row in f_csv:
        print(type(row), row)
```

读取`csv`文件内容并封装为字典

```python
import csv

with open('test_2.csv') as f:
    # 创建一个reader对象，迭代时能够提取到每一行（包括表头）
    f_csv = csv.DictReader(f)
    for row in f_csv:
        # print(type(row), row)
        print(row.get("class_name"), row.get("name"), row.get("gender"), row.get("phone"), row.get("qq"))
```

**爬虫案例 - B站数据采集**

目标网站地址：[https://search.bilibili.com/video?keyword=美女&from_source=webtop_search&spm_id_from=333.1007&search_source=5](https://search.bilibili.com/video?keyword=%E7%BE%8E%E5%A5%B3&from_source=webtop_search&spm_id_from=333.1007&search_source=5)

```python
import csv
import requests

class SaveVideoInfo:
    def __init__(self):
        self.api_url = 'https://api.bilibili.com/x/web-interface/wbi/search/type?context=&search_type=video&page={}&order=&keyword=%E7%BE%8E%E5%A5%B3&duration=&category_id=&tids_1=&tids_2=&__refresh__=true&_extra=&highlight=1&single_column=0&web_location=1430654&w_rid=3035c5c80e8a24b01a6685e0d276c5ff&wts=1698906392'
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36',
            'Cookie': "buvid3=24B2D828-680C-AC77-E787-13669953D2F063538infoc; b_nut=1697620963; i-wanna-go-back=-1; b_ut=7; _uuid=3FCB9672-AD35-B791-34EB-FBB394C1055DF61815infoc; enable_web_push=DISABLE; home_feed_column=5; buvid4=C4202519-7247-E9C8-E20C-E27302EDE53564679-023101817-STZcpyLLRzNibWQwQhkmXw%3D%3D; buvid_fp=77df88e74294ee94582ad08cb5f71cf4; CURRENT_FNVAL=4048; rpdid=|(kmJY|k~u~u0J'uYm~Yk|m)k; header_theme_version=CLOSE; SESSDATA=486a3b8a%2C1713507656%2C2602c%2Aa1CjADLWhb8hOljLOeLuedVVT6dmPfFDlTTryt7ZuXcTQacp6C-HeRFrNK59oZVhcUxtISVkRFWXNpNDhLbzZUUVNyT0xpNS1TaFJCYUx0NWQzNm4taDhva3hjM1EzTmZpc3Myc2gtdHZNTkNEYTFrdzZqSWxFcURoSjY1djZpVEN2V3JwaEFjdnBBIIEC; bili_jct=996967c0ccbec984e13ab8ccbfcf01cc; DedeUserID=508205460; DedeUserID__ckMd5=73fc57c2f075cc42; browser_resolution=1920-853; CURRENT_QUALITY=120; b_lsid=4D97C954_18B8EB2AAE2; bili_ticket=eyJhbGciOiJIUzI1NiIsImtpZCI6InMwMyIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2OTkxNjUzOTEsImlhdCI6MTY5ODkwNjEzMSwicGx0IjotMX0.NFeZiUClRc6JlUBJR0GFIPUKH4nY3fBqDZbOPGGsLgU; bili_ticket_expires=1699165331; sid=7i7fya69; PVID=2"
        }

    def save(self):
        with open('video_info.csv', 'a', encoding='utf-8', newline='') as f:
            field_names = ['author', 'arcurl', 'tag']
            f_csv = csv.DictWriter(f, fieldnames=field_names)
            f_csv.writeheader()
            for page in range(1, 6):
                response = requests.get(self.api_url.format(page), headers=self.headers).json()
                for result in response['data']['result']:
                    item = dict()
                    item['author'] = result['author']
                    item['arcurl'] = result['arcurl']
                    item['tag'] = result['tag']
                    print(item)
                    f_csv.writerow(item)
```

> 来自: [5.1.CSV文件存储](https://www.yuque.com/tuling_python/spider_base/vna1xv4laed7cdq2)

### 5.2.JSON文件存储

**`json`数据格式介绍**

`JSON`全称为`JavaScript Object Notation`， 也就是`JavaScript`对象标记，它通过对象和数组的组合来表示数据，构造简洁但是结构化程度非常高，是一种轻量级的数据交换格式。本节中，我们就来了解如何利用`Python`保存数据到`JSON`文件。

常见的`json`数据格式如下：

```json
[
    {
		"name": "Bob",
		"gender": "male",
		"birthday": "1992-10-18"
	},
 	{
		"name": "Selina",
		"gender": "female",
		"birthday": "1995-10-18"
	}
]
```

由中括号包围的就相当于列表类型，列表中的每个元素可以是任意类型，这个示例中它是字典类型，由大括号包围。

`json`可以由以上两种形式自由组合而成，可以无限次嵌套，结构清晰，是数据交换的极佳方式。

**`python`中的`json`模块**

方法

作用

`json.dumps()`

把`python`对象转换成`json`对象，生成的是字符串。

`json.dump()`

用于将`dict`类型的数据转成`str`，并写入到`json`文件中

`json.loads()`

将`json`字符串解码成`python`对象

`json.load()`

用于从`json`文件中读取数据。

**爬虫案例 - 4399网站游戏信息采集**

目标地址：[https://www.4399.com/flash/](https://www.4399.com/flash/)

```python
import json
import requests
from lxml import etree

url = 'https://www.4399.com/flash/'

response = requests.get(url)
html = etree.HTML(response.content.decode('gbk'))
a_list = html.xpath('//ul[@class="n-game cf"]/li/a')
data_list = []
for a in a_list:
    item = dict()
    item['href'] = a.xpath('./@href')[0]
    item['title'] = a.xpath('./b/text()')[0]
    data_list.append(item)

with open('data.json', 'w', encoding='utf-8') as f:
    # f.write(json.dumps(data_list))
    # 禁止ascii编码
    f.write(json.dumps(data_list, indent=2, ensure_ascii=False))
```

在写入过程中如果没有指定`indent`则写入的数据没有缩进，如果没有指定`ensure_ascii`参数则无法显示中文。

> 来自: [5.2.JSON文件存储](https://www.yuque.com/tuling_python/spider_base/ggb218xgapl895yf)

### 5.3.MySQL数据库存储

在大多数爬虫项目中，普遍还是将清洗后的数据存储到`MySQL`或者`MongoDB`中。接下来我们通过某讯招聘抓取案例来完成对数据的入库操作。

**环境准备**

在完成案例之前，我们需要做好准备工作：

- 安装`pymysql`

```python
pip install pymysql -i https://pypi.douban.com/simple
```

- 创建数据库

```sql
create database py_spider charset=utf8;
```

**代码示例**

网站地址：[https://careers.tencent.com/search.html?keyword=python&query=at_1](https://careers.tencent.com/search.html?keyword=python&query=at_1)

```python
import pymysql
import requests

class TxWork:
    url = "https://careers.tencent.com/tencentcareer/api/post/Query?timestamp=1727013372560&countryId=&cityId=&bgIds=&productId=&categoryId=&parentCategoryId=&attrId=1&keyword=python&pageIndex=1&pageSize=10&language=zh-cn&area=cn"

    headers = {
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) '
                      'AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36'
    }

    def __init__(self):
        self.db = pymysql.connect(host='localhost', user='root', password='root', db='py_spider')
        self.cursor = self.db.cursor()

    @classmethod
    def get_work_info(cls):
        for page in range(1, 46):
            response = requests.get(cls.url.format(page), headers=cls.headers).json()
            print(f'正在抓取第{page}页')
            work_list = response['Data']['Posts']
            yield work_list

    def create_table(self):
        sql = """
            create table if not exists tx_work(
                id int primary key auto_increment,
                work_name varchar(100) not null,
                country_name varchar(50),
                city_name varchar(50),
                work_desc text
            );
        """
        try:
            self.cursor.execute(sql)
            print('数据表创建成功...')
        except Exception as e:
            print('数据表创建失败: ', e)

    def insert_work_info(self, *args):
        """
        :param args:
            id
            work_name
            country_name
            city_name
            work_desc
        :return:
        """
        sql = """
            insert into tx_work(
                id,
                work_name,
                country_name,
                city_name,
                work_desc
            ) values (%s, %s, %s, %s, %s);
        """

        try:
            self.cursor.execute(sql, args)
            self.db.commit()
            print('数据插入成功...')
        except Exception as e:
            print('数据插入失败: ', e)
            self.db.rollback()

    def main(self):
        self.create_table()
        all_work_generator_object = self.get_work_info()
        work_id = 0
        for work_info_list in all_work_generator_object:
            if work_info_list is not None:
                for work_info in work_info_list:
                    print(work_info)
                    work_name = work_info['RecruitPostName']
                    country_name = work_info['CountryName']
                    city_name = work_info['LocationName']
                    work_desc = work_info['Responsibility']
                    self.insert_work_info(work_id, work_name, country_name, city_name, work_desc)
            else:
                print('数据为空:', work_info_list)
                continue
        # 任务完成后关闭数据库链接
        self.db.close()

if __name__ == '__main__':
    tx_work = TxWork()
    tx_work.main()
```

**作业**

获取指定网站中的数据并保存到数据库，目标站点：`https://talent.taotian.com/off-campus/position-list?lang=zh`

> 来自: [5.3.MySQL数据库存储](https://www.yuque.com/tuling_python/spider_base/rvz0a63gg31m1ugf)

### 5.4.MySQL数据库连接池

在处理`python web`开发或者其他需要频繁进行数据库操作的项目时，重复的打开和关闭数据库连接既耗费时间也浪费资源。为了解决这个问题我们采用数据库连接池的方式复用已经创建好的连接对象，从而无需频繁的开启连接和关闭连接。

**`DBUtils`的简单使用**

如需使用数据库连接池首先需要安装第三方模块：`DBUtils`

安装指令

```bash
# 在安装数据库连接池之前，必须确保pymysql已经安装成功
pip install DBUtils
```

导入模块

```python
from dbutils.pooled_db import PooledDB
```

创建数据库连接池

使用`PooledDB`创建数据库连接池，连接池使用了一种新的`DB-API`连接方式，可以维护活动连接的池。当需要数据库连接时，直接从池中获取连接对象。完成操作后，将无需使用的连接对象返回到池中。无需频繁的关闭和开启连接。

```python
pool = PooledDB(
    creator=pymysql,  # 使用链接数据库的模块
    maxconnections=6,  # 连接池允许的最大连接数，0和None表示无限制连接数
    mincached=2,  # 初始化时，链接池中至少创建的空闲的链接，0表示不创建
    maxcached=5,  # 链接池中最多闲置的链接，0和None不限制
    maxshared=3,  # 链接池中最多共享的链接数量，0和None表示全部共享。PS: 无用，因为pymysql和mysqldb的模块都不支持共享链接
    blocking=True,  # 连接池中如果没有可用链接后，是否阻塞等待。False，不等待直接报错；True，等待直到有可用链接，再返回。
    host='127.0.0.1',
    port=3306,
    user='',
    password='',
    database='',
    charset='utf8'
)
```

使用数据库连接池

连接池对象创建成功后，可以从此对象中获取连接：

```python
# 你可以使用这个游标进行所有的常规的数据库交互操作
db_cursor = pool.connection().cursor()
```

查询示例

```python
import pymysql
from dbutils.pooled_db import PooledDB

# 创建连接池对象
pool = PooledDB(
    creator=pymysql,  # 使用链接数据库的模块
    maxconnections=6,  # 连接池允许的最大连接数，0和None表示无限制连接数
    mincached=2,  # 初始化时，链接池中至少创建的空闲的链接，0表示不创建
    maxcached=5,  # 链接池中最多闲置的链接，0和None不限制
    maxshared=3,  # 链接池中最多共享的链接数量，0和None表示全部共享。PS: 无用，因为pymysql和mysqldb的模块都不支持共享链接
    blocking=True,  # 连接池中如果没有可用链接后，是否阻塞等待。False，不等待直接报错；True，等待直到有可用链接，再返回。
    host='127.0.0.1',
    port=3306,
    user='root',
    password='root',
    database='py_spider',
    charset='utf8'
)

# 获取数据库连接
conn = pool.connection()

# 获取游标
cursor = conn.cursor()

# 执行查询操作
cursor.execute('SELECT * FROM ali_work;')

# 获取查询结果
result = cursor.fetchall()

# 打印结果
print(result)

# 关闭游标和连接
cursor.close()
conn.close()

# 关闭连接池
pool.close()
```

**总结**

数据库连接池是一种节省资源并提高效率的方法，特别是在处理大量数据库操作的`web`程序和网络应用程序中。创建连接池对象并获取到游标后，游标的使用方式与`pymysql`中的游标使用方式一致，在后面的并发爬虫章节中，我们会利用数据库连接池完成数据的并发读写操作。

> 来自: [5.4.MySQL数据库连接池](https://www.yuque.com/tuling_python/spider_base/ibufc1pppfbg8aan)

### 5.5.MongoDB数据库存储

**`MongoDB`回顾**

`MongoDB`是由`C++`语言编写的非关系型数据库，是一个基于分布式文件存储的开源数据库系统，其内容存储形式类似`JSON`对象，它的字段值可以包含其他文档、数组及文档数组。在这一节中，我们就来回顾`Python 3`下`MongoDB`的存储操作。

常用命令:

- 查询数据库:  `show dbs`
- 使用数据库:  `use 库名`
- 查看集合:  `show tables/show collections`
- 查询表数据:  `db.集合名.find()`
- 删除表:  `db.集合名.drop()`

**链接`MongoDB`**

连接`MongoDB`时，我们需要使用`PyMongo`库里面的`MongoClient`。一般来说，传入`MongoDB`的`IP`及端口即可，其中第一个参数为地址`host`，第二个参数为端口`port`（如果不给它传递参数，默认是`27017`）

```python
import pymongo # 如果是云服务的数据库 用公网IP连接

client = pymongo.MongoClient(host='localhost', port=27017)
```

**指定数据库与表**

```python
import pymongo

client = pymongo.MongoClient(host='localhost', port=27017)
collection = client['students']
```

**插入数据**

对于`students`这个集合，新建一条学生数据，这条数据以字典形式表示：

```python
import pymongo

mongo_client = pymongo.MongoClient(host='localhost', port=27017)
collection = mongo_client['students']['stu_info']

# 插入单条数据
# student = {'id': '20170101', 'name': 'Jordan', 'age': 20, 'gender': 'male'}
# result = collection.insert_one(student)
# print(result)

# 插入多条数据
student_1 = {'id': '20170101', 'name': 'Jordan', 'age': 20, 'gender': 'male'}
student_2 = {'id': '20170202', 'name': 'Mike', 'age': 21, 'gender': 'male'}
result = collection.insert_many([student_1, student_2])
print(result)
```

**结合`MongoDB`采集数据入库**

获取到爱奇艺视频数据信息：标题、播放地址、简介

目标地址：[https://list.iqiyi.com/www/2/15-------------11-1-1-iqiyi--.html?s_source=PCW_SC](https://list.iqiyi.com/www/2/15-------------11-1-1-iqiyi--.html?s_source=PCW_SC)

```python
import pymongo
import requests

class AiQiYi:
    def __init__(self):
        self.mongo_client = pymongo.MongoClient(host='localhost', port=27017)
        self.collection = self.mongo_client['py_spider']['AiQiYi']
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36',
            'Referer': 'https://list.iqiyi.com/www/2/15-------------11-1-1-iqiyi--.html?s_source=PCW_SC'
        }

        self.api_url = 'https://pcw-api.iqiyi.com/search/recommend/list'

    def get_movie_info(self, params):
        response = requests.get(self.api_url, headers=self.headers, params=params).json()
        return response

    def parse_movie_info(self, response):
        category_movies = response['data']['list']
        for movie in category_movies:
            item = dict()
            item['title'] = movie['title']
            item['playUrl'] = movie['playUrl']
            item['description'] = movie['description']
            print(item)
            self.save_movie_info(item)

    def save_movie_info(self, item):
        self.collection.insert_one(item)

    def main(self):
        for page in range(1, 6):
            params = {
                'channel_id': '2',
                'data_type': '1',
                'mode': '11',
                'page_id': page,
                'ret_num': '48',
                'session': 'c34d983f1e0c84ea7fc01dd923c9833e',
                'three_category_id': '15;must',
            }
            info = self.get_movie_info(params)
            self.parse_movie_info(info)

        # 程序完成后关闭数据库链接
        self.mongo_client.close()

if __name__ == '__main__':
    aqy = AiQiYi()
    aqy.main()
```

> 来自: [5.5.MongoDB数据库存储](https://www.yuque.com/tuling_python/spider_base/vdadwxri5k164fcg)

### 5.6.数据去重

在抓取数据的过程中可能因为网络原因造成爬虫程序崩溃退出，如果重新启动爬虫的话会造成数据入库重复的问题。接下来我们使用`redis`来进行数据去重。

**环境准备**

- 安装`redis`

```plain
pip install redis -i https://pypi.douban.com/simple
```

**项目需求以及思路分析**

目标网址：[https://www.mgtv.com/lib/2?lastp=list_index&lastp=ch_tv&kind=19&area=10&year=all&sort=c2&chargeInfo=a1&fpa=2912&fpos=](https://www.mgtv.com/lib/2?lastp=list_index&lastp=ch_tv&kind=19&area=10&year=all&sort=c2&chargeInfo=a1&fpa=2912&fpos=)

思路分析

- 首先判断当前网站上的数据是否为动态数据，如果为动态数据则使用浏览器抓包工具获取数据接口，当前接口地址如下：[https://pianku.api.mgtv.com/rider/list/pcweb/v3?allowedRC=1&platform=pcweb&channelId=2&pn=1&pc=80&hudong=1&_support=10000000&kind=19&area=10&year=all&chargeInfo=a1&sort=c2](https://pianku.api.mgtv.com/rider/list/pcweb/v3?allowedRC=1&platform=pcweb&channelId=2&pn=1&pc=80&hudong=1&_support=10000000&kind=19&area=10&year=all&chargeInfo=a1&sort=c2)
- 当获取到数据后对数据进行哈希编码，因为每一个哈希值是唯一的，所以可以利用这一特性判断数据是否重复。
- 将获取的数据存储到`mongodb`数据库中，在调用保存方法之前，先调用哈希方法将数据转为哈希并保存到`redis`中，再判断当前获取的数据的哈希是否存在于`redis`数据库，如果存在则不保存，反之亦然。

**代码示例**

```python
# -*- coding: utf-8 -*-
# @Time    : 2024/4/23 19:06
# @Author  : 顾安
# @File    : movie_spider.py
# @Software: PyCharm

import redis
import pymongo
import hashlib
import requests

class MovieInfo:
    headers = {
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) '
                      'AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36'
    }

    url = 'https://pianku.api.mgtv.com/rider/list/pcweb/v3'

    def __init__(self):
        self.mongo_client = pymongo.MongoClient()
        self.collection = self.mongo_client['py_spider']['mg_movie_info']
        self.redis_client = redis.Redis()

    # 请求数据
    @classmethod
    def get_movie_info(cls, params):
        response = requests.get(cls.url, headers=cls.headers, params=params).json()
        return response

    # 数据清洗以及数据结构调整
    def parse_data(self, response):
        movie_list = response['data']['hitDocs']
        for movie in movie_list:
            item = dict()
            item['title'] = movie['title']
            item['subtitle'] = movie['subtitle']
            item['story'] = movie['story']

            # 在数据清洗之后可以调用保存方法
            self.save_data(item)

    @staticmethod
    def get_md5(value):
        # md5方法只能接收字节数据
        # 计算哈希值, 哈希值是唯一的, 哈希值长度为32位
        md5_hash = hashlib.md5(str(value).encode('utf-8')).hexdigest()
        return md5_hash

    def save_data(self, item):
        value = self.get_md5(item)
        # 当前返回的是redis是否成功保存md5数据, 保存成功: result=1, 保存失败: result=0
        result = self.redis_client.sadd('movie:filter', value)
        # print(result)
        if result:
            self.collection.insert_one(item)
            print(item)
            print('保存成功...')
        else:
            print('数据重复...')

    def main(self):
        for page in range(1, 3):
            params = {
                "allowedRC": "1",
                "platform": "pcweb",
                "channelId": "2",
                "pn": page,
                "pc": "80",
                "hudong": "1",
                "_support": "10000000",
                "kind": "19",
                "area": "10",
                "year": "all",
                "chargeInfo": "a1",
                "sort": "c2"
            }
            response = self.get_movie_info(params)
            self.parse_data(response)

        # 任务执行完成后关闭数据库
        self.close_spider()

    def close_spider(self):
        print('数据库即将关闭...')
        self.redis_client.close()
        self.mongo_client.close()

if __name__ == '__main__':
    movie_info = MovieInfo()
    movie_info.main()
```

> 来自: [5.6.数据去重](https://www.yuque.com/tuling_python/spider_base/glaqg8ggnrqpr8q8)

## 6.并发爬虫

在本章节中，我们会学习使用协程、线程、进程等方式完成并发爬虫任务。因为在`python`中的线程受到`GIL`的影响，导致线程不能充分利用多核`CPU`的优势，所以可以利用协程来弥补性能上的不足。

除了使用协程完成爬虫并发外，使用线程池或者进程池也可以完成并发任务。

本章节会使用到的第三方库：

```bash
pip install aiohttp
pip install aiomysql
pip install aiofile
```

> 来自: [6.并发爬虫](https://www.yuque.com/tuling_python/spider_base/rzmc5r4xymrofite)

### 6.1.asyncio结合requests完成爬虫任务

`requests`是`python`中的同步网络爬虫库，并不能直接使用`asyncio`运行。所以我们使用`asyncio`中的`run_in_executor`方法创建线程池完成并发。

代码示例

```python
import asyncio
import requests
from functools import partial
from bs4 import BeautifulSoup

url = 'https://movie.douban.com/top250?start={}&filter='

headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
    "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36"
}

loop = asyncio.get_event_loop()

async def get_movie_info(page):
    # run_in_executor不支持关键字参数传递headers, 使用偏函数传递
    response = await loop.run_in_executor(None, partial(requests.get, url.format(page * 25), headers=headers))
    # print(response)
    # print(response.request.headers)
    soup = BeautifulSoup(response.text, 'lxml')
    div_list = soup.find_all('div', class_='hd')
    for title in div_list:
        print(title.get_text())

if __name__ == '__main__':
    # 异步随机调度导致输出的标题不是顺序输出
    tasks = [loop.create_task(get_movie_info(page)) for page in range(10)]
    loop.run_until_complete(asyncio.wait(tasks))
```

注意点

- `asyncio.loop.run_in_executor`方法:

- 执行那些不支持异步的阻塞函数, 默认并不支持关键字参数的传递。
- 想要使用关键字参数，就必须预先设定某些参数值。这就是`partial`处理的。

- `partial(requests.get, url.format(page * 25), headers=headers)`等同以下代码:

- `requests.get(url.format(page * 25), headers=headers)`

- `functools.partial`:

- 可以帮你创建一个新的函数，这个函数在调用时会自动将某些参数传给原函数

> 来自: [6.1.asyncio结合requests完成爬虫任务](https://www.yuque.com/tuling_python/spider_base/rwrrn7kxxkh00uhu)

### 6.2.使用aiohttp完成爬虫任务

由于`requests`爬虫库本身不支持异步，在`asyncio`中需要开启线程池才能使用。在使用上稍微有些麻烦，为了解决这个问题，我们使用支持异步操作的`aiohttp`来完成爬虫任务。

**介绍与安装**

介绍

`aiohttp`是一个异步的网络库，可以实现`HTTP`客户端，也可以实现`HTTP`服务器，爬虫阶段我们只用它来实现`HTTP`客户端功能。

官网：[https://docs.aiohttp.org/en/stable/](https://docs.aiohttp.org/en/stable/)

`aiohttp`客户端相关的官方文档：[https://docs.aiohttp.org/en/stable/client.html#aiohttp-client](https://docs.aiohttp.org/en/stable/client.html#aiohttp-client)

安装

```bash
pip install aiohttp -i https://pypi.tuna.tsinghua.edu.cn/simple
```

**基本使用**

示例代码

```python
import asyncio
from aiohttp import ClientSession

url = "https://www.baidu.com"

headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36"
}

async def get_baidu():
    async with ClientSession() as session:
        async with session.get(url, headers=headers) as response:
            response = await response.text()
            print(response)

if __name__ == '__main__':
    asyncio.run(get_baidu())
```

**并发操作**

示例代码

```python
import asyncio
import aiohttp

# 回调函数: 任务完成后打印返回值结果
def download_completed_callback(task_obj):
    print("下载的内容为:", task_obj.result())

async def baidu_spider():
    print("---百度蜘蛛---")
    url = "https://www.baidu.com"
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as r:
            return await r.text()

async def sogou_spider():
    print("---搜狗蜘蛛---")
    url = "https://www.sogou.com"
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as r:
            return await r.text()

async def jingdong_spider():
    print("---京东蜘蛛---")
    url = "https://www.jd.com"
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as r:
            return await r.text()

async def main():
    # 创建多个Task，且添加回调函数
    task_baidu = asyncio.create_task(baidu_spider())
    task_baidu.add_done_callback(download_completed_callback)

    task_sogou = asyncio.create_task(sogou_spider())
    task_sogou.add_done_callback(download_completed_callback)

    task_jingdong = asyncio.create_task(jingdong_spider())
    task_jingdong.add_done_callback(download_completed_callback)

    tasks = [task_baidu, task_sogou, task_jingdong]
    # 等待下载
    await asyncio.wait(tasks)

if __name__ == '__main__':
    asyncio.run(main())
```

**练习：使用`aiohttp`抓取豆瓣电影标题**

示例代码

```python
import asyncio
import aiohttp
from bs4 import BeautifulSoup

url = 'https://movie.douban.com/top250?start={}&filter='

headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36"
}

async def get_movie_info(page):
    async with aiohttp.ClientSession() as session:
        async with session.get(url.format(page * 25), headers=headers) as response:
            soup = BeautifulSoup(await response.text(), 'lxml')
            div_list = soup.find_all('div', class_='hd')
            for title in div_list:
                print(title.get_text())

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    tasks = [loop.create_task(get_movie_info(page)) for page in range(10)]
    loop.run_until_complete(asyncio.wait(tasks))
```

> 来自: [6.2.使用aiohttp完成爬虫任务](https://www.yuque.com/tuling_python/spider_base/bu5eq6fg12gkeo3r)

### 6.3.aiomysql的使用

**安装**

```shell
pip install aiomysql
```

利用`python3`中新加入的异步关键词`async/await`, 我们使用各种异步操作为来执行各种异步的操作，如使用`aiohttp`来代替`requests`来执行异步的网络请求操作，使用`motor`来代替同步的`pymongo`库来操作`mongo`数据库，我们在开发同步的`python`程序时，我们会使用`PyMySQL`来操作`mysql`数据库，同样，我们会使用`aiomysql`来异步操作`mysql`数据库。

**使用方式**

```python
import asyncio
import aiomysql

loop = asyncio.get_event_loop()

async def test_example():
    conn = await aiomysql.connect(host='127.0.0.1', port=3306,
                                  user='root', password='root', db='py_spider',
                                  loop=loop)

    cursor = await conn.cursor()
    await cursor.execute("SELECT * from ali_work")

    # 打印输出当前表中的字段信息
    print(cursor.description)
    result = await cursor.fetchall()
    print(result)
    await cursor.close()
    conn.close()

loop.run_until_complete(test_example())
```

**通过异步爬虫完成数据存储**

使用`asyncio`完成汽车之家的汽车参数信息并保存到`mysql`数据库中

网址：[https://www.che168.com/china/a0_0msdgscncgpi1ltocsp7exf4x0/?pvareaid=102179#currengpostion](https://www.che168.com/china/a0_0msdgscncgpi1ltocsp7exf4x0/?pvareaid=102179#currengpostion)

思路分析：

- 当前页面数据为静态数据，在翻页时`url`中的`sp1`会变更为`sp2`，所以当前页面可以使用`xpath`提取数据。
- 通过首页进入到详情页有当前汽车的配置信息，汽车配置信息页中的数据是动态数据，可以使用抓包的方式获取`api`。
- 根据获取的`api`链接发现当前链接中存在查询字符串：`specid`
- 回到首页，在汽车列表中通过元素发现`li`标签中存在汽车的`id`值，获取`id`值拼接`api`链接地址。
- 构造请求访问构造好的`api`地址获取数据。

注意点：

- 查看`api`接口返回的数据我们发现当前返回的数据类型并不是`json`数据，需要对返回的数据进行处理。处理方式有以下两种：

- 拿到返回数据后进行字符串切片，保留`json`数据部分
- 将`api`链接中的`callback=configTitle`查询字符串参数删除

- 汽车之家页面编码格式会随机变换，需要使用`chardet`第三方包实时监测编码格式，并且当页面编码格式为`UTF-8-SIG`时`specid`数据不存在。

```shell
pip install chardet
```

代码实现：

```python
"""
分析思路:
    1.在首页中获取汽车id
    2.将获取到的汽车id拼接到api数据接口中
    3.保存数据
"""
import redis
import chardet
import hashlib
import asyncio
import aiohttp
import aiomysql
from lxml import etree

class CarSpider:
    redis_client = redis.Redis()

    def __init__(self):
        self.url = 'https://www.che168.com/china/a0_0msdgscncgpi1ltocsp{}exf4x0/?pvareaid=102179#currengpostion'
        self.api_url = 'https://cacheapigo.che168.com/CarProduct/GetParam.ashx?specid={}'
        self.headers = {
            'User-Agent':
                'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36'
        }

    def __del__(self):
        print('redis数据库连接即将关闭...')
        self.redis_client.close()

    # 获取汽车id
    async def get_car_id(self, page, session, pool):
        async with session.get(self.url.format(page), headers=self.headers) as response:
            content = await response.read()
            encoding = chardet.detect(content)['encoding']  # 汽车之家会检测是否频繁请求, 如果频繁请求则将页面替换成UTF8编码格式并无法获取汽车id
            # print(encoding)
            if encoding == 'GB2312' or encoding == 'ISO-8859-1':
                result = content.decode('gbk')
            else:
                result = content.decode(encoding)
                print('被反爬了...')

            tree = etree.HTML(result)
            id_list = tree.xpath('//ul[@class="viewlist_ul"]/li/@specid')
            if id_list:
                print(id_list)
                tasks = [asyncio.create_task(self.get_car_info(spec_id, session, pool)) for spec_id in id_list]
                await asyncio.wait(tasks)

    # 当获取到页面中所有的汽车id之后要进行api连接的拼接并获取api数据
    async def get_car_info(self, spec_id, session, pool):
        async with session.get(self.api_url.format(spec_id), headers=self.headers) as response:
            result = await response.json()
            if result['result'].get('paramtypeitems'):
                item = dict()
                item['name'] = result['result']['paramtypeitems'][0]['paramitems'][0]['value']
                item['price'] = result['result']['paramtypeitems'][0]['paramitems'][1]['value']
                item['brand'] = result['result']['paramtypeitems'][0]['paramitems'][2]['value']
                item['altitude'] = result['result']['paramtypeitems'][1]['paramitems'][2]['value']
                item['breadth'] = result['result']['paramtypeitems'][1]['paramitems'][1]['value']
                item['length'] = result['result']['paramtypeitems'][1]['paramitems'][0]['value']
                await self.save_car_info(item, pool)
            else:
                print('数据不存在...')

    @staticmethod
    def get_md5(dict_item):
        md5 = hashlib.md5()
        md5.update(str(dict_item).encode('utf-8'))
        return md5.hexdigest()

    async def save_car_info(self, item, pool):
        print(item)
        # 使用异步上下文管理器创建链接对象以及游标对象
        async with pool.acquire() as conn:
            async with conn.cursor() as cursor:
                val_md5 = self.get_md5(item)
                # 保存成功返回1, 保存失败返回0
                redis_result = self.redis_client.sadd('car:filter', val_md5)
                if redis_result:
                    sql = """
                        insert into car_info(
                            id, name, price, brand, altitude, breadth, length) values (
                                %s, %s, %s, %s, %s, %s, %s
                            );
                    """
                    try:
                        await cursor.execute(sql, (
                            0,
                            item['name'],
                            item['price'],
                            item['brand'],
                            item['altitude'],
                            item['breadth'],
                            item['length']
                        ))
                        await conn.commit()
                        print('插入成功...')
                    except Exception as e:
                        print('数据插入失败:', e)
                        await conn.rollback()
                else:
                    print('数据重复...')

    # 启动函数
    async def main(self):
        # 创建数据库连接池并获取游标对象
        async with aiomysql.create_pool(user='root', password='root', db='py_spider') as pool:
            async with pool.acquire() as conn:
                async with conn.cursor() as cursor:
                    # 创建表
                    create_table_sql = """
                        create table car_info(
                            id int primary key auto_increment,
                            name varchar(100),
                            price varchar(100),
                            brand varchar(100),
                            altitude varchar(100),
                            breadth varchar(100),
                            length varchar(100)
                        );
                    """

                    # 在异步代码中必须先要检查表是否存在, 直接使用if not语句无效
                    check_table_query = "show tables like 'car_info'"
                    result = await cursor.execute(check_table_query)  # 如果表存在返回1 不存在返回0
                    if not result:
                        await cursor.execute(create_table_sql)

            # 创建请求对象
            async with aiohttp.ClientSession() as session:
                tasks = [asyncio.create_task(self.get_car_id(page, session, pool)) for page in range(1, 16)]
                await asyncio.wait(tasks)

if __name__ == '__main__':
    car = CarSpider()
    asyncio.run(car.main())
```

**补充：使用`motor`保存数据**

使用`fake_useragent`实现随机`UA`

安装指令：

- `pip install fake_useragent`
- `pip install motor`

```python
import redis
import chardet
import hashlib
import asyncio
import aiohttp
from lxml import etree
from fake_useragent import UserAgent
from motor.motor_asyncio import AsyncIOMotorClient

class CarSpider:
    redis_client = redis.Redis()
    mongo_client = AsyncIOMotorClient('localhost', 27017)['py_spider']['car_info']

    def __init__(self):
        self.url = 'https://www.che168.com/china/a0_0msdgscncgpi1ltocsp{}exf4x0/?pvareaid=102179#currengpostion'
        self.api_url = 'https://cacheapigo.che168.com/CarProduct/GetParam.ashx?specid={}'
        self.user_agent = UserAgent()

    def __del__(self):
        print('redis数据库连接即将关闭...')
        self.redis_client.close()

    async def get_car_id(self, page, session):
        headers = {'User-Agent': self.user_agent.random}
        async with session.get(self.url.format(page), headers=headers) as response:
            content = await response.read()
            encoding = chardet.detect(content)['encoding']
            if encoding == 'GB2312' or encoding == 'ISO-8859-1':
                result = content.decode('gbk')
                print('请求成功:', headers)
            else:
                result = content.decode(encoding)
                print('被反爬了:', headers)

            tree = etree.HTML(result)
            id_list = tree.xpath('//ul[@class="viewlist_ul"]/li/@specid')
            if id_list:
                tasks = [asyncio.create_task(self.get_car_info(spec_id, session)) for spec_id in id_list]
                await asyncio.wait(tasks)

    async def get_car_info(self, spec_id, session):
        headers = {'User-Agent': self.user_agent.random}
        async with session.get(self.api_url.format(spec_id), headers=headers) as response:
            result = await response.json()
            if result['result'].get('paramtypeitems'):
                item = dict()
                item['name'] = result['result']['paramtypeitems'][0]['paramitems'][0]['value']
                item['price'] = result['result']['paramtypeitems'][0]['paramitems'][1]['value']
                item['brand'] = result['result']['paramtypeitems'][0]['paramitems'][2]['value']
                item['altitude'] = result['result']['paramtypeitems'][1]['paramitems'][2]['value']
                item['breadth'] = result['result']['paramtypeitems'][1]['paramitems'][1]['value']
                item['length'] = result['result']['paramtypeitems'][1]['paramitems'][0]['value']
                await self.save_car_info(item)
            else:
                print('数据不存在...')

    @staticmethod
    def get_md5(dict_item):
        md5 = hashlib.md5()
        md5.update(str(dict_item).encode('utf-8'))
        return md5.hexdigest()

    async def save_car_info(self, item):
        """
        motor自行维护了数据库连接池, 无需手动关闭连接对象
        """
        md5_hash = self.get_md5(item)
        redis_result = self.redis_client.sadd('car:filter', md5_hash)
        if redis_result:
            await self.mongo_client.insert_one(item)
            print('数据插入成功:', item)
        else:
            print('数据重复...')

    async def main(self):
        async with aiohttp.ClientSession() as session:
            tasks = [asyncio.create_task(self.get_car_id(page, session)) for page in range(1, 101)]
            await asyncio.wait(tasks)

if __name__ == '__main__':
    car = CarSpider()
    asyncio.run(car.main())
```

> 来自: [6.3.aiomysql的使用](https://www.yuque.com/tuling_python/spider_base/bcqugbimo621c0er)

### 6.4.使用多线程完成并发爬虫

在上一小节中我们使用了`asyncio`的方式完成了并发爬虫，但是大多数时候最常用的还是基于多线程的方式来完成爬虫需求，所以还是有必要回顾一下之前所学习的多线程知识点。

**爬虫需求**

根据豆瓣协程爬虫代码改写成多线程模式

```python
import requests
import threading
from lxml import etree

url = 'https://movie.douban.com/top250?start={}&filter='

headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36"
}

def get_movie_info(page):
    response = requests.get(url.format(page * 25), headers=headers).text
    tree = etree.HTML(response)
    result = tree.xpath("//div[@class='hd']/a/span[1]/text()")
    print(result)

if __name__ == '__main__':
    thread_obj_list = [threading.Thread(target=get_movie_info, args=(page,)) for page in range(10)]
    for thread_obj in thread_obj_list:
        thread_obj.start()
```

> 来自: [6.4.使用多线程完成并发爬虫](https://www.yuque.com/tuling_python/spider_base/fnm5fwxr5qu5qhl3)

### 6.5.使用线程池完成并发爬虫

还是以当前豆瓣爬虫为例，将上面的代码改写成线程池模式

代码示例

```python
import requests
from lxml import etree
from concurrent.futures import ThreadPoolExecutor, as_completed

url = 'https://movie.douban.com/top250?start={}&filter='

headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36"
}

def get_movie_info(page):
    response = requests.get(url.format(page * 25), headers=headers).text
    tree = etree.HTML(response)
    result = tree.xpath("//div[@class='hd']/a/span[1]/text()")
    return result

if __name__ == '__main__':
    with ThreadPoolExecutor(max_workers=5) as pool:
        futures = [pool.submit(get_movie_info, page) for page in range(10)]
        # future对象获取返回值会造成主线程堵塞
        # for future in futures:
        #     print(future.result())

        # as_completed会立即返回处理完成的结果而不会堵塞主线程
        for future in as_completed(futures):
            print(future.result())
```

> 来自: [6.5.使用线程池完成并发爬虫](https://www.yuque.com/tuling_python/spider_base/lzqmwhc2wbm2pgen)

### 6.6.使用多进程完成并发爬虫

因为在`Python`中存在`GIL`锁，无法充分利用多核优势。所以为了能够提高程序运行效率我们也会采用进程的方式来完成代码需求。

**进程代码回顾**

```python
from multiprocessing import Process

# 创建进程对象
p = Process(target=func, args=(,))

# 设置守护进程
p.daemon = True

# 启动进程
p.start()
```

**进程中的队列**

多进程中使用普通的队列模块会发生阻塞，对应的需要使用`multiprocessing`提供的`JoinableQueue`模块，其使用过程和在线程中使用的`queue`方法相同。

接下来我们通过腾讯招聘代码案例学习如何在进程中使用`JoinableQueue`队列模块。

页面地址：[https://careers.tencent.com/search.html?keyword=python](https://careers.tencent.com/search.html?keyword=python)

代码示例

```python
import time
import pymongo
import requests
import jsonpath
from multiprocessing import Process, JoinableQueue as Queue

url = 'https://careers.tencent.com/tencentcareer/api/post/Query'

headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36"
}

def get_work_info_json(page_num, queue):
    params = {
        'timestamp': 1696774900608,
        'countryId': '',
        'cityId': '',
        'bgIds': '',
        'productId': '',
        'categoryId': '',
        'parentCategoryId': '',
        'attrId': '',
        'keyword': 'python',
        'pageIndex': page_num,
        'pageSize': 10,
        'language': 'zh-cn',
        'area': 'cn'
    }

    response = requests.get(url, headers=headers, params=params).json()
    # 在某些页面中不存在当前的json数据会跑出异常
    try:
        for info in response['Data']['Posts']:
            work_info_dict = dict()
            work_info_dict['recruit_post_name'] = jsonpath.jsonpath(info, '$..RecruitPostName')[0]
            work_info_dict['country_name'] = jsonpath.jsonpath(info, '$..CountryName')[0]
            work_info_dict['location_name'] = jsonpath.jsonpath(info, '$..LocationName')[0]
            work_info_dict['category_name'] = jsonpath.jsonpath(info, '$..CategoryName')[0]
            work_info_dict['responsibility'] = jsonpath.jsonpath(info, '$..Responsibility')[0]
            work_info_dict['last_update_time'] = jsonpath.jsonpath(info, '$..LastUpdateTime')[0]

            queue.put(work_info_dict)
    except TypeError:
        print('数据不存在:', params.get('pageIndex'))

def save_work_info(queue):
    mongo_client = pymongo.MongoClient()
    collection = mongo_client['py_spider']['tx_work']
    while True:
        dict_data = queue.get()
        print(dict_data)
        collection.insert_one(dict_data)
        # 计数器减1, 为0解堵塞
        queue.task_done()

if __name__ == '__main__':
    dict_data_queue = Queue()
    # 创建进程对象列表
    process_list = list()

    for page in range(1, 50):
        p_get_info = Process(target=get_work_info_json, args=(page, dict_data_queue))
        process_list.append(p_get_info)

    # get_work_info_json不是无限循环任务, 无需设置守护进程直接启动即可
    for process_obj in process_list:
        process_obj.start()

    # save_work_info是无限循环任务, 则需要设置为守护进程让主进程可以正常退出
    p_save_work = Process(target=save_work_info, args=(dict_data_queue,))
    p_save_work.daemon = True
    p_save_work.start()

    # 让主进程等待有限任务执行完毕
    for process_obj in process_list:
        process_obj.join()

    # 等待队列任务完成
    dict_data_queue.join()
    print('爬虫任务完成...')
```

> 来自: [6.6.使用多进程完成并发爬虫](https://www.yuque.com/tuling_python/spider_base/wbccfi9phx7ffpel)

### 6.7.并发爬虫实战

**使用多线程抓取爱奇艺视频信息**

网站地址：[https://list.iqiyi.com/www/2/15-------------11-1-1-iqiyi--.html?s_source=PCW_SC](https://list.iqiyi.com/www/2/15-------------11-1-1-iqiyi--.html?s_source=PCW_SC)

```python
import pymongo
import requests
import threading
from queue import Queue

class AiQiYi:
    def __init__(self):
        self.mongo_client = pymongo.MongoClient(host='localhost', port=27017)
        self.collection = self.mongo_client['py_spider']['Thread_AiQiYi']
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36',
            'Referer': 'https://list.iqiyi.com/www/2/15-------------11-1-1-iqiyi--.html?s_source=PCW_SC'
        }
        self.api_url = 'https://pcw-api.iqiyi.com/search/recommend/list?channel_id=2&data_type=1&mode=11&page_id={}&ret_num=48&session=85dd981b69cead4b60f6d980438a5664&three_category_id=15;must'

        # 创建队列
        self.url_queue = Queue()  # 请求地址队列
        self.json_queue = Queue()  # json数据队列
        self.content_dict_queue = Queue()  # 内容字典队列

    def get_url(self):
        for page in range(1, 6):
            self.url_queue.put(self.api_url.format(page))  # 将请求地址上传到url队列

    def get_api_json(self):
        while True:
            url = self.url_queue.get()
            response = requests.get(url, headers=self.headers)
            self.json_queue.put(response.json())  # 将获取的json数据上传到json队列
            self.url_queue.task_done()  # 让url队列获取一条数据后队列内部计数器减1

    def parse_movie_info(self):
        while True:
            json_data = self.json_queue.get()
            category_movies = json_data['data']['list']
            for movie in category_movies:
                item = dict()
                item['title'] = movie['title']
                item['playUrl'] = movie['playUrl']
                item['description'] = movie['description']
                self.content_dict_queue.put(item)  # 将内容上传到内容字典队列

            self.json_queue.task_done()  # 循环上传字典数据完成后则json队列计数器减1

    def save_movie_info(self):
        while True:
            item = self.content_dict_queue.get()
            self.collection.insert_one(item)
            print('插入成功:', item)
            self.content_dict_queue.task_done()  # 获取一条内容让内容字典队列计数器减1

    def main(self):
        # 初始化线程对象列表
        thread_obj_list = list()

        # 有限循环任务无需使用守护线程, 直接启动即可
        t_url = threading.Thread(target=self.get_url)
        t_url.start()
        t_url.join()

        # 创建发送请求的线程对象并加入到线程对象列表中
        for _ in range(3):
            t_get_json = threading.Thread(target=self.get_api_json)
            thread_obj_list.append(t_get_json)

        # 创建数据提取的线程对象并加入到线程对象列表中
        t_parse_info = threading.Thread(target=self.parse_movie_info)
        thread_obj_list.append(t_parse_info)

        # 创建保存数据的线程对象并加入到线程对象列表中
        t_save_info = threading.Thread(target=self.save_movie_info)
        thread_obj_list.append(t_save_info)

        # 循环线程列表设置线程对象为守护线程并启动
        for t_obj in thread_obj_list:
            t_obj.daemon = True
            t_obj.start()

        # 判断所有队列中的计数器是否为零, 如果为零则退出程序, 否则让主线程堵塞
        for q in [self.url_queue, self.json_queue, self.content_dict_queue]:
            q.join()

        print('主线程结束...')

if __name__ == '__main__':
    aqy = AiQiYi()
    aqy.main()
```

注意点：

- `put`会让队列的计数`+1`，但是单纯的使用`get`不会让其`-1`，需要和`task_done`同时使用才能够`-1`
- `task_done`不能放在另一个队列的`put`之前，否则可能会出现数据没有处理完成，程序结束的情况

**使用线程池完成百度招聘信息**

网站地址：[https://talent.baidu.com/jobs/social-list?search=python](https://talent.baidu.com/jobs/social-list?search=python)

注意点：当前网站`api`请求方式为`post`

```python
import pymysql
import requests
from dbutils.pooled_db import PooledDB
from concurrent.futures import ThreadPoolExecutor, as_completed

class BaiDuWork:
    def __init__(self):
        self.pool = PooledDB(
            creator=pymysql,  # 使用链接数据库的模块
            maxconnections=6,  # 连接池允许的最大连接数，0和None表示不限制连接数
            mincached=2,  # 初始化时，链接池中至少创建的空闲的链接，0表示不创建
            maxcached=5,  # 链接池中最多闲置的链接，0和None不限制
            maxshared=3,  # 设置线程之间的共享连接
            blocking=True,  # 连接耗尽则等待直至有可用的连接为止
            maxusage=None,  # 一个链接最多被重复使用的次数，None表示无限制
            setsession=[],  # 开始会话前执行的命令列表。如：["set datestyle to ...", "set time zone ..."]
            ping=0,
            host='localhost',
            port=3306,
            user='root',
            password='root',
            database='py_spider',
            charset='utf8'
        )
        self.api_url = 'https://talent.baidu.com/httservice/getPostListNew'
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36',
            'Referer': 'https://talent.baidu.com/jobs/social-list?search=python',
        }

    # 连接池对象已经使用了上下文协议关闭连接，无需手动关闭
    # def __del__(self):
    #     self.pool.close()
    #     print('数据库连接池已关闭...')

    def get_work_info(self, page):
        post_data = {
            'recruitType': 'SOCIAL',
            'pageSize': 10,
            'keyWord': '',
            'curPage': page,
            'projectType': '',
        }
        response = requests.post(url=self.api_url, headers=self.headers, data=post_data)
        return response.json()

    def parse_work_info(self, response):
        works = response['data']['list']
        for work_info in works:
            education = work_info['education'] if work_info['education'] else '空'
            name = work_info['name']
            service_condition = work_info['serviceCondition']
            self.save_work_info(education, name, service_condition)

    def create_table(self):
        with self.pool.connection() as db:  # 从连接池中取出一个可用的连接
            with db.cursor() as cursor:
                sql = """
                            CREATE TABLE IF NOT EXISTS baidu_work_threadpool (
                            id INT PRIMARY KEY AUTO_INCREMENT,
                            education VARCHAR(200),
                            name VARCHAR(100),
                            service_condition TEXT
                            )
                        """

                try:
                    cursor.execute(sql)
                    print('表创建成功...')
                except Exception as e:
                    print('表创建失败: ', e)

    def save_work_info(self, education, name, service_condition):
        with self.pool.connection() as db:
            with db.cursor() as cursor:
                sql = """
                    INSERT INTO baidu_work_threadpool(id, education, name, service_condition) VALUES (
                    %s, %s, %s, %s
                    )
                """

                try:
                    cursor.execute(sql, (0, education, name, service_condition))
                    db.commit()
                    print('数据保存成功:', (education, name, service_condition))
                except Exception as e:
                    print('数据保存失败: ', e)
                    db.rollback()

    def main(self):
        self.create_table()

        with ThreadPoolExecutor(max_workers=5) as pool:
            futures = [pool.submit(self.get_work_info, page) for page in range(1, 32)]
            """
            future.result()方法会造成主线程堵塞
            需要使用as_completed转为非阻塞
            """
            for future in as_completed(futures):
                pool.submit(self.parse_work_info, future.result())

if __name__ == '__main__':
    baidu_work = BaiDuWork()
    baidu_work.main()
```

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def do_some_work(x):
    return x

with ThreadPoolExecutor(max_workers=5) as executor:
    futures = [executor.submit(do_some_work, i) for i in range(10)]

    # 没有使用as_completed方法则结果为顺序返回
    for future in futures:
        result = future.result()
        print('Task returned:', result)

    print('-' * 30)

    # 使用as_completed方法结果为并发返回
    for future in as_completed(futures):
        result = future.result()
        print('Task returned:', result)
```

**使用多进程抓取芒果视频信息**

网站网址：[https://www.mgtv.com/lib/2?lastp=list_index&lastp=ch_tv&kind=19&area=10&year=all&sort=c2&chargeInfo=a1&fpa=2912&fpos=](https://www.mgtv.com/lib/2?lastp=list_index&lastp=ch_tv&kind=19&area=10&year=all&sort=c2&chargeInfo=a1&fpa=2912&fpos=)

```python
import time
import redis
import pymongo
import hashlib
import requests
from multiprocessing import Process, JoinableQueue as Queue

class MovieInfo:
    # 数据库链接对象不能在进程环境中创建成实例属性
    mongo_client = pymongo.MongoClient()
    collection = mongo_client['py_spider']['mg_movie']
    redis_client = redis.Redis()

    def __init__(self):
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36'
        }

        self.url = "https://pianku.api.mgtv.com/rider/list/pcweb/v3"

        self.params_queue = Queue()
        self.json_queue = Queue()
        self.content_queue = Queue()

    # 上传翻页信息
    def put_params(self):
        for page in range(1, 6):
            params_dict = {
                "allowedRC": "1",
                "platform": "pcweb",
                "channelId": "2",
                "pn": page,
                "pc": "80",
                "hudong": "1",
                "_support": "10000000",
                "kind": "19",
                "area": "10",
                "year": "all",
                "chargeInfo": "a1",
                "sort": "c2",
                "feature": "all"
            }
            self.params_queue.put(params_dict)

    # 请求数据
    def get_movie_info(self):
        while True:
            params_dict = self.params_queue.get()
            response = requests.get(self.url, headers=self.headers, params=params_dict).json()
            self.json_queue.put(response)
            self.params_queue.task_done()

    # 数据结构调整
    def parse_info(self):
        while True:
            response = self.json_queue.get()
            movie_list = response['data']['hitDocs']
            for movie in movie_list:
                item = dict()
                item['title'] = movie['title']
                item['subtitle'] = movie['subtitle']
                item['story'] = movie['story']
                self.content_queue.put(item)
            self.json_queue.task_done()

    # 去重方法
    @staticmethod
    def get_md5(dict_item):
        md5_hash = hashlib.md5(str(dict_item).encode('utf-8')).hexdigest()
        return md5_hash

    # 数据保存
    def save_movie_info(self):
        while True:
            item = self.content_queue.get()
            md5_hash = self.get_md5(item)
            result = self.redis_client.sadd('mg_movie:filter', md5_hash)
            if result:
                self.collection.insert_one(item)
                print('数据插入成功:', item)
            else:
                print('数据重复...')
            self.content_queue.task_done()

    def close_spider(self):
        print('爬虫即将退出, 准备关闭数据库链接...')
        self.mongo_client.close()
        self.redis_client.close()

    # 启动函数
    def main(self):
        process_list = list()

        # 有限循环任务直接启动进程即可, 无需创建守护进程
        p_put_params = Process(target=self.put_params)
        p_put_params.start()

        for _ in range(3):
            p_get_movie = Process(target=self.get_movie_info)
            process_list.append(p_get_movie)

        p_parse_info = Process(target=self.parse_info)
        process_list.append(p_parse_info)

        p_save_info = Process(target=self.save_movie_info)
        process_list.append(p_save_info)

        for process_obj in process_list:
            process_obj.daemon = True
            process_obj.start()

        # 让主进程等待有限任务执行完毕后解堵塞
        p_put_params.join()

        # 如果队列中的任务没有完成则防止主进程退出
        for q in [self.params_queue, self.json_queue, self.content_queue]:
            q.join()

        self.close_spider()  # 使用主进程关闭数据库连接

if __name__ == '__main__':
    movie_info = MovieInfo()
    movie_info.main()
```

注意：在多进程环境中，数据库连接对象不能在`__init__`方法中执行，会出现序列化失败的问题，需要将连接方法放置在类属性中。

进程池版本示例

```python
import redis
import pymongo
import hashlib
import requests
from concurrent.futures import ProcessPoolExecutor, as_completed

class MovieInfo:
    # 数据库链接对象不能在进程环境中创建成实例属性
    mongo_client = pymongo.MongoClient()
    collection = mongo_client['py_spider']['mg_movie']
    redis_client = redis.Redis()

    def __init__(self):
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36'
        }

        self.url = "https://pianku.api.mgtv.com/rider/list/pcweb/v3"

    # 上传翻页信息
    @staticmethod
    def put_params():
        params_list = []
        for page in range(1, 6):
            params_dict = {
                "allowedRC": "1",
                "platform": "pcweb",
                "channelId": "2",
                "pn": page,
                "pc": "80",
                "hudong": "1",
                "support": "10000000",
                "kind": "19",
                "area": "10",
                "year": "all",
                "chargeInfo": "a1",
                "sort": "c2",
                "feature": "all"
            }
            params_list.append(params_dict)
        return params_list

    # 请求数据
    def get_movie_info(self, params_dict):
        response = requests.get(self.url, headers=self.headers, params=params_dict).json()
        return response

    # 数据结构调整
    @staticmethod
    def parse_info(response):
        movie_list = response['data']['hitDocs']
        items = []
        for movie in movie_list:
            item = dict()
            item['title'] = movie['title']
            item['subtitle'] = movie['subtitle']
            item['story'] = movie['story']
            items.append(item)
        return items

    # 去重方法
    @staticmethod
    def get_md5(dict_item):
        md5_hash = hashlib.md5(str(dict_item).encode('utf-8')).hexdigest()
        return md5_hash

    # 数据保存
    def save_movie_info(self, items):
        for item in items:
            md5_hash = self.get_md5(item)
            result = self.redis_client.sadd('mg_movie:filter', md5_hash)
            if result:
                self.collection.insert_one(item)
                print('数据插入成功:', item)
            else:
                print('数据重复...')

    def close_spider(self):
        print('爬虫即将退出, 准备关闭数据库链接...')
        self.mongo_client.close()
        self.redis_client.close()

    # 启动函数
    def main(self):
        params_list = self.put_params()

        with ProcessPoolExecutor(max_workers=4) as executor:
            # 提交请求数据的任务
            future_to_params = {executor.submit(self.get_movie_info, params): params for params in params_list}

            # 处理请求数据的结果
            for future in as_completed(future_to_params):
                response = future.result()
                items = self.parse_info(response)
                self.save_movie_info(items)

        self.close_spider()

if __name__ == '__main__':
    movie_info = MovieInfo()
    movie_info.main()
```

**使用协程完成王者荣耀英雄图片下载**

采集王者荣耀官网中所有英雄的图片信息

网站地址：[https://pvp.qq.com/web201605/herolist.shtml](https://pvp.qq.com/web201605/herolist.shtml)

```python
import os
import aiofile
import aiohttp
import asyncio

class HeroSkin:
    def __init__(self):
        self.json_url = 'https://pvp.qq.com/web201605/js/herolist.json'
        self.skin_url = 'https://game.gtimg.cn/images/yxzj/img201606/skin/hero-info/{}/{}-bigskin-{}.jpg'
        self.headers = {
            'User-Agent':
                'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36'
        }

    async def get_image_content(self, session, e_name, c_name):
        # 因为不确定每个英雄具体的皮肤数量, 所以设置一个超出英雄皮肤数量的最大值
        # 根据链接状态码判断是否请求成功, 如果请求失败则终止循环并获取下一个英雄的皮肤图片
        for skin_id in range(1, 31):
            async with session.get(self.skin_url.format(e_name, e_name, skin_id), headers=self.headers) as response:
                if response.status == 200:
                    content = await response.read()
                    async with aiofile.async_open('./images/' + c_name + '-' + str(skin_id) + '.jpg', 'wb') as f:
                        await f.write(content)
                        print('保存成功:', c_name)
                else:
                    break

    async def main(self):
        tasks = list()
        async with aiohttp.ClientSession() as session:
            async with session.get(self.json_url, headers=self.headers) as response:
                result = await response.json(content_type=None)
                for item in result:
                    e_name = item['ename']
                    c_name = item['cname']
                    # print(e_name, c_name)
                    coro_obj = self.get_image_content(session, e_name, c_name)
                    tasks.append(asyncio.create_task(coro_obj))
                await asyncio.wait(tasks)

if __name__ == '__main__':
    if not os.path.exists('./images'):
        os.mkdir('./images')

    hero_skin = HeroSkin()
    asyncio.run(hero_skin.main())
```

> 来自: [6.7.并发爬虫实战](https://www.yuque.com/tuling_python/spider_base/dk7ewwlgkv685l9p)

## 7.自动化测试框架

有很多网站，例如淘宝，它上面的很多页面 的数据是由`JavaScript`生成的，而不是原始`HTML`代码，而且还有很多`ajax`获取的数据，甚至有些数据是加密的。当我们使用普通的`requests`来处理时，需要分析很多的`js`代码，此时非常困难，所以我们就用`selenium`来解决。

**`selenium`框架的简单介绍**

`selenium`是一个`Web`的自动化测试工具，最初是为网站自动化测试而开发的，利用它可以控制浏览器执行特定的动作，例如点击、下拉、输入内容等

`selenium`文档地址：[https://selenium-python.readthedocs.io/](https://selenium-python.readthedocs.io/)

本章重点：

- 配置`selenium`运行环境
- `selenium`框架中提供的属性与方法
- 元素定位
- 浏览器交互

> 来自: [7.自动化测试框架](https://www.yuque.com/tuling_python/spider_base/gswdmll6qwgng0fs)

### 7.1.环境配置

安装命令：

```python
# 在安装过程中最好限定框架版本为4.9.1
pip install selenium==4.9.1 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

安装完`selenium`后，还需要安装使用`selenium`控制的浏览器需要的驱动。

谷歌驱动下载地址：[https://googlechromelabs.github.io/chrome-for-testing/#stable](https://googlechromelabs.github.io/chrome-for-testing/#stable)

驱动下载完成后将文件移动到系统环境变量中：

- `MacOS`：将文件移动到`/usr/local/bin`目录
- `Windows`：将文件移动到`miniconda3`安装目录

编写以下代码，验证是否能正常运行：

```python
from selenium import webdriver

browser = webdriver.Chrome()
```

如果可以正常打开浏览器则配置成功。

除了可以配置谷歌浏览器之外也可以配置火狐浏览器，配置方式与谷歌浏览器大致相同。

火狐驱动下载地址：[https://github.com/mozilla/geckodriver/releases](https://github.com/mozilla/geckodriver/releases)

> 来自: [7.1.环境配置](https://www.yuque.com/tuling_python/spider_base/pyczukgbhng1plpq)

### 7.2.基本使用

**加载网页**

```python
from selenium import webdriver

# 获取要操作的浏览器驱动对象（直白点说，这个对象可以控制浏览器）
browser = webdriver.Chrome()

# 加载指定的页面
browser.get("http://www.baidu.com")

# 截屏
browser.save_screenshot("百度首页.png")
```

**定位与操作**

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By

# 获取要操作的浏览器驱动对象
browser = webdriver.Chrome()

# 加载指定的页面
browser.get("http://www.baidu.com")

# 获取指定的元素
browser.find_element(By.ID, 'kw').send_keys('java')

# 延时，以便看清楚要进行的操作
time.sleep(2)

# 点击 "百度一下"
browser.find_element(By.ID, 'su').click()
```

**查看响应信息**

```python
import time
from selenium import webdriver

# 获取要操作的浏览器驱动对象
browser = webdriver.Chrome()

# 加载指定的页面
browser.get("http://www.baidu.com")

# 查看访问的页面源代码
print(browser.page_source)

# 查看cookie
print(browser.get_cookies())

# 查看经过处理之后，本页面最后显示的url，如果有302的话，那么就是302之后的url
print(browser.current_url)
```

**打开新页面**

```python
import time
from selenium import webdriver

# 创建浏览器
browser = webdriver.Chrome()

# 打开百度
browser.get("http://www.baidu.com")

time.sleep(3)

# 打开京东
browser.get("https://jd.com")
time.sleep(3)

# 关闭
browser.close()
```

**通过`js`脚本打开新标签页**

```python
import time
from selenium import webdriver

browser = webdriver.Chrome()

# 打开淘宝
browser.get("http://login.taobao.com")

time.sleep(3)

# 打开搜狗: 执行js脚本打开新的标签页
js = "window.open('http://www.sogou.com')"
browser.execute_script(js)
```

**切换标签页**

```python
import time
from selenium import webdriver

browser = webdriver.Chrome()

# 打开淘宝
browser.get("http://login.taobao.com")

time.sleep(3)

# 打开搜狗
js = "window.open('http://www.sogou.com')"
browser.execute_script(js)

time.sleep(3)

# 切换到第1个标签页
browser.switch_to.window(browser.window_handles[0])
time.sleep(1)

browser.switch_to.window(browser.window_handles[1])
time.sleep(1)

# 关闭第2个标签页
browser.close()

time.sleep(1)

# 切换到第1个标签页
browser.switch_to.window(browser.window_handles[0])
# 关闭第1个标签页
browser.close()
```

**退出**

```python
import time
from selenium import webdriver

# 获取要操作的浏览器驱动对象
browser = webdriver.Chrome()

# 加载指定的页面
browser.get("http://www.baidu.com")

# 为了演示，浏览器打开后关闭的效果，要先延时一会
time.sleep(3)

# 关闭当前页面（当浏览器只有1个页面时，此操作会让浏览器退出）
browser.close()

# 让浏览器退出(如果用selenium打开了很多标签页的情况下)
browser.quit()
```

**多个标签页切换顺序混乱的问题**

`window_handles`列表保存了根据顺序打开的标签页句柄，但是在某些特殊的情况下标签页顺序和列表句柄元素顺序不一致，比如网络速度或页面响应速度的不同会导致实际打开页面的顺序和预期不同。所以在代码中不能完全依赖列表索引的方式完成页面切换。

解决方式如下：

```python
import time
from selenium import webdriver

browser = webdriver.Chrome()

# 打开淘宝
browser.get("http://login.taobao.com")
time.sleep(3)

# 打开搜狗
js = "window.open('http://www.sogou.com')"
browser.execute_script(js)
time.sleep(3)

# 打开必应
js = "window.open('http://www.bing.com')"
browser.execute_script(js)
time.sleep(3)

# 打印当前所有标签页的窗口句柄
print(browser.window_handles)

# 打印所有句柄对应的标签页名称
for handle in browser.window_handles:
    browser.switch_to.window(handle)
    print("句柄: {}，页面标题: {}".format(handle, browser.title))

for handle in browser.window_handles:
    browser.switch_to.window(handle)
    # 通过页面标题或URL来定位
    if ("搜狗" in browser.title) or ("sogou.com" in browser.current_url):
        print("已切换到搜狗页面:", handle)
        time.sleep(2)
        browser.close()  # 关闭搜狗标签页
    elif ("必应" in browser.title) or ("bing.com" in browser.current_url):
        print("已切换到必应页面:", handle)
        time.sleep(2)
    else:
        print('已切换到淘宝页面:', handle)
        time.sleep(2)

print(browser.window_handles)
browser.quit()
```

> 来自: [7.2.基本使用](https://www.yuque.com/tuling_python/spider_base/og7098otrd6l8yrx)

### 7.3.元素定位方法

**元素定位的基本使用方式**

在定位元素时，需要借助`selenium`框架提供的定位工具来进行元素定位。元素定位工具导入路径如下：

```python
from selenium.webdriver.common.by import By
```

为了能够点击某个按钮，此时我们就需要准确无误的定位到需要的元素。元素定位主要分为以下两种：

- 单个节点（返回是一个对象）

- `find_element(By.ID, '定位规则')`
- `find_element(By.NAME, '定位规则')`
- `find_element(By.XPATH, '定位规则')`
- `find_element(By.LINK_TEXT, '定位规则')`
- `find_element(By.PARTIAL_LINK_TEXT, '定位规则')`
- `find_element(By.TAG_NAME, '定位规则')`
- `find_element(By.CLASS_NAME, '定位规则')`
- `find_element(By.CSS_SELECTOR, '定位规则')`

- 多个节点（返回是一个列表）

- `find_elements(By.ID, '定位规则')`
- `find_elements(By.NAME, '定位规则')`
- `find_elements(By.XPATH, '定位规则')`
- `find_elements(By.LINK_TEXT, '定位规则')`
- `find_elements(By.PARTIAL_LINK_TEXT, '定位规则')`
- `find_elements(By.TAG_NAME, '定位规则')`
- `find_elements(By.CLASS_NAME, '定位规则')`
- `find_elements(By.CSS_SELECTOR, '定位规则')`

**案例：获取单个节点**

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By

# 获取浏览器驱动对象
browser = webdriver.Chrome()

# 打开指定URL
browser.get('http://news.baidu.com/')

# 定位搜索框
ret = browser.find_element(By.ID, 'ww')
# ret = browser.find_element(By.CSS_SELECTOR, '#ww')  # 查询id为ww
# ret = browser.find_element(By.CSS_SELECTOR, '.word')  # 查询class为word
# ret = browser.find_element(By.XPATH, "//input[@class='word']")
print(ret)

time.sleep(3)
browser.quit()
```

**案例：获取多个节点**

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By

# 获取浏览器驱动对象
browser = webdriver.Chrome()

# 打开指定URL
browser.get('https://movie.douban.com/top250')

# 定位25个电影信息
ret = browser.find_elements(By.CSS_SELECTOR, '.item')  # 查询class为item
print(ret)

ret = browser.find_elements(By.XPATH, "//*[@class='item']")
print(ret)

time.sleep(3)
browser.quit()
```

**注意点**

`find_element`和`find_elements`的区别是：前者返回一个对象，后者返回一个列表

`by_link_text`和`by_partial_link_text`的区别：前者匹配全部文本，后者包含某个文本

> 来自: [7.3.元素定位方法](https://www.yuque.com/tuling_python/spider_base/abaelxo1g9ckgml3)

### 7.4.selenium框架的其他方法

**提取标签内容与属性值**

`find_element`仅仅能够获取元素，不能够直接获取其中的数据，如果需要获取数据需要使用以下方法：

- 获取文本：`element.text`
- 获取属性值：`element.get_attribute("href")`

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By

# 获取浏览器驱动对象
browser = webdriver.Chrome()

# 打开指定URL
browser.get('https://www.douban.com')

# 定位h1标签
ret = browser.find_elements(By.TAG_NAME, "h1")
print(ret[0].text)
# 输出：豆瓣

ret = browser.find_elements(By.LINK_TEXT, "下载豆瓣 App")
print(ret[0].get_attribute("href"))
# 输出：https://www.douban.com/doubanapp/app?channel=nimingye

time.sleep(3)
browser.quit()
```

**处理`cookie`**

通过`driver.get_cookies()`能够获取所有的`cookie`

`cookie`转`dict`

```python
from selenium import webdriver

browser = webdriver.Chrome()
browser.get("http://www.baidu.com")

cookie_list = browser.get_cookies()
print(cookie_list)

# 整理为requests等需要的字典方式，因为浏览器在发送新请求时携带的cookie只有name、value
# 所以此时提取的也只有name、value，其他的不需要
cookie_dict = {x["name"]: x["value"] for x in cookie_list}
print(cookie_dict)

# 关闭页面
browser.close()
```

删除`cookie`

```python
# 删除一条cookie
browser.delete_cookie("CookieName")
# 删除所有的cookie
browser.delete_all_cookies()
```

添加`cookie`

注意点：添加的字段必须包含`name`和`value`，`name`是`cookie`信息中的键，`value`是`cookie`中的值

```python
# 添加cookie与获取指定cookie
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.baidu.com')

browser.add_cookie({'name': 'username', 'value': '安娜'})
browser.add_cookie({'name': 'gender', 'value': '女'})

cookie_list = browser.get_cookies()

cookie_dict = {x["name"]: x["value"] for x in cookie_list}
print(cookie_dict)

print(browser.get_cookie('username'))  # 获取指定cookie
print(browser.get_cookie('gender'))

browser.quit()
```

**页面等待**

如果网站采用了动态`html`技术，那么页面上的部分元素出现时间便不能确定，这个时候就可以设置一个等待时间，强制要求在时间内出现，否则报错。

获取京东网站的搜索输入框

`presence_of_element_located`：判定符合查询条件的一个元素是否存在

`presence_of_elements_located`：判断符合查询条件的所有元素是否存在

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# 创建浏览器驱动对象
browser = webdriver.Chrome()
# 窗口最大化: 保证页面元素不会被其他元素遮挡
browser.maximize_window()

# 创建等待操作对象
wait_ob = WebDriverWait(browser, 10)

# 加载url
browser.get("http://jd.com")

# 等待条件满足
search_input = wait_ob.until(EC.presence_of_element_located((By.ID, 'key')))
# 输入内容
search_input.send_keys("Mac Pro")

# 点击查询
search_button = wait_ob.until(EC.presence_of_element_located((By.XPATH, '//*[@id="search"]/div/div[2]/button')))
search_button.click()

# 延时等待以便于观察页面变化
time.sleep(10)
# 退出
browser.quit()
```

**页面的前进与后退**

```python
import time
from selenium import webdriver

browser = webdriver.Chrome()
browser.get("http://jd.com")

time.sleep(2)
browser.get("http://ganji.com")
time.sleep(2)

# 后退
browser.back()
time.sleep(1)

# 前进
browser.forward()

time.sleep(1)
browser.quit()
```

**动作链**

在`selenium`中，动作链是一种用于模拟用户交互的技术。它允许你执行一系列连续的动作，例如鼠标移动、鼠标点击、按键操作等。通过使用动作链，你可以模拟用户在网页上的复杂交互操作，例如拖拽元素、悬停、双击等。

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver import ActionChains

browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
browser.get(url)

log = browser.find_element(By.XPATH, '//div[@id="iframewrapper"]/iframe')
browser.switch_to.frame(log)
source = browser.find_element(By.CSS_SELECTOR, '#draggable')
target = browser.find_element(By.CSS_SELECTOR, '#droppable')

actions = ActionChains(browser)
actions.drag_and_drop(source, target)
actions.perform()
```

**案例：登录网易邮箱**

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By

class WyMail:
    def __init__(self):
        self.driver = webdriver.Chrome()

    def open_email(self, url):
        self.driver.get(url)
        time.sleep(1)

    def login_email(self, email, password):
        iframe = self.driver.find_element(By.XPATH, '//div[@id="loginDiv"]/iframe[@scrolling = "no"]')

        # 登录表单为一个子页面, 需要切入到当前这个子页面中
        self.driver.switch_to.frame(iframe)

        self.driver.find_element(By.XPATH, '//input[@name="email"]').send_keys(email)
        self.driver.find_element(By.XPATH, '//div[@class="u-input box"]//input[@name="password"]').send_keys(password)

        self.driver.find_element(By.XPATH, './/*[@id="dologin"]').click()

    def close(self):
        time.sleep(10)
        self.driver.quit()

if __name__ == '__main__':
    email = WyMail()
    email.open_email('https://mail.163.com/')
    email.login_email('admin@admin.com', 'admin123')
    email.close()
```

**页面滚动**

大部分网站数据是动态数据，需要触发`ajax`请求后才能在页面中进行数据渲染，触发`ajax`请求的方式之一就是页面滑动，例如爱奇艺、今日头条等网站。

```python
import time
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://36kr.com/')

for num in range(1, 10):
    # 绝对位置
    # browser.execute_script(f'window.scrollTo(0, {num * 700})')

    # 相对位置
    browser.execute_script(f'window.scrollBy(0, {num * 700})')
    time.sleep(1)
```

**绕过检测**

在一些网站当中有专门对浏览器驱动程序进行检测，例如：[https://bot.sannysoft.com/](https://bot.sannysoft.com/)

我们可以谷歌浏览器配置项来隐藏浏览器驱动信息，代码如下：

```python
import time
from selenium import webdriver

# 创建浏览器配置对象
options = webdriver.ChromeOptions()
options.add_argument('--disable-blink-features=AutomationControlled')

browser = webdriver.Chrome(options=options)

browser.get('https://bot.sannysoft.com/')
time.sleep(10)
browser.quit()
```

**浏览器信息配置**

```python
import time
from selenium import webdriver

# 浏览器配置加载
options = webdriver.ChromeOptions()

# 禁止图片加载
prefs = {"profile.managed_default_content_settings.images": 2}
options.add_experimental_option('prefs', prefs)

# 设置UA
user_agent = 'abc'
options.add_argument(f'user-agent={user_agent}')

# 隐藏开发者警告
options.add_experimental_option('useAutomationExtension', False)
options.add_experimental_option('excludeSwitches', ['enable-automation'])

# 设置代理
options.add_argument("--proxy-server=http://127.0.0.1:7890")

# 初始化浏览器对象并加载自定义配置
browser = webdriver.Chrome(options=options)
browser.get('https://www.baidu.com')

# 程序休眠以便观察浏览器中的参数设置
time.sleep(100)
```

**`selenium`中的异常处理**

在`selenium`框架中自定义了一些异常类，可以使用框架定义的异常类进行异常捕获。

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.common.exceptions import NoSuchElementException

browser = webdriver.Chrome()

try:
    browser.find_element(By.ID, 'hello')
except NoSuchElementException:
    print('No Element')
finally:
    browser.close()
```

> 来自: [7.4.selenium框架的其他方法](https://www.yuque.com/tuling_python/spider_base/pb9gsqyog9rf0hic)

### 7.5.项目实战

**案例：唯品会商品数据抓取**

```python
import time
from random import randint
from pymongo import MongoClient
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.common.exceptions import NoSuchElementException

class WpShop:
    mongo_client = MongoClient()
    collection = mongo_client['py_spider']['wp_shop']

    # 创建浏览器配置对象
    options = webdriver.ChromeOptions()
    # 屏蔽图片
    prefs = {"profile.managed_default_content_settings.images": 2}
    options.add_experimental_option('prefs', prefs)

    # 驱动配置
    browser = webdriver.Chrome(options=options)

    # cookies信息
    cookies = {
        "mars_cid": "1730106769949_4ed705c897904ab6da6ed2df7c05b3fe",
        "mars_pid": "0",
        "vip_address": "%257B%2522pid%2522%253A%2522104103%2522%252C%2522cid%2522%253A%2522104103101%2522%252C%2522pname%2522%253A%2522%255Cu6e56%255Cu5357%255Cu7701%2522%252C%2522cname%2522%253A%2522%255Cu957f%255Cu6c99%255Cu5e02%2522%257D",
        "vip_province": "104103",
        "vip_province_name": "%E6%B9%96%E5%8D%97%E7%9C%81",
        "vip_city_name": "%E9%95%BF%E6%B2%99%E5%B8%82",
        "vip_city_code": "104103101",
        "vip_wh": "VIP_HZ",
        "smidV2": "20241028171250025f8fa5509f7d38e275ae812288691700fc76827844722a0",
        "VipRUID": "469642189",
        "VipUID": "b4fcb7035781600c26fa0fd1f5ce4581",
        "VipRNAME": "ph_*****************************739",
        "pc_fdc_area_id": "104103101",
        "pc_fdc_source_ip": "1",
        "VipDFT": "0",
        "vip_ipver": "31",
        "user_class": "b",
        "VipUINFO": "luc%3Ab%7Csuc%3Ab%7Cbct%3Ac_new%7Chct%3Ac_new%7Cbdts%3A0%7Cbcts%3A0%7Ckfts%3A0%7Cc10%3A0%7Crcabt%3A0%7Cp2%3A0%7Cp3%3A1%7Cp4%3A0%7Cp5%3A1%7Cul%3A3105",
        "mst_area_code": "104104",
        "visit_id": "E126929EE78BFFA527BA00E1DF14EADE",
        "VIP_QR_FIRST": "1",
        "vip_access_times": "%7B%22list%22%3A4%7D",
        "vipshop_passport_src": "https%3A%2F%2Fcategory.vip.com%2Fsuggest.php%3Fkeyword%3D%25E7%2594%25B5%25E8%2584%2591%26ff%3D235%257C12%257C1%257C1%26tfs_url%3D%252F%252Fmapi-pc.vip.com%252Fvips-mobile%252Frest%252Fshopping%252Fpc%252Fsearch%252Fproduct%252Frank",
        "_jzqco": "%7C%7C%7C%7C%7C1.132200.1730106770684.1733379498656.1733899693050.1733379498656.1733899693050.0.0.0.5.5",
        "PASSPORT_ACCESS_TOKEN": "A1E6FBB16FE08B779FBC0584A6E6BFB6C216BCA5",
        "VipLID": "0%7C1733899726%7C36a609",
        "VipDegree": "D1",
        "mars_sid": "003a0190be16c7096355fdc127695897",
        "sfl_d": "0",
        ".thumbcache_f65dad1092aa9e66c73b4823b4493a2f": "XyVkJWUOgfG07LTNryoorllgUzp645alot0AsbsCnY2tLH7/WmW5mimEiatW/KXMZcGu4Wavdc6BbnbmKtPjgw%3D%3D",
        "tfs_fp_token": "BXyVkJWUOgfG07LTNryoorllgUzp645alot0AsbsCnY2tLH7/WmW5mimEiatW/KXMZcGu4Wavdc6BbnbmKtPjgw%3D%3D",
        "tfs_fp_timestamp": "1733900421487",
        "pg_session_no": "1",
        "vip_tracker_source_from": "",
        "waitlist": "%7B%22pollingId%22%3A%22BEBAB3CA-A350-4F61-B240-E50056DA4087%22%2C%22pollingStamp%22%3A1733900428480%7D"
    }

    # 发送请求
    @classmethod
    def base(cls):
        cls.browser.get('https://category.vip.com/suggest.php?keyword=%E7%94%B5%E8%84%91&ff=235|12|1|1')
        for key, value in cls.cookies.items():
            cookie_dict = {'name': key, 'value': value}
            cls.browser.add_cookie(cookie_dict)

        # 页面刷新
        cls.browser.refresh()
        time.sleep(randint(1, 3))

    # 页面滚动
    @classmethod
    def drop_down(cls):
        for i in range(1, 12):
            js_code = f'document.documentElement.scrollTop = {i * 1000}'
            cls.browser.execute_script(js_code)
            time.sleep(randint(1, 2))

    # 数据提取
    @classmethod
    def parse_data(cls):
        cls.drop_down()
        div_list = cls.browser.find_elements(
            By.XPATH,
            '//section[@id="J_searchCatList"]/div[@class="c-goods-item  J-goods-item c-goods-item--auto-width"]'
        )

        for div in div_list:
            price = div.find_element(
                By.XPATH,
                './/div[@class="c-goods-item__sale-price J-goods-item__sale-price"]'
            ).text

            title = div.find_element(
                By.XPATH,
                './/div[2]/div[2]'
            ).text

            item = {
                'title': title,
                'price': price
            }
            print(item)
            cls.save_mongo(item)
        cls.next_page()  # 当前页面获取完成之后需要点击下一页

    # 数据保存
    @classmethod
    def save_mongo(cls, item):
        cls.collection.insert_one(item)

    # 翻页
    @classmethod
    def next_page(cls):
        try:
            next_button = cls.browser.find_element(By.XPATH, '//*[@id="J_nextPage_link"]')
            if next_button:
                next_button.click()
                cls.parse_data()  # 进入到下一页需要重新解析页面数据
            else:
                cls.browser.close()
        except NoSuchElementException:
            print('最后一页...')
            cls.browser.quit()

    # 启动函数
    @classmethod
    def main(cls):
        cls.base()
        cls.parse_data()

if __name__ == '__main__':
    WpShop.main()
```

> 来自: [7.5.项目实战](https://www.yuque.com/tuling_python/spider_base/sq24isfclkgc9qpm)

### 7.6.补充 - DrissionPage框架

**引入**

`DrissionPage`与之前学习的`Selenium`大致相同，也是一个基于`Python`的页面自动化工具。区别在于`DrissionPage`既可以控制浏览器也能收发数据包，将两者功能合二为一。功能相对`Selenium`更强大，语法也更加简洁优雅，具有`Selenium`基础之后再学习`DrissionPage`也会更加简单。

官网地址：[https://www.drissionpage.cn/](https://www.drissionpage.cn/)

**环境安装**

操作系统：`Windows`、`Linux`和`Mac`。

`Python`版本：`3.6`及以上

支持浏览器：`Chromium`内核（如`Chrome`和`Edge`）

```shell
pip install DrissionPage
```

**标签定位**

本小节中主要学习如何创建浏览器对象并使用浏览器对象的内置方法完成网站访问以及元素定位。

`ele`：定位符合条件的单个元素（支持`xpath`定位以及选择器定位，可以设定`timeout`超时时间）

官网参数说明：[https://www.drissionpage.cn/browser_control/get_elements/find_in_object/#-ele](https://www.drissionpage.cn/browser_control/get_elements/find_in_object/#-ele)

```python
import time
from DrissionPage.common import By
from DrissionPage import ChromiumPage

# 1.创建浏览器对象并访问百度首页
page = ChromiumPage()
page.get('https://www.baidu.com')

# 2.浏览器最大化
page.set.window.max()

# 3.输入搜索内容
# page.ele('xpath://input[@id="kw"]').input('jk')
# page.ele('xpath://input[@id="su"]').click()

page.ele((By.XPATH, '//input[@id="kw"]')).input('jk')
page.ele((By.XPATH, '//input[@id="su"]')).click()

time.sleep(5)

# 4.关闭浏览器
page.quit()
```

**标签等待**

本小节主要学习`page`对象中的`eles_loaded`方法，用于等待某一个元素或所有元素是否载入到页面中

官网参数说明：[https://www.drissionpage.cn/browser_control/waiting/#-waiteles_loaded](https://www.drissionpage.cn/browser_control/waiting/#-waiteles_loaded)

```python
from DrissionPage import ChromiumPage

page = ChromiumPage()
page.get('https://www.baidu.com')
flag = page.wait.eles_loaded('xpath://input[@id="su"]', timeout=2)
print(flag)

page.quit()
```

**多标签定位**

本小节主要学习`eles`方法定位多标签以及如何使用`ele / eles`对象完成文本数据提取以及属性值的提取

`eles`：定位符合条件的多个元素，返回值类型为列表

官网参数说明：[https://www.drissionpage.cn/browser_control/get_elements/find_in_object/#-eles](https://www.drissionpage.cn/browser_control/get_elements/find_in_object/#-eles)

```python
from DrissionPage import ChromiumPage

page = ChromiumPage()
page.get('https://movie.douban.com/top250')
div_list = page.eles("xpath://ol[@class='grid_view']/li/div[@class='item']")

for item in div_list:
    item_dict = dict()
    movie_name = item.ele('xpath:./div[@class="info"]/div[@class="hd"]//span[1]').text

    movie_image = item.ele('xpath:./div[@class="pic"]/a').attr('href')
    """
    attr: 获取指定的属性值
    attrs: 获取当前标签中的所有属性与值, 返回的数据类型为字典
    详情: https://www.drissionpage.cn/SessionPage/get_ele_info/#%EF%B8%8F%EF%B8%8F-attrs
    """
    # movie_attr = item.ele('xpath:./div[@class="pic"]/a').attrs

    print(movie_name, movie_image)

page.quit()
```

**子页面切换**

在`Selenium`中如果需要控制`iframe`标签中的内容则需要使用`switch_to.frame`方法，`DrissionPage`无需操作。

`DrissionPage`也可以单独对`iframe`进行操作：[https://www.drissionpage.cn/browser_control/iframe](https://www.drissionpage.cn/browser_control/iframe)

```python
import time
from DrissionPage.common import By
from DrissionPage import ChromiumPage

page = ChromiumPage()
page.get('https://www.douban.com')

page.ele((By.CLASS_NAME, 'account-tab-account')).click()
page.ele((By.ID, 'username')).input('admin')
page.ele((By.ID, 'password')).input('admin123')

time.sleep(3)
page.quit()
```

```python
import time
from DrissionPage import ChromiumPage

class MyEmail:
    def __init__(self):
        self.page = ChromiumPage()
        self.url = 'https://mail.163.com/'

    def login_email(self, email, password):
        self.page.get(self.url)
        self.page.ele('xpath://input[@name="email"]').input(email)
        self.page.ele('xpath://div[@class="u-input box"]//input[@name="password"]').input(password)
        self.page.ele('xpath://*[@id="dologin"]').click()
        time.sleep(10)
        self.page.quit()

if __name__ == '__main__':
    my_email = MyEmail()
    my_email.login_email('admnin@163.com', 'admin123')
```

**接口监听**

本小节主要学习如何使用`DrissionPage`监听指定的`api`接口并直接获取接口中的数据

`page.listen.start('接口地址')`：启动监听器

`page.listen.steps(count=你要获取的数据包总数)`：返回一个可迭代对象，用于`for`循环，每次循环可从中获取到数据包

`.response.body`：获取数据包中的数据

网站请求地址：[http://www.ccgp-hunan.gov.cn/page/content/more.jsp?column_code=2](http://www.ccgp-hunan.gov.cn/page/content/more.jsp?column_code=2)

官网参数说明：[https://www.drissionpage.cn/browser_control/listener/#-listenstart](https://www.drissionpage.cn/browser_control/listener/#-listenstart)

```python
from DrissionPage.common import By
from DrissionPage import ChromiumPage

page = ChromiumPage()
url = 'http://www.ccgp-hunan.gov.cn/page/content/more.jsp?column_code=2'

# api监听: 先监听再请求
page.listen.start('/mvc/getContentList.do')
page.get(url)

# 获取api返回的数据(page.listen.steps(): 返回的是一个迭代器)
# print(page.listen.steps())

# 循环获取数据(count: 监听次数)
page_num = 1
for item in page.listen.steps(count=8):
    # 打印监听后返回的对象
    # print(item)

    # 打印数据
    print(f'第{page_num}页(数据类型[{type(item.response.body[0])}]):', item.response.body)

    # 下一页
    flag = page.ele((By.LINK_TEXT, '下一页'), timeout=3)
    if flag:
        flag.click()
    else:
        print('翻页结束...')
        page.quit()
        break
    page_num += 1
```

**动作链**

本小节主要学习`actions`动作链对象所提供的内置方法

官网内置方法说明：[https://www.drissionpage.cn/browser_control/actions/#%EF%B8%8F-%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95](https://www.drissionpage.cn/browser_control/actions/#%EF%B8%8F-%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95)

```python
import time
from DrissionPage.common import By
from DrissionPage import ChromiumPage

page = ChromiumPage()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
page.get(url)

source = page.ele((By.ID, 'draggable'))
target = page.ele((By.ID, 'droppable'))

page.actions.hold(source).release(target)
time.sleep(2)
page.quit()
```

```python
import time
from DrissionPage.common import By
from DrissionPage import ChromiumPage

url = 'https://ynjzjgcx.com/dataPub/enterprise'
page = ChromiumPage()
page.get(url)

# 定位滑动控件并选中控件
button = page.ele((By.CLASS_NAME, 'slide-verify-slider-mask-item'))
page.actions.hold(button)

# 向右移动指定的像素位并释放
page.actions.right(150).release()

time.sleep(3)
page.quit()

"""
绕过验证码需要专门学习ocr图像识别库
"""
```

**`SessionPage`的使用**

`SessionPage`基于`requests`进行网络连接，因此可使用`requests`内置的所有请求方式，包括`get()`、`post()`、`head()`、`options()`、`put()`、`patch()`、`delete()`。

官网内置方法说明：[https://www.drissionpage.cn/SessionPage/visit/](https://www.drissionpage.cn/SessionPage/visit/)

```python
from DrissionPage import SessionPage

url = 'http://www.ccgp-hunan.gov.cn/mvc/getContentList.do'
form_data = {
    'column_code': 2,
    'title': '',
    'pub_time1': '',
    'pub_time2': '',
    'own_org': 1,
    'page': 1,
    'pageSize': 18
}

page = SessionPage()
page.post(url, data=form_data)
print(page.user_agent)  # 自动构建请求头
print(page.response.json())
# page.close()
```

**`WebPage`的使用**

`WebPage`是整合了上面两者的页面对象，既可控制浏览器，又可收发数据包，并且可以在这两者之间共享登录信息。

它有两种工作模式：`d`模式和`s`模式。`d`模式用于控制浏览器，`s`模式用于收发数据包。`WebPage`可在两种模式间切换，但同一时间只能处于其中一种模式。

当前对象在新版本中已被作者刻意淡化，不建议使用。

官方文档说明：[https://drissionpage.cn/DP32Docs/get_start/basic_concept/#webpage](https://drissionpage.cn/DP32Docs/get_start/basic_concept/#webpage)

```python
import time
from DrissionPage import WebPage

# 创建页面对象，初始d模式
page = WebPage('d')

# 访问百度
page.get('http://www.baidu.com')

# 定位输入框并输入关键字
page.ele('#kw').input('DrissionPage')

# 点击"百度一下"按钮
page.ele('@value=百度一下').click()

# 等待页面加载
page.wait.load_start()

# 切换到s模式
page.change_mode()

# 获取所有结果元素
results = page.eles('tag:h3')

# 遍历所有结果
for result in results:
    # 打印结果文本
    print(result.text)

time.sleep(3)
page.quit()
```

**请求头加密的爬虫案例**

网站访问地址：[https://kaoyan.docin.com/pdfreader/web/#/docin/documents?type=1&keyword=%E5%A4%8D%E8%AF%95%E4%BB%BF%E7%9C%9F%E6%A8%A1%E6%8B%9F](https://kaoyan.docin.com/pdfreader/web/#/docin/documents?type=1&keyword=%E5%A4%8D%E8%AF%95%E4%BB%BF%E7%9C%9F%E6%A8%A1%E6%8B%9F)

```python
import requests

headers = {
    "Accept": "application/json, text/plain, */*",
    "Accept-Language": "zh-CN,zh;q=0.9",
    "Cache-Control": "no-cache",
    "Connection": "keep-alive",
    "Content-Type": "application/json",
    "Origin": "https://kaoyan.docin.com",
    "Pragma": "no-cache",
    "Referer": "https://kaoyan.docin.com/",
    "Sec-Fetch-Dest": "empty",
    "Sec-Fetch-Mode": "cors",
    "Sec-Fetch-Site": "cross-site",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36",
    "X-Application": "Pdfreader.Web",
    "X-Nonce": "02fec55d-8a78-2453-cb18-42f62ef46732",
    "X-Sign": "C0FC7B5784DDE2FD774DD00C6AB89BE5",
    "X-Timestamp": "1733317931",
    "X-Token": "null",
    "X-Version": "V2.2",
    "sec-ch-ua": "Google Chrome;v=131, Chromium;v=131, Not_A Brand;v=24",
    "sec-ch-ua-mobile": "?0",
    "sec-ch-ua-platform": "macOS"
}
url = "https://www.handebook.com/api/web/document/list"
json_data = {
    "SearchType": 0,
    "SearchKeyword": "复试仿真模拟",
    "DocumentType": " ",
    "UniversityCode": "",
    "MajorCode": "",
    "ExamSubjectList": [],
    "PageIndex": 1,
    "PageSize": 30
}

response = requests.post(url, headers=headers, json=json_data)
print(response.json())
```

```python
import pymongo
from DrissionPage import WebPage
from DrissionPage.common import By

class DouDing:
    mongo_client = pymongo.MongoClient()

    def __init__(self):
        self.url = 'https://kaoyan.docin.com/pdfreader/web/#/docin/documents?type=1&keyword=%E5%A4%8D%E8%AF%95%E4%BB%BF%E7%9C%9F%E6%A8%A1%E6%8B%9F'
        # self.page = WebPage(mode='d')  # mode为设置session模式或者driver模式, 默认为driver
        self.page = WebPage()

    def get_info(self):
        self.page.listen.start('/api/web/document/list')
        self.page.get(self.url)
        for item in self.page.listen.steps(count=20):
            # print(item)
            self.parse_info(item.response.body)

            # 翻页
            self.page.ele((By.CLASS_NAME, 'btn-next'), timeout=3).click()
        self.page.quit()

    def parse_info(self, info):
        for temp in info['Data']['DocumentInfos']:
            item = dict()
            item['DocumentGuid'] = temp['DocumentGuid']
            item['DocumentName'] = temp['DocumentName']
            item['DocumentPrice'] = temp['DocumentPrice']
            self.save_info(item)

    def save_info(self, info_dict):
        collection = self.mongo_client['py_spider']['dou_ding']
        collection.insert_one(info_dict)
        print('保存成功:', info_dict)

if __name__ == '__main__':
    dou_ding = DouDing()
    dou_ding.get_info()
```

**文件下载**

`DrissionPage`内置下载器，无需手动构建`open()`文件对象

官网说明：[https://www.drissionpage.cn/download/intro](https://www.drissionpage.cn/download/intro)

```python
import os
from DrissionPage import SessionPage

url = 'https://www.lpbzj.vip/allimg'
page = SessionPage()
page.get(url)
element_div = page.s_eles("xpath://div[@id='posts']")[0]
detail_url_list = element_div.s_eles("xpath:.//div[@class='img']/a")

save_path = './美女写真/'
os.makedirs(save_path, exist_ok=True)  # 确保保存目录存在

for detail_url in detail_url_list:
    page.get(detail_url.attr('href'))
    div_element = page.s_eles("xpath://div[@class='article-content clearfix']")[0]
    image_url_list = div_element.s_eles("xpath:.//img")
    for image_url in image_url_list:
        img_src = image_url.attr('src')

        # 同步下载
        # res = page.download(img_src, save_path)
        # print('task status:', res)

        # 添加下载任务并发下载
        page.download.add(img_src, save_path)
        print(f"Downloading image from: {img_src}")
```

**项目实战**

唯品会接口数据抓取

```python
import json
from DrissionPage.common import By
from DrissionPage import ChromiumPage

page = ChromiumPage()
url = 'https://category.vip.com/suggest.php?keyword=%E7%94%B5%E8%84%91&ff=235%7C12%7C1%7C1&tfs_url=%2F%2Fmapi-pc.vip.com%2Fvips-mobile%2Frest%2Fshopping%2Fpc%2Fsearch%2Fproduct%2Frank&page=1'
page.listen.start('vips-mobile/rest/shopping/pc/product/module/list/v2')
page.get(url)

for item in page.listen.steps():
    response_body = item.response.body

    try:
        # 提取 JSON 字符串部分
        start_index = response_body.find('{')
        end_index = response_body.rfind('}') + 1
        json_str = response_body[start_index:end_index]

        info_dict = json.loads(json_str)

        for temp in info_dict['data']['products']:
            shop_info = dict()
            shop_info['商品名称'] = temp['title']
            shop_info['商品品牌'] = temp['brandShowName']
            shop_info['商品价格'] = temp['price']['salePrice']
            print(shop_info)

        button = page.ele((By.CLASS_NAME, 'cat-paging-next'), timeout=3)
        if button:
            button.click()
        else:
            print('爬虫结束...')
            page.quit()
    except AttributeError:
        print('数据为空:', response_body)
```

小红书首页数据抓取

```python
from DrissionPage import ChromiumPage

page = ChromiumPage()
page.listen.start('api/sns/web/v1/homefeed')
page.get('https://www.xiaohongshu.com/explore')

while True:
    js_code = f"document.documentElement.scrollTop = document.documentElement.scrollHeight * {1000}"
    page.run_js(js_code)

    # 未获取返回false, 获取则返回列表, 列表元素类型为DataPacket
    is_api_list = page.listen.wait(count=5, timeout=1)
    print('数据状态:', is_api_list)
    if is_api_list:
        for item in is_api_list:
            print(item.response.body)
```

补充 - 小红书自动评论脚本（自行研究）

```python
import time
from DrissionPage import ChromiumPage

search = str(input("输入关键词："))
content = str(input("输入评论内容："))

page = ChromiumPage()
page.get('https://www.xiaohongshu.com/search_result?keyword=' + search + '&source=unknown&type=51')
print("网站加载成功！")
time.sleep(2)
for time_button in range(1, 20):  # 下滑多少下 你就改多少下
    time.sleep(2)
    page.scroll.to_bottom()
    print("当前下滑：", time_button, "次，剩余", 20 - time_button, "次后，将会开始抓取数据...")
print("全部下滑完毕开始抓取页面的元素链接！")
my_list = list()
ele = page.eles('.cover ld mask')
name_ele = page.eles('.title')
for href, name in zip(ele, name_ele):
    lian = href.link
    na = name.text
    print(na, lian)
    my_list.extend(lian.split(','))

sums = 0

print("本次获取数据：", len(my_list), "条")
for like_list in my_list:
    sums = sums + 1
    print("序号：", sums, "链接：", like_list)
    page.get(like_list)
    time.sleep(1)
    input_list1 = page.ele('.chat-wrapper').click()
    input_list2 = page.ele('.content-input').input(content)
    time.sleep(0.5)
    button = page.ele('.btn submit').click()
    print("发送成功：", content, "-")
    time.sleep(2)
print("*" * 30)
input("主程序结束...")
```

> 来自: [7.6.补充 - DrissionPage框架](https://www.yuque.com/tuling_python/spider_base/xa66dh25hbo6x6ox)

## 8.代理池的搭建

**使用`IP`的原因与作用**

首先代理`ip`可以保护用户信息的安全。在如今的大数据互联网时代，每个人上网总会留下一点信息，很有可能被别人利用，而使用代理`ip`可以完美解决这个问题。高匿名代理`ip`可以隐藏用户的真实`ip`地址，保护用户的个人数据和信息安全，提高用户上网的安全性。

其次可以提高访问速度，有时出现过访问网页时出现卡顿的问题，通过代理`ip`一定程度上可以解决这个问题。通过代理`ip`访问的一些网站等信息会存留在代理服务器的缓冲区内，假如别人访问过的信息你再访问，则会直接在缓冲区内拉取数据，进一步提高访问速度。遇到对`ip`检测比较严格的网址也需要进行替换。

**不同的代理类型**

开放代理：是由全网扫描而来，就是别人搭建了代理服务器被扫到了拿来用，采用分布在全球各地的云服务器使用扫描软件，借助于`nmap`工具，`7*24*365`不间断全网扫描、验证。开放代理容易随时失效存在稳定性差、速度不稳定、安全性、可用率低等问题，目前市面上很多都是这种开放代理，价格低廉，我们不推荐这种代理，建议考虑私密代理。

私密代理：非扫描而来的，而是`ip`提供商租用全国各地实体服务器或拨号服务器，采用自研的服务端代理程序和高可用的调度系统，并支持`Http/Https/Scoks5`等协议，具有高匿、高速、稳定的特点。

独享代理：是私密代理的一种，客户需要长期稳定的长效`IP`使用。

**免费代理网址**

可以从提供的免费代理网址中采集`ip`地址，不保证可用：

```plain
西刺代理: http://www.xicidaili.com
快代理: https://www.kuaidaili.com
云代理: http://www.ip3366.net
无忧代理: http://www.data5u.com/
360代理: http://www.swei360.com
66ip代理: http://www.66ip.cn
ip海代理: http://www.iphai.com
大象代理: http://www.daxiangdaili.com/
米扑代理: https://proxy.mimvp.com/freesecret.php
站大爷: http://ip.zdaye.com/
讯代理: http://www.xdaili.cn/
蚂蚁代理: http://www.mayidaili.com/free
89免费代理: http://www.89ip.cn/
全网代理: http://www.goubanjia.com/buy/high.html
开心代理: http://www.kxdaili.com/dailiip.html
猿人云: https://www.apeyun.com/
```

> 来自: [8.代理池的搭建](https://www.yuque.com/tuling_python/spider_base/ycb0w9hp2cpy7147)

### 8.1.免费代理采集脚本

公开的免费`ip`多数是不可用的，不建议在爬虫项目中使用免费代理。

以下为免费代理采集脚本示例：

```python
import re
import json
import requests

class FreeAgent:
    def __init__(self):
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36'
        }

    def get_ip(self, page):
        url = f'https://www.kuaidaili.com/free/inha/{page}/'
        response = requests.get(url, headers=self.headers).text
        data = re.findall(r'const fpsList = (.*?);', response)[0]
        ip_pattern = r'"ip": "(\d{1,3}(?:\.\d{1,3}){3})"'
        port_pattern = r'"port": "(\d{1,5})"'

        ips = re.findall(ip_pattern, data)
        ports = re.findall(port_pattern, data)
        for temp in zip(ips, ports):
            ip_dict = dict()
            ip_dict['ip'] = temp[0]
            ip_dict['port'] = temp[1]
            yield ip_dict

    def test_ip(self, max_page_num):
        for page_num in range(1, max_page_num + 1):
            for result in self.get_ip(page_num):
                proxies = {
                    'http': 'http://' + result['ip'] + ':' + result['port'],
                    'https': 'http://' + result['ip'] + ':' + result['port']
                }

                print('ip代理:', proxies)
                try:
                    response = requests.get('http://httpbin.org/ip',
                                            headers=self.headers, proxies=proxies, timeout=3)
                    if response.status_code == 200:
                        print(response.text)
                        with open('success_ip.txt', 'a', encoding='utf-8') as f:
                            f.write(json.dumps(proxies, ensure_ascii=False, indent=4) + '\n')
                except Exception as e:
                    print('请求超时:', e)

free_agent = FreeAgent()
free_agent.test_ip(10)
```

> 来自: [8.1.免费代理采集脚本](https://www.yuque.com/tuling_python/spider_base/hyrt9tu1weh2td3f)

### 8.2.付费代理的设置与搭建

**付费代理平台**

- [https://proxy.ip3366.net/](https://proxy.ip3366.net/)
- [https://www.kuaidaili.com/](https://www.kuaidaili.com/)

选择自己喜欢的代理平台自行购买：不同的代理平台都有对应的`api`文档。在代理平台中需要自己注册账号并实名认证。

快代理官方使用文档：[https://www.kuaidaili.com/doc/dev/sdk_http/#requests](https://www.kuaidaili.com/doc/dev/sdk_http/#requests)

**爬虫案例 - 当当网商品数据采集**

```python
import os
import time
import pymongo
import requests
import threading
from lxml import etree
from loguru import logger
from retrying import retry
from queue import Queue, Empty
from fake_useragent import UserAgent

class DangDangShop:
    mongo_client = pymongo.MongoClient()
    collection = mongo_client['py_spider']['dangdang_shop']

    def __init__(self):
        self.url = 'https://search.dangdang.com/?key=python&act=input'
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36'
        }
        self.ip_url = 'https://dps.kdlapi.com/api/getdps/?secret_id=orsnof8y6rxqtgnf2v33&signature=zugc8en4ct195aoish9iwma4dnvgeya5&num=1&pt=1&format=text&sep=1'

        self.ip_queue = Queue()  # IP队列
        self.url_queue = Queue()  # URL队列
        self.response_queue = Queue()  # 响应队列
        self.detail_queue = Queue()  # 商品信息队列

    def fetch_proxy(self, stop_event, min_count=5):
        while not stop_event.is_set():
            if self.ip_queue.qsize()  来自: [8.2.付费代理的设置与搭建](https://www.yuque.com/tuling_python/spider_base/zkzi4ut0wprk9fft)

## 9.scrapy框架 - 基础操作

**学习`scrapy`的原因**

- `scrapy`不能解决剩下的`10%`的爬虫需求
- 能够让开发过程方便、快速
- `scrapy`框架能够让我们的爬虫效率更高，并且代码结构更清晰

**什么是`scrapy`**

文档地址：

- 中文版（2.5版本）：[https://www.osgeo.cn/scrapy/intro/install.html](https://www.osgeo.cn/scrapy/intro/install.html)
- 最新版（英文）：[https://docs.scrapy.org/en/latest/index.html](https://docs.scrapy.org/en/latest/index.html)

`scrapy`使用了`Twisted['twɪstɪd]`异步网络框架，可以加快我们的下载速度。

`scrapy`是一个为了爬取网站数据，提取结构性数据而编写的应用框架，我们只需要实现少量的代码，就能够快速的抓取。

**框架安装与环境测试**

安装指令

```plain
# 注意: 需要将scrapy框架限制在2.6.3版本
pip install scrapy==2.6.3 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

环境测试

在终端中输入`scrapy`，如果能看到如下效果，说明安装成功。

![](images/image-031.png)

> 来自: [9.scrapy框架 - 基础操作](https://www.yuque.com/tuling_python/spider_base/tmfld8zosg67xos8)

### 9.1.scrapy项目的创建与启动

**创建项目**

```bash
# scrapy startproject 项目名称
scrapy startproject mySpider
```

![](images/image-032.png)

执行完成后会创建一个项目文件夹，文件夹结构如下：

![](images/image-033.png)

**创建爬虫**

```bash
# scrapy genspider 爬虫名 允许爬取的域名
scrapy genspider baidu baidu.com
```

![](images/image-034.png)

生成的爬虫代码文件如下：

```python
import scrapy

class BaiduSpider(scrapy.Spider):
    # 爬虫名称
    name = "baidu"

    # 允许爬取的域名
    allowed_domains = ["baidu.com"]
    # 爬取地址
    start_urls = ["https://baidu.com"]

    # 数据解析方法，之后需要我们自己编写逻辑
    def parse(self, response):
        pass
```

**运行爬虫文件**

在项目目录下执行`scrapy crawl 爬虫名称`指令：

```bash
# 启动命令中的爬虫名称需要和创建时设置的爬虫名称保持一致
scrapy crawl baidu
```

![](images/image-035.png)

注意点：

- 这个案例并没有对爬取到的数据进行任何操作，怎样提取数据等后面讲解
- 本案例就是让大家对`scrapy`的使用大体有个了解即可

**启动报错解决**

如果运行项目出现以下报错：

```plain
AttributeError: 'AsyncioSelectorReactor' object has no attribute '_handleSignals'
```

将`scrapy`中的`Twisted`降级到`22.10.0`版本：

```plain
pip install Twisted==22.10.0
```

**总结**

- 用`scrapy`来实现爬虫，并不是我们想象的那样创建一个`.py`然后在这个文件中`import`导入，而是通过`scrapy`命令来构建相关的目录结构，最终通过命令来开启`scrapy`
- `scrapy`框架的项目创建与项目启动流程：

- 创建`scrapy`项目，命令是`scrapy startproject 项目名称`
- 进入到创建出来的`scrapy`项目文件夹
- 创建爬虫，命令是：`scrapy genspider 爬虫名 允许爬取的域名`
- 运行`scrapy`，命令是：`scrapy crawl 爬虫名`

> 来自: [9.1.scrapy项目的创建与启动](https://www.yuque.com/tuling_python/spider_base/ei87cty5tib7ty6l)

### 9.2.parse方法中的响应对象

**案例需求**

我们以爬取`蜻蜓FM排行榜`为例进行学习如何使用`Scrapy`提取数据

网站地址：[https://m.qingting.fm/rank/](https://m.qingting.fm/rank/)

![](images/image-036.png)

**项目创建**

```bash
# 创建项目目录
scrapy startproject fm

# 进入项目根目录
cd fm

# 创建爬虫文件
scrapy genspider qingting https://m.qingting.fm/rank/
```

**`parse`方法**

`scrapy`请求成功后会将`response`对象传递给`parse`方法

```python
import scrapy
from scrapy import cmdline

# 导入response响应对象的类型
from scrapy.http import HtmlResponse

class QingtingSpider(scrapy.Spider):
    # 爬虫名称
    name = "qingting"

    # 允许抓取的域名
    allowed_domains = ["m.qingting.fm"]
    # 抓取地址
    start_urls = ["https://m.qingting.fm/rank/"]

    # 设定当前函数接收的response对象的类型，方便语法提示
    def parse(self, response: HtmlResponse, **kwargs):
        """
        :param response: 当scrapy请求成功后会返回response对象, 由parse方法接收
        :return:
        此函数为回调函数，当对start_url进行请求后，会将请求完成的响应对象传递给此函数
            response参数接收响应对象
        """
        # 验证当前函数是否被调用
        print(response)

if __name__ == '__main__':
    # 使用scrapy框架自带的命令工具来启动爬虫脚本
    cmdline.execute('scrapy crawl qingting'.split())
```

知识点补充

在`scrapy`框架中可以使用`cmdline.execute()`方法来执行启动命令，启动命令有以下两种：

```python
from scrapy import cmdline  # 导入运行指令的工具

# 默认启动方式，打印日志信息
cmdline.execute('scrapy crawl qingting'.split())

# 忽略日志信息
cmdline.execute('scrapy crawl qingting --nolog'.split())
```

> 来自: [9.2.parse方法中的响应对象](https://www.yuque.com/tuling_python/spider_base/xi521xoor9woy1l6)

### 9.3.响应对象中的属性与方法

通过上一小节的代码示例可以看出，`parse`方法是在`scrapy`运行时被自动调用的，且从默认生成的`parse`方法的参数名是`response`就能够看出，`parse`方法应该是`scrapy`对`start_urls`中的`url`爬取之后，接收的响应对象。

所以，我们只需要在`parse`方法中使用之前学习过的提取数据的方式提取数据即可，例如正则表达式、`xpath`等。

**`response`对象的常用属性**

为了能够在`parse`函数中对`response`进行操作，下面列举了常用的`response`属性：

- `response.url`：当前响应的`url`地址
- `response.request.url`：当前响应对应的请求的`url`地址
- `response.headers`：响应头
- `response.request.headers`：当前响应的请求头
- `response.body`：响应体，数据返回类型为`byte`类型
- `response.status`：响应状态码
- `response.text`：文本数据

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class QingtingSpider(scrapy.Spider):
    # 爬虫名称
    name = "qingting"

    # 允许抓取的域名
    allowed_domains = ["m.qingting.fm"]
    # 抓取地址
    start_urls = ["https://m.qingting.fm/rank/"]

    def parse(self, response: HtmlResponse, **kwargs):
        # response常用属性的使用
        print("---1--->", response.url)  # 响应的url地址
        print("---2--->", response.headers)  # 响应头
        print("---3--->", response.status)  # 响应状态码
        print("---4--->", response.body)  # 响应体, 类型为字节
        print("---5--->", response.request.url)  # 请求地址
        print("---6--->", response.request.headers)  # 请求头
        print("---7--->", response.text)  # 文本数据

if __name__ == '__main__':
    cmdline.execute('scrapy crawl qingting'.split())
```

**数据解析**

可以使用`response`响应对象中提供的`xpath`方法提取数据

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class QingtingSpider(scrapy.Spider):
    # 爬虫名称
    name = "qingting"

    # 允许抓取的域名
    allowed_domains = ["m.qingting.fm"]
    # 抓取地址
    start_urls = ["https://m.qingting.fm/rank/"]

    def parse(self, response: HtmlResponse, **kwargs):
        a_list = response.xpath('//div[@class="rank-list"]/a')
        # print(a_list)
        for a_temp in a_list:
            rank_number = a_temp.xpath('./div[@class="badge"]/text()')  # 排名
            img_url = a_temp.xpath('./img/@src')  # 图片地址
            title = a_temp.xpath('./div[@class="content"]/div[@class="title"]/text()')  # 标题
            desc = a_temp.xpath('./div[@class="content"]/div[@class="desc"]/text()')  # 描述
            play_number = a_temp.xpath('.//div[@class="info-item"][1]/span/text()')  # 播放次数
            print('---***--->', rank_number, img_url, title, desc, play_number)

if __name__ == '__main__':
    cmdline.execute('scrapy crawl qingting'.split())
```

以上案例中使用`response.xpath`方法所获取的数据是类似列表的数据集，其中包含的是`selector`对象，操作和列表一样，但是有一些额外的方法。

- `extract`方法：返回一个包含有字符串的列表
- `extract_first`方法：返回列表中的第一个字符串，列表为空返回`None`

`extract()`方法

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class QingtingSpider(scrapy.Spider):
    name = 'qingting'
    allowed_domains = ['qingting.fm']
    start_urls = ['https://m.qingting.fm/rank/']

    def parse(self, response: HtmlResponse, **kwargs):
        a_list = response.xpath("//div[@class='rank-list']//a")
        for a_temp in a_list:
            rank_number = a_temp.xpath("./div[@class='badge']//text()").extract()
            img_url = a_temp.xpath("./img/@src").extract()
            title = a_temp.xpath("./div[@class='content']/div[@class='title']/text()").extract()
            desc = a_temp.xpath("./div[@class='content']/div[@class='desc']/text()").extract()
            play_number = a_temp.xpath(".//div[@class='info-item'][1]/span/text()").extract()
            print("---***--->", rank_number, img_url, title, desc, play_number)

if __name__ == '__main__':
    cmdline.execute('scrapy crawl qingting'.split())
```

返回结果如下：

![](images/image-037.png)

`extrace_first()`方法

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class QingtingSpider(scrapy.Spider):
    name = 'qingting'
    allowed_domains = ['qingting.fm']
    start_urls = ['https://m.qingting.fm/rank/']

    def parse(self, response: HtmlResponse, **kwargs):
        a_list = response.xpath("//div[@class='rank-list']//a")
        for a_temp in a_list:
            rank_number = a_temp.xpath("./div[@class='badge']//text()").extract_first()
            img_url = a_temp.xpath("./img/@src").extract_first()
            title = a_temp.xpath("./div[@class='content']/div[@class='title']/text()").extract_first()
            desc = a_temp.xpath("./div[@class='content']/div[@class='desc']/text()").extract_first()
            play_number = a_temp.xpath(".//div[@class='info-item'][1]/span/text()").extract_first()
            print("---***--->", rank_number, img_url, title, desc, play_number)

if __name__ == '__main__':
    cmdline.execute('scrapy crawl qingting'.split())
```

返回结果如下：

![](images/image-038.png)

**解析警告**

如果在解析过程中出现如下警告：

```bash
UserWarning: Selector got both text and root, root is being ignored.
  super().__init__(text=text, type=st, root=root, **kwargs)
```

![](images/image-039.png)

将`parsel`依赖包降级到`1.7.0`

```plain
pip install parsel==1.7.0
```

**总结**

- `parse`方法是`scrapy`在得到`HTTP(S)`响应之后的回调方法。
- `parse`方法中默认的参数是响应对象，这个对象可以直接使用`xpath`进行数据的提取，使得在处理非结构化数据（一般指`html`文件）时非常方便。

> 来自: [9.3.响应对象中的属性与方法](https://www.yuque.com/tuling_python/spider_base/lhng7gqcs1eknddt)

### 9.4.管道的基本使用

**管道的作用**

对`parse`函数中提取到的数据进一步处理的操作，例如保存到`csv`、`MongoDB`、`MySQL`等。

**管道的触发条件**

修改爬虫文件`qingting.py`中`parse()`方法

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class QingtingSpider(scrapy.Spider):
    name = 'qingting'
    allowed_domains = ['qingting.fm']
    start_urls = ['https://m.qingting.fm/rank/']

    def parse(self, response: HtmlResponse, **kwargs):
        a_list = response.xpath("//div[@class='rank-list']/a")
        for a_temp in a_list:
            rank_number = a_temp.xpath("./div[@class='badge']//text()").extract_first()
            img_url = a_temp.xpath("./img/@src").extract_first()
            title = a_temp.xpath("./div[@class='content']/div[@class='title']/text()").extract_first()
            desc = a_temp.xpath("./div[@class='content']/div[@class='desc']/text()").extract_first()
            play_number = a_temp.xpath(".//div[@class='info-item'][1]/span/text()").extract_first()

            # 使用yield关键字将解析的数据返回给pipline
            yield {
                'rank_number': rank_number,
                'img_url': img_url,
                'title': title,
                'desc': desc,
                'play_number': play_number
            }

if __name__ == '__main__':
    cmdline.execute('scrapy crawl qingting --nolog'.split())
```

思考：为什么使用`yield`返回数据？

- 遍历这个函数的返回值的时候，挨个把数据读到内存，不会造成内存的瞬间占用过高，`Python3`中的`range`和`python2`中的`xrange`同理。
- `scrapy`是异步爬取，所以通过`yield`能够将运行权限教给其他的协程任务去执行，这样整个程序运行效果会更高。

注意点：解析函数中的`yield`能够传递的对象只能是：`BaseItem`、`Request`、`dict`、`None`

在项目中找到`piplines.py`文件并修改

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html

# useful for handling different item types with a single interface
from itemadapter import ItemAdapter

class FmPipeline:
    def process_item(self, item, spider):
        """
        :param item: parse方法返回的数据
        :param spider: 爬虫名称

        当前函数是一个回调函数，在spider爬虫文件中使用了yield返回数据之后，则自动调用管道方法
            注意点: 需要在配置文件中打开管道配置
        """
        print(item)
        # return item
```

在`settings.py`文件中开启管道配置

![](images/image-040.png)

> 来自: [9.4.管道的基本使用](https://www.yuque.com/tuling_python/spider_base/zfuvm4vwhrn03sqo)

### 9.5.scrapy框架的内置模块与执行流程

**`scrapy`内置模块图解**

![](images/image-041.png)

注意：爬虫中间件和下载中间件只是运行逻辑的位置不同，作用是重复的：如替换`User-Agent`等。

**`scrapy`框架的执行流程**

![](images/image-042.png)

上图中的`1 - 12`序号的解释说明：

- `scrapy`从`spider`子类中提取`start_urls`，然后构造为`request`请求对象
- 将`request`请求对象传递给爬虫中间件
- 将`request`请求对象传递给`scrapy`引擎（就是核心代码）
- 将`request`请求对象传递给调度器（它负责对多个`request`调度，好比交通管理员负责交通的指挥员）
- 将`request`请求对象传递给`scrapy`引擎
- `scrapy`引擎将`request`请求对象传递给下载中间件（可以更换代理`IP`，更换`Cookies`，更换`User-Agent`，自动重试。等）
- `request`请求对象传给到下载器（下载器通过异步的方式发送`HTTP(S)`请求），得到响应封装为`response`对象
- 将`response`对象传递给下载中间件
- 下载中间件将`response`对象传递给`scrapy`引擎
- `scrapy`引擎将`response`对象传递给爬虫中间件（这里可以处理异常等情况）
- 爬虫对象中的`parse`函数被调用（在这里可以对得到的`response`对象进行处理，例如用`status`得到响应状态码，`xpath`可以进行提取数据等）
- 将提取到的数据传递给`scrapy`引擎，它将数据再传递给管道（在管道中我们可以将数据存储到`csv`、`MongoDB`等）

> 来自: [9.5.scrapy框架的内置模块与执行流程](https://www.yuque.com/tuling_python/spider_base/pmegmprq39ug6kwd)

### 9.6.多数据保存

在解析`response`响应对象的过程当中，解析出来的数据可能是一个新的可访问的`URL`，如果需要对解析出来的`URL`地址进行请求并获取数据该如何完成？

**确定需求**

从响应对象中提取`URL`，对这样的`URL`也发送请求然后提取它的数据。

**项目准备**

- 创建`蜻蜓FM`项目：`scrapy startproject fm`
- 进入到`fm`文件夹，创建`qingting`爬虫：`scrapy genspider qingting qingting.fm/rank`
- 编辑`spiders/qingting.py`

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class QingtingSpider(scrapy.Spider):
    name = 'qingting'

    # 注意点: 当前图片域名与蜻蜓FM域名不一致
    # allowed_domains = ['qingting.fm']
    allowed_domains = ['qingting.fm', 'pic.qtfm.cn']
    start_urls = ['https://m.qingting.fm/rank/']

    def parse(self, response: HtmlResponse, **kwargs):
        a_list = response.xpath("//div[@class='rank-list']/a")
        for a_temp in a_list:
            rank_number = a_temp.xpath("./div[@class='badge']//text()").extract_first()
            img_url = a_temp.xpath("./img/@src").extract_first()
            title = a_temp.xpath("./div[@class='content']/div[@class='title']/text()").extract_first()
            desc = a_temp.xpath("./div[@class='content']/div[@class='desc']/text()").extract_first()
            play_number = a_temp.xpath(".//div[@class='info-item'][1]/span/text()").extract_first()

            # 使用yield关键字将解析的数据返回给pipline
            yield {
                'rank_number': rank_number,
                'img_url': img_url,
                'title': title,
                'desc': desc,
                'play_number': play_number
            }

            # 构造新的请求对象: 使用cb_kwargs传递形参
            yield scrapy.Request(img_url, callback=self.parse_image, cb_kwargs={"image_name": title})

    # 解析图片方法
    def parse_image(self, response, image_name):
        # print('图片解析方法:', response.url)
        # print(image_name)
        # print(response.body)
        yield {
            "image_name": image_name + ".png",
            "image_content": response.body
        }

if __name__ == '__main__':
    cmdline.execute('scrapy crawl qingting --nolog'.split())
```

**图片保存**

在`piplines.py`中编辑代码：

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html

# useful for handling different item types with a single interface
import os
from itemadapter import ItemAdapter

class FmPipeline:
    def process_item(self, item, spider):
        # getcwd(): 用于获取当前工作目录(Current Working Directory)的路径
        download_path = os.getcwd() + '/download/'
        if not os.path.exists(download_path):
            os.mkdir(download_path)

        # 图片保存
        image_name = item.get("image_name")
        image_content = item.get("image_content")
        if image_name:
            with open(download_path + image_name, "wb") as f:
                f.write(image_content)
                print("图片保存成功: ", image_name)
```

代码编写完成后开启管道并运行爬虫脚本。

在代码运行完毕后会输出大量的`Scrapy`日志信息，我们可以输出的日志信息简化，配置如下：

```python
# 在settings.py中写入配置项
# 设置scrapy日志信息级别为warning, 忽略info信息

LOG_LEVEL = 'WARNING'
```

**保存图片的同时一并保存`FM`信息**

在`process_item`函数中，我们可以将图片保存到文件中，那么如果既想保存图片，又想保存在`parse`函数中提取到的信息应该怎么办呢？

在代码内部判断数据是图片还是信息，如果是图片就保存到图片文件，如果是信息就保存到`csv`文件或者存储到数据库中。

代码如下：

`qingting.py`

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class QingtingSpider(scrapy.Spider):
    name = 'qingting'

    # 注意点: 当前图片域名与蜻蜓FM域名不一致
    # allowed_domains = ['qingting.fm']
    allowed_domains = ['qingting.fm', 'pic.qtfm.cn']
    start_urls = ['https://m.qingting.fm/rank/']

    def parse(self, response: HtmlResponse, **kwargs):
        a_list = response.xpath("//div[@class='rank-list']/a")
        for a_temp in a_list:
            rank_number = a_temp.xpath("./div[@class='badge']//text()").extract_first()
            img_url = a_temp.xpath("./img/@src").extract_first()
            title = a_temp.xpath("./div[@class='content']/div[@class='title']/text()").extract_first()
            desc = a_temp.xpath("./div[@class='content']/div[@class='desc']/text()").extract_first()
            play_number = a_temp.xpath(".//div[@class='info-item'][1]/span/text()").extract_first()

            # 使用yield关键字将解析的数据返回给pipline
            yield {
                'type': 'info',
                'rank_number': rank_number,
                'img_url': img_url,
                'title': title,
                'desc': desc,
                'play_number': play_number
            }

            # 构造新的请求对象: 使用cb_kwargs传递形参
            yield scrapy.Request(img_url, callback=self.parse_image, cb_kwargs={"image_name": title})

    # 解析图片方法
    def parse_image(self, response, image_name):
        # print('图片解析方法:', response.url)
        # print(image_name)
        # print(response.body)
        yield {
            'type': 'image',
            "image_name": image_name + ".png",
            "image_content": response.body
        }

if __name__ == '__main__':
    cmdline.execute('scrapy crawl qingting --nolog'.split())
```

`pipelines.py`

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html

# useful for handling different item types with a single interface
import os
import pymongo
from itemadapter import ItemAdapter

class FmPipeline:
    def process_item(self, item, spider):
        # 获取返回的数据类型
        type_ = item.get('type')
        if type_ == 'image':
            # getcwd(): 用于获取当前工作目录(Current Working Directory)的路径
            download_path = os.getcwd() + '/download/'
            if not os.path.exists(download_path):
                os.mkdir(download_path)
            # 图片保存
            image_name = item.get("image_name")
            image_content = item.get("image_content")
            with open(download_path + image_name, "wb") as f:
                f.write(image_content)
                print("图片保存成功: ", image_name)
        elif type_ == 'info':
            mongo_client = pymongo.MongoClient()
            collection = mongo_client['py_spider']['qingtingFM']
            collection.insert_one(item)
            print('数据插入成功:', item.get('title'))
        else:
            print('数据类型不符合规定...')
```

> 来自: [9.6.多数据保存](https://www.yuque.com/tuling_python/spider_base/fi51wyuzq566ci8p)

### 9.7.案例 - 豆瓣爬虫

**项目构建**

命令如下：

```bash
# 项目创建
scrapy startproject douban

# 爬虫文件创建
cd douban
scrapy genspider top250 https://movie.douban.com/top250?start=0&filter=
```

`spiders`文件夹下的`top250.py`文件内容如下：

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class Top250Spider(scrapy.Spider):
    name = "top250"
    allowed_domains = ["douban.com"]
    start_urls = ["https://movie.douban.com/top250?start=0&filter="]

    def parse(self, response: HtmlResponse, **kwargs):
        pass

if __name__ == '__main__':
    cmdline.execute('scrapy crawl top250'.split())
```

**数据解析**

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class Top250Spider(scrapy.Spider):
    name = "top250"
    allowed_domains = ["douban.com"]
    start_urls = ["https://movie.douban.com/top250?start=0&filter="]

    def parse(self, response: HtmlResponse, **kwargs):
        # 查看请求头信息
        # print(response.request.headers)

        li_list = response.xpath("//ol[@class='grid_view']/li")
        for li_temp in li_list:
            image = li_temp.xpath(".//img/@src").extract_first()
            title = li_temp.xpath(".//span[@class='title'][1]/text()").extract_first()
            rating_num = li_temp.xpath(".//span[@class='rating_num']/text()").extract_first()
            people_num = li_temp.xpath(".//div[@class='star']/span[4]/text()").extract_first()

            # 信息验证
            print('--->', image, title, rating_num, people_num)

if __name__ == '__main__':
    cmdline.execute('scrapy crawl top250'.split())
```

注意点：

豆瓣网站设置了`rebots.txt`验证，需要在`settings.py`文件中关闭`rebots.txt`验证。

```python
ROBOTSTXT_OBEY = False
```

另外需要在`setting.py`中设置爬虫的`User-Agent`。

```python
# 方式一
USER_AGENT = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 " \
             "(KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36"

# 方式二
DEFAULT_REQUEST_HEADERS = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 "
                  "(KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36"
}

"""
在配置文件中设置的请求头是固定的, 如果发送的请求过多也可能造成当前请求头失效。
所以需要在请求的过程中要对请求头进行随机变换，想要完成这种功能需要借助中间件完成。
"""
```

> 来自: [9.7.案例 - 豆瓣爬虫](https://www.yuque.com/tuling_python/spider_base/glvxteeno3sgov60)

### 9.8.中间件的使用

**`scrapy`中间件的分类**

根据`scrapy`运行流程中所在位置不同分为：

- 下载中间件
- 爬虫中间件

**中间件的作用**

预处理`request`和`response`对象

- 如果响应状态码不是`200`则请求重试（重新构造`Request`对象返回给引擎）
- 可以对`header`以及`cookie`进行更换和处理
- 使用代理`ip`等

但在`Scrapy`默认的情况下，两种中间件都在`middlewares.py`一个文件中。爬虫中间件使用方法和下载中间件相同，且功能重复，常使用下载中间件。

**下载中间件的内部方法**

`Downloader Middlewares`默认的方法：

- `process_request(self, request, spider)`

- 当每个`request`通过下载中间件时，该方法被调用
- 返回`None`值：没有`return`也是返回`None`，该`request`对象传递给下载器，或通过引擎传递给其他权重低的`process_request`方法
- 返回`Response`对象：不再请求，把`response`返回给引擎
- 返回`Request`对象：把`request`对象通过引擎交给调度器，此时将不通过其他权重低的`process_request`方法

- `process_response(self, request, response, spider)`

- 当下载器完成`http`请求，传递响应给引擎的时候调用
- 返回`Resposne`对象：通过引擎交给爬虫处理或交给权重更低的其他下载中间件的`process_response`方法
- 返回`Request`对象：通过引擎交给调度器继续请求，此时将不通过其他权重低的`process_request`方法

注意：需要在`settings.py`文件中开启中间件，权重越小越优先执行。

**下载中间件代码示例 - 随机`UA`**

```python
import random

class UserAgentDownloaderMiddleware:
    USER_AGENTS_LIST = [
        "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
        "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
        "Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)",
        "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0",
        "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5"
    ]

    def process_request(self, request, spider):
        print("------下载中间件----")
        # 随机选择UA
        user_agent = random.choice(self.USER_AGENTS_LIST)
        request.headers['User-Agent'] = user_agent
        # 不写return
        """
        如果返回None, 表示当前的response提交下一个权重低的process_request。
        如果传递到最后一个process_request,则传递给下载器进行下载。
        """
```

在`settings.py`中设置中间件：

```python
DOWNLOADER_MIDDLEWARES = {
    "douban.middlewares.DoubanDownloaderMiddleware": 543,
    "douban.middlewares.UserAgentDownloaderMiddleware": 400
}
```

**豆瓣爬虫代码完善**

`top250.py`

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class Top250Spider(scrapy.Spider):
    name = "top250"

    # 图片域名与网站域名不一致
    allowed_domains = ["douban.com", "doubanio.com"]
    start_urls = ["https://movie.douban.com/top250?start=0&filter="]

    def parse(self, response: HtmlResponse, **kwargs):
        # 查看请求头信息
        # print(response.request.headers)

        li_list = response.xpath("//ol[@class='grid_view']/li")
        for li_temp in li_list:
            image_url = li_temp.xpath(".//img/@src").extract_first()
            title = li_temp.xpath(".//span[@class='title'][1]/text()").extract_first()
            rating_num = li_temp.xpath(".//span[@class='rating_num']/text()").extract_first()
            people_num = li_temp.xpath(".//div[@class='star']/span[4]/text()").extract_first()

            # 信息验证
            # print('--->', image, title, rating_num, people_num)

            yield {
                'type': 'info',
                'image': image_url,
                'title': title,
                'rating_num': rating_num,
                'people_num': people_num
            }

            # 创建新的请求对象下载图片
            yield scrapy.Request(url=image_url, callback=self.parse_image, cb_kwargs={'image_name': title})

    def parse_image(self, response, image_name):
        yield {
            'type': 'image',
            'image_name': image_name + '.jpg',
            'image_content': response.body
        }

if __name__ == '__main__':
    cmdline.execute('scrapy crawl top250'.split())
```

`pipelines.py`

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html

# useful for handling different item types with a single interface
import os
import pymongo
from itemadapter import ItemAdapter

class DoubanPipeline:
    def process_item(self, item, spider):
        type_ = item.get('type')
        if type_ == 'info':
            mongo_client = pymongo.MongoClient()
            collection = mongo_client['py_spider']['movie_info']
            collection.insert_one(item)
            print('数据插入成功:', item.get('title'))
        elif type_ == 'image':
            print(type_)
            download_path = os.getcwd() + '/download/'
            if not os.path.exists(download_path):
                os.mkdir(download_path)
            # 图片保存
            image_name = item.get("image_name")
            image_content = item.get("image_content")
            with open(download_path + image_name, "wb") as f:
                f.write(image_content)
                print("图片保存成功: ", image_name)
        else:
            print('数据类型不符合规定...')

        # return item
```

`middlewares.py`

```python
class UserAgentDownloaderMiddleware:
    USER_AGENTS_LIST = [
        "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
        "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
        "Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)",
        "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0",
        "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5"
    ]

    def process_request(self, request, spider):
        print("------下载中间件----")
        # 随机选择
        user_agent = random.choice(self.USER_AGENTS_LIST)
        request.headers['User-Agent'] = user_agent
        return None
```

中间件编写完成后记住需要在`settings.py`中启用。

**翻页操作**

使用`xpath`定位到翻页控件并获取当前控件的`href`属性，将这个属性值拼接到`start_urls`链接中，使用`response`响应对象中的`response.urljoin`方法完成地址拼接。

注意：当前案例写入的`url`地址携带查询字符串，需要将原本的查询字符串去除。

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class Top250Spider(scrapy.Spider):
    name = "top250"

    # 图片域名与网站域名不一致
    allowed_domains = ["douban.com", "doubanio.com"]
    start_urls = ["https://movie.douban.com/top250"]

    def parse(self, response: HtmlResponse, **kwargs):
        # 查看请求头信息
        # print(response.request.headers)

        li_list = response.xpath("//ol[@class='grid_view']/li")
        for li_temp in li_list:
            image_url = li_temp.xpath(".//img/@src").extract_first()
            title = li_temp.xpath(".//span[@class='title'][1]/text()").extract_first()
            rating_num = li_temp.xpath(".//span[@class='rating_num']/text()").extract_first()
            people_num = li_temp.xpath(".//div[@class='star']/span[4]/text()").extract_first()

            # 信息验证
            # print('--->', image, title, rating_num, people_num)

            yield {
                'type': 'info',
                'image': image_url,
                'title': title,
                'rating_num': rating_num,
                'people_num': people_num
            }

            # 创建新的请求对象下载图片
            yield scrapy.Request(url=image_url, callback=self.parse_image, cb_kwargs={'image_name': title})

        # 获取下一页的链接, 最后一页停止运行
        if response.xpath("//span[@class='next']/a/@href"):
            next_url = response.urljoin(response.xpath("//span[@class='next']/a/@href").extract_first())
            print('开始抓取下一页: ', next_url)
            yield scrapy.Request(url=next_url, callback=self.parse)
        else:
            print('全站抓取完成...')

    def parse_image(self, response, image_name):
        yield {
            'type': 'image',
            'image_name': image_name + '.jpg',
            'image_content': response.body
        }

if __name__ == '__main__':
    cmdline.execute('scrapy crawl top250'.split())
```

除了可以使用`url`拼接查询字符串进行翻页之外，也可以自己手动构造翻页的请求对象，在`spider`爬虫类中重写`start_requests`方法即可。

```python
class Top250Spider(scrapy.Spider):
    def start_requests(self):
        for page in range(0, 10):
            url = 'https://movie.douban.com/top250?start={}&filter='.format(page * 25)
            print('当前页数:', url)
            yield scrapy.Request(url)
```

**请求延时**

在豆瓣爬虫案例中，我们已经完成了代码编写。但是，在翻页抓取时因为`scrapy`框架的异步抓取可能会导致我们的爬虫被网站服务器封禁。所以，我们需要控制爬虫的请求频率。

在`settings.py`中设置请求频率：

```python
# 在配置文件中搜索此配置开启即可
# 当前参数不会等待固定的3秒钟，而是使用当前设置的参数乘以0.5 - 1.5之间的等待时间(1.5秒到4.5秒之间)
DOWNLOAD_DELAY = 3
```

作用：`scrapy`爬取同一个域名下的间隔时间，不是固定时间。

详情可参考：[https://www.osgeo.cn/scrapy/topics/settings.html?highlight=download_delay#std-setting-DOWNLOAD_DELAY](https://www.osgeo.cn/scrapy/topics/settings.html?highlight=download_delay#std-setting-DOWNLOAD_DELAY)

**中间件代码示例 - 设置`IP`代理**

在爬虫项目中虽然设置了请求延时，但是在某些情况下网站服务器依然会封禁我们的爬虫程序。此时就可以使用不同的`IP`地址来访问目标站点。

在`scrapy`的`Request`对象当中包含`meta`元信息，可以使用`meta`参数设置代理。

在下载中间件中设置代理

```python
# 免费代理ip
class FreeProxyDownloaderMiddleware:
    def process_request(self, request, spider):
        print('下载中间件 - 代理设置')
        # 当前设置免费代理
        request.meta['proxy'] = 'http://127.0.0.1:7890'
        return None  # 当前return可省略

    def process_response(self, request, response, spider):
        print('下载中间件 - 代理检测')
        if response.status != 200:
            request.dont_filter = True  # 关闭过滤, 并重新发送失败的请求
            return request
        return response  # 通过引擎交给爬虫处理或交给权重更低的其他下载中间件的process_response方法

# 付费代理ip
class TollProxyDownloaderMiddleware:
    """
    付费代理配置文档(快代理):
        https://www.kuaidaili.com/doc/dev/sdk_http/#proxy_python-scrapy
    """
    pass
```

代理中间件编写完成后记住在`settings.py`中开启。

**在`scrapy`中使用`selenium`**

在某些网站中的数据是通过`ajax`动态渲染的，不能直接通过`scrapy`获取到页面渲染的数据。此时就可以通过之前学习的`selenium`来获取动态数据。

接下来我们通过腾讯招聘爬虫案例来学习如何在`scrapy`框架中集成`selenium`，目标站点：[https://careers.tencent.com/search.html](https://careers.tencent.com/search.html)

项目创建

```shell
scrapy startproject TxWork
```

爬虫创建

```shell
cd TxWork
scrapy genspider tx_work_info https://careers.tencent.com/search.html
```

下载中间件：`middlewares.py`

```python
import scrapy
from scrapy import signals
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

class SeleniumDownloaderMiddleware:
    def __init__(self):
        self.browser = webdriver.Chrome()

    # 监测爬虫状态
    @classmethod
    def from_crawler(cls, crawler):
        s = cls()
        # 如果爬虫关闭则调用spider_closed方法
        crawler.signals.connect(s.spider_closed, signal=signals.spider_closed)
        return s

    def process_request(self, request, spider):
        self.browser.get(request.url)
        wait = WebDriverWait(self.browser, 10)
        wait.until(EC.presence_of_element_located(
            (By.CLASS_NAME, 'recruit-list')
        ))

        # 获取页面信息
        body = self.browser.page_source
        return scrapy.http.HtmlResponse(url=response.url, body=body, request=request, encoding='utf-8')

    def spider_closed(self):
        # self.browser.quit()
        self.browser.close()
```

爬虫文件：`tx_work_info.py`

在爬虫文件代码中，我们需要手动构造请求地址完成翻页功能，需要重写`spider`类中的`start_requests`方法。

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class TxWorkInfoSpider(scrapy.Spider):
    name = "tx_work_info"
    allowed_domains = ["careers.tencent.com"]
    # start_urls = ["https://careers.tencent.com/search.html"]

    # 手动构建请求地址
    def start_requests(self):
        url = 'https://careers.tencent.com/search.html?index={}&keyword=python'
        for page in range(1, 6):
            yield scrapy.Request(url=url.format(page))

    def parse(self, response: HtmlResponse, **kwargs):
        div_list = response.xpath("//div[@class='correlation-degree']/div/div")
        for div in div_list:
            item = dict()
            item['title'] = div.xpath('./a//span[@class="job-recruit-title"]/text()').extract_first()
            item['department'] = div.xpath('./a/p[1]/span[1]/text()').extract_first()
            item['address'] = div.xpath('./a//span[2]/text()').extract_first()
            item['post'] = div.xpath('./a/p[1]/span[3]/text()').extract_first()
            item['date'] = div.xpath('./a/p[1]/span[last()]/text()').extract_first()
            item['recruit_data'] = div.xpath('./a/p[2]/text()').extract_first()
            yield item

        # 当前方法无法对python岗位页面翻页，原因是首页岗位页数与python岗位页数不一致
        # if response.xpath("//li[last()-1]/span/text()"):
        #     page_num = response.xpath("//li[last()-1]/span/text()").extract_first()
        #     page_num = int(page_num) + 1
        #     for page in range(1, page_num):
        #         next_url = response.urljoin(f"?index={page}&keyword=python")
        #         print(next_url)
        #         yield scrapy.Request(url=next_url, callback=self.parse)

if __name__ == "__main__":
    cmdline.execute('scrapy crawl tx_work_info'.split())
```

管道文件：`pipelines.py`

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html

# useful for handling different item types with a single interface
import pymongo
from itemadapter import ItemAdapter

class TxworkPipeline:
    def process_item(self, item, spider):
        mongo_client = pymongo.MongoClient()
        collection = mongo_client['py_spider']['tx_work_info']
        collection.insert_one(item)
        print('数据插入成功: ', item.get('title'))
        # return item
```

`settings.py`配置

```python
ROBOTSTXT_OBEY = False

DOWNLOADER_MIDDLEWARES = {
    # "TxWork.middlewares.TxworkDownloaderMiddleware": 543,
    "TxWork.middlewares.SeleniumDownloaderMiddleware": 543,
}

ITEM_PIPELINES = {
    "TxWork.pipelines.TxworkPipeline": 300,
}
```

> 来自: [9.8.中间件的使用](https://www.yuque.com/tuling_python/spider_base/ntqpbx17dukmmyrx)

### 9.9.管道的详细使用

在前面学习`scrapy`时，我们用过管道，它其实就是一个类，这个类中有`process_item`方法，在这个方法中，可以实现将数据存储到`MongoDB`中。

但问题来了：如果有一个`scrapy`爬虫项目，它需要在存储数据之前，先进行清洗数据（所谓清洗就是去除不符合要求的数据），然后再存储数据。此时应该怎么办呢？

答：可以创建多个管道。

**自定义管道**

如果我们需要自定义管道`pipeline`，那么就要注意在管道类中可以编写的方法如下：

`process_item(self, item, spider)`

- 管道类中必须要有的方法
- 实现对`item`数据的处理
- 一般情况下都会`return item`，如果没有`return`，那么就相当于将`None`传递给权重低的`process_item`

`open_spider(self, spider)`

- 在爬虫开启的时候仅执行一次
- 可以在该方法中链接数据库、打开文件等等

`close_spider(self, spider)`

- 在爬虫关闭的时候仅执行一次
- 可以在该方法中关闭数据库连接、关闭文件对象等

**代码示例**

我们以之前的腾讯招聘爬虫为例，在当前项目中修改`piplines.py`文件：

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html

# useful for handling different item types with a single interface
import json
import pymongo
from itemadapter import ItemAdapter

class TxWorkFilePipeline:
    def open_spider(self, spider):
        if spider.name == 'tx_work_info':
            self.file_obj = open('json.txt', 'a', encoding='utf-8')

    def process_item(self, item, spider):
        """
        在当前方法中可以对item进行数据判断，如果不符合数据要求，一般有两种方式来处理:
            1. 扔掉
                如果数据不符合特定的条件或者质量标准，你可以直接从管道中排除它。
                这可以通过在管道的process_item方法中简单地返回None或抛出DropItem异常
                来实现。

            2. 修复
                在当前方法中编辑修复代码逻辑并使用return将修复的数据传递给下一个item

            注意:
                当前方法如果存在return item则将item数据传递给下一个item
                如果return不存在则将None传递给下一个item
        """
        if spider.name == 'tx_work_info':
            self.file_obj.write(json.dumps(item, ensure_ascii=False, indent=4) + ',\n')
        return item

    def close_spider(self, spider):
        if spider.name == 'tx_work_info':
            self.file_obj.close()

class TxWorkMongoPipeline:
    def open_spider(self, spider):
        if spider.name == 'tx_work_info':
            self.mongo_client = pymongo.MongoClient()
            self.collection = self.mongo_client['py_spider']['tx_work_info']

    def process_item(self, item, spider):
        if spider.name == 'tx_work_info':
            self.collection.insert_one(item)
            print('数据插入成功: ', item.get('title'))
            return item

    def close_spider(self, spider):
        if spider.name == 'tx_work_info':
            self.mongo_client.close()
```

将修改完成后的`pipline`添加到`settings.py`配置文件中：

```python
ITEM_PIPELINES = {
    "TxWork.pipelines.TxWorkFilePipeline": 300,
    "TxWork.pipelines.TxWorkMongoPipeline": 301,
}
```

**思考**

在`settings.py`中能够开启多个管道，为什么需要开启多个？

- 不同的`pipeline`可以处理不同爬虫的数据，通过`spider.name`属性来区分
- 不同的`pipeline`能够对一个或多个爬虫进行不同的数据处理的操作，比如一个进行数据清洗，一个进行数据的保存
- 同一个管道类也可以处理不同爬虫的数据，通过`spider.name`属性来区分

**总结**

- 使用之前需要在`settings.py`中开启
- 多个管道在项目中的位置可以自定义，值表示距离引擎的远近，越近数据会越先经过：权重值小的优先执行
- 有多个`pipeline`的时候，`process_item`方法应该`return item`,否则后一个`pipeline`取到的数据为`None`值
- `pipeline`中`process_item`的方法必须有，否则`item`没有办法接收和处理
- `process_item`方法接受`item`和`spider`，其中`spider`表示当前传递`item`过来的`spider`引用
- `open_spider(spider)`：能够在爬虫开启的时候执行一次
- `close_spider(spider)`：能够在爬虫关闭的时候执行一次
- 上述俩个方法经常用于爬虫和数据库的交互，在爬虫开启的时候建立数据库的连接，在爬虫关闭的时候断开数据库的连接

> 来自: [9.9.管道的详细使用](https://www.yuque.com/tuling_python/spider_base/gpqohv4mqmbeprrg)

### 9.10.数据去重

在实际爬取某网站时，可能由于某些原因导致爬虫意外结束，当开发人员修复之后，需要在之前爬取的基础上继续爬取，此时就需要进行过滤掉已爬取的`URL`或者数据，完成数据去重操作。

- 可以判断`URL`是否爬取过
- 可以判断数据是否存储过

**对数据进行去重**

```python
import json
import redis
import hashlib
from scrapy.exceptions import DropItem

class TxWorkCheckPipeline:
    """
    使用redis进行数据去重
    """

    def open_spider(self, spider):
        if spider.name == 'tx_work_info':
            self.redis_client = redis.Redis()

    def process_item(self, item, spider):
        if spider.name == 'tx_work_info':
            # 将传递过来的item数据转为字符串并加密成md5数据
            item_str = json.dumps(item)
            md5_hash = hashlib.md5()
            md5_hash.update(item_str.encode())
            hash_value = md5_hash.hexdigest()

            # 判断hash值是否存在于redis中
            if self.redis_client.get(f'tx_work_item_filter:{hash_value}'):
                # 如果存在则抛出异常停止管道传递数据
                raise DropItem('数据已存在...')
            else:
                # 如果不存在则将hash保存到redis中
                # tx_work_filter:前缀会在redis中创建文件夹, 便于管理
                self.redis_client.set(f'tx_work_item_filter:{hash_value}', item_str)
            return item

    def close_spider(self, spider):
        if spider.name == 'tx_work_info':
            self.redis_client.close()
```

以上代码已经实现了对数据的去重，但是在项目启动时对之前已经访问过的`URL`地址还是会重复访问。所以接下来可以对之前访问过的`URL`进行去重。

**对地址进行去重**

```python
import redis
import scrapy
import hashlib
from scrapy import cmdline
from scrapy.http import HtmlResponse

class TxWorkInfoSpider(scrapy.Spider):
    name = "tx_work_info"
    allowed_domains = ["careers.tencent.com"]

    # start_urls = ["https://careers.tencent.com/search.html"]
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.redis_client = redis.Redis()

    # 当程序退出时关闭redis连接
    def __del__(self):
        self.redis_client.close()

    # 手动构建请求地址
    def start_requests(self):
        url = 'https://careers.tencent.com/search.html?index={}&keyword=python'
        for page in range(1, 6):
            md5_hash = hashlib.md5()
            md5_hash.update(url.format(page).encode())
            hash_value = md5_hash.hexdigest()

            if self.redis_client.get(f'tx_work_url_filter:{hash_value}'):
                print('url重复...')
                continue
            else:
                self.redis_client.set(f'tx_work_url_filter:{hash_value}', url.format(page))
                yield scrapy.Request(url=url.format(page))

    def parse(self, response: HtmlResponse, **kwargs):
        div_list = response.xpath("//div[@class='correlation-degree']/div/div")
        for div in div_list:
            item = dict()
            item['title'] = div.xpath('./a//span[@class="job-recruit-title"]/text()').extract_first()
            item['department'] = div.xpath('./a/p[1]/span[1]/text()').extract_first()
            item['address'] = div.xpath('./a//span[2]/text()').extract_first()
            item['post'] = div.xpath('./a/p[1]/span[3]/text()').extract_first()
            item['date'] = div.xpath('./a/p[1]/span[last()]/text()').extract_first()
            item['recruit_data'] = div.xpath('./a/p[2]/text()').extract_first()
            yield item

if __name__ == "__main__":
    cmdline.execute('scrapy crawl tx_work_info'.split())
```

> 来自: [9.10.数据去重](https://www.yuque.com/tuling_python/spider_base/sl641ipd9lfv5bg3)

### 9.11.增量爬虫

有些情况下，我们希望能暂停爬虫，之后在恢复运行，尤其是抓取大型站点的时候可以完成暂停与恢复。此时就用到了`scrapy`的爬虫暂停与爬虫恢复。

**暂停爬虫的命令**

想要实现暂停，`scrapy`代码不用修改，只需要在启动时修改运行命令即可：

```shell
# scrapy crawl 爬虫名称 -s JOBDIR=缓存scrapy信息的路径

scrapy crawl MySpider -s JOBDIR=crawls/my_spider-1
```

**暂停爬虫的快捷键**

在终端启动爬虫之后，只需要按下`ctrl + c`就可以让爬虫暂停

注意点：`ctrl + c`不能执行两次，只需一次即可

**恢复爬虫的命令**

与暂停爬虫的指令类似，恢复爬虫时运行相同的命令：

```python
# scrapy crawl 爬虫名称 -s JOBDIR=缓存scrapy信息的路径

scrapy crawl MySpider -s JOBDIR=crawls/my_spider-1
```

> 来自: [9.11.增量爬虫](https://www.yuque.com/tuling_python/spider_base/gy04sokv1wb5no1d)

### 9.12.dont_filter参数与start_requests方法

**`dont_filter`参数**

当我们在使用`scrapy`生成一个新的`Request`请求对象时，需要根据业务场景判断是否请求重复的`Request`对象，如果不需要重复请求则通过`dont_filter`进行过滤。

`scrapy.Request`初始化方法部分源码：

```python
def __init__(
        self,
        url: str,
        callback: Optional[Callable] = None,
        method: str = "GET",
        headers: Optional[dict] = None,
        body: Optional[Union[bytes, str]] = None,
        cookies: Optional[Union[dict, List[dict]]] = None,
        meta: Optional[dict] = None,
        encoding: str = "utf-8",
        priority: int = 0,
        dont_filter: bool = False,
        errback: Optional[Callable] = None,
        flags: Optional[List[str]] = None,
        cb_kwargs: Optional[dict] = None,
    ) -> None:
    ...
```

通过以上代码我们得知`dont_filter`参数的默认值为`False`，即默认开启重复请求过滤。如果需要对重复的`Request`对象发起请求则设置`dont_filter`参数值为`True`。

**`start_requests`方法**

`start_requests`是`scrapy.Spider`父类中的方法，在没有重写的情况下，`scrapy`提取`start_urls`列表中的地址并构建请求对象。

但是如果在重写的情况下，则调用重写后的代码而不经过`start_urls`，只要保证这个方法的返回值可以迭代即可。

如何确定在什么场景下需要重写`start_requests`方法？

- 如果`start_urls`列表中的地址需要登录后才能访问，则需要重写`start_requests`方法并手动添加`cookie`
- 需要自己构建翻页地址的情况下可以重写`start_requests`方法
- 如果在`start_urls`中的`URL`需要用`post`提交的话，则需要在`start_requests`方法中修改
- 默认情况下`start_urls`中的`URL`在被生成`Request`对象时，都是设置为不过滤，即`dont_filter=True`，所以如果想使用暂停、恢复爬取功能的话，就需要重写此方法了。

**再次理解豆瓣爬虫代码**

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class Top250Spider(scrapy.Spider):
    name = "top250"
    allowed_domains = ["douban.com", "doubanio.com"]

    """
    start_urls中的地址默认是不过滤的
    	所以需要对列表中的地址过滤则重写start_requests

    	详情可查看父类中的start_requests方法
    """
    start_urls = ["https://movie.douban.com/top250"]

    def parse(self, response: HtmlResponse, **kwargs):
        li_list = response.xpath("//ol[@class='grid_view']/li")
        for li_temp in li_list:
            image_url = li_temp.xpath(".//img/@src").extract_first()
            title = li_temp.xpath(".//span[@class='title'][1]/text()").extract_first()
            rating_num = li_temp.xpath(".//span[@class='rating_num']/text()").extract_first()
            people_num = li_temp.xpath(".//div[@class='star']/span[4]/text()").extract_first()

            yield {
                'type': 'info',
                'image': image_url,
                'title': title,
                'rating_num': rating_num,
                'people_num': people_num
            }

            # 创建新的请求对象下载图片
            # 自己生成的新的Request对象Scrapy默认是过滤的
            yield scrapy.Request(url=image_url, callback=self.parse_image, cb_kwargs={'image_name': title})

        if response.xpath("//span[@class='next']/a/@href"):
            next_url = response.urljoin(response.xpath("//span[@class='next']/a/@href").extract_first())
            print('开始抓取下一页: ', next_url)
            yield scrapy.Request(url=next_url, callback=self.parse)
        else:
            print('全站抓取完成...')

    def parse_image(self, response, image_name):
        yield {
            'type': 'image',
            'image_name': image_name + '.jpg',
            'image_content': response.body
        }

"""
if __name__ == '__main__':
    cmdline.execute('scrapy crawl top250 -s JOBDIR=crawls/my_spider-1'.split())
"""
```

**修改豆瓣爬虫代码让其支持地址过滤**

重写`start_requests`方法即可

```python
import scrapy
from scrapy import cmdline
from scrapy.http import HtmlResponse

class Top250Spider(scrapy.Spider):
    name = "top250"
    allowed_domains = ["douban.com", "doubanio.com"]
    start_urls = ["https://movie.douban.com/top250"]

    def start_requests(self):
        for url in self.start_urls:
            # 重新构造请求对象, dont_filter=False可不写
            yield scrapy.Request(url=url, callback=self.parse, dont_filter=False)

    def parse(self, response: HtmlResponse, **kwargs):
        li_list = response.xpath("//ol[@class='grid_view']/li")
        for li_temp in li_list:
            image_url = li_temp.xpath(".//img/@src").extract_first()
            title = li_temp.xpath(".//span[@class='title'][1]/text()").extract_first()
            rating_num = li_temp.xpath(".//span[@class='rating_num']/text()").extract_first()
            people_num = li_temp.xpath(".//div[@class='star']/span[4]/text()").extract_first()

            yield {
                'type': 'info',
                'image': image_url,
                'title': title,
                'rating_num': rating_num,
                'people_num': people_num
            }

            yield scrapy.Request(url=image_url, callback=self.parse_image, cb_kwargs={'image_name': title})

        if response.xpath("//span[@class='next']/a/@href"):
            next_url = response.urljoin(response.xpath("//span[@class='next']/a/@href").extract_first())
            print('开始抓取下一页: ', next_url)
            yield scrapy.Request(url=next_url, callback=self.parse)
        else:
            print('全站抓取完成...')

    def parse_image(self, response, image_name):
        yield {
            'type': 'image',
            'image_name': image_name + '.jpg',
            'image_content': response.body
        }

"""
if __name__ == '__main__':
    cmdline.execute('scrapy crawl top250 -s JOBDIR=crawls/my_spider-1'.split())
"""
```

> 来自: [9.12.dont_filter参数与start_requests方法](https://www.yuque.com/tuling_python/spider_base/dgn2ny90h3ugivrv)

### 9.13.发送post请求

在之前的学习当中我们经常会使用`scrapy`发送`get`请求，那么如果一些网站接收的请求类型为`post`应该怎么处理？

接下来我们以巨潮资讯网为例，讲解如何发送`post`请求。

目标站点：[http://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse](http://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse)

**代码示例 - `form`表单**

爬虫文件

```python
# import json
import scrapy
from scrapy import cmdline
from HcInfo.items import HcInfoItem

class HcInfoDataSpider(scrapy.Spider):
    name = "HcInfoData"

    def start_requests(self):
        url = 'http://www.cninfo.com.cn/new/disclosure'

        # 表单数据
        for page in range(1, 16):
            data = {
                'column': 'szse_latest',
                'pageNum': str(page),
                'pageSize': '30',
                'sortName': '',
                'sortType': '',
                'clusterFlag': 'true',
            }

            yield scrapy.FormRequest(url=url, formdata=data, dont_filter=False)

    def parse(self, response, **kwargs):
        for info_list in response.json()['classifiedAnnouncements']:
            for info in info_list:
                item = HcInfoItem()
                item['announcementTitle'] = info['announcementTitle']
                item['announcementTypeName'] = info['announcementTypeName']
                item['batchNum'] = info['batchNum']
                item['secName'] = info['secName']
                item['adjunctType'] = info['adjunctType']
                yield item

if __name__ == '__main__':
    cmdline.execute('scrapy crawl HcInfoData'.split())
```

`items.py`文件

首先需要了解`items.py`文件可以定义抓取的数据结构，在校验完数据结构之后则可以递交给管道进行数据存储。总之，`items.py`主要功能是检查抓取的数据是否符合自己定义的数据结构。

```python
# Define here the models for your scraped items
#
# See documentation in:
# https://docs.scrapy.org/en/latest/topics/items.html

import scrapy

class HcInfoItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    announcementTitle = scrapy.Field()
    announcementTypeName = scrapy.Field()
    batchNum = scrapy.Field()
    secName = scrapy.Field()
    adjunctType = scrapy.Field()
```

管道文件

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html

# useful for handling different item types with a single interface
import pymongo
from itemadapter import ItemAdapter

class HcInfoPipeline:
    def process_item(self, item, spider):
        return item

class MongoPipeline:
    def open_spider(self, spider):
        if spider.name == 'HcInfoData':
            self.mongo_client = pymongo.MongoClient()
            self.collection = self.mongo_client['py_spider']['jc_info']

    def process_item(self, item, spider):
        if spider.name == 'HcInfoData':
            self.collection.insert_one(dict(item))
        return item

    def close_spider(self, spider):
        if spider.name == 'HcInfoData':
            self.mongo_client.close()
```

**代码示例 - `json`载荷**

在一些网站中的`post`请求数据并不是表单数据而是`payload`数据，需要传递一个字典。那么如果遇到这种情况则可以使用`scrapy`框架给我们提供的`JsonRequest`对象。

目标站点：[https://hr.163.com/job-list.html?workType=0](https://hr.163.com/job-list.html?workType=0)

```python
import json
import scrapy
from scrapy import cmdline
from scrapy.http import JsonRequest

class JobInfoSpider(scrapy.Spider):
    name = "job_info"

    # allowed_domains = ["hr.163.com"]
    # start_urls = ["http://hr.163.com/"]

    def start_requests(self):
        url = 'https://hr.163.com/api/hr163/position/queryPage'
        payload = {
            'currentPage': 1,
            'pageSize': 10,
            'workType': "0"
        }

        # headers = {
        #     'Content-Type': 'application/json',
        # }
        #
        # yield scrapy.Request(url, method='POST', body=json.dumps(payload), headers=headers)
        yield JsonRequest(url=url, data=payload)

    def parse(self, response, **kwargs):
        print(response.json())

if __name__ == '__main__':
    cmdline.execute('scrapy crawl job_info'.split())
```

> 来自: [9.13.发送post请求](https://www.yuque.com/tuling_python/spider_base/skgugrynuirgg0am)

## 10.scrapy框架 - 分布式爬虫以及爬虫部署

**什么是分布式爬虫**

分布式爬虫是网络爬虫的一种，它将任务分散在多台计算机上，这些计算机协同工作以更高效地收集网络数据。与传统的单机爬虫相比，分布式爬虫由多个节点组成，每个节点都可以执行爬虫任务，而且这些节点之间相互协作，共享资源和信息。

**学习分布式爬虫的原因**

- 高效性能：分布式爬虫可以充分利用多台计算机的计算资源和网络带宽，同时执行多个爬虫任务，从而大大提高数据抓取的效率。它能够快速地处理大规模的数据，并在较短的时间内完成爬取任务。
- 高拓展性：分布式爬虫系统可以根据需求进行横向扩展，通过增加更多的爬虫节点来处理更大规模的数据抓取任务。这使得系统能够适应不断增长的数据量和更高的并发需求。
- 高可靠性：分布式爬虫系统具有容错和冗余的特性。当某个节点出现故障或者网络问题时，其他节点可以继续执行任务，从而保证数据抓取的连续性和可靠性。
- 数据一致性：分布式爬虫可以通过合理的任务调度和数据同步机制，确保多个节点爬取的数据保持一致性。这对于需要对多个数据源进行聚合和分析应用非常重要。
- 大规模数据处理：分布式爬虫系统可以方便地应对大规模数据的处理和存储需求。通过将数据分布在多个节点上，系统可以更高效地处理和存储大量数据。

**爬虫部署的定义**

爬虫部署（`Spider Deployment`）是指将一个网络爬虫发布到一个运行环境中的过程，以便它可以自动执行其抓取任务。

爬虫部署可能会涉及以下步骤：

- 代码测试：在本地或开发环境中对爬虫进行测试，确保其按预期工作，没有错误或问题。
- 选择运行环境：选择合适的服务器或云服务来部署爬虫程序。考虑因素可能包含成本、性能、地理位置、可用性等。
- 配置环境：配置运行环境，例如安装必要的软件依赖、设置网络权限、配置数据库等。
- 安全措施：实施安全措施，如设置防火墙规则、使用`SSH`密钥等，确保爬虫在安全环境中运行。
- 持续集成/持续部署（`CI/CD`）：对于需要频繁更新或维护的爬虫程序，可以设置自动化的`CI/CD`管道，以便代码更新后可以自动测试和部署到成产环境中。
- 资源监控和管理：监控爬虫的性能和资源消耗情况，如`cpu`、内存用量、网络带宽等，以便及时优化和解决问题。
- 日志记录和错误处理：设置日志记录系统以记录爬虫的活动和可能的错误，便于调试和问题追踪。
- 任务调度：配置任务调度系统，比如使用`cron`设置定时任务或者使用更复杂的任务队列和调度器来控制爬虫的运行频率和时间。

在分布式环境中，每一台计算机就是一个节点，我们可以利用批量部署的方式将爬虫程序分批次部署到不同的节点上以提高抓取速度。

> 来自: [10.scrapy框架 - 分布式爬虫以及爬虫部署](https://www.yuque.com/tuling_python/spider_base/pswwr2gg71igv6ap)

### 10.1.scrapy-redis实现增量爬虫

**增量爬虫的含义**

在前面我们学习了可以在运行`scrapy`项目时，使用`scrpay crawl 爬虫名 -s JOBDIR=xxx`来实现暂停、恢复爬取（其实就是增量爬虫，再次运行时可以在之前基础上继续爬），这种方式虽然能够实现增量爬取，但我们无法从`JOBDIR`指定的路径中看出我们现在爬取的情况。

**使用`scrapy-redis`完成增量爬虫**

安装

```shell
# 使用scrapy-redis之前最好将scrapy版本保持在2.6.3, 2.11.0版本有兼容性问题
pip install scrapy==2.6.3 scrapy-redis
```

配置

想要让`scrapy`实现增量爬取（即暂停、恢复）功能，就需要在`scrapy`项目中的`settings.py`文件中进行配置

```python
""" scrapy-redis配置 """
# 调度器类
SCHEDULER = "scrapy_redis.scheduler.Scheduler"

# 指纹去重类
 """
 指纹是指使用哈希值标识一个请求对象
 确保每个对象都是唯一的
 """
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
# 可以替换成布隆过滤器
# 下载 - pip install scrapy-redis-bloomfilter
# from scrapy_redis_bloomfilter.dupefilter import RFPDupeFilter
# DUPEFILTER_CLASS = 'scrapy_redis_bloomfilter.dupefilter.RFPDupeFilter'

# 是否在关闭时候保留原来的调度器和去重记录，True=保留，False=清空
SCHEDULER_PERSIST = True

# Redis服务器地址
REDIS_URL = "redis://127.0.0.1:6379/0"  # Redis默认有16库，/1的意思是使用序号为2的库，默认是0号库（这个可以任意）

SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.PriorityQueue'  # 优先级队列, 使用有序集合来存储
# SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.FifoQueue'  # 先进先出
# SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.LifoQueue'  # 先进后出、后进先出

# 配置redis管道
# from scrapy_redis.pipelines import RedisPipeline
ITEM_PIPELINES = {
    "douban.pipelines.DoubanPipeline": 300,

    # 可以将获取到的数据存储在redis中, 如果不期望将数据存储在redis则注释以下中间件
    # 'scrapy_redis.pipelines.RedisPipeline': 301
}

# 重爬: 一般不配置，在分布式中使用重爬机制会导致数据混乱，默认是False
# SCHEDULER_FLUSH_ON_START = True
```

运行指令以及运行效果

```shell
scrapy crawl 爬虫名称
```

在爬取过程中，使用`ctrl + c`让爬虫暂停抓取。停止后使用`redis`客户端查看对应的数据信息。

- 在数据库中显示的`requests`表示还没有来得及抓取的请求，下次抓取时根据当前数据继续运行。

![](images/image-043.png)

- 数据库中显示的`dupfilter`表示已经抓取过的请求，重新启动后如果存在相同的`requests`则不再抓取。

![](images/image-044.png)

**爬虫作业**

访问网易招聘站点获取列出的所有招聘信息，网站地址：[https://hr.163.com/job-list.html](https://hr.163.com/job-list.html)

示例代码

`spiders/zhaopin.py`

```python
import scrapy
from scrapy import cmdline
from scrapy.http import JsonRequest

class ZhaoPinSpider(scrapy.Spider):
    name = "zhaopin"
    allowed_domains = ["hr.163.com"]

    # start_urls = ["https://hr.163.com/api/hr163/position/queryPage"]

    def start_requests(self):
        api_url = 'https://hr.163.com/api/hr163/position/queryPage'
        for page in range(1, 279):
            params = {
                'currentPage': page,
                'pageSize': 10
            }
            yield JsonRequest(api_url, data=params)

    def parse(self, response, **kwargs):
        work_list = response.json()['data']['list']
        for work in work_list:
            item = dict()
            item['work_id'] = work['id']
            item['postTypeFullName'] = work['postTypeFullName']
            item['reqEducationName'] = work['reqEducationName']
            item['workPlaceNameList'] = work['workPlaceNameList'][0]
            item['requirement'] = work['requirement']

            yield item

if __name__ == '__main__':
    cmdline.execute('scrapy crawl zhaopin'.split())
```

`pipelines.py`

```python
import pymongo
from itemadapter import ItemAdapter

class NetEasePipeline:
    def open_spider(self, spider):
        self.mongo_client = pymongo.MongoClient()
        self.collection = self.mongo_client['py_spider']['netEase_job']

    def process_item(self, item, spider):
        self.collection.insert_one(item)
        print('保存成功:', item)

    def close_spider(self, spider):
        self.mongo_client.close()
```

`settings.py`

```python
BOT_NAME = "netease"

SPIDER_MODULES = ["netease.spiders"]
NEWSPIDER_MODULE = "netease.spiders"

ROBOTSTXT_OBEY = False

DOWNLOAD_DELAY = 5

ITEM_PIPELINES = {
    "netease.pipelines.NetEasePipeline": 300,
}

REQUEST_FINGERPRINTER_IMPLEMENTATION = "2.7"
TWISTED_REACTOR = "twisted.internet.asyncioreactor.AsyncioSelectorReactor"
FEED_EXPORT_ENCODING = "utf-8"

"""scrapy-redis配置"""
# 调度器类
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
# 指纹去重类
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
# 是否在关闭时候保留原来的调度器和去重记录，True=保留，False=清空
SCHEDULER_PERSIST = True
# Redis服务器地址
REDIS_URL = "redis://127.0.0.1:6379/1"  # Redis默认有16库，/1的意思是使用序号为2的库，默认是0号库（这个可以任意）
```

> 来自: [10.1.scrapy-redis实现增量爬虫](https://www.yuque.com/tuling_python/spider_base/nilw7xakkikag45p)

### 10.2.scrapy-redis实现分布式爬虫

**`scrapy-redis`的概念**

之前我们已经学习了`scrapy`，它是一个通用的爬虫框架，能够耗费很少的时间就能够写出爬虫代码。`scrapy-redis`是`scrapy`的一个组件，它使用了`redis`数据库做为基础，目的为了更方便地让`scrapy`实现分布式爬取。

`scrapy`能做的事情很多，但是要做到大规模的分布式应用则捉襟见肘。有人改变了`scrapy`的队列调度，将起始的网址从`start_urls`里分离出来，改为从`redis`读取，多个客户端可以同时读取同一个`redis`，从而实现了分布式的爬虫。

**`scrapy-redis`的作用**

`scrapy-redis`在`scrapy`的基础上实现了更多更强大的功能，具体体现在：

通过持久化请求队列和请求的指纹集合来实现：

- 断点续爬
通俗的说：这次爬取的数据，下载再运行时不会爬取，只爬取之前没有爬过的数据
- 分布式快速抓取
通俗的说：多个电脑（也可以在一个电脑上运行多个程序来模拟）可以一起爬取数据，而且不会有冲突

**`scrapy-redis`的工作流程**

- `scrapy`的流程（与之前所讲的`scrapy`图解一样，只是表现形式不一样）

![](images/image-045.png)

- `scrapy-redis`实现分布式图解

![](images/image-046.png)

说明：原本只有1个`scrapy`时，它所有的请求对象`request`都直接存放到了内存中，此时可以完成本台电脑上`scrapy`的功能，但是其他电脑上的`scrapy`是不能够获取另外一台电脑内存中的数据的，所以借助了`Redis`数据库。将原本直接存储到内存中的数据（像请求对象等）放到了`Redis`数据库中（因为`Redis`效率非常高，所以用它而不用`MySQL`、`MongoDB`），又因为`Redis`是支持网络访问的，所以在本电脑上的`Redis`中存储的数据，就可以让其他电脑上的`scrapy`去共用，此时哪个请求对象已经处理过，哪个没有并处理过，一目了然。

**注意点**

- 在`scrapy-redis`中，所有的待抓取的`request`对象和去重的`request`对象指纹都存在共用的`redis`中
- 所有的服务器中的`scrapy`进程共用同一个`redis`中的`request`对象的队列
- 所有的`request`对象存入`redis`前，都会通过该`redis`中的`request`指纹集合进行判断，之前是否已经存入过
- 在默认情况下所有的数据会保存在`redis`中

![](images/image-047.png)

**`scrapy-redis`分布式架构的思路分析**

假设有四台电脑：`Windows 10`、`Mac OS X`、`Ubuntu 16.04`、`CentOS 7.2`任意一台电脑都可以作为`Master`端或`Slaver`端。

![](images/image-048.png)

- `Master端`(核心服务器) ：使用`Windows 10`，搭建一个`Redis`数据库，不负责爬取，只负责`url`指纹判重、`Request`的分配，以及数据的存储
- `Slaver端`(爬虫程序执行端) ：使用`Mac OS X`、`Ubuntu 16.04`、`CentOS 7.2`负责执行爬虫程序，运行过程中提交新的`Request`给`Master`

执行流程

- 首先`Slaver`端从`Master`端拿任务（`Request`、`url`）进行数据抓取，`Slaver`抓取数据的同时，产生新任务的`Request`便提交给`Master`处理；
- `Master`端只有一个`Redis`数据库，负责将未处理的`Request`去重和任务分配，将处理后的`Request`加入待爬队列，并且存储爬取的数据。

`scrapy-redis`默认使用的就是这种策略，我们实现起来很简单，因为任务调度等工作`scrapy-redis`都已经帮我们做好了，我们只需要继承`RedisSpider`、指定`redis_key`就行了。

缺点：`scrapy-redis`调度的任务是`Request`对象，里面信息量比较大（不仅包含`url`，还有`callback`函数、`headers`等信息），可能导致的结果就是会降低爬虫速度、而且会占用`Redis`大量的存储空间，所以如果要保证效率，那么就需要一定硬件水平。

**豆瓣爬虫案例实现**

`top250.py`

```python
import base64
import scrapy
from scrapy import cmdline
from scrapy_redis.spiders import RedisSpider
from scrapy.http import HtmlResponse

class Top250Spider(RedisSpider):
    name = "top250"
    allowed_domains = ["movie.douban.com", 'doubanio.com']
    # start_urls = ["https://movie.douban.com/top250"]
    redis_key = 'top250:start_urls'

    def parse(self, response: HtmlResponse, **kwargs):
        li_list = response.xpath("//ol[@class='grid_view']/li")
        for li_temp in li_list:
            image_url = li_temp.xpath(".//img/@src").extract_first()
            title = li_temp.xpath(".//span[@class='title'][1]/text()").extract_first()
            rating_num = li_temp.xpath(".//span[@class='rating_num']/text()").extract_first()
            people_num = li_temp.xpath(".//div[@class='star']/span[4]/text()").extract_first()

            yield {
                'type': 'info',
                'image_url': image_url,
                'title': title,
                'rating_num': rating_num,
                'people_num': people_num
            }

            yield scrapy.Request(url=image_url, callback=self.parse_image, cb_kwargs={'image_name': title})

        if response.xpath("//span[@class='next']/a/@href"):
            next_url = response.urljoin(response.xpath("//span[@class='next']/a/@href").extract_first())
            print('开始抓取下一页:', next_url)
            yield scrapy.Request(url=next_url, callback=self.parse)
        else:
            print('全站抓取成功...')

    def parse_image(self, response, image_name):
        base64_data = base64.b64encode(response.body)
        str_data = base64_data.decode()
        yield {
            'type': 'image',
            'image_name': image_name + '.jpg',
            'image_content': str_data
        }

# if __name__ == '__main__':
#     cmdline.execute('scrapy crawl top250'.split())
```

`pipelines.py`

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html

# useful for handling different item types with a single interface
import os
import base64
import pymongo
from itemadapter import ItemAdapter

class DoubanPipeline:
    def process_item(self, item, spider):
        type_ = item.get('type')

        if type_ == 'info':
            mongo_client = pymongo.MongoClient()
            collection = mongo_client['py_spider']['douban_movie_info']
            collection.insert_one(item)
            print('数据保存成功:', item)
        elif type_ == 'image':
            download_path = os.getcwd() + '/download/'
            if not os.path.exists(download_path):
                os.mkdir(download_path)

            image_name = item.get('image_name')
            image_content = item.get('image_content')

            # 为了保证二进制数据可以存储在redis中对图片数据进行了编码
            # 图片写入本地之前需要将图片数据还原
            image_content = base64.b64decode(image_content.encode())
            with open(download_path + image_name, 'wb') as f:
                f.write(image_content)
                print('图片下载成功:', image_name)
        else:
            print('数据类型不符合规定...')

        return item
```

`settings.py`

```python
BOT_NAME = "douban"

SPIDER_MODULES = ["douban.spiders"]
NEWSPIDER_MODULE = "douban.spiders"

ROBOTSTXT_OBEY = False

# 控制抓取速率
DOWNLOAD_DELAY = 3

DOWNLOADER_MIDDLEWARES = {
    # "douban.middlewares.DoubanDownloaderMiddleware": 543,
    "douban.middlewares.UserAgentDownloaderMiddleware": 543,
}

REQUEST_FINGERPRINTER_IMPLEMENTATION = "2.7"
TWISTED_REACTOR = "twisted.internet.asyncioreactor.AsyncioSelectorReactor"
FEED_EXPORT_ENCODING = "utf-8"

""" scrapy-redis配置 """
# 调度器类
SCHEDULER = "scrapy_redis.scheduler.Scheduler"

# 指纹去重类
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
# 可以替换成布隆过滤器
# 下载 - pip install scrapy-redis-bloomfilter
# from scrapy_redis_bloomfilter.dupefilter import RFPDupeFilter
# DUPEFILTER_CLASS = 'scrapy_redis_bloomfilter.dupefilter.RFPDupeFilter'

# 是否在关闭时候保留原来的调度器和去重记录，True=保留，False=清空
SCHEDULER_PERSIST = True

# Redis服务器地址
REDIS_URL = "redis://127.0.0.1:6379/0"  # Redis默认有16库，/1的意思是使用序号为2的库，默认是0号库（这个可以任意）

SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.PriorityQueue'  # 使用有序集合来存储
# SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.FifoQueue'  # 先进先出
# SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.LifoQueue'  # 先进后出、后进先出

# 配置redis管道
# from scrapy_redis.pipelines import RedisPipeline
ITEM_PIPELINES = {
    "douban.pipelines.DoubanPipeline": 301,
    'scrapy_redis.pipelines.RedisPipeline': 300  # 需要将redis管道设置为优先保存
}

# 重爬: 一般不配置，在分布式中使用重爬机制会导致数据混乱
# SCHEDULER_FLUSH_ON_START = True
```

`middlewares.py`

```python
# 自定义下载中间件
class UserAgentDownloaderMiddleware:
    USER_AGENTS_LIST = [
        "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
        "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
        "Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)",
        "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1",
        "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0",
        "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5"
    ]

    def process_request(self, request, spider):
        """
        如果返回None, 表示当前的request提交给下载器或者其他的中间件
        如果返回的是request对象, 则表示将当前对象提交给调度器进行重新请求
        """
        print('下载中间件 ---> process_request')
        # print('请求对象:', request)
        user_agent = random.choice(self.USER_AGENTS_LIST)
        request.headers['User-Agent'] = user_agent
        return None
```

`insert_start_url.py`

```python
import redis

redis_client = redis.Redis()
redis_client.lpush('top250:start_urls', 'https://movie.douban.com/top250?start=0&filter=')
print('插入完成...')
redis_client.close()
```

**当当网爬虫实战**

站点地址：[http://search.dangdang.com/?key=Python&act=input](http://search.dangdang.com/?key=Python&act=input)

- 项目初始化

```shell
scrapy startproject dangdang_book
```

- 创建爬虫文件

```shell
cd dangdang_book
scrapy genspider book http://search.dangdang.com/
```

- 修改父类以及配置`redis_key`

```python
import scrapy
from scrapy_redis.spiders import RedisSpider

class BookSpider(RedisSpider):
    name = "book"
    allowed_domains = ["search.dangdang.com"]
    # start_urls = ["http://search.dangdang.com/"]
    redis_key = 'dd_book:start_url'

    def parse(self, response, **kwargs):
        pass
```

- 修改配置文件

```python
# 持久化配置
SCHEDULER_PERSIST = True
# 使用scrapy-redis调度器
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
# scrapy-redis指纹过滤器
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
# redis链接地址
REDIS_URL = 'redis://127.0.0.1:6379/0'
# 任务的优先级别
SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.PriorityQueue'
# 存放的管道
ITEM_PIPELINES = {
    "scrapy_redis.pipelines.RedisPipeline": 300,
    'dangdang_book.pipelines.MySQLPipeline': 301,
}

DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36'
}
```

- 管道配置

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html

# useful for handling different item types with a single interface
from itemadapter import ItemAdapter
import pymysql

class MySQLPipeline:
    def open_spider(self, spider):
        # 判断是哪个爬虫, 名字不同执行的爬虫项目也不同
        if spider.name == 'book':
            self.db = pymysql.connect(host="localhost", user="root", password="root", db="py_spider")
            self.cursor = self.db.cursor()
            # 创建表语法
            sql = '''
            CREATE TABLE IF NOT EXISTS book_info(
                id int primary key auto_increment not null,
                title VARCHAR(255) NOT NULL,
                price VARCHAR(255) NOT NULL,
                author VARCHAR(255) NOT NULL,
                date_data VARCHAR(255) NOT NULL,
                detail TEXT,
                producer VARCHAR(255) NOT NULL)
            '''
            try:
                self.cursor.execute(sql)
                print("表创建成功...")
            except Exception as e:
                print(f"表创建失败:", e)

    def process_item(self, item, spider):
        if spider.name == 'book':
            # SQL 插入语句
            sql = """INSERT INTO book_info(id, title, price, author, date_data, detail, producer)
            values (%s, %s, %s, %s, %s, %s, %s)"""
            # 执行 SQL 语句
            try:
                self.cursor.execute(sql, (
                    0, item['title'], item['price'], item['author'], item['date_data'], item['detail'],
                    item['producer']))
                # 提交到数据库执行
                self.db.commit()
                print('数据插入成功:', (
                    0, item['title'], item['price'], item['author'], item['date_data'], item['detail'],
                    item['producer']))
            except Exception as e:
                print(f'数据插入失败: {e}')
                # 如果发生错误就回滚
                self.db.rollback()
        return item  # 将数据提交给redis管道

    def close_spider(self, spider):
        # 关闭数据库连接
        if spider.name == 'book':
            self.db.close()
```

- `items.py`配置

```python
# Define here the models for your scraped items
#
# See documentation in:
# https://docs.scrapy.org/en/latest/topics/items.html

import scrapy

class DangDangBookItem(scrapy.Item):
    title = scrapy.Field()
    price = scrapy.Field()
    author = scrapy.Field()
    date_data = scrapy.Field()
    detail = scrapy.Field()
    producer = scrapy.Field()
```

- 完善`parse`函数解析逻辑

```python
import scrapy
from scrapy import cmdline

# 使用上一级路径导入Item文件, 如果使用当前导入方式请使用命令启动项目
from ..items import DangDangBookItem
from scrapy_redis.spiders import RedisSpider

class BookSpider(RedisSpider):
    name = "book"
    allowed_domains = ["search.dangdang.com"]
    # start_urls = ["http://search.dangdang.com/"]
    redis_key = 'dd_book:start_url'

    def parse(self, response, **kwargs):
        li_list = response.xpath('//ul[@class="bigimg"]/li')
        for li in li_list:
            item = DangDangBookItem()
            item['title'] = li.xpath('./a/@title').extract_first()
            item['price'] = li.xpath('./p[@class="price"]/span[1]/text()').extract_first()
            item['author'] = li.xpath('./p[@class="search_book_author"]/span[1]/a[1]/@title').extract_first()
            item['date_data'] = li.xpath('./p[@class="search_book_author"]/span[last()-1]/text()').extract_first()
            item['detail'] = li.xpath('./p[@class="detail"]/text()').extract_first() if li.xpath(
                './p[@class="detail"]/text()') else '空'
            item['producer'] = li.xpath(
                './p[@class="search_book_author"]/span[last()]/a/text()').extract_first() if li.xpath(
                './p[@class="search_book_author"]/span[last()]/a/text()') else '空'

            yield item

        if response.xpath('//ul[@name="Fy"]/li[@class="next"]/a/@href').extract_first() is not None:
            next_url = response.urljoin(response.xpath('//ul[@name="Fy"]/li[@class="next"]/a/@href').extract_first())
            yield scrapy.Request(url=next_url, callback=self.parse)

# if __name__ == '__main__':
#     cmdline.execute('scrapy crawl book'.split())
```

- `redis`存储`start_url`运行脚本

`insert_start_url.py`

```python
import redis

redis_client = redis.Redis()
redis_client.lpush('dd_book:start_url', 'http://search.dangdang.com/?key=python&act=input&page_index=1')
redis_client.close()
```

- 从`redis`中获取数据

`get_redis_info.py`

```python
import json
import redis

redis_client = redis.Redis()

for temp in redis_client.lrange('book:items', 0, -1):
    print(json.loads(temp))

redis_client.close()
```

**发送`post`请求 - 表单数据**

在当前案例中我们需要掌握如何使用`scrapy-redis`发送`post`请求并完成翻页。

站点请求地址：[http://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse](http://www.cninfo.com.cn/new/commonUrl?url=disclosure/list/notice#szse)

`jc.py`

```python
import json
from scrapy import cmdline
from scrapy.http import FormRequest
from scrapy_redis.spiders import RedisSpider

class JcSpider(RedisSpider):
    name = "jc"
    allowed_domains = ["www.cninfo.com.cn"]
    # start_urls = ["http://www.cninfo.com.cn/"]
    redis_key = 'jc:start_urls'

    def make_request_from_data(self, data):
        url = 'http://www.cninfo.com.cn/new/disclosure'
        """
        :param data: 是scrapy-redis读取redis中的[url, form_data, meta]
        :return:
        """
        data = json.loads(data)
        form_data = data.get('form_data')
        print(form_data)

        headers = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'
        }

        return FormRequest(url=url, headers=headers, formdata=form_data, callback=self.parse)

    def parse(self, response, **kwargs):
        print('返回信息:', response.json())

if __name__ == '__main__':
    cmdline.execute('scrapy crawl jc'.split())
```

`push_start_url_data.py`

```python
import json
import redis

def push_start_url_data(db, request_obj):
    """
    :param db: redis链接对象
    :param request_obj: {'url':url, 'form_data':form_data, 'meta':meta}
    :return:
    """
    db.lpush('jc:start_urls', request_obj)

if __name__ == '__main__':
    redis_client = redis.Redis()
    # 需要将表单中的数字类型强转为字符串类型
    for page in range(1, 26):
        form_data = {
            'column': 'szse_latest',
            'pageNum': str(page),
            'pageSize': '30',
            'sortName': '',
            'sortType': '',
            'clusterFlag': 'true'
        }

        request_data = {
            'form_data': form_data
        }

        push_start_url_data(redis_client, json.dumps(request_data))
    redis_client.close()
```

`settings.py`

```python
"""scrapy-redis配置"""
# 调度器类
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
# 指纹去重类
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
# 是否在关闭时候保留原来的调度器和去重记录，True=保留，False=清空
SCHEDULER_PERSIST = True
# Redis服务器地址
REDIS_URL = "redis://127.0.0.1:6379/0"
```

**发送`post`请求 - 载荷数据**

`job_info.py`

```python
import json
from scrapy import cmdline
from scrapy.http import JsonRequest
from scrapy_redis.spiders import RedisSpider

class JobInfoSpider(RedisSpider):
    name = 'job_info'
    allowed_domains = ['hr.163.com']
    # start_urls = ['https://hr.163.com/job-list.html?keyword=python']
    redis_key = 'job:start_urls'

    def make_request_from_data(self, data):
        url = 'https://hr.163.com/api/hr163/position/queryPage'
        data = json.loads(data)
        json_data = data.get('json_data')
        print('载荷信息为:', json_data)
        return JsonRequest(url, data=json_data, callback=self.parse)

    def parse(self, response, **kwargs):
        print('api返回的数据为:', response.json())

if __name__ == '__main__':
    cmdline.execute('scrapy crawl job_info'.split())
```

`push_start_url_json.py`

```python
import json
import redis

def push_start_url_data(db, request_obj):
    db.lpush('job:start_urls', request_obj)

if __name__ == '__main__':
    redis_client = redis.Redis()
    for page in range(1, 26):
        json_data = {
            "currentPage": page,
            "pageSize": 10,
            "keyword": "python"
        }

        request_data = {
            'json_data': json_data
        }

        push_start_url_data(redis_client, json.dumps(request_data))
    redis_client.close()
```

`settings.py`

```python
ROBOTSTXT_OBEY = False

DEFAULT_REQUEST_HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36',
    'Referer': 'https://hr.163.com/job-list.html?keyword=python',
}

SCHEDULER = "scrapy_redis.scheduler.Scheduler"
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
SCHEDULER_PERSIST = True
REDIS_URL = "redis://127.0.0.1:6379/0"
SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.PriorityQueue'
```

> 来自: [10.2.scrapy-redis实现分布式爬虫](https://www.yuque.com/tuling_python/spider_base/qp0qgyq64f2g7a2a)

### 10.3.项目部署 - scrapyd

**`scrapyd`的概念**

`scarpy`是一个爬虫框架，而`scrapyd`相当于一个组件，能够将`scrapy`项目进行远程部署，调度使用等。

**服务端相关配置 - 执行爬虫的服务器**

安装`scrapyd`

```bash
pip install scrapyd  -i https://pypi.tuna.tsinghua.edu.cn/simple
```

运行`scrapyd`

在服务端执行`scrapyd`指令

![](images/image-049.png)

修改配置让`scrapyd`支持远程访问

使用`ctrl+c`停止上一步运行的`scrapyd`，并在你想要运行爬虫项目的路径之下新建`scrapyd.conf`文件。

![](images/image-050.png)

![](images/image-051.png)

输入的内容如下：

```shell
[scrapyd]
# 监听的IP地址，默认为127.0.0.1（只有改成0.0.0.0才能在别的电脑上能够访问scrapyd运行之后的服务器）
bind_address = 0.0.0.0
# 监听的端口，默认为6800
http_port = 6800
# 是否打开debug模式，默认为off
debug = off
```

重新运行

在刚刚新建`scrpayd.conf`文件所在路径下通过终端运行`scrapyd`。

![](images/image-052.png)

在物理机上使用浏览器访问服务器

![](images/image-053.png)

**客户端相关配置 - 将本地编写的爬虫代码上传到服务端**

安装`scrapyd-client`

```bash
pip install scrapyd-client==1.2.3 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

配置`scrapy`项目

![](images/image-054.png)

如果运行的`scrapy`的服务器只有一个则配置一个`deploy`即可，如果有多台服务器则配置多个`deploy`。

检查配置是否生效

在`scrapy`项目根路径之下运行如下命令：

```shell
# 注意是小写的L，不是数字1
scrapyd-deploy -l
```

![](images/image-055.png)

发布`scrapy`项目到`scrapyd`所在的服务器（当前爬虫是未运行状态）

```shell
scrapyd-deploy  -p  --version
```

- `target`：就是前面配置文件里`deploy`后面的的`target`名字，例如`ubuntu-1`
- `project`：可以随意定义，跟爬虫的工程名字无关，一般建议与`scrapy`爬虫项目名相同
- `version`：自定义版本号，不写的话默认为当前时间戳，一般不写

```shell
scrapyd-deploy ubuntu-1 -p dangdang_book
```

爬虫目录下不要放无关的`py`文件，放无关的`py`文件会导致发布失败。当爬虫发布成功后，会在当前目录生成一个`setup.py`文件，可以删除掉。

![](images/image-056.png)

发布成功后可以在服务端查看到上传的爬虫项目，效果如下：

![](images/image-057.png)

刷新物理机上的浏览器则显示上传成功后的`scrapy`项目，效果如下：

![](images/image-058.png)

**启动通过`scrapyd`部署的爬虫**

**运行指令**

`scrapyd`已经给出了怎样开始运行爬虫，如下图所示：

![](images/image-059.png)

将上述的命令复制，之后在终端中进行适当的修改即可开启爬虫，命令如下：

```bash
curl http://192.168.70.82:6800/schedule.json -d project=dangdang_book -d spider=book
```

![](images/image-060.png)

爬虫启动成功后可在浏览器中查看爬虫运行状态，在主页中点击`Jobs`，效果如下：

![](images/image-061.png)

验证服务端是否开始抓取信息：

![](images/image-062.png)

验证服务端中的`Redis`信息：

![](images/image-063.png)

**停止爬虫**

停止爬虫的指令如下：

```python
curl http://ip:6800/cancel.json -d project=项目名 -d job=任务的id值
```

任务的`id`值可在浏览器中查询：

![](images/image-064.png)

根据页面中显示的`id`编辑指令：

```bash
curl http://192.168.70.82:6800/cancel.json -d project=dangdang_book -d job=55b4b87a98c111eeb1af001c42c139d2
```

![](images/image-065.png)

浏览器查询爬虫运行状态：

![](images/image-066.png)

**总结**

- 如果`scrapy`项目代码，修改了，只需要重新发布到`scrapyd`所在服务器即可
- 如果`scrapy`项目暂停了，可以再次通过`curl`的方式发送命令让其"断点续爬"

> 来自: [10.3.项目部署 - scrapyd](https://www.yuque.com/tuling_python/spider_base/cag8i6hpwsr82ztp)

### 10.4.项目部署 - scrapydweb

**简介**

`scrapyweb`是一个基于`scrapyd`的可视化组件，集成并且提供更多可视化功能和更优美的界面。`scrapydweb`后端是采用`flask`框架编写的，所以对于会`flask`的同学是可以适当定制的。

**安装与运行**

**安装方式**

```shell
pip install scrapydweb -i https://pypi.tuna.tsinghua.edu.cn/simple
```

**运行指令**

在运行`scrapydweb`之前一定要确保`scrapyd`正在运行，可以在`scrapyd`服务启动之后重新创建一个新终端窗口来启动`scrapydweb`。

```shell
scrapyd
scrapydweb
```

`scrapydweb`项目地址：[https://github.com/my8100/scrapydweb?tab=readme-ov-file](https://github.com/my8100/scrapydweb?tab=readme-ov-file)

`scrapydweb`第一次启动会报错，报错之后重新启动即可，重新启动后会在启动路径的位置生成脚本文件。如果重新启动失败并抛出版本依赖问题请查看项目地址中的`requirements.txt`文件并安装最新依赖。

当成功启动后可使用物理机浏览器访问地址：[http://192.168.70.82:5000/](http://192.168.70.82:5000/)

显示如下：

![](images/image-067.png)

**基本使用**

**发布`scrapy`项目**

![](images/image-068.png)

在计算机中搜索需要部署的项目，项目可以打包成压缩包，在打包压缩包之前需要确认项目中只有`scrapy`框架代码。

![](images/image-069.png)

![](images/image-070.png)

项目上传成功后的效果如下：

![](images/image-071.png)

项目运行：

![](images/image-072.png)

> 来自: [10.4.项目部署 - scrapydweb](https://www.yuque.com/tuling_python/spider_base/noot8tllatn4186w)

### 10.5.项目部署 - gerapy

`gerapy`是一款国人开发的爬虫管理软件（有中文界面）。它是一个管理爬虫项目的可视化工具，把项目部署到管理的操作全部变为交互式，实现批量部署，更方便控制、管理、实时查看结果。

`gerapy`和`scrapyd`的关系就是：我们可以通过`gerapy`中配置`scrapyd`后，不使用命令，直接通过图形化界面开启爬虫。

**安装方式**

安装：`gerapy`作为客户端所以需要安装到物理机中

```bash
# 强烈建议安装前创建新的虚拟环境, gerapy下载时会自动安装最新版本的scrapy框架
# conda create -n spider_deploy python=3.9
pip install gerapy==0.9.12 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

安装成功后可以在终端输入指令进行测试：

![](images/image-073.png)

**使用方式**

新建文件夹并启动指令：

```shell
gerapy init
```

以下为`gerapy`配置操作：

```shell
cd gerapy
gerapy migrate  # 同步sqlite数据库
gerapy createsuperuser  # 创建超级管理员
gerapy runserver  # 启动服务，访问地址为：127.0.0.1:8000
```

输入用户名密码即可登录：

![](images/image-074.png)

登录成功后显示如下：

![](images/image-075.png)

**配置完成后创建主机**

![](images/image-076.png)

主机连接成功之后的效果如下：

![](images/image-077.png)

**运行已经存在的项目**

在主机管理页面中点击调度则可以查看到已经存在的项目状态：

![](images/image-078.png)

当前页面中有运行按钮，如果想要运行当前爬虫则点击运行即可，如果运行之后想要停止则页面中会显示停止按钮，点击即可。

**本地项目上传**

在项目管理中点击上传并将项目压缩包上传即可，压缩包类型必须为`zip`。

![](images/image-079.png)

> 来自: [10.5.项目部署 - gerapy](https://www.yuque.com/tuling_python/spider_base/vgpi60w68ibmfgiz)

## 11.feapder框架

**简介**

- `feapder`是一款上手简单，功能强大的`Python`爬虫框架，内置`AirSpider`、`Spider`、`TaskSpider`、`BatchSpider`四种爬虫解决不同场景的需求。
- 支持断点续爬、监控报警、浏览器渲染、海量数据去重等功能。
- 更有功能强大的爬虫管理系统`feaplat`为其提供方便的部署及调度

**文档地址与环境配置**

官方文档：[https://feapder.com](https://feapder.com)

```shell
# 在安装之前建议使用miniconda3创建一个新的虚拟环境
conda create -n feapder_base python=3.9
conda activate feapder_base

# 完整版安装指令
pip install "feapder[all]"
```

**架构设计**

![](images/image-080.png)

模块名称

模块功能

`spider`

框架调度核心

`parser_control模版控制器`

负责调度`parser`

`collector任务收集器`

负责从任务队里中批量取任务到内存，以减少爬虫对任务队列数据库的访问频率及并发量

`parser`

数据解析器

`start_request`

初始任务下发函数

`item_buffer数据缓冲队列`

批量将数据存储到数据库中

`request_buffer请求任务缓冲队列`

批量将请求任务存储到任务队列中

`request数据下载器`

封装了`requests`

，用于从互联网上下载数据

`response请求响应`

封装了`response`, 支持`xpath`、`css`、`re`等解析方式，自动处理中文乱码

**执行流程**

- `spider`调度`start_request`生产任务
- `start_request`下发任务到`request_buffer`中
- `spider`调度`request_buffer`批量将任务存储到任务队列数据库中
- `spider`调度`collector`从任务队列中批量获取任务到内存队列
- `spider`调度`parser_control`从`collector`的内存队列中获取任务
- `parser_control`调度`request`请求数据
- `request`请求与下载数据
- `request`将下载后的数据给`response`，进一步封装
- 将封装好的`response`返回给`parser_control`（图示为多个`parser_control`，表示多线程）
- `parser_control`调度对应的`parser`，解析返回的`response`（图示多组`parser`表示不同的网站解析器）
- `parser_control`将`parser`解析到的数据`item`及新产生的`request`分发到`item_buffer`与`request_buffer`
- `spider`调度`item_buffer`与`request_buffer`将数据批量入库

> 来自: [11.feapder框架](https://www.yuque.com/tuling_python/spider_base/gd6ygs0sd4kkl3g6)

### 11.1.feapder框架的简单使用

**创建爬虫**

```shell
feapder create -s douban
```

执行命令后需要手动选择对应的爬虫模板，模板功能如下：

- `AirSpider` 轻量爬虫：学习成本低，可快速上手
- `Spider` 分布式爬虫：支持断点续爬、爬虫报警、数据自动入库等功能
- `TaskSpider`分布式爬虫：内部封装了取种子任务的逻辑，内置支持从`redis`或者`mysql`获取任务，也可通过自定义实现从其他来源获取任务
- `BatchSpider` 批次爬虫：可周期性的采集数据，自动将数据按照指定的采集周期划分。（如每7天全量更新一次商品销量的需求）

命令执行成功后选择`AirSpider`模板。默认生成的代码继承了`feapder.AirSpider`，包含 `start_requests` 及 `parser` 两个函数，含义如下：

- `feapder.AirSpider`：轻量爬虫基类
- `start_requests`：初始任务下发入口
- `feapder.Request`：基于`requests`库类似，表示一个请求，支持`requests`所有参数，同时也可携带些自定义的参数，详情可参考[Request](https://boris-code.gitee.io/feapder/#/source_code/Request)
- `parse`：数据解析函数
- `response`：请求响应的返回体，支持`xpath`、`re`、`css`等解析方式，详情可参考[Response](https://boris-code.gitee.io/feapder/#/source_code/Response)

除了`start_requests`、`parser`两个函数。系统还内置了下载中间件等函数，具体支持可参考[BaseParser](https://boris-code.gitee.io/feapder/#/source_code/BaseParser)

**使用`AirSpider`模板完成豆瓣爬虫**

`douban.py`

```python
# -*- coding: utf-8 -*-
"""
Created on 2023-12-18 19:10:28
---------
@summary:
---------
@author: poppies
"""

import feapder

class Douban(feapder.AirSpider):
    def start_requests(self):
        for page in range(10):
            yield feapder.Request(f"https://movie.douban.com/top250?start={page * 25}&filter=")

    def parse(self, request, response):
        li_list = response.xpath('//ol/li/div[@class="item"]')
        for li in li_list:
            item = dict()
            item['title'] = li.xpath('./div[@class="info"]/div/a/span[1]/text()').extract_first()
            item['detail_url'] = li.xpath('./div[@class="info"]/div/a/@href').extract_first()
            item['score'] = li.xpath('.//div[@class="star"]/span[2]/text()').extract_first()
            yield feapder.Request(item['detail_url'], callback=self.parse_detail, item=item)

    def parse_detail(self, request, response):
        if response.xpath('//div[@class="indent"]/span[@class="all hidden"]//text()'):
            request.item['detail_text'] = response.xpath(
                '//div[@class="indent"]/span[@class="all hidden"]//text()').extract_first().strip()
        else:
            request.item['detail_text'] = response.xpath(
                '//div[@class="indent"]/span[1]//text()').extract_first().strip()
        print(request.item)

if __name__ == "__main__":
    # 开启五个线程完成爬虫任务
    Douban(thread_count=5).start()
```

爬虫抓取成功后将数据存放到`MySQL`中，`feapder`框架已经集成了数据库增删改查的功能。下面我们以`MySQL`数据库为例，使用内置的`MysqlDB`模块完成数据的保存。

**`feapder`对接`MySQL`完成数据保存**

在当前目录下创建`insert_sql_info.py`文件用于数据库测试：

```python
from feapder.db.mysqldb import MysqlDB

db = MysqlDB(ip='localhost', port=3306, user_name='root', user_pass='root', db='py_spider')

sql = """
    create table if not exists douban_feapder(
        id int primary key auto_increment,
        title varchar(255) not null,
        score varchar(255) not null,
        detail_url varchar(255) not null,
        detail_text text
    );
"""
db.execute(sql)

# insert ignore: 数据插入，如果数据重复则忽略，例如id重复
insert_sql = """
    insert ignore into douban_feapder (id, title, score, detail_url, detail_text) values (
        0, '测试数据', 10, 'https://www.baidu.com', '测试数据'
    );
"""

db.add(insert_sql)
```

根据以上案例将豆瓣爬虫中获取的数据存储到`MySQL`中：

- 在项目文件夹之下创建配置文件用于连接`MySQL`

```shell
feapder create --setting
```

- 在`setting.py`文件中激活`MySQL`数据库配置

```python
# MYSQL
MYSQL_IP = "localhost"
MYSQL_PORT = 3306
MYSQL_DB = "py_spider"
MYSQL_USER_NAME = "root"
MYSQL_USER_PASS = "root"
```

- 创建`items`文件

```shell
# 在创建items文件之前必须确保文件名与数据库已存在的表名一致
feapder create -i douban_feapder
```

```python
# -*- coding: utf-8 -*-
"""
Created on 2023-12-18 20:10:06
---------
@summary:
---------
@author: poppies
"""

from feapder import Item

class DoubanFeapderItem(Item):
    """
    This class was generated by feapder
    command: feapder create -i douban_feapder
    """

    __table_name__ = "douban_feapder"

    def __init__(self, *args, **kwargs):
        # self.id = None
        self.title = None
        self.score = None
        self.detail_url = None
        self.detail_text = None
```

- 将生成的`DoubanFeapderItem`类载入到`douban.py`文件中

```python
# -*- coding: utf-8 -*-
"""
Created on 2023-12-18 19:10:28
---------
@summary:
---------
@author: poppies
"""

import feapder
from douban_feapder_item import DoubanFeapderItem

class Douban(feapder.AirSpider):
    def start_requests(self):
        for page in range(11):
            yield feapder.Request(f"https://movie.douban.com/top250?start={page * 25}&filter=")

    def parse(self, request, response):
        li_list = response.xpath('//ol/li/div[@class="item"]')
        for li in li_list:
            # 将字典类型替换成DoubanFeapderItem用于数据校验
            item = DoubanFeapderItem()
            item['title'] = li.xpath('./div[@class="info"]/div/a/span[1]/text()').extract_first()
            item['detail_url'] = li.xpath('./div[@class="info"]/div/a/@href').extract_first()
            item['score'] = li.xpath('.//div[@class="star"]/span[2]/text()').extract_first()
            yield feapder.Request(item['detail_url'], callback=self.parse_detail, item=item)

    def parse_detail(self, request, response):
        if response.xpath('//div[@class="indent"]/span[@class="all hidden"]//text()'):
            request.item['detail_text'] = response.xpath(
                '//div[@class="indent"]/span[@class="all hidden"]//text()').extract_first().strip()
        else:
            request.item['detail_text'] = response.xpath(
                '//div[@class="indent"]/span[1]//text()').extract_first().strip()

        # 执行yield会将解析好的数据保存到数据库中
        yield request.item

if __name__ == "__main__":
    Douban().start()
```

**下载中间件**

- 下载中间件用于在请求之前，对请求做一些处理，如添加`cookie`、`header`等
- 默认所有的解析函数在请求之前都会经过此下载中间件

```python
# -*- coding: utf-8 -*-
"""
Created on 2023-12-18 19:10:28
---------
@summary:
---------
@author: poppies
"""

import feapder

class Douban(feapder.AirSpider):
    def start_requests(self):
        for page in range(11):
            yield feapder.Request(f"https://movie.douban.com/top250?start={page * 25}&filter=")

    # 默认的下载中间件
    def download_midware(self, request):
        request.headers = {
            'User-Agent': 'abc'
        }
        request.proxies = {
            "http": "http://127.0.0.1:7890"
        }
        return request

if __name__ == "__main__":
    Douban().start()
```

除了可以重写默认的下载中间件之外，也可以自定义下载中间件：使用`Request`对象中的`download_midware`参数指定自己创建的中间件方法名即可。

```python
# -*- coding: utf-8 -*-
"""
Created on 2023-12-18 19:10:28
---------
@summary:
---------
@author: poppies
"""

import feapder

class Douban(feapder.AirSpider):
    def start_requests(self):
        for page in range(11):
            yield feapder.Request(f"https://movie.douban.com/top250?start={page * 25}&filter=",
                                  download_midware=self.custom_download_midware)

    def custom_download_midware(self, request):
        request.headers = {
            'User-Agent': '123'
        }
        return request

if __name__ == "__main__":
    Douban().start()
```

**校验响应对象**

- `feapder`框架给到一个方法`validate`用来检验返回的数据是否正常。
- 框架支持重试机制，下载失败或解析函数抛出异常会自动重试请求。
- 默认最大重试次数为`100`次，我们可以引入配置文件或自定义配置来修改重试次数

```python
# 校验函数源码
def validate(self, request, response):
    """
    @summary: 校验函数, 可用于校验response是否正确
    若函数内抛出异常，则重试请求
    若返回True或None，则进入解析函数
    若返回False，则抛弃当前请求
    可通过request.callback_name 区分不同的回调函数，编写不同的校验逻辑
    ---------
    @param request:
    @param response:
    ---------
    @result: True / None / False
    """

    pass
```

代码示例

```python
# -*- coding: utf-8 -*-
"""
Created on 2023-12-18 19:10:28
---------
@summary:
---------
@author: poppies
"""

import feapder
from douban_feapder_item import DoubanFeapderItem

class Douban(feapder.AirSpider):
    def start_requests(self):
        for page in range(11):
            yield feapder.Request(f"https://movie.douban.com/top250?start={page * 25}&filter=", download_midware=self.custom_download_midware)

    def custom_download_midware(self, request):
        request.headers = {
            'User-Agent': '123'
        }

        request.proxies = {
            "http": "http://127.0.0.1:7890"
        }
        return request

    def parse(self, request, response):
        li_list = response.xpath('//ol/li/div[@class="item"]')
        for li in li_list:
            # 将字典类型替换成DoubanFeapderItem用于数据校验
            item = DoubanFeapderItem()
            item['title'] = li.xpath('./div[@class="info"]/div/a/span[1]/text()').extract_first()
            item['detail_url'] = li.xpath('./div[@class="info"]/div/a/@href').extract_first()
            item['score'] = li.xpath('.//div[@class="star"]/span[2]/text()').extract_first()
            yield feapder.Request(item['detail_url'], callback=self.parse_detail, item=item)

    def parse_detail(self, request, response):
        if response.xpath('//div[@class="indent"]/span[@class="all hidden"]//text()'):
            request.item['detail_text'] = response.xpath(
                '//div[@class="indent"]/span[@class="all hidden"]//text()').extract_first().strip()
        else:
            request.item['detail_text'] = response.xpath(
                '//div[@class="indent"]/span[1]//text()').extract_first().strip()

        # 执行yield会将解析好的数据保存到数据库中
        yield request.item

    def validate(self, request, response):
        print('响应状态码:', response.status_code)
        if response.status_code != 200:
            raise Exception("状态码异常")  # 请求重试

if __name__ == "__main__":
    Douban().start()
```

**浏览器渲染 - `selenium`**

采集动态页面时（`Ajax`渲染的页面），常用的有两种方案。一种是找接口拼参数，这种方式比较复杂但效率高，需要一定的爬虫功底；另外一种是采用浏览器渲染的方式，直接获取源码，简单方便

框架内置一个浏览器渲染池，默认的池大小为1，请求时重复利用浏览器实例，只有当代理失效请求异常时，才会销毁、创建一个新的浏览器实例

内置浏览器渲染支持 `CHROME`、`EDGE`、`PHANTOMJS`、`FIREFOX`

使用方式

```python
def start_requests(self):
    # 在返回的Request中传递render=True即可。
    yield feapder.Request("https://news.qq.com/", render=True)
```

注意点：需要在`setting.py`文件中开启自动化配置。

```python
# 在setting.py中有以下代码配置

# 浏览器渲染
WEBDRIVER = dict(
    pool_size=1,  # 浏览器的数量
    load_images=True,  # 是否加载图片
    user_agent=None,  # 字符串 或 无参函数，返回值为user_agent
    proxy=None,  # xxx.xxx.xxx.xxx:xxxx 或 无参函数，返回值为代理地址
    headless=False,  # 是否为无头浏览器
    driver_type="CHROME",  # CHROME、EDGE、PHANTOMJS、FIREFOX
    timeout=30,  # 请求超时时间
    window_size=(1024, 800),  # 窗口大小
    executable_path=None,  # 浏览器路径，默认为默认路径
    render_time=0,  # 渲染时长，即打开网页等待指定时间后再获取源码
    custom_argument=[
        "--ignore-certificate-errors",
        "--disable-blink-features=AutomationControlled",
    ],  # 自定义浏览器渲染参数
    xhr_url_regexes=None,  # 拦截xhr接口，支持正则，数组类型
    auto_install_driver=True,  # 自动下载浏览器驱动 支持chrome 和 firefox
    download_path=None,  # 下载文件的路径
    use_stealth_js=False,  # 使用stealth.min.js隐藏浏览器特征
)
```

以上配置含有浏览器驱动路径：`executable_path`，如果在默认情况下启动报错则手动配置浏览器驱动文件路径。

示例代码

```python
import feapder
from selenium.webdriver.common.by import By
from feapder.utils.webdriver import WebDriver

class Baidu(feapder.AirSpider):
    def start_requests(self):
        yield feapder.Request("https://www.baidu.com", render=True)

    def parse(self, request, response):
        browser: WebDriver = response.browser
        browser.find_element(By.ID, 'kw').send_keys('feapder')
        browser.find_element(By.ID, 'su').click()

if __name__ == "__main__":
    Baidu().start()
```

**拦截动态数据接口**

```python
WEBDRIVER = dict(
    ...
    xhr_url_regexes=[
        "接口1正则",
        "接口2正则",
    ]
)
```

**获取数据**

```python
browser: WebDriver = response.browser
# 提取文本
text = browser.xhr_text("接口1正则")
# 提取json
data = browser.xhr_json("接口2正则")
```

**获取对象**

```python
browser: WebDriver = response.browser
xhr_response = browser.xhr_response("接口正则")
print("请求接口", xhr_response.request.url)
print("请求头", xhr_response.request.headers)
print("请求体", xhr_response.request.data)
print("返回头", xhr_response.headers)
print("返回地址", xhr_response.url)
print("返回内容", xhr_response.content)
```

**官方文档给出的测试代码**

文档地址：[https://feapder.com/#/source_code/浏览器渲染-Selenium?id=拦截xhr数据](https://feapder.com/#/source_code/%E6%B5%8F%E8%A7%88%E5%99%A8%E6%B8%B2%E6%9F%93-Selenium?id=%e6%8b%a6%e6%88%aaxhr%e6%95%b0%e6%8d%ae)

`test_render.py`

```python
import time
import feapder
from feapder.utils.webdriver import WebDriver

class TestRender(feapder.AirSpider):
    def start_requests(self):
        yield feapder.Request("https://spidertools.cn", render=True)

    def parse(self, request, response):
        browser: WebDriver = response.browser
        time.sleep(3)

        # 获取接口数据 文本类型
        ad = browser.xhr_text("/ad")
        print(ad)

        # 获取接口数据 转成json，本例因为返回的接口是文本，所以不转了
        # browser.xhr_json("/ad")

        xhr_response = browser.xhr_response("/ad")
        print("请求接口: ", xhr_response.request.url)
        # 请求头目前获取不完整
        print("请求头: ", xhr_response.request.headers)

        print("请求体: ", xhr_response.request.data)
        print("返回头: ", xhr_response.headers)
        print("返回地址: ", xhr_response.url)
        print("返回内容: ", xhr_response.content)
```

`setting.py`

```python
WEBDRIVER = dict(
    pool_size=1,  # 浏览器的数量
    load_images=True,  # 是否加载图片
    user_agent=None,  # 字符串 或 无参函数，返回值为user_agent
    proxy=None,  # xxx.xxx.xxx.xxx:xxxx 或 无参函数，返回值为代理地址
    headless=False,  # 是否为无头浏览器
    driver_type="CHROME",  # CHROME、EDGE、PHANTOMJS、FIREFOX
    timeout=30,  # 请求超时时间
    window_size=(1024, 800),  # 窗口大小
    executable_path=None,  # 浏览器路径，默认为默认路径
    render_time=0,  # 渲染时长，即打开网页等待指定时间后再获取源码
    custom_argument=[
        "--ignore-certificate-errors",
        "--disable-blink-features=AutomationControlled",
    ],  # 自定义浏览器渲染参数
    xhr_url_regexes=["/ad"],  # 拦截xhr接口，支持正则，数组类型
    auto_install_driver=False,  # 自动下载浏览器驱动 支持chrome 和 firefox
    download_path=None,  # 下载文件的路径
    use_stealth_js=False,  # 使用stealth.min.js隐藏浏览器特征
)
```

**使用浏览器渲染的方式获取应届生招聘岗位数据**

目标站点：[https://q.yingjiesheng.com/jobs/search/Python?jobarea=220200](https://q.yingjiesheng.com/jobs/search/Python?jobarea=220200)

`job_info.py`

```python
# -*- coding: utf-8 -*-
"""
Created on 2023-12-19 13:52:29
---------
@summary:
---------
@author: poppies
"""

import time
import feapder
from feapder.utils.webdriver import WebDriver

class JobInfo(feapder.AirSpider):
    def start_requests(self):
        yield feapder.Request("https://q.yingjiesheng.com/jobs/search/Python?jobarea=220200", render=True)

    def parse(self, request, response):
        browser: WebDriver = response.browser
        # 等待加载api接口数据
        time.sleep(2)

        json_data = browser.xhr_json('open/noauth/job/search')
        for temp in json_data['resultbody']['searchData']['joblist']['items']:
            item = dict()
            item['jobname'] = temp['jobname']
            item['coname'] = temp['coname']
            item['jobarea'] = temp['jobarea']
            item['issuedate'] = temp['issuedate']
            item['jobtag'] = temp['jobtag']
            item['providesalary'] = temp['providesalary']
            print(item)

if __name__ == "__main__":
    JobInfo().start()
```

> 来自: [11.1.feapder框架的简单使用](https://www.yuque.com/tuling_python/spider_base/srytw2hdq4tgh2fc)

### 11.2.feapder框架创建完整项目

**创建项目目录以及爬虫文件的相关指令**

```shell
feapder create -p
feapder create -p wp_shop
```

项目创建成功后会存在以下目录：

- `items`文件夹： 存放与数据库表映射的`item`
- `spiders`文件夹： 文件夹存放爬虫脚本
- `main.py`文件： 运行入口
- `setting.py`文件： 爬虫配置文件

进入到`spiders`文件夹创建爬虫脚本：

```shell
cd spiders
feapder create -s wp_spider
```

**`setting.py`文件的配置以及数据表的创建**

```python
# MYSQL
MYSQL_IP = "localhost"
MYSQL_PORT = 3306
MYSQL_DB = "py_spider"
MYSQL_USER_NAME = "root"
MYSQL_USER_PASS = "root"
```

在项目根目录下创建`create_table.py`文件，内容如下：

```python
from feapder.db.mysqldb import MysqlDB

db = MysqlDB(ip='localhost', user_name='root', user_pass='root', db='py_spider')
create_table_sql = """
    create table wp_shop_info(
        id int primary key auto_increment,
        title varchar(255) default null,
        price varchar(255) default null
    );
"""
db.execute(create_table_sql)
```

**创建`items`文件**

```shell
cd items

# item文件名称是数据表名称
feapder create -i wp_shop_info
```

**完成唯品会数据抓取以及数据入库**

`wp_spider.py`

```python
# -*- coding: utf-8 -*-
"""
Created on 2024-03-19 20:32:21
---------
@summary:
---------
@author: poppies
"""

import time
import feapder
from random import randint
from items import wp_shop_info_item
from selenium.webdriver.common.by import By
from feapder.utils.webdriver import WebDriver

class WpSpider(feapder.AirSpider):
    def start_requests(self):
        url = 'https://category.vip.com/suggest.php?keyword=%E7%94%B5%E8%84%91&ff=235%7C12%7C1%7C1&page={}'
        for page in range(1, 13):
            yield feapder.Request(url=url.format(page), render=True)

    def parse(self, request, response):
        browser: WebDriver = response.browser
        # 让浏览器等待加载数据
        time.sleep(2)
        # 页面下滑
        self.drop_down(browser)

        div_list = browser.find_elements(
            By.XPATH,
            '//section[@id="J_searchCatList"]/div[@class="c-goods-item  J-goods-item c-goods-item--auto-width"]'
        )
        for div in div_list:
            price = div.find_element(By.XPATH,
                                     './/div[@class="c-goods-item__sale-price J-goods-item__sale-price"]').text

            title = div.find_element(By.XPATH, './/div[2]/div[2]').text

            item = wp_shop_info_item.WpShopInfoItem()

            item['title'] = title
            item['price'] = price
            # print(item)
            yield item  # 将商品数据保存到MySQL中

    def drop_down(self, browser):
        for i in range(1, 12):
            js_code = f'document.documentElement.scrollTop = {i * 1000}'
            browser.execute_script(js_code)
            time.sleep(randint(1, 2))

if __name__ == "__main__":
    WpSpider(thread_count=1).start()
```

> 来自: [11.2.feapder框架创建完整项目](https://www.yuque.com/tuling_python/spider_base/pxggpuf1ub0kszxf)
