Vue组件化－插槽Slot/非父子通信

## 目录

认识插槽Slot作用 content 插槽Slot基本使用 具名插槽Slot使用 作用域插槽Slot使用 全局事件总线使用 依赖注入Provide/lnject

## 认识插槽Slot

1在开发中，我们会经常封装一个个可复用的组件：

- 前面我们会通过props传递给组件一些数据，让组件来进行展示;
- 但是为了让这个组件具备更强的通用性，我们不能将组件中的内容限制为固定的div、span等等这些元素;
- 比如某种情况下我们使用组件，希望组件显示的是一个按钮，某种情况下我们使用组件希望显示的是一张图片；
- 我们应该让使用者可以决定某一块区域到底存放什么内容和元素；
举个栗子：假如我们定制一个通用的导航组件－NavBar

- 这个组件分成三块区域：左边-中间-右边，每块区域的内容是不固定;
- 左边区域可能显示一个菜单图标，也可能显示一个返回按钮，可能什么都不显示；
- 中间区域可能显示一个搜索框，也可能是一个列表，也可能是一个标题，等等；
- 右边可能是一个文字，也可能是一个图标，也可能什么都不显示;
三 JD Q签字笔 登录 购物车 商品 评价 详情 推荐 Q烧烤炉

## 如何使用插槽slot？

1这个时候我们就可以来定义插槽slot：

- 插槽的使用过程其实是抽取共性、预留不同；
- 我们会将共同的元素、内容依然在组件内进行封装；
- 同时会将不同的元素使用slot作为占位，让外部决定到底显示什么样的元素；
如何使用slot呢？

```javascript
Vue中将<slot>元素作为承载分发内容的出口;
```

- 在封装组件中，使用特殊的元素<slot>就可以为封装组件开启一个插槽；
- 该插槽插入什么内容取决于父组件如何使用；
```javascript
parent template <FancyButton>template
<FancyButton> <button>
```

replace

```javascript
Click Me <slot></slot>
</FancyButton> </button>
```

slot content slot outlet

## 插槽的默认内容

1有时候我们希望在使用插槽时，如果没有插入对应的内容，那么我们需要显示一个默认的内容：

- 当然这个默认的内容只会在没有提供插入的内容时，才会显示；
.vueU MySlotCpn.vue U

```javascript
b7_插槽的使用>VMySlotCpn.vue>{}"MysivtCon.vue">script
<template> App.vue U MySlotCpn.vue U
<div> src>O7_插槽的使用>VApp.vue>{}"App.vue">script>
<h2>MySlotCpn开始</h2> <template>
<slot> <div> 没有插入内容
<h2>我是默认显示的内容</h2> <my-slot-cpn>
</slot> </my-slot-cpn>
<h2>MySlotCpn结尾</h2> </div>
```

默认内容会显示

```javascript
</div> </template>
</template>
```

## 多个插槽的效果

我们先测试一个知识点：如果一个组件中含有多个插槽，我们插入多个内容时是什么效果？

- 我们会发现默认情况下每个插槽都会获取到我们插入的内容来显示；
App.vue U NavBar.vueUX App.vue U x NavBar.vueU

```javascript
src>07_插槽的使用>VNavBar.vue>{}"NavBar.vue"> template>div.nav-bar>div.right src>07_插槽的使用>
<template> <template>
<div·class="nav-bar"> <div>
<div class="left"> <nav-bar>
<slot></slot> <button>左边按钮</button>
</div> <h2>中间标题</h2>
<div.class="center"> <i>右边i元素</i>
<slot></slot> </nav-bar>
</div> </div>
<div class="right"> </template>
<slot></slot>
</div>
</div> 左边按钮左边按钮 左边按钮
</template>
```

中间标中间标题 中间标 题 题

```javascript
<script> 右边i元素
export default { 右边i元素 右边元素
```

## 插槽的基本使用

我们一个组件MySlotCpn.vue：该组件中有一个插槽，我们可以在插槽中放入需要显示的内容； 我们在App.vue中使用它们：

- 我们可以插入普通的内容、html元素、组件，都可以是可以的；
App.vue UX

```javascript
src>07_插槽的使用>VApp.vue >{}"App.vue"> script
MySlotCpn.vue U X <template>
src>O7_插槽的使用>VMySlotCpn.vue>{}"MySlotCpn.vue" <div>
<template>
<my-slot-cpn>
<div>
```

- 1。普通的内容
```javascript
<h2>MySlotCpn开始</h2>
```

Hello World

```javascript
<slot></slot>
<！--·2.html元素。-->
<h2>MySlotCpn结尾</h2>
<^LP/> <button>我是按钮</button>
</template> <！--·3.组件元素。-->
<my-button></my-button>
</my-slot-cpn>
</div>
</template>
```

## 具名插槽的使用

事实上，我们希望达到的效果是插槽对应的显示，这个时候我们就可以使用具名插槽：

- 具名插槽顾名思义就是给插槽起一个名字，<slot>元素有一个特殊的attribute：name;
- 一个不带name的slot，会带有隐含的名字default;
App.vueU NavBar.vueUx App.vue UX NavBar.vueU

```javascript
src>07_插槽的使用>VNavBar.vue>{}"NavBar.vue">script src>07_插槽的使用> App.vue>{}"App.vue">template
<template> <template>
<div class="nav-bar"> <div>
<div class="left"> <nav-bar>
<slot name="left"></slot> <template v-slot:left>
</div> <button>左边按钮</button>
<div class="center"> </template>
<slot name="center"></slot> <template v-slot:center>
</div> <h2>中间标题</h2>
<div class="right"> </template>
<slot name="right"></slot> <template v-slot:right>
</div> <i>右边i元素</i>
</div> </template>
</template> </nav-bar>
左边按钮 右边i元素 </div>
```

中间标题

```javascript
<script> </template>
export default {
```

## 动态插槽名

■什么是动态插槽名呢？

- 目前我们使用的插槽名称都是固定的，
```javascript
比如v-slot:left、v-slot:center等等;
```

- 我们可以通过v-slot:[dynamicSlotName]方式动态绑定一个名称；
```javascript
<template>
<div>
<nav-bar>
<template v-slot:[name]
<button>左边按钮</button> data()
</template> return
<template v-slot:center> name:"left"
<h2>中间标题</h2>
</template>
<template v-slot:right>
<i>右边i元素</i>
</template>
</nav-bar>
</div>
</template>
```

## 具名插槽使用的时候缩写

具名插槽使用的时候缩写：

- 跟v-on和v-bind一样，V-slot 也有缩写；
- 即把参数之前的所有内容(v-slot:)替换为字符#;
```javascript
<template>
<div>
<nav-bar>
<template #left>
<button>左边按钮</button>
</template>
<template #center>
<h2>中间标题</h2>
</template>
<template #right>
<i>右边i元素</i>
</template>
</nav-bar>
</div>
</template>
```

## 渲染作用域

1在Vue中有渲染作用域的概念：

- 父级模板里的所有内容都是在父级作用域中编译的；
- 子模板里的所有内容都是在子作用域中编译的；
如何理解这句话呢？我们来看一个案例：

- 在我们的案例中ChildCpn自然是可以让问自己作用域中的title内容的；
- 但是在App中，是访问不了ChildCpn中的内容的，因为它们是跨作用域的访问;
■案例见下页

## 渲染作用域案例

App.vue U ChildCpn.vue U ×

```javascript
src>08_作用域插槽
<template>
```

h2元素中可以访问title

```javascript
<div>
<h2>{{title}}</h2>
```

App.vueUx ChildCpn.vue U

```javascript
<slot></s lot>
src>08_作用域插槽 App.vue>{}"App.vue">template
</div>
<template>
</template> 插槽span不可以访问title
<div>
```

只能访问App作用域中的内容

```javascript
<child-cpn>
<script>
<span>{{title}}</span>
```

export detault

```javascript
</child-cpn>
data()
</div>
```

retur

```javascript
</template>
```

title:."Hello Child Cpn"

```javascript
</script>
```

## 认识作用域插槽

1但是有时候我们希望插槽可以访问到子组件中的内容是非常重要的：

- 当一个组件被用来渲染一个数组元素时，我们使用插槽，并且希望插槽中没有显示每项的内容；
- 这个Vue给我们提供了作用域插槽；
我们来看下面的一个案例：

- 1.在App.vue中定义好数据
- 2.传递给ShowNames组件中
- 3.ShowNames组件中遍历names数据
- 4.定义插槽的prop
- 5.通过v-slot:default的方式获取到slot的props
- 6.使用slotProps中的item和index

## 作用域插槽的案例

VApp.vueUX ShowNames.vueU

```javascript
src>08_作用域插槽>VApp.vue>{}"App.vue">template>div>show-names App.vue U ShowNames.vue UX
<template> src>08_作用域插槽>VShowNames.vue>{}"ShowNames.vue">script>[e] default
<div> 5.通过v-slot:default的方式获取到slot的props <template>
<show-names:names="names"> <div> 3.ShowNames组件中遍历names数据
<template v-slot:default="slotProps" <template v-for="(item,·index) in names".:key="item">
<span>{{slotProps.item}}-{{slotProps.index}}</span> 插槽prop 4.定义插槽prop
</template> 6.使用其中的item和index属性 <slot :item="item" :index="ndex"></slot>
```

- show-names> </template>
```javascript
</div> </div>
</template> </template>
<script> <script>
import·ShowNames·from './ShowNames.vue'; export defaylt
```

props 2.传递到了ShowNames组件中

```javascript
export default { names: {
components:·{ type: Array,
ShowNames, default:·() =>·[]
data(){ 1.在App.vue中定义好数据
return
names:["why""kobe""james"""curry"] </script>
```

## 独占默认插槽的缩写

```javascript
<show-names :names="names">
<templatev-slot="slotProps">
<span>{{slotProps.item}}-{{slotProps.index}}</span>
</template>
</show-names>
```

并且如果我们的插槽只有默认插槽时，组件的标签可以被当做插槽的模板来使用，这样，我们就可以将v-slot直接用在组件上：

```javascript
<show-names:names="names"v-slot="slotProps">
<span>{{slotProps.item}}-{{slotProps.index}}</span>
</show-names>
```

## 默认插槽和具名插槽混合

但是，如果我们有默认插槽和具名插槽，那么按照完整的template来编写。

```javascript
<template>
<div>
<show-names :names="names" V-slot="slotProps">
<span>{{slotProps.item]}-[vue/valid-v-slot]
Default slot must use <template>! on a custom element when there are other
<template v-slot:why> named slots.eslint-plugin-vue
<h2>哈哈哈</h2> View Problem(FB)Noquickfixesavailable
</template>
</show-names>
只要出现多个插槽，请始终为所有的插槽使用完整的基于<template>的语法：
<show-names :names="names">
<templatev-slot="slotProps">
<span>{{slotProps.item}}-{{slotProps.index}}</span>
</template>
<template v-slot:why>
<h2>哈哈哈</h2>
</template>
show-names>
```

## 非父子组件的通信

！在开发中，我们构建了组件树之后，除了父子组件之间的通信之外，还会有非父子组件之间的通信。 这里我们主要讲两种方式：

- 全局事件总线；
```javascript
Provide/lnject;
```

## 全局事件总线mitt库

1Vue3从实例中移除了$on、$off和$once方法，所以我们如果希望继续使用全局事件总线，要通过第三方的库：

- Vue3官方有推荐一些库，例如mitt或tiny-emitter;
- 这里我们主要讲解一下hy-event-store的使用；
首先，我们需要先安装这个库： npm install hy-event-bus 1其次，我们可以封装一个工具eventbus.js：

```javascript
import {·HYEventBus-} from-'hy-event-store'
const·eventBus=·new-HYEventBus()
```

export default eventBus

## 使用事件总线工具

在项目中可以使用它们：

- 我们在App.vue中监听事件；
- 我们在Banner.vue中触发事件；
```javascript
<script>
```

import eventBus from-'./utils/event-bus' export default import eventBus from './utils/event-bus

```javascript
methods:-{
prevclick() { export default {
components: {
nextclick() { AppContent
created() {
eventBus.on("bannerclick",·(payload) => {
</script> console.log(payload)
<template>
<button @click="prevclick">上一个</button>
<button @click="nextclick">下一个</button>
</template>
```

## Mitt的事件取消

1在某些情况下我们可能希望取消掉之前注册的函数监听：

```javascript
cancelListener() {
eventBus.off("bannerclick", this.bannerClick)
```

## Provide和lnject

1Provide/lnject用于非父子组件之间共享数据：

- 比如有一些深度嵌套的组件，子组件想要获取父组件的部分内
```javascript
容;
```

- 在这种情况下，如果我们仍然将props沿着组件链逐级传递下 Provide
去，就会非常的麻烦； 对于这种情况下，我们可以使用Provide和Inject：

- 无论层级结构有多深，父组件都可以作为其所有子组件的依赖
```javascript
提供者;
```

- 父组件有一个provide选项来提供数据;
- 子组件有一个 inject 选项来开始使用这些数据;
实际上，你可以将依赖注入看作是"longrange props"，除了: Inject

- 父组件不需要知道哪些子组件使用它provide的property
- 子组件不需要知道inject的property来自哪里

## Provide和lnject基本使用

我们开发一个这样的结构： App.vue Home.vue HomeContent.vue App.vueUX VHomeContent.vueU

```javascript
src>05_Provide和inject>VApp.vue>{}"App.vue>script>[e]default>provide
<template> vueU HomeContent.vue UX
<div> 5_Provide和lnject>HomeContent.vue>{}"HomeContent.vue"
<home></home> <template>
</div> <div>
</template> <h2>HomeContent</h2>
<h2>{{name}}-{{age}}</h2>
<script> </div>
import Home from'./Home.vue'; </template>
export default <script>
```

components: export defaul Home inject:.["name","age"

```javascript
provide: </script>
```

name:."why'

```javascript
age: 18 <style scoped>
</style>
</script>
```

## Provide和lnject函数的写法

如果Provide中提供的一些数据是来自data，那么我们可能会想要通过this来获取： 这个时候会报错：

- 这里给大家留一个思考题，我们的this使用的是哪里的this？
```javascript
<script>
import Home from ./Home.vue'; Uncaught TypeError:Cannot read propertynames'of undefined
ateval(App.vue?af3e:22)
```

atModule../nodemodules/cache-loader/dist/cis.is?!./nodemodules/babel-loader/lib/index.is!./ export default components: Home

```javascript
provide()
data() return
return
```

names!["abc","cba"] name:"why" age: 18, length: this.names.length provide: name:"why' age: 18, length: this.names.length

```javascript
</script>
```

## 处理响应式数据

我们先来验证一个结果：如果我们修改了this.names的内容，那么使用length的子组件会不会是响应式的？ 我们会发现对应的子组件中是没有反应的：

- 这是因为当我们修改了names之后，之前在provide中引l入的this.names.length本身并不是响应式的；
那么怎么样可以让我们的数据变成响应式的呢？

- 非常的简单，我们可以使用响应式的一些APl来完成这些功能，比如说computed函数；
- 当然，这个computed是vue3的新特性，在后面我会专门讲解，这里大家可以先直接使用一下;
注意：我们在使用length的时候需要获取其中的value

- 这是因为computed返回的是一个ref对象，需要取出其中的value来使用；
```javascript
provide()
return
<div>
```

name:why

```javascript
<h2>HomeContent</h2>
age:18 <h2>{{name}}-{fagell-length.value}]</h2>
Length: computed(()-=> this.names.length)
</div>
```
