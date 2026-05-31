# 13 - TypeScript 高级类型(整理版)

> 源文档:`typescript笔记/03-TypeScript 高级类型.md`

---

## 一、知识点清单

### 1. 泛型 Generic

#### 概念

**泛型**:在保证类型安全的前提下,让函数 / 接口 / class **与多种类型协作**,实现复用。

#### 基本用法

```ts
function id<T>(value: T): T { return value }

// 显式传:
id<number>(10)
id<string>('a')

// 隐式推断(推荐):
id(10)         // T 被推断为 number
```

要点:
- `<T>` 是 **类型变量**(也叫类型参数),命名随意,常用 `T / U / K / V / Type`
- 编译时确定,运行时无痕(类型擦除)

#### 多个类型变量 + 约束

```ts
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}
const p = { name: 'jack', age: 18 }
getProp(p, 'name')   // 类型 string
getProp(p, 'xxx')    // 报错:'xxx' 不是 keyof p
```

- `extends ILength`:对类型变量加约束 —— "必须满足某接口"
- `extends keyof T`:跨变量约束 —— K 必须是 T 的键之一

#### 泛型接口

```ts
interface IdFunc<T> {
  id: (v: T) => T
  ids: () => T[]
}
const obj: IdFunc<number> = { id: v => v, ids: () => [1,2,3] }
```

JS 的数组在 TS 里就是泛型接口 `Array<T>`。

### 2. keyof

```ts
type Person = { name: string; age: number }
type Keys = keyof Person     // 'name' | 'age'
```

- 取对象类型的"键的联合"
- 配合泛型作约束:`K extends keyof T`(参数 key 必须是 T 的属性名)

### 3. 索引签名类型

```ts
interface AnyObject { [key: string]: number }
const o: AnyObject = { a: 1, b: 2 }

// 数组泛型接口的简化形态
interface MyArray<T> { [n: number]: T }
```

- 用于"我不知道有哪些 key,但 value 类型固定"的场景
- `key` 只是占位符,可以叫任意名字

### 4. 映射类型 Mapped Type

基于已有类型生成新类型:

```ts
type PropKeys = 'x' | 'y' | 'z'
type T = { [K in PropKeys]: number }
// 等价:type T = { x:number; y:number; z:number }

type Props = { a: number; b: string }
type Num   = { [K in keyof Props]: number }   // 把所有字段变成 number
```

- 只能在 `type` 中使用,不能在 `interface` 中使用
- `in` 表示"遍历联合类型的每个成员"(类似 `for..in`)

### 5. 索引访问类型 T[K]

```ts
type Props = { a: number; b: string; c: boolean }

type A    = Props['a']            // number
type AB   = Props['a' | 'b']      // number | string
type All  = Props[keyof Props]    // number | string | boolean
```

### 6. 内置工具类型(都是基于映射类型实现)

| 工具类型 | 作用 | 等价定义 |
| --- | --- | --- |
| `Partial<T>` | 所有属性变可选 | `{ [P in keyof T]?: T[P] }` |
| `Required<T>` | 所有属性变必选 | `{ [P in keyof T]-?: T[P] }` |
| `Readonly<T>` | 所有属性变只读 | `{ readonly [P in keyof T]: T[P] }` |
| `Pick<T, K>` | 从 T 中挑出 K 属性 | `{ [P in K]: T[P] }` |
| `Omit<T, K>` | 从 T 中剔除 K 属性 | `Pick<T, Exclude<keyof T, K>>` |
| `Record<K, V>` | 构造 key→value 对象类型 | `{ [P in K]: V }` |
| `Exclude<T, U>` | 从联合 T 排除可赋给 U 的部分 | 条件类型 |
| `Extract<T, U>` | 从联合 T 提取可赋给 U 的部分 | 条件类型 |
| `NonNullable<T>` | 排除 `null \| undefined` | 条件类型 |
| `ReturnType<F>` | 取函数返回值类型 | infer R |
| `Parameters<F>` | 取函数参数元组类型 | infer P |
| `Awaited<T>` | 解包 Promise 类型 | 递归 infer |

### 7. 条件类型(高阶)

```ts
T extends U ? X : Y
type IsString<T> = T extends string ? true : false
type A = IsString<'hi'>   // true
type B = IsString<123>    // false
```

加上 `infer` 可以"抽取"类型片段:

```ts
type GetReturn<F> = F extends (...args:any[]) => infer R ? R : never
type R = GetReturn<() => number>   // number
```

### 8. 类型守卫与收窄(常用)

```ts
function fn(x: string | number) {
  if (typeof x === 'string') x.toUpperCase()   // 收窄为 string
  else                       x.toFixed(2)       // 收窄为 number
}

interface Cat { meow(): void }
interface Dog { bark(): void }
function isCat(a: Cat | Dog): a is Cat {        // 自定义类型守卫
  return (a as Cat).meow !== undefined
}
```

---

## 二、多文件调用流程图

### 2.1 泛型函数被多个调用方复用

```
src/utils/id.ts
  export function id<T>(v: T): T { return v }
                │ export
                ▼
src/use-number.ts       src/use-string.ts
  const n = id(10)         const s = id('a')
  // T 推断为 number       // T 推断为 string
                │                       │
                ▼                       ▼
   编辑器看到 n: number     编辑器看到 s: string
   所有后续操作受 number    所有后续操作受 string
   类型保护                  类型保护
```

### 2.2 keyof + 泛型 = 类型安全的属性访问

```
调用方:                       泛型函数:
  obj = { name, age }    ─┐     getProp<T, K extends keyof T>(obj:T,key:K): T[K]
  getProp(obj, 'name') ──┤                  │
                          │                  ▼ 编译期
                          │      ┌──────────────────────────┐
                          │      │ T = { name, age }         │
                          │      │ K 必须 ∈ 'name' | 'age'   │
                          │      └──────────────────────────┘
  getProp(obj, 'xx')  ───┘
                                  ↓ 'xx' ∉ keyof T
                                  ↓
                          编辑器/编译器 报错
                          运行前阻止 typo
```

### 2.3 映射类型派生新类型 一图流

```
type User = { id:string; name:string; age:number }
        │
        │  Partial<User>
        ▼
{ id?:string; name?:string; age?:number }
        │
        │  Pick<User, 'id'|'name'>
        ▼
{ id:string; name:string }
        │
        │  Omit<User, 'age'>
        ▼
{ id:string; name:string }
        │
        │  Readonly<User>
        ▼
{ readonly id:string; readonly name:string; readonly age:number }

这些"派生"全部在编译期完成,运行时没有任何痕迹。
```

---

## 三、勘误与修正

| # | 原文 | 问题 | 正确表述 |
| --- | --- | --- | --- |
| 1 | "**映射类型只能在类型别名中使用,不能在接口中使用**" | 表述准确,但缺补充 | 准确说法:`[K in U]` 这种语法仅出现在 `type ... = {...}`;`interface` 不支持映射语法,但可以 `extends` 一个映射出来的类型再扩展属性 |
| 2 | "推荐:使用 keyof 操作符" 没区分 keyof 与 JS 的 Object.keys | 易混淆 | `keyof T` 是 **类型层面**,返回 **联合类型字面量**;`Object.keys(obj)` 是 **运行时**,返回 `string[]`。两者完全不同维度 |
| 3 | "Omit<K, T>" 解释 | 顺序写反 | 正确签名是 **`Omit<T, K>`**:T 是源类型、K 是要剔除的键名联合;原文标注的"K 是对象类型名称、T 是剔除属性"颠倒了 |
| 4 | "在调用泛型函数时,可以省略 `<类型>` 来简化泛型函数的调用" | 表述对,但缺反例 | 当编译器 **无法推断**(无参数 / 多参数推断歧义)或推断 **不准确** 时,必须显式传 `<T>`。例如 `JSON.parse<MyType>(s)`(注:其实 JSON.parse 没有该签名,需自定义 helper) |
| 5 | "PartialProps... 所有属性都变为可选的" 但没说陷阱 | 不完整 | Partial **只递归一层**,深层对象属性不会变可选。需要"深度 Partial"得自己写 `DeepPartial<T>`(递归条件类型) |
| 6 | "通过 extends 关键字使用该接口,为泛型添加约束" | 字面对,但易引起歧义 | TS 里 `extends` 在不同上下文有不同含义:① 类继承;② 接口继承;③ **泛型约束**`<T extends U>`(此处);④ **条件类型** `T extends U ? X : Y`。同一关键字四种用法,要会区分 |
| 7 | "应该用 object,而不是 Object" 旁注 | 描述准确,但太轻 | 这是一个 **重要的坑**:`object`(小写)= 非原始值;`Object` = 几乎所有值(含原始);`{}` = 几乎所有值。约束对象 **始终用小写 `object`** 或更精确的 `Record<string, unknown>` |
| 8 | 没区分 `extends keyof T` 与 `in keyof T` | 易混淆 | `extends keyof T`:**泛型约束**(类型参数声明位置);`in keyof T`:**映射类型**(遍历键)。同样关键字 `keyof`,两种语境下用法不同 |
| 9 | 缺 **条件类型 + infer** 这一重要主题 | 重大遗漏 | TS 高级类型的核心机制之一:`T extends U ? X : Y` + `infer R`。`ReturnType / Parameters / Awaited` 等内置工具都基于它 |
| 10 | "索引签名类型 `[key: string]`,key 是占位符" | 描述对,但缺约束 | 索引签名类型一旦声明,**其他显式属性的类型必须兼容索引签名值类型**,例如 `[k:string]:number` 时不能再写 `name: string` |
| 11 | 关于 `as const` 完全没提 | 重要缺失 | `as const` 把字面量"锁死"成具体值:`const dirs = ['up','down'] as const` → `readonly ['up','down']`。配合 `typeof + [number]` 可以从值生成字面量联合,工程实用度很高 |
| 12 | "[n: number] 模拟原生数组" 解释 | 准确,但缺真相 | TS 标准库定义的数组类型 `Array<T>` 在 `lib.es5.d.ts` 中,数十个方法签名都用到了泛型和索引签名。读源码可以学到大量类型技巧 |

