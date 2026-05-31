# TypeScript 学习整理

这份笔记基于 `Typescript` 目录下的 5 份 Markdown 归纳整理，目标不是简单拼接原文，而是按 TypeScript 的学习路径重新组织内容，并给每个核心知识点补上代码示例和应用场景，方便系统学习和回顾。

## 学习路线

建议按下面的顺序学习：

1. 先理解为什么需要 TypeScript，以及它解决了什么问题
2. 再掌握基础类型、函数类型、对象类型
3. 然后学习联合类型、接口、类型缩小、断言等类型系统能力
4. 接着学习类、接口、抽象类、枚举等面向对象写法
5. 再深入泛型、映射类型、条件类型和工具类型
6. 最后补模块化、声明文件、`tsconfig` 等工程化内容

## 原始资料对应关系

| 主题 | 原文件 |
|---|---|
| TypeScript 入门与基础类型 | `01_邂逅TypeScript语法.md` |
| TypeScript 语法细节 | `02_TypeScript语法细节.md` |
| TypeScript 面向对象 | `03_TypeScript面向对象.md` |
| TypeScript 泛型编程 | `04_TypeScript泛型编程.md` |
| TypeScript 语法扩展与工程化 | `05_TypeScript语法扩展.md` |

## 一、为什么要学 TypeScript

### 1. JavaScript 的痛点

JavaScript 非常灵活，但灵活也会带来问题：

- 参数类型不明确
- 返回值不明确
- 对象结构不明确
- 重构时容易埋坑
- 大型项目协作成本高

代码示例：

```js
function add(num1, num2) {
  return num1 + num2
}

add(10, 20)      // 30
add("10", "20")  // "1020"
add(10, "20")    // "1020"
```

应用场景：

- 团队协作项目中，别人调用你的函数时，很难直接知道参数应该传什么
- API 数据结构变化时，容易在运行时才发现错误

### 2. TypeScript 解决什么问题

TypeScript 本质上是 JavaScript 的超集，它为 JavaScript 增加了：

- 静态类型检查
- 更强的编辑器提示
- 更安全的重构体验
- 更清晰的代码约束

代码示例：

```ts
function add(num1: number, num2: number): number {
  return num1 + num2
}

add(10, 20)
// add("10", "20") // 会直接报类型错误
```

应用场景：

- 开发中后台系统、组件库、前端工程脚手架
- 大型 Vue / React 项目

## 二、运行环境与基础写法

### 1. TypeScript 运行方式

TypeScript 代码不能直接在浏览器中运行，一般需要：

1. 编写 `.ts` 文件
2. 使用 `tsc` 编译成 `.js`
3. 再运行生成的 JavaScript

命令示例：

```bash
tsc index.ts
node index.js
```

也可以直接用：

```bash
ts-node index.ts
```

应用场景：

- 快速验证一个 TypeScript 文件
- 写 Node.js 脚本工具时很方便

### 2. 变量声明与类型推导

```ts
let message: string = "Hello TypeScript"
let age: number = 18
let isAdmin: boolean = false
```

如果声明时已经赋值，很多情况下 TS 可以自动推导类型：

```ts
let title = "TS Guide"   // 推导为 string
let count = 100          // 推导为 number
```

应用场景：

- 简单局部变量可依赖推导
- 公共函数参数、返回值、对象结构建议显式写类型

## 三、基础类型

### 1. number / string / boolean

```ts
let score: number = 95
let userName: string = "Tom"
let isLogin: boolean = true
```

应用场景：

- 数值计算、用户名、登录状态等最常见的数据

### 2. array

```ts
let nums: number[] = [1, 2, 3]
let names: Array<string> = ["Tom", "Jack"]
```

应用场景：

- 列表数据、表格数据、接口返回数组

### 3. object

```ts
let info: { name: string; age: number } = {
  name: "Tom",
  age: 18
}
```

应用场景：

- 用户信息对象
- 商品对象
- 配置项对象

### 4. symbol

```ts
const id: symbol = Symbol("id")
```

应用场景：

- 定义唯一属性键
- 某些框架内部标记字段

### 5. null / undefined

```ts
let data: null = null
let token: undefined = undefined
```

应用场景：

- 表示“刻意为空”或“尚未赋值”

### 6. tuple

元组表示“固定长度、固定类型顺序”的数组。

```ts
let point: [number, number] = [100, 200]
let user: [string, number, boolean] = ["Tom", 18, true]
```

应用场景：

- 经纬度坐标
- React Hook 返回值的结构理解
- 某些函数返回多个固定含义的数据

## 四、函数类型

### 1. 参数类型

```ts
function sum(num1: number, num2: number) {
  return num1 + num2
}
```

### 2. 返回值类型

```ts
function getMessage(name: string): string {
  return `Hello ${name}`
}
```

### 3. 匿名函数参数类型

```ts
const names = ["Tom", "Jack", "Lucy"]

names.forEach((item: string) => {
  console.log(item)
})
```

### 4. void

```ts
function logMessage(message: string): void {
  console.log(message)
}
```

应用场景：

- 只负责执行操作，不需要返回值的函数

### 5. never

```ts
function throwError(message: string): never {
  throw new Error(message)
}
```

应用场景：

- 一定抛错的函数
- 死循环函数
- 联合类型穷尽检查

### 6. 可选参数 / 默认参数 / 剩余参数

```ts
function greet(name: string, age?: number) {
  console.log(name, age)
}

function request(url: string, method: string = "GET") {
  console.log(url, method)
}

function total(...nums: number[]) {
  return nums.reduce((sum, item) => sum + item, 0)
}
```

应用场景：

- 可选参数：某些配置可传可不传
- 默认参数：请求方式、分页大小
- 剩余参数：求和、日志函数、参数透传

## 五、对象类型与可选属性

### 1. 对象类型

```ts
function printUser(user: { name: string; age: number }) {
  console.log(user.name, user.age)
}
```

### 2. 可选属性

```ts
function createUser(user: { name: string; age?: number }) {
  console.log(user)
}
```

应用场景：

- 表单对象里部分字段不是必填
- 请求参数中有些条件可选

## 六、特殊类型

### 1. any

```ts
let value: any = "hello"
value = 123
value = true
```

应用场景：

- 老项目迁移时临时兜底
- 第三方库类型不明确时过渡使用

建议：

- 能不用 `any` 就不用

### 2. unknown

`unknown` 比 `any` 更安全，使用前必须先缩小类型。

```ts
let result: unknown = "hello"

if (typeof result === "string") {
  console.log(result.toUpperCase())
}
```

应用场景：

- 后端接口返回的未知数据
- `catch` 捕获的异常对象

## 七、联合类型、别名、接口

### 1. 联合类型

```ts
let value: string | number = "hello"
value = 100
```

应用场景：

- 接口参数可能是字符串或数字
- 输入框值在处理前后可能有不同类型

### 2. 类型别名 type

```ts
type ID = string | number

let userId: ID = 1001
```

应用场景：

- 复用复杂类型定义
- 联合类型命名

### 3. 接口 interface

```ts
interface IUser {
  name: string
  age: number
}

const user: IUser = {
  name: "Tom",
  age: 18
}
```

应用场景：

- 定义接口返回结构
- 定义组件 props 结构
- 定义类要实现的能力

### 4. interface 和 type 的选择

简单理解：

- `interface` 更适合描述对象结构、类实现契约
- `type` 更灵活，适合联合类型、交叉类型、复杂类型组合

## 八、交叉类型与断言

### 1. 交叉类型

```ts
type Person = { name: string }
type Worker = { company: string }

type Staff = Person & Worker

const staff: Staff = {
  name: "Tom",
  company: "OpenAI"
}
```

应用场景：

- 合并多个对象能力
- 组合多个配置结构

### 2. 类型断言 as

```ts
const el = document.getElementById("title") as HTMLHeadingElement
console.log(el.innerText)
```

应用场景：

- DOM 查询结果更具体的类型
- 你比 TS 更明确某个值的结构

### 3. 非空断言 !

```ts
const el = document.getElementById("app")!
el.innerHTML = "Hello"
```

应用场景：

- 确定 DOM 一定存在时

注意：

- 滥用会绕过类型保护

## 九、字面量类型与类型缩小

### 1. 字面量类型

```ts
let direction: "left" | "right" | "center" = "left"
```

应用场景：

- 限制按钮尺寸：`"small" | "medium" | "large"`
- 限制状态：`"success" | "error" | "loading"`

### 2. typeof 缩小

```ts
function printValue(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase())
  } else {
    console.log(value.toFixed(2))
  }
}
```

### 3. instanceof 缩小

```ts
function printDate(date: Date | string) {
  if (date instanceof Date) {
    console.log(date.toISOString())
  } else {
    console.log(date)
  }
}
```

### 4. in 操作符缩小

```ts
type Fish = { swim: () => void }
type Bird = { fly: () => void }

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim()
  } else {
    animal.fly()
  }
}
```

应用场景：

- 处理联合类型对象时判断具体分支

## 十、函数类型、重载与 this

### 1. 函数类型

```ts
type CalcFn = (num1: number, num2: number) => number

const add: CalcFn = (a, b) => a + b
```

### 2. 调用签名

```ts
interface ICounter {
  (num: number): number
}
```

### 3. 构造签名

```ts
interface IConstructor {
  new (name: string): { name: string }
}
```

### 4. 函数重载

```ts
function format(value: string): string
function format(value: number): string
function format(value: string | number): string {
  return String(value)
}
```

应用场景：

- 不同参数类型对应不同逻辑
- 封装工具库 API 时很常见

### 5. 指定 this 类型

```ts
function print(this: { name: string }) {
  console.log(this.name)
}

print.call({ name: "Tom" })
```

应用场景：

- 一些老式函数写法里，明确 `this` 应该是什么结构

## 十一、类与面向对象

### 1. 类的定义

```ts
class Person {
  name: string
  age: number

  constructor(name: string, age: number) {
    this.name = name
    this.age = age
  }

  running() {
    console.log(this.name + " is running")
  }
}
```

应用场景：

- 封装业务模型
- 某些 SDK、工具类、服务类

### 2. 继承

```ts
class Student extends Person {
  sno: number

  constructor(name: string, age: number, sno: number) {
    super(name, age)
    this.sno = sno
  }
}
```

### 3. 成员修饰符

```ts
class User {
  public name: string
  private password: string
  protected token: string

  constructor(name: string, password: string, token: string) {
    this.name = name
    this.password = password
    this.token = token
  }
}
```

应用场景：

- `public`：对外公开字段
- `private`：只允许类内部使用
- `protected`：允许子类使用

### 4. readonly

```ts
class Article {
  readonly id: number

  constructor(id: number) {
    this.id = id
  }
}
```

应用场景：

- 订单 id、文章 id 这种创建后不该再改的数据

### 5. getter / setter

```ts
class PersonInfo {
  private _name = "Tom"

  get name() {
    return this._name
  }

  set name(value: string) {
    this._name = value.trim()
  }
}
```

### 6. 参数属性

```ts
class Product {
  constructor(public title: string, public price: number) {}
}
```

应用场景：

- 简化模型类的写法

### 7. 抽象类

```ts
abstract class Shape {
  abstract getArea(): number
}

class Circle extends Shape {
  constructor(public radius: number) {
    super()
  }

  getArea() {
    return Math.PI * this.radius * this.radius
  }
}
```

应用场景：

- 定义一组子类必须实现的能力

## 十二、接口、索引签名、枚举

### 1. 接口继承

```ts
interface Person {
  name: string
}

interface Student extends Person {
  sno: number
}
```

### 2. 接口实现

```ts
interface Runner {
  running(): void
}

class Dog implements Runner {
  running() {
    console.log("dog running")
  }
}
```

### 3. 索引签名

```ts
interface IStringMap {
  [key: string]: string
}

const messages: IStringMap = {
  title: "Hello",
  desc: "World"
}
```

应用场景：

- 国际化字典
- 配置表
- 动态 key 对象

### 4. 枚举

```ts
enum Direction {
  LEFT,
  RIGHT,
  TOP,
  BOTTOM
}

const moveDirection = Direction.LEFT
```

应用场景：

- 状态值
- 方向值
- 权限值

## 十三、泛型

### 1. 泛型函数

```ts
function identity<T>(value: T): T {
  return value
}

const num = identity<number>(100)
const str = identity<string>("hello")
```

应用场景：

- 工具函数希望输入输出保持同类型

### 2. 泛型接口

```ts
interface ApiResponse<T> {
  code: number
  data: T
  message: string
}
```

应用场景：

- 封装统一接口响应结构

### 3. 泛型类

```ts
class DataStore<T> {
  private items: T[] = []

  add(item: T) {
    this.items.push(item)
  }

  getAll() {
    return this.items
  }
}
```

应用场景：

- 通用仓库类
- 数据缓存容器

### 4. 泛型约束

```ts
interface Lengthwise {
  length: number
}

function getLength<T extends Lengthwise>(arg: T) {
  return arg.length
}
```

应用场景：

- 限制泛型必须具备某些属性

## 十四、映射类型、条件类型、工具类型

### 1. 映射类型

```ts
type MapUser<T> = {
  [P in keyof T]: boolean
}

type User = {
  name: string
  age: number
}

type UserPermissions = MapUser<User>
```

应用场景：

- 批量把一个对象的字段映射成另一套类型

### 2. 条件类型

```ts
type IsString<T> = T extends string ? true : false

type Result1 = IsString<string>
type Result2 = IsString<number>
```

### 3. infer

```ts
type ReturnType2<T> = T extends (...args: any[]) => infer R ? R : never
```

应用场景：

- 从函数类型中提取返回值
- 从 Promise 中提取 resolve 类型

### 4. 常见工具类型

#### `Partial<T>`

```ts
interface User {
  name: string
  age: number
}

type PartialUser = Partial<User>
```

应用场景：

- 编辑表单时只提交部分字段

#### `Required<T>`

```ts
type FullUser = Required<User>
```

#### `Readonly<T>`

```ts
type ReadonlyUser = Readonly<User>
```

#### `Pick<T, K>`

```ts
type UserPreview = Pick<User, "name">
```

应用场景：

- 只取接口中的一小部分字段

#### `Omit<T, K>`

```ts
type UserWithoutAge = Omit<User, "age">
```

#### `Record<K, T>`

```ts
type RoleMap = Record<string, number>

const roles: RoleMap = {
  admin: 1,
  editor: 2
}
```

应用场景：

- 字典对象
- 配置映射表

#### `Exclude<T, U>` / `Extract<T, U>`

```ts
type Status = "success" | "error" | "loading"

type OnlyDone = Exclude<Status, "loading">
type OnlyLoading = Extract<Status, "loading">
```

#### `NonNullable<T>`

```ts
type Value = string | null | undefined
type SafeValue = NonNullable<Value>
```

#### `ReturnType<T>`

```ts
function getUser() {
  return { name: "Tom", age: 18 }
}

type UserType = ReturnType<typeof getUser>
```

#### `InstanceType<T>`

```ts
class Person {
  name = "Tom"
}

type PersonInstance = InstanceType<typeof Person>
```

## 十五、模块化与声明文件

### 1. ES Module

```ts
// math.ts
export function add(a: number, b: number) {
  return a + b
}

// index.ts
import { add } from "./math"
console.log(add(1, 2))
```

应用场景：

- 现代 TypeScript 项目默认模块化写法

### 2. 第三方库声明

安装某些库时，经常需要类型声明：

```bash
npm install axios
npm install -D @types/node
```

应用场景：

- 使用第三方库时获得类型提示和校验

### 3. 自定义声明文件

```ts
// types/demo.d.ts
declare module "demo-lib" {
  export function format(value: string): string
}
```

应用场景：

- 第三方库没有类型声明时自己补
- 给图片、样式模块补声明

### 4. declare

```ts
declare const BASE_URL: string
```

应用场景：

- 某些全局变量由构建工具注入时，告诉 TypeScript 它存在

## 十六、tsconfig 基础

### 1. 一个最小配置

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "strict": true,
    "moduleResolution": "Node",
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

### 2. 常见选项理解

- `target`：编译到哪个 JS 版本
- `module`：模块化方案
- `strict`：是否开启严格模式
- `lib`：内置类型库范围
- `outDir`：输出目录
- `baseUrl` / `paths`：路径别名

应用场景：

- 团队统一编译规范
- 和 Vite / Webpack / Node 项目对接

## 十七、这套 TypeScript 内容的核心学习重点

如果时间有限，建议优先掌握：

### 第一优先级

- 基础类型
- 函数类型
- 对象类型
- 联合类型
- 接口与 type
- 泛型

### 第二优先级

- 类型缩小
- 类、接口、抽象类
- 工具类型
- 模块化与声明文件

### 第三优先级

- 枚举
- 条件类型
- `infer`
- `tsconfig` 细节

## 十八、容易混淆的知识点

### 1. any 和 unknown

- `any`：完全放弃检查
- `unknown`：更安全，使用前要缩小

### 2. interface 和 type

- `interface` 偏对象契约
- `type` 偏灵活组合

### 3. 联合类型和交叉类型

- 联合：二选一 / 多选一
- 交叉：同时具备

### 4. 可选属性和联合 `undefined`

```ts
type A = { age?: number }
type B = { age: number | undefined }
```

它们语义接近，但不完全一样。

### 5. 类型断言和真实运行结果

类型断言不会改变运行时值，它只是告诉编译器“我知道它是什么类型”。

## 十九、建议你的复习方式

### 第一轮

- 先把整份笔记顺一遍
- 明白每个知识点解决什么问题

### 第二轮

- 把每个代码示例手敲一遍
- 重点练：接口、联合类型、泛型、工具类型

### 第三轮

- 自己做小练习
- 例如：封装用户类型、接口响应类型、商品列表类型、泛型工具函数

### 第四轮

- 在一个 Vue / React 小项目里真实使用 TypeScript
- 把接口返回值、组件 props、store 状态都加上类型

## 二十、一个最小知识树

```text
TypeScript
├─ 基础
│  ├─ 基础类型
│  ├─ 函数类型
│  ├─ 对象类型
│  └─ 特殊类型(any / unknown / never / tuple)
├─ 类型系统
│  ├─ 联合类型
│  ├─ 接口 / type
│  ├─ 交叉类型
│  ├─ 类型缩小
│  └─ 断言
├─ 面向对象
│  ├─ class
│  ├─ 继承
│  ├─ abstract
│  ├─ interface
│  └─ enum
├─ 泛型与高级类型
│  ├─ 泛型函数
│  ├─ 泛型接口 / 类
│  ├─ 映射类型
│  ├─ 条件类型
│  └─ 工具类型
└─ 工程化
   ├─ 模块化
   ├─ 声明文件
   └─ tsconfig
```

## 二十一、学完这部分后的目标

学完这部分后，你至少应该能做到：

- 给函数、对象、数组、接口返回值写基础类型
- 理解联合类型、接口、泛型和工具类型
- 在 Vue / React 项目中为 props、state、接口数据加类型
- 看懂常见第三方库的 TypeScript 写法
- 配置基础的 `tsconfig.json`
