TypeScript知识扩展

目录 TypeScript模块使用 content TypeScript命名空间 内置声明文件的使用 第三方库声明的文件 编写自定义声明文件 tsconfig配置文件解析

## TypeScript模块化

■JavaScript有一个很长的处理模块化代码的历史，TypeScript从2012年开始跟进，现在已经实现支持了很多格式。但是随着 时间流逝，社区和JavaScript规范已经使用为名为ESModule的格式，这也就是我们所知的import/export语法。

- ES模块在2015年被添加到JavaScript规范中，到2020年，大部分的web浏览器和JavaScript运行环境都已经广泛支持。
- 所以在TypeScript中最主要使用的模块化方案就是ESModule；
```javascript
ck_dev > src >utils >TS math.ts > ..
export function add(numl: number, num2:·number) {
return numl + num2
export function sub(num1:·number,·num2: number) {
return num1.-·num2
```

1在前面我们已经学习过各种各样模块化方案以及对应的细节，这里我们主要学习TypeScript中一些比较特别的细节。

## 非模块(Non-modules)

我们需要先理解TypeScript认为什么是一个模块。

- JavaScript规范声明任何没有export的JavaScript文件都应该被认为是一个脚本，而非一个模块。
- 在一个脚本文件中，变量和类型会被声明在共享的全局作用域，将多个输入文件合并成一个输出文件，或者在HTML使用多
```javascript
个<script>标签加载这些文件。
```

1如果你有一个文件，现在没有任何import或者export，但是你希望它被作为模块处理，添加这行代码：

```javascript
export}
```

■这会把文件改成一个没有导出任何内容的模块，这个语法可以生效，无论你的模块目标是什么。

## 内置类型导入(Inline typeimports)

1TypeScript4.5也允许单独的导入，你需要使用type前缀，表明被导入的是一个类型：

```javascript
import· type IFoo, type IDType } from·"./foo"
const id:·IDType = 100
const foo:·IFoo ={
```

name: "why", age:°18 1这些可以让一个非TypeScript编译器比如Babel、swc或者esbuild知道什么样的导入可以被安全移除。 Together these allow a non-TypeScript transpiler like Babel, swc or esbuild toknow what imports can besafely removed.

## 命名空间namespace (了解)

### 1TypeScript有它自己的模块格式，名为namespaces，它在ES 模块标准之前出现。

- 命名空间在TypeScript早期时，称之为内部模块，目的是将一个模块内部再进行作用域的划分，防止一些命名冲突的问题;
- 虽然命名空间没有被废弃，但是由于ES模块已经拥有了命名空间的大部分特性，因此更推荐使用ES模块，这样才能与
```javascript
JavaScript的(发展)方向保持一致。
```

export·namespace·Time

```javascript
export·namespace·Price{
export function format(time: string)·{
return "2022-10-10" export function format(price: string)·{
return."￥20.00"
export const name = "time"
```

usefulfeaturesforcreatingcomplexdefinitionfiles,andstillsees activeuseinDefinitelyIyped.Whilenotdeprecated,the majorityof thefeaturesinnamespacesexistinESModulesandwerecommendyouusethattoalignwithJavaScript'sdirection.

## 类型的查找

之前我们所有的typescript中的类型，几乎都是我们自己编写的，但是我们也有用到一些其他的类型： 1大家是否会奇怪，我们的HTMLlmageElement类型来自哪里呢？甚至是document为什么可以有getElementByld的方法呢？

- 其实这里就涉及到typescript对类型的管理和查找规则了。
我们这里先给大家介绍另外的一种typescript文件：.d.ts文件

- 我们之前编写的typescript文件都是.ts文件，这些文件最终会输出js文件，也是我们通常编写代码的地方；
- 还有另外一种文件.d.ts 文件，它是用来做类型的声明(declare)，称之为类型声明(Type Declaration)或者类型定义(Type
Definition)文件。

- 它仅仅用来做类型检测，告知typescript我们有哪些类型；
那么typescript会在哪里查找我们的类型声明呢？

- 内置类型声明;
- 外部定义类型声明;
- 自己定义类型声明；

## 内置类型声明

内置类型声明是typescript自带的、帮助我们内置了JavaScript运行时的一些标准化APl的声明文件；

- 包括比如Function、String、Math、Date等内置类型;
- 也包括运行环境中的DOMAPl，比如Window、Document等；
TypeScript使用模式命名这些声明文件lib.[something].d.ts。 lib.dom.d.ts

```javascript
Applications>Visual Studio Code.app>Contents>Resources>app>extensions>node_modules>typescript lib>TSlib.dom.d.ts
```

4315 interface Document extends·Node,·DocumentAndElementEventHandlers,·DocumentorShadowRoot,

```javascript
FontFaceSource, GlobalEventHandlers,·NonElementParentNode, ParentNode, XPathEvaluatorBase {
4316 /**·Sets or gets the·URL·for the·current·document.·*
4317 readonly URL: string;
```

4318

```javascript
4319 *·Sets or gets the·color of·all·active·links in·the document.
```

4320 *·@deprecated 4321

```javascript
4322 alinkColor: string;
```

内置类型声明通常在我们安装typescript的环境中会带有的； https://github.com/microsoft/TypeScript/tree/main/lib

## 内置声明的环境

## 我们可以通过target和lib来决定哪些内置类型声明是可以使用的：

- 例如，startsWith字符串方法只能从称为ECMAScript6的JavaScript版本开始使用；
我们可以通过target的编译选项来配置：TypeScript通过lib根据您的target设置更改默认包含的文件来帮助解决此问题。

## https://www.typescriptlang.org/tsconfig#lib

Name Contents

```javascript
ES5 Core definitions for all ES3 and ES5 functionality
ES2015 Aditional APls available in ES2015 (also known as ES6) - array · find, Promise,
```

Proxy, Symbol, Map, Set, Reflect, etc.

```javascript
ES6 Alias for "ES2015"
```

ES2016 Additional APls available in ES2016 - array . include, etc.

```javascript
ES7 Alias for "ES2016*
ES2017 Additional APls available in ES2017 - Object.entries, Object.values, Atomics,
```

SharedArrayBuffer, date.formatToParts, typed arrays, etc. ES2018 Additional APls available in ES2018 - async iterables, promise . fina1ly. Intl.PluralRules, regexp.groups, etc.

```javascript
ES2019 Additional APls available in ES2019 - array.flat, array.flatMap,
Object.fromEntries, string.trimStart, string.trimEnd, etc.
```

ES2020 Additional APls available in ES2020 - string ·matchA1l, etc. ES2021 Additional APls available in ES2021 - promise any, string .replaceA11 etc. ESNext Additional APls available in ESNext - This changes as the JavaScript specification evolves

## 外部定义类型声明一第三方库

```javascript
1外部类型声明通常是我们使用一些库(比如第三方库)时，需要的一些类型声明。
```

1这些库通常有两种类型声明方式：

```javascript
方式一：在自己库中进行类型声明(编写.d.ts文件)，比如axios
```

方式二：通过社区的一个公有库DefinitelyTyped存放类型声明文件 该库的GitHub地址：https://github.com/DefinitelyTyped/DefinitelyTyped/

- 该库查找声明安装方式的地址：https://www.typescriptlang.org/dt/search?search=
- 比如我们安装react的类型声明：npmi@types/react--save-dev
TypeScript automatically finds type definitions under node_modules/@types,so there's no other step needed to get these types availableinyourprogram. import React from."react" import axios from."axios"

## 外部定义类型声明一自定义声明

1什么情况下需要自己来定义声明文件呢？

- 情况一：我们使用的第三方库是一个纯的JavaScript库，没有对应的声明文件；比如lodash
- 情况二：我们给自己的代码中声明一些类型，方便在其他地方直接进行使用;
```javascript
let wName = "coderwhy"
let wAge = 18 declare let wName: string;
let wHeight = 1.88
declare let wAge: ·number;
declare let wHeight: number
function wFoo()
console.log("wFoo")
declare function wFoo():·void
declare function·wBar():·void
function wBar() {
declare class Person {
console.log("wBar")
```

name: string age: number

```javascript
function Person(name, age) {
this.name = name constructor(name: string, age:·number)
this.age = age
```

## declare声明模块

我们也可以声明模块，比如lodash模块默认不能使用的情况，可以自己来声明这个模块：

```javascript
declare module."lodash"{
export function join(args: any[]): any;
```

声明模块的语法：declaremodule'模块名'。

- 在声明模块的内部，我们可以通过export导出对应库的类、函数等；

## declare声明文件

1在某些情况下，我们也可以声明文件：

- 比如在开发vue的过程中，默认是不识别我们的.vue文件的，那么我们就需要对其进行文件的声明
- 比如在开发中我们使用了jpg这类图片文件，默认typescript也是不支持的，也需要对其进行声明;
```javascript
declare module '*.vue' {
import·{·DefineComponent·}·from.'vue'
const·component:·DefineComponent
```

export·default·component

```javascript
declare module '*.jpg' {
const src: string
```

export default src

## declare命名空间

比如我们在index.html中直接引l入了jQuery： CDNt地t址 : https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.js 我们可以进行命名空间的声明：

```javascript
declare namespace $ {
function ajax(settings: any) : void
```

在main.ts中就可以使用了：

```javascript
$.ajax({
```

url::"http://123.207.32.32:8000/home/multidata"

```javascript
success:·(res: any) => {
console.log(res) ;
```

## 认识tsconfig.json文件

```javascript
什么是tsconfigjson文件呢？(官方的解释)
```

- 当目录中出现了tsconfigjson文件，则说明该目录是TypeScript项目的根目录;
- tsconfigjson文件指定了编译项目所需的根目录下的文件以及编译选项。
！官方的解释有点"官方"，直接看我的解释。 tsconfig.json文件有两个作用： √比如是否允许不明确的this选项，是否允许隐式的any类型； √将TypeScript代码编译成什么版本的JavaScript代码；

```javascript
作用二：让编辑器(比如VSCode)可以按照正确的方式识别TypeScript代码；
```

√对于哪些语法进行提示、类型错误检测等等； JavaScript项目可以使用jsconfig.json文件，它的作用与tsconfig.json基本相同，只是默认启用了一些JavaScript相关的 编译选项。

- 在之前的Vue项目、React项目中我们也有使用过；

## tsconfig.json配置

tsconfig.json在编译时如何被使用呢？

- 在调用tsc命令并且没有其它输入文件参数时，编译器将由当前目录开始向父级目录寻找包含tsconfig文件的目录。
- 调用 tsc 命令并且没有其他输入文件参数，可以使用--project(或者只是-p)的命令行选项来指定包含了tsconfigjson 的
```javascript
目录;
```

- 当命令行中指定了输入文件参数，tsconfig.json文件会被忽略；
1webpack中使用ts-loader进行打包时，也会自动读取tsconfig文件，根据配置编译TypeScript代码。 1tsconfig.json文件包括哪些选项呢？

- tsconfig.json本身包括的选项非常非常多，我们不需要每一个都记住；
- 当我们开发项目的时候，选择TypeScript模板时，tsconfig文件默认都会帮助我们配置好的;
接下来我们学习一下哪些重要的、常见的选项。

## tsconfig.json顶层选项

[***s...'.***s..pu... compilerOptions 后续详细讲解 Whichwould include: 编写一个数组，用于指定项目中包括哪些文件 files 通常当项目中文件比较少时，可以使用这个选项 scripts lint.ts 编写一个数组，用于指定在项目中包括哪些文件 update_deps.ts include 默认匹配的是根目录下所有的文件 utils.ts src 手动指定 "include":·["src/**/*",."types/**/*.d.ts"] client index.ts utils.ts exclude 编写一个数组，用于指定从include中排除哪些 server 文件 index.ts tests app.test.ts 其他配置选项 等Vue、React项目中见到再解析 utils.ts tests.d.ts package.json tsconfig.json yarn.lock

## tsconfig.json文件

Itsconfig.json是用于配置TypeScript编译时的配置选项： https://www.typescriptlang.org/tsconfig 我们这里讲解几个比较常见的： "compileroptions":

- /·目标代码
"target":·"esnext",

- /·生成代码使用的模块化
"module": "esnext",

- /·打开所有的严格模式检查
"strict": true, "allowJs": false, "noImplicitAny": false,

- /·jsx的处理方式(保留原有的jsx格式)
"jsx":·"preserve",

- /·是否帮助导入一些需要的功能模块
" mportHelpers": true,

- ·按照node的模块解析规则
- /·https://www.typescriptlang.org/docs/handbook/module-resolution.html#module-resolution-strategies
"moduleResolution":."node",

- /·跳过对整个库进行类似检测，·而仅仅检测你用到的类型
"skipLibcheck": true,

## tsconfig.json文件

可以让es·module·和·commonjs相互调用 "esModuleInterop": true, 允许合成默认模块导出 import·*·as·react·from·'react':·false import·react·from·'react':·true "allowSyntheticDefaultImports": true,

- /·是否要生成sourcemap文件
"sourceMap": true,

- /·文件路径在解析时的基本url
"baseUrl":·".""

```javascript
指定types文件需要加载哪些(默认是都会进行加载的)
```

"types":·[ "webpack-env"

- /·路径的映射设置，类似于webpack中的·alias
```javascript
"paths": {
```

"@/*":·["src/*"]

- /·指定我们需要使用到的库(也可以不配置，直接根据target来获取)
"Lib":· ["esnext", "dom", "dom.iterable", "scripthost"] "include":·[ "exclude":·["node_modules"]
