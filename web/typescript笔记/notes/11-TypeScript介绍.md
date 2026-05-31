# 11 - TypeScript 介绍(整理版)

> 源文档:`typescript笔记/01-TypeScript 介绍.md`

---

## 一、知识点清单

### 1. TypeScript 是什么

- **TS = Type + JavaScript**:在 JS 基础上添加 **静态类型系统**
- TS 是 JS 的 **超集** (superset):所有合法 JS 都是合法 TS
- 微软开源,**任何能跑 JS 的地方都能跑编译后的 TS**
- 编译目标:`*.ts` → `*.js`,运行环境拿到的依然是纯 JS

### 2. 为什么需要 TS

| 维度 | JavaScript | TypeScript |
| --- | --- | --- |
| 类型检查时机 | 运行时(Runtime) | 编译时(Compile time) |
| 错误发现 | 晚:浏览器跑起来才报错 | 早:写代码 / 保存时编辑器即报错 |
| 代码提示 | 弱(只能靠 JSDoc) | 强(基于类型推导) |
| 重构 | 易漏改 | 类型驱动,改了一处编辑器全标红 |
| 大型项目 | 难维护 | 适合长期维护 |

### 3. TS 主要优势

1. **更早发现错误**(写代码同时即报错)
2. **完善的代码提示和补全**
3. **强类型系统让重构安全**
4. **可以使用最新的 ES 语法**(TS 内置降级编译)
5. **类型推断**让你不用到处显式写类型

### 4. 行业现状

- React + TS 是大中型项目标配
- Vue 3 源码用 TS 重写;Angular 默认就是 TS
- 主流 IDE (VSCode / WebStorm) 对 TS 一等公民支持
- npm 上大量库自带 `.d.ts` 类型声明,或可通过 `@types/xxx` 装类型

### 5. 安装与编译

```bash
# 全局安装 tsc 编译器
npm i -g typescript

# 验证
tsc -v

# 编译单文件:hello.ts -> hello.js
tsc hello.ts

# 执行
node hello.js
```

### 6. 简化:`ts-node`

```bash
npm i -g ts-node
ts-node hello.ts   # 一步搞定,不会落地 .js 文件
```

> ts-node 内部仍调用 TS 编译器,把 TS 在内存里翻译成 JS 再交给 Node。

### 7. 工程化使用 (tsconfig.json)

实际项目用 `tsconfig.json` 集中配置:

```bash
tsc --init   # 生成 tsconfig.json
tsc          # 不带文件名,按 tsconfig 编译整个工程
```

关键字段(后续文档展开):
- `target`:编译输出的 ES 版本
- `module`:模块化方案
- `strict`:严格类型(推荐 true)
- `jsx`:React 的 JSX 处理
- `outDir / rootDir`:输出 / 输入目录
- `include / exclude`:扫哪些文件

---

## 二、多文件调用流程图

### 2.1 TS 从源码到运行的链路

```
开发者
   │ 写
   ▼
┌──────────────────┐
│ src/foo.ts        │
│ src/bar.ts        │ ── ESM/CJS 互相 import ──┐
└────────┬──────────┘                          │
         │ tsc(或 ts-node、Babel、SWC、esbuild)│
         ▼                                     │
┌──────────────────────────────────────────────┘
│ 编译阶段(纯静态)
│   1. 解析每个 .ts → AST
│   2. 类型检查(发现错误就报错,可能终止)
│   3. 去掉类型注解(类型擦除)
│   4. 按 target 把新语法转译成旧语法
│   5. 按 module 配置生成模块代码
└────────┬─────────────────────────────────────
         ▼
┌──────────────────┐
│ dist/foo.js       │
│ dist/bar.js       │
└────────┬──────────┘
         │ node / 浏览器 运行
         ▼
        结果
```

### 2.2 tsc vs ts-node

```
方式 A:tsc + node
  ┌─────────┐  tsc  ┌─────────┐  node   输出
  │ a.ts    │ ────▶│ a.js    │ ───────▶
  └─────────┘       └─────────┘
  → 落地 js,适合发布到生产

方式 B:ts-node
  ┌─────────┐    ts-node a.ts
  │ a.ts    │  ┌─────────────────────┐   输出
  └─────────┘─▶│ 内存中转译 + node 执行 │────▶
               └─────────────────────┘
  → 不落地 js,适合开发 / 脚本

方式 C:打包器(webpack/vite + ts-loader 或 babel)
  → 用于实际 SPA 项目,见 React 工程
```

### 2.3 编辑器 + tsc 实时反馈

```
   编辑器(VSCode)
         │
         │  调用内置 TypeScript Language Server
         ▼
   ┌────────────────────────────┐
   │ 增量解析所有 import 的文件   │
   │ 增量类型检查                 │
   │ 输出诊断结果(红波浪线)      │
   └────────────────────────────┘
         │
         ▼
   用户保存时(可选)
   ┌────────────────────────────┐
   │ tsc --noEmit               │  ← 仅做检查,不输出
   │ (或 CI 流水线中作为 gate)   │
   └────────────────────────────┘
```

---

## 三、勘误与修正

| # | 原文 | 问题 | 正确表述 |
| --- | --- | --- | --- |
| 1 | "TypeScript 是 JavaScript 的超集" 配图 | 表述对,但不全 | 严谨说法:TS = JS 语法的超集 + 静态类型系统 + 一些类型相关语法(`interface / enum / type / as` 等)。**类型系统部分不能在 JS 中运行**,会被编译时擦除 |
| 2 | "Vue 2 对 TS 的支持不好" | 已过时 | Vue 3 与 TS 深度结合;Vue 2.7 也加入了 Composition API,TS 支持比早期好很多;另外 Vue 3 + `<script setup>` 是当前主流写法 |
| 3 | "TS 中文参考 - 不再维护" 但仍列在文档头 | 容易误导新手 | 推荐 **官方英文文档**(typescriptlang.org)或社区维护更新版,而非已不更新的镜像。中文学习材料首选官方"TypeScript 中文文档"(typescriptlang.org/zh) |
| 4 | "tsc 命令实现了 TS → JS 的转化" | 不全面 | tsc 同时做 **类型检查 + 编译输出**;可以分别执行(`tsc --noEmit` 只检查不输出),也可以让 **Babel/SWC/esbuild 只负责编译**,把类型检查交给独立的 `tsc` 进程(大型项目通用做法,因 Babel 不做类型检查、速度快) |
| 5 | "ts-node hello.ts 相当于 1 tsc 命令 2 node" | 解释含糊 | 准确说法:ts-node 把 `.ts` 文件 hook 进 Node.js 的 require/import 加载器,**在内存里转译再交给 V8 执行**,不会写出 `.js` 文件 |
| 6 | "由 TS 编译生成的 JS 文件,代码中就没有类型信息了" | 描述对,但要补充 | 这种现象叫 **类型擦除 (type erasure)**;但若开启 `declaration: true`,tsc 还会同时输出 `.d.ts` 文件,保留类型信息供下游消费 |
| 7 | "强大的类型系统提升了代码的可维护性,使得**重构代码更加容易**" | 没说成本 | 收益是真的,但代价是:① 学习曲线;② 早期编码慢;③ 类型本身有时复杂(高级泛型)。Pareto 实践:80% 场景只用基础类型;**不要为了 100% 类型化而过度工程**(过度的 any/unknown 反而抵消好处) |
| 8 | 没提 **tsconfig.json / strict 模式** | 重要遗漏 | 真实工程的 TS 几乎都靠 `tsconfig.json` 驱动,默认推荐 **`strict: true`**(打开严格 null / 严格函数类型 / 严格属性初始化等),否则等于裸奔 |
| 9 | "TS 类型推断机制,不需要在代码中的每个地方都显示标注类型" | "显示" 是错别字 | 应为 **"显式"标注** |
| 10 | 没说 TS 不能阻止 **运行时** 错误 | 重要边界 | TS 只在编译期保证类型对齐;**运行时数据**(网络请求返回、用户输入、`JSON.parse` 等)仍可能与声明的类型不符。需要时配合 `zod / yup / io-ts` 等运行时校验库 |

