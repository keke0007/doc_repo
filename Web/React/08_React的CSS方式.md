React中如何编写cSS?

## 目录

React中csS的概述 content 内联样式CSS写法 普通CSS文件写法 CSSModule写法 CSSinJS解决方案

```javascript
classnames库使用
```

## 组件化天下的CSS

前面说过，整个前端已经是组件化的天下： 在组件化中选择合适的CSS解决方案应该符合以下条件：

- 可以编写局部cSS：CSs具备自己的具备作用域，不会随意污染其他组件内的元素；
- 可以编写动态的css：可以获取当前组件的一些状态，根据状态的变化生成不同的css样式;
- 支持所有的css特性：伪类、动画、媒体查询等；
- 编写起来简洁方便、最好符合一贯的css风格特点
- 等等.

## React中的css

事实上，css一直是React的痛点，也是被很多开发者吐槽、垢病的一个点。 1在这一点上，Vue做的要好于React：

- Vue通过在.vue文件中编写<style><style>标签来编写自己的样式;
- 通过是否添加scoped属性来决定编写的样式是全局有效还是局部有效；
- 通过lang属性来设置你喜欢的less、sass等预处理器；
- 通过内联样式风格的方式来根据最新状态设置和改变cSS；
- 等等..
1Vue在CSS上虽然不能称之为完美，但是已经足够简洁、自然、方便了，至少统一的样式风格不会出现多个开发人员、多个项目 采用不一样的样式风格。 相比而言，React官方并没有给出在React中统一的样式风格：

- 由此，从普通的css，到cssmodules，再到cssinjs，有几十种不同的解决方案，上百个不同的库；
- 大家一致在寻找最好的或者说最适合自己的CSS方案，但是到目前为止也没有统一的方案；

## 内联样式

内联样式是官方推荐的一种css样式的写法：

- style接受一个采用小驼峰命名属性的JavaScript对象，，而不是CSS字符串;
- 并且可以引用state中的状态来设置相关的样式；
内联样式的优点：

- 1.内联样式，样式之间不会有冲突
- 2.可以动态获取当前state中的状态
内联样式的缺点：

- 1.写法上都需要使用驼峰标识
- 2.某些样式没有提示
- 3.大量的样式，代码混乱
- 4.某些样式无法编写(比如伪类/伪元素)
所以官方依然是希望内联合适和普通的css来结合编写；

## 普通的CSS

普通的css我们通常会编写到一个单独的文件，之后再进行引I入。 这样的编写方式和普通的网页开发中编写方式是一致的：

- 如果我们按照普通的网页标准去编写，那么也不会有太大的问题；
- 但是组件化开发中我们总是希望组件是一个独立的模块，即便是样式也只是在自己内部生效，不会相互影响;
- 但是普通的css都属于全局的csS，样式之间会相互影响；
■这种编写方式最大的问题是样式之间会相互层叠掉；

## cssmodules

cssmodules并不是React特有的解决方案，而是所有使用了类似于webpack配置的环境下都可以使用的。

- 如果在其他项目中使用它，那么我们需要自己来进行配置，比如配置webpack.configjs中的modules:true等。
React的脚手架已经内置了cssmodules的配置：

- .css/.less/.scss等样式文件都需要修改成.module.css/.module.less/.module.scss等；
- 之后就可以引用并且进行使用了；
cssmodules确实解决了局部作用域的问题，也是很多人喜欢在React中使用的一种方案。 但是这种方案也有自己的缺陷：

- 引l用的类名，不能使用连接符(.home-title)，在JavaScript中是不识别的;
- 所有的className都必须使用{style.className}的形式来编写；
- 不方便动态来修改某些样式，依然需要使用内联样式的方式

## 认识CSSinJS

官方文档也有提到过cSSinJS这种方案：

- "CSS-in-JS"是指一种模式，其中CSS由JavaScript生成而不是在外部文件中定义;
- 注意此功能并不是React的一部分，而是由第三方库提供；
- React对样式如何定义并没有明确态度；
```javascript
在传统的前端开发中，我们通常会将结构(HTML)、样式(CSS)、逻辑(JavaScript)进行分离。
```

- 但是在前面的学习中，我们就提到过，React的思想中认为逻辑本身和UI是无法分离的，所以才会有了JSX的语法。
- 样式呢？样式也是属于UI的一部分
- 事实上CSS-in-JS的模式就是一种将样式(CSS)也写入到JavaScript中的方式，并且可以方便的使用JavaScript的状态；
- 所以React有被人称之为AllinJS；
当然，这种开发的方式也受到了很多的批评：

```javascript
Stop using CSS in JavaScript for web development
```

https://hackernoon.com/stop-using-css-in-javascript-for-web-development-fa32fb873dcc

## 认识styled-components

批评声音虽然有，但是在我们看来很多优秀的CSS-in-JS的库依然非常强大、方便：

- CSS-in-JS通过JavaScript来为CSS赋予一些能力，包括类似于CSS预处理器一样的样式嵌套、函数定义、逻辑复用、动态修
```javascript
改状态等等;
```

- 虽然CSS预处理器也具备某些能力，但是获取动态状态依然是一个不好处理的点；
- 所以，目前可以说CSS-in-JS是React编写CSS最为受欢迎的一种解决方案；
目前比较流行的CSS-in-JS的库有哪些呢？ styled-components emotion glamorous 目前可以说styled-components依然是社区最流行的cSS-in-JS库，所以我们以styled-components的讲解为主； I安装styled-components: %yarnadd styled-components

## ES6标签模板字符串

ES6中增加了模板字符串的语法，这个对于很多人来说都会使用。 但是模板字符串还有另外一种用法：标签模板字符串(TaggedTemplate Literals) .

```javascript
function foo(...args) {
console.log(args) ;
```

我们一起来看一个普通的JavaScript的函数：

- 正常情况下，我们都是通过函数名(方式来进行调用的，其实函数还有另外一种 foo("Hello World") ;
调用方式：

```javascript
foo^Hello World;
```

如果我们在调用的时候插入其他的变量：

- 模板字符串被拆分了; const name = "Kobe";
```javascript
fooHello ${name};
```

- 第一个元素是数组，是被模块字符串拆分的字符串组合；
- 后面的元素是一个个模块字符串传入的内容；
1在styledcomponent中，就是通过这种方式来解析模块字符串，最终生成我们想要 的样式的

## styled的基本使用

```javascript
Istyled-components的本质是通过函数的调用，最终 const HomeWrapper = styled.div
color: Purple;
```

创建出一个组件：

- 这个组件会被自动添加上一个不重复的class; export defaolt class Home extends PureComponent
```javascript
render()
return
```

- styled-components会给该class添加相关的样式; <HomeWrapper>
```javascript
<h2>我是Home标题</h2>
<ul> 我是Home标题
<i>我是列表1</i> ·我是列表1
<li>我是列表2</i> 我是列表2
```

我是列表3

```javascript
1另外，它支持类似于CSS预处理器一样的样式嵌套： <1i>我是列表3</i>
</ul> <divclass="sc-AxjAm dCkvIn"
```

- 支持直接子代选择器或后代选择器，并且直接编写 </HomeWrapper> <h2>我是Home标题</h2>==$0
```javascript
<ul></ul>
</div>
样式;
```

- 可以通过&符号获取当前元素
- 直接伪类选择器、伪元素等； 我是Home标题
我是列表1abc 我是列表2abc

- 我是列表3abc

## props、attrs属性

props可以传递

```javascript
<HYInput type="password" left="2opx" />
```

props可以被传递给styled组件

- 获取props需要通过$0传入一个插值函数，props会作为该函数的参数;
- 这种方式可以有效的解决动态样式的问题;
添加attrs属性

```javascript
const HYInput = styled.input.attrs({
```

placeholder：。"请填写密码"，

```javascript
paddingLeft: props => props.left Il "5px"
border-color: red;
padding-left: ${props => props.paddingLeft};
&:focus {
outline-color: orange;
```

## styled高级特性

支持样式的继承

```javascript
const HYButton = styled.button const HYWarnButton = styled(HYButton)
padding: 8px 30px; background-color: red;
border-radius: 5px; color: #fff;
```

styled设置主题

```javascript
import { ThemeProvider } from 'styled-components'
const ProfileWrapper = styled.div
<Home /> color: ${props => props.theme.color};
<Profile /> font-size: ${props => props.theme.fontSize};
</ThemeProvider>
```

## vue中添加class

Ivue中添加class是一件非常简单的事情： 你可以通过传入一个对象：

```javascript
<div class="static"
v-bind:class="{ active: isActive,·'text-danger':·hasError }">
</div>
```

你也可以传入一个数组：

```javascript
<div v-bind:class="[activeclass, errorclass]"></div>
```

甚至是对象和数组混合使用：

```javascript
<div v-bind:class="[{ active: isActive }, errorclass]"></div>
```

## React中添加oclass

React在JSX给了我们开发者足够多的灵活性，你可以像编写JavaScript代码一样，通过一些逻辑来决定是否添加某些class：

```javascript
<div>
<h2className={"title。。+。(isActive°？。"active"：。"")}>我是标题</h2>
<h2 className={["title"，。(isActive？。"active"：。"")］.join("。")}>我是标题</h2>
</div>
```

1这个时候我们可以借助于一个第三方的库：classnames

- 很明显，这是一个用于动态添加classnames的一个库。
```javascript
classNames('foo',·'bar'); //·=>.'foo·bar!
classNames('foo',·{ bar: true }); //·=>.'foo·bar
classNames({·'foo-bar': true }); //·=>.'foo-bar!
classNames({ 'foo-bar': false }); //·=>.11
classNames({ foo: true }, { bar: true }); //·=>.'foo·bar
classNames({ foo: true,·bar: true }); //·=>.Ifoo·bar!
classNames('foo',·{ bar: true, duck: false },·'baz',· quux: true });·//·=>.'foo·bar baz quux"
classNames(null, false,·'bar',·undefined,·0,·1,·{ baz: null·},.''); //·=>.'bar·I'
```
