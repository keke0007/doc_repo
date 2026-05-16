Vue基础- Options API

目录 computed计算属性使用 content computed和method区别 computed的set和get 侦听器watch选项使用 侦听器watch其他写法 阶段性综合案例练习

## 复杂data的处理方式

我们知道，在模板中可以直接通过插值语法显示一些data中的数据 但是在某些情况，我们可能需要对数据进行一些转化后再显示，或者需要将多个数据结合起来进行显示；

- 比如我们需要对多个data数据进行运算、三元运算符来决定结果、数据进行某种转化后显示；
- 在模板中使用表达式，可以非常方便的实现，但是设计它们的初衷是用于简单的运算；
- 在模板中放入太多的逻辑会让模板过重和难以维护；
- 并且如果多个地方都使用到，那么会有大量重复的代码；
我们有没有什么方法可以将逻辑抽离出去呢？

- 可以，其中一种方式就是将逻辑抽取到一个method中，放到methods的options中；
- 但是，这种做法有一个直观的弊端，就是所有的data使用过程都会变成了一个方法的调用；
- 另外一种方式就是使用计算属性computed;

## 认识计算属性computed

1什么是计算属性呢？

- 官方并没有给出直接的概念解释；
- 而是说：对于任何包含响应式数据的复杂逻辑，你都应该使用计算属性；
- 计算属性将被混入到组件实例中
```javascript
／所有getter和setter的this上下文自动地绑定为组件实例;
```

1计算属性的用法：

- 选项：computed
```javascript
类型: {[key: string]: Function |{ get: Function, set: Function }}
```

那接下来我们通过案例来理解一下这个计算属性。

## 案例实现思路

我们来看三个案例： 案例一：我们有两个变量：firstName和lastName，希望它们拼接之后在界面上显示； 案例二：我们有一个分数：score

- 当score大于60的时候，在界面上显示及格；
- 当score小于60的时候，在界面上显示不及格；
案例三：我们有一个变量message，记录一段文字：比如HelloWorld

- 某些情况下我们是直接显示这段文字；
- 某些情况下我们需要对这段文字进行反转；
我们可以有三种实现思路：

- 思路一：在模板语法中直接使用表达式；
- 思路二：使用method对逻辑进行抽取；
- 思路三：使用计算属性computed;

## 实现思路一：模板语法

思路一的实现：模板语法

- 缺点一：模板中存在大量的复杂逻辑，不便于维护(模板中表达式的初衷是用于简单的计算)；
- 缺点二：当有多次一样的逻辑时，存在重复的代码；
- 缺点三：多次使用的时候，很多运算也需要多次执行，没有缓存；
```javascript
<！--·1。实现思路一：。--
<template id="my-app">
<h2>{{·firstName +·lastName·}}</h2>
<h2>{{·score°>=°60·？。"及格"：。"不及格"·}}</h2>
<h2>{{ message.split("") .reverse().join(".") }}</h2>
</template>
```

## 实现思路二：method实现

思路二的实现：method实现

- 缺点一：我们事实上先显示的是一个结果，但是都变成了一种方法的调用；
- 缺点二：多次使用方法的时候，没有缓存，也需要多次计算；
```javascript
methods:{
getFullName()·{
<！--·2。实现思路二：·--
return this.firstName + . + this.lastName;
<template id="my-app">
<h2>{{ getFullName()}}</h2> getResult()·{
<h2>{{ getResult()·}}</h2> return this.score·>=·6o ？。"及格"：。"不及格";
<h2>{{·getReverseMessage()·}}</h2>
</template> getReverseMessage()·{
return this.message.split(" ").reverse().join(" ") ;
```

## 思路三的实现：computed实现

思路三的实现：computed实现

- 注意：计算属性看起来像是一个函数，但是我们在使用的时候不需要加O，这个后面讲setter和getter时会讲到;
- 我们会发现无论是直观上，还是效果上计算属性都是更好的选择；
- 并且计算属性是有缓存的;
```javascript
computed:{
fullName()·{
<！--·3.实现思路三：·-- return this.firstName + this.lastName;
<template id="my-app">
<h2>{{ fullName }}</h2> result()°{
<h2>{{ result·}}</h2> return this.score >=·6θ ？·"及格"：·"不及格";
<h2>{{·reverseMessage }}</h2>
</template> reverseMessage() {
return this.message.split(" ").reverse().join(" ") ;
```

## 计算属性vsmethods

■在上面的实现思路中，我们会发现计算属性和methods的实现看起来是差别是不大的，而且我们多次提到计算属性有缓存的。 1接下来我们来看一下同一个计算多次使用，计算属性和methods的差异：

```javascript
computed: {
<！--。1.使用methods result() {
<h2>{{getResult()}}</h2> console.log("调用了计算属性result的getter")；
<h2>{{getResult()}}</h2> return·this.score·>=·6θ ？·"及格"：。"不及格";
```

调用了getResult方法

```javascript
<h2>{{getResult()}}</h2>
```

调用了getResult方法 调用了getResult方法

```javascript
<！--。2.使用computed methods: { 调用了计算属性result的getter
<h2>{{result}}</h2> getResult() {
<h2>{{result}}</h2> console.log("调用了getResult方法")；
<h2>{{result}}</h2> return this.score >=·6θ ？·"及格"：·"不及格";
```

## 计算属性的缓存

1这是什么原因呢？

- 这是因为计算属性会基于它们的依赖关系进行缓存；
- 在数据不发生变化时，计算属性是不需要重新计算的；
- 但是如果依赖的数据发生变化，在使用时，计算属性依然会重新进行计算；
调用了计算属性result的getter 调用了getResult方法 调用了getResult方法 第一次变化 调用了getResult方法 调用了计算属性result的getter 调用了getResult方法 调用了getResult方法 第二次变化 调用了getResult方法 调用了计算属性result的getter

## 计算属性的setter和getter

1计算属性在大多数情况下，只需要一个getter方法即可，所以我们会将计算属性直接写成一个函数。 但是，如果我们确实想设置计算属性的值呢？

- 这个时候我们也可以给计算属性设置一个setter的方法；
```javascript
computed:{
fullName: {
get(){
return this.firstName +.. + this.lastName;
set(value) {
const names = value.split(".") ;
this.firstName = names[o];
this.lastName = names[i];
```

## 源码如何对setter和getter处理呢？ (了解)

你可能觉得很奇怪，Vue内部是如何对我们传入的是一个getter，还是说是一个包含setter和getter的对象进行处理的呢？

- 事实上非常的简单，Vue源码内部只是做了一个逻辑判断而已；
```javascript
if(computedoptions)[
for (const key in computedoptions)
const opt=(computedoptionsas Computedoptions)[key]
const get = isFunction(opt)
?opt.bind(publicThis,publicThis) 判断是否是一个函数
isFunction(opt.get) 函数就取get
？opt.get.bind(publicThis,publicThis) 否则在opt中取get
```

NOOP

```javascript
if(__DEV_-&&get ===NOOP){
const set
!isFunction(opt)&&isFunction(opt.set)
?opt.set.bind(publicThis) 在opt中取set
```

-_DEV_.

```javascript
()=>[
```

:NOOP

```javascript
const c=computed({
```

get, set

## 认识侦听器watch

1什么是侦听器呢？

- 开发中我们在data返回的对象中定义了数据，这个数据通过插值语法等方式绑定到template中；
- 当数据变化时，template会自动进行更新来显示最新的数据；
- 但是在某些情况下，我们希望在代码逻辑中监听某个数据的变化，这个时候就需要用侦听器watch来完成了;
侦听器的用法如下：

- 选项：watch
```javascript
类型: { [key: string]: string | Function | Object | Array}
```

## 侦听器案例

```javascript
举个栗子(例子)：
```

- 比如现在我们希望用户在input中输入一个问题；
- 每当用户输入了最新的内容，我们就获取到最新的内容，并且使用该问题去服务器查询答案；
- 那么，我们就需要实时的去获取最新的数据变化；
```javascript
watch:{
question(newValue,·oldvalue)
<template id="my-app"> console.log(newValue) ;
<label·for="question"> this.getAnwser(newValue);
```

请输入问题：

```javascript
<input type="text" id="question" v-model="question">
</label> methods:.{
</template> getAnwser(question) {
console.log(^${question}的问题答案是哈哈哈哈^);
```

## 侦听器watch的配置选项

我们先来看一个例子：

- 当我们点击按钮的时候会修改info.name的值；
- 这个时候我们使用watch来侦听info，可以侦听到吗？答案是不可以。
这是因为默认情况下，Watch只是在侦听info的引l用变化，对于内部属性的变化是不会做出响应的：

- 这个时候我们可以使用一个选项deep进行更深层的侦听；
- 注意前面我们说过watch里面侦听的属性对应的也可以是一个Object;
1还有另外一个属性，是希望一开始的就会立即执行一次：

- 这个时候我们使用immediate选项；
- 这个时候无论后面数据是否有变化，侦听的函数都会有限执行一次；
■代码在下一页课件中

## 侦听器watch的配置选项页(代码)

```javascript
watch:{
info:{
handler(newValue, oldvalue) {
console.log(newValue, oldvalue) ;
```

deep: true, immediate: true

```javascript
'info.name': function(newValue, oldvalue) {
console.log(newValue, oldvalue) ;
```

## 侦听器watch的其他方式(一)

字符串方法名 "someMethod" 你可以传入回调数组，它们会被逐一调用 ]:J "handlel",

```javascript
function handle2(val, oldVal) {
console.log("handle2 triggered");
handler: function handle3(val, oldval)
console.log("handle3 triggered");
```

## 侦听器watch的其他方式(二)

1另外一个是Vue3文档中没有提到的，但是Vue2文档中有提到的是侦听对象的属性：

```javascript
'info.name': function(newValue, oldValue)
console.log(newValue, oldvalue) ;
```

还有另外一种方式就是使用$watch的APl：

```javascript
我们可以在created的生命周期(后续会讲到)中，使用this.$watchs来侦听；
```

- 第一个参数是要侦听的源；
- 第二个参数是侦听的回调函数callback；
- 第三个参数是额外的其他选项，比如deep、immediate;
```javascript
created() {
this.$watch('message',·(newValue, oldvalue) => {
console.log(newValue, oldvalue) ;
}，·{deep: true,·imediate: true})
```

## 综合案例

现在我们来做一个相对综合一点的练习：书籍购物车 书籍名称 出版日期 价格 购买数量 操作 《算法导论》 2006-9 ￥85 移除 《UNIX编程艺术》 2006-2 ¥59 移除 《编程珠玑》 2008-10 ￥39 移除 《代码大全》 2006-3 ￥128 + 移除 总价：￥311 案例说明：

- 1.在界面上以表格的形式，显示一些书籍的数据；
- 2.在底部显示书籍的总价格；
- 3.点击+或者-可以增加或减少书籍数量(如果为1，那么不能继续-);
- 4.点击移除按钮，可以将书籍移除(当所有的书籍移除完毕时，显示：购物车为空～);
