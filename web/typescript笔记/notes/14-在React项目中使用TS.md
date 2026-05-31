# 14 - 在 React 项目中使用 TS(整理版)

> 源文档:`typescript笔记/04-在 React 项目中使用 TS.md`

---

## 一、知识点清单

### 1. 创建 TS 版 React 项目

```bash
npx create-react-app my-app --template typescript
```

新增的关键文件:
- `tsconfig.json` —— TS 编译配置
- 文件后缀:`.js` → `.ts` / `.tsx`(包含 JSX 的必须用 `.tsx`)
- `src/react-app-env.d.ts` —— CRA 注入的环境类型声明(支持 CSS modules、svg 等导入)

### 2. tsconfig.json 关键字段速查

| 字段 | 作用 | 常见取值 |
| --- | --- | --- |
| `target` | 编译输出的 ES 版本 | `es5 / es2017 / esnext` |
| `module` | 输出的模块化方案 | `esnext / commonjs` |
| `lib` | 编译时可用的内置类型库 | `["dom","dom.iterable","esnext"]` |
| `jsx` | JSX 编译方式 | `react-jsx`(新)/ `react`(旧) |
| `strict` | 严格类型检查总开关 | `true`(强烈推荐) |
| `esModuleInterop` | 抹平 CJS/ESM 差异 | `true` |
| `allowSyntheticDefaultImports` | 允许默认导入无 default 的模块 | `true` |
| `allowJs` | 允许 JS 文件参与编译 | 渐进迁移用 true |
| `skipLibCheck` | 跳过依赖里 .d.ts 的类型检查 | `true`(提速) |
| `forceConsistentCasingInFileNames` | 大小写一致(防 Linux 部署翻车) | `true` |
| `noFallthroughCasesInSwitch` | switch 缺少 break 时报错 | `true` |
| `moduleResolution` | 模块查找策略 | `node` |
| `resolveJsonModule` | 允许 import json | `true` |
| `isolatedModules` | 每个文件可独立编译 | `true` |
| `noEmit` | 只检查不输出 | `true`(由 Babel 实际编译时) |
| `include / exclude` | 编译范围 | `["src"]` / `["node_modules"]` |
| `baseUrl / paths` | 路径别名 | `{"@/*": ["src/*"]}` |

`tsc --init` 可以生成一份带注释的默认 tsconfig 模板。

### 3. 两种文件类型:`.ts` vs `.d.ts`

| 维度 | `.ts` | `.d.ts` |
| --- | --- | --- |
| 内容 | 既有类型又有可执行代码 | **只有类型,不允许出现可执行语句** |
| 输出 | 编译后产出 `.js` | 不产物,纯类型 |
| 用途 | 写业务/逻辑 | 为 JS 库 / 已有 JS 文件提供类型 |

### 4. 类型声明文件的来源

1. **TS 内置**:`lib.es5.d.ts`、`lib.dom.d.ts` 等(Array / Window / Document)
2. **库自带**:库的 `package.json` 中标 `"types": "dist/index.d.ts"`(如 axios)
3. **DefinitelyTyped 社区维护**:`@types/lodash`、`@types/react` 等,装到 devDependencies
4. **项目内自建**:`src/types/*.d.ts` 或 `src/x.d.ts` 与 JS 文件同名

查询社区类型:https://www.typescriptlang.org/dt

### 5. `declare` 关键字

为"别处已存在的运行时变量"标注类型,而不创建新值:

```ts
declare let count: number
declare function add(a:number,b:number): number
declare const config: { apiBase: string }
declare module '*.svg' {              // 给非 JS 资源声明类型
  const src: string
  export default src
}
```

规则:
- `type / interface / enum` 已经是 TS 专属,**不需要 declare**
- `let / const / function / class / namespace / module` 在 JS 也存在,**必须用 declare** 表明"此处仅声明类型"

### 6. 项目内类型共享

```
src/types/
  └─ index.d.ts            # 公共类型
src/components/Foo.tsx     # import type { User } from '../types'
```

约定:
- 公共业务类型放 `src/types/` 或就近放在使用模块
- `import type { ... }` 显式表明只导入类型(更优,被 erased 编译时移除)

### 7. 在现有 JS 项目中加 TS

1. `npm i -D typescript @types/node @types/react @types/react-dom @types/jest`
2. 项目根目录添加 `tsconfig.json`(可参考 CRA 模板)
3. 复制 `react-app-env.d.ts` 到 `src/`
4. 把 `path / 别名` 等原 `jsconfig.json` 中的配置放进 `tsconfig.json`(或 `path.tsconfig.json` extends)
5. 重启 dev server
6. **渐进迁移**:新组件用 `.tsx`,旧 `.js` 暂留;`allowJs:true` 让二者共存

### 8. 文件后缀对照

| 老 | 新 |
| --- | --- |
| `.js` 普通逻辑 | `.ts` |
| `.jsx` 含 JSX 的组件 | **`.tsx`**(必须) |
| `.json` | 不变,需要 `resolveJsonModule:true` |

---

## 二、多文件调用流程图

### 2.1 React + TS 项目编译链

```
开发者
  └─ App.tsx, Foo.tsx, utils.ts, types.d.ts
        │ import
        ▼
   ┌──────────────────────────────────────────────────┐
   │ Webpack / Vite                                    │
   │   ├─ ts-loader / @babel/preset-typescript / SWC   │
   │   │    把 TS 转 JS(类型擦除)                     │
   │   ├─ tsc --noEmit (单独/CI 中跑) 做类型检查        │
   │   └─ 输出 bundle.js                                │
   └────────────────┬──────────────────────────────────┘
                    │
                    ▼
              浏览器执行(纯 JS)

CRA 内部:
   Babel 负责快速编译 → tsc 单独跑 fork-ts-checker-webpack-plugin 做检查
   两条流水线并行,既快又类型安全
```

### 2.2 类型声明文件的解析顺序

```
import axios from 'axios'
        │
        ▼
   TS 在 node_modules/axios 寻找:
     1) package.json 的 "types" 字段
     2) 同级 index.d.ts
                │
                ├─ 找到 → 加载,提供类型
                │
                └─ 没找到 → 看 node_modules/@types/axios
                              │
                              ├─ 找到 → 加载
                              │
                              └─ 没找到 → 在 IDE 中标黄
                                           需自己 declare module 'axios'
```

### 2.3 项目内类型共享

```
       types/api.d.ts                 types/user.d.ts
       export type Resp<T>            export interface User {...}
                │                              │
                │ import type { Resp }         │ import type { User }
                ▼                              ▼
       api/userApi.ts ────────────────▶ 业务逻辑
                │
                │ 返回 Resp<User>
                ▼
       components/UserCard.tsx
                │ 渲染时享受 User 类型补全
                ▼
              UI
```

### 2.4 declare module + 资源导入(svg/css)

```
src/global.d.ts
   declare module '*.svg' { const x: string; export default x }
   declare module '*.module.css' {
     const classes: { [k:string]: string }
     export default classes
   }
            │
            ▼
src/Foo.tsx
   import logo from './logo.svg'           ← 类型 string
   import s from './Foo.module.css'        ← 类型 {[k:string]:string}
   <img src={logo}/>
   <div className={s.title}/>
```

---

## 三、勘误与修正

| # | 原文 | 问题 | 正确表述 |
| --- | --- | --- | --- |
| 1 | "命令:`npx create-react-app my-app --template typescript`" | 仍可用但 CRA 已逐渐式微 | 2023 后 CRA 处于半维护状态;新项目推荐 **Vite (`npm create vite@latest my-app -- --template react-ts`)** 或 **Next.js (`create-next-app --typescript`)**,启动更快、HMR 更稳 |
| 2 | "**.d.ts 文件中不允许出现可执行的代码**" | 表述太绝对 | `.d.ts` 中 **不允许** 任何会产生 JS 输出的语句(变量初始化、函数体);但允许 `declare`、`interface`、`type`、`namespace`、`export ... from ...` 等纯类型声明 |
| 3 | "tsconfig.json 中 `jsx`: `react-jsx`" | 没解释这个选项 | TS 4.1+ 引入的新 JSX 编译模式,**不需要 `import React from 'react'`** 即可用 JSX(配合 React 17+ 的新 JSX runtime)。旧选项 `react` 仍需手动 import |
| 4 | "把 jsconfig.json 改成 path.tsconfig.json" 步骤 | 表述混乱 | 正确做法:① 删除 jsconfig.json(或保留);② 新增 `tsconfig.json`(主配置);③ 别名等可以保留单独的 `path.tsconfig.json`,在 `tsconfig.json` 通过 `"extends": "./path.tsconfig.json"` 引入。原文叙述顺序拼接错乱 |
| 5 | 步骤序号从 1 跳到 2,再跳到 4、5,然后两个 5 | 编号错误 | 重新排序为 1-7 步即可。已在上方"在现有 JS 项目中加 TS"里列清 |
| 6 | 示例 `function fomartPoint` 和导出 | 拼写错误且类型声明不一致 | 应为 **`formatPoint`**;且 .d.ts 的 export 中既导出 `FomartPoint`(类型) 又导出 `fomartPoint`(值),拼写一致才能被对接 |
| 7 | "TS 也会自动加载该类声明包" 中的"类声明包" | 错别字 | 应为"**类型声明包**" |
| 8 | "DefinitelyTyped 包名 `@types/*`" 没说装到 dev 还是 prod | 缺关键 | `@types/*` 必须装到 **`devDependencies`**;它们仅在编译期使用,运行时不需要 |
| 9 | 没提 `import type` 与 `verbatimModuleSyntax` | 缺现代特性 | 现代实践:**`import type { X }`** 显式标识"仅类型导入",编译时被完全擦除;TS 5.0+ 提供 `verbatimModuleSyntax` 选项强制使用 `import type`,防止类型/值意外保留到运行时 |
| 10 | "react-app-env.d.ts" 描述含糊 | 不完整 | 该文件包含 `/// <reference types="react-scripts" />`,**引入 CRA 为各种资源(svg、css module、png 等)提供的类型声明**。Vite 等价文件是 `src/vite-env.d.ts` |
| 11 | "allowJs: true" 用于 CRA | 描述对 | 在迁移阶段尤其重要;还要配合 **`checkJs: true`** 才会对 JS 也做类型检查(可结合 JSDoc) |
| 12 | tsconfig 中没提 **`incremental` / `composite` / 项目引用** | 大项目优化遗漏 | `incremental:true` + 缓存文件,可让 tsc 增量编译;monorepo 中用 **`composite:true` + `references`** 做"项目引用",并行编译多个子包 |

