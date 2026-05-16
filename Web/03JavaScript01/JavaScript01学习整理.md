# JavaScript01 学习整理

这份笔记基于 `JavaScript01` 目录下的 15 份 Markdown 整理而成，目标不是机械合并，而是按学习顺序归纳成一份更适合复习和建立知识体系的总笔记。

## 学习路线

建议按下面的顺序学习：

1. 先知道 JavaScript 是什么、能做什么
2. 再掌握基础语法、变量、数据类型、运算符
3. 接着学习分支、循环、函数
4. 然后进入对象、数组、字符串、日期等内置能力
5. 再学习 DOM、事件、BOM
6. 最后通过定时器、轮播图、购物车等案例把知识串起来

## 原始资料对应关系

| 主题 | 原文件 |
|---|---|
| JavaScript 入门认知 | `01_邂逅JavaScript.md` |
| 基础语法 | `02_JavaScript基础语法.md` |
| 变量与数据类型 | `03_JavaScript变量和数据类型.md` |
| 运算符 | `04_JavaScript常见的运算符.md` |
| 分支与逻辑运算 | `05_JavaScript分支语句和逻辑运算符.md` |
| 循环 | `06_JavaScript的while和for循环.md` |
| 函数 | `07_JavaScript函数的使用.md` |
| 对象与面向对象基础 | `08_JavaScript的面向对象.md` |
| 事件处理 | `08_JavaScript的事件监听.md`、`12_JavaScript的事件监听.md` |
| 内置类 | `09_JavaScript的内置类.md` |
| DOM 操作 | `10_JavaScript的DOM操作（一）.md`、`11_JavaScript的DOM操作（二）.md` |
| DOM/BOM 实战案例 | `13_JavaScript-DOM实战案例.md` |
| BOM/JSON/Storage | `14_JavaScript的BOM操作.md` |

## 一、认识 JavaScript

应用场景：

- 用来建立前端三件套的整体认知，适合刚开始做网页和交互页面时快速分清 JavaScript 负责什么。
- 适合按钮点击、弹窗提示、表单校验、页面联动这类入门练习，先理解它在项目里的职责范围。

### 1. JavaScript 在前端里的位置

前端最核心的三部分：

- `HTML` 负责结构
- `CSS` 负责样式
- `JavaScript` 负责行为和交互

可以把它理解为：

- `HTML` 搭骨架
- `CSS` 做外观
- `JavaScript` 让页面“动起来”

### 2. JavaScript 能做什么

- 操作网页内容
- 响应用户点击、输入、滚动等行为
- 与浏览器能力交互
- 处理数据、校验表单、发送请求
- 在 Node.js 环境中做服务端开发

最小示例：

```js
var titleEl = document.querySelector("h1")

titleEl.textContent = "欢迎学习 JavaScript"

titleEl.addEventListener("click", function() {
  alert("你点击了标题")
})
```

### 3. JavaScript 的组成

这套资料里多次提到 JavaScript 的几部分内容：

- `ECMAScript`：语言本身的语法规范
- `DOM`：操作页面元素
- `BOM`：操作浏览器窗口和浏览器相关对象

学习时可以把它们拆开理解：

- 前期先学 `ECMAScript`
- 中期学 `DOM` 和事件
- 后期补 `BOM`、`JSON`、`Storage`

## 二、基础语法

应用场景：

- 用在编写第一段脚本、调试页面逻辑、验证功能是否生效时。
- 适合做小 demo、课堂练习、静态页加少量交互时建立基础编码习惯。

### 1. JavaScript 的三种编写方式

- 行内写法：不推荐
- `script` 标签中直接写：适合演示
- 外部 `.js` 文件：正式开发最常用

推荐做法：

```html
<script src="./main.js"></script>
```

### 2. 基本交互方式

常见交互函数：

- `alert()`：弹窗提示
- `prompt()`：接收用户输入
- `console.log()`：控制台输出，最常用
- `document.write()`：直接写入页面，不推荐在现代项目中常用

```js
alert("欢迎来到 JavaScript 世界")

var name = prompt("请输入你的名字")
console.log("你好：", name)

document.write("<p>页面加载完成</p>")
```

### 3. 基本书写规范

- 语句结尾可以写分号，建议统一风格
- 注释分为单行和多行
- JavaScript 严格区分大小写
- `script` 标签不要写成单标签
- 推荐把脚本放在 `body` 结束前，或使用 `defer`

```js
// 单行注释

/*
  多行注释
*/
```

## 三、变量与数据类型

应用场景：

- 用在保存用户名、价格、数量、状态、搜索关键字这类会变化的数据。
- 适合表单处理、购物车统计、接口数据暂存、页面条件筛选等常见业务场景。

### 1. 变量的作用

变量用来保存会变化的数据，比如：

- 用户名
- 购物车数量
- 商品价格
- 页面状态

```js
var age = 18
var message = "Hello"
```

### 2. 变量命名规则

- 不能以数字开头
- 不能使用关键字
- 建议见名知意
- 常用驼峰命名法：`userName`、`totalPrice`

### 3. 常见数据类型

原始类型：

- `Number`
- `String`
- `Boolean`
- `Undefined`
- `Null`

复杂类型：

- `Object`

补充理解：

- 数组本质上也是对象
- 函数本质上也是对象

### 4. `typeof`

用于查看数据类型：

```js
typeof 123        // "number"
typeof "abc"      // "string"
typeof true       // "boolean"
typeof undefined  // "undefined"
typeof {}         // "object"
```

注意：

- `typeof null` 的结果是 `"object"`，这是历史遗留问题

### 5. 类型转换

常见转换方式：

- 转字符串：`String(value)`
- 转数字：`Number(value)`
- 转布尔：`Boolean(value)`

```js
String(123)      // "123"
Number("18")     // 18
Boolean("")      // false
Boolean("abc")   // true
```

常见“假值”：

- `0`
- `""`
- `null`
- `undefined`
- `NaN`
- `false`

## 四、运算符

应用场景：

- 用在金额计算、数量增减、条件比较、状态判断、分页切换等逻辑里。
- 适合购物车总价统计、成绩评级、按钮启用禁用、倒计时这类基础功能。

### 1. 算术运算符

- `+`
- `-`
- `*`
- `/`
- `%`
- `**`

```js
5 % 2   // 1
2 ** 3  // 8
```

### 2. 赋值运算符

- `=`
- `+=`
- `-=`
- `*=`
- `/=`

```js
var count = 10

count += 5   // 15
count -= 2   // 13
count *= 2   // 26
count /= 13  // 2
```

### 3. 自增和自减

- `i++`
- `++i`
- `i--`
- `--i`

理解重点：

- 单独使用时差别不大
- 放到表达式里时，前置和后置结果不同

```js
var a = 5
var b = a++   // b = 5, a = 6

var x = 5
var y = ++x   // y = 6, x = 6
```

### 4. 比较运算符

- `>`
- `<`
- `>=`
- `<=`
- `==`
- `===`
- `!=`
- `!==`

最重要的一条：

- 优先使用 `===` 和 `!==`

原因：

- `==` 会发生隐式类型转换
- `===` 同时比较值和类型，更安全

```js
console.log(1 == "1")   // true
console.log(1 === "1")  // false
```

## 五、分支语句与逻辑运算

应用场景：

- 用在登录状态判断、权限控制、表单校验、订单状态分支、错误提示处理里。
- 适合根据用户输入或业务状态决定页面展示和后续逻辑的场景。

### 1. `if` 结构

```js
if (score >= 60) {
  console.log("及格")
}
```

### 2. `if...else`

```js
if (age >= 18) {
  console.log("成年人")
} else {
  console.log("未成年人")
}
```

### 3. `if...else if...else`

适合多条件判断：

```js
if (score >= 90) {
  console.log("优秀")
} else if (score >= 60) {
  console.log("及格")
} else {
  console.log("不及格")
}
```

### 4. 三元运算符

适合简单判断：

```js
var result = age >= 18 ? "成年" : "未成年"
```

### 5. 逻辑运算符

- `&&`：与
- `||`：或
- `!`：非

常见用途：

- 多条件同时满足
- 提供默认值
- 条件取反

```js
if (userName && password) {
  console.log("可以提交")
}
```

```js
var nickname = ""
var displayName = nickname || "匿名用户"
console.log(displayName) // 匿名用户

var isLogin = true
console.log(!isLogin) // false
```

### 6. `switch`

适合判断同一个表达式的多个固定值：

```js
switch (day) {
  case 1:
    console.log("周一")
    break
  case 2:
    console.log("周二")
    break
  default:
    console.log("其他")
}
```

## 六、循环

应用场景：

- 用在遍历数组、批量生成列表、汇总数据、处理一组 DOM 节点时。
- 适合商品列表渲染、评论输出、成绩统计、批量绑定事件这类重复任务。

### 1. `while`

```js
var i = 0
while (i < 5) {
  console.log(i)
  i++
}
```

适合：

- 不确定循环次数
- 依赖某个条件持续执行

### 2. `do...while`

特点：

- 先执行一次，再判断条件

```js
do {
  console.log("至少执行一次")
} while (false)
```

### 3. `for`

最常用的循环：

```js
for (var i = 0; i < 5; i++) {
  console.log(i)
}
```

### 4. 循环控制

- `break`：终止循环
- `continue`：跳过本次循环

```js
for (var i = 1; i <= 5; i++) {
  if (i === 3) continue
  if (i === 5) break
  console.log(i)
}
```

### 5. 学循环时的重点

- 先看初始值
- 再看条件
- 再看每轮如何变化
- 最后模拟前几次执行过程

## 七、函数

应用场景：

- 用在封装重复逻辑，比如格式化数据、校验表单、切换状态、发请求前处理参数。
- 适合把零散代码整理成可复用能力，后面做模块化和组件化时会非常高频。

### 1. 函数是什么

函数就是把可重复使用的逻辑封装起来，做到：

- 提高复用性
- 降低重复代码
- 让代码更容易理解

### 2. 函数声明和调用

```js
function sayHello() {
  console.log("Hello")
}

sayHello()
```

### 3. 参数与返回值

```js
function add(num1, num2) {
  return num1 + num2
}

var result = add(10, 20)
```

要掌握两个问题：

- 参数解决“函数处理什么输入”
- 返回值解决“函数给我什么结果”

### 4. 局部变量和全局变量

- 函数内部声明的变量通常是局部变量
- 函数外部声明的变量通常是全局变量

建议：

- 少用全局变量
- 优先使用参数和返回值传递数据

```js
var siteName = "Coder" // 全局变量

function printUser() {
  var userName = "Tom" // 局部变量
  console.log(siteName, userName)
}

printUser()
```

### 5. 函数表达式

```js
var sum = function(a, b) {
  return a + b
}
```

### 6. 回调函数

函数可以作为参数传递，这就是回调函数思想的基础：

```js
function exec(fn) {
  fn()
}
```

```js
function greet() {
  console.log("Hello")
}

exec(greet)
```

### 7. 立即执行函数 IIFE

```js
(function() {
  console.log("立即执行")
})()
```

作用：

- 形成独立作用域
- 避免变量污染全局

### 8. 递归

递归本质：

- 函数调用自己

必须具备：

- 重复问题
- 明确终止条件

```js
function factorial(n) {
  if (n <= 1) return 1
  return n * factorial(n - 1)
}
```

## 八、对象与面向对象基础

应用场景：

- 用在描述用户、商品、订单、文章等带有多个属性的数据实体。
- 适合把相关数据和行为组织在一起，减少页面里零散变量和重复方法。

### 1. 对象是什么

对象是“键值对”的集合，用来描述更复杂的数据。

```js
var person = {
  name: "Tom",
  age: 18,
  running: function() {
    console.log("running")
  }
}
```

### 2. 常见对象操作

- 读取属性：`obj.name`
- 修改属性：`obj.age = 20`
- 新增属性：`obj.height = 1.8`
- 删除属性：`delete obj.height`

```js
var obj = { name: "Tom", age: 18 }

console.log(obj.name)
obj.age = 20
obj.height = 1.8
delete obj.height
```

### 3. `this`

学习对象时一定会碰到 `this`。

初学阶段先记住：

- 谁调用函数，`this` 通常就指向谁

```js
var person = {
  name: "Tom",
  sayName: function() {
    console.log(this.name)
  }
}
```

### 4. 工厂函数和构造函数

工厂函数：

```js
function createPerson(name, age) {
  return {
    name: name,
    age: age
  }
}
```

构造函数：

```js
function Person(name, age) {
  this.name = name
  this.age = age
}

var p1 = new Person("Tom", 18)
```

理解目标：

- 会创建对象
- 知道对象能封装属性和行为
- 知道 `new` 和构造函数是面向对象基础

## 九、内置类与常用数据处理

应用场景：

- 用在字符串截取、数组增删改查、日期格式化、随机数生成、数值取整等日常开发操作。
- 适合搜索建议、列表处理、时间展示、去重排序、分页计算这类高频业务。

### 1. Number 和 Math

常见能力：

- 数字转换
- 四舍五入
- 最大值最小值
- 随机数

```js
Math.floor(3.9)
Math.ceil(3.1)
Math.round(3.5)
Math.random()
```

```js
var price = "99.8"
console.log(Number(price))      // 99.8
console.log(Math.max(10, 20))   // 20
console.log(Math.min(10, 20))   // 10
```

### 2. String

常见操作：

- 获取长度：`str.length`
- 转大小写
- 查找内容：`includes`、`indexOf`
- 截取：`slice`、`substring`
- 替换：`replace`
- 拆分：`split`

```js
var message = "hello javascript"

console.log(message.length)
console.log(message.toUpperCase())
console.log(message.includes("java"))
console.log(message.slice(6, 16))
console.log(message.replace("javascript", "JS"))
console.log("a-b-c".split("-"))
```

### 3. Array

数组是学习重点，建议重点掌握：

- 创建数组
- 通过索引读写元素
- `length`
- 遍历
- 增删改查

常见方法：

- `push`
- `pop`
- `shift`
- `unshift`
- `splice`
- `slice`
- `concat`
- `join`
- `find`
- `forEach`
- `map`
- `filter`

```js
var names = ["Tom", "Jack"]
names.push("Lucy")
```

```js
console.log(names[0])              // Tom
console.log(names.length)          // 3
console.log(names.slice(1))        // ["Jack", "Lucy"]
console.log(names.join(", "))      // Tom, Jack, Lucy
console.log(names.map(function(item) {
  return item.toUpperCase()
}))
```

### 4. Date

主要掌握：

- 创建时间对象
- 获取年月日时分秒
- 获取时间戳

```js
var date = new Date()
date.getFullYear()
date.getMonth()
date.getDate()
Date.now()
```

```js
var date = new Date("2026-05-05 10:30:00")

console.log(date.getFullYear()) // 2026
console.log(date.getMonth() + 1)
console.log(date.getDate())
console.log(Date.now())
```

## 十、DOM 基础

应用场景：

- 用在获取元素、修改文本、切换图片、读取输入框内容、设置属性等页面基础交互里。
- 适合原生 JavaScript 小项目、静态页面增强、后台模板页交互改造等场景。

### 1. 什么是 DOM

DOM 是浏览器把 HTML 文档转换成的一棵对象树。

你可以理解为：

- 页面上的每一个标签
- 每一段文本
- 每一个属性

都可以通过 JavaScript 找到并操作。

### 2. 获取元素

常见方法：

- `document.getElementById()`
- `document.getElementsByClassName()`
- `document.getElementsByTagName()`
- `document.querySelector()`
- `document.querySelectorAll()`

现代开发中最常用的是：

- `querySelector`
- `querySelectorAll`

```js
var titleEl = document.getElementById("title")
var boxEl = document.querySelector(".box")
var itemEls = document.querySelectorAll(".item")

console.log(titleEl)
console.log(boxEl)
console.log(itemEls.length)
```

### 3. 常见节点属性

- `nodeType`
- `nodeName`
- `textContent`
- `innerHTML`

区别重点：

- `textContent` 处理纯文本
- `innerHTML` 可以读写 HTML 结构

```js
var boxEl = document.querySelector(".box")

console.log(boxEl.nodeType)
console.log(boxEl.nodeName)
console.log(boxEl.textContent)

boxEl.innerHTML = "<strong>加粗文本</strong>"
```

### 4. 节点导航

常见导航属性：

- `parentNode`
- `children`
- `firstElementChild`
- `lastElementChild`
- `nextElementSibling`
- `previousElementSibling`

```js
var listEl = document.querySelector(".list")

console.log(listEl.parentNode)
console.log(listEl.children)
console.log(listEl.firstElementChild)
console.log(listEl.lastElementChild)
```

## 十一、DOM 进阶操作

应用场景：

- 用在动态创建节点、删除节点、修改样式、读取尺寸、监听滚动等复杂交互里。
- 适合做弹窗、标签页、评论列表、吸顶效果、回到顶部、懒加载等功能。

### 1. attribute 和 property

这是 DOM 学习里非常重要的一组概念。

- `attribute`：写在 HTML 标签上的特性
- `property`：DOM 对象上的属性

```html
<input class="account" value="Tom" />
```

```js
var inputEl = document.querySelector(".account")

console.log(inputEl.getAttribute("value")) // HTML 里的值
console.log(inputEl.value)                 // 当前输入框里的值

inputEl.value = "Jack"
inputEl.setAttribute("data-role", "admin")
```

### 2. 操作 class 和 style

推荐优先使用：

- `classList.add()`
- `classList.remove()`
- `classList.toggle()`

```js
var box = document.querySelector(".box")

box.classList.add("active")
box.style.color = "red"
```

```js
var box = document.querySelector(".box")

box.classList.remove("hidden")
box.classList.toggle("selected")
box.style.fontSize = "20px"
box.style.backgroundColor = "skyblue"
```

### 3. 创建、插入、删除元素

```js
var list = document.querySelector(".list")
var li = document.createElement("li")
li.textContent = "新内容"
list.append(li)
```

还要掌握：

- `append`
- `prepend`
- `before`
- `remove`
- `cloneNode`

```js
var list = document.querySelector(".list")
var firstLi = list.firstElementChild
var copyLi = firstLi.cloneNode(true)

list.prepend(copyLi)
firstLi.remove()
```

### 4. 元素尺寸和滚动

常见属性：

- `clientWidth`
- `clientHeight`
- `offsetWidth`
- `offsetHeight`
- `scrollTop`
- `scrollLeft`

```js
var box = document.querySelector(".scroll-box")

console.log(box.clientWidth)
console.log(box.offsetHeight)
console.log(box.scrollTop)
```

### 5. `window` 的尺寸和滚动

和页面联动时经常会用到：

- `window.innerWidth`
- `window.innerHeight`
- `window.scrollTo()`

```js
console.log(window.innerWidth)
console.log(window.innerHeight)

window.scrollTo({
  top: 500,
  behavior: "smooth"
})
```

## 十二、事件处理

应用场景：

- 用在点击、输入、提交、滚动、键盘操作等所有用户交互行为中。
- 适合做事件委托列表、搜索输入提示、表单提交拦截、菜单展开收起、拖拽联动等功能。

`08_JavaScript的事件监听.md` 和 `12_JavaScript的事件监听.md` 内容高度重复，这里已经合并整理。

### 1. 什么是事件

事件就是用户或浏览器触发的动作，比如：

- 点击
- 输入
- 键盘按下
- 鼠标移入移出
- 页面加载完成

### 2. 绑定事件

```js
var btn = document.querySelector(".btn")

btn.addEventListener("click", function() {
  console.log("按钮被点击了")
})
```

### 3. 事件对象 `event`

在事件回调里通常能拿到事件对象：

```js
var btn = document.querySelector(".btn")

btn.addEventListener("click", function(event) {
  console.log(event)
})
```

常见用途：

- 获取触发源
- 阻止默认行为
- 阻止事件传播

```js
var linkEl = document.querySelector(".link")

linkEl.addEventListener("click", function(event) {
  event.preventDefault()
  console.log("点击源：", event.target)
})
```

### 4. 事件流

事件流包含两个重要阶段：

- 捕获
- 冒泡

初学阶段先重点理解冒泡：

- 点击子元素时，父元素通常也能收到事件

```js
var parentEl = document.querySelector(".parent")
var childEl = document.querySelector(".child")

parentEl.addEventListener("click", function() {
  console.log("父元素被点击")
})

childEl.addEventListener("click", function(event) {
  console.log("子元素被点击")
  event.stopPropagation()
})
```

### 5. 事件委托

当多个子元素都需要相同事件处理时，可以把事件绑定到父元素上。

优点：

- 减少事件绑定数量
- 动态新增元素也能响应

```js
var listEl = document.querySelector(".list")

listEl.addEventListener("click", function(event) {
  if (event.target.matches("li")) {
    console.log("点击了：", event.target.textContent)
  }
})
```

### 6. 常见事件

鼠标事件：

- `click`
- `dblclick`
- `mousedown`
- `mouseup`
- `mouseenter`
- `mouseleave`

键盘事件：

- `keydown`
- `keyup`

表单事件：

- `input`
- `change`
- `focus`
- `blur`
- `submit`

文档事件：

- `DOMContentLoaded`

```js
var inputEl = document.querySelector(".keyword")

document.addEventListener("DOMContentLoaded", function() {
  console.log("DOM 已加载完成")
})

inputEl.addEventListener("input", function() {
  console.log("正在输入：", inputEl.value)
})
```

## 十三、BOM、JSON 和本地存储

应用场景：

- 用在页面跳转、读写浏览器缓存、序列化接口数据、获取浏览器环境信息时。
- 适合记住登录、购物车本地缓存、主题切换、最近浏览记录、地址参数解析等场景。

### 1. 什么是 BOM

BOM 是浏览器对象模型，主要关注浏览器窗口本身，而不是页面节点。

### 2. `window`

`window` 是浏览器中的全局对象。

常见能力：

- 弹窗
- 定时器
- 页面滚动
- 获取窗口尺寸

```js
var timerId = setTimeout(function() {
  console.log("2 秒后执行")
}, 2000)

console.log(window.innerWidth)
clearTimeout(timerId)
```

### 3. `location`

常见用途：

- 获取当前地址
- 页面跳转
- 解析查询参数

```js
location.href
location.reload()
```

```js
console.log(location.href)
console.log(location.pathname)

// location.href = "https://example.com"
```

### 4. `history`

可以操作浏览历史：

- `history.back()`
- `history.forward()`
- `history.go()`

```js
console.log(history.length)

// history.back()
// history.go(-1)
```

### 5. `navigator` 和 `screen`

了解即可，日常业务使用相对少。

```js
console.log(navigator.userAgent)
console.log(screen.width, screen.height)
```

### 6. JSON

JSON 是一种数据交换格式。

最常用的两个方法：

- `JSON.stringify()`
- `JSON.parse()`

```js
var obj = { name: "Tom", age: 18 }
var str = JSON.stringify(obj)
var data = JSON.parse(str)
```

### 7. localStorage 和 sessionStorage

共同点：

- 都是浏览器端的键值对存储

区别：

- `localStorage` 长期保存
- `sessionStorage` 当前会话有效

常见方法：

- `setItem`
- `getItem`
- `removeItem`
- `clear`

```js
localStorage.setItem("token", "123")
localStorage.getItem("token")
```

```js
sessionStorage.setItem("page", "home")
console.log(sessionStorage.getItem("page"))
localStorage.removeItem("token")
```

## 十四、实战案例该怎么学

应用场景：

- 用在把前面的变量、条件、循环、函数、DOM、事件真正组合成完整功能时。
- 适合轮播图、tab 切换、购物车结算、倒计时提示这类综合练习和面试手写题。

`13_JavaScript-DOM实战案例.md` 里比较重要的不是某一段代码，而是“把多个知识点组合起来”的思路。

### 1. 定时器

两个核心 API：

- `setTimeout`
- `setInterval`

它们经常和下面这些场景一起出现：

- 轮播图
- 自动消息提示
- 倒计时
- 自动切换 tab

```js
setTimeout(function() {
  console.log("3 秒后显示提示")
}, 3000)

var counter = 0
var timer = setInterval(function() {
  counter++
  console.log("第", counter, "次轮播")
  if (counter === 3) clearInterval(timer)
}, 1000)
```

### 2. 轮播图类案例的知识点

一般会同时用到：

- 数组或索引管理当前项
- DOM 查询和切换 class
- 点击事件
- 定时器
- 边界处理

```js
var currentIndex = 0
var banners = ["图1", "图2", "图3"]

function showBanner(index) {
  console.log("当前显示：", banners[index])
}

currentIndex = (currentIndex + 1) % banners.length
showBanner(currentIndex)
```

### 3. tab 切换类案例的知识点

一般会同时用到：

- 获取多个按钮和内容区
- 点击某个按钮
- 移除旧状态
- 添加新状态

```js
var tabs = document.querySelectorAll(".tab")
var panels = document.querySelectorAll(".panel")

tabs.forEach(function(tab, index) {
  tab.addEventListener("click", function() {
    tabs.forEach(function(item) {
      item.classList.remove("active")
    })
    panels.forEach(function(panel) {
      panel.classList.remove("active")
    })

    tab.classList.add("active")
    panels[index].classList.add("active")
  })
})
```

### 4. 购物车类案例的知识点

一般会同时用到：

- 数据与界面同步
- 数量加减
- 单价与总价计算
- 事件委托
- DOM 更新

```js
var count = 2
var price = 99
var total = count * price

console.log("总价：", total)
```

## 十五、这套资料的核心学习重点

如果时间有限，优先把下面这些内容学扎实：

### 第一优先级

- 变量和数据类型
- 运算符
- 分支和循环
- 函数

### 第二优先级

- 对象
- 数组
- 字符串
- DOM 查询与修改
- 事件绑定

### 第三优先级

- BOM
- JSON
- Storage
- 实战案例

## 十六、容易混淆的知识点

### 1. `==` 和 `===`

- `==` 会做类型转换
- `===` 不会

结论：

- 默认使用 `===`

### 2. `null` 和 `undefined`

- `undefined`：通常表示“没有赋值”
- `null`：通常表示“刻意置空”

```js
var a
var user = null

console.log(a)    // undefined
console.log(user) // null
```

### 3. `innerHTML` 和 `textContent`

- `innerHTML` 会解析标签
- `textContent` 只处理纯文本

```js
var box = document.querySelector(".box")

box.innerHTML = "<span>你好</span>"
box.textContent = "<span>你好</span>"
```

### 4. `attribute` 和 `property`

- `attribute` 更接近 HTML
- `property` 更接近 DOM 对象当前状态

```js
var checkbox = document.querySelector('input[type="checkbox"]')

checkbox.getAttribute("checked")
checkbox.checked
```

### 5. 冒泡和委托

- 冒泡是事件传播现象
- 委托是利用冒泡优化事件处理的模式

```js
var ulEl = document.querySelector(".menu")

ulEl.addEventListener("click", function(event) {
  if (event.target.matches("button")) {
    console.log("通过委托处理按钮点击")
  }
})
```

## 十七、建议你的复习方式

### 第一轮

- 只看概念
- 能说出每个主题解决什么问题

### 第二轮

- 每个主题自己手敲 3 到 5 个例子
- 重点练：分支、循环、函数、数组、DOM、事件

### 第三轮

- 自己做小案例
- 例如：登录提示、计数器、tab 切换、轮播图、购物车

### 第四轮

- 回头看原始 Markdown
- 把 OCR 不够准确的地方按你自己的理解再修一遍

## 十八、一个最小知识树

```text
JavaScript
├─ 基础语法
│  ├─ 变量
│  ├─ 数据类型
│  ├─ 运算符
│  ├─ 分支
│  ├─ 循环
│  └─ 函数
├─ 对象与内置类
│  ├─ Object
│  ├─ Array
│  ├─ String
│  ├─ Number/Math
│  └─ Date
├─ 浏览器编程
│  ├─ DOM
│  ├─ Event
│  └─ BOM
└─ 实战案例
   ├─ 定时器
   ├─ tab 切换
   ├─ 轮播图
   └─ 购物车
```

## 十九、学习这套内容的最终目标

学完这部分后，至少应该能独立完成：

- 用 JavaScript 写基础逻辑
- 写函数拆分代码
- 用数组和对象管理数据
- 操作页面元素
- 监听点击、输入等事件
- 做简单交互页面

如果你接下来要继续深入，建议下一阶段重点学：

- `let` / `const`
- 作用域和闭包
- 原型与原型链
- `this` 的完整规则
- Promise 和异步编程
- 模块化
- 网络请求
