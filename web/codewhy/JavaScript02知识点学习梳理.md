# JavaScript02 知识点学习梳理

本文根据 `JavaScript02` 文件夹中的课程主题整理，覆盖内容包括：

- `this` 指向
- 浏览器渲染原理
- JavaScript 运行原理
- 内存管理和闭包
- 函数增强知识
- 对象增强知识
- ES5 / ES6 继承
- ES6 ~ ES13 新特性
- Proxy / Reflect
- Promise
- Iterator / Generator
- `async` / `await` / 事件循环
- `storage` 和正则表达式
- 防抖 / 节流 / 深拷贝 / 事件总线
- JavaScript 网络编程

目标是把这些内容重组成一份适合学习、复习和查漏补缺的文档。

---

## 1. JavaScript02 主要学什么

一句话理解：

`JavaScript02 这一阶段，主要是在学 JavaScript 的进阶语法、底层原理、异步编程和工程实用能力。`

和 `JavaScript01` 相比，它不再只是“会写交互”，而是继续往这几个方向深入：

- 理解 JavaScript 为什么这样运行
- 学会写更现代、更规范的 JavaScript
- 学会处理异步逻辑
- 学会更复杂的对象与函数设计
- 具备真实项目里的常见能力

---

## 2. 这套 JavaScript02 内容的学习主线

这套内容大致可以分成 6 个层次：

1. JavaScript 原理：`this`、执行过程、内存、闭包、事件循环
2. 函数和对象的进阶能力
3. 原型链、继承、ES6 class
4. ES6+ 新特性与现代写法
5. Promise / async / await / 网络请求
6. storage、正则、防抖节流、深拷贝等项目常用能力

如果你是从 JavaScript01 继续往后学，可以先抓住一句话：

`JavaScript02 的核心是：原理、异步、对象模型、现代语法。`

---

## 3. 哪些知识点是必须掌握的

下面这些内容，是继续学习 Vue、React、Node.js、工程化前端时一定要打牢的基础。

---

## 4. `this` 指向

### 4.1 `this` 是什么

**必须掌握**

- `this` 不是在函数定义时决定
- `this` 是在函数调用时决定
- 不同调用方式会影响 `this`

---

### 4.2 常见 `this` 指向场景

**必须掌握**

- 默认绑定
- 隐式绑定
- 显式绑定：`call` / `apply` / `bind`
- `new` 绑定
- 箭头函数中的 `this`

**学习示例**

```js
const obj = {
  name: "Tom",
  sayHello() {
    console.log(this.name);
  }
};

obj.sayHello(); // Tom
```

```js
function foo() {
  console.log(this);
}

foo(); // 普通调用
foo.call({ name: "Kobe" }); // 显式绑定
```

**应用场景**

- 对象方法调用
- 构造函数
- 事件回调
- 定时器回调
- 类和组件方法

**必须理解**

- 箭头函数没有自己的 `this`
- 箭头函数会继承外层作用域的 `this`

---

## 5. JavaScript 运行原理

### 5.1 执行上下文和作用域

**必须掌握**

- 全局执行上下文
- 函数执行上下文
- 作用域链

**应用场景**

- 理解变量为什么能访问
- 理解函数嵌套时的查找规则
- 排查变量覆盖问题

---

### 5.2 浏览器渲染和 JavaScript 执行

**建议掌握程度：理解**

需要知道：

- JavaScript 会阻塞页面渲染
- 浏览器有渲染流程
- DOM、CSSOM、布局、绘制是页面显示的重要环节

**应用场景**

- 理解为什么脚本位置影响页面加载
- 理解性能优化的基本方向

---

## 6. 内存管理和闭包

### 6.1 内存管理基础

**建议掌握程度：理解**

需要知道：

- 基本数据和引用数据存储特点不同
- 对象、函数、数组通常会占用堆内存
- JavaScript 有垃圾回收机制

**应用场景**

- 理解为什么对象赋值会互相影响
- 理解内存泄漏的风险

---

### 6.2 闭包

**必须掌握**

闭包的核心理解：

`一个函数访问了它外层作用域中的变量，并且这些变量在函数执行结束后仍然被保留下来。`

**学习示例**

```js
function createCounter() {
  let count = 0;

  return function () {
    count++;
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

**应用场景**

- 封装私有变量
- 防抖节流
- 模块化设计
- 回调函数

**必须理解**

- 闭包是高频面试题
- 但更重要的是理解它在实际代码里的用途

---

## 7. 函数的增强知识

### 7.1 参数与返回值增强

**必须掌握**

- 默认参数
- 剩余参数 `...args`
- 展开运算符

**学习示例**

```js
function sum(...nums) {
  return nums.reduce((total, item) => total + item, 0);
}

console.log(sum(1, 2, 3, 4));
```

**应用场景**

- 参数数量不固定
- 函数封装更灵活

---

### 7.2 高阶函数

**必须掌握**

高阶函数指：

- 接收函数作为参数
- 或返回函数

**学习示例**

```js
function exec(fn) {
  fn();
}

exec(() => {
  console.log("执行了回调函数");
});
```

**应用场景**

- 事件监听
- 数组方法
- Promise
- 防抖节流

---

## 8. 对象增强知识

### 8.1 对象属性操作

**必须掌握**

- 属性描述符
- `Object.defineProperty`
- `Object.keys`
- `Object.assign`

**学习示例**

```js
const obj = { name: "Tom" };
const newObj = Object.assign({}, obj, { age: 18 });
console.log(newObj);
```

**应用场景**

- 对象合并
- 配置项处理
- 浅拷贝

---

### 8.2 解构赋值

**必须掌握**

- 数组解构
- 对象解构

**学习示例**

```js
const user = {
  name: "Jack",
  age: 20
};

const { name, age } = user;
```

**应用场景**

- 接口数据处理
- 函数参数解构
- Vue / React 项目中频繁使用

---

## 9. 原型、原型链和继承

### 9.1 原型和原型链

**必须掌握**

- 每个函数都有 `prototype`
- 每个对象可以通过原型链访问上层属性
- 查找属性时会沿原型链向上查找

**应用场景**

- 理解实例方法从哪里来
- 理解 `Array`、`Object` 等内置类型的方法

---

### 9.2 ES5 继承

**建议掌握程度：理解**

需要知道：

- 构造函数继承
- 原型链继承
- 组合继承

**为什么不是最高优先级**

- 现代项目更常用 `class`
- 但 ES5 继承有助于理解原型链

---

### 9.3 ES6 `class` 继承

**必须掌握**

- `class`
- `constructor`
- `extends`
- `super`

**学习示例**

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  sayHello() {
    console.log(`Hello, ${this.name}`);
  }
}

class Student extends Person {
  constructor(name, grade) {
    super(name);
    this.grade = grade;
  }
}
```

**应用场景**

- 面向对象封装
- UI 组件封装思路
- 类式代码组织

---

## 10. ES6 ~ ES13 新特性

### 10.1 必须优先掌握的 ES6+ 能力

**必须掌握**

- `let` / `const`
- 模板字符串
- 解构赋值
- 箭头函数
- 剩余参数
- 展开运算符
- `class`
- `Promise`
- `Map` / `Set` 要理解常用场景
- 可选链 `?.`
- 空值合并 `??`

**学习示例**

```js
const name = "Vue";
console.log(`正在学习 ${name}`);
```

```js
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4];
```

**应用场景**

- 现代前端项目几乎到处都在用
- Vue / React / Node 代码中都是常见语法

---

## 11. Proxy 和 Reflect

### 11.1 Proxy

**必须掌握程度：理解 + 会基础使用**

Proxy 可以拦截对象操作，比如：

- 读取
- 设置
- 删除

**学习示例**

```js
const obj = { name: "Tom" };

const proxy = new Proxy(obj, {
  get(target, key) {
    return target[key];
  },
  set(target, key, value) {
    target[key] = value;
    return true;
  }
});
```

**应用场景**

- Vue3 响应式原理
- 数据代理
- 拦截和校验对象操作

---

### 11.2 Reflect

**建议掌握程度：理解**

Reflect 常和 Proxy 配合使用，让对象操作更规范。

**应用场景**

- Proxy 内部转发默认行为

---

## 12. Promise

### 12.1 Promise 基础

**必须掌握**

- `new Promise`
- `resolve`
- `reject`
- `then`
- `catch`
- `finally`

**学习示例**

```js
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("请求成功");
  }, 1000);
});

p.then((res) => {
  console.log(res);
}).catch((err) => {
  console.log(err);
});
```

**应用场景**

- 网络请求
- 异步任务处理
- 多个异步流程串联

**必须理解**

- Promise 是异步编程的核心
- `then` 返回的还是 Promise

---

### 12.2 Promise 常用静态方法

**必须掌握**

- `Promise.resolve`
- `Promise.reject`
- `Promise.all`
- `Promise.race`

**应用场景**

- 并发请求
- 聚合多个异步结果

---

## 13. Iterator 和 Generator

### 13.1 Iterator

**建议掌握程度：理解**

需要知道：

- 可迭代对象
- `next()`
- `value` / `done`

**应用场景**

- 理解 `for...of`
- 理解迭代协议

---

### 13.2 Generator

**建议掌握程度：理解**

- `function*`
- `yield`

**应用场景**

- 理解异步流程演进
- 理解更底层的控制流程思想

**学习建议**

- 初学阶段知道概念和基本写法即可

---

## 14. `async` / `await`

### 14.1 基础使用

**必须掌握**

- `async` 函数
- `await`
- 配合 `try...catch`

**学习示例**

```js
function requestData() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve("数据加载完成");
    }, 1000);
  });
}

async function getData() {
  try {
    const res = await requestData();
    console.log(res);
  } catch (error) {
    console.log(error);
  }
}

getData();
```

**应用场景**

- 接口请求
- 串行异步逻辑
- 页面初始化数据加载

**必须理解**

- `await` 后面通常接 Promise
- `async / await` 本质上是 Promise 的语法糖

---

## 15. 事件循环

### 15.1 事件循环核心认知

**必须掌握**

- 同步任务
- 异步任务
- 宏任务
- 微任务

**必须知道的结论**

- 先执行同步代码
- 再清空微任务
- 再执行下一个宏任务

**学习示例**

```js
console.log(1);

setTimeout(() => {
  console.log(2);
}, 0);

Promise.resolve().then(() => {
  console.log(3);
});

console.log(4);
```

输出顺序通常是：

```js
1
4
3
2
```

**应用场景**

- 理解异步执行顺序
- 理解 Promise 和定时器顺序
- 面试高频题

---

## 16. storage 和正则表达式

### 16.1 localStorage / sessionStorage

**必须掌握**

- `setItem`
- `getItem`
- `removeItem`
- `clear`

**学习示例**

```js
localStorage.setItem("token", "123456");
const token = localStorage.getItem("token");
```

**应用场景**

- 保存 token
- 保存用户主题设置
- 保存上次浏览信息

---

### 16.2 正则表达式

**必须掌握程度：基础必须会**

重点先掌握：

- 创建正则
- `test`
- `match`
- 常见元字符

**学习示例**

```js
const reg = /^1[3-9]\\d{9}$/;
console.log(reg.test("13812345678"));
```

**应用场景**

- 手机号校验
- 邮箱校验
- 密码规则校验
- 文本匹配与替换

---

## 17. 项目常用手写能力

### 17.1 防抖

**必须掌握**

核心作用：

`频繁触发时，只在最后一次真正执行。`

**应用场景**

- 搜索框输入联想
- 窗口 resize
- 防止按钮重复提交

---

### 17.2 节流

**必须掌握**

核心作用：

`频繁触发时，按固定时间间隔执行。`

**应用场景**

- 滚动监听
- 鼠标移动
- 高频点击

---

### 17.3 深拷贝

**必须掌握程度：基础会用，会区分浅拷贝**

**必须理解**

- 浅拷贝只复制第一层
- 深拷贝会递归复制嵌套对象

**应用场景**

- 表单编辑前复制原数据
- 状态数据隔离

---

### 17.4 事件总线

**建议掌握程度：理解**

**应用场景**

- 模块之间通信
- 非父子组件通信思路

---

## 18. JavaScript 网络编程

### 18.1 常见网络请求能力

**必须掌握**

- `XMLHttpRequest` 要认识
- `fetch` 建议掌握
- 知道请求和响应的基本流程

**学习示例**

```js
fetch("https://api.example.com/users")
  .then((res) => res.json())
  .then((data) => {
    console.log(data);
  });
```

**应用场景**

- 获取列表数据
- 提交表单
- 登录注册
- 加载页面信息

---

## 19. JavaScript02 必须掌握到什么程度

### 19.1 必须会默写

- `this` 常见绑定规则
- 闭包基础示例
- `call` / `apply` / `bind`
- ES6 常见语法
- `Promise` 基础写法
- `async` / `await`
- 事件循环基础顺序题
- `localStorage` 基础操作
- 正则基础校验写法
- 防抖 / 节流基础实现思路

### 19.2 必须会独立写

- 一个 `this` 指向判断示例
- 一个闭包计数器
- 一个 `Promise` 封装异步任务
- 一个 `async / await` 请求示例
- 一个手机号正则校验
- 一个基础防抖函数
- 一个基础节流函数

### 19.3 必须能看懂并改动

- ES6+ 项目代码
- Promise 链式调用
- `async / await` 异步逻辑
- `class` 继承代码
- 常见对象和数组解构代码

---

## 20. 推荐学习顺序

建议按这个顺序学：

1. `this` 指向
2. 执行上下文、作用域、运行原理
3. 内存管理和闭包
4. 函数增强知识
5. 对象增强知识
6. 原型链和继承
7. ES6+ 新特性
8. Promise
9. `async` / `await`
10. 事件循环
11. `storage`
12. 正则表达式
13. 防抖 / 节流 / 深拷贝
14. 网络编程
15. Proxy / Reflect
16. Iterator / Generator

---

## 21. 一组最实用的综合示例

下面这个例子把 JavaScript02 的几个核心知识点串起来了：

```js
function debounce(fn, delay) {
  let timer = null;

  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

function search(keyword) {
  console.log("发送请求:", keyword);
}

const handleSearch = debounce(search, 500);

handleSearch("v");
handleSearch("vu");
handleSearch("vue");
```

这个例子涉及：

- 闭包
- 定时器
- 高阶函数
- 剩余参数
- `this`
- `apply`

如果你能自己把这个例子写出来并解释清楚，说明 JavaScript02 的基础已经进阶了不少。

---

## 22. 每个知识点的典型应用场景速查

| 知识点 | 典型场景 |
| --- | --- |
| `this` | 对象方法、类方法、回调函数 |
| 闭包 | 私有变量、防抖节流、模块封装 |
| 原型链 | 理解继承和实例方法来源 |
| `class` 继承 | 面向对象封装 |
| ES6+ | 现代项目代码书写 |
| Promise | 异步请求、异步流程控制 |
| `async` / `await` | 接口请求、串行异步逻辑 |
| 事件循环 | 理解执行顺序、排查异步问题 |
| Proxy | Vue3 响应式、对象代理 |
| storage | token、本地缓存、配置保存 |
| 正则 | 手机号、邮箱、密码校验 |
| 防抖 | 搜索框、输入联想 |
| 节流 | 滚动监听、拖拽、鼠标移动 |
| 深拷贝 | 表单副本、状态隔离 |
| `fetch` / 网络请求 | 接口获取、数据提交 |

---

## 23. 给你的学习建议

### 23.1 第一阶段：先攻克最难但最重要的原理

优先吃透：

- `this`
- 闭包
- 原型链
- Promise
- `async / await`
- 事件循环

这些内容是 JavaScript 进阶的核心。

### 23.2 第二阶段：用现代语法替换旧写法

你要开始习惯使用：

- `const` / `let`
- 箭头函数
- 解构
- 展开运算符
- 模板字符串
- `class`
- `async / await`

### 23.3 第三阶段：把知识点落到项目能力上

建议反复练这些小能力：

- 防抖
- 节流
- Promise 封装
- fetch 请求
- storage 缓存
- 正则校验

---

## 24. 最后总结

如果只用一句话概括 JavaScript02 阶段最重要的任务，那就是：

`从“会写基础交互”升级为“理解 JavaScript 原理、会写现代语法、会处理异步流程”。`

优先掌握下面这些内容：

- `this`
- 闭包
- 原型链和 `class` 继承
- ES6+ 常用新特性
- Promise
- `async` / `await`
- 事件循环
- `storage`
- 正则表达式
- 防抖 / 节流
- 网络请求

这些内容掌握后，你就已经具备继续学习前端框架、Node.js、工程化和源码原理的 JavaScript 进阶基础了。

