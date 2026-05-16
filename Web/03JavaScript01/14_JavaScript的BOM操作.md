JavaScript的BOM操作

目录 认识BOM操作 content 全局对象window 事件对象event location、 history navigator、 screen JSON、 Storage

## 认识BOM

```javascript
BOM：浏览器对象模型(BrowserObjectModel)
```

window

- 简称 BOM，由浏览器提供的用于处理文档(document)之外的所有内容的其他对象;
DOM BOM JavaScript

```javascript
比如navigator、location、history等对象;
```

document navigator Object 1JavaScript有一个非常重要的运行环境就是浏览器 screen Array

- 而且浏览器本身又作为一个应用程序需要对其本身进行操作; location Function
- 所以通常浏览器会有对应的对象模型(BOM，BrowserObjectModel); frames
- 我们可以将BOM看成是连接JavaScript脚本与浏览器窗口的桥梁； history
BOM主要包括一下的对象模型： XMLHttpRequest

- window：包括全局属性、方法，控制浏览器窗口相关的属性、方法；
- location：浏览器连接到的对象的位置(URL);
- history：操作浏览器的历史；
- navigator：用户代理(浏览器)的状态和标识(很少用到)；
- screen：屏幕窗口信息(很少用到)；

## window对象

1window对象在浏览器中可以从两个视角来看待：

- 视角一：全局对象。
```javascript
√我们知道ECMAScript其实是有一个全局对象的，这个全局对象在Node中是global;
```

√在浏览器中就是window对象；

- 视角二：浏览器窗口对象。
```javascript
√作为浏览器窗口时，提供了对浏览器操作的相关的API;
```

1当然，这两个视角存在大量重叠的地方，所以不需要刻意去区分它们：

- 事实上对于浏览器和Node中全局对象名称不一样的情况，目前已经指定了对应的标准，称之为globalThis，并且大多数现代
浏览器都支持它；

- 放在window对象上的所有属性都可以被访问；
- 使用var定义的变量会被添加到window对象中；
- window默认给我们提供了全局的函数和类：setTimeout、Math、Date、Object等；

## window对象的作用

事实上window对象上肩负的重担是非常大的：

- 第一：包含大量的属性，localStorage、console、location、history、screenX、scrollx等等(大概6O+个属性)；
- 第二：包含大量的方法，alert、close、scrolITo、open等等(大概40+个方法);
- 第三：包含大量的事件，focus、blur、load、hashchange等等(大概30+个事件);
- 第四：包含从EventTarget继承过来的方法，addEventListener、removeEventListener、dispatchEvent方法;
那么这些大量的属性、方法、事件在哪里查看呢？ MDN文档:https://developer.mozilla.org/zh-CN/docs/Web/APl/Window 1查看MDN文档时，我们会发现有很多不同的符号，这里我解释一下是什么意思：

- 删除符号：表示这个API已经废弃，不推荐继续使用了；
- 点踩符号：表示这个API不属于W3C规范，某些浏览器有实现(所以兼容性的问题)；
- 实验符号：该API是实验性特性，以后可能会修改，并且存在兼容性问题；

## window常见的属性

我们来看一下常见的window属性： 浏览器高度

```javascript
console.log(window.outerHeight)
console.log(window.innerHeight)
console.log("screenx:", window.screenx)
console.log("screenY: ", window.screenY)
```

监听

```javascript
window.addEventListener("scroll",·(event) => {
console.log(window.scrolly)
console.log(window.scrollx)
})
```

## window常见的方法

我们来看一下常见的window方法：

- /·close方法
```javascript
const closeBtn = document.querySelector("#close")
closeBtn.onclick = function() {
close()
```

scrollTo

```javascript
const scrollBtn = document.querySelector("#scroll")
scrollBtn.onclick = function() {
scrollTo({ top: 10o0 })
```

- /·打开新创建
```javascript
const openBtn = document.querySelector("#open")
openBtn.onclick = function() {
open("./about.html",."_self")
```

## window常见的事件

我们来看一下常见的window事件：

```javascript
window.onfocus = function()·{
console.log("窗口获取到焦点")
window.onblur = function() {
console.log("窗口失去了焦点")
```

- /·整个页面以及所有的资源都加载完成
```javascript
window.onload = function() {
console.log("页面加载完成")
```

- /·hash改变
```javascript
const hashBtn = document.querySelector("#hash")
hashBtn.onclick = function() {
location.hash =."aaa"
window.onhashchange = function() ·
console.log("hash被修改了")
```

## location对象常见的属性

location对象用于表示window上当前链接到的URL信息。 常见的属性有哪些呢？

- href:当前window对应的超链接URL，整个URL;
- protocol:当前的协议;
```javascript
host:主机地址;
hostname:主机地址(不带端口);
port:端口;
pathname:路径;
```

- search:查询字符串；
```javascript
hash:哈希值;
```

- username：URL中的username(很多浏览器已经禁用)；
- password：URL中的password(很多浏览器已经禁用)；

## Location对象常见的方法

我们会发现location其实是URL的一个抽象实现： scheme path host query fragment userinfo ? port ■location有如下常用的方法：

- assign：赋值一个新的URL，并且跳转到该URL中;
```javascript
replace：打开一个新的URL，并且跳转到该URL中(不同的是不会在浏览记录中留下之前的记录)；
```

- reload：重新加载页面，可以传入一个Boolean类型；
```javascript
const locationBtn = document.querySelector("#location")
locationBtn.onclick = function() {
location.assign("http: / /www.baidu.com")
location.replace("http: / /www.baidu.com")
location.reload()
```

## URLSearchParams

1URLSearchParams定义了一些实用的方法来处理URL的查询字符串。

- 可以将一个字符串转化成URLSearchParams类型；
- 也可以将一个URLSearchParams类型转成字符串；
```javascript
var urlsearch = new URLSearchParams("name=why&age=18&height=1.88")
console.log(urlsearch.get("name"))·// why
```

URLSearchParams常见的方法有如下：

- get：获取搜索参数的值;
- set：设置一个搜索参数和值;
- append：追加一个搜索参数和值;
- has：判断是否有某个搜索参数；
https://developer.mozilla.org/zh-CN/docs/Web/APl/URLSearchParams 中文会使用encodeURiComponent和decodeURlComponent进行编码和解码

## history对象常见属性和方法

1history对象允许我们访问浏览器曾经的会话历史记录。

```javascript
1有两个属性： console.log(history.length)
console.log(history.state)
```

- length：会话中的记录条数；
- state：当前保留的状态值; const jumpBtn·= document.querySelector("#jump")
```javascript
const·backBtn =document.querySelector("#back")
```

有五个方法：

```javascript
jumpBtn.onclick =:function()°{
back)：返回上一页，等价于history.go(-1); history.pushstate({name:."why"},."11",."aaa")
console.log(history.length,·history.state)
forward(：前进下一页，等价于history.go(1);
go(：加载历史中的某一页;
backBtn.onclick = function() {
```

- pushState(：打开一个指定的地址; history.back()
```javascript
console.log(history.length, history.state)
replaceState(：打开一个新的地址，并且使用replace;
```

Ihistory和hash目前是vue、react等框架实现路由的底层原理，具体的实现方式我会在后续讲解。

## navigator对象(很少使用)

## ■navigator对象表示用户代理的状态和标识等信息。

属性/方法 说明 locks 返回暴露WebLocks API的LockManager对象 mediaCapabilities 返回暴露MediaCapabilitiesAPI的MediaCapabilities对象 mediaDevices 返回可用的媒体设备 maxTouchPoints 返回设备触摸屏支持的最大触点数 mimeTypes 返回浏览器中注册的MIME类型数组 onLine 返回布尔值，表示浏览器是否联网

```javascript
ndoso 返回浏览器运行设备的操作系统和(或)CPU
```

permissions 返回暴露PermissionsAPI的Permissions对象 platform 返回浏览器运行的系统平台

```javascript
plugins 返回浏览器安装的插件数组。在IE中，这个数组包含页面中所有<embed>元素
product 返回产品名称(通常是"Gecko")
productSub 返回产品的额外信息(通常是Gecko的版本)
registerProtocolHandler() 将一个网站注册为特定协议的处理程序
requestMediaKeySystemAccess()返回一个期约，解决为MediaKeySystemAccess对象
sendBeacon() 异步传输一些小数据
```

serviceWorker 返回用来与ServiceWorker实例交互的ServiceWorkerContainer

```javascript
share () 返回当前平台的原生共亨机制
```

storage 返回暴露StorageAPI的StorageManager对象 userAgent 返回浏览器的用户代理字符串 vendor 返回浏览器的厂商名称 vendorSub 返回浏览器厂商的更多信息

```javascript
vibrate() 触发设备振动
```

webdriver 返回浏览器当前是否被自动化程序控制

## screen对象 (很少使用)

screen主要记录的是浏览器窗口外面的客户端显示器的信息：

- 比如屏幕的逻辑像素 screen.width、screen.height;
属性 说明

```javascript
availHeight 屏幕像素高度减去系统组件高度(只读)
availLeft 没有被系统组件占用的屏幕的最左侧像素(只读)
availTop 没有被系统组件占用的屏幕的最顶端像素(只读)
availwidth 屏幕像素宽度减去系统组件宽度(只读)
colorDepth 表示屏幕颜色的位数：多数系统是32(只读)
```

height 屏幕像素高度 left 当前屏幕左边的像素距离

```javascript
pixelDepth 屏幕的位深(只读)
```

top 当前屏幕顶端的像素距离 width 屏幕像素宽度 orientation 返回ScreenOrientationAPI中屏幕的朝向

## JSON的由来

在目前的开发中，JSON是一种非常重要的数据格式，它并不是编程语言，而是一种可以在服务器和客户端之间传输的数据格式。

```javascript
JSON的全称是JavaScriptObjectNotation(JavaScript对象符号)：
```

- JSON是由DouglasCrockford构想和设计的一种轻量级资料交换格式，算是JavaScript的一个子集；
- 但是虽然JSON被提出来的时候是主要应用JavaScript中，但是目前已经独立于编程语言，可以在各个编程语言中使用;
其他的传输格式：

```javascript
少在被使用了;
```

- Protobuf：另外一个在网络传输中目前已经越来越多使用的传输格式是protobuf，但是直到2021年的3.x版本才支持JavaScript，所
以目前在前端使用的较少； 目前JSON被使用的场景也越来越多：

- 网络数据的传输JSON数据
- 项目的某些配置文件;
- 非关系型数据库(NoSQL)将json作为存储格式;

## 小程序的app.json

```javascript
{ app.json X
```

M - app.json ) .. "pages": [ "pages/index/index", "pages/logs/logs" J..MopuTM.. "backgroundTextStyle":"light", "navigationBarBackgroundColor": "#fff"" "navigationBarTitleText": "Weixin", "navigationBarTextStyle":"black"" "style": "v2", "sitemapLocation": "sitemap. json"

## JSON基本语法

JSON的顶层支持三种类型的值：

- 简单值：数字(Number)、字符串(String，不支持单引l号)、布尔类型(Boolean)、null类型;
- 对象值：由key、value组成，key是字符串类型，并且必须添加双引l号，值可以是简单值、对象值、数组值;
- 数组值：数组的值可以是简单值、对象值、数组值；
"name":."why" 123, "age":·18, "abc"

```javascript
"friend": {
```

"name":。"kobe" "name":"kobe"

## JSON序列化

某些情况下我们希望将JavaScript中的复杂类型转化成JSON格式的字符串，这样方便对其进行处理：

- 比如我们希望将一个对象保存到localStorage中；
- 但是如果我们直接存放一个对象，这个对象会被转化成［objectObject]格式的字符串，并不是我们想要的结果；
```javascript
const obj= {
```

name: "why", age:·18,

```javascript
friend: {
```

name:."kobe" hobbies：·["篮球"，"足球"，"乒乓球"] Key Value info [object Object]

## JSON序列化方法

在ES5中引I用了JSON全局对象，该对象有两个常用的方法：

- stringify方法：将JavaScript类型转成对应的JSON字符串;
```javascript
parse方法：解析JSON字符串，转回对应的JavaScript类型;
```

那么上面的代码我们可以通过如下的方法来使用：

- /·转成字符串保存
```javascript
const objString =·JsoN.stringify(obj)
localstorage.setItem("info", objString)
```

获取字符串转回对象

```javascript
const itemString = localstorage.getItem("info")
const info = JsoN.parse(itemString)
console.log(info)
```

## Stringify的参数replace

## ■JSON.stringifyO方法将一个JavaScript对象或值转换为JSON字符串：

- 如果指定了一个replacer函数，则可以选择性地替换值；
- 如果指定的replacer是数组，则可选择性地仅包含数组指定的属性；
- 转成字符串
```javascript
const objStringl = JsoN.stringify(obj)
```

- /·{"name":"why","age":18,"friend":["name":"kobe"},"hobbies"：["篮球","足球","乒乓球"]}
```javascript
console.log(objStringl)
```

- /·replace参数是一个数组
```javascript
const objString2 = JsoN.stringify(obj,·["name", "age"])
```

- /["name":"why","age":18}
```javascript
console.log(objString2)
```

- replace参数是一个函数
```javascript
console. log(key, value)
if·(key ===-"name")·{
return "coderwhy"
return value
["name":"coderwhy"，"age"：18，"friend"：["name"："coderwhy"}，"hobbies"：["篮球"，"足球"，"乒乓球"]}
console.log(objString3)
```

## Stringify的参数space

如果对象本身包含toJSON方法，那么会直接使用toJSON方法的结果：

```javascript
const obj = {
```

name: "why", age: 18, friend:. hobbies：·["篮球"，"足球"，"乒乓球"]，

```javascript
toJsoN:°function()·{
return."coderwhy"
const objString5 = JsoN.stringify(obj)
console.log(objString5)
```

## parse方法

JSON.parseO方法用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。

- 提供可选的reviver 函数用以在返回之前对所得到的对象执行变换(操作)。
转回对象，·并且转换某些值

```javascript
const info2 = JsoN.parse(objString,°(key, value) => {
if·(key ===·"time")·{
return·new·Date(value)
return value
console.log(info2)
```

JSON的方法可以帮我们实现对象的深拷贝：

- 但是目前我们还没有了解什么是对象的拷贝、浅拷贝、深拷贝的概念；
- 我们会在JavaScript高级中学习；

## 认识storage

### 1WebStorage主要提供了一种机制，可以让浏览器提供一种比cookie更直观的key、value存储方式：

- localStorage：本地存储，提供的是一种永久性的存储方法，在关闭掉网页重新打开时，存储的内容依然保留；
- sessionStorage：会话存储，提供的是本次会话的存储，在关闭掉会话时，存储的内容会被清除；
```javascript
localstorage.setItem("name",."localstorage")
sessionStorage.setItem("name",."sessionStorage")
```

曰 Elements Console Sources Network Performance Memory Application Security Lighthouse Redux Application Filter Manifest Key Value ServiceWorkers name localStorage Storage Storage LocalStorage http://127.0.0.1:8080 ESessionStorage http://127.0.0.1:8080 IndexedDB WebSQL Cookies

## localstorage和sessionStorage的区别

我们会发现localStorage和sessionStorage看起来非常的相似。 那么它们有什么区别呢？

- 验证一：关闭网页后重新打开，localStorage会保留，而sessionStorage会被删除；
- 验证二：在页面内实现跳转，localStorage会保留，sessionStorage也会保留；
- 验证三：在页面外实现跳转(打开新的网页)，localStorage会保留，sessionStorage不会被保留；

## Storage常见的方法和属性

IStorage有如下的属性和方法： 属性：

- Storage.length：只读属性
√返回一个整数，表示存储在Storage对象中的数据项数量； ■方法：

- Storage.keyO：该方法接受一个数值n作为参数，返回存储中的第n个key名称；
- Storage.getltem(：该方法接受一个key作为参数，并且返回key对应的value;
- Storage.setltemO：该方法接受一个key和value，并且将会把key和value添加到存储中。
- 如果key存储，则更新其对应的值；
- Storage.removeltem(：该方法接受一个key作为参数，并把该key从存储中删除;
- Storage.clear(：该方法的作用是清空存储中的所有key;
