# 12 - TypeScript 常用类型(整理版)

> 源文档:`typescript笔记/02-TypeScript 常用类型.md`

---

## 一、知识点清单

### 1. 类型注解

```ts
let age: number = 18      // :number 即"类型注解"
```

- 写在变量名后,用 `:类型` 形式
- 一旦标注,赋值必须匹配类型,否则编译报错

### 2. 类型分类

| 大类 | 类型 |
| --- | --- |
| 原始类型 | `number / string / boolean / null / undefined / symbol / bigint` |
| 对象类型 | 数组、对象、函数、Date、Map、Set 等 |
| TS 新增 | 联合类型、字面量类型、字面量联合、元组、枚举、接口、类型别名、`any / unknown / never / void`、字面量、字面量联合 |

### 3. 数组类型

```ts
let a: number[] = [1,2,3]         // 推荐
let b: Array<number> = [1,2,3]    // 等价的泛型写法
```

### 4. 联合类型

```ts
let arr: (number | string)[] = [1, 'a']
let id: number | string
```

`|` = "或者",不要与 JS 的逻辑 `||` 混淆。

### 5. 类型别名 type

```ts
type CustomArray = (number | string)[]
type Id = number | string
type User = { name: string; age: number }
```

- 用于"复杂类型起短名字"
- 命名约定首字母大写
- 可以为 **任意类型** 起别名(不限于对象)

### 6. 函数类型

#### 单独标注参数、返回值

```ts
function add(a: number, b: number): number { return a + b }
const sub = (a: number, b: number): number => a - b
```

#### 整体标注(只能用于函数表达式)

```ts
type AddFn = (a: number, b: number) => number
const add: AddFn = (a, b) => a + b
```

#### void

```ts
function log(msg: string): void { console.log(msg) }
```

- 无返回值用 `void`,**不要写 `undefined`**(写 undefined 表示必须 `return undefined`)

#### 可选参数 / 默认参数

```ts
function f(a: number, b?: number) {}     // 可选
function f(a: number, b: number = 0) {}  // 默认值,自动可选
```

- 可选参数 `?` **只能出现在末尾**

### 7. 对象类型

```ts
let p: { name: string; age: number; greet(): void } = {...}

// 推荐用 type / interface 抽取
type Person = {
  name: string
  age: number
  greet(name: string): void          // 方法签名
  hi?: () => void                    // 可选方法,箭头函数形式
}
```

- 同一行多属性用 `;`,跨行可省略
- 可选属性同样用 `?`

### 8. interface 与 type 对比

| 特性 | interface | type |
| --- | --- | --- |
| 描述对象/函数 | ✅ | ✅ |
| 联合 / 交叉 | ❌ | ✅(`A \| B`, `A & B`) |
| 元组 / 基本类型别名 | ❌ | ✅ |
| 同名"声明合并" | ✅(多次声明会合并) | ❌(重复报错) |
| 继承 | `extends` | 交叉 `&` |

**经验**:
- 公共库 / 需要被第三方扩展 → `interface`
- 内部应用 / 复杂复合类型 → `type`

### 9. 接口继承

```ts
interface Point2D { x: number; y: number }
interface Point3D extends Point2D { z: number }
```

### 10. 元组 Tuple

```ts
let position: [number, number] = [39.5, 116.2]
let info: [string, number, boolean] = ['ke', 30, true]
```

固定 **长度 + 每个位置的类型**,与普通数组区分。

### 11. 类型推论

- 声明变量 + 初始化时,会自动推导;函数返回值也会推导

```ts
let x = 18                // x: number
function add(a:number,b:number) { return a+b } // 返回值推导为 number
```

- 推荐:**能省略就省略**;不能推断时再标注

### 12. 字面量类型

```ts
const s = 'Hello'         // 类型是 'Hello'(字面量类型)
let   s2 = 'Hello'        // 类型是 string
```

`const` 锁住具体值,`let` 是更宽泛的同类型。

#### 字面量联合(常用模式)

```ts
type Direction = 'up' | 'down' | 'left' | 'right'
function move(d: Direction) {}
move('up')               // ok
move('north')            // 编译错
```

可读、可补全、可精确受限。

### 13. enum(了解)

```ts
enum Direction { Up, Down, Left, Right }   // 数字枚举,默认 0,1,2,3
enum Color { R = 'RED', G = 'GREEN' }      // 字符串枚举,必须给值
```

- 与字面量联合的差异:**枚举会编译成真实的 JS 对象**(占体积、影响 tree-shaking)
- 现代项目里 **推荐字面量联合 + `as const`** 替代枚举

### 14. any / unknown / never

| 类型 | 含义 | 何时使用 |
| --- | --- | --- |
| `any` | 关掉类型检查,任意操作不报错 | 临时绕过,**尽量避免** |
| `unknown` | 未知类型,**比 any 安全**,使用前必须收窄 | 接收不可信数据(如 `JSON.parse`) |
| `never` | 永远不会发生的值 | 抛错函数返回值 / 永真分支兜底 |

```ts
function fail(msg: string): never { throw new Error(msg) }
let val: unknown = JSON.parse(s)
if (typeof val === 'string') val.toUpperCase()  // 收窄后才能用
```

### 15. 类型断言 as

```ts
const link = document.getElementById('link') as HTMLAnchorElement
link.href            // ok
```

- 作用:**告诉编译器你比它更清楚**;运行时 **不做任何转换**
- `<HTMLAnchorElement>x` 是旧写法,**JSX/TSX 中冲突**,统一用 `as`
- 双重断言(逃生口):`x as unknown as T`(慎用)

### 16. typeof 类型查询

```ts
const p = { x: 1, y: 2 }
type P = typeof p                 // { x: number; y: number }
function fmt(point: typeof p) {}
```

注意:此处 `typeof` 是 **类型上下文**(在 `:` 之后或 `type` 里),不同于 JS 的 `typeof`。

---

## 二、多文件调用流程图

### 2.1 类型如何在不同 .ts 文件间传递

```
src/types/User.ts
   export interface User { id: string; name: string }
            │ export
            ▼
src/api/userApi.ts
   import { User } from '../types/User'
   export async function getUser(id: string): Promise<User> {
     return fetch('/u/' + id).then(r => r.json())   // 编译期保证返回类型
   }
            │ export
            ▼
src/components/UserCard.tsx
   import { getUser } from '../api/userApi'
   import type { User } from '../types/User'         // 仅类型导入(更优)
   const user: User = await getUser('1')
            │
            ▼
   编辑器 / tsc 全链路类型校验:任何一处签名变了,所有 import 处都会标红
```

### 2.2 类型推论 + 字面量类型 + 联合的协作

```
type Status = 'idle' | 'loading' | 'success' | 'error'

setState(s: Status) {}
                                  ┌─────────────────────────┐
传入字符串 'loading'          ─▶ │ 字面量 'loading' 属于    │ ✅ 通过
                                  │ Status 联合 → 合法       │
                                  └─────────────────────────┘
传入字符串 'fetching'        ─▶ │ 字面量 'fetching' 不属于 │ ❌ 报错
                                  └─────────────────────────┘

在 switch (status) 中
   case 'idle':   ...
   case 'loading':...
   case 'fetching': ← 多此一举,编辑器会标记 dead branch
   default: const _: never = status  ← 编译器穷尽性检查
```

### 2.3 模块化中的类型擦除

```
源文件
  user.ts
    export interface User { id: string }
    export const ke: User = { id: 'ke' }

经过 tsc 编译:
  ┌───────────────────────────┐
  │ user.js  (运行时使用)      │
  │   export const ke = {id:'ke'}│   ← interface 被擦掉
  └───────────────────────────┘
  ┌───────────────────────────┐
  │ user.d.ts (供类型导入)     │
  │   export interface User {…}│
  │   export const ke: User    │
  └───────────────────────────┘

发布 npm 包时,JS 跑代码 + d.ts 提供类型,二合一
```

---

## 三、勘误与修正

| # | 原文 | 问题 | 正确表述 |
| --- | --- | --- | --- |
| 1 | "推荐:**能使用 type 就使用 type**" | 表述武断 | 主流社区指南(包括 TS 官方 handbook):**没有绝对推荐**。`interface` 在描述对象/类形状、被外部扩展(如组件库)时更合适;`type` 在描述联合 / 元组 / 基本类型别名时更合适。两者择最匹配场景使用 |
| 2 | "如果什么都不写,此时 add 函数的返回值类型为 void" 示例 `const add = () => {}` | 不严谨 | TS 推断空箭头函数的返回值为 **`void`**,这是对的;但实际 JS 中 `add()` 返回 `undefined`。`void` 是 TS 的 **类型**,`undefined` 是 JS 中的 **值**,两者不要混淆 |
| 3 | "let person: { name: string } = { name: '同学' }" | 写法对,但缺关键规则 | TS 对对象有 **结构子类型**:多余的属性默认报错(尤其字面量赋值时,称作"excess property check");但若先存到变量再赋值,则只检查必要属性。这是初学常踩的坑 |
| 4 | "字面量 `{ name: 'jack' }` `18` `'abc'` `false` `function() {}` 都可以做类型" | 写法对,但函数字面量做类型几乎不会用 | 实际中常用作类型的字面量:**字符串、数字、布尔、null、undefined**;对象字面量更多用 `as const` 配合提取 |
| 5 | 枚举 "推荐使用字面量类型 + 联合类型组合" 提示 | 表述对,但缺新方案 | 现代 TS 实践常用:`const Direction = { Up:'UP', Down:'DOWN' } as const; type Direction = typeof Direction[keyof typeof Direction]`。既有运行时值又有字面量联合类型,**比 enum 体积小** |
| 6 | "any 让 TS 变成 AnyScript" 强调避免 | 没提 `unknown` 这个更安全的替代 | unknown 应优先于 any。`unknown` 强制开发者在使用前收窄类型,既保留灵活性,又不丢失类型安全 |
| 7 | 类型断言示例 `aLink as HTMLAnchorElement` | 没说断言的风险 | 断言 **不会真的转换值**,只是让 TS 闭嘴。如果断言错了(实际是 div 被断言成 a),运行时仍然会报错。**安全做法:先 `instanceof` 校验,再使用** |
| 8 | "typeof 出现在类型注解的位置(参数名称的冒号后面)所处的环境就在类型上下文" | 表述含混 | 准确说:**类型位置**包括 `:` 之后、`type X = ...`、`interface` 内部、泛型 `<T>` 等;在这些位置,`typeof` 表示"取这个变量(运行时值)的类型" |
| 9 | "枚举编译成 JS 代码" 示例存在语法错误:`(Direction || Direction = {})` | 输出 JS 写错 | 标准编译输出应是 `(Direction = {}))` 在 `||` 短路时赋值;原文示例括号错误。正确形式:`(Direction || (Direction = {}))` |
| 10 | 没提 **strictNullChecks** | 重要缺失 | 在 strict 模式下,`null` 与 `undefined` 不能赋给其他类型;要表达"可能为空"必须显式联合 `string \| null` 或使用 `?:`。这是工程中最常遇到的类型错误来源 |
| 11 | "TS 提供了 typeof 操作符" 与 JS 的 typeof 易混淆 | 强调不足 | **JS 的 typeof 是运行时操作符**(返回字符串如 'number');**TS 的 typeof 是类型操作符**(返回类型本身)。它们书写一样,但靠上下文区分,初学者必须意识到 |
| 12 | "可选参数后面不能再出现必选参数" | 描述准确,但缺工程建议 | 实际中:**接口/对象的可选属性可以放任意位置**,只有 **函数参数** 受此限制。若希望参数顺序自由,常用对象作为单一参数(options 模式) |

