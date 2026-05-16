Vue3 - Composition API

目录 computed函数使用 content 组件的生命周期函数 Provide/lnject使用 watch/watchEffect 自定义Hook练习 scriptsetup语法糖

## computed

1在前面我们讲解过计算属性computed：当我们的某些属性是依赖其他状态时，我们可以使用计算属性来处理

- 在前面的OptionsAPl中，我们是使用computed选项来完成的；
- 在CompositionAPl中，我们可以在setup函数中使用computed方法来编写一个计算属性；
如何使用computed呢？

- 方式一：接收一个getter函数，并为getter 函数返回的值，返回一个不变的ref对象;
- 方式二：接收一个具有get和set的对象，返回一个可变的(可读写)ref对象；
```javascript
const fullName =·computed({
get: () =>{
return firstName.value + Il II + lastName.value;
const fullName = computed(() => {
return firstName.value + ".I + lastName.value; set: newValue =>
}) const names = newValue.split(" ") ;
firstName.value = names[o];
lastName.value = names[1] ;
```

## setup中使用ref

在setup中如何使用ref获取元素或者组件？

- 其实非常简单，我们只需要定义一个ref对象，绑定到元素或者组件的ref属性上即可;
```javascript
<template>
<div>
<h2·ref="titleRef!>我是标题</h2>
</div>
</template>
<script>
import { ref·} from."vue";
export default {
setup()
const titleRef =·ref(null);
```

retuln titleRef

```javascript
</script>
```

## 生命周期钩子

我们前面说过setup 可以用来替代data、methods、computed等等这些选项，也可以替代生命周期钩子。 那么setup中如何使用生命周期函数呢？

- 可以使用直接导入的on×函数注册生命周期钩子；
选项式API Hookinside setup

```javascript
onMounted(() => { beforecreate Not needed*
console.log("onMounted")
```

created Not needed* beforeMount onBeforeMount

```javascript
onUpdated(() => { mounted onMounted
console.log('onUpdate') beforeupdate onBeforeUpdate TIP
```

updated onUpdated 因为setup是围绕beforeCreate和created生命周期钩子运行的，所以不需要显式地定 义它们。换句话说，在这些钩子中编写的任何代码都应该直接在setup函数中编写。 beforeunmount onBeforeUnmount

```javascript
onUnmounted(() =>{
console.log('onUnmounted') unmounted onUnmounted
```

activated onActivated deactivated onDeactivated

## Provide函数

1事实上我们之前还学习过Provide和lnject，CompositionAPl也可以替代之前的Provide和Inject的选项。 我们可以通过provide来提供数据：

- 可以通过provide方法来定义每个Property;
provide可以传入两个参数：

- name：提供的属性名称;
- value：提供的属性值;
```javascript
let counter =·100
let info = {
```

name::"why", age: 10

```javascript
provide("counter",°counter)
provide("info", info)
```

## Inject函数

1在后代组件中可以通过inject来注入需要的属性和对应的值：

- 可以通过inject来注入需要的内容；
Iinject可以传入两个参数：

- 要 inject 的 property 的 name;
- 默认值；
```javascript
const counter = inject("counter")
const info =.inject("info")
```

## 数据的响应式

为了增加 provide 值和 inject 值之间的响应性，我们可以在 provide 值时使用 ref和 reactive。

```javascript
let:counter =:ref(1oo)
let info = reactive({
```

name: :"why", age: 18 3)

```javascript
provide("counter",°counter)
provide("info", info)
```

## 侦听数据的变化

1在前面的OptionsAPl中，我们可以通过watch选项来侦听data或者props的数据变化，当数据变化时执行某一些操作。 在CompositionAPl中，我们可以使用watchEffect和watch来完成响应式数据的侦听；

- watchEffect：用于自动收集响应式数据的依赖；
- watch：需要手动指定侦听的数据源；

## Watch的使用

watch的APl完全等同于组件watch选项的Property：

- watch需要侦听特定的数据源，并且执行其回调函数；
- 默认情况下它是惰性的，只有当被侦听的源发生变化时才会执行回调；
```javascript
const name = ref("kobe")
watch(name,°(newValue,oldvalue) => {
console.log(newValue, oldValue) ;
})
constchangeName =()=>{
name.value: = "james";
```

## 侦听多个数据源

侦听器还可以使用数组同时侦听多个源：

```javascript
const name = ref("why") ;
const age = ref(18)
const changeName =()·=>{
name.value = "james";
watch([name, age],·(newValues, oldvalues) => {
console.log(newValues, oldValues);
})
```

## Watch的选项

如果我们希望侦听一个深层的侦听，那么依然需要设置deep为true：

- 也可以传入immediate立即执行；
```javascript
const info = reactive({
```

name::"why", age: 18,

```javascript
friend: {
```

name:."kobe" (C

```javascript
watch(info,°(newValue,oldvalue) => {
console.log(newValue,°oldValue)
}，
```

immediate: true, deep:·true

```javascript
})
```

## WatchEffect

1当侦听到某些响应式数据变化时，我们希望执行某些操作，这个时候可以使用watchEffect。 我们来看一个案例：

- 首先，watchEffect传入的函数会被立即执行一次，并且在执行的过程中会收集依赖;
- 其次，只有收集的依赖发生变化时，watchEffect传入的函数才会再次执行;
```javascript
const name = ref("why") ;
const age = ref(18) ;
watchEffect(() => {
console.log("watchEffect执行~",·name.value, age.value) ;
```

## WatchEffect的停止侦听

如果在发生某些情况下，我们希望停止侦听，这个时候我们可以获取watchEffect的返回值函数，调用该函数即可。 比如在上面的案例中，我们age达到20的时候就停止侦听：

```javascript
const stopwatch =·watchEffect(() =>{
console.log("watchEffect执行~",·name.value,age.value);
const changeAge =。()·=>{
age.value++;
if (age.value > 20) {
stopWatch() ;
```

## useCounter

我们先来对之前的counter逻辑进行抽取：

```javascript
import { ref } from 'vue'
export function useCounter() {
const counter =:ref(o) ;
const increment =:○=>counter.value++
constdecrement =()=>counter.value--
return {
```

counter, increment, decrement

## useTitle

我们编写一个修改title的Hook:

```javascript
import { ref, watch } from.'vue'
export function useTitle(title =。'默认值')·{
const titleRef = ref(title);
watch(titleRef,·(newValue) => {
document.title = newValue;
```

immediate: true

```javascript
return titleRef;
```

## useScrollPosition ((作业)

我们来完成一个监听界面滚动位置的Hook：

```javascript
import { ref } from "vue";
export function useScrollPosition() {
const scrollx: = ref(0)
const scrolly:=: ref(0)
document.addEventListener('scroll',°() => {
scrollx.value = window.scrollx
scrolly.value = window.scrolly
return { scrollx,·scrolly·}
```

## scriptsetup语法

```javascript
<scriptsetup>是在单文件组件(SFC)中使用组合式API的编译时语法糖，当同时使用SFC与组合式API时则推荐该语法。
```

- 更少的样板内容，更简洁的代码；
- 能够使用纯Typescript声明prop和抛出事件；
- 更好的运行时性能；
- 更好的IDE类型推断性能；
```javascript
使用这个语法，需要将setupattribute添加到<script>代码块上：
<script setup>
console.log("Hello World")
</script>
```

里面的代码会被编译成组件setupO函数的内容：

- 这意味着与普通的<script>只在组件被首次引l入的时候执行一次不同；

## 顶层的绑定会被暴露给模板

```javascript
当使用<script setup>的时候，任何在<script setup>声明的顶层的绑定(包括变量，函数声明，以及import引l入的内容)
```

都能在模板中直接使用：

```javascript
<script setup>
const message = "Hello World"
function btnclick() {
console.log("btnclick")
</script>
<template>
<h2>message:·{{·message·}}</h2>
<button·@click="btnclick">按钮</button>
</template>
```

响应式数据需要通过ref、reactive来创建。

## 导入的组件直接使用

```javascript
<scriptsetup>范围里的值也能被直接作为自定义组件的标签名使用：
<script setup>
```

import ShowInfo from './ShowInfo.vue'

```javascript
</script>
<template>
<show-info></show-info>
</template>
```

## definePropsO 和 defineEmits(

1为了在声明props和emits选项时获得完整的类型推断支持，我们可以使用defineProps和defineEmitsAPl，它们将自动

```javascript
地在<scriptsetup>中可用：
<script setup>
const props = defineProps({
name: {
```

type: String, default:."

```javascript
<template>
age:{
<h2>ShowInfo: {{ name }}-{{ age }}</h2>
```

type:·Number,

```javascript
default:.0 <button·@click="changeAge">修改age</button>
</template>
const emit = defineEmits(["changeAge"])
function changeAge()·{
emit("changeAge", 200)
</script>
```

## defineExposeO

```javascript
使用<scriptsetup>的组件是默认关闭的：
```

- 通过模板ref或者$parent链获取到的组件的公开实例，不会暴露任何在<scriptsetup〉中声明的绑定；
```javascript
通过defineExpose编译器宏来显式指定在<scriptsetup>组件中要暴露出去的property:
function foo() {
console.log("foo:function") const showInfoRef = ref(null)
function:callShowInfo() {
showInfoRef.value.foo()
defineExpose({
```

foo

```javascript
})
```

## 案例实战练习

高分好评房源 品质房源，低至5折 整套公寓型住宅1室1卫1床 整套公寓型住宅1室1卫1床 独立房间1室1卫1床 价格真实实图拍摄整套单独使用每客消毒高漫漫|杨桃轻奢一居室/地铁口/近春熙路太古 【可月租】【网红美食街区】/近春熙路/宽窄 清投影【方糖】人民北路地铁|龙湖上城|火车..里/4米9挑高/全景落地窗 巷子/熊猫基地//【轻奢大床房】 ￥158/晚 ￥201/晚 ￥150/晚 超赞房东 超赞房东 超赞房东 整套公寓型住宅1室1卫1床 整套公寓型住宅1室1卫1床 整套公寓1室1卫1床 可月租！品质大床！高空观景露台/步行地铁 「精致mini房]楼下商场|地铁直达近春熙路 【住.颜23】免清洁费/下楼就是太古里春熙路/ 站/白天免费停车/直达春熙路/近建设巷小吃街太古里|建设巷小吃街|白天免费停车|可..高空浴缸/落地窗带阳台/百寸极米投影双地.. ￥146/晚 ￥146/晚 ￥212/晚 超赞房东 超赞房东 超赞房东
