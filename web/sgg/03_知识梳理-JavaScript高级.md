# JavaScript 高级知识梳理

> 文档生成日期: 2026-05-25

---

## 目录

1. [BFC 与浮动](#bfc-与浮动)
2. [JS 内置对象](#js-内置对象)
3. [引用类型与值传递的本质](#引用类型与值传递的本质)
4. [BOM](#bom)
5. [DOM 树与节点关系](#dom-树与节点关系)
6. [DOM 查询 API](#dom-查询-api)
7. [DOM 操作](#dom-操作)
8. [DOM 事件模型](#dom-事件模型)
9. [原型与原型链](#原型与原型链)
10. [继承的多种实现](#继承的多种实现)
11. [闭包](#闭包)
12. [this 绑定规则](#this-绑定规则)
13. [执行上下文与作用域链](#执行上下文与作用域链)
14. [同步异步与事件循环](#同步异步与事件循环)
15. [易错速查](#易错速查)

---

## BFC 与浮动

### 什么是 BFC

**Block Formatting Context (BFC)** 即块级格式上下文。

#### W3C 定义
浮动、绝对定位元素、不是块盒子的块容器（如inline-blocks、table-cells 和 table-captions），以及 `overflow` 属性值除 `visible` 以外的块盒，将为其内容建立新的块格式化上下文。

#### MDN 定义
块格式化上下文是 Web 页面可视 CSS 渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。

#### 简化理解
创建 BFC 的元素可视为一个独立的容器，容器内的元素布局不会影响外面。

### 创建 BFC 的方式

- 根元素（html）
- 浮动元素（float 不为 none）
- 绝对定位或固定定位的元素（position: absolute/fixed）
- 行内块元素（display: inline-block）
- 表格相关元素（table、tr、td、thead、tbody、tfoot、caption）
- `overflow` 值不为 `visible` 的块元素
- 伸缩项目（display: flex/inline-flex 的直接子元素）
- 网格项目（display: grid/inline-grid 的直接子元素）
- `column-span: all` 的元素

### BFC 的应用

#### 1. 清除子元素浮动影响

```css
.parent {
    overflow: auto;  /* 或 overflow: hidden */
    /* 现在父元素高度会包括浮动子元素 */
}
```

#### 2. 解决外边距塌陷

```css
.parent {
    overflow: auto;  /* 创建 BFC */
}

/* 第一个和最后一个子元素的外边距不会塌陷 */
```

#### 3. 阻止元素被浮动元素覆盖

```css
.container {
    overflow: auto;  /* 创建 BFC，不会被浮动元素覆盖 */
}
```

---

## JS 内置对象

### 1. Boolean

```javascript
// 直接量
true; false;

// Boolean 函数（返回原始值）
Boolean(0);           // false
Boolean(1);           // true
Boolean('');          // false
Boolean('任何字符串'); // true

// Boolean 构造函数（返回对象）
new Boolean(false);  // [Boolean: false] 对象
```

### 2. Number

#### 实例方法

| 方法 | 说明 |
|-----|------|
| `toFixed(n)` | 返回 n 位小数的字符串（四舍五入） |
| `toString(radix)` | 返回指定进制的字符串（2~36） |
| `toPrecision(n)` | 返回 n 位有效数字 |

#### 静态属性

| 属性 | 值 |
|-----|-----|
| `Number.MAX_VALUE` | 1.7976931348623157e+308 |
| `Number.MIN_VALUE` | 5e-324 |
| `Number.POSITIVE_INFINITY` | Infinity |
| `Number.NEGATIVE_INFINITY` | -Infinity |
| `Number.NaN` | NaN |

#### 静态方法

```javascript
Number.isNaN(value)      // 判断是否是 NaN
Number.isFinite(value)   // 判断是否是有限数
Number.parseInt(str)     // 字符串转整数
Number.parseFloat(str)   // 字符串转浮点数
```

### 3. String

#### 实例属性和方法

| 属性/方法 | 说明 |
|----------|------|
| `length` | 字符串长度 |
| `charAt(index)` | 返回指定位置的字符 |
| `indexOf(str)` | 首次出现的位置，不存在返回 -1 |
| `lastIndexOf(str)` | 最后出现的位置 |
| `slice(start, end)` | 截取子字符串 |
| `substring(start, end)` | 截取子字符串（不支持负数） |
| `substr(start, length)` | 按长度截取（已弃用） |
| `split(separator)` | 按分隔符分割成数组 |
| `toUpperCase()` | 转大写 |
| `toLowerCase()` | 转小写 |
| `charCodeAt(index)` | 返回字符的 Unicode 编码 |
| `trim()` | 去除首尾空白 |
| `repeat(count)` | 重复字符串 |
| `startsWith(str)` | 是否以...开头 |
| `endsWith(str)` | 是否以...结尾 |
| `includes(str)` | 是否包含子字符串 |

#### 静态方法

```javascript
String.fromCharCode(code)    // Unicode 编码转字符
String.fromCodePoint(code)   // 码点转字符（支持更大范围）
```

### 4. Math

#### 常用属性

```javascript
Math.PI        // 3.141592653589793
Math.E         // 2.718281828459045
```

#### 常用方法

| 方法 | 说明 |
|-----|------|
| `Math.abs(x)` | 绝对值 |
| `Math.pow(x, y)` | x 的 y 次方 |
| `Math.sqrt(x)` | 平方根 |
| `Math.floor(x)` | 向下取整 |
| `Math.ceil(x)` | 向上取整 |
| `Math.round(x)` | 四舍五入 |
| `Math.max(...args)` | 最大值 |
| `Math.min(...args)` | 最小值 |
| `Math.random()` | 随机数 [0, 1) |

#### 取随机整数

```javascript
// 随机 [0, n] 的整数
Math.floor(Math.random() * (n + 1))

// 随机 [m, n] 的整数
Math.floor(Math.random() * (n - m + 1)) + m

// 随机 [1, 10] 的整数
Math.floor(Math.random() * 10) + 1
```

> ⚠️ 原文有误: `Math.round()` 对负数处理特殊
> 
> 错误认知: 负数也是四舍五入
> 
> 正确理解:
> - `Math.round(-1.5)` 返回 **-1**（向上取整）
> - `Math.round(-1.4)` 返回 **-1**（向下取整）
> - `Math.round(-2.5)` 返回 **-2**（向上取整）
> 
> 原因: `Math.round()` 对 0.5 的处理是向正无穷方向

### 5. Date

#### 创建日期对象

```javascript
new Date()                           // 当前日期和时间
new Date('2025-05-25')              // 指定日期字符串
new Date('2025-05-25T10:30:00')     // ISO 8601 格式
new Date(2025, 4, 25)               // 年、月(0-11)、日
new Date(2025, 4, 25, 10, 30, 45)   // 包含时分秒
new Date(1747737600000)             // Unix 时间戳（毫秒）
```

#### 获取日期时间

| 方法 | 说明 | 范围 |
|-----|------|------|
| `getFullYear()` | 获取年 | - |
| `getMonth()` | 获取月 | **0-11** |
| `getDate()` | 获取日 | 1-31 |
| `getDay()` | 获取周几 | 0-6（0为周日） |
| `getHours()` | 获取时 | 0-23 |
| `getMinutes()` | 获取分 | 0-59 |
| `getSeconds()` | 获取秒 | 0-59 |
| `getMilliseconds()` | 获取毫秒 | 0-999 |
| `getTime()` | 时间戳 | - |

#### 设置日期时间

```javascript
date.setFullYear(2025)
date.setMonth(4)       // 5月（0-11）
date.setDate(25)
date.setHours(10)
date.setMinutes(30)
date.setSeconds(45)
date.setTime(1747737600000)
```

#### 静态方法

```javascript
Date.now()                  // 当前时间戳
Date.UTC(2025, 4, 25)       // 指定日期的 UTC 时间戳
```

> ⚠️ 原文有误: Date 月份从 0 开始容易混淆
> 
> 错误代码:
> ```javascript
> new Date(2025, 5, 25)  // 误以为是5月，实际是6月
> ```
> 
> 正确用法:
> ```javascript
> new Date(2025, 4, 25)  // 这才是5月
> // getMonth() 返回 4 代表 5月
> ```

### 6. Array

#### 访问器方法（不修改原数组）

| 方法 | 说明 | 返回值 |
|-----|------|--------|
| `concat(...args)` | 连接多个数组 | 新数组 |
| `slice(start, end)` | 截取部分数组 | 新数组 |
| `join(separator)` | 数组转字符串 | 字符串 |
| `indexOf(item)` | 首次出现的位置 | 索引或 -1 |
| `lastIndexOf(item)` | 最后出现的位置 | 索引或 -1 |
| `includes(item)` | 是否包含该元素 | 布尔值 |
| `forEach(callback)` | 遍历数组 | undefined |
| `map(callback)` | 映射转换 | 新数组 |
| `filter(callback)` | 过滤元素 | 新数组 |
| `reduce(callback, initial)` | 累计运算 | 最终值 |
| `find(callback)` | 查找第一个匹配 | 元素或 undefined |
| `findIndex(callback)` | 查找第一个位置 | 索引或 -1 |
| `every(callback)` | 全部满足条件 | 布尔值 |
| `some(callback)` | 至少一个满足 | 布尔值 |

#### 修改器方法（修改原数组）

| 方法 | 说明 | 返回值 |
|-----|------|--------|
| `push(...items)` | 末尾添加元素 | 新长度 |
| `unshift(...items)` | 开头添加元素 | 新长度 |
| `pop()` | 删除末尾元素 | 被删除的元素 |
| `shift()` | 删除开头元素 | 被删除的元素 |
| `splice(index, count, ...items)` | 删除/插入 | 删除的元素数组 |
| `reverse()` | 反转数组 | 反转后的数组 |
| `sort(compareFn)` | 排序数组 | 排序后的数组 |
| `fill(value, start, end)` | 填充数组 | 修改后的数组 |

```javascript
// reduce 示例：求和
const sum = [1, 2, 3, 4].reduce((acc, cur) => acc + cur, 0)  // 10

// filter + map 示例：取偶数的平方
const result = [1, 2, 3, 4, 5, 6]
  .filter(n => n % 2 === 0)
  .map(n => n * n)  // [4, 16, 36]
```

---

## 引用类型与值传递的本质

### 值类型 vs 引用类型

#### 值类型（原始类型）

```
Number、String、Boolean、null、undefined、Symbol、BigInt
```

特点:
- 存储在**栈内存**中
- 传参是**值传递**
- **不可变**：无法修改其中一部分
- **判等**只看值

```javascript
let a = 10;
let b = a;
b = 20;
console.log(a);  // 10（不受影响）

// 字符串不可变
let str = 'hello';
str[0] = 'H';
console.log(str);  // 'hello'（未改变）
```

#### 引用类型

```
Object、Array、Function、Date、RegExp、Map、Set 等
```

特点:
- 存储在**堆内存**中，栈中存**地址**
- 传参是**引用传递**（地址传递）
- **可变**：可以修改属性
- **判等**看地址

```javascript
let obj1 = { x: 10 };
let obj2 = obj1;
obj2.x = 20;
console.log(obj1.x);  // 20（受影响）

// 地址相同
console.log(obj1 === obj2);  // true

// 地址不同
let obj3 = { x: 20 };
console.log(obj1 === obj3);  // false
```

### 内存模型图

```
┌─────────────────────────────────────────────────────────┐
│                     JavaScript 内存                      │
├──────────────────────┬──────────────────────────────────┤
│      栈内存          │        堆内存                    │
│  (Execution Stack)   │   (Memory Heap)                │
├──────────────────────┼──────────────────────────────────┤
│  let a = 10          │                                 │
│  [a: 10]             │                                 │
│                      │                                 │
│  let obj = {...}     │  ┌─────────────────┐           │
│  [obj: Addr_001]────────→ { x: 10, y: 20 } │           │
│                      │  └─────────────────┘           │
│                      │                                 │
│  let arr = [1,2]     │  ┌─────────────────┐           │
│  [arr: Addr_002]────────→ [1, 2, 3]        │           │
│                      │  └─────────────────┘           │
└──────────────────────┴──────────────────────────────────┘
```

### 传值过程

```
函数调用: func(a, obj)

栈内存：
┌─────────────────────┐
│ 全局作用域          │
│ a = 10              │
│ obj = Addr_001 ──┐  │
├─────────────────────┤ → 堆: {name: 'John'}
│ 函数执行上下文      │  │
│ param_a = 10    │  │
│ param_obj = Addr_001
└─────────────────────┘

结论：值类型复制值，引用类型复制地址
```

> ⚠️ 原文有误: 引用传值容易导致错误理解
> 
> 误区: "对象是按引用传值"意味着可以替换外层对象
> 
> 正确理解:
> ```javascript
> let obj = {x: 1};
> 
> function change(o) {
>     o.x = 2;        // ✓ 有效：修改对象属性
>     o = {x: 3};     // ✗ 无效：替换局部引用
> }
> 
> change(obj);
> console.log(obj.x);  // 2（不是3）
> // 因为函数内的 o = {x: 3} 只改变了局部变量
> ```

---

## BOM

### window 对象

`window` 是浏览器全局对象，所有全局变量都是 window 的属性。

#### 属性

| 属性 | 说明 |
|-----|------|
| `name` | 窗口名称（可读写） |
| `innerWidth` | 视口宽度（包含滚动条） |
| `innerHeight` | 视口高度（包含滚动条） |
| `outerWidth` | 浏览器窗口宽度 |
| `outerHeight` | 浏览器窗口高度 |
| `screenX, screenY` | 窗口相对屏幕的位置 |
| `scrollX, scrollY` | 页面滚动位置 |
| `document` | 文档对象 |
| `location` | 地址对象 |
| `history` | 历史记录对象 |
| `navigator` | 浏览器信息对象 |
| `screen` | 屏幕信息对象 |

#### 弹框方法

```javascript
alert('提示信息');           // 警告框，无返回值
confirm('确认吗?');          // 确认框，返回 true/false
const input = prompt('输入'); // 输入框，返回字符串或 null
```

#### 窗口控制

```javascript
window.open('url', 'name', 'width=800,height=600')  // 打开新窗口
window.close()              // 关闭当前窗口
```

#### 页面滚动

```javascript
// 滚动到指定位置
window.scrollTo(0, 0)
window.scrollTo({ left: 0, top: 0, behavior: 'smooth' })

// 滚动指定距离
window.scrollBy(0, 100)
window.scrollBy({ top: 100, behavior: 'smooth' })
```

#### 定时器

```javascript
// 重复执行
const id = setInterval(callback, 1000)
clearInterval(id)

// 单次执行
const id = setTimeout(callback, 1000)
clearTimeout(id)
```

> ⚠️ 原文有误: setTimeout 最小延迟有限制
> 
> 误区: setTimeout(fn, 0) 表示立即执行
> 
> 正确理解:
> ```javascript
> // 嵌套定时器的最小延迟是 4ms
> setTimeout(() => {
>     setTimeout(() => {
>         console.log('执行');
>     }, 0);
> }, 0);
> // 实际延迟会被调整为 4ms 左右
> ```

### location 对象

表示当前页面的 URL 信息。

| 属性 | 说明 | 示例 |
|-----|------|------|
| `href` | 完整 URL | `http://example.com:8080/path?id=1#top` |
| `protocol` | 协议 | `http:` |
| `hostname` | 主机名 | `example.com` |
| `host` | 主机名+端口 | `example.com:8080` |
| `port` | 端口 | `8080` |
| `pathname` | 路径 | `/path` |
| `search` | 查询字符串 | `?id=1` |
| `hash` | 片段标识符 | `#top` |

#### 方法

```javascript
location.reload()          // 刷新页面
location.assign(url)       // 跳转并保留历史
location.replace(url)      // 跳转不保留历史
location.reload(true)      // 强制清除缓存刷新
```

### history 对象

表示浏览器的历史记录。

| 属性/方法 | 说明 |
|----------|------|
| `length` | 历史记录数量 |
| `back()` | 后退（同浏览器后退按钮） |
| `forward()` | 前进 |
| `go(n)` | 前进/后退 n 步 |
| `go(-1)` | 后退 1 步 |

### navigator 对象

浏览器和操作系统信息。

| 属性 | 说明 |
|-----|------|
| `userAgent` | 用户代理字符串 |
| `platform` | 操作系统平台 |
| `language` | 浏览器语言 |
| `onLine` | 是否在线 |

```javascript
// 判断浏览器类型
if (navigator.userAgent.includes('Chrome')) {
    console.log('Chrome 浏览器');
}
```

### screen 对象

屏幕信息。

| 属性 | 说明 |
|-----|------|
| `width` | 屏幕宽度 |
| `height` | 屏幕高度 |
| `availWidth` | 可用宽度（去除任务栏等） |
| `availHeight` | 可用高度 |

---

## DOM 树与节点关系

### 五大节点类型

| 节点类型 | nodeType | nodeName | 说明 |
|---------|---------|---------|------|
| Document | 9 | `#document` | 文档节点 |
| Element | 1 | 标签名 | 元素节点（如 div、p） |
| Attribute | 2 | 属性名 | 属性节点 |
| Text | 3 | `#text` | 文本节点 |
| Comment | 8 | `#comment` | 注释节点 |

### 节点属性

```javascript
node.nodeName       // 节点名称
node.nodeValue      // 节点值
node.nodeType       // 节点类型（1-9）
node.parentNode     // 父节点
node.childNodes     // 所有子节点（包含文本节点）
node.firstChild     // 第一个子节点
node.lastChild      // 最后一个子节点
node.nextSibling    // 下一个兄弟节点
node.previousSibling // 前一个兄弟节点
```

### DOM 树结构图

```
                    Document (nodeType: 9)
                        │
                      <html>
                   ┌─────┴─────┐
                <head>        <body>
                 │             │
            ┌────┴────┐    ┌──┴──┬────┐
         <meta>    <title> <div> <p>  <span>
          (属性)    (文本)
                    │
              "页面标题"(#text)
```

### 元素节点间的关系操作

```javascript
// 子元素关系（仅元素，不含文本节点）
element.children              // HTMLCollection
element.firstElementChild     // 第一个子元素
element.lastElementChild      // 最后一个子元素

// 父元素关系
element.parentElement         // 父元素

// 兄弟元素关系
element.previousElementSibling  // 前一个兄弟元素
element.nextElementSibling      // 后一个兄弟元素
```

---

## DOM 查询 API

### 按 ID 查询

```javascript
document.getElementById('myId')
// 返回单个元素或 null
```

### 按标签名查询

```javascript
document.getElementsByTagName('div')        // 整个文档
parentElement.getElementsByTagName('div')  // 指定元素内

// 返回 HTMLCollection（动态集合）
```

### 按类名查询

```javascript
document.getElementsByClassName('active')
parentElement.getElementsByClassName('active')

// 返回 HTMLCollection（IE8 及以下不支持）
```

### 按 name 属性查询

```javascript
document.getElementsByName('username')
// 返回 NodeList（仅 document 有此方法）
```

### CSS 选择器查询 ⭐

```javascript
// 查询单个元素
document.querySelector('.box > .item')
parentElement.querySelector('div')

// 查询所有匹配元素
document.querySelectorAll('.item')
parentElement.querySelectorAll('div')

// querySelector 返回单个元素或 null
// querySelectorAll 返回 NodeList（静态集合）
```

### 快捷方式

```javascript
document.body              // <body> 元素
document.head              // <head> 元素
document.documentElement   // <html> 根元素
document.all               // 所有元素（HTMLCollection）
```

### HTMLCollection vs NodeList

```javascript
// HTMLCollection
// - 来自: getElementsByTagName, getElementsByClassName, children
// - 成员: 仅元素节点
// - 特性: 动态集合，页面变化会实时更新
// - forEach: 不支持（仅现代浏览器支持）

// NodeList
// - 来自: querySelectorAll, getElementsByName, childNodes
// - 成员: 可以是任何节点类型
// - 特性: 静态集合（querySelectorAll）或动态（childNodes）
// - forEach: 支持
```

---

## DOM 操作

### 读写内容

```javascript
element.innerHTML      // 读写内部 HTML 和文本
element.outerHTML      // 读写包括自身的 HTML
element.innerText      // 读写内部文本（剔除标签）
element.textContent    // 读写内部文本（保留空白）

// 示例
const div = document.querySelector('div');
div.innerHTML = '<p>Hello</p>';      // 解析 HTML
div.textContent = '<p>Hello</p>';    // 作为纯文本
```

### 元素尺寸

```javascript
// offset 系列：包括边框
element.offsetWidth    // 宽度（内容+padding+border）
element.offsetHeight   // 高度
element.offsetLeft     // 相对定位祖先的左距离
element.offsetTop      // 相对定位祖先的上距离
element.offsetParent   // 定位祖先元素

// client 系列：不包括边框和滚动条
element.clientWidth    // 宽度（内容+padding）
element.clientHeight   // 高度
element.clientLeft     // 左边框宽度
element.clientTop      // 上边框宽度

// scroll 系列：包括溢出部分
element.scrollWidth    // 宽度（含溢出）
element.scrollHeight   // 高度（含溢出）
element.scrollLeft     // 水平滚动位置
element.scrollTop      // 垂直滚动位置

// getBoundingClientRect：相对视口位置
const rect = element.getBoundingClientRect();
// {x, y, left, top, right, bottom, width, height}
```

### 属性操作

#### 内置属性

```javascript
// 映射到 JS 属性
element.id = 'myId'
element.className = 'class1 class2'
element.checked = true        // 复选框
element.disabled = false      // 表单控件
element.src = 'image.png'     // 图片
```

#### 自定义属性

```javascript
// 属性节点方式
element.getAttribute('data-id')
element.setAttribute('data-id', '123')
element.removeAttribute('data-id')
element.hasAttribute('data-id')

// data-* 属性（推荐）
element.dataset.id = '123'      // 对应 data-id
element.dataset.userId = '456'  // 对应 data-user-id
```

### 样式操作

#### 行内样式

```javascript
// 读取
element.style.color         // 'red'
element.style.backgroundColor  // 'blue'

// 设置
element.style.color = 'red'
element.style.width = '100px'
element.style.display = 'none'

// 删除
element.style.display = ''
```

#### 计算样式

```javascript
// 获取最终作用的样式（只读）
const computed = getComputedStyle(element);
computed.color
computed.display
computed.backgroundColor

// 注意：CSS 属性转 JS 属性（驼峰）
// background-color -> backgroundColor
```

#### 类名操作

```javascript
// 字符串操作（容易出错）
element.className = 'box active'

// classList 对象（推荐）
element.classList.add('active')
element.classList.remove('active')
element.classList.toggle('active')      // 切换
element.classList.contains('active')    // 检查
element.classList.replace('old', 'new') // 替换
```

### 节点操作

#### 创建节点

```javascript
document.createElement('div')
document.createTextNode('hello')
document.createComment('注释')
```

#### 添加节点

```javascript
// 添加到末尾
parent.appendChild(newNode)

// 插入到指定位置
parent.insertBefore(newNode, refNode)
parent.insertBefore(newNode, parent.firstChild)  // 插入开头

// 现代方法
parent.append(...nodes)     // 末尾
parent.prepend(...nodes)    // 开头
element.before(...nodes)    // 前面
element.after(...nodes)     // 后面
```

#### 删除节点

```javascript
parent.removeChild(child)
child.remove()  // 现代方法
```

#### 替换节点

```javascript
parent.replaceChild(newNode, oldNode)
oldNode.replaceWith(newNode)  // 现代方法
```

#### 克隆节点

```javascript
element.cloneNode()      // 只克隆元素，不克隆内容
element.cloneNode(true)  // 递归克隆，包括所有内容
```

### DOM 操作最佳实践

```javascript
// ✓ 好：批量操作，减少重排
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
    const li = document.createElement('li');
    li.textContent = `Item ${i}`;
    fragment.appendChild(li);
}
ul.appendChild(fragment);

// ✗ 差：每次循环都重排
for (let i = 0; i < 1000; i++) {
    const li = document.createElement('li');
    ul.appendChild(li);  // 每次都触发重排
}
```

---

## DOM 事件模型

### 事件流三阶段

事件触发过程分为三个阶段：

```
┌─────────────────────────────────────────────────────────┐
│ 1. 捕获阶段 (Capture Phase)                            │
│    window → document → html → body → target 祖先        │
│                                                          │
│ 2. 目标阶段 (Target Phase)                             │
│    事件到达目标元素                                      │
│                                                          │
│ 3. 冒泡阶段 (Bubbling Phase)                           │
│    target → 祖先 → body → html → document → window     │
└─────────────────────────────────────────────────────────┘
```

### 事件流示例图

```
HTML 结构:
<div id="outer">
    <div id="inner">
        <button>点击</button>
    </div>
</div>

事件流过程:
window
  │
document
  │
<html>
  │
<body>
  │
#outer (捕获 → 冒泡)
  │
#inner (捕获 → 冒泡)
  │
button (目标)
```

### 事件监听

#### 三种方式

```javascript
// 方式 1: HTML 属性（不推荐）
// <button onclick="handleClick()">点击</button>

// 方式 2: 元素属性（可覆盖）
element.onclick = function(event) {}
element.onclick = null  // 移除监听

// 方式 3: addEventListener（推荐）
element.addEventListener('click', handler)
element.addEventListener('click', handler, true)  // 捕获阶段
element.removeEventListener('click', handler)
```

#### 监听配置

```javascript
element.addEventListener('click', handler, {
    capture: false,     // 冒泡阶段触发（默认）
    once: true,         // 只触发一次
    passive: false      // 是否可调用 preventDefault
})
```

> ⚠️ 原文有误: 事件捕获顺序理解有误
> 
> 误区: 认为捕获阶段会跳过某些元素
> 
> 正确流程（click 按钮示例）:
> ```
> 捕获阶段: window → document → html → body → 中间元素 → button
> 目标阶段: button（同时执行捕获和冒泡监听器）
> 冒泡阶段: button → 中间元素 → body → html → document → window
> ```

### 事件对象

#### 通用属性和方法

| 属性/方法 | 说明 |
|----------|------|
| `type` | 事件类型（'click' 等） |
| `target` | 触发事件的元素 |
| `currentTarget` | 监听事件的元素 |
| `eventPhase` | 事件阶段（1=捕获, 2=目标, 3=冒泡） |
| `timeStamp` | 事件发生时间 |
| `bubbles` | 是否冒泡 |
| `cancelable` | 是否可取消 |

#### 方法

```javascript
event.preventDefault()    // 阻止浏览器默认行为
event.stopPropagation()   // 阻止事件传播（冒泡/捕获）
event.stopImmediatePropagation()  // 阻止其他监听器
```

### 常用事件

#### 鼠标事件

| 事件 | 说明 |
|-----|------|
| `click` | 单击 |
| `dblclick` | 双击 |
| `mousedown` | 按键按下 |
| `mouseup` | 按键抬起 |
| `mousemove` | 鼠标移动 |
| `mouseover` | 进入元素（含冒泡） |
| `mouseout` | 离开元素（含冒泡） |
| `mouseenter` | 进入元素（不冒泡） |
| `mouseleave` | 离开元素（不冒泡） |
| `contextmenu` | 右键菜单 |
| `wheel` | 滚轮滚动 |

#### 键盘事件

| 事件 | 说明 |
|-----|------|
| `keydown` | 按键按下（所有键） |
| `keyup` | 按键抬起 |
| `keypress` | 按键按下（仅可输入键） |

```javascript
// 获取按键信息
document.addEventListener('keydown', (e) => {
    console.log(e.key)        // '
a'、'Enter' 等
    console.log(e.keyCode)    // ASCII 码
    console.log(e.ctrlKey)    // 是否按下 Ctrl
    console.log(e.shiftKey)   // 是否按下 Shift
})
```

#### 表单事件

| 事件 | 说明 |
|-----|------|
| `focus` | 获得焦点 |
| `blur` | 失去焦点 |
| `input` | 值改变（实时） |
| `change` | 值改变（失焦时） |
| `submit` | 表单提交 |
| `reset` | 表单重置 |
| `select` | 文本被选中 |

#### 文档事件

| 事件 | 说明 |
|-----|------|
| `load` | 资源加载完毕（包括外部文件） |
| `DOMContentLoaded` | DOM 解析完毕（不含外部文件） |
| `scroll` | 滚动 |
| `resize` | 视口大小改变 |

### 事件委托

利用事件冒泡，在祖先元素上监听后代元素的事件。

```javascript
// HTML
// <ul id="list">
//   <li class="item">Item 1</li>
//   <li class="item">Item 2</li>
// </ul>

// 事件委托
const list = document.getElementById('list');
list.addEventListener('click', (e) => {
    if (e.target.classList.contains('item')) {
        console.log('点击了:', e.target.textContent);
    }
});

// 动态添加的元素也能响应事件
const li = document.createElement('li');
li.className = 'item';
li.textContent = 'Item 3';
list.appendChild(li);  // 自动具有点击事件
```

### 阻止浏览器默认行为

```javascript
// 方式 1: preventDefault
element.addEventListener('click', (e) => {
    e.preventDefault();  // 阻止默认行为
})

// 方式 2: return false（仅限属性方式）
element.onclick = function() {
    return false;
}

// 常见默认行为
// - 点击链接跳转
// - 提交按钮提交表单
// - 右键显示菜单
// - 选中文本
// - 滚轮页面滚动
```

---

## 原型与原型链

### 原型基础概念

```javascript
// 构造函数
function Person(name) {
    this.name = name;
}

// Person.prototype 是实例的原型
Person.prototype.greet = function() {
    console.log(`Hi, I'm ${this.name}`);
}

// 创建实例
const person1 = new Person('Alice');

// 实例 -> 原型的关系
person1.__proto__ === Person.prototype        // true
Object.getPrototypeOf(person1) === Person.prototype  // true
person1.constructor === Person                // true
```

### __proto__ 和 prototype 的区别

```javascript
// 函数类型对象
function Foo() {}

Foo.__proto__           // 指向 Function.prototype（自己的原型）
Foo.prototype           // 指向实例的原型

// 普通对象
const obj = {};

obj.__proto__           // 指向 Object.prototype（自己的原型）
obj.prototype           // undefined（没有 prototype 属性）
```

> ⚠️ 原文有误: __proto__ 和 prototype 容易混淆
> 
> 常见错误:
> ```javascript
> function Animal() {}
> const dog = new Animal();
> 
> // 错误: dog.prototype 返回 undefined
> console.log(dog.prototype);  // undefined
> 
> // 正确: dog 的原型
> console.log(dog.__proto__)  // Animal.prototype
> console.log(Object.getPrototypeOf(dog))  // Animal.prototype
> ```

### 原型链查找图

```
对象属性查找过程:

obj.property
    │
    ├─ obj 自身有吗?
    │  ├─ 有 → 返回
    │ │  └─ 没有 ↓
    │
    ├─ obj.__proto__ 有吗?
    │  ├─ 有 → 返回
    │ │  └─ 没有 ↓
    │
    ├─ obj.__proto__.__proto__ 有吗?
    │  ├─ 有 → 返回
    │ │  └─ 没有 ↓
    │
    └─ 到达 Object.prototype
       ├─ 有 → 返回
       └─ 没有 → undefined

┌──────────────┐
│   dog        │ (实例)
└────┬─────────┘
     │ __proto__
     ↓
┌──────────────┐
│   Animal     │ (构造函数原型)
│  .prototype  │
├──────────────┤
│   name       │
│   age        │
│   bark()     │
└────┬─────────┘
     │ __proto__
     ↓
┌──────────────┐
│   Object     │
│  .prototype  │
├──────────────┤
│  toString()  │
│  hasOwnProp..│
└────┬─────────┘
     │ __proto__
     ↓
   null
```

### 完整原型链关系

```javascript
// 自定义构造函数
function Foo() {}

// Foo 的实例
const f1 = new Foo();

// Object 的实例
const o1 = {};

// 原型链路径
f1              → Foo.prototype → Object.prototype → null
o1              → Object.prototype → null
Foo             → Function.prototype → Object.prototype → null
Object          → Function.prototype → Object.prototype → null
Function        → Function.prototype → Object.prototype → null
```

### 特殊现象

```javascript
// 现象 1: Object 的原型是 Function.prototype
Object.__proto__ === Function.prototype  // true

// 现象 2: Function.prototype 的构造函数是 Object
Function.prototype.constructor === Object  // true

// 现象 3: Function 的原型就是 Function.prototype
Function.__proto__ === Function.prototype  // true

// 这些是特殊设计，无需死记，理解即可
```

### hasOwnProperty 检查

```javascript
const obj = {};
obj.name = 'Alice';

obj.hasOwnProperty('name')           // true（自身属性）
obj.hasOwnProperty('toString')       // false（继承自原型）

// for...in 会遍历继承的属性
for (let key in obj) {
    console.log(key);  // name, toString, ...
}

// 检查自身属性
for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
        console.log(key);  // 只有 name
    }
}
```

### Object.create 创建对象

```javascript
// 创建一个原型为 null 的对象
const obj1 = Object.create(null);
obj1.__proto__        // undefined

// 创建一个以指定对象为原型的对象
const proto = { x: 10 };
const obj2 = Object.create(proto);
obj2.x                // 10（继承自原型）

// 指定属性描述符
const obj3 = Object.create(proto, {
    name: {
        value: 'Alice',
        writable: true,
        enumerable: true
    }
})
```

---

## 继承的多种实现

### 1. 原型链继承

```javascript
// 父类
function Animal(name) {
    this.name = name;
}
Animal.prototype.speak = function() {
    console.log(`${this.name} 发出声音`);
}

// 子类
function Dog(name, breed) {
    Animal.call(this, name);  // 必须！
    this.breed = breed;
}

// 关键：设置原型链
Dog.prototype = new Animal();
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
    console.log(`${this.name} 汪汪叫`);
}

const dog = new Dog('Buddy', 'Golden');
dog.speak();  // "Buddy 发出声音"
dog.bark();   // "Buddy 汪汪叫"
```

### 2. 构造函数继承（Call 继承）

```javascript
function Animal(name) {
    this.name = name;
}

function Dog(name, breed) {
    Animal.call(this, name);  // 在子类中调用父类
    this.breed = breed;
}

const dog = new Dog('Buddy', 'Golden');
console.log(dog.name);     // 'Buddy' ✓
console.log(dog.breed);    // 'Golden' ✓

// 问题：无法继承原型上的方法
console.log(dog.speak);    // undefined ✗
```

> ⚠️ 原文有误: 构造函数继承的局限性
> 
> 缺陷:
> ```javascript
> function Animal(name) {
>     this.name = name;
> }
> Animal.prototype.move = function() {}
> 
> function Dog(name) {
>     Animal.call(this, name);
>     // 只能继承实例属性，不能继承原型方法！
> }
> 
> const dog = new Dog('Max');
> dog.move()  // ✗ TypeError: dog.move is not a function
> ```

### 3. 原型链继承 + 构造函数继承

```javascript
function Animal(name) {
    this.name = name;
}
Animal.prototype.speak = function() {
    console.log('说话');
}

function Dog(name, breed) {
    // 继承实例属性
    Animal.call(this, name);
    this.breed = breed;
}

// 继承原型方法
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
    console.log('汪汪');
}

const dog = new Dog('Max', 'Husky');
dog.speak();  // ✓
dog.bark();   // ✓
```

### 4. 寄生式继承

```javascript
function createDog(name) {
    const dog = Object.create(Animal.prototype);
    dog.name = name;
    dog.bark = function() {
        console.log('汪汪');
    }
    return dog;
}

const dog = createDog('Max');
```

### 5. Class 语法糖（ES6+，推荐）

```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        console.log(`${this.name} 说话`);
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name);  // 必须在构造函数开头调用
        this.breed = breed;
    }
    
    bark() {
        console.log(`${this.name} 汪汪叫`);
    }
}

const dog = new Dog('Max', 'Husky');
dog.speak();  // "Max 说话"
dog.bark();   // "Max 汪汪叫"
```

### 继承方案对比

| 方案 | 优点 | 缺点 |
|-----|------|------|
| 原型链 | 简单直观 | 无法传递参数，属性共享 |
| 构造函数 | 属性独立 | 无法继承方法 |
| 混合 | 最实用 | 调用两次父类构造函数 |
| 寄生 | 灵活 | 复杂，不通用 |
| Class | 现代标准 | 需要 Babel 转译 |

---

## 闭包

### 闭包定义

**闭包是一个函数以及它能访问的外层作用域变量的组合。**

```javascript
function outer() {
    const x = 10;  // 外层变量
    
    function inner() {
        console.log(x);  // 访问外层变量
    }
    
    return inner;  // 返回内层函数
}

const fn = outer();
fn();  // 10（形成闭包）
```

### 闭包内存模型

```
┌─────────────────────────────────────────────────────────┐
│  栈（Stack）            │         堆（Heap）            │
├─────────────────────────┼──────────────────────────────┤
│  全局作用域             │                              │
│  ├─ outer: Addr_1  ────────→ function outer() { ... }  │
│  ├─ fn: Addr_2     ────────→ function inner() {        │
│  │                          │   console.log(x)  ◄─┐    │
│  │                          │ }                   │    │
│  │                          │ [[Scope]]: {       │    │
│  │                          │   x: 10 ◄──────────┘    │
│  │                          │ }                       │
│  └─ outer() 执行栈           │                        │
│     ├─ x = 10            │ outer 执行上下文（AO）   │
│     └─ inner = Addr_2    │ { x: 10, inner: Addr_2 } │
│                          │ 不会被回收！（因为有引用）│
└─────────────────────────┴──────────────────────────────┘

关键点：
1. inner 函数的 [[Scope]] 引用了 outer 的执行上下文
2. outer() 执行完毕后，正常来说应该被销毁
3. 但因为 inner 还持有引用，所以不会被销毁
4. 这就是闭包延长变量生命周期的本质
```

### 闭包产生条件

1. **函数嵌套**：内层函数嵌套在外层函数中
2. **访问外层变量**：内层函数访问外层函数的变量
3. **外部引用**：内层函数被返回或赋值给全局变量

```javascript
// 条件 1: 嵌套
function outer() {
    // 条件 2: 访问外层变量
    const x = 10;
    
    function inner() {
        console.log(x);
    }
    
    // 条件 3: 外部引用
    return inner;
}

const fn = outer();
fn();  // ✓ 形成闭包
```

### 闭包的三种形式

#### 形式 1: 返回函数

```javascript
function makeAdder(x) {
    return function(y) {
        return x + y;
    }
}

const add5 = makeAdder(5);
console.log(add5(3));  // 8
console.log(add5(10)); // 15
```

#### 形式 2: 赋值给全局变量

```javascript
let globalFn;

function outer() {
    const x = 10;
    globalFn = function() {
        console.log(x);
    }
}

outer();
globalFn();  // 10
```

#### 形式 3: 作为事件回调

```javascript
for (let i = 0; i < 3; i++) {
    const btn = document.querySelector(`button:nth-child(${i+1})`);
    btn.addEventListener('click', function() {
        console.log(i);  // 闭包保存了 i 的值
    })
}
```

### 闭包与垃圾回收

```javascript
// 无闭包：outer 的变量被回收
function outer1() {
    let x = 10;
    return 5;  // x 不被引用，会被回收
}
outer1();

// 有闭包：outer 的变量不被回收
function outer2() {
    let x = 10;
    return function() {
        return x;  // x 被引用，不会被回收
    }
}

const fn = outer2();
fn();  // x 仍然占用内存
```

### 闭包的应用场景

#### 1. 数据私有化

```javascript
function createCounter() {
    let count = 0;  // 私有变量
    
    return {
        increment() { return ++count; },
        decrement() { return --count; },
        get() { return count; }
    }
}

const counter = createCounter();
counter.increment();  // 1
counter.increment();  // 2
// 无法直接访问 count，只能通过方法
```

#### 2. 函数工厂

```javascript
function createMultiplier(n) {
    return function(x) {
        return x * n;
    }
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

#### 3. for 循环的 setTimeout 问题

```javascript
// ✗ 错误：所有输出都是 3
for (var i = 0; i < 3; i++) {
    setTimeout(() => {
        console.log(i);  // 3, 3, 3
    }, 1000)
}

// ✓ 方法 1：使用 let（块级作用域）
for (let i = 0; i < 3; i++) {
    setTimeout(() => {
        console.log(i);  // 0, 1, 2
    }, 1000)
}

// ✓ 方法 2：闭包
for (var i = 0; i < 3; i++) {
    (function(j) {
        setTimeout(() => {
            console.log(j);  // 0, 1, 2
        }, 1000)
    })(i)
}
```

### 闭包的缺点

```javascript
// 内存泄漏风险
function createLargeObject() {
    const largeArray = new Array(1000000);  // 占用大量内存
    
    return function() {
        console.log(largeArray.length);
    }
}

const fn = createLargeObject();
// largeArray 会一直占用内存，直到 fn 被删除
fn = null;  // 手动释放引用，垃圾回收才能清理
```

---

## this 绑定规则

### this 绑定的四种方式

#### 1. 默认绑定（函数调用）

```javascript
function greet() {
    console.log(this);
}

greet();  // this = window（非严格模式）或 undefined（严格模式）

// 严格模式
'use strict';
function greet() {
    console.log(this);  // undefined
}
greet();
```

#### 2. 隐式绑定（方法调用）

```javascript
const obj = {
    name: 'Alice',
    greet() {
        console.log(this.name);
    }
};

obj.greet();  // this = obj，输出 'Alice'

// 丢失 this 的常见问题
const fn = obj.greet;
fn();  // this = window，输出 undefined
```

#### 3. 显式绑定（call/apply/bind）

```javascript
function greet(greeting, punctuation) {
    console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const person = { name: 'Bob' };

// call：立即执行，参数逐个传递
greet.call(person, 'Hi', '!')   // "Hi, I'm Bob!"

// apply：立即执行，参数作为数组传递
greet.apply(person, ['Hello', '!!!'])  // "Hello, I'm Bob!!!"

// bind：返回新函数，this 被永久绑定
const boundGreet = greet.bind(person, 'Hey');
boundGreet('~')  // "Hey, I'm Bob~"
```

> ⚠️ 原文有误: call/apply/bind 的差异容易混淆
> 
> 对比表:
> ```javascript
> function test(a, b) {
>     console.log(this, a, b);
> }
> 
> const obj = {x: 1};
> 
> test.call(obj, 1, 2)         // 立即执行，逐个参数
> test.apply(obj, [1, 2])      // 立即执行，数组参数
> test.bind(obj, 1, 2)()       // 返回函数，需要调用
> 
> // ✓ 记忆技巧
> // call: C 开头 → 逐个参数 (C for "Comma")
> // apply: A 开头 → 数组参数 (A for "Array")
> // bind: B 开头 → 返回函数 (B for "Binding, returns")
> ```

#### 4. new 绑定（构造函数调用）

```javascript
function Person(name) {
    this.name = name;  // this 指向新创建的对象
}

const person = new Person('Charlie');
console.log(person.name);  // 'Charlie'
console.log(person instanceof Person);  // true
```

### this 绑定优先级

```
new 绑定 > 显式绑定 (call/apply/bind) > 隐式绑定 > 默认绑定

优先级演示：
```

```javascript
function test() {
    console.log(this);
}

const obj1 = { x: 1 };
const obj2 = { x: 2 };

// 优先级测试
const fn = test.bind(obj1);  // 显式绑定
fn.call(obj2);  // 显式绑定优先级高，this 仍是 obj1

// new 优先级最高
const result = new fn();  // new 的优先级 > bind
console.log(result.x);  // undefined（new 创建新对象）
```

### this 绑定决策树

```
                    函数被调用?
                        │
        ┌───────────────┼───────────────┐
        │               │               │
    使用 new?        显式绑定?      是否为方法?
       /│\            / \            / \
      是 │否         是   否        是   否
        │            │   │         │    │
        │        this=  继续      this= this=
        │        显式    判断      对象  全局
        │        值              (隐式) (默认)
        │
     this=新建对象
     ↓
  this.[[Prototype]] = 构造函数.prototype
```

### 特殊情况

#### 箭头函数（没有自己的 this）

```javascript
const obj = {
    name: 'Alice',
    greet() {
        const arrow = () => {
            console.log(this.name);  // this 继承自外层
        }
        arrow();  // 'Alice'
    }
}

obj.greet();

// 箭头函数不能绑定 this
const arrowFn = () => {
    console.log(this);
}

arrowFn.call({x: 1});  // 仍然指向全局对象，call 无效
```

#### setTimeout 中的 this

```javascript
const obj = {
    name: 'Bob',
    test() {
        // 普通函数：this = window
        setTimeout(function() {
            console.log(this);  // window
        }, 1000)
        
        // 箭头函数：this = obj
        setTimeout(() => {
            console.log(this);  // obj
        }, 1000)
    }
}

obj.test();
```

---

## 执行上下文与作用域链

### 执行上下文

执行上下文是代码执行时的环境对象，包含变量、函数声明等信息。

#### 全局执行上下文

```javascript
// 页面打开时创建
// window 对象就是全局执行上下文

// 预处理阶段
var a;        // 添加属性 a: undefined
function test() {}  // 添加属性 test: Function
this = window;  // this 指向 window

// 代码执行阶段
a = 10;
test();
```

#### 函数执行上下文

```javascript
function test(x, y) {
    var z = 10;
    
    // 预处理阶段（函数调用时）
    // 形参赋值: x, y 作为属性
    // arguments 对象
    // var z: undefined
    // 嵌套函数声明提升
    // this 赋值
    
    // 代码执行阶段
    z = x + y;
}

test(1, 2);

// 函数执行上下文创建过程：
// 1. 预处理：形参 → AO 属性
// 2. 预处理：var 声明 → AO 属性（undefined）
// 3. 预处理：function 声明 → AO 属性（函数体）
// 4. 预处理：this → AO.this
// 5. 代码执行
```

> ⚠️ 原文有误: 执行上下文和作用域容易混淆
> 
> 关键区别:
> ```javascript
> function outer() {
>     const x = 10;
>     
>     function inner() {
>         console.log(x);  // 向上查找（作用域链）
>     }
>     
>     inner();
> }
> 
> 作用域: 在函数声明时就确定了（词法作用域）
> 执行上下文: 在函数调用时才创建，每次调用都创建新的
> 
> // 作用域是静态的，执行上下文是动态的
> ```

### 作用域链

变量查找遵循作用域链：当前作用域 → 外层作用域 → 全局作用域

```javascript
const global = 'global';

function outer() {
    const outer_var = 'outer';
    
    function inner() {
        const inner_var = 'inner';
        
        console.log(inner_var);   // 'inner'（自身）
        console.log(outer_var);   // 'outer'（外层）
        console.log(global);      // 'global'（全局）
    }
    
    inner();
}

outer();
```

### 作用域链查找过程

```
变量查找: x

当前作用域
├─ x 存在? ─ 是 → 使用当前的 x
└─ x 不存在? ↓

外层作用域 1
├─ x 存在? ─ 是 → 使用外层 1 的 x
└─ x 不存在? ↓

外层作用域 2
├─ x 存在? ─ 是 → 使用外层 2 的 x
└─ x 不存在? ↓

...

全局作用域
├─ x 存在? ─ 是 → 使用全局的 x
└─ x 不存在? → undefined（非严格模式）
              或 ReferenceError（严格模式）
```

### 执行栈

执行栈是一个栈结构，存储所有执行上下文。

```
代码执行顺序：

script 启动
    ↓
创建全局执行上下文 [Global]

function1() 调用
    ↓
创建 function1 执行上下文
执行栈: [Global, Function1]

    function2() 调用
        ↓
    创建 function2 执行上下文
    执行栈: [Global, Function1, Function2]

    function2() 结束
        ↓
    销毁 function2 执行上下文
    执行栈: [Global, Function1]

function1() 结束
    ↓
销毁 function1 执行上下文
执行栈: [Global]

页面关闭
    ↓
销毁全局执行上下文
执行栈: []
```

### let/const 与 var 的区别

```javascript
// var: 函数作用域
function test() {
    for (var i = 0; i < 3; i++) {}
    console.log(i);  // 3（i 泄露到函数作用域）
}

// let: 块级作用域
function test() {
    for (let j = 0; j < 3; j++) {}
    console.log(j);  // ReferenceError（j 在块内）
}

// const: 块级作用域 + 不可重新赋值
const PI = 3.14;
PI = 3.14159;  // TypeError: Assignment to constant variable
```

### 变量提升

```javascript
// var 提升（声明提升，赋值不提升）
console.log(x);  // undefined
var x = 10;
console.log(x);  // 10

// 实际执行过程：
var x;           // 提升声明，值为 undefined
console.log(x);  // undefined
x = 10;
console.log(x);  // 10

// function 提升（整个函数提升）
test();  // 'hello' 可以在声明前调用

function test() {
    console.log('hello');
}

// let/const 不提升（暂存死区）
console.log(y);  // ReferenceError
let y = 10;
```

---

## 同步异步与事件循环

### 进程与线程

```
进程 (Process)
├─ 定义: 程序的一次执行
├─ 特性: 占有独立的内存空间
├─ 创建: 启动程序时创建
└─ 数量: 一个程序多个进程

线程 (Thread)
├─ 定义: CPU 的基本调度单位
├─ 特性: 进程内共享内存
├─ 创建: 进程创建后的执行流
└─ 关系: 一个进程 ≥ 一个线程
        一个线程 < 一个进程
```

### JavaScript 单线程模型

```javascript
// JS 是单线程的证明
const timer = setTimeout(() => {
    console.log('定时器回调');
}, 1000);

// 执行大量计算（堵塞主线程）
let sum = 0;
for (let i = 0; i < 1000000000; i++) {
    sum += i;
}

// 结论：即使 1000ms 已过，定时器回调也要等主线程空闲
// 输出顺序：先输出 sum，再输出"定时器回调"
```

### 同步任务 vs 异步任务

```javascript
// 同步：顺序执行
console.log(1);
console.log(2);
console.log(3);
// 输出: 1, 2, 3

// 异步：需要等待条件满足
console.log(1);
setTimeout(() => {
    console.log(2);
}, 0);
console.log(3);
// 输出: 1, 3, 2
// 定时器回调是异步任务，需要等主线程空闲
```

### 异步任务的种类

```javascript
// 1. 定时器
setTimeout(callback, 1000)
setInterval(callback, 1000)

// 2. 事件监听
element.addEventListener('click', callback)

// 3. Promise
new Promise((resolve) => {
    resolve(value);
}).then(callback)

// 4. async/await
async function fn() {
    await promise;
}

// 5. 网络请求
fetch(url).then(callback)
```

### 事件循环（Event Loop）

事件循环是 JavaScript 异步执行的核心机制。

```
┌────────────────────────────────────────────────────┐
│         JavaScript 事件循环机制                     │
├────────────────────────────────────────────────────┤
│                                                    │
│  ┌──────────────┐      ┌─────────────────┐       │
│  │  执行栈      │      │  异步任务管理   │       │
│  │ (Call Stack) │      │  (Worker Pool)  │       │
│  ├──────────────┤      ├─────────────────┤       │
│  │ function() ──┼─────→ 定时器           │       │
│  │   code       │      DOM 事件          │       │
│  │   ...        │      网络请求          │       │
│  └──────────────┘      │ 条件满足?      │       │
│        ↑               └────┬────────────┘       │
│        │                    │ (满足)              │
│        │               ┌────▼────────────┐       │
│        │               │  微任务队列     │       │
│        │               │ (Microtask)     │       │
│        │               ├─────────────────┤       │
│        │               │ Promise.then    │       │
│        │               │ MutationObserver│       │
│        │               │ process.next... │       │
│        │               └────┬────────────┘       │
│        │                    │                    │
│        │               ┌────▼────────────┐       │
│        │               │  宏任务队列     │       │
│        │               │ (Macrotask)     │       │
│        │               ├─────────────────┤       │
│        │               │ setTimeout      │       │
│        │               │ setInterval     │       │
│        │               │ setImmediate    │       │
│        │               │ I/O             │       │
│        │               └────┬────────────┘       │
│        │                    │                    │
│        └────────────────────┘                    │
│                                                  │
└────────────────────────────────────────────────────┘

事件循环流程：
1. 执行栈中代码执行完毕
2. 微任务队列是否有任务? ─ 是 → 执行所有微任务
3. 宏任务队列是否有任务? ─ 是 → 执行一个宏任务
4. 返回步骤 2（不断循环）
```

### Event Loop 执行顺序

```javascript
console.log('开始');  // 1

setTimeout(() => {
    console.log('setTimeout');  // 6
}, 0);

Promise.resolve()
    .then(() => {
        console.log('Promise.then');  // 3
    })
    .then(() => {
        console.log('Promise.then 2');  // 4
    });

console.log('中间');  // 2

async function test() {
    console.log('async');  // 5
}

test();

console.log('结束');  // 结束

// 输出顺序：
// 1. 开始
// 2. 中间
// 3. async（async 函数体同步执行）
// 4. 结束
// 5. Promise.then
// 6. Promise.then 2
// 7. setTimeout
```

### 微任务 vs 宏任务

```javascript
// 宏任务 (Macrotask)
setTimeout(() => {}, 0)
setInterval(() => {}, 1000)
setImmediate(() => {})  // Node.js
script (主程序)
I/O 操作
UI 渲染

// 微任务 (Microtask)
Promise.then/catch/finally
async/await
MutationObserver
process.nextTick()  // Node.js
queueMicrotask()
```

### 复杂示例

```javascript
console.log('1');  // 宏任务开始

setTimeout(() => {
    console.log('2');  // 宏任务
}, 0);

Promise.resolve()
    .then(() => {
        console.log('3');  // 微任务
        
        setTimeout(() => {
            console.log('4');  // 宏任务
        }, 0);
    })
    .then(() => {
        console.log('5');  // 微任务
    });

console.log('6');  // 宏任务

// 执行顺序:
// 步骤 1：执行主程序（宏任务）
//   输出: 1, 6

// 步骤 2：执行微任务队列
//   输出: 3, 5（所有微任务）

// 步骤 3：执行第一个宏任务
//   输出: 2

// 步骤 4：执行微任务队列（空）

// 步骤 5：执行下一个宏任务
//   输出: 4

// 最终顺序: 1, 6, 3, 5, 2, 4
```

> ⚠️ 原文有误: Event Loop 宏微任务执行顺序容易混淆
> 
> 正确理解:
> ```javascript
> // ✗ 错误认知: 所有宏任务执行完才执行微任务
> 
> // ✓ 正确执行: 每个宏任务后都检查微任务队列
> 
> 1. 执行一个宏任务
> 2. 检查并执行所有微任务 ← 关键！
> 3. 执行下一个宏任务
> 4. 检查并执行所有微任务
> 5. 重复...
> ```

---

## 易错速查

### 数据类型常见错误

#### 1. Date 月份从 0 开始

```javascript
// ✗ 错误
new Date(2025, 5, 25)  // 预期5月，实际6月

// ✓ 正确
new Date(2025, 4, 25)  // 5月
date.getMonth()        // 返回 4（代表5月）

// 记忆技巧：getMonth() 返回值 + 1 = 实际月份
```

#### 2. Math.round 对负数特殊处理

```javascript
// ✗ 错误认知：都是四舍五入
Math.round(-1.5)   // 期望 -2，实际 -1

// ✓ 正确理解：向正无穷方向
Math.round(1.5)    // 2
Math.round(-1.5)   // -1（向上取整）
Math.round(-1.4)   // -1（向下取整）

// Math.round() 特殊处理 0.5：总是向上（正无穷方向）
```

#### 3. Array 访问器方法返回值

```javascript
// ✗ 容易混淆
[1,2,3].map(() => {})      // 返回新数组 [undefined, undefined, undefined]
[1,2,3].forEach(() => {})  // 返回 undefined

// ✓ 区别
// map: 返回映射后的新数组
[1,2,3].map(x => x * 2)    // [2, 4, 6]

// forEach: 返回 undefined，仅用于迭代
[1,2,3].forEach(x => console.log(x))  // undefined
```

### 引用类型常见错误

#### 1. 对象赋值陷阱

```javascript
// ✗ 错误
let obj1 = {x: 1};
let obj2 = obj1;
obj2.x = 2;
console.log(obj1.x);  // 2（修改了原对象！）

// ✓ 浅复制
let obj3 = Object.assign({}, obj1);
let obj4 = {...obj1};

// ✓ 深复制
let obj5 = JSON.parse(JSON.stringify(obj1));
```

#### 2. 函数参数不能替换外层对象

```javascript
// ✗ 错误理解
let obj = {x: 1};

function change(o) {
    o = {x: 2};  // 只改变局部变量
}

change(obj);
console.log(obj.x);  // 仍是 1（未改变）

// ✓ 修改属性有效
function change(o) {
    o.x = 2;  // 修改对象属性
}

change(obj);
console.log(obj.x);  // 2（改变了！）
```

### DOM 常见错误

#### 1. querySelector 返回值

```javascript
// ✗ 错误
const elements = document.querySelector('.item');  // 返回单个元素或 null
elements.forEach(el => {})  // TypeError!

// ✓ 正确
const element = document.querySelector('.item')  // 单个元素
const elements = document.querySelectorAll('.item')  // NodeList
elements.forEach(el => {})
```

#### 2. HTMLCollection 动态更新陷阱

```javascript
// ✗ 错误：删除时集合也在变化
const items = document.getElementsByClassName('item');
for (let i = 0; i < items.length; i++) {
    items[i].remove();  // 删除后 length 减少，可能跳过元素
}

// ✓ 正确
const items = Array.from(document.getElementsByClassName('item'));
items.forEach(item => item.remove());

// 或逆序删除
for (let i = items.length - 1; i >= 0; i--) {
    items[i].remove();
}
```

#### 3. 获取计算样式

```javascript
// ✗ 错误：直接访问属性
element.style.color  // 只能获取行内样式

// ✓ 正确：使用 getComputedStyle
getComputedStyle(element).color  // 获取最终样式
```

### 事件常见错误

#### 1. 事件冒泡和捕获顺序

```javascript
// ✗ 错误认知
// 认为捕获阶段会跳过某些元素

// ✓ 正确理解
// 点击按钮时完整事件流：
// 捕获: window → document → html → body → 祖先 → button
// 目标: button（同时执行捕获和冒泡监听器）
// 冒泡: button → 祖先 → body → html → document → window

const btn = document.querySelector('button');
const container = document.querySelector('.container');

// 捕获阶段触发
container.addEventListener('click', (e) => {
    console.log('container 捕获');  // 先输出
}, true);

// 冒泡阶段触发
btn.addEventListener('click', (e) => {
    console.log('button 冒泡');  // 后输出
});
```

#### 2. 阻止冒泡不会阻止默认行为

```javascript
// ✗ 错误认知
const link = document.querySelector('a');
link.addEventListener('click', (e) => {
    e.stopPropagation();  // 仅阻止冒泡，链接仍会跳转！
});

// ✓ 正确
link.addEventListener('click', (e) => {
    e.preventDefault();  // 阻止默认行为（跳转）
    // e.stopPropagation();  // 可选：也阻止冒泡
});
```

#### 3. 事件委托的目标判断

```javascript
// HTML
// <ul>
//   <li class="item"><span>Item 1</span></li>
// </ul>

// ✗ 容易出错
const ul = document.querySelector('ul');
ul.addEventListener('click', (e) => {
    if (e.target.tagName === 'LI') {
        console.log('点击了 li');
    }
});
// 如果点击 <span>，e.target 是 span，不会匹配！

// ✓ 正确：向上查找
ul.addEventListener('click', (e) => {
    const li = e.target.closest('li');
    if (li) {
        console.log('点击了 li 或其子元素');
    }
});
```

### this 绑定常见错误

#### 1. 箭头函数无法重新绑定 this

```javascript
// ✗ 错误
const arrow = () => {
    console.log(this);
};

arrow.call({x: 1});  // this 仍是全局对象，call 无效

// ✓ 正确理解
const regular = function() {
    console.log(this);
};

regular.call({x: 1});  // this = {x: 1}
```

#### 2. setTimeout 中的 this

```javascript
// ✗ 错误
const obj = {
    name: 'Alice',
    test() {
        setTimeout(function() {
            console.log(this.name);  // undefined（this = window）
        }, 1000);
    }
};

// ✓ 方案 1：箭头函数
obj.test = function() {
    setTimeout(() => {
        console.log(this.name);  // 'Alice'
    }, 1000);
};

// ✓ 方案 2：bind
obj.test = function() {
    setTimeout(function() {
        console.log(this.name);  // 'Alice'
    }.bind(this), 1000);
};
```

### 原型链常见错误

#### 1. __proto__ 和 prototype 混淆

```javascript
// ✗ 错误
function Animal() {}
const dog = new Animal();

console.log(dog.prototype);  // undefined（实例无 prototype）
console.log(dog.__proto__);  // Animal.prototype ✓

// ✓ 正确理解
// __proto__: 实例的原型（内部指针）
// prototype: 构造函数的原型（普通属性）

// 只有函数有 prototype 属性
function Foo() {}
Foo.prototype  // {constructor: Foo}

const obj = {};
obj.prototype   // undefined
```

#### 2. 继承时需要修复 constructor

```javascript
// ✗ 错误
function Animal() {}
function Dog() {}

Dog.prototype = new Animal();
Dog.prototype.constructor  // Animal（错误！）

// ✓ 正确
Dog.prototype = new Animal();
Dog.prototype.constructor = Dog;  // 修复 constructor
```

### 异步常见错误

#### 1. setTimeout 嵌套延迟有最小值

```javascript
// ✗ 可能期望 0ms 执行
setTimeout(() => {
    setTimeout(() => {
        console.log('内层');
    }, 0);  // 实际延迟约 4ms
}, 0);

// 原因：浏览器对嵌套定时器有最小延迟限制（约 4ms）
```

#### 2. Promise 链中的 return

```javascript
// ✗ 容易出错
Promise.resolve(1)
    .then(x => {
        console.log(x);
        x + 1;  // 没有 return！
    })
    .then(x => {
        console.log(x);  // undefined
    });

// ✓ 正确
Promise.resolve(1)
    .then(x => {
        console.log(x);
        return x + 1;  // 必须 return
    })
    .then(x => {
        console.log(x);  // 2
    });
```

#### 3. async/await 中的错误处理

```javascript
// ✗ 未捕获错误
async function test() {
    const result = await fetch('invalid-url');  // 可能抛出错误
}
test();  // 错误会导致未捕获的 Promise rejection

// ✓ 正确
async function test() {
    try {
        const result = await fetch('invalid-url');
    } catch (error) {
        console.log('错误:', error);
    }
}
test();
```

#### 4. 事件循环中微任务执行完才执行宏任务

```javascript
// ✗ 错误理解：所有宏任务执行完才执行微任务
setTimeout(() => console.log('宏1'), 0);
Promise.resolve().then(() => console.log('微1'));
setTimeout(() => console.log('宏2'), 0);
Promise.resolve().then(() => console.log('微2'));

// ✓ 正确顺序：微1, 微2, 宏1, 宏2
// 因为: 每个宏任务后都清空微任务队列
```

### 闭包常见错误

#### 1. for 循环的 setTimeout 陷阱

```javascript
// ✗ 所有输出都是 3
for (var i = 0; i < 3; i++) {
    setTimeout(() => {
        console.log(i);  // 3, 3, 3
    }, 1000);
}

// ✓ 方案 1：使用 let（块级作用域）
for (let i = 0; i < 3; i++) {
    setTimeout(() => {
        console.log(i);  // 0, 1, 2 ✓
    }, 1000);
}

// ✓ 方案 2：闭包（立即执行函数）
for (var i = 0; i < 3; i++) {
    (function(index) {
        setTimeout(() => {
            console.log(index);  // 0, 1, 2 ✓
        }, 1000);
    })(i);
}
```

#### 2. 闭包内存泄漏

```javascript
// ✗ 占用大量内存
function create() {
    const largeArray = new Array(1000000);
    return function() {
        console.log(largeArray.length);
    };
}

const fn = create();
// largeArray 会一直占用内存

// ✓ 手动释放
fn = null;  // 允许垃圾回收
```

---

**文档生成日期: 2026-05-25**

