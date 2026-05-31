# JavaScript02 学习整理

这份笔记基于 `JavaScript02` 目录下的 18 份 Markdown 归纳整理，目标是把“JavaScript 进阶”内容整理成一条更清晰的学习路径，并在每个核心知识点下面补上最小可运行代码示例，方便连续学习和回顾。

## 学习路线

建议按下面的顺序学习：

1. 先理解 `this`、执行上下文、作用域、闭包这些语言底层机制
2. 再学习函数增强、对象增强、原型和继承
3. 接着掌握 ES6+ 新特性和现代语法
4. 然后进入 `Proxy`、`Promise`、`Iterator`、`Generator`、`async/await`
5. 最后学习浏览器渲染、事件循环、Storage、正则、防抖节流、网络编程

## 原始资料对应关系

| 主题 | 原文件 |
|---|---|
| `this` 指向 | `01_JavaScript高级-this指向.md` |
| 浏览器渲染原理 | `02_深入浏览器的渲染原理.md` |
| JavaScript 运行原理 | `03_深入JavaScript的运行原理.md` |
| 内存管理和闭包 | `04_JavaScript内存管理和闭包.md` |
| 函数增强 | `05_JavaScript函数的增强知识.md` |
| 对象增强 | `06_JavaScript对象的增强知识.md` |
| ES5 原型与继承 | `07_JavaScript ES5实现继承.md` |
| ES6 class 继承 | `08_JavaScript ES6实现继承.md` |
| ES6~ES13 新特性 | `09_`、`10_`、`11_` 系列 |
| Proxy / Reflect | `12_Proxy-Reflect详解.md` |
| Promise | `13_Promise用法详解.md` |
| Iterator / Generator | `14_Iterator-Generator.md` |
| async / await / 事件循环 | `15_await-async-事件循环.md` |
| Storage / RegExp | `16_storage和正则表达式.md` |
| 防抖 / 节流 / 深拷贝 / 事件总线 | `17_防抖-节流-深拷贝-事件总线.md` |
| XHR / Fetch / HTTP | `18_JavaScript网络编程.md` |

## 一、this 指向

应用场景：

- 用在对象方法、构造函数、类实例、事件回调、定时器回调里判断当前上下文是谁。
- 适合组件方法、面向对象代码、工具库封装和面试题里分析调用者变化。

### 1. 默认绑定

普通函数直接调用时，非严格模式下 `this` 通常指向 `window`，严格模式下是 `undefined`。

```js
function foo() {
  console.log(this)
}

foo()
```

### 2. 隐式绑定

对象调用方法时，`this` 通常指向调用它的对象。

```js
var obj = {
  name: "Tom",
  sayName: function() {
    console.log(this.name)
  }
}

obj.sayName() // Tom
```

### 3. 显式绑定

可以用 `call`、`apply`、`bind` 手动指定 `this`。

```js
function introduce(city, hobby) {
  console.log(this.name, city, hobby)
}

var user = { name: "Tom" }

introduce.call(user, "Shanghai", "code")
introduce.apply(user, ["Beijing", "music"])

var fn = introduce.bind(user, "Hangzhou")
fn("reading")
```

### 4. new 绑定

通过 `new` 调用构造函数时，`this` 指向新创建的实例对象。

```js
function Person(name) {
  this.name = name
}

var p = new Person("Jack")
console.log(p.name)
```

### 5. 箭头函数的 this

箭头函数没有自己的 `this`，它会继承外层作用域的 `this`。

```js
var obj = {
  name: "Lucy",
  run: function() {
    setTimeout(() => {
      console.log(this.name)
    }, 100)
  }
}

obj.run() // Lucy
```

### 6. this 优先级

通常可以这样记：

- `new` 绑定优先级最高
- 显式绑定高于隐式绑定
- 隐式绑定高于默认绑定

## 二、浏览器渲染原理

应用场景：

- 用在排查页面卡顿、白屏、闪烁、样式抖动和首屏加载慢这类性能问题时。
- 适合优化大列表、动画效果、脚本加载顺序和资源渲染时机。

### 1. 从 HTML 到页面显示的大致流程

浏览器渲染大致经历：

1. 解析 HTML，构建 DOM Tree
2. 解析 CSS，构建 CSSOM
3. 合成 Render Tree
4. Layout 计算布局
5. Paint 绘制
6. Composite 合成图层

### 2. 回流和重绘

- 回流：元素几何信息变化，重新布局
- 重绘：样式变化但不影响布局，只重新绘制

```js
var box = document.querySelector(".box")

box.style.width = "300px"   // 可能触发回流
box.style.color = "red"     // 通常只触发重绘
```

### 3. defer 和 async

它们都能避免脚本阻塞 HTML 解析，但行为不同：

- `defer`：HTML 解析完后，按顺序执行
- `async`：下载完立刻执行，不保证顺序

```html
<script defer src="./main.js"></script>
<script async src="./analytics.js"></script>
```

## 三、JavaScript 执行原理

应用场景：

- 用在理解变量查找、函数调用、作用域嵌套、提升行为和代码执行顺序时。
- 适合排查“为什么这个变量拿不到”或“为什么结果和预期不一样”的问题。

### 1. 执行上下文

JavaScript 在执行代码时，会为全局代码和每个函数创建执行上下文。

```js
var message = "global"

function foo() {
  var count = 10
  console.log(count)
}

foo()
```

可以先简单理解为：

- 全局代码有全局执行上下文
- 每次调用函数，都会创建函数执行上下文

### 2. 作用域和作用域链

变量查找会沿着当前作用域一层层向外找。

```js
var message = "global"

function outer() {
  var name = "outer"

  function inner() {
    console.log(message)
    console.log(name)
  }

  inner()
}

outer()
```

### 3. 变量提升

`var` 声明会提升，函数声明也会提升。

```js
console.log(num) // undefined
var num = 10

foo()
function foo() {
  console.log("foo")
}
```

## 四、内存管理和闭包

应用场景：

- 用在封装私有状态、实现函数工厂、缓存数据、模块隔离逻辑时。
- 适合计数器、配置记忆、一次性初始化函数、事件回调保留上下文等场景。

### 1. JavaScript 的内存管理

JavaScript 会自动管理内存，常见的垃圾回收思想包括：

- 引用计数
- 标记清除

### 2. 闭包是什么

闭包可以简单理解为：函数和它能访问到的词法环境组合在一起。

```js
function makeCounter() {
  var count = 0

  return function() {
    count++
    return count
  }
}

var counter = makeCounter()
console.log(counter()) // 1
console.log(counter()) // 2
```

### 3. 闭包的作用

- 延长局部变量的生命周期
- 封装私有状态
- 实现函数工厂

```js
function createAdder(num) {
  return function(value) {
    return num + value
  }
}

var add10 = createAdder(10)
console.log(add10(5)) // 15
```

### 4. 闭包使用注意

闭包很有用，但如果引用了本该释放的大对象或 DOM，可能造成不必要的内存占用。

## 五、函数增强

应用场景：

- 用在做通用工具函数、组合业务逻辑、封装权限校验、参数预处理时。
- 适合函数式编程风格、日志包装、埋点包装、统一错误处理这类需求。

### 1. arguments

普通函数内部可以拿到 `arguments`。

```js
function sum() {
  var total = 0
  for (var i = 0; i < arguments.length; i++) {
    total += arguments[i]
  }
  return total
}

console.log(sum(1, 2, 3))
```

### 2. 剩余参数 rest

现代开发更推荐使用剩余参数。

```js
function sum(...nums) {
  return nums.reduce(function(total, item) {
    return total + item
  }, 0)
}

console.log(sum(10, 20, 30))
```

### 3. 纯函数

纯函数特点：

- 相同输入一定得到相同输出
- 不修改外部状态

```js
function add(a, b) {
  return a + b
}
```

反例：

```js
var count = 0

function addCount() {
  count++
}
```

### 4. 柯里化

把一个接收多个参数的函数转换成一系列接收单个参数的函数。

```js
function add(x) {
  return function(y) {
    return function(z) {
      return x + y + z
    }
  }
}

console.log(add(1)(2)(3))
```

### 5. 组合函数

把多个小函数组合成一个大函数。

```js
function double(num) {
  return num * 2
}

function square(num) {
  return num * num
}

function compose(fn1, fn2) {
  return function(value) {
    return fn2(fn1(value))
  }
}

var newFn = compose(double, square)
console.log(newFn(5)) // 100
```

### 6. 严格模式

严格模式会限制一些不安全的写法。

```js
"use strict"

function foo() {
  // age = 18 // 严格模式下会报错
}
```

## 六、对象增强

应用场景：

- 用在拦截属性读写、做数据劫持、定义只读字段、统一对象访问入口时。
- 适合表单模型封装、响应式实现、权限字段控制和对象兼容层设计。

### 1. Object.defineProperty

可以精细控制对象属性。

```js
var obj = {}

Object.defineProperty(obj, "name", {
  value: "Tom",
  writable: false,
  enumerable: true,
  configurable: false
})

console.log(obj.name)
```

### 2. getter / setter

可以拦截属性读取和设置。

```js
var user = {
  _name: "Jack"
}

Object.defineProperty(user, "name", {
  get: function() {
    return this._name
  },
  set: function(value) {
    this._name = value.trim()
  }
})

user.name = " Lucy "
console.log(user.name)
```

### 3. defineProperties

一次定义多个属性。

```js
var person = {}

Object.defineProperties(person, {
  name: {
    value: "Tom",
    enumerable: true
  },
  age: {
    value: 18,
    enumerable: true
  }
})

console.log(person)
```

## 七、原型与继承

应用场景：

- 用在理解 JavaScript 面向对象底层机制、封装共享方法、复用实例能力时。
- 适合老项目原型链代码阅读、类库设计、面试中的继承实现题。

### 1. 原型 prototype

函数都有 `prototype`，实例对象可以通过原型共享方法。

```js
function Person(name) {
  this.name = name
}

Person.prototype.sayName = function() {
  console.log(this.name)
}

var p1 = new Person("Tom")
var p2 = new Person("Jack")

p1.sayName()
p2.sayName()
```

### 2. 原型链

对象查找属性时，会先找自己，再沿着原型链向上查找。

```js
var obj = { name: "Tom" }

console.log(obj.toString()) // 来自原型链
```

### 3. ES5 寄生组合继承

这是 ES5 里比较完整的继承方案。

```js
function Person(name) {
  this.name = name
}

Person.prototype.sayName = function() {
  console.log(this.name)
}

function Student(name, sno) {
  Person.call(this, name)
  this.sno = sno
}

Student.prototype = Object.create(Person.prototype)
Student.prototype.constructor = Student

Student.prototype.study = function() {
  console.log(this.name + " is studying")
}
```

### 4. ES6 class 和 extends

现代开发更常见的是 `class`。

```js
class Person {
  constructor(name) {
    this.name = name
  }

  sayName() {
    console.log(this.name)
  }
}

class Student extends Person {
  constructor(name, sno) {
    super(name)
    this.sno = sno
  }

  study() {
    console.log(this.name + " is studying")
  }
}

var stu = new Student("Lucy", 101)
stu.sayName()
stu.study()
```

### 5. 静态方法 / getter / setter

```js
class User {
  constructor(name) {
    this._name = name
  }

  get name() {
    return this._name
  }

  set name(value) {
    this._name = value
  }

  static createGuest() {
    return new User("Guest")
  }
}

console.log(User.createGuest().name)
```

## 八、ES6 ~ ES13 常用新特性

应用场景：

- 用在现代前端日常开发里，几乎所有 Vue、React、Node 项目都会频繁出现。
- 适合简化数据处理、提升可读性、减少样板代码、写出更现代的模块化代码。

### 1. let / const 和块级作用域

```js
if (true) {
  let message = "hello"
  const age = 18
  console.log(message, age)
}
```

### 2. 模板字符串

```js
var name = "Tom"
var age = 18

console.log(`我是 ${name}，今年 ${age} 岁`)
```

### 3. 默认参数

```js
function request(url, method = "GET") {
  console.log(url, method)
}

request("/users")
```

### 4. 展开运算符

```js
var nums = [1, 2, 3]
console.log(...nums)

var arr1 = [1, 2]
var arr2 = [...arr1, 3, 4]
console.log(arr2)
```

### 5. 解构赋值

```js
var user = { name: "Tom", age: 18 }
var { name, age } = user

var nums = [10, 20, 30]
var [a, b] = nums

console.log(name, age, a, b)
```

### 6. Symbol

```js
var id = Symbol("id")
var obj = {
  [id]: 1001
}

console.log(obj[id])
```

### 7. Set 和 Map

```js
var set = new Set([1, 2, 2, 3])
console.log(set)

var map = new Map()
map.set("name", "Tom")
map.set("age", 18)

console.log(map.get("name"))
```

### 8. includes / flat / flatMap

```js
console.log([1, 2, 3].includes(2))
console.log([1, [2, [3]]].flat(2))

var result = [1, 2, 3].flatMap(function(item) {
  return [item, item * 2]
})

console.log(result)
```

### 9. Optional Chaining 和 Nullish Coalescing

```js
var user = {
  profile: {
    nickName: "Tom"
  }
}

console.log(user?.profile?.nickName)
console.log(user.address?.city ?? "未知城市")
```

### 10. BigInt 和 at

```js
var big = 12345678901234567890n
console.log(big)

var arr = [10, 20, 30]
console.log(arr.at(-1))
```

## 九、Proxy 和 Reflect

应用场景：

- 用在响应式系统、表单校验代理、访问拦截、日志记录、默认值兜底这类高级封装里。
- 适合自己实现简化版响应式、代理配置对象、做开发调试辅助工具。

### 1. Proxy 监听对象

```js
var obj = { name: "Tom" }

var proxy = new Proxy(obj, {
  get: function(target, key) {
    console.log("读取属性：", key)
    return target[key]
  },
  set: function(target, key, value) {
    console.log("设置属性：", key, value)
    target[key] = value
    return true
  }
})

proxy.name
proxy.age = 18
```

### 2. Reflect

`Reflect` 通常和 `Proxy` 配合使用，让默认行为更规范。

```js
var proxy = new Proxy({}, {
  set: function(target, key, value, receiver) {
    return Reflect.set(target, key, value, receiver)
  }
})

proxy.name = "Jack"
console.log(proxy.name)
```

## 十、Promise

应用场景：

- 用在接口请求、异步任务编排、并发处理、错误统一捕获时。
- 适合页面初始化加载多个接口、上传后继续下一步、重试机制、链式异步流程。

### 1. Promise 基本结构

```js
var promise = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve("请求成功")
  }, 1000)
})

promise.then(function(res) {
  console.log(res)
})
```

### 2. then / catch / finally

```js
Promise.resolve(100)
  .then(function(res) {
    return res + 1
  })
  .then(function(res) {
    console.log(res)
  })
  .catch(function(err) {
    console.log("错误：", err)
  })
  .finally(function() {
    console.log("完成")
  })
```

### 3. Promise 静态方法

```js
var p1 = Promise.resolve("A")
var p2 = Promise.resolve("B")

Promise.all([p1, p2]).then(function(res) {
  console.log(res)
})

Promise.race([p1, p2]).then(function(res) {
  console.log(res)
})
```

## 十一、Iterator 和 Generator

应用场景：

- 用在自定义遍历规则、惰性计算、分步执行任务、理解异步流程底层模型时。
- 适合实现可迭代对象、生成序列数据、学习中间件和协程式控制流思想。

### 1. Iterator 基本概念

迭代器本质上是一个带有 `next()` 方法的对象。

```js
var iterator = {
  index: 0,
  names: ["Tom", "Lucy"],
  next: function() {
    if (this.index < this.names.length) {
      return { value: this.names[this.index++], done: false }
    }
    return { value: undefined, done: true }
  }
}

console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())
```

### 2. 可迭代对象

实现 `Symbol.iterator` 后，就可以被 `for...of` 遍历。

```js
var classroom = {
  students: ["Tom", "Jack", "Lucy"],
  [Symbol.iterator]: function() {
    var index = 0
    var students = this.students
    return {
      next: function() {
        if (index < students.length) {
          return { value: students[index++], done: false }
        }
        return { value: undefined, done: true }
      }
    }
  }
}

for (var item of classroom) {
  console.log(item)
}
```

### 3. Generator

生成器函数可以更方便地创建迭代器。

```js
function* foo() {
  yield 1
  yield 2
  yield 3
}

var generator = foo()
console.log(generator.next())
console.log(generator.next())
console.log(generator.next())
```

### 4. Generator 传值

```js
function* foo() {
  var num = yield "first"
  console.log(num)
}

var generator = foo()
console.log(generator.next())
console.log(generator.next(100))
```

## 十二、async / await 与事件循环

应用场景：

- 用在写更直观的异步代码、串行等待接口、统一处理异步异常时。
- 适合页面初始化请求链、提交表单后等待结果、理解 UI 更新与异步回调先后顺序。

### 1. async / await

```js
function requestData() {
  return new Promise(function(resolve) {
    setTimeout(function() {
      resolve("data")
    }, 1000)
  })
}

async function main() {
  var res = await requestData()
  console.log(res)
}

main()
```

### 2. try / catch 处理异步错误

```js
async function main() {
  try {
    var res = await Promise.reject("error")
    console.log(res)
  } catch (err) {
    console.log("捕获到错误：", err)
  }
}

main()
```

### 3. 宏任务和微任务

可以先简单理解为：

- 宏任务：`setTimeout`、`setInterval`、I/O
- 微任务：`Promise.then`、`queueMicrotask`

```js
console.log("start")

setTimeout(function() {
  console.log("setTimeout")
}, 0)

Promise.resolve().then(function() {
  console.log("promise")
})

console.log("end")
```

输出顺序通常是：

```text
start
end
promise
setTimeout
```

## 十三、Storage 和正则表达式

应用场景：

- 用在本地缓存 token、搜索历史、表单草稿，以及手机号、邮箱、时间文本校验提取。
- 适合登录状态保留、输入规则校验、日志文本提取、内容过滤等业务需求。

### 1. localStorage / sessionStorage

```js
localStorage.setItem("token", "abc123")
console.log(localStorage.getItem("token"))

sessionStorage.setItem("page", "home")
console.log(sessionStorage.getItem("page"))
```

### 2. 正则表达式基本使用

```js
var reg = /abc/
console.log(reg.test("123abc456"))
```

### 3. 常见规则

- `\d`：数字
- `\w`：字母数字下划线
- `.`：任意字符
- `^`：开头
- `$`：结尾
- `*`、`+`、`?`、`{n,m}`：量词

### 4. 手机号校验示例

```js
var phoneReg = /^1[3-9]\d{9}$/

console.log(phoneReg.test("13812345678"))
console.log(phoneReg.test("123456"))
```

### 5. 提取时间字符串

```js
var text = "[03:25]Hello World"
var reg = /\[(\d{2}):(\d{2})\](.*)/
var result = text.match(reg)

console.log(result)
```

## 十四、防抖、节流、深拷贝、事件总线

应用场景：

- 用在搜索框输入优化、滚动监听优化、复杂对象复制、跨组件通信时。
- 适合窗口 resize、按钮防重复提交、拖拽滚动、管理后台筛选器联动等场景。

### 1. 防抖 debounce

频繁触发时，只在最后一次触发后执行。

```js
function debounce(fn, delay) {
  var timer = null
  return function() {
    var args = arguments
    var context = this
    clearTimeout(timer)
    timer = setTimeout(function() {
      fn.apply(context, args)
    }, delay)
  }
}
```

### 2. 节流 throttle

频繁触发时，按固定频率执行。

```js
function throttle(fn, interval) {
  var lastTime = 0
  return function() {
    var nowTime = Date.now()
    if (nowTime - lastTime >= interval) {
      fn.apply(this, arguments)
      lastTime = nowTime
    }
  }
}
```

### 3. 深拷贝

简化版深拷贝示例：

```js
function deepClone(obj) {
  if (obj === null || typeof obj !== "object") return obj

  var newObj = Array.isArray(obj) ? [] : {}

  for (var key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      newObj[key] = deepClone(obj[key])
    }
  }

  return newObj
}

var info = { name: "Tom", friend: { name: "Jack" } }
var copy = deepClone(info)
console.log(copy)
```

### 4. 事件总线

```js
function EventBus() {
  this.events = {}
}

EventBus.prototype.on = function(eventName, handler) {
  if (!this.events[eventName]) {
    this.events[eventName] = []
  }
  this.events[eventName].push(handler)
}

EventBus.prototype.emit = function(eventName, payload) {
  var handlers = this.events[eventName] || []
  handlers.forEach(function(handler) {
    handler(payload)
  })
}

var bus = new EventBus()
bus.on("login", function(data) {
  console.log("登录成功：", data)
})
bus.emit("login", { name: "Tom" })
```

## 十五、HTTP、XHR 和 Fetch

应用场景：

- 用在前后端数据交互、文件上传、接口封装、错误处理和请求流程管理里。
- 适合登录注册、列表查询、表单提交、图片上传、下载导出等真实项目功能。

### 1. HTTP 基本认识

学习网络编程时，先掌握这些最核心概念：

- 请求行 / 请求头 / 请求体
- 响应状态码
- GET / POST
- JSON 数据格式

### 2. XMLHttpRequest

```js
var xhr = new XMLHttpRequest()
xhr.open("GET", "https://jsonplaceholder.typicode.com/todos/1")
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4 && xhr.status >= 200 && xhr.status < 300) {
    console.log(xhr.responseText)
  }
}
xhr.send()
```

### 3. Fetch

```js
fetch("https://jsonplaceholder.typicode.com/todos/1")
  .then(function(res) {
    return res.json()
  })
  .then(function(data) {
    console.log(data)
  })
```

### 4. POST 请求

```js
fetch("https://jsonplaceholder.typicode.com/posts", {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    title: "hello",
    body: "world"
  })
})
  .then(function(res) {
    return res.json()
  })
  .then(function(data) {
    console.log(data)
  })
```

### 5. 文件上传思路

上传文件常见流程：

1. 用户选择文件
2. 用 `FormData` 封装文件
3. 发起 `POST` 请求
4. 服务端返回上传结果

```js
var formData = new FormData()
formData.append("file", fileInput.files[0])

fetch("/upload", {
  method: "POST",
  body: formData
})
```

## 十六、这部分内容的核心学习重点

如果时间有限，建议优先掌握这些内容：

### 第一优先级

- `this` 绑定规则
- 作用域、执行上下文、闭包
- 原型、原型链、继承
- `let / const`、解构、展开、模板字符串
- Promise
- `async / await`

### 第二优先级

- Proxy / Reflect
- Iterator / Generator
- 防抖 / 节流
- 深拷贝
- 正则表达式

### 第三优先级

- 浏览器渲染原理
- 事件循环细节
- Storage
- XHR / Fetch / 文件上传

## 十七、容易混淆的知识点

### 1. 普通函数 this 和箭头函数 this

```js
var obj = {
  name: "Tom",
  foo: function() {
    console.log(this.name)
  },
  bar: () => {
    console.log(this.name)
  }
}

obj.foo()
obj.bar()
```

### 2. var / let / const

- `var` 有变量提升、没有块级作用域
- `let` 有块级作用域
- `const` 有块级作用域，且不能重新赋值

### 3. 浅拷贝和深拷贝

```js
var obj = { info: { name: "Tom" } }
var copy = obj

copy.info.name = "Jack"
console.log(obj.info.name) // Jack
```

### 4. 宏任务和微任务

- 先执行同步代码
- 再清空微任务队列
- 再执行下一个宏任务

### 5. Promise 和 async / await 的关系

`async / await` 本质上是基于 Promise 的语法糖。

## 十八、建议你的复习方式

### 第一轮

- 先把整份笔记顺一遍
- 目标是说清楚每个主题解决什么问题

### 第二轮

- 手敲每个主题的最小示例
- 特别练：`this`、闭包、原型链、Promise、async/await

### 第三轮

- 自己做小练习
- 例如：手写防抖、深拷贝、事件总线、Promise 链式调用

### 第四轮

- 自己口述运行机制
- 尝试解释：变量提升、闭包、原型链、事件循环、Promise 顺序

## 十九、一个最小知识树

```text
JavaScript Advanced
├─ 语言底层
│  ├─ this
│  ├─ 执行上下文
│  ├─ 作用域/作用域链
│  ├─ 内存管理
│  └─ 闭包
├─ 函数与对象
│  ├─ arguments / rest
│  ├─ 纯函数 / 柯里化 / 组合
│  ├─ 属性描述符
│  ├─ prototype
│  ├─ 原型链
│  └─ 继承
├─ 现代语法
│  ├─ let / const
│  ├─ 模板字符串
│  ├─ 解构 / 展开
│  ├─ Set / Map / Symbol
│  └─ ES7~ES13
├─ 异步编程
│  ├─ Promise
│  ├─ Iterator
│  ├─ Generator
│  ├─ async / await
│  └─ 事件循环
└─ 浏览器与工程实践
   ├─ 渲染流程
   ├─ Storage
   ├─ RegExp
   ├─ 防抖 / 节流
   ├─ 深拷贝 / 事件总线
   └─ XHR / Fetch / HTTP
```

## 二十、学完这部分后的目标

学完这部分后，你至少应该能做到：

- 解释 `this` 的几种绑定规则
- 看懂闭包、原型链、继承代码
- 使用 ES6+ 常见语法写更现代的 JavaScript
- 用 Promise 和 `async/await` 处理异步逻辑
- 理解宏任务、微任务和事件循环
- 手写常见工具函数
- 使用 XHR 或 Fetch 发网络请求
