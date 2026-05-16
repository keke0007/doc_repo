Vue全家桶-Pinia状态管理

## 目录

Pinia和Vuex的对比 content 创建Pinia的Store Pinia核心概念State Pinia核心概念Getters Pinia核心概念Actions

## 什么是Pinia呢？

```javascript
1Pinia(发音为/pi:nj^/，如英语中的"peenya")是最接近pina(西班牙语中的菠萝)的词;
```

- 从那时到现在，最初的设计原则依然是相同的，并且目前同时兼容Vue2、Vue3，也并不要求你使用CompositionAPl;
- Pinia本质上依然是一个状态管理的库，用于跨组件、页面进行状态共享(这点和Vuex、Redux一样)；
## Pinia

TheVueStorethatyouwill enjoyusing

## Pinia和Vuex的区别

那么我们不是已经有Vuex了吗？为什么还要用Pinia呢？

- Pinia最初是为了探索Vuex的下一次迭代会是什么样子，结合了Vuex5核心团队讨论中的许多想法；
- 最终，团队意识到Pinia已经实现了Vuex5中大部分内容，所以最终决定用Pinia来替代Vuex;
- 与Vuex相比，Pinia 提供了一个更简单的 APl，具有更少的仪式，提供了Composition-API 风格的 APl;
- 最重要的是，在与TypeScript一起使用时具有可靠的类型推断支持;
和vuex相比，Pinia有很多的优势：

- 比如mutations不再存在：
他们经常被认为是非常沉长； 他们最初带来了devtools集成，但这不再是问题；

- 更友好的TypeScript支持，Vuex之前对Ts的支持很不友好； Store StoreA
State Actions

- 不再有modules的嵌套结构： Getters
Module A ModuleB (你可以灵活使用每一个store，它们是通过扁平化的方式来相互使用的； StoreB State State Mutations Mutations State Actions Actions Actions

- 也不再有命名空间的概念，不需要记住它们的复杂关系； Getters Getters Getters

## 如何使用Pinia？

使用Pinia之前，我们需要先对其进行安装： yarn add pinia #or with npm npm instatt pinia 创建一个pinia并且将其传递给应用程序：

```javascript
import { createPinia } from 'pinia'
const pinia = createPinia()
```

export default pinia import pinia from.'./store'

```javascript
createApp(App).use(pinia).mount('#app')
```

## 认识store

什么是Store？

- 它有点像始终存在，并且每个人都可以读取和写入的组件；
- 你可以在你的应用程序中定义任意数量的store来管理你的状态；
Store有三个核心概念：

```javascript
state、getters、actions;
```

- 等同于组件的data、computed、methods;
- 一旦store 被实例化，你就可以直接在store上访问 state、getters和 actions 中定义的任何属性;

## 定义一个Store

定义一个store:

- 我们需要知道Store是使用defineStoreO定义的，
- 并且它需要一个唯一名称，作为第一个参数传递；
```javascript
export const useCounter = defineStore("counter",{
state(){
return {
```

counter:0 这个name，也称为id，是必要的，Pinia使用它来将store 连接到devtools。 返回的函数统一使用useX作为命名方案，这是约定的规范；

## 使用定义的Store

Store在它被使用之前是不会创建的，我们可以通过调用use函数来使用store：

```javascript
<template>
<div·class="home">
<h2>Counter: {{ counterStore.counter }}</h2>
</div>
</template>
<script setup>
import { useCounter } from '@/store/counter'
const counterStore = useCounter()
</script>
```

注意Store获取到后不能被解构，那么会失去响应式：

- 为了从Store中提取属性同时保持其响应式，您需要使用storeToRefs(。
```javascript
const·{·counter·} =·counterStore//·不是响应式的
const { counter：·counter2 } = toRefs(counterStore)·//·是响应式的
const { counter:°counter3 } = storeToRefs(counterStore)°// 是响应式的
```

## 认识和定义State

state是store的核心部分，因为store是用来帮助我们管理状态的。

- 在 Pinia中，状态被定义为返回初始状态的函数;
```javascript
export const useCounter = defineStore("counter",°
state:()=>({
```

counter:0, name: "why", age: 18

```javascript
})
```

## 操作State 一

读取和写入state：

- 默认情况下，您可以通过store实例访问状态来直接读取和写入状态；
```javascript
constcounterStore=·useCounter()
```

counterStore.counter++

```javascript
counterStore.name =."coderwhy"
```

重置State：

- 你可以通过调用store上的$reset)方法将状态重置到其初始值；
```javascript
const counterStore =·useCounter()
counterStore.$reset()
```

## 操作State (二)

改变State:

- 除了直接用store.counter++修改store，你还可以调用$patch方法；
- 它允许您使用部分"state"对象同时应用多个更改；
```javascript
const·counterStore =·useCounter()
counterStore.$patch({
```

counter:·1o0, name::"kobe"

```javascript
})
```

替换State:

- 您可以通过将其$state属性设置为新对象来替换Store的整个状态：
```javascript
counterStore.$state ={
```

counter:·1, name::"why"

## 认识和定义Getters

IGetters相当于Store的计算属性：

- 它们可以用defineStore(中的getters属性定义;
```javascript
getters中可以定义接受一个state作为参数的函数;
export const useCounter = defineStore("counter",{
state:()=>({
```

counter: 100, firstname:."kobe" Lastname:."bryant" age: 18

```javascript
})，
getters: {
doubleCounter:·(state) => state.counter * 2,
doublePlusOne: (state) => state.counter * 2 + 1,
fullname:·(state) => state.firstname + state.lastname
```

## 访i问Getters

访问当前store的Getters:

```javascript
const counterStore = useCounter()
console.log(counterStore.doubleCounter)
console.log(counterStore.fullname)
```

Getters中访问自己的其他Getters：

- 我们可以通过this来访问到当前store实例的所有其他属性
```javascript
doublePlusOne: function(state) {
return this.doubleCounter + 1
```

访问其他store的Getters：

```javascript
message: function(state) {
const userStore = useuser()
return this.fullname + ":" + userStore.nickname
```

## 访问Getters (二)

Getters也可以返回一个函数，这样就可以接受参数：

```javascript
export const useMain = defineStore("main",·{
state:·() => ({
```

users:·[

```javascript
const mainStore =·useMain()
{ id: 11l, name: "why", age:·18 },
{ id: 112, name:·"kobe", age: 20 }, const getUserById = mainStore.getUserById
{ id: 113, name:·"james", age: 25 },
getters:·{ <h2>User: {{ getUserById(111) }}</h2>
getUserById(state)·{ <h2>User: {{ getUserById(112) }}</h2>
return userId·=>{
return state.users.find(item => item.id === userId)
```

## 认识和定义Actions

Actions相当于组件中的methods.

- 可以使用defineStoreO中的actions属性定义，并且它们非常适合定义业务逻辑；
```javascript
actions:{
function increment() {
increment() {
counterStore.increment()
```

this.counter++

```javascript
randomCounter() {
function randomClick() {
this.counter = Math.random()
counterStore.randomCounter()
```

1和getters一样，在action中可以通过this访问整个store实例的所有操作；

## Actions执行异步操作

并且Actions中是支持异步操作的，并且我们可以编写异步函数，在函数中使用await；

```javascript
actions: {
increment() {.
randomCounter() {"
async fetchHomeDataAction() {
const res = await fetch("http://123.207.32.32:8000/home/multidata")
const data = await res.json()
console.log("data:",·data)
return data
const counterStore = useCounter()
counterStore.fetchHomeDataAction().then(res => {
console.log(res)
```
