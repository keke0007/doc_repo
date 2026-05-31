# 在 React 项目中使用 TS

1. 创建新项目
2. 在现有项目中添加 TS

## 创建新项目

- 命令：`npx create-react-app my-app --template typescript`
- 说明：在命令行中，添加 `--template typescript` 表示创建支持 TS 的项目

- 项目目录的变化：
  1. 在项目根目录中多了一个文件：`tsconfig.json`
     - TS 的配置文件
  2. 在 src 目录中，文件的后缀有变化，由原来的 .js 变为 `.ts` 或 `.tsx`
     - `.ts` ts 文件的后缀名
     - `.tsx` 是在 TS 中使用 React 组件时，需要使用该后缀
  3. 在 src 目录中，多了 `react-app-env.d.ts` 文件
     - `.d.ts` 类型声明文件，用来指定类型

## tsconfig的介绍

+ tsconfig.json是typescript项目的配置文件，用于配置typescript
+ tsconfig.json配置文件可以通过 `tsc --init` 生成

- 说明：所有的配置项都可以通过鼠标移入的方式，来查看配置项的解释说明。
- [tsconfig 文档链接](https://www.typescriptlang.org/tsconfig)

```json
{
  // 编译选项
  "compilerOptions": {
    // 生成代码的语言版本：将我们写的 TS 代码编译成哪个版本的 JS 代码
    // 命令行： tsc --target es5 11-测试TS配置文件.ts
    "target": "es5",
    // 指定要包含在编译中的 library
    "lib": ["dom", "dom.iterable", "esnext"],
    // 允许 ts 编译器编译 js 文件
    "allowJs": true,
    // 跳过类型声明文件的类型检查
    "skipLibCheck": true,
    // es 模块 互操作，屏蔽 ESModule 和 CommonJS 之间的差异
    "esModuleInterop": true,
    // 允许通过 import x from 'y' 即使模块没有显式指定 default 导出
    "allowSyntheticDefaultImports": true,
    // 开启严格模式
    "strict": true,
    // 对文件名称强制区分大小写
    "forceConsistentCasingInFileNames": true,
    // 为 switch 语句启用错误报告
    "noFallthroughCasesInSwitch": true,
    // 生成代码的模块化标准
    "module": "esnext",
    // 模块解析（查找）策略
    "moduleResolution": "node",
    // 允许导入扩展名为.json的模块
    "resolveJsonModule": true,
    // 是否将没有 import/export 的文件视为旧（全局而非模块化）脚本文件
    "isolatedModules": true,
    // 编译时不生成任何文件（只进行类型检查）
    "noEmit": true,
    // 指定将 JSX 编译成什么形式
    "jsx": "react-jsx"
  },
  // 指定允许 ts 处理的目录
  "include": ["src"]
}
```

# typescript声明文件

今天几乎所有的 JavaScript 应用都会引入许多第三方库来完成任务需求。

这些第三方库不管是否是用 TS 编写的，最终都要编译成 JS 代码，才能发布给开发者使用。

我们知道是 TS 提供了类型，才有了代码提示和类型保护等机制。 

但在项目开发中使用第三方库时，你会发现它们几乎都有相应的 TS 类型，这些类型是怎么来的呢? `类型声明文件`

- **类型声明文件：用来为已存在的 JS 库提供类型信息**

这样在 TS 项目中使用这些库时，就像用 TS 一样，都会有代码提示、类型保护等机制了。

1. TS 的两种文件类型
2. 类型声明文件的使用说明

## TS 中的两种文件类型

- TS 中有两种文件类型：1 `.ts` 文件 2 `.d.ts` 文件
- .ts 文件:
  1. `既包含类型信息又可执行代码`
  2. 可以被编译为 .js 文件，然后，执行代码
  3. 用途：编写程序代码的地方
- .d.ts 文件:
  1. `只包含类型信息`的类型声明文件
  2. **不会生成 .js 文件，仅用于提供类型信息,在.d.ts文件中不允许出现可执行的代码，只用于提供类型**
  3. 用途：为 JS 提供类型信息

- 总结：.ts 是 `implementation`(代码实现文件)；**.d.ts 是 declaration(类型声明文件)**
- 如果要为 JS 库提供类型信息，要使用 `.d.ts` 文件

## 类型声明文件的使用说明

- 在使用 TS 开发项目时，类型声明文件的使用包括以下两种方式: 
  1. 使用已有的类型声明文件
  2. 创建自己的类型声明文件

使用已有的类型声明文件

1. 内置类型声明文件
2. 第三方库的类型声明文件
3. 自己提供的

## 内置类型声明文件

- TS 为 JS 运行时可用的所有标准化内置 API 都提供了声明文件
- 比如，在使用数组时，数组所有方法都会有相应的代码提示以及类型信息:

```ts
const strs = ['a', 'b', 'c']
// 鼠标放在 forEach 上查看类型
strs.forEach
```

- 实际上这都是 TS 提供的内置类型声明文件
- 可以通过 Ctrl + 鼠标左键(Mac：Command + 鼠标左键)来查看内置类型声明文件内容
- 比如，查看 forEach 方法的类型声明，在 VSCode 中会自动跳转到 `lib.es5.d.ts` 类型声明文件中
- 当然，像 window、document 等 BOM、DOM API 也都有相应的类型声明(`lib.dom.d.ts`)

## 第三方库的类型声明文件 

- 目前，几乎所有常用的第三方库都有相应的类型声明文件
- 第三方库的类型声明文件有两种存在形式:1 库自带类型声明文件 2 由 DefinitelyTyped 提供。 

1. 库自带类型声明文件：比如，axios

  - 查看 `node_modules/axios` 目录

解释：这种情况下，正常导入该库，**TS 就会自动加载库自己的类型声明文件**，以提供该库的类型声明。

2. 由 DefinitelyTyped 提供

- DefinitelyTyped 是一个 github 仓库，用来提供高质量 TypeScript 类型声明
- [DefinitelyTyped 链接](https://github.com/DefinitelyTyped/DefinitelyTyped/)
- 可以通过 npm/yarn 来下载该仓库提供的 TS 类型声明包，这些包的名称格式为:`@types/*`
- 比如，@types/react、@types/lodash 等
- 说明：在实际项目开发时，如果你使用的第三方库没有自带的声明文件，VSCode 会给出明确的提示

```ts
import _ from 'lodash'

// 在 VSCode 中，查看 'lodash' 前面的提示
```

- 解释：当安装 `@types/*` 类型声明包后，**TS 也会自动加载该类声明包**，以提供该库的类型声明
- 补充：TS 官方文档提供了一个页面，可以来查询 @types/* 库
- [@types/* 库](https://www.typescriptlang.org/dt)

## 创建自己的类型声明文件

1. 项目内共享类型
2. 为已有 JS 文件提供类型声明

### 项目内共享类型

- 如果多个 .ts 文件中都用到同一个类型，此时可以创建 .d.ts 文件提供该类型，实现类型共享。
- 操作步骤:
  1. 创建 index.d.ts 类型声明文件。
  2. 创建需要共享的类型，并使用 export 导出(TS 中的类型也可以使用 import/export 实现模块化功能)。
  3. 在需要使用共享类型的 .ts 文件中，通过 import 导入即可(.d.ts 后缀导入时，直接省略)。

### 为已有 JS 文件提供类型声明

1. 在将 JS 项目迁移到 TS 项目时，为了让已有的 .js 文件有类型声明。 
2. 成为库作者，创建库给其他人使用。

- 注意:类型声明文件的编写与模块化方式相关，不同的模块化方式有不同的写法。但由于历史原因，JS 模块化的发展 经历过多种变化(AMD、CommonJS、UMD、ESModule 等)，而 TS 支持各种模块化形式的类型声明。这就导致 ，类型声明文件相关内容又多又杂。
- 演示:基于最新的 ESModule(import/export)来为已有 .js 文件，创建类型声明文件。 

### 类型声明文件的使用说明

- 说明:TS 项目中也可以使用 .js 文件。
- 说明:在导入 .js 文件时，TS 会自动加载与 .js 同名的 .d.ts 文件，以提供类型声明。
- declare 关键字:用于类型声明，为其他地方(比如，.js 文件)已存在的变量声明类型，而不是创建一个新的变量。
  1. 对于 type、interface 等这些明确就是 TS 类型的(只能在 TS 中使用的)，可以省略 declare 关键字。
  2. 对于 let、function 等具有双重含义(在 JS、TS 中都能用)，应该使用 declare 关键字，明确指定此处用于类型声明。



```jsx
let count = 10
let songName = '痴心绝对'
let position = {
  x: 0,
  y: 0
}

function add(x, y) {
  return x + y
}

function changeDirection(direction) {
  console.log(direction)
}

const fomartPoint = point => {
  console.log('当前坐标：', point)
}

export { count, songName, position, add, changeDirection, fomartPoint }

```

定义类型声明文件

```jsx
declare let count:number

declare let songName: string

interface Position {
  x: number,
  y: number
}

declare let position: Position

declare function add (x :number, y: number) : number

type Direction = 'left' | 'right' | 'top' | 'bottom'

declare function changeDirection (direction: Direction): void

type FomartPoint = (point: Position) => void

declare const fomartPoint: FomartPoint

export {
  count, songName, position, add, changeDirection, FomartPoint, fomartPoint
}
```



# 在现有项目中添加 TS

- [CRA 添加 ts 文档](https://create-react-app.dev/docs/adding-typescript)
- 如果要在现有的 JS 项目中，添加 TS，需要以下操作：

1. 安装包：`yarn add typescript @types/node @types/react @types/react-dom @types/jest`
2. 把 `jsconfig.json`改成 path.tsconfig.json
4. 将原来通过 React 脚手架创建的 TS 项目中的 tsconfig.json 中的配置，拷贝到咱们自己的项目中
5. 创建 `path.tsconfig.json` 文件，将原来 `jsconfig.json` 文件中的内容拿过来

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"],
      "@scss/*": ["src/assets/styles/*"]
    }
  }
}

```

5. 在 `tsconfig.json` 中，添加以下配置：

```json
{
  // 添加这一句
  "extends": "./path.tsconfig.json",

  "compilerOptions": {
    ...
  }
}
```

6. 将通过 React 脚手架创建的 TS 项目中的 `src/react-app-env.d.ts` 拷贝到咱们自己项目的 src 目录下
7. 重启项目

## 说明

1. 项目中使用 TS 时，既可以包含 js 文件，又可以包含 ts 文件
  - `.js`、`.jsx`（使用 JS 时，React 组件对应的文件后缀）
  - `.ts`、`.tsx`（使用 TS 时，React 组件对应的文件后缀）、`.d.ts`
2. 在已有项目中，添加 TS 时的推荐模式
  - 新的功能用 TS
  - 已实现的功能，可以继续保持 JS 文件，慢慢修改为 TS 即可
3. React 组件对应的文件后缀，修改为：`.tsx`
4. 工具函数对应的文件后缀，修改为：`.ts` 或者为其添加类型声明文件 `.d.ts`

