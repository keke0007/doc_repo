# JavaScript01 知识点学习梳理

本文根据 `JavaScript01` 文件夹中的课程主题整理，覆盖内容包括：

- 邂逅 JavaScript
- JavaScript 基础语法
- 变量和数据类型
- 常见运算符
- 分支语句和逻辑运算符
- `while` / `for` 循环
- 函数
- 事件监听
- 面向对象
- 内置类
- DOM 操作
- BOM 操作
- DOM 实战案例

目标是把这些内容重组成一份适合学习、复习和查漏补缺的文档。

---

## 1. JavaScript01 主要学什么

一句话理解：

`JavaScript01 这一阶段，主要是在学“让网页动起来”的基础能力。`

它解决的问题包括：

- 让网页响应用户操作
- 处理数据和逻辑
- 操作页面元素
- 实现交互效果

你可以把它看成从“会写静态页面”到“会写交互页面”的过渡阶段。

---

## 2. 这套 JavaScript01 内容的学习主线

这套内容大致可以分成 6 个层次：

1. 认识 JavaScript 和基本语法
2. 掌握变量、数据类型、运算符
3. 掌握分支、循环、函数这些核心逻辑能力
4. 理解事件监听和用户交互
5. 学会操作 DOM，真正控制页面
6. 了解 BOM、内置类和基础面向对象

如果你是初学者，可以先抓住这一句：

`JavaScript 的核心是：数据 + 逻辑 + 交互 + DOM 操作。`

---

## 3. 哪些知识点是必须掌握的

下面这些内容，是你后面继续学 JavaScript 进阶、Vue、React 时一定要打牢的基础。

---

## 4. JavaScript 基础入门

### 4.1 JavaScript 是什么

**必须掌握**

- JavaScript 是运行在浏览器中的脚本语言
- 它可以和 HTML、CSS 一起完成网页交互
- HTML 负责结构，CSS 负责样式，JavaScript 负责行为

**应用场景**

- 点击按钮切换内容
- 登录校验
- 轮播图
- 弹窗
- 表单提交

---

### 4.2 JavaScript 基础语法

**必须掌握**

- 语句
- 注释
- 大小写敏感
- 分号
- 代码块

**学习示例**

```js
// 这是单行注释
/*
  这是多行注释
*/

console.log("Hello JavaScript");
```

**应用场景**

- 所有 JavaScript 编写的基础

---

## 5. 变量和数据类型

### 5.1 变量

**必须掌握**

- `let`
- `const`
- `var` 要认识，但现代开发优先用 `let` / `const`

**学习示例**

```js
let age = 18;
const name = "Tom";
```

**应用场景**

- 保存用户输入
- 保存接口返回数据
- 保存状态值

**必须理解**

- 会变的数据一般用 `let`
- 不希望重新赋值的数据优先用 `const`

---

### 5.2 数据类型

**必须掌握**

基础类型常见有：

- `number`
- `string`
- `boolean`
- `undefined`
- `null`

引用类型先重点认识：

- `object`
- `array`
- `function`

**学习示例**

```js
let score = 100;
let username = "小明";
let isLogin = true;
let info = null;
let city;
```

**应用场景**

- 字符串用于用户名、标题、提示语
- 数字用于价格、数量、分数
- 布尔值用于开关状态、是否登录

---

### 5.3 类型判断

**必须掌握**

- `typeof`

**学习示例**

```js
console.log(typeof 123); // number
console.log(typeof "abc"); // string
console.log(typeof true); // boolean
```

**应用场景**

- 调试数据
- 判断传入值类型
- 排查逻辑错误

---

## 6. 运算符

### 6.1 算术运算符

**必须掌握**

- `+`
- `-`
- `*`
- `/`
- `%`

**学习示例**

```js
let total = 100 + 20;
let remain = 10 % 3;
```

**应用场景**

- 价格计算
- 分页计算
- 奇偶判断

---

### 6.2 赋值和比较运算符

**必须掌握**

- `=`
- `==`
- `===`
- `!=`
- `!==`
- `>`
- `<`
- `>=`
- `<=`

**学习示例**

```js
let a = 10;
console.log(a === 10);
console.log(a > 5);
```

**必须理解**

- 优先使用 `===` 和 `!==`
- `=` 是赋值，不是比较

**应用场景**

- 判断登录状态
- 判断成绩等级
- 判断输入是否合法

---

### 6.3 逻辑运算符

**必须掌握**

- `&&`
- `||`
- `!`

**学习示例**

```js
let age = 20;
let isStudent = true;

console.log(age > 18 && isStudent);
```

**应用场景**

- 多条件判断
- 登录校验
- 权限判断

---

## 7. 分支语句

### 7.1 `if` / `else`

**必须掌握**

- 单分支
- 双分支
- 多分支

**学习示例**

```js
let score = 85;

if (score >= 90) {
  console.log("优秀");
} else if (score >= 60) {
  console.log("及格");
} else {
  console.log("不及格");
}
```

**应用场景**

- 成绩判断
- 登录状态切换
- 表单校验

---

### 7.2 `switch`

**必须掌握**

- 基本语法
- `break`

**学习示例**

```js
let day = 1;

switch (day) {
  case 1:
    console.log("星期一");
    break;
  case 2:
    console.log("星期二");
    break;
  default:
    console.log("其他");
}
```

**应用场景**

- 菜单类型判断
- 状态值映射
- 多个固定选项的处理

---

## 8. 循环

### 8.1 `while`

**必须掌握**

- 循环条件
- 防止死循环

**学习示例**

```js
let i = 1;

while (i <= 5) {
  console.log(i);
  i++;
}
```

**应用场景**

- 基础逻辑练习
- 条件不确定次数的循环

---

### 8.2 `for`

**必须掌握**

- 初始化
- 条件判断
- 步进表达式

**学习示例**

```js
for (let i = 0; i < 5; i++) {
  console.log(i);
}
```

**应用场景**

- 遍历数组
- 生成列表
- 批量创建内容

**必须理解**

- `for` 是最常用的基础循环

---

### 8.3 `break` 和 `continue`

**必须掌握**

- `break`：直接结束循环
- `continue`：跳过本次循环

**应用场景**

- 查找某个目标后停止
- 跳过无效数据

---

## 9. 函数

### 9.1 函数的定义和调用

**必须掌握**

- 函数声明
- 参数
- 返回值
- 调用

**学习示例**

```js
function add(a, b) {
  return a + b;
}

let result = add(10, 20);
console.log(result);
```

**应用场景**

- 封装重复逻辑
- 计算数据
- 响应事件

**必须建立的意识**

- 重复代码要尝试抽成函数

---

### 9.2 函数表达式和箭头函数

**必须掌握**

- 函数表达式
- 箭头函数基础写法

**学习示例**

```js
const sayHi = function () {
  console.log("Hi");
};

const multiply = (a, b) => a * b;
```

**应用场景**

- 回调函数
- 数组处理
- 事件处理

---

## 10. 数组和对象

### 10.1 数组

**必须掌握**

- 创建数组
- 通过索引取值
- 修改值
- `length`

**学习示例**

```js
let fruits = ["苹果", "香蕉", "橙子"];
console.log(fruits[0]);
console.log(fruits.length);
```

**应用场景**

- 商品列表
- 用户列表
- 菜单数据

---

### 10.2 对象

**必须掌握**

- 属性
- 方法
- 取值与赋值

**学习示例**

```js
let user = {
  name: "小明",
  age: 18,
  sayHello() {
    console.log("你好");
  }
};

console.log(user.name);
user.sayHello();
```

**应用场景**

- 用户信息
- 商品信息
- 配置项

---

## 11. 事件监听

### 11.1 事件绑定

**必须掌握**

- `onclick`
- `addEventListener`

**学习示例**

```js
const btn = document.querySelector(".btn");

btn.addEventListener("click", function () {
  console.log("按钮被点击了");
});
```

**应用场景**

- 按钮点击
- 输入框输入
- 鼠标移入移出
- 页面加载后的初始化逻辑

**必须理解**

- 现代开发更推荐 `addEventListener`

---

### 11.2 常见事件类型

**必须掌握**

- `click`
- `input`
- `change`
- `focus`
- `blur`
- `keydown`

**应用场景**

- 点击交互
- 实时输入校验
- 表单处理
- 键盘监听

---

## 12. DOM 操作

### 12.1 获取元素

**必须掌握**

- `document.getElementById`
- `document.querySelector`
- `document.querySelectorAll`

**学习示例**

```js
const title = document.querySelector(".title");
const items = document.querySelectorAll(".item");
```

**应用场景**

- 获取按钮、标题、列表项
- 后续修改内容和样式

---

### 12.2 操作内容

**必须掌握**

- `innerText`
- `textContent`
- `innerHTML`

**学习示例**

```js
const box = document.querySelector(".box");
box.innerText = "新的内容";
```

**应用场景**

- 修改标题
- 更新提示信息
- 动态生成内容

---

### 12.3 操作属性

**必须掌握**

- `src`
- `href`
- `value`
- `className`
- `setAttribute`
- `getAttribute`

**学习示例**

```js
const img = document.querySelector("img");
img.src = "./images/new.png";
```

**应用场景**

- 切换图片
- 修改链接
- 获取输入框内容

---

### 12.4 操作样式

**必须掌握**

- `style`
- `classList.add`
- `classList.remove`
- `classList.toggle`

**学习示例**

```js
const card = document.querySelector(".card");
card.classList.add("active");
```

**应用场景**

- 高亮当前项
- 显示隐藏模块
- 切换主题样式

**必须建立的意识**

- 简单样式可直接改 `style`
- 更推荐通过切换类名来控制样式

---

### 12.5 创建、插入、删除节点

**必须掌握**

- `createElement`
- `appendChild`
- `remove`

**学习示例**

```js
const li = document.createElement("li");
li.innerText = "新列表项";

const ul = document.querySelector("ul");
ul.appendChild(li);
```

**应用场景**

- 动态添加评论
- 动态生成商品列表
- 删除待办事项

---

## 13. DOM 实战能力

### 13.1 交互逻辑组合

**必须掌握**

你要能把下面这些能力组合起来：

- 获取元素
- 监听事件
- 修改内容
- 切换样式
- 动态创建元素

**典型场景**

- 选项卡切换
- 简单轮播图
- 待办事项列表
- 登录框校验
- 点赞按钮

---

## 14. BOM 操作

### 14.1 什么是 BOM

**必须掌握**

- BOM 是浏览器对象模型
- 常见对象：`window`、`location`、`history`

---

### 14.2 常见 BOM 能力

**必须掌握**

- `alert`
- `confirm`
- `prompt`
- `setTimeout`
- `setInterval`
- `location.href`

**学习示例**

```js
setTimeout(function () {
  console.log("3 秒后执行");
}, 3000);
```

```js
location.href = "https://www.example.com";
```

**应用场景**

- 延迟执行
- 倒计时
- 页面跳转
- 提示框

---

## 15. 面向对象和内置类

### 15.1 面向对象基础

**建议掌握程度：理解**

需要知道：

- 对象是什么
- 属性和方法
- `this` 的基础概念

**应用场景**

- 组织复杂数据
- 封装相关行为

---

### 15.2 常见内置类

**建议掌握程度：必须会常用部分**

重点先掌握：

- `String`
- `Number`
- `Math`
- `Date`
- `Array`

**学习示例**

```js
console.log("hello".length);
console.log(Math.max(10, 20, 30));
console.log(new Date());
```

**应用场景**

- 字符串处理
- 数学计算
- 日期展示
- 数组操作

---

## 16. JavaScript01 必须掌握到什么程度

### 16.1 必须会默写

- 变量声明
- 常见数据类型
- `if / else`
- `for`
- 函数定义
- 事件监听基础
- DOM 获取元素
- DOM 修改内容
- DOM 修改类名
- `setTimeout`

### 16.2 必须会独立写

- 一个点击按钮修改文字的小案例
- 一个输入框实时显示内容的小案例
- 一个列表循环输出案例
- 一个选项卡切换案例
- 一个简单登录校验案例
- 一个动态添加列表项案例

### 16.3 必须能看懂并改动

- 基础 DOM 交互代码
- 基础事件监听代码
- 基础循环和条件判断
- 基础数组、对象代码

---

## 17. 推荐学习顺序

建议按这个顺序学：

1. 认识 JavaScript
2. 基础语法
3. 变量和数据类型
4. 运算符
5. 分支语句
6. 循环
7. 函数
8. 数组和对象
9. 事件监听
10. DOM 操作
11. DOM 实战案例
12. BOM
13. 面向对象和内置类

---

## 18. 一组最实用的综合示例

下面这个例子把 JavaScript01 的核心知识点串起来了：

```html
<input class="input" placeholder="请输入任务" />
<button class="btn">添加</button>
<ul class="list"></ul>
```

```js
const input = document.querySelector(".input");
const btn = document.querySelector(".btn");
const list = document.querySelector(".list");

btn.addEventListener("click", function () {
  const value = input.value;

  if (value === "") {
    alert("请输入内容");
    return;
  }

  const li = document.createElement("li");
  li.innerText = value;
  list.appendChild(li);

  input.value = "";
});
```

这个例子覆盖了：

- 获取元素
- 事件监听
- 变量
- 条件判断
- DOM 创建节点
- DOM 插入节点
- 输入框取值

如果你能自己写出这个例子，说明 JavaScript01 的基础已经比较稳了。

---

## 19. 每个知识点的典型应用场景速查

| 知识点 | 典型场景 |
| --- | --- |
| 变量 | 保存状态、保存输入、保存结果 |
| 数据类型 | 用户信息、价格、开关状态 |
| 运算符 | 计算价格、判断条件、组合逻辑 |
| `if / else` | 登录校验、状态判断 |
| `for / while` | 遍历列表、批量处理数据 |
| 函数 | 封装重复逻辑 |
| 数组 | 商品列表、菜单列表 |
| 对象 | 用户对象、商品对象 |
| 事件监听 | 点击、输入、提交、聚焦 |
| DOM 获取元素 | 找到页面中的目标元素 |
| DOM 操作内容 | 修改标题、提示语、列表内容 |
| DOM 操作样式 | 高亮、显示隐藏、切换类名 |
| DOM 节点操作 | 动态增删列表、评论、卡片 |
| BOM | 跳转页面、延时、定时器 |
| 内置类 | 字符串处理、时间处理、数学计算 |

---

## 20. 给你的学习建议

### 20.1 第一阶段：先把“语法和逻辑”练熟

先重点练：

- 变量
- 类型
- 运算符
- 分支
- 循环
- 函数

这部分是后面所有 JavaScript 学习的地基。

### 20.2 第二阶段：尽快进入 DOM

很多人只会写控制台练习，不会做网页交互。要尽快把知识点落到页面上：

- 点击按钮
- 获取输入框内容
- 改文字
- 改样式
- 加节点

### 20.3 第三阶段：多做小案例

建议反复练这些小案例：

- 计数器
- 登录校验
- 选项卡
- 动态列表
- 待办事项

---

## 21. 最后总结

如果只用一句话概括 JavaScript01 阶段最重要的任务，那就是：

`先学会用 JavaScript 处理数据和逻辑，再学会通过 DOM 把这些逻辑作用到页面上。`

优先掌握下面这些内容：

- 基础语法
- 变量和数据类型
- 运算符
- 分支和循环
- 函数
- 数组和对象
- 事件监听
- DOM 操作
- BOM 基础

这些内容掌握后，你就已经具备继续学习 JavaScript 进阶、ES6、异步编程和前端框架的基础了。

