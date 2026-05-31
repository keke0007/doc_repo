JavaScript XHR. Fetch

## 目录

前端数据请求方式 content Http协议的解析 XHR的基本用法 XHR的进阶和封装 Fetch的使用详解 前端文件上传流程

## 前后端分离的优势

```javascript
早期的网页都是通过后端渲染来完成的：服务器端渲染(SSR，serversiderender)：
```

- 客户端发出请求->服务端接收请求并返回相应HTML文档－>页面刷新，客户端加载新的HTML文档；
服务器端渲染的缺点：

- 当用户点击页面中的某个按钮向服务器发送请求时，页面本质上只是一些数据发生了变化，而此时服务器却要将重绘的整个页面再返
- 而且明明只是一些数据的变化却迫使服务器要返回整个HTML文档，这本身也会给网络带宽带来不必要的开销。
有没有办法在页面数据变动时，只向服务器请求新的数据，并且在阻止页面刷新的情况下，动态的替换页面中展示的数据呢？

- 答案正是"AJAX"。
- AJAX最吸引人的就是它的"异步"特性，也就是说它可以在不重新刷新页面的情况下与服务器通信，交换数据，或更新页面。
你可以使用AJAX最主要的两个特性做下列事：

- 在不重新加载页面的情况下发送请求给服务器；
- 接受并使用从服务器发来的数据。

## 网页的渲染过程－服务器端渲染

②查询数据 ①请求页面 服务器 ③返回数据 组装HTML片段返回 浏览器 数据库 返回HTML片段 ④传递数据染HTML片段 模板

## 网页的渲染过程一前后端分离

①请求页面 ③请求JavaScript脚本 前端服务器 查询数据库 ②返回静态HTML页面 ④返回JavaScript脚本 渲染页面 浏览器 9返回数据 后端服务器 ⑧返回数据库数据 数据库 执行JavaScript脚本 请求数据 JS JS脚本

## 什么是HTTP？

！什么是HTTP呢？我们来看一下维基百科的解释：

- 超文本传输协议(英语：HyperTextTransferProtocol，缩写：HTTP)是一种用于分布式、协作式和超媒体信息系统的应用层协议;
- HTTP是万维网的数据通信的基础，设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法；
- 通过HTTP或者HTTPS协议请求的资源由统一资源标识符(UniformResourceIdentifiers，URI)来标识;
```javascript
HTTP是一个客户端(用户)和服务端(网站)之间请求和响应的标准。
```

- 通过使用网页浏览器、网络爬虫或者其它的工具，客户端发起一个HTTP请求到服务器上指定端口(默认端口为80)；
```javascript
我们称这个客户端为用户代理程序(useragent)；
```

- 响应的服务器上存储着一些资源，比如HTML文件和图像。
```javascript
(我们称这个响应服务器为源服务器(originserver)；
```

HTTPRequest HTTPResponse Client Server Fig:HTTPProtocol

## 网页中资源的获取

我们网页中的资源通常是被放在Web资源服务器中，由浏览器自动发送HTTP请求来获取、解析、展示的。 GET layout.css WebAPIs Image GETimaqe.png Webserver HTML CSS TheInternet GET page.html The Web GET video.mp4 JavaScript Video Ads GET ads.jpg HTTP Webdocument DNS TLS Ads server Videoserver TCP UDP IP 目前我们页面中很多数据是动态展示的：

- 比如页面中的数据展示、搜索数据、表单验证等等，也是通过在JavaScript中发送HTTP请求获取的；

## HTTP的组成

```javascript
一次HTTP请求主要包括：请求(Request)和响应(Response)
```

Server

```javascript
(Web server)
```

HTTP Request Message Client

```javascript
(Web browser)
```

HTTP Response Message 方法 URI 协议版本 状态码的原因短语 协议版本 状态码

```javascript
POST / form/entry HTTP/1.1 请求首部字段 响应首部字段
```

HTTP/1.1 OK Host: hackr.jp Date: Tue， 10 Jul 2012 06:50:15 GMT Connection: keep-alive Content-Length: 362 Content-Type: application/x-www-form-urlencoded Content-Type: text/html Content-Length: 16

```javascript
name=ueno&age=37 <html>
```

内容实体 主体

## HTTP的版本

1 HTTP/0.9

- 发布于1991年;
- 只支持GET请求方法获取文本数据，当时主要是为了获取HTML页面内容；
HTTP/1.0

- 发布于1996年;
- 支持POST、HEAD等请求方法，支持请求头、响应头等，支持更多种数据类型(不再局限于文本数据)；
- 但是浏览器的每次请求都需要与服务器建立一个TCP连接，请求处理完成后立即断开TCP连接，每次建立连接增加了性能损耗，
```javascript
HTTP/1.1(目前使用最广泛的版本)
```

- 发布于1997年;
- 增加了PUT、DELETE等请求方法;
2015年，HTTP/2.0 12018年，HTTP/3.0

## HTTP的请求方式

■在RFC中定义了一组请求方式，来表示要对给定资源执行的操作：

- GET：GET方法请求一个指定资源的表示形式，使用GET的请求应该只被用于获取数据。
- HEAD：HEAD方法请求一个与GET请求的响应相同的响应，但没有响应体。
```javascript
√比如在准备下载一个文件前，先获取文件的大小，再决定是否进行下载;
```

- POST：POST方法用于将实体提交到指定的资源。
- PUT：PUT方法用请求有效载荷(payload)替换目标资源的所有当前表示；
- DELETE：DELETE方法删除指定的资源;
- PATCH：PATCH方法用于对资源应部分修改；
- TRACE：TRACE方法沿看到目标资源的路径执行一个消息环回测试。
在开发中使用最多的是GET、POST请求；

- 在后续的后台管理项目中，我们也会使用PATCH、DELETE请求；

## HTTP Request Header 一

在request对象的header中也包含很多有用的信息，客户端会默认传递过来一些信息： Request Headers Accept: */* Accept-Encoding: gzip, deflate

```javascript
Accept-Language: zh-CN,zh;q=0.9
```

Access-Control-Request-Headers: token Access-Control-Request-Method: POST Connection: keep-alive Host: 192.168.0.110:1888 Origin: http://127.0.0.1:5500 Referer: http://127.0.0.1:5500/ Sec-Fetch-Mode: cors

```javascript
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36
```

content-type是这次请求携带的数据的类型：

```javascript
application/x-www-form-urlencoded：表示数据被编码成以'&'分隔的键－值对，同时以'='分隔键和值
```

- application/json：表示是一个json类型;
- text/plain：表示是文本类型；
- application/xml：表示是xml类型;
- multipart/form-data：表示是上传文件；

## HTTP Request Header (二)

1content-length：文件的大小长度 keep-alive:

- http是基于TCP协议的，但是通常在进行一次请求和响应结束后会立刻中断;
- 在http1.0中，如果想要继续保持连接：
```javascript
√浏览器需要在请求头中添加connection:keep-alive;
服务器需要在响应头中添加connection:keey-alive;
```

√当客户端再次放请求时，就会使用同一个连接，直接一方中断连接；

- 在http1.1中，所有连接默认是connection:keep-alive的;
```javascript
√不同的Web服务器会有不同的保持keep-alive的时间;
√Node中默认是5s中;
1accept-encoding：告知服务器，客户端支持的文件压缩格式，比如js文件可以使用gzip编码，对应.gz文件;
```

accept：告知服务器，客户端可接受文件的格式类型； user-agent：客户端相关的信息；

## HTTPResponse响应状态码

```javascript
1Http状态码(HttpStatusCode)是用来表示Http响应状态的数字代码：
```

- Http状态码非常多，可以根据不同的情况，给客户端返回不同的状态码；
MDN响应码解析地址：https://developer.mozilla.org/zh-CN/docs/web/http/status 常见HTTP状态码 状态描述 信息说明 OK 客户端请求成功 Created POST请求，创建新的资源 Moved Permanently 请求资源的URL已经修改，响应中会给出新的URL Bad Request 客户端的错误，服务器无法或者不进行处理 Unauthorized 未授权的错误，必须携带请求的身份信息

```javascript
Forbidden 客户端没有权限访问，被拒接
```

Not Found 服务器找不到请求的资源。 Internal Server Error 服务器遇到了不知道如何处理的情况。 Service Unavailable 服务器不可用，可能处理维护或者重载状态，暂时无法访问

## HTTP Request Header

响应的header中包括一些服务器给客户端的信息： Response Headers Viewsource Access-Control-Allow-Origin: http: //127.0.0.1:5500 Connection: keep-alive Content-Length: 87

```javascript
Content-Type: application/json; charset=utf-8
```

Date: Sat, 18 Jun 2022 12:38:42 GMT

```javascript
Keep-Alive: timeout=5
Vary: Origin
```

## Chrome安装插件-FeHelper

1为了之后查看数据更加的便捷、优雅，我们安装一个chrome插件：

- 方式一：可以直接通过Chrome的扩展商店安装；
- 方式二：手动安装
V下载t地址:https://github.com/zxlie/FeHelper/tree/master/apps/static/screenshot/crx

```javascript
FeHelper(前端助手)2020.5.2810
```

JSON自动格式化、手动格式化，支持排序、解 码、下载等，更多功能可在配置页按需安装！ ID: jmdjmgjdimbbfhkmeaebpbibneafoocb 查看视图background/index.html 详情 移除 错误

## AJAX发送请求

```javascript
AJAX 是异步的 JavaScript 和 XML (Asynchronous JavaScript And XML)
```

- 它可以使用JSON，XML，HTML和text文本等格式发送和接收数据;
如何来完成AJAX请求呢？

- 第一步：创建网络请求的AJAx对象(使用xMLHttpRequest)
- 第二步：监听xMLHttpRequest对象状态的变化，或者监听onload事件(请求完成时触发);
- 第三步：配置网络请求(通过open方法)；
- 第四步：发送send网络请求；
- /·1.创建XMLHttpReqest对象
```javascript
const xhr = new XMLHttpRequest()
```

1/·2.监听对象状态的改变

```javascript
xhr.onreadystatechange = function() {
console.log("状态发生了改变")
```

- /·3.配置请求的方式/URL
```javascript
xhr.0pen("get",:"http://192.168.0.110:1888/01_basic/hello_text")
```

- /·4.发送请求
```javascript
xhr.send()
```

## XMLHttpRequest的state (状态)

事实上，我们在一次网络请求中看到状态发生了很多次变化，这是因为对于一次请求来说包括如下的状态： 值 状态 描述

```javascript
UNSENT 代理被创建，但尚未调用 open()方法。
```

OPENED open(方法已经被调用。 HEADERS_RECEIVED send(方法已经被调用，并且头部和状态已经可获得。 LOADING 下载中；responseText属性已经包含部分数据。 DONE 下载操作已完成。 注意：这个状态并非是HTTP的相应状态，而是记录的XMLHttpRequest对象的状态变化。

- http响应状态通过status获取;
发送同步请求：

- 将open的第三个参数设置为false
- ·3.配置请求的方式/URL
```javascript
xhr.0pen("get",:"http://192.168.0.110:1888/01_basic/hello_text", false)
```

## XMLHttpRequest其他事件监听

除了onreadystatechange还有其他的事件可以监听

- loadstart：请求开始。
- progress：一个响应数据包到达，此时整个 response body 都在 response 中。
- abort：调用xhr.abortO取消了请求。
- error：发生连接错误，例如，域错误。不会发生诸如 404 这类的HTTP 错误。
- load：请求成功完成。
- timeout：由于请求超时而取消了该请求(仅发生在设置了timeout的情况下)。
loadend：在load，error，timeout或 abort 之后触发。 我们也可以使用load来获取数据：

```javascript
xhr.onload = function() {
console.log(xhr.response)
```

## 响应数据和响应类型

1发送了请求后，我们需要获取对应的结果：response属性

- XMLHttpRequestresponse属性返回响应的正文内容；
- 返回的类型取决于responseType的属性设置；
通过responseType可以设置获取数据的类型

- 如果将responseType的值设置为空字符串，则会使用text作为默认值。
```javascript
xhr.responseType-=:"json"
```

和responseText、responseXML的区别:

- 早期通常服务器返回的数据是普通的文本和XML，所以我们通常会通过responseText、responseXML来获取响应结果;
- 之后将它们转化成JavaScript对象形式；
- 目前服务器基本返回的都是json数据，直接设置为json即可；

## HTTP响应的状态status

XMLHttpRequest的state是用于记录xhr对象本身的状态变化，并非针对于HTTP的网络请求状态。 如果我们希望获取HTTP响应的网络状态，可以通过status和statusText来获取：

```javascript
console.log(xhr.status)
console.log(xhr.statusText)
```

常见HTTP状态码 状态描述 信息说明 OK 客户端请求成功 Created POST请求，创建新的资源 Moved Permanently 请求资源的URL已经修改，响应中会给出新的URL Bad Request 客户端的错误，服务器无法或者不进行处理 Unauthorized 未授权的错误，必须携带请求的身份信息

```javascript
Forbidden 客户端没有权限访问，被拒接
```

Not Found 服务器找不到请求的资源。 Internal Server Error 服务器遇到了不知道如何处理的情况。 Service Unavailable 服务器不可用，可能处理维护或者重载状态，暂时无法访问

## GET/POST请求传递参数

1在开发中，我们使用最多的是GET和POST请求，在发送请求的过程中，我们也可以传递给服务器数据。 常见的传递给服务器数据的方式有如下几种：

- 方式一：GET请求的query参数
- 方式二：POST请求x-www-form-urlencoded格式
- 方式三：POST请求FormData格式
- 方式四：POST请求JSON格式
- /·1.get请求传递参数 1.3.post请求传递参数(json)
```javascript
xhr.open("get",:"http://192.168.0.110:1888/01_param/get?name=why&age=18" xhr.open("post", "http://192.168.0.110:1888/02_param/postjson")
xhr.send()
xhr.setRequestHeader('Content-type', 'application/json; charset=utf-8')
xhr.send(jsonParam)
```

- /·2.post请求传递参数(form)
```javascript
xhr.open("post", "http://192.168.0.110:1888/02_param/postform") // 4.post请求(urLencoded)
const infoEl = document.querySelector(".info") xhr.0pen("post",."http://192.168.0.110:1888/02_param/postur1")
const form = new FormData(infoEl) const urlParam = "name=why&age=18"
form.append("height", 1.88)
xhr.send(form) xhr.send(urlParam)
```

## ajax网络请求封装

- /·1.创建xhr对象
```javascript
const xhr =· new-XMLHttpRequest()
xhr.onreadystatechange = function(){
if-(xhr.readyState !==·XMLHttpRequest.DoNE)·return
if (xhr.status >= 200 && xhr.status-< 300-ll·xhr.status === 304)·{
success && success(xhr.response)
else {
failure && failure(xhr.response)
```

- /·2.设置响应的类型
```javascript
xhr.responseType = "json"
```

- /·3.发送请求
```javascript
const paramsString = params.join("&")
```

设置header

```javascript
if (method.toLowerCase() === "get") {
xhr.open(method, url +"?" + paramsString)
Object.keys(headers) .forEach(headerKey => xhr.setRequestHeader(headerKey, -headers[headerKey]))
xhr.send()
} else {
xhr.open(method, url)
xhr.setRequestHeader('Content-type',·'application/x-www-form-urlencoded')
Object.keys(headers).forEach(headerKey => xhr.setRequestHeader(headerKey, headers[headerKey]))
xhr.send(paramsString)
return xhr
```

## 延迟时间timeout和取消请求

1在网络请求的过程中，为了避免过长的时间服务器无法返回数据，通常我们会为请求设置一个超时时间：timeout。

- 当达到超时时间后依然没有获取到数据，那么这个请求会自动被取消掉；
- 默认值为0，表示没有设置超时时间；
我们也可以通过abort方法强制取消请求；

```javascript
const xhr = hyajax({
```

url::"http://192.168.0.110:1888/01_basic/timeout" timeout::10000,

```javascript
success:·(res) => {
console.log("success:", res)
failure:·(err) => {
console.log("error:", err)
const cancelBtn = document.querySelector(".cancel")
cancelBtn.onclick = function() {
xhr.abort()
```

## 认识Fetch和Fetch APl

Fetch可以看做是早期的XMLHttpRequest的替代方案，它提供了一种更加现代的处理方案：

- 比如返回值是一个Promise，提供了一种更加优雅的处理结果方式
```javascript
√在请求发送成功时，调用resolve回调then;
√在请求发送失败时，调用reject回调catch;
```

- 比如不像XMLHttpRequest一样，所有的操作都在一个对象上;
```javascript
Ifetch函数的使用：
Promise<Response> fetch(input[, init]);
```

- input：定义要获取的资源地址，可以是一个URL字符串，也可以使用一个Request对象(实验性特性)类型;
- init：其他初始化参数
```javascript
method:请求使用的方法，如GET、POST;
headers:请求的头信息;
```

- body:请求的body信息;

## Fetch数据的响应(Response)

1Fetch的数据响应主要分为两个阶段：

```javascript
阶段一：当服务器返回了响应(response)
```

- fetch返回的promise就使用内建的Responseclass对象来对响应头进行解析；
- 在这个阶段，我们可以通过检查响应头，来检查HTTP状态以确定请求是否成功;
- 如果fetch 无法建立一个 HTTP 请求，例如网络问题，亦或是请求的网址不存在，那么 promise 就会reject;
- 异常的HTTP状态，例如404或500，不会导致出现error;
我们可以在response的属性中看到HTTP状态：

- status：HTTP 状态码，例如 200;
- ok：布尔值，如果HTTP状态码为 200-299，则为true;
1第二阶段，为了获取responsebody，我们需要使用一个其他的方法调用。

```javascript
response.text()一一读取 response，并以文本形式返回 response;
response.json() 一一将 response 解析为 JsON ;
```

## Fetch网络请求的演练

基于Promise的使用方案：

- /·发送fetch请求
```javascript
fetch("http://192.168.0.110:1888/03_project/products").then(res => {
return res.json()
}).then(res => {
console.log("res:", res)
})
```

基于async、await的使用方案：

- /·发送fetch请求
```javascript
async function getData()·{
const·data =- await·response.json()
console.log("data:", data)
getData()
```

## FetchPOST请求

创建一个POST请求，或者其他方法的请求，我们需要使用fetch选项： method：HTTP 方法，例如 POST, body：request body，其中之一:

```javascript
√字符串(例如JSON编码的)，
```

√FormData对象，以multipart/form-data形式发送数据，

```javascript
async function getData() {
const response = await fetch("http://192.168.0.110:1888/02_param/posturl", {
```

method::"post"

```javascript
headers:-{
```

"Content-Type":~"application/x-www-form-urlencoded"

```javascript
body: "name=why&age=18"
const data = await response.json()
console.log("data:", data)
```

## XMLHttpRequest文件上传

！文件上传是开发中经常遇到的需求，比如头像上传、照片等。

- 要想真正理解文件上传，必须了解服务器如何处理上传的文件信息；
```javascript
uploadBtn.onclick = function() {
```

- /1.获取文件
```javascript
const files = fileEl.files
if (!files.length) {
选择文件未选择任何文件 alert("请选择要上传的文件")
上传 return
const avatarFile = files[0]
const fileForm = new FormData()
fileForm.append("avatar", avatarFile)
<form-class="fileform">
<input class="file" type="file"> //·2.开始上传
</form> const xhr = new XMLHttpRequest()
xhr.upload.onprogress = function(event) {
<button class="upload">上传</button>
console.log(^${event.loaded}/${event.total})
<script> xhr.onload = function() {
const fileEl = document.querySelector(".file") console.log(xhr.response)
const uploadBtn = document.querySelector(".upload")
xhr.open("P0sT", "http://192.168.0.110:1888/02_param/upload")
xhr.setRequestHeader("Content-Type", "multipart/form-data")
xhr.send(fileForm)
```

## Fetch文件上传

1Fetch也支持文件上传，但是Fetch没办法监听进度。

```javascript
const avatarFile = files[0]
const fileForm-= new FormData()
fileForm.append("avatar", avatarFile)
```

- /~2.开始上传
```javascript
fetch("http://192.168.0.110:1888/02_param/upload", {
```

method:-"post", body: fileForm

```javascript
}).then(res => {
return- res.json()
}).then(res => {
console.log("res:", res)
})
```
