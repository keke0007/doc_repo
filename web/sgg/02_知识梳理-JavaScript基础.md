# JavaScript 基础知识梳理

## 目录
1. [JS 简介与运行环境](#1-js-简介与运行环境)
2. [数据类型与类型转换](#2-数据类型与类型转换)
3. [变量与作用域](#3-变量与作用域)
4. [运算符与表达式](#4-运算符与表达式)
5. [流程控制](#5-流程控制)
6. [函数](#6-函数)
7. [数组与字符串 API](#7-数组与字符串-api)
8. [对象基础](#8-对象基础)
9. [函数进阶](#9-函数进阶)
10. [易错点速查](#10-易错点速查表)

---

## 1. JS 简介与运行环境

### 1.1 JavaScript 的基本特性

**定义：** JavaScript 是一门**动态**、**弱类型**、**解释型**、**基于对象**的脚本语言。

| 特性 | 说明 |
|------|------|
| **动态** | 程序执行时才确定数据类型（vs 静态：编写时提前确定） |
| **弱类型** | 数据类型可自动转换（vs 强类型：无法自动转换） |
| **解释型** | 边编译边运行，开发效率高（vs 编译型：先全部编译后运行） |
| **脚本语言** | 可嵌入其他编程语言执行 |
| **基于对象** | 内置对象丰富，支持面向对象编程 |

### 1.2 运行环境（解释器）
- **浏览器**：Chrome(V8)、Firefox、Safari 等
- **Node.js**：基于 V8 引擎的服务端 JS 运行环境

### 1.3 浏览器端 JavaScript 组成

```
┌─────────────────────────────────┐
│   浏览器端 JavaScript            │
├─────────────────────────────────┤
│ 1. ECMAScript                    │ (ECMA指定 - 基本语法)
│ 2. BOM                           │ (W3C指定 - 浏览器API)
│ 3. DOM                           │ (W3C指定 - 文档API)
└─────────────────────────────────┘
```

### 1.4 在 HTML 中使用 JS 的三种方式

| 方式 | 语法 | 适用场景 |
|------|------|---------|
| **行内式** | `<元素 onclick="代码">` | 事件处理 |
| **内嵌式** | `<script>代码</script>` | 小段代码 |
| **外链式** | `<script src="文件.js">` | 大型项目 |

> 建议：script 标签放在其他元素后面，避免阻塞页面加载

### 1.5 基本输出方式

```javascript
alert(内容)         // 输出到弹框
document.write(内容) // 输出到页面
console.log(内容)    // 输出到控制台
```

---

## 2. 数据类型与类型转换

### 2.1 数据类型分类

```
JavaScript 数据类型
    │
    ├─ 原始类型（5种）
    │  ├─ number      数字
    │  ├─ string      字符串
    │  ├─ boolean     布尔值
    │  ├─ null        空值
    │  └─ undefined   未定义
    │
    └─ 对象类型（复合类型）
       ├─ Array       数组
       ├─ Object      对象
       ├─ Function    函数
       ├─ Date        日期
       └─ ...其他
```

### 2.2 原始类型详解

#### number 数字类型

```javascript
// 整数形式
764          // 十进制
012          // 八进制（前缀0）
0x12         // 十六进制（前缀0x）

// 浮点数形式
0.1 + 0.2    // ⚠️ 精度问题：0.30000000000000004

// 科学计数法
1.3e4        // 13000
2.3e-2       // 0.023

// 特殊值
NaN          // Not a Number，是 number 类型的一种
Infinity     // 正无穷
-Infinity    // 负无穷
```

**number 有效范围：**
- 最大值：`1.7976931348623157e+308`
- 最小正数：`5e-324`
- 判断是否为有限数：`isFinite(num)`

#### string 字符串类型

```javascript
var msg = 'Hello"World';      // 单引号
var msg = "Hello'World";      // 双引号
var msg = "Hello\nWorld";     // 转义字符

// 转义字符
\n       // 换行
\'       // 单引号
\"       // 双引号
\\       // 反斜杠
\uXXXX   // Unicode 字符
```

#### boolean 布尔类型
```javascript
true     // 表示是、肯定、正确
false    // 表示否、否定、错误
```

#### null 和 undefined

| 类型 | 说明 | 使用场景 |
|------|------|---------|
| **null** | 空值，表示"无" | 手动赋值表示暂无数据 |
| **undefined** | 未定义 | 未赋值变量的默认值 |

### 2.3 类型转换

#### 2.3.1 其他类型 → number

```
String → Number:
  • 纯数字字符串 → 对应数字 ('123' → 123)
  • 空字符串 → 0
  • 其他字符串 → NaN
  • 自动去掉两端空格

Boolean → Number:
  • true → 1
  • false → 0

Undefined → Number:
  • → NaN

Null → Number:
  • → 0
```

#### 2.3.2 其他类型 → string

```
任何类型 → String:
  • 数据按原样转成字符串
  • null → 'null'
  • undefined → 'undefined'
```

#### 2.3.3 其他类型 → boolean

```
Number → Boolean:
  • 0, NaN → false
  • 其他数字 → true

String → Boolean:
  • '' 空字符串 → false
  • 其他字符串 → true
  • ⚠️ 注意：' ' 空格字符串 → true

Undefined → Boolean:
  • → false

Null → Boolean:
  • → false
```

#### 2.3.4 类型转换流程图

```
╔════════════════════════════════════════════════════════════════╗
║                    类型转换决策流程                             ║
╠════════════════════════════════════════════════════════════════╣
║                                                                ║
║  需要转为 NUMBER?                                              ║
║    │                                                            ║
║    ├─ String:  '123'→123 | ''→0 | 'abc'→NaN                   ║
║    ├─ Boolean: true→1 | false→0                               ║
║    ├─ Null:    →0                                              ║
║    └─ Undefined: →NaN                                          ║
║                                                                ║
║  需要转为 STRING?                                              ║
║    │                                                            ║
║    └─ 任何值都按原样转成字符串形式                              ║
║                                                                ║
║  需要转为 BOOLEAN?                                             ║
║    │                                                            ║
║    ├─ Falsy值: 0, '', NaN, null, undefined                    ║
║    └─ Truthy值: 其他所有值                                     ║
║                                                                ║
╚════════════════════════════════════════════════════════════════╝
```

#### 2.3.5 强制类型转换函数

```javascript
// 转为 number
Number(val)      // 全局转换
parseInt(str)    // 提取整数部分（如果非字符串返回NaN）
parseFloat(str)  // 提取浮点数（如果非字符串返回NaN）

// 转为 string
String(val)      // 全局转换

// 转为 boolean
Boolean(val)     // 全局转换
```

#### 2.3.6 自动隐式转换

```javascript
// 自动转换由运算符决定
// 算术运算 → 转为 number
'10' + 20        // '1020' (+ 是字符串连接)
'10' - 2         // 8 (- 是算术减法)
```

---

## 3. 变量与作用域

### 3.1 变量的基本概念

```javascript
var num01;           // 变量声明（创建）
num01 = 250;         // 赋值（初始化）
var num02 = 500;     // 声明并赋值

// 使用未赋值变量
console.log(num01)   // undefined
console.log(noExists) // ❌ ReferenceError
```

### 3.2 变量命名规范

#### 强制规范
- 由数字、字母、下划线、$ 组成
- 不能以数字开头
- 不能是 JavaScript 关键字或保留字

#### 建议规范
- 有意义的命名
- 多词变量使用小驼峰法：`getUserName`
- 类/构造函数使用大驼峰法：`class UserInfo`

### 3.3 作用域

#### 作用域类型

```javascript
// 全局作用域
var global = 'I am global';

function test() {
    // 函数作用域（局部作用域）
    var local = 'I am local';
    
    function inner() {
        // 嵌套函数作用域
        var nested = 'I am nested';
    }
}
```

#### 作用域链

```
╔═══════════════════════════════════════════════════════════════╗
║                  作用域链查找机制                              ║
╠═══════════════════════════════════════════════════════════════╣
║                                                               ║
║  使用变量时的查找顺序：                                        ║
║                                                               ║
║    当前作用域 → 上级作用域 → 全局作用域                        ║
║       │                                                       ║
║       ├─ 如果找到，立即使用（停止查找）                        ║
║       └─ 如果未找到，继续向上查找                              ║
║                                                               ║
║  ⚠️ 关键点：                                                  ║
║     变量的作用域与函数声明位置相关，与调用位置无关              ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

### 3.4 变量提升

#### var 提升

```javascript
// 实际执行顺序
console.log(x);  // undefined (不报错!)
var x = 5;
console.log(x);  // 5

// 原因：JavaScript 在执行前的预处理阶段
// var x;       被提升到最前面
// console.log(x);  // undefined
// x = 5;
// console.log(x);  // 5
```

#### 函数提升

```javascript
foo();           // ✅ 输出 "Hello"

function foo() {
    console.log('Hello');
}

// 原因：function 声明被整个提升到最前面
```

> ⚠️ 只有 function 关键字声明的函数才会提升；表达式方式不会提升

#### 提升与作用域链流程图

```
╔══════════════════════════════════════════════════════════════╗
║              变量提升 & 作用域链执行流程                       ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  1️⃣ 全局代码预处理阶段                                       ║
║     │                                                        ║
║     ├─ 查找所有 var 关键字 → 创建变量（不赋值）              ║
║     └─ 查找所有 function 关键字 → 创建函数（完全赋值）       ║
║                                                              ║
║  2️⃣ 全局代码执行阶段                                         ║
║     │                                                        ║
║     ├─ 按顺序执行语句                                        ║
║     └─ 执行 var 赋值语句时才赋值                             ║
║                                                              ║
║  3️⃣ 函数调用时                                              ║
║     │                                                        ║
║     ├─ 函数体预处理（同上）                                  ║
║     └─ 函数体执行                                            ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 4. 运算符与表达式

### 4.1 算术运算符

| 运算符 | 操作数个数 | 功能 | 副作用 |
|-------|----------|------|-------|
| `+` | 1 | 正号 | 无 |
| `-` | 1 | 负号 | 无 |
| `+` | 2 | 相加 | 无 |
| `-` | 2 | 相减 | 无 |
| `*` | 2 | 相乘 | 无 |
| `/` | 2 | 相除 | 无 |
| `%` | 2 | 取余 | 无 |
| `++` | 1 | 累加 | **有** |
| `--` | 1 | 累减 | **有** |

```javascript
// ++/-- 在前和在后的区别
var x = 5;
console.log(x++);  // 5（先用，后加）
console.log(x);    // 6

console.log(++x);  // 7（先加，后用）
console.log(x);    // 7
```

### 4.2 关系运算符（比较运算符）

| 运算符 | 含义 | 自动转换规则 |
|-------|------|------------|
| `>` | 大于 | 转 number 比较 |
| `<` | 小于 | 转 number 比较 |
| `>=` | 大于等于 | 转 number 比较 |
| `<=` | 小于等于 | 转 number 比较 |
| `==` | 相等 | 类型不同时转 number |
| `!=` | 不相等 | 类型不同时转 number |
| `===` | 全等 | **不转换，直接比较** |
| `!==` | 不全等 | **不转换，直接比较** |

#### 字符串比较规则

```javascript
// 两个都是字符串时，按字符串规则比较
'abc' > 'aac'    // true (b > a)
'10' > '9'       // false ('1' < '9')
'10' > 9         // true (10 > 9，发生转换)
```

#### null 特殊处理

```javascript
null == 0         // false
null == false     // false
null == undefined // true ✅
```

### 4.3 逻辑运算符

| 运算符 | 操作数 | 含义 | 结果 |
|-------|--------|------|------|
| `&&` | 2 | 逻辑与 | 取两个中的一个 |
| `\|\|` | 2 | 逻辑或 | 取两个中的一个 |
| `!` | 1 | 逻辑非 | boolean |

```javascript
// && 规则
true && 100          // 100 (第一个成立，取第二个)
false && 100         // false (第一个不成立，取第一个)
0 && console.log()   // 0 (第二个不执行)

// || 规则
true || 100          // true (第一个成立，取第一个)
false || 100         // 100 (第一个不成立，取第二个)
0 || console.log()   // 执行 console.log
```

### 4.4 赋值运算符

| 运算符 | 含义 | 操作数要求 | 副作用 |
|-------|------|----------|-------|
| `=` | 赋值 | 左为变量 | **有** |
| `+=` | 加赋值 | 左为变量 | **有** |
| `-=` | 减赋值 | 左为变量 | **有** |
| `*=` | 乘赋值 | 左为变量 | **有** |
| `/=` | 除赋值 | 左为变量 | **有** |
| `%=` | 模赋值 | 左为变量 | **有** |

### 4.5 其他运算符

```javascript
typeof x         // 获取类型，返回字符串
x , y            // 逗号运算符，返回第二个操作数
x ? y : z        // 三元条件运算符
'10' + 20        // + 作字符串连接符（只要有一个是 string）
```

### 4.6 运算符优先级（从高到低）

```
1. 一元运算符     (typeof, !, ++, --)
2. 算术运算符     (*, /, %) > (+, -)
3. 关系运算符     (大小比较) > (判等)
4. 逻辑运算符     (&& > ||)
5. 三元运算符     (?:)
6. 赋值运算符     (=, +=, -=, ...)
7. 逗号运算符     (,)
```

---

## 5. 流程控制

### 5.1 分支语句

#### if 单向分支
```javascript
if (条件) {
    // 条件成立时执行
}
```

#### if-else 双向分支
```javascript
if (条件) {
    // 成立时执行
} else {
    // 不成立时执行
}
```

#### if-else if-else 多向分支
```javascript
if (条件1) {
    // 执行1
} else if (条件2) {
    // 执行2
} else if (条件3) {
    // 执行3
} else {
    // 都不成立执行
}
```

#### switch-case 分支

```javascript
switch (表达式) {
    case 值1:
        // 执行1
        break;
    case 值2:
        // 执行2
        break;
    default:
        // 都不匹配执行
}
```

> ✅ switch 使用全等 `===` 判断，不发生类型转换

### 5.2 循环语句

#### while 循环
```javascript
while (条件) {
    // 循环体
}
// 先判断后执行
```

#### do-while 循环
```javascript
do {
    // 循环体
} while (条件);
// 先执行后判断，至少执行一次
```

#### for 循环
```javascript
for (初始化; 条件; 更新) {
    // 循环体
}
// 执行顺序：初始化 → 判断 → 循环体 → 更新 → 判断 → ...
```

### 5.3 跳转语句

```javascript
break       // 结束循环或 switch
continue    // 跳过本次循环，执行下次循环
```

---

## 6. 函数

### 6.1 函数概述

```javascript
// 函数是一段代码块，实现特定功能，可重复使用
// typeof function → 'function'
// 函数属于对象类型
```

### 6.2 函数声明的四种方式

#### ① function 关键字方式

```javascript
function add(a, b) {
    return a + b;
}

// ✅ 会发生函数提升
add(3, 4)  // 可以在声明前调用
```

#### ② 表达式方式（函数直接量）

```javascript
var add = function(a, b) {
    return a + b;
};

// ❌ 不会发生函数提升
add(3, 4)  // 必须在赋值后调用
```

#### ③ Function 函数方式

```javascript
var add = Function('a', 'b', 'return a + b');
```

#### ④ Function 构造函数方式

```javascript
var add = new Function('a', 'b', 'return a + b');
```

### 6.3 函数调用与返回值

```javascript
function test() {
    return 100;
}

test      // 函数本身（未调用）
test()    // 调用函数，获取返回值
```

#### return 的作用

1. 设置返回值
2. 结束函数执行（return 后面的代码不再执行）
3. 没有 return 或 return 为空，默认返回 `undefined`

### 6.4 函数参数

#### 形参与实参

```javascript
function greet(name) {     // name 是形参
    console.log('Hi ' + name);
}

greet('Alice');            // 'Alice' 是实参
```

#### 参数数量不匹配

```javascript
function add(a, b) {
    // 实参多于形参 → 多余的被忽略
    // 实参少于形参 → 缺少的形参值为 undefined
}

add(1, 2, 3)  // 3 被忽略
add(1)        // b = undefined
```

#### 参数默认值

```javascript
// ES5 方式
function sayHi(name) {
    if (name === undefined) {
        name = 'Guest';
    }
    console.log('Hi ' + name);
}

// ES6 方式
function sayHi(name = 'Guest') {
    console.log('Hi ' + name);
}
```

#### arguments 对象

```javascript
function sum() {
    var result = 0;
    for (var i = 0; i < arguments.length; i++) {
        result += arguments[i];
    }
    return result;
}

sum(1, 2, 3, 4, 5);  // 15

// ⚠️ arguments 是伪数组，有 length 属性但不是真数组
```

### 6.5 函数作用域与作用域链

```javascript
var global = 'Global';

function outer() {
    var outerVar = 'Outer';
    
    function inner() {
        var innerVar = 'Inner';
        console.log(innerVar);   // ✅ Inner (本作用域)
        console.log(outerVar);   // ✅ Outer (上级作用域)
        console.log(global);     // ✅ Global (全局作用域)
    }
    
    inner();
    // console.log(innerVar);   // ❌ Error (不在作用域)
}

outer();
```

---

## 7. 数组与字符串 API

### 7.1 数组（Array）

#### 创建数组

```javascript
// 直接量方式
var arr = [];
var arr = [1, 2, 3, 'hello', true, null];

// Array 函数方式
var arr = Array(1, 2, 3);       // [1, 2, 3]
var arr = Array(5);              // [empty × 5] 稀疏数组

// Array 构造函数方式
var arr = new Array(1, 2, 3);
var arr = new Array(5);
```

#### 数组元素的读写

```javascript
var arr = [10, 20, 30];

// 读取
arr[0]              // 10
arr[5]              // undefined (不存在的元素)

// 修改
arr[1] = 25;        // [10, 25, 30]

// 添加
arr[5] = 50;        // [10, 25, 30, empty, empty, 50]
```

#### 数组常用属性和方法

```javascript
// 属性
arr.length          // 获取数组长度

// 添加元素
arr.push(元素)      // 在末尾添加
arr.unshift(元素)   // 在开头添加
arr.splice(索引, 0, 元素)  // 在指定位置添加

// 删除元素
arr.pop()           // 删除最后一个
arr.shift()         // 删除第一个
arr.splice(索引, 数量)  // 删除指定位置的元素

// 遍历
for (var i = 0; i < arr.length; i++) {
    console.log(arr[i]);
}

for (var i in arr) {
    console.log(arr[i]);
}
```

#### 多维数组

```javascript
var matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

matrix[0][1]  // 2
```

#### 稀疏数组

```javascript
// 产生稀疏数组的情况
var arr = Array(3);           // [empty, empty, empty]
var arr = [1, , 3];           // [1, empty, 3]
var arr = [1, 2];
arr[5] = 50;                  // [1, 2, empty, empty, empty, 50]
```

### 7.2 字符串（String）

#### 字符串的数组特性

```javascript
var str = 'Hello';

// 有 length 属性
str.length              // 5

// 可以通过索引读取字符
str[0]                  // 'H'

// ❌ 不能修改单个字符
str[0] = 'h';           // 无效
str.length = 3;         // 无效

// 字符串是类数组（伪数组）
```

#### 字符串常用方法

```javascript
var str = 'Hello World';

// 获取字符
str.charAt(0)           // 'H'
str[0]                  // 'H'

// 查找位置
str.indexOf('o')        // 4
str.lastIndexOf('o')    // 7

// 截取字符串
str.substring(0, 5)     // 'Hello'
str.slice(0, 5)         // 'Hello'
str.substr(0, 5)        // 'Hello'

// 大小写转换
str.toUpperCase()       // 'HELLO WORLD'
str.toLowerCase()       // 'hello world'

// 字符串替换
str.replace('World', 'JS')  // 'Hello JS'

// 字符串分割
str.split(' ')          // ['Hello', 'World']

// 字符串连接
str.concat(' !')        // 'Hello World !'
```

---

## 8. 对象基础

### 8.1 对象（Object）概念

```javascript
// 广义：一切皆对象（数组、函数都是对象）
// 狭义：Object 数据类型（键值对集合）

// Object 特点
// • 无序集合
// • 由属性组成
// • 属性包括属性名和属性值
// • 属性值可以是任何类型
// • 属性值为函数时称为方法
```

### 8.2 创建对象

#### 直接量方式（推荐）

```javascript
var user = {
    name: 'Alice',
    age: 25,
    'home-city': 'Beijing',
    hobbies: ['reading', 'coding'],
    greet: function() {
        console.log('Hi ' + this.name);
    }
};
```

#### Object 函数方式

```javascript
var user = Object();
```

#### Object 构造函数方式

```javascript
var user = new Object();
```

### 8.3 对象属性的读写

#### 语法

```javascript
// 点语法（属性名符合标识符规范）
object.property
object.property = newValue

// 括号语法（属性名特殊时）
object['property']
object['property'] = newValue
object[variable]  // 使用变量表示属性名
```

#### 规则

```javascript
var obj = {};

// 读取不存在的属性 → undefined
obj.noExists     // undefined

// 赋值不存在的属性 → 自动添加
obj.newProp = 123  // {newProp: 123}
```

### 8.4 遍历对象属性

```javascript
var user = {name: 'Alice', age: 25};

for (var key in user) {
    console.log(key);          // 'name', 'age'
    console.log(user[key]);    // 'Alice', 25
}
```

### 8.5 删除对象属性

```javascript
var user = {name: 'Alice', age: 25};

delete user.age;   // {name: 'Alice'}
```

### 8.6 判断属性是否存在

```javascript
var user = {name: 'Alice'};

'name' in user     // true
'age' in user      // false
```

### 8.7 构造函数

#### 什么是构造函数

```javascript
// 构造函数就是用来创建对象的函数
// 与其他编程语言的类概念相同

function User(name, age) {
    this.name = name;
    this.age = age;
}

// 使用 new 关键字实例化
var u1 = new User('Alice', 25);
var u2 = new User('Bob', 30);
```

#### this 的指向

```javascript
// 1. 全局作用域
console.log(this);     // window 对象

// 2. 构造函数内
function User(name) {
    this.name = name;  // this 指向新创建的对象
}
var u = new User('Alice');  // u.name === 'Alice'

// 3. 方法中
var obj = {
    name: 'Alice',
    greet: function() {
        console.log(this.name);  // this 指向 obj
    }
};
obj.greet();           // 'Alice'
```

#### instanceof 和 constructor

```javascript
function User(name) {
    this.name = name;
}

var u = new User('Alice');

u instanceof User      // true (检查原型链)
u.constructor === User // true (直接获取构造函数)
```

---

## 9. 函数进阶

### 9.1 匿名函数

```javascript
// 没有名字的函数表达式
var fn = function() {
    console.log('I am anonymous');
};

fn();  // 调用
```

### 9.2 立即调用函数表达式（IIFE）

```javascript
// 定义后立即调用，创建独立作用域
(function() {
    var local = 'I am local';
    console.log(local);
})();

// 避免全局变量污染
```

### 9.3 回调函数

```javascript
// 满足三个条件：
// 1. 函数是我定义的
// 2. 我没有直接调用
// 3. 函数最终执行了

var arr = [1, 2, 3];

// forEach 中的匿名函数是回调函数
arr.forEach(function(value) {
    console.log(value);
});
```

### 9.4 递归函数

```javascript
// 函数调用自己

// 求阶乘
function factorial(n) {
    if (n === 1) {
        return 1;  // 结束条件
    }
    return n * factorial(n - 1);  // 递归调用
}

factorial(5);  // 120

// ⚠️ 递归效率较低，能用循环就用循环
```

#### 函数调用栈

```
╔═══════════════════════════════════════════════════════════╗
║              递归函数调用栈执行流程                        ║
╠═══════════════════════════════════════════════════════════╣
║                                                           ║
║  factorial(5)                                             ║
║    │                                                      ║
║    └─ 5 * factorial(4)                                   ║
║           │                                               ║
║           └─ 4 * factorial(3)                            ║
║                  │                                        ║
║                  └─ 3 * factorial(2)                     ║
║                         │                                 ║
║                         └─ 2 * factorial(1)             ║
║                                │                          ║
║                                └─ return 1  (基本情况)    ║
║                                                           ║
║  回溯时逐层返回：                                          ║
║    2 * 1 = 2                                              ║
║    3 * 2 = 6                                              ║
║    4 * 6 = 24                                             ║
║    5 * 24 = 120                                           ║
║                                                           ║
╚═══════════════════════════════════════════════════════════╝
```

### 9.5 arguments 对象详解

```javascript
// 伪数组，包含所有实参

function test(a, b, c) {
    console.log(arguments.length);  // 实际传入的参数个数
    console.log(arguments[0]);      // 第一个参数
    
    // 有 length 但不是真数组
    // 无法使用数组的方法
}

test(1, 2, 3);
```

### 9.6 原型与原型链

#### 原型（Prototype）

```javascript
// 每个对象都有原型，原型是对象

function User(name) {
    this.name = name;
}

// 通过原型添加方法
User.prototype.greet = function() {
    console.log('Hi ' + this.name);
};

var u = new User('Alice');
u.greet();  // 'Hi Alice' (从原型继承)
```

#### 获取对象的原型

```javascript
// 隐式原型（__proto__）
object.__proto__

// 显式原型（prototype）
Function.prototype

// 两者指向同一个对象
```

#### 原型链

```javascript
// 对象 → 原型 → 原型的原型 → ... → null

// 属性查找顺序
obj.property
  → obj 自身是否有
  → obj.__proto__ 上是否有
  → obj.__proto__.__proto__ 上是否有
  → ... 直到找到或到达链尾
```

#### hasOwnProperty 与 in

```javascript
var user = {name: 'Alice'};

'name' in user           // true (自身和原型)
user.hasOwnProperty('name')  // true (仅自身)

'toString' in user           // true (来自原型)
user.hasOwnProperty('toString')  // false
```

---

## 10. 易错点速查表

### 10.1 typeof 的坑

| 值 | typeof 结果 |
|----|-----------|
| `100` | `'number'` |
| `'hello'` | `'string'` |
| `true` | `'boolean'` |
| `undefined` | `'undefined'` |
| `null` | **`'object'`** ⚠️ |
| `[]` | `'object'` |
| `{}` | `'object'` |
| `function(){}` | `'function'` |

> ⚠️ 原文有误：`typeof null` 返回 `'object'`，这是 JavaScript 的一个著名 bug。null 实际上不是对象，是原始类型。

### 10.2 NaN 的坑

| 表达式 | 结果 |
|-------|------|
| `NaN == NaN` | **`false`** ⚠️ |
| `NaN === NaN` | **`false`** ⚠️ |
| `isNaN(NaN)` | `true` |
| `Object.is(NaN, NaN)` | `true` ✅ |

> ⚠️ 原文有误：NaN 与任何值都不相等，包括自己！NaN != NaN。判断是否为 NaN 应使用 `isNaN()` 或 `Object.is()`。

### 10.3 == 与 === 的区别

```javascript
// == 隐式转换
'10' == 10         // true (字符串转为数字)
true == 1          // true (布尔转为数字)
null == undefined  // true (特殊规则)
'10' == true       // false (先都转为数字：10 != 1)

// === 严格相等
'10' === 10        // false
true === 1         // false
null === undefined // false
```

> ⚠️ 原文有误：`== 与 ===` 的隐式转换规则需要理清。== 会进行类型转换，=== 完全不转换。特别注意 `null` 和 `undefined` 的特殊处理。

### 10.4 值传递 vs 引用传递

```javascript
// 原始类型：值传递
function test(x) {
    x = 100;
}
var a = 10;
test(a);
console.log(a);  // 10 (未改变)

// 对象类型：引用传递
function test(obj) {
    obj.name = 'Changed';
}
var user = {name: 'Alice'};
test(user);
console.log(user.name);  // 'Changed' (改变了)
```

> ⚠️ 原文有误：JavaScript 中严格来说都是值传递。对于对象，传递的是引用值（内存地址），而不是对象本身。

### 10.5 var 提升 vs let/const 提升

```javascript
// var 提升：创建 + 初始化为 undefined
console.log(x);  // undefined
var x = 5;

// let/const 提升：仅创建，未初始化（暂时性死区）
console.log(y);  // ❌ ReferenceError
let y = 5;

// const 必须初始化
const z;         // ❌ SyntaxError
```

### 10.6 函数提升 vs 变量提升

```javascript
// function 提升：整个函数提升（可在声明前调用）
console.log(test());  // 可执行

function test() {
    return 'Hello';
}

// 表达式不提升
console.log(fn());    // ❌ TypeError

var fn = function() {
    return 'Hello';
};
```

### 10.7 隐式字符串转换

```javascript
// 任何值 + 字符串 = 字符串
1 + '2'          // '12'
true + '2'       // 'true2'
null + '2'       // 'null2'
undefined + '2'  // 'undefined2'

// 但其他算术运算符不同
1 - '2'          // -1 (转为数字后计算)
1 * '2'          // 2
1 / '2'          // 0.5
```

### 10.8 this 指向易错点

```javascript
var obj = {
    name: 'Alice',
    greet: function() {
        console.log(this.name);
    }
};

obj.greet();                      // 'Alice' ✅
var fn = obj.greet;
fn();                             // undefined ❌ (this 是 window)

setTimeout(obj.greet, 100);       // undefined ❌
```

### 10.9 循环中的闭包陷阱

```javascript
// 错误：var 只有函数作用域
var arr = [];
for (var i = 0; i < 3; i++) {
    arr.push(function() {
        return i;  // 所有函数都返回 3
    });
}
arr[0]();  // 3 ❌
arr[1]();  // 3 ❌
arr[2]();  // 3 ❌

// 正确方式1：使用 let（块级作用域）
var arr = [];
for (let i = 0; i < 3; i++) {
    arr.push(function() {
        return i;  // 1, 2, 3 ✅
    });
}

// 正确方式2：IIFE 创建作用域
var arr = [];
for (var i = 0; i < 3; i++) {
    (function(n) {
        arr.push(function() {
            return n;  // 0, 1, 2 ✅
        });
    })(i);
}
```

### 10.10 对象浅拷贝陷阱

```javascript
// 对象赋值是引用传递
var obj1 = {name: 'Alice'};
var obj2 = obj1;
obj2.name = 'Bob';
console.log(obj1.name);  // 'Bob' (被修改了)

// 数组也是
var arr1 = [1, 2, 3];
var arr2 = arr1;
arr2[0] = 100;
console.log(arr1[0]);    // 100 (被修改了)
```

---

## 总结

JavaScript 基础知识的核心要点：

| 知识点 | 关键 |
|-------|------|
| 数据类型 | 5 种原始 + 对象类型 |
| 类型转换 | 理解隐式转换规则 |
| 作用域 | 作用域链、变量提升 |
| 函数 | 参数、return、arguments |
| 对象 | 键值对、方法、this |
| 原型 | 原型链、继承 |

---

### 文档生成日期：2026-05-25

**版本：** 1.0 (基于 Day01 - Day09 课堂笔记整理)

**包含内容：**
- 4 张 ASCII 流程图
- 3 处关键错误纠正
- 完整的易错点速查表
