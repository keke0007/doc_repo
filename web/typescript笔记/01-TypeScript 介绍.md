# TypeScript

- TypeScript 介绍
- TypeScript 初体验
- TypeScript 常用类型
- TypeScript 高级类型
- TypeScript 类型声明文件
- 在 React 中使用 TypeScript

## TypeScript 介绍

- [TS 官方文档](https://www.typescriptlang.org/)
- [TS 中文参考 - 不再维护](https://www.tslang.cn/)

![TS Logo](./images/TS_logo.png)

1. TypeScript 是什么
2. TypeScript 为什么要为 JS 添加类型支持?
3. TypeScript 相比 JS 的优势

### TypeScript 是什么

- **`TypeScript` 简称：TS，是 JavaScript 的超集**，简单来说就是：JS 有的 TS 都有

![TS是JS的超集](./images/TS是JS的超集.png)

- TypeScript = `Type` + JavaScript（在 JS 基础之上，**为 JS 添加了类型支持**）
- TypeScript 是*微软*开发的开源编程语言，可以在任何运行 JavaScript 的地方运行

![TS 和 JS 的对比](./images/TSvsJS.png)

### TypeScript 为什么要为 JS 添加类型支持?

- 背景：JS 的类型系统存在“先天缺陷”，JS 代码中绝大部分错误都是类型错误（Uncaught TypeError） 
- 这些经常出现的错误，导致了在使用 JS 进行项目开发时，增加了找 Bug、改 Bug 的时间，严重影响开发效率

为什么会这样？

- 从编程语言的动静来区分，**TypeScript 属于静态类型的编程语言**，**JavaScript 属于动态类型的编程语言**
  - 静态类型：**编译**期做类型检查
  - 动态类型：**执行**期做类型检查
- 代码编译和代码执行的顺序：1 编译 2 执行

- 对于 JS 来说：需要等到代码真正去执行的时候才能发现错误（晚）
- 对于 TS 来说：在代码编译的时候（代码执行前）就可以发现错误（早）

并且，配合 VSCode 等开发工具，TS 可以提前到**在编写代码的同时就发现代码中的错误**，减少找 Bug、改 Bug 时间  

对比：

- 使用 JS：
  1. 在 VSCode 里面写代码
  2. 在浏览器中运行代码 --> 运行时，才会发现错误【晚】
- 使用 TS：
  1. 在 VSCode 里面写代码 --> 写代码的同时，就会发现错误【早】
  2. 在浏览器中运行代码

### TypeScript 相比 JS 的优势

1. **更早（写代码的同时）发现错误**，减少找 Bug、改 Bug 时间，提升开发效率
2. 程序中任何位置的代码都有**代码提示**，随时随地的安全感，增强了开发体验
3. 强大的类型系统提升了代码的可维护性，使得**重构代码更加容易**
4. 支持最新的 ECMAScript 语法，**优先体验最新的语法，让你走在前端技术的最前沿** 
5. TS 类型推断机制，不需要在代码中的每个地方都显示标注类型，让你在享受优势的同时，尽量降低了学习负担

除此之外，Vue 3 源码使用 TS 重写、Angular 默认支持 TS、React 与 TS 完美配合，TypeScript 已成为大中型前端 项目的首选编程语言

目前，前端最新的开发技术栈：

1. React： TS + Hooks
2. Vue： TS + Vue3

  - 注意： Vue2 对 TS 的支持不好~ 

---

## TypeScript 初体验

1. 安装编译 TS 的工具包
2. 编译并运行 TS 代码
3. 简化运行 TS 的步骤

### 安装编译 TS 的工具包

- 问题：为什么要安装编译 TS 的工具包?
- 回答：Node.js/浏览器，只认识 JS 代码，不认识 TS 代码。需要先将 TS 代码转化为 JS 代码，然后才能运行
- 安装命令：`npm i -g typescript`
  - typescript 包：用来编译 TS 代码的包，提供了 `tsc` 命令，实现了 TS -> JS 的转化
  - *注意：Mac 电脑安装全局包时，需要添加 `sudo` 获取权限：`sudo npm i -g typescript`*  yarn 全局安装：`sudo yarn global add typescript`
- 验证是否安装成功：tsc –v(查看 typescript 的版本)

![TS 编译](./images/TS编译.png)

###  编译并运行 TS 代码

1. 创建 hello.ts 文件（注意：**TS 文件的后缀名为 `.ts`**）
2. 将 TS 编译为 JS：在终端中输入命令，`tsc hello.ts`（此时，在同级目录中会出现一个同名的 JS 文件）
3. 执行 JS 代码：在终端中输入命令，`node hello.js`

1 创建 ts 文件  ===>  2 编译 TS  ===>  3 执行 JS  

- 说明：所有合法的 JS 代码都是 TS 代码，有 JS 基础只需要学习 TS 的类型即可
- 注意：由 TS 编译生成的 JS 文件，代码中就没有类型信息了

### 简化运行 TS 的步骤

- 问题描述：每次修改代码后，都要重复执行两个命令，才能运行 TS 代码，太繁琐
- 简化方式：使用 `ts-node` 包，直接在 Node.js 中执行 TS 代码
- 安装命令：`npm i -g ts-node`
  - ts-node 包提供了 `ts-node` 命令
  - 使用方式：`ts-node hello.ts` 相当于：1 tsc 命令  2 node（注意：ts-node 不会生成 js 文件）

解释：ts-node 命令在内部偷偷的将 TS -> JS，然后，再运行 JS 代码

