深入JavaScript的运行原理

## 目录

深入V8引l擎原理 content JS执行上下文 全局代码执行过程 函数代码执行过程 作用域和作用域链

## JavaScript代码的执行

1JavaScript代码下载好之后，是如何一步步被执行的呢？ 我们知道，浏览器内核是由两部分组成的，以webkit为例：

```javascript
WebCore：负责HTML解析、布局、渲染等等相关的工作;
```

- JavaScriptCore：解析、执行JavaScript代码；
WebKit WebCore JavaScriptCore

```javascript
(JSCore)
```

1另外一个强大的JavaScript引l擎就是V8引l擎。

## V8引擎的执行原理

我们来看一下官方对V8引擎的定义：

- V8是用C++编写的Google开源高性能JavaScript和WebAssembly引l擎，它用于Chrome和Node.js等。
器的Linux系统上运行。

- V8可以独立运行，也可以嵌入到任何C++应用程序中。
JavaScript AST bytecode 源代码 Parse 抽象语法树 Ignition 字节码 运行结果 收集信息，比如类型信息 Deoptimization MachineCode TurboFan 运行结果 优化的机器码

## V8引擎的架构

V8引|擎本身的源码非常复杂，大概有超过100w行C++代码，通过了解它的架构，我们可以知道它是如何对JavaScript执行的：

```javascript
Parse模块会将JavaScript代码转换成AST(抽象语法树)，这是因为解释器并不直接认识JavaScript代码；
```

- 如果函数没有被调用，那么是不会被转换成AST的；
- Parse的v8官方文档：https://v8.dev/blog/scanner
```javascript
lgnition是一个解释器，会将AST转换成ByteCode(字节码)
```

- 同时会收集TurboFan优化所需要的信息(比如函数参数的类型信息，有了类型才能进行真实的运算)；
- 如果函数只调用一次，lIgnition会解释执行ByteCode;
- lgnition的V8官方文档：https://v8.dev/blog/ignition-interpreter
TurboFan是一个编译器，可以将字节码编译为CPU可以直接执行的机器码；

- 如果一个函数被多次调用，那么就会被标记为热点函数，那么就会经过TurboFan转换成优化的机器码，提高代码的执行性能
- 但是，机器码实际上也会被还原为ByteCode，这是因为如果后续执行函数的过程中，类型发生了变化(比如sum函数原来执
行的是number类型，后来执行变成了string类型)，之前优化的机器码并不能正确的处理运算，就会逆向的转换成字节码；

- TurboFan的V8官方文档：https://v8.dev/blog/turbofan-jit

## V8引擎的解析图(官方)

词法分析(英文lexical analysis) PreParser 口将字符序列转换成token序列 的过程。 tokens

- token是记号化
```javascript
(tokenization)的缩写
```

Blink Stream Scanner Parser 口词法分析器(lexical ASCII UTF-16 tokens AST Latin1 code analyzer，简称lexer)，也 UTF-8 units

```javascript
chunks bytecode 叫扫描器(scanner)
```

* 语法分析(英语：Syntactic analysis, 也叫 parsing)

- 语法分析器也可以称之为
parser.

## V8引擎的解析图

TOKENS NETWORK CACHE WORKER punctuator PRE-PARSER BYTESTREAM keyword

```javascript
6e00750066 return
```

DECODER PARSER keyword

```javascript
function
```

ABSTRACTSYNTAXTREE TYPE NODE COMPILER INTERPRETER Program FEEDBACK NODE

```javascript
FunctionLiteral FunctionLiteral NODE
```

MACHINE CODE BYTE CODE

```javascript
f D4, E171 ReturnStatement BlockStatenent
```

## JavaScript代码执行原理－版本说明

- 目前网上大多数流行的说法都是基于ECMAScript3版本的解析，并且在面试时问到的大多数都是ECMAScript3的版本内容。
- 但是ECMAScript3终将过去，ECMAScript5必然会成为主流，所以最好也理解ECMAScript5甚至包括ECMAScript6以及更
好版本的内容；

- 事实上在TC39(ECMAScript5)的最新描述中，和ECMAScript5之后的版本又出现了一定的差异；
那么我们课程按照如下顺序学习：

- 通过ECMAScript3中的概念学习JavaScript执行原理、作用域、作用域链、闭包等概念；
- 通过ECMAScript5中的概念学习块级作用域、let、const等概念；
■事实上，它们只是在对某些概念上的描述不太一样，在整体思路上都是一致的。

## JavaScript的执行过程

假如我们有下面一段代码，它在JavaScript中是如何被执行的呢？

```javascript
var name = "why"
function foo(
varname=Ifoo
console.log(name)
var num1 =20
var num2=30
var result= numl +num2
console.log(result)
foo()
```

## 初始化全局对象

```javascript
js引l擎会在执行代码之前，会在堆内存中创建一个全局对象：Global Object(GO)
```

- 该对象所有的作用域(scope)都可以访问;
```javascript
里面会包含Date、Array、String、Number、setTimeout、setlnterval等等;
```

- 其中还有一个window属性指向自己；
堆内存 GlobalObject:0x100 Global Object

```javascript
There is a unique global object (15.1),which is created before control enters any execution context. +Date
```

Initially theglobal object has thefollowingproperties: +Array +String

```javascript
Built-in objects such as Math,String,Date,parseInt, etc.These have attributes {DontEnum }. +Number
```

+setTimeount Additional host defined properties.This may include a property whose value is the global object +window

```javascript
itself;forexample,in theHTML document objectmodel thewindowproperty of theglobal object is
```

the global object itself.

## 执行上下文(ExecutionContexts)

```javascript
js引擎内部有一个执行上下文栈(ExecutionContextStack，简称ECS)，它是用于执行代码的调用栈。
```

那么现在它要执行谁呢？执行的是全局的代码块：

- 全局的代码块为了执行会构建一个GlobalExecutionContext(GEC)；
- GEC会被放入到ECS中执行;
GEC被放入到ECS中里面包含两部分内容：

- 第一部分：在代码执行前，在parser转成AST的过程中，会将全局定义的变量、函数等加入到GlobalObject中，但是并不会
赋值；

```javascript
√这个过程也称之为变量的作用域提升(hoisting)
```

- 第二部分：在代码执行中，对变量赋值，或者执行其他的函数；
Execution Contexts When control is transferred to ECMAScript executable code, control is entering an execution context. Active

```javascript
execution contexts logically form a stack. The top execution context on this logical stack is the running
```

execution context.

## 认识vo对象(VariableObject)

```javascript
■每一个执行上下文会关联一个VO(VariableObject，变量对象)，变量和函数声明会被添加到这个VO对象中。
Every execution context has associated with it a variable object. Variables and functions declared in the
source text are added as properties of the variable object. For function code, parameters are added as
properties of the variable object.
```

当全局代码被执行的时候，VO就是GO对象了 Global Code The scope chain is created and initialised to contain the global object and no others.

```javascript
Variable instantiation is performed using the global object as the variable object and using property
attributes {DontDelete }.
The this value is the global object.
```

## 全局代码执行过程(执行前)

ECS执行上下文栈 堆内存

## GlobalObject:0x100 FunctionObject:0xa00

```javascript
+Date functionfoo0{
+Array varname=foo'
+String console.log(name)
```

+Number +setTimeount +window 自己定义的变量 name:undefined

```javascript
foo:0xa00(指向函数对象)
```

num1:undefined num2:undefined result:undefined GEC全局执行上下文 执行前：

```javascript
VO(0x100):GEC中VO就是GO对象
```

开始执行： 从上往下一次执行代码

## 全局代码执行过程(执行后)

ECS执行上下文栈 堆内存

```javascript
var name = why" GlobalObject:0x100 FunctionObject:0xa00
function foo()[
+Date functionfoo({
var name =!foo' +Array varname=foo'
console.log(name) +String console.log(name)
```

+Number +setTimeount +window

```javascript
var numl =20 自己定义的变量
varnum2=30 name:"why"
foo:0xa00(指向函数对象)
var result=numl+num2 num1:20
```

num2:30 result:50

```javascript
foo() GEC全局执行上下文
```

代码一次执行改变GO 执行前：

```javascript
VO(0x100):GEC中VO就是GO对象
```

开始执行： 从上往下一次执行代码

## 函数如何被执行呢？

```javascript
■在执行的过程中执行到一个函数时，就会根据函数体创建一个函数执行上下文(FunctionalExecutionContext，简称FEC)，
```

并且压入到ECStack中。 因为每个执行上下文都会关联一个VO，那么函数执行上下文关联的VO是什么呢？

- 当进入一个函数执行上下文时，会创建一个AO对象(Activation Object)；
- 这个AO对象会使用arguments作为初始化，并且初始值是传入的参数；
- 这个AO对象会作为执行上下文的VO来存放变量的初始化；
```javascript
When control enters an execution context for function code,an object called the activation object is
```

created and associated with the execution context. The activation object is initialised with a property

```javascript
with name arguments and attributes { DontDelete }. The initial value of this property is the arguments
```

object described below.

```javascript
The activation object is then used as the variable object for the purposes of variable instantiation.
```

## 函数的执行过程(执行前)

ECS执行上下文栈 堆内存 FEC函数执行上下文

```javascript
执行前： GlobalObject:0x100 FunctionObject:Oxa00
VO(0x100)：FEC中VO就是AO对象 +Date functionfoo0{
function foo() scopechain:[vo+parent scopes] +Array varname=foo
thisBinging:后面讲 +String console.log(name)
var name =foo +Number
console.log(name) +setTimeount
```

开始执行： +window 从上往下一次执行代码 自己定义的变量 name:undefined

```javascript
foo:0xa00(指向函数对象)
```

num1:undefined GEC全局执行上下文 num2:undefined result:undefined 执行前：

```javascript
VO(0x100):GEC中VO就是GO对象
```

scopechain:[vo] ActivationObject:0x200 thisBinding:window 形参 arguments 定义的变量 开始执行： name:"foo" 从上往下一次执行代码

## 函数的执行过程(执行后)

ECS执行上下文栈 堆内存 FEC函数执行上下文

```javascript
执行前： Global0bject:0x100 FunctionObject:0xa00
VO(0x100):FEC中VO就是AO对象 +Date functionfoo0{
scopechain:[vo+parentscopes] +Array varname=foo'
thisBinging:后面讲 +String console.log(name)
```

+Number +setTimeount 开始执行： +window 从上往下一次执行代码 自己定义的变量 name:undefined

```javascript
foo:0xa00(指向函数对象)
```

num1:undefined GEC全局执行上下文 num2:undefined result:undefined 执行前：

```javascript
VO(0x100)：GEC中VO就是GO对象
```

scopechain:[vo] ActivationObject:0x200 thisBinding:window 形参： arguments: 定义的变量 开始执行： name:undefined 从上往下一次执行代码

## 作用域和作用域链连(Scope Chain)

```javascript
1当进入到一个执行上下文时，执行上下文也会关联一个作用域链(ScopeChain)
```

- 作用域链是一个对象列表，用于变量标识符的求值;
- 当进入一个执行上下文时，这个作用域链被创建，并且根据代码类型，添加一系列的对象；
Every execution context has associated with it a scope chain. A scope chain is a list of objects that are searched when evaluating an Identifier. When control enters an execution context,a scope chain is created and populated with an initial set of objects, depending on the type of code. During execution within an execution context, the scope chain of the execution context is affected only by with

```javascript
statements (see 12.10) and catch clauses (see 12.14).
function foo(age) {
function·bar() {
console.log(age) Scope
```

Local this: Window

```javascript
return-bar
closure (foo)
```

Global Window

```javascript
var baz = foo(18)
baz()
```

## 作用域提升面试题

```javascript
var n = 100
var n = 100 function foo() {
function foo() { console.log(n) function fool() {
console.log(n)//·2.100
n=°200 var n = 200
console.log(n)
foo() function foo2() {
var n = 200
console.log(n) var n =·100 console.log(n)//·1.200
foo() foo1()
var a = 100
function foo() { fo02()
var a =·b =·100 console.log(n)·//·3.100
function foo() {
console.log(a)
return
var a =·100 foo()
console.log(a)
foo() console.log(b)
```
