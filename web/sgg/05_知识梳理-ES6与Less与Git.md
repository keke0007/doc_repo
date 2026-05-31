# 知识梳理：ES6+ 与 Less 与 Git

**文档生成日期: 2026-05-25**

---

## 目录

- [一、ES6+](#一es6)
- [二、Less](#二less)
- [三、Git](#三git)
- [附录：速查表](#附录速查表)

---

# 一、ES6+

## 1. let/const 与块级作用域

### 1.1 let 关键字特性

```
① 不能重复声明         ④ 块级作用域
② 不会提升            ⑤ 不是全局对象属性
③ 可以修改值
```

### 1.2 const 关键字特性

```
① 不能重复声明         ④ 块级作用域
② 不会提升            ⑤ 不是全局对象属性
③ 不能修改值（指针）
```

> **⚠️ 原文有误**：const 声明的变量不能修改"值"—— 准确的说法是 const 不能修改"指针"。对于引用类型（对象、数组），const 可以修改对象属性或数组元素，只是不能重新赋值。
> 
> ```javascript
> const obj = { name: 'Alice' };
> obj.name = 'Bob';  // 正确，修改属性
> obj = {};          // 错误，不能重新赋值
> ```

### 1.3 块级作用域产生情况

```javascript
// 1. let/const 声明 + 大括号
{ let x = 1; }

// 2. 分支结构
if (true) { let x = 1; }
switch (val) { case 1: let x = 1; }

// 3. 循环结构
for (let i = 0; i < 10; i++) { }
while (true) { let x = 1; }
```

### 1.4 暂时性死区（TDZ）

即使外层作用域有同名变量，进入块级作用域后到声明语句前，访问该变量也会报错。

```javascript
let x = 100;
{
    console.log(x);  // ReferenceError: Cannot access 'x' before initialization
    let x = 200;
}
```

---

## 2. 解构赋值

### 2.1 数组解构赋值

根据索引位置进行匹配，可解构数组、字符串、伪数组等。

```javascript
// 基本解构
const [a, b, c] = [100, 200, 300];

// 默认值
const [x, y, z = 250] = [100, 200];

// 嵌套解构
const [a, [b, c], [d, [e, f]]] = [100, ['高', '乐'], [1, [2, 3]]];

// 在函数中解构
function func([name, age]) { console.log(name, age); }
func(['Alice', 25]);
```

### 2.2 对象解构赋值

根据属性名进行匹配，属性顺序无关。

```javascript
// 基本解构
const { name, age } = { name: 'Alice', age: 25 };

// 重命名
const { name: username, age: userAge } = { name: 'Alice', age: 25 };

// 默认值
const { name = 'Unknown', role = 'user' } = { name: 'Alice' };

// 嵌套解构
const { user: { name, email } } = { user: { name: 'A', email: 'a@test.com' } };
```

---

## 3. 字符串新增特性

### 3.1 模板字符串

```javascript
// 直接写换行和插值
const name = 'Alice';
const msg = `Hello ${name}
Welcome to our site!`;

// 标签模板
const a = 100, b = 200;
function tag(strings, ...values) {
    console.log(strings);  // ['Hello ', 'World', '']
    console.log(values);   // [100, 200]
}
tag`Hello ${a}World${b}`;
```

### 3.2 字符串新增方法

| 方法 | 作用 |
|------|------|
| `repeat()` | 字符串重复 |
| `includes()` | 是否包含子串 |
| `startsWith()` | 是否以...开始 |
| `endsWith()` | 是否以...结尾 |
| `padStart()` | 前面填充至指定长度 |
| `padEnd()` | 后面填充至指定长度 |
| `trim()` / `trimStart()` / `trimEnd()` | 删除空格 |
| `replaceAll()` | 替换所有匹配项 |

---

## 4. 数值新增特性

### 4.1 二进制和八进制表示

```javascript
0b1010;      // 二进制
0o75;        // 八进制
```

### 4.2 Number 静态方法

| 方法 | 作用 |
|------|------|
| `Number.MAX_SAFE_INTEGER` | 最大安全整数 (2^53-1) |
| `Number.MIN_SAFE_INTEGER` | 最小安全整数 |
| `Number.EPSILON` | 最小差值（精度） |
| `Number.isNaN()` | 判断是否为 NaN |
| `Number.isFinite()` | 判断是否为有限数 |
| `Number.isInteger()` | 判断是否为整数 |
| `Number.isSafeInteger()` | 判断是否为安全整数 |

### 4.3 Math 新增方法

```javascript
Math.trunc(3.9);      // 3（取整数部分）
Math.sign(-5);        // -1（返回符号）
Math.cbrt(27);        // 3（立方根）
Math.hypot(3, 4);     // 5（平方和的平方根）
```

### 4.4 指数运算符 **

```javascript
2 ** 10;   // 1024
```

### 4.5 BigInt 类型（ES2020）

```javascript
// 表示方式
45n;
0b101n;
0o75n;
0xab1n;

// 特点
// ① 不能与其他类型进行数学运算
// ② 只能表示整数
// ③ 计算精度是精确的，无范围限制
```

### 4.6 数字分隔符（ES2021）

```javascript
123_0000;        // 1230000
12_0000_0000;    // 1200000000
```

---

## 5. 函数新增特性

### 5.1 参数默认值

```javascript
function greet(name = 'Guest', role = 'user') {
    console.log(`Hello ${name}(${role})`);
}
greet('Alice', 'admin');
greet('Bob');
```

### 5.2 Rest 参数

```javascript
// rest 参数替代 arguments
function sum(...nums) {
    console.log(nums);  // 纯数组，不是伪数组
}
sum(1, 2, 3, 4, 5);   // [1, 2, 3, 4, 5]

// 与普通参数结合
function func(name, age, ...hobbies) {
    console.log(hobbies);  // ['swimming', 'reading']
}
func('Alice', 25, 'swimming', 'reading');
```

### 5.3 箭头函数

```javascript
// 语法变体
const fn01 = () => { };              // 无参
const fn02 = (n1, n2) => n1 + n2;    // 简化版
const fn03 = n => n * 2;             // 单参数

// 返回对象
const fn04 = (a, b) => ({ x: a, y: b });  // 需要括号
```

> **⚠️ 原文有误**：箭头函数无 arguments 对象 —— 但可以使用 rest 参数。箭头函数的 arguments 会继承外层作用域的 arguments。
>
> ```javascript
> const fn = (...args) => console.log(args);  // 正确用法
> fn(1, 2, 3);  // [1, 2, 3]
> ```

### 5.4 箭头函数特点

```javascript
// 1. 没有 arguments，可用 rest 参数
// 2. 没有 this，继承上层作用域的 this
// 3. 不能用作构造函数（不能 new）
// 4. 不能用作生成器函数

// this 继承示例
const obj = {
    value: 100,
    getValueArrow: () => this.value,     // undefined（继承全局 this）
    getValueFunc: function() { return this.value; }  // 100
};
```

### 5.5 逻辑赋值运算符

```javascript
x &&= y;   // 相当于 x = x && y
x ||= y;   // 相当于 x = x || y
x ??= y;   // 相当于 x = x ?? y（空值赋值）
```

---

## 6. 数组新增特性

### 6.1 扩展运算符

```javascript
// 1. 拆分数组为参数序列
const nums = [1, 2, 3];
Math.max(...nums);  // 3

// 2. 复制数组
const arr2 = [...arr1];

// 3. 合并数组
const arr3 = [...arr1, ...arr2];

// 4. 伪数组转纯数组
const nodes = document.querySelectorAll('div');
const arr = [...nodes];
```

### 6.2 Array 静态方法

| 方法 | 作用 |
|------|------|
| `Array.of(...values)` | 创建数组（参数作为元素） |
| `Array.from(iterable)` | 可遍历对象→纯数组 |
| `Array.isArray(obj)` | 判断是否为数组 |

### 6.3 Array 实例新增方法

| 方法 | 作用 |
|------|------|
| `find()` | 找第一个满足条件的元素 |
| `findIndex()` | 找第一个满足条件的索引 |
| `fill()` | 用固定值填充数组 |
| `flat(depth)` | 数组扁平化（Infinity→完全扁平） |
| `flatMap()` | 先map再flat（深度1） |
| `includes()` | 是否包含某元素 |
| `at(index)` | 根据索引读取（支持负数） |
| `keys()` / `values()` / `entries()` | 返回迭代器 |

```javascript
// flat 例子
[1, [2, [3, [4]]]].flat(Infinity);  // [1, 2, 3, 4]

// at 例子
const arr = [10, 20, 30];
arr.at(-1);  // 30（最后一个）
```

---

## 7. 对象新增特性

### 7.1 属性和方法简写

```javascript
const name = 'Alice', age = 25;
const user = {
    name,               // 属性简写
    age,
    say() {             // 方法简写
        console.log(`I'm ${this.name}`);
    }
};
```

### 7.2 计算属性名

```javascript
const prop = 'email';
const obj = {
    [prop]: 'alice@test.com',      // 用变量作属性名
    [10 + 10]: 'twenty',           // 用表达式作属性名
    ['prefix-' + Math.random()]: 'random'
};
```

### 7.3 扩展运算符

```javascript
// 对象复制
const obj2 = { ...obj1 };

// 对象合并
const merged = { ...obj1, ...obj2 };

// 对象解构中使用
const { name, ...rest } = obj;
```

### 7.4 Object 静态方法

| 方法 | 作用 |
|------|------|
| `Object.is(a, b)` | 严格判等（NaN === NaN → true） |
| `Object.assign()` | 对象合并（浅拷贝） |
| `Object.keys()` | 获取属性名数组 |
| `Object.values()` | 获取属性值数组 |
| `Object.entries()` | 获取[key, value]二维数组 |
| `Object.fromEntries()` | entries 逆运算 |
| `Object.getOwnPropertyDescriptors()` | 获取所有属性描述符 |
| `Object.hasOwn(obj, prop)` | 判断是否有自身属性 |

### 7.5 super 关键字

在对象方法中使用 super 指向原型对象：

```javascript
const proto = {
    greet() { return 'Hello'; }
};
const obj = {
    greet() {
        return super.greet() + ' World';  // 只能在简写方法中用
    }
};
Object.setPrototypeOf(obj, proto);
obj.greet();  // 'Hello World'
```

---

## 8. 对象属性特性

### 8.1 数据属性特性

```javascript
const obj = {};
Object.defineProperty(obj, 'name', {
    value: 'Alice',
    writable: true,       // 可写（默认 true）
    enumerable: true,     // 可枚举（默认 true）
    configurable: true    // 可配置（默认 true）
});
```

### 8.2 访问器属性 (getter/setter)

```javascript
class Book {
    #year = 2004;
    #edition = 1;
    
    get year() {
        return this.#year;
    }
    
    set year(newValue) {
        if (newValue > 2004) {
            this.#year = newValue;
            this.#edition += newValue - 2004;
        }
    }
}

// 或用 Object.defineProperty
Object.defineProperty(obj, 'year', {
    get() { return this._year; },
    set(val) { this._year = val; },
    enumerable: true,
    configurable: true
});
```

### 8.3 对象冻结和封闭

```javascript
Object.seal(obj);          // 封闭：不能添加删除，可修改属性值
Object.freeze(obj);        // 冻结：不能改任何东西
Object.isSealed(obj);      // 判断是否被封闭
Object.isFrozen(obj);      // 判断是否被冻结
```

---

## 9. Symbol 类型

```javascript
// 创建 Symbol
const sym1 = Symbol();
const sym2 = Symbol('desc');

// Symbol 用作属性名
const obj = {
    [sym1]: 'value1',
    [sym2]: 'value2'
};

// 每个 Symbol 独一无二
Symbol() === Symbol();  // false
typeof Symbol();        // 'symbol'
```

---

## 10. Set 和 Map

### 10.1 Set（集合，成员唯一）

```javascript
// 创建 Set
const s1 = new Set();
const s2 = new Set([1, 2, 2, 3]);  // {1, 2, 3}

// Set 方法
s2.add(4);
s2.has(2);          // true
s2.delete(2);
s2.clear();
s2.size;

// 遍历
for (const item of s2) { }
s2.forEach(item => { });

// 应用：数组去重
const unique = [...new Set([1, 1, 2, 2, 3])];  // [1, 2, 3]
```

### 10.2 WeakSet

- 成员只能是对象类型
- 不可遍历
- 方法：add()、delete()、has()

### 10.3 Map（键值对集合）

```javascript
// 创建 Map
const m = new Map([
    [key1, 'value1'],
    [key2, 'value2']
]);

// Map 方法
m.set('name', 'Alice');
m.get('name');       // 'Alice'
m.has('name');       // true
m.delete('name');
m.clear();
m.size;

// 遍历
for (const [key, value] of m) { }
m.forEach((value, key) => { });
```

### 10.4 WeakMap

- 键只能是对象
- 不可遍历
- 方法：set()、get()、delete()、has()

---

## 11. Iterator 和 Generator

### 11.1 Iterator 遍历器

```javascript
// 手动创建迭代器
function createIterator(arr) {
    let index = 0;
    return {
        next() {
            return {
                value: arr[index++],
                done: index > arr.length
            };
        }
    };
}

const iter = createIterator([1, 2, 3]);
console.log(iter.next());  // {value: 1, done: false}
console.log(iter.next());  // {value: 2, done: false}
console.log(iter.next());  // {value: 3, done: false}
console.log(iter.next());  // {value: undefined, done: true}
```

### 11.2 Iterable 可遍历对象

部署了 Symbol.iterator 的对象为可遍历对象。

```javascript
// 内置可遍历对象
// Array, Set, Map, String, Arguments, NodeList 等

// 自定义可遍历对象
const obj = {
    name: 'Alice',
    age: 25,
    [Symbol.iterator]() {
        const keys = Object.keys(this);
        let index = 0;
        return {
            next: () => ({
                value: this[keys[index++]],
                done: index > keys.length
            })
        };
    }
};

for (const val of obj) {
    console.log(val);  // Alice, 25
}
```

### 11.3 Generator 生成器

```javascript
// 定义生成器函数
function* gen() {
    yield 1;
    yield 2;
    yield 3;
    return 4;
}

// 调用生成器函数得到迭代器
const iter = gen();
console.log(iter.next());  // {value: 1, done: false}
console.log(iter.next());  // {value: 2, done: false}
console.log(iter.next());  // {value: 3, done: false}
console.log(iter.next());  // {value: 4, done: true}

// 用于 for-of 循环
for (const val of gen()) {
    console.log(val);  // 1, 2, 3（不包括 return 的值）
}
```

#### Generator 给对象部署 Iterator

```javascript
const obj = {
    data: [10, 20, 30],
    *[Symbol.iterator]() {
        for (const val of this.data) {
            yield val;
        }
    }
};

for (const val of obj) {
    console.log(val);  // 10, 20, 30
}
```

#### Generator 执行流程

```
┌─────────────────────────────────────┐
│   调用生成器函数 gen()              │
│                                     │
│   返回迭代器对象(不执行函数体)       │
└────────────┬────────────────────────┘
             │
             ↓
┌─────────────────────────────────────┐
│   调用 iter.next()                  │
│   执行至第一个 yield 停止            │
│   返回 {value: ..., done: false}    │
└────────────┬────────────────────────┘
             │
             ↓
┌─────────────────────────────────────┐
│   再次调用 next()                   │
│   从上一个 yield 后继续执行         │
│   至下一个 yield 停止               │
└────────────┬────────────────────────┘
             │
             ↓
┌─────────────────────────────────────┐
│   重复直到遇到 return 或函数末尾    │
│   返回 {value: ..., done: true}     │
└─────────────────────────────────────┘
```

---

## 12. Class 类

### 12.1 定义类和构造器

```javascript
class Person {
    // 属性
    name = 'Unknown';
    age = 0;
    
    // 构造器方法
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    // 实例方法
    greet() {
        console.log(`Hello, I'm ${this.name}`);
    }
}

const p = new Person('Alice', 25);
p.greet();  // Hello, I'm Alice
```

### 12.2 私有属性（#）

```javascript
class User {
    #password;  // 私有属性，只能在类内访问
    
    constructor(username, password) {
        this.username = username;
        this.#password = password;
    }
    
    checkPassword(pwd) {
        return this.#password === pwd;
    }
}

const u = new User('alice', '123456');
u.#password;      // SyntaxError: 私有属性无法访问
u.checkPassword('123456');  // true
```

### 12.3 静态方法和属性

```javascript
class MathUtils {
    static PI = 3.14159;
    
    static add(a, b) {
        return a + b;
    }
}

MathUtils.add(2, 3);    // 5
MathUtils.PI;           // 3.14159
```

### 12.4 继承（extends）

```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        console.log(`${this.name} makes a sound`);
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name);        // 调用父类构造器
        this.breed = breed;
    }
    
    speak() {
        super.speak();      // 调用父类方法
        console.log(`${this.name} barks`);
    }
}

const dog = new Dog('Rex', 'Labrador');
dog.speak();  // Rex makes a sound. Rex barks
```

> **⚠️ 原文有误**：在子类构造器中必须先调用 super() ——— 这是当子类定义了构造器方法时的要求。如果子类没有定义构造器，会自动调用父类构造器。

---

## 13. Promise

### 13.1 Promise 基础

```javascript
const promise = new Promise((resolve, reject) => {
    // 异步操作
    if (/* 成功 */) {
        resolve('Success!');
    } else {
        reject('Error!');
    }
});

promise
    .then(result => console.log(result))
    .catch(error => console.log(error));
```

### 13.2 Promise 状态机

```
                    ┌──────────────┐
                    │   pending    │
                    │ (初始状态)    │
                    └──────┬───────┘
                           │
            ┌──────────────┼──────────────┐
            ↓              ↓              ↓
      ┌──────────┐  ┌──────────┐  (无法到达)
      │ fulfilled│  │ rejected │
      │(resolve) │  │(reject)  │
      └──────────┘  └──────────┘
            │              │
            └──────┬───────┘
                   ↓
        (状态不可逆转，确定后永不改变)
```

> **⚠️ 原文有误**：Promise 状态一旦确定（fulfilled 或 rejected），就不可逆转，无论如何调用 resolve 或 reject 都不会再改变状态。
>
> ```javascript
> const p = new Promise((resolve, reject) => {
>     resolve('first');
>     reject('second');   // 被忽略，状态已确定
> });
> p.then(v => console.log(v));  // 'first'
> ```

### 13.3 Promise 方法

| 方法 | 作用 |
|------|------|
| `Promise.resolve(value)` | 返回已 resolve 的 Promise |
| `Promise.reject(reason)` | 返回已 reject 的 Promise |
| `Promise.all(iterable)` | 所有都成功才成功 |
| `Promise.race(iterable)` | 先完成的决定结果 |
| `Promise.allSettled(iterable)` | 等待所有完成（无论结果） |

### 13.4 async/await

```javascript
// async 函数返回 Promise
async function fetchData() {
    try {
        const response = await fetch('/api/data');
        const data = await response.json();
        return data;
    } catch (error) {
        console.log('Error:', error);
    }
}

fetchData().then(data => console.log(data));
```

---

## 14. 模块化（import/export）

### 14.1 ES6 Module

```javascript
// math.js - 导出
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
export default { add, subtract };

// main.js - 导入
import math, { add, subtract } from './math.js';
console.log(add(1, 2));

// 导入所有
import * as m from './math.js';
m.add(1, 2);
```

### 14.2 模块解析流程

```
┌────────────────────────────────────┐
│   浏览器遇到 <script type="module">│
│         或 import 语句              │
└────────┬───────────────────────────┘
         │
         ↓
┌────────────────────────────────────┐
│   构造阶段：找到所有模块文件        │
│   解析 import/export 关系           │
│   构建模块记录                       │
└────────┬───────────────────────────┘
         │
         ↓
┌────────────────────────────────────┐
│   实例化阶段：创建模块命名空间       │
│   绑定导入的绑定                    │
│   不执行代码，只做关联               │
└────────┬───────────────────────────┘
         │
         ↓
┌────────────────────────────────────┐
│   求值阶段：执行模块代码           │
│   按依赖顺序执行                    │
│   填充绑定值                        │
└────────┬───────────────────────────┘
         │
         ↓
┌────────────────────────────────────┐
│   模块加载完成，可以使用导出内容   │
└────────────────────────────────────┘
```

---

## 15. ES6 对象浅拷贝和深拷贝

### 15.1 浅拷贝

```javascript
// 数组浅拷贝
const arr1 = [1, 2, 3];
const arr2 = [...arr1];
const arr3 = arr1.slice();
const arr4 = arr1.concat();

// 对象浅拷贝
const obj1 = { name: 'Alice', age: 25 };
const obj2 = { ...obj1 };
const obj3 = Object.assign({}, obj1);
```

### 15.2 深拷贝

```javascript
// 方式 1：JSON（无法拷贝方法和 undefined）
const deepClone1 = JSON.parse(JSON.stringify(obj));

// 方式 2：递归实现（完整）
function deepClone(data) {
    if (data === null || typeof data !== 'object') return data;
    
    const result = Array.isArray(data) ? [] : {};
    for (const key in data) {
        if (data.hasOwnProperty(key)) {
            result[key] = deepClone(data[key]);
        }
    }
    return result;
}
```

---

# 二、Less

## 1. Less 基础

### 1.1 Less 文件编译

方式一：使用 less.js 库（浏览器实时编译）
```html
<style type="text/less">
    // Less 代码
</style>
<script src="less.js"></script>
```

方式二：VSCode 插件 "Easy Less"（推荐开发）
```
1. 安装 Easy Less 扩展
2. 创建 .less 文件
3. 保存时自动编译为 .css
```

### 1.2 注释

```less
/* CSS 注释，会编译到 .css */
// Less 注释，不会编译
```

---

## 2. Less 变量

### 2.1 定义和使用变量

```less
@primary-color: #900;
@secondary-color: #090;
@base-padding: 20px;
@container-width: 1200px;

.header {
    color: @primary-color;
    padding: @base-padding;
}

.container {
    width: @container-width;
    background: @secondary-color;
}
```

### 2.2 变量作为属性名或选择器

```less
@prop: padding;
@selector: .news;

.box {
    @{prop}: 10px;      // 属性名插值
}

@{selector} li {        // 选择器插值
    color: blue;
}
```

### 2.3 特殊字符变量

```less
@min768: ~"min-width:768px";
@selector: ~".box li";

@media (@min768) {
    .container {
        width: 750px;
    }
}
```

---

## 3. Less 混合（Mixins）

### 3.1 基本混合

```less
// 定义混合
.center-box() {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
}

// 使用混合
.modal {
    .center-box();
    width: 400px;
    height: 300px;
}
```

### 3.2 带参数的混合

```less
// 定义参数化混合
.border(@width: 1px, @color: #000, @style: solid) {
    border: @width @style @color;
}

// 使用（按顺序传参）
.box {
    .border(2px, red, dashed);
}

// 使用（按名字传参）
.container {
    .border(@color: blue, @width: 3px);
}
```

### 3.3 @arguments

```less
.box-shadow(@x, @y, @blur, @spread, @color) {
    -webkit-box-shadow: @arguments;
    -moz-box-shadow: @arguments;
    box-shadow: @arguments;
}

.card {
    .box-shadow(3px, 3px, 10px, 0, rgba(0,0,0,0.1));
}
```

### 3.4 条件判断

```less
.triangle(@size, @color, @direction) when (@direction = up) {
    width: 0;
    height: 0;
    border-left: @size solid transparent;
    border-right: @size solid transparent;
    border-bottom: @size solid @color;
}

.triangle(@size, @color, @direction) when (@direction = down) {
    width: 0;
    height: 0;
    border-left: @size solid transparent;
    border-right: @size solid transparent;
    border-top: @size solid @color;
}

// 使用
.arrow-up {
    .triangle(10px, red, up);
}
```

---

## 4. Less 嵌套

### 4.1 基本嵌套（层级选择器）

```less
.news {
    color: blue;
    
    li {                // .news li
        margin: 10px 0;
    }
    
    > li {              // .news > li
        margin: 15px 0;
    }
    
    + li {              // .news + li
        margin-left: 20px;
    }
    
    ~ li {              // .news ~ li
        margin-top: 5px;
    }
}
```

### 4.2 & 父选择符

```less
.btn {
    padding: 10px 20px;
    
    &:hover {           // .btn:hover
        background: #090;
    }
    
    &.active {          // .btn.active
        font-weight: bold;
    }
    
    &-primary {         // .btn-primary
        color: white;
    }
}
```

> **⚠️ 原文有误**：& 符号只在嵌套中有意义 —— 它代表父选择符，不在最顶层使用时作为字面量处理。

### 4.3 媒体查询嵌套

```less
.container {
    width: 100%;
    
    @media (min-width: 768px) {
        width: 750px;
    }
    
    @media (min-width: 992px) {
        width: 970px;
    }
    
    @media (min-width: 1200px) {
        width: 1170px;
    }
}

/* 编译为 */
@media (min-width: 768px) {
    .container {
        width: 750px;
    }
}
/* ... */
```

### 4.4 混合与嵌套结合

```less
.clearfix() {
    &::after {
        content: "";
        display: block;
        clear: both;
    }
}

.header {
    .clearfix();
}
```

---

## 5. Less 运算

```less
@base-width: 100px;
@multiplier: 3;
@margin: 20px;

.box {
    width: @base-width + @multiplier;  // 100px + 3 = 103px（单位）
    margin: @margin * 2;               // 40px
    padding: (@base-width / 4);        // 需要括号
    height: 300px - 50px;              // 250px
}

/* 注意：
   1. 除法需要括号
   2. 单位混合时：使用第一个操作数的单位
   3. 只有一个操作数有单位时：使用该单位
*/
```

---

## 6. Less 导入

```less
// 导入其他 Less 文件（编译到输出 CSS 中）
@import "variables";
@import "mixins.less";
@import "components";

// 导入 CSS 文件（原样保留）
@import "reset.css";
```

---

## 7. Less 函数

| 函数 | 作用 |
|------|------|
| `percentage(value)` | 转为百分比 |
| `lighten(color, %)` | 提高亮度 |
| `darken(color, %)` | 降低亮度 |
| `saturate(color, %)` | 增加饱和度 |
| `desaturate(color, %)` | 降低饱和度 |
| `mod(n1, n2)` | 取余 |
| `ceil(value)` | 向上取整 |
| `floor(value)` | 向下取整 |
| `round(value)` | 四舍五入 |

```less
.primary-btn {
    background: #0066cc;
    
    &:hover {
        background: lighten(#0066cc, 10%);  // 提亮 10%
    }
}

.disabled {
    background: desaturate(#0066cc, 50%);   // 降低饱和度
}

@width: 800px;
.sidebar {
    width: ceil(@width / 3);  // 267px
}
```

---

# 三、Git

## 1. Git 基础概念

### 1.1 版本控制系统的功能

- 代码备份
- 版本回退
- 多人协作
- 权限控制

### 1.2 Git 三个工作区

```
┌─────────────────────────────────────────┐
│           工作区（Working Area）          │
│        编辑代码的地方（*.js文件等）       │
└──────────────────┬──────────────────────┘
                   │ git add
                   ↓
┌─────────────────────────────────────────┐
│           暂存区（Staging Area）          │
│          git add 后的临时存储区          │
│          (.git/index 文件)               │
└──────────────────┬──────────────────────┘
                   │ git commit
                   ↓
┌─────────────────────────────────────────┐
│        版本库（Repository）              │
│      真正存储代码历史的地方               │
│       (.git/objects 目录)               │
└─────────────────────────────────────────┘
```

---

## 2. Git 初始化和基础命令

### 2.1 初始化配置

仅需在安装 Git 后执行一次：

```bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

### 2.2 仓库初始化

每个新项目初始化一次：

```bash
git init
```

### 2.3 基本操作流程

```bash
# 1. 修改代码
# （编辑文件...）

# 2. 查看状态
git status

# 3. 添加到暂存区
git add .                    # 添加所有修改和新文件
git add -A                   # 推荐：添加所有变化
git add <file>              # 添加指定文件
git add -u                  # 添加已跟踪文件的修改和删除

# 4. 查看差异
git diff                    # 工作区 vs 版本库
git diff --cached           # 暂存区 vs 版本库

# 5. 提交到版本库
git commit -m "提交信息"

# 或一步到位：
git commit -am "提交信息"   # 跳过 git add，直接提交跟踪文件
```

---

## 3. 撤销修改

### 3.1 撤销工作区修改

```bash
# 未添加到暂存区
git restore <file>          # 恢复指定文件
git restore .               # 恢复所有文件

# 旧命令（仍可用）
git checkout -- <file>
git checkout -- .
```

### 3.2 撤销暂存区

```bash
# 从暂存区移除但保留工作区修改
git restore --staged <file>
git restore --staged .
```

### 3.3 完全撤销（回到某个版本）

```bash
git reset --hard <commitID>    # 回到指定版本
git reset --hard HEAD^         # 回到上一个版本
git reset --hard HEAD^^        # 回到上两个版本
```

---

## 4. 历史版本管理

### 4.1 查看提交记录

```bash
git log                     # 详细日志
git log --oneline           # 单行日志
git log -n                  # 查看最近 n 次提交
git log --oneline -5        # 最近 5 次单行日志

git reflog                  # 查看所有操作记录（包括回滚）
```

### 4.2 版本号（CommitID）

- 完整形式：40位16进制哈希值
- 简写形式：前7位通常足够

---

## 5. Git 分支

### 5.1 分支操作

```bash
# 创建分支
git branch <branch-name>

# 查看分支（本地）
git branch

# 查看所有分支（包括远程）
git branch -a

# 切换分支
git switch <branch-name>
git checkout <branch-name>  # 旧命令

# 创建并切换到新分支
git switch -c <branch-name>
git checkout -b <branch-name>  # 旧命令

# 重命名分支
git branch -m <old-name> <new-name>

# 删除分支
git branch -d <branch-name>    # 安全删除（已合并）
git branch -D <branch-name>    # 强制删除
```

### 5.2 分支合并

```bash
# 切换到目标分支（要合并到哪个分支）
git switch main

# 合并源分支
git merge <source-branch>

# 合并时创建合并提交（保留历史）
git merge --no-ff <branch-name>
```

> **⚠️ 原文有误**：`git merge --no-ff` 即使源分支没有新提交，也会创建合并提交 —— 这是重要的规范做法，能保留完整的合并历史，方便日后回溯。

### 5.3 分支合并冲突解决

当多个分支修改同一文件的同一部分时产生冲突：

```bash
# 1. 查看状态
git status

# 2. 手动编辑冲突文件
# 移除冲突标记，保留最终代码

# 3. 再次提交
git add .
git commit -m "Resolve merge conflicts"
```

---

## 6. .gitignore 忽略文件

### 6.1 配置文件

创建 `.gitignore` 文件于项目根目录：

```
# 注释
*.log
*.tmp
node_modules/
dist/
.env

# 忽略所有 .a 文件
*.a

# 但跟踪 lib.a
!lib.a

# 只忽略当前目录的 TODO
/TODO

# 忽略任何目录的 build 文件夹
build/

# 忽略深层 pdf
doc/**/*.pdf
```

### 6.2 规则语法

| 规则 | 作用 |
|------|------|
| `*` | 匹配任意字符（除 `/`） |
| `**` | 匹配多级目录 |
| `?` | 匹配单个字符 |
| `[]` | 字符范围 `[abc]` `[a-z]` |
| `!` | 否定规则（重新包含） |
| `/` 开头 | 相对于 .gitignore 位置 |
| `/` 结尾 | 只匹配目录 |
| `#` | 注释 |

### 6.3 已跟踪文件的忽略

```bash
# 从版本库中删除但保留本地文件
git rm --cached <file>

# 更新 .gitignore
# 然后提交
git add .gitignore
git commit -m "Remove cached file"
```

---

## 7. 远程仓库

### 7.1 远程仓库基础

```bash
# 添加远程仓库别名
git remote add <name> <url>
git remote add origin https://github.com/user/repo.git

# 查看远程信息
git remote -v

# 删除远程别名
git remote remove <name>

# 重命名别名
git remote rename <old> <new>

# 修改 URL
git remote set-url <name> <new-url>
```

### 7.2 场景一：本地有仓库，远程无仓库

```bash
# 1. 在 GitHub 创建空仓库
# 2. 本地添加远程别名
git remote add origin <github-url>

# 3. 第一次推送（-u 记录跟踪）
git push -u origin master

# 4. 后续推送
git push
```

### 7.3 场景二：本地无仓库，远程有仓库

```bash
# 1. 克隆远程仓库
git clone <github-url>

# 2. 创建本地分支对应远程分支
git switch -c <local-branch> origin/<remote-branch>

# 3. 常规开发
git add .
git commit -m "..."
git pull        # 先拉取
git push        # 再推送
```

> **⚠️ 原文有误**：`git pull` 等于 `git fetch + git merge`，而不是 `git fetch + git rebase` —— 这是 pull 的默认行为，会产生合并提交。

### 7.4 推送和拉取

```bash
# 推送本地分支到远程
git push origin <local-branch>:<remote-branch>

# 简写（本地分支与远程分支同名）
git push origin <branch-name>

# 拉取远程分支
git pull                        # 拉取并合并
git fetch origin <branch-name>  # 仅拉取，不合并
```

---

## 8. 多人协作工作流

### 8.1 每日工作流程

**第一天：**
1. 克隆仓库：`git clone <url>`
2. 开发代码
3. 下班前：
   - `git add .`
   - `git commit -m "..."`
   - `git pull`（先拉取他人更新）
   - `git push`（推送自己的代码）

**后续每天：**
1. 早上：`git pull`（获取最新代码）
2. 开发代码
3. 下班前：
   - `git add .`
   - `git commit -m "..."`
   - `git pull`
   - `git push`

### 8.2 冲突解决

```bash
# 1. 拉取时发生冲突
git pull

# 2. 查看冲突
git status

# 3. 手动编辑冲突文件

# 4. 提交解决
git add .
git commit -m "Resolve conflicts"
git push
```

---

## 9. SSH 免密登录

### 9.1 生成 SSH 密钥

```bash
ssh-keygen -t rsa -C "email@example.com"
```

生成的文件位置：
- 私钥：`~/.ssh/id_rsa`
- 公钥：`~/.ssh/id_rsa.pub`

### 9.2 配置公钥

1. 复制公钥内容：`cat ~/.ssh/id_rsa.pub`
2. GitHub：Settings → SSH and GPG keys → New SSH Key
3. 粘贴公钥，保存

### 9.3 使用 SSH 克隆

```bash
git clone git@github.com:user/repo.git
```

---

## 10. Git 分支策略（GitFlow）

```
┌─────────────────────────────────────────┐
│          Master 主分支                  │
│  仅保存正式发布版本，直接对应生产环境    │
└─────────────────────────────────────────┘
          ↑         ↑          ↑
     (merge) (merge) (merge)
          │         │          │
  ┌───────┴─────┬───┴──┐  ┌────┴────┐
  │             │       │  │         │
┌─┴──┐     ┌────┴──┐   │  │ ┌──────┴─┐
│Hot │     │Release│   │  │ │ Feature │
│fix │     │Branch │   │  │ │ Branch  │
└────┘     └───────┘   │  │ └─────────┘
  │             │       │  │
  └─────────────┼───────┘  │
                │          │
            ┌───┴──────────┘
            │
      ┌─────┴──────┐
      │  Develop   │
      │ 开发分支    │
      │ 功能集成点  │
      └────────────┘
```

**分支说明：**
- **Master**: 生产发布分支
- **Develop**: 开发主分支
- **Feature**: 功能开发分支（从 Develop 创建）
- **Release**: 待发布分支（从 Develop 创建）
- **Hotfix**: 线上 Bug 修复（从 Master 创建）

---

## 11. 常用 Linux 命令

| 命令 | 作用 |
|------|------|
| `ls` | 列举目录内容 |
| `ls -a` | 包含隐藏文件 |
| `ls -l` | 详细信息 |
| `cd <path>` | 进入目录 |
| `cd ..` | 进入上级目录 |
| `pwd` | 显示当前路径 |
| `mkdir <dir>` | 创建目录 |
| `rm -rf <file>` | 删除文件/目录 |
| `clear` | 清屏 |

**快捷键：**
- `Tab`: 自动补全
- `Ctrl+L`: 清屏
- `Ctrl+C`: 中止命令
- 上下箭头: 历史命令

---

# 附录：速查表

## ES6 关键特性速查

| 特性 | 语法 | 用途 |
|------|------|------|
| let/const | `let x = 1; const y = 2;` | 块级作用域变量 |
| 解构 | `const {a, b} = obj;` | 快速提取属性 |
| 箭头函数 | `() => {}` | 简洁函数，继承 this |
| 模板字符串 | `` `${var}` `` | 字符串插值 |
| 扩展运算符 | `...array` | 展开/收集 |
| rest 参数 | `(...args) => {}` | 收集剩余参数 |
| class | `class Foo {}` | 类定义 |
| Promise | `new Promise()` | 异步操作 |
| async/await | `async () => { await p }` | 异步语法糖 |
| Set/Map | `new Set(); new Map()` | 新集合类型 |
| Symbol | `Symbol()` | 唯一值类型 |
| 模块 | `import/export` | 代码模块化 |

## Less 速查

| 特性 | 语法 | 例子 |
|------|------|------|
| 变量 | `@name: value;` | `@primary: #090;` |
| 混合 | `.name() {}` | `.border() { border: 1px; }` |
| 嵌套 | `parent { child {} }` | `.box { .item {} }` |
| & 符 | `&:hover` | `.btn { &:hover {} }` |
| 运算 | `@a + @b` | `@width / 2` |
| 导入 | `@import "file"` | `@import "var"` |
| 函数 | `lighten()` | `lighten(@color, 10%)` |
| 条件 | `when (@x = 1)` | `.mixin when (@dir = up) {}` |

## Git 速查

| 操作 | 命令 |
|------|------|
| 初始化 | `git init` |
| 配置 | `git config --global user.name "Name"` |
| 查看状态 | `git status` |
| 添加文件 | `git add .` |
| 提交 | `git commit -m "msg"` |
| 查看日志 | `git log --oneline` |
| 回滚 | `git reset --hard HEAD^` |
| 创建分支 | `git branch <name>` |
| 切换分支 | `git switch <name>` |
| 合并分支 | `git merge <name>` |
| 克隆仓库 | `git clone <url>` |
| 拉取更新 | `git pull` |
| 推送代码 | `git push` |
| 忽略文件 | `.gitignore` |
| SSH 密钥 | `ssh-keygen -t rsa` |

---

## 常见错误与纠正总结

> **⚠️ 原文有误 (1)**: let 不会提升，但存在暂时性死区 —— 从进入块级作用域到 let 声明语句执行前，访问该变量会报 ReferenceError。

> **⚠️ 原文有误 (2)**: const 声明的变量不能修改"值" —— 准确说是不能修改"指针"。对于引用类型，属性值可以修改。

> **⚠️ 原文有误 (3)**: 箭头函数无 arguments —— 但可以使用 rest 参数代替，箭头函数的 arguments 会继承外层作用域。

> **⚠️ 原文有误 (4)**: Promise 状态一旦确定（fulfilled 或 rejected），就永不改变 —— 后续的 resolve/reject 调用被忽略。

> **⚠️ 原文有误 (5)**: git merge --no-ff 会强制创建合并提交 —— 即使可以快进（fast-forward）也会创建，这是好的规范。

> **⚠️ 原文有误 (6)**: git pull 等于 fetch + merge（不是 rebase）—— 默认行为是 merge，会产生合并提交。

> **⚠️ 原文有误 (7)**: Less 中 & 符号代表父选择符 —— 在嵌套中使用时有特殊含义，如 &:hover 表示 .parent:hover。

> **⚠️ 原文有误 (8)**: 子类定义构造器时必须先调用 super() —— 这是强制要求，如果没有定义构造器则自动调用。

---

**文档生成日期: 2026-05-25**

---

