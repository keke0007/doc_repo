```javascript
Vue基础－模板语法((二)
```

目录 v-for列表渲染 content v-for渲染类型 数组更新的检测 v-for的key属性 Vue的虚拟DOM

```javascript
v-for的diff算法(后续)
```

## 列表渲染

在真实开发中，我们往往会从服务器拿到一组数据，并且需要对其进行渲染。

- 这个时候我们可以使用v-for来完成；
- v-for类似于JavaScript的for循环，可以用于遍历一组数据；
热门推荐华语流行摇滚民谣电子 更多 声音来信 24万 1227万 1280万 18348 C 逃避烦恼|去尝尝夏天 【华语励志篇】你就是 安静细腻的欧美小调， 电台家境县殊的恋 独有的梅子味晚霞 主角谁都无可代替 献给失眠的你 爱：每天都像在走钢 丝，我不敢往下看 一周 音乐 时空 影耕歌 6512 965万 1456万 [一周影视热歌]王嘉尔 电台回到2003年： 点燃二次元百首燃值 电台哪里都是你&水

```javascript
演唱《小黄人》原声 王心凌林俊杰出道，天 爆表的ACG神曲 星记(完整版)
```

后归位！

## v-for基本使用

Iv-for的基本格式是"itemin数组"：

- 数组通常是来自data或者prop，也可以是其他方式;
- item是我们给每项元素起的一个别名，这个别名可以自定来定义；
我们知道，在遍历一个数组的时候会经常需要拿到数组的索引：

- 如果我们需要索引l，可以使用格式："(item,index)in数组";
- 注意上面的顺序：数组元素项item是在前面的，索引l项index是在后面的；
```javascript
<template id="my-app"> <template id="my-app">
<h2>电影列表</h2> <h2>电影列表</h2>
<ul> <ul>
<li·v-for="item in movies">{{item}}</li> <li v-for="(item,·index)·in movies">{{index}}-{{item}}</li>
</ul> </ul>
</template> </template>
```

## v-for支持的类型

v-for也支持遍历对象，并且支持有一二三个参数：

```javascript
一个参数："value in object";
二个参数："(value,key) in object";
三个参数："(value, key, index) in object";
```

Iv-for同时也支持数字的遍历：

```javascript
每一个item都是一个数字;
1v-for也可以遍历其他可选代对象(lterable)
<template id="my-app">
<h2>遍历对象</h2> <template id="my-app">
<ul> <ul>
<li·v-for="(value, key, index) in info">
<li v-for="item in 10">{{item}}</li>
{{index}}.-{{key}}.-{{value}}
</li> </ul>
</ul> </template>
</template>
```

## template元素

类似于v-if，你可以使用template元素来循环渲染一段包含多个元素的内容：

- 我们使用template来对多个元素进行包裹，而不是使用div来完成；
```javascript
<template id="my-app">
<ul>
<template v-for="(value,·key) in info">
<li>{{key}}</li>
<li>{{value}}</li>
<hr>
</template>
</ul>
</template>
```

## 数组更新检测

1Vue将被侦听的数组的变更方法进行了包裹，所以它们也将会触发视图更新。 这些被包裹过的方法包括： push) popO

```javascript
shift()
```

unshifto

```javascript
splice()
```

sorto

```javascript
reverse()
```

替换数组的方法

- 上面的方法会直接修改原来的数组；
- 但是某些方法不会替换原来的数组，而是会生成新的数组，比如 filterO、concat)和sliceO;

## v-for中的key是什么作用？

在使用v-for进行列表渲染时，我们通常会给元素或者组件绑定一个key属性。 这个key属性有什么作用呢？我们先来看一下官方的解释：

- key属性主要用在Vue的虚拟DOM算法，在新l日nodes对比时辨识VNodes;
- 如果不使用key，Vue会使用一种最大限度减少动态元素并且尽可能的尝试就地修改/复用相同类型元素的算法;
- 而使用key时，它会基于key的变化重新排列元素顺序，并且会移除/销毁key不存在的元素;
！官方的解释对于初学者来说并不好理解，比如下面的问题：

- 什么是新l日nodes，什么是VNode？
- 没有key的时候，如何尝试修改和复用的？
- 有key的时候，如何基于key重新排列的？

## 认识VNode

我们先来解释一下VNode的概念：

- 因为目前我们还没有比较完整的学习组件的概念，所以目前我们先理解HTML元素创建出来的VNode；
```javascript
VNode的全称是VirtualNode，也就是虚拟节点;
```

- 事实上，无论是组件还是元素，它们最终在Vue中表示出来的都是一个个VNode；
- VNode的本质是一个JavaScript的对象;
```javascript
<div class="title" style="font-size: 30px; color: red;">哈哈哈</div>
constvnode=
```

type: "div",

```javascript
props: {
class:."title",
style: {
```

"font-size":."30px", color:."red", template VNode 真实DOM children："哈哈哈"，

## 虚拟DOM

如果我们不只是一个简单的div，而是有一大堆的元素，那么它们应该会形成一个VNodeTree：

```javascript
<div>
<p>
<i>哈哈哈哈</i>
<i>哈哈哈哈</i>
</p>
<span>嘻嘻嘻嘻</span>
<strong>呵呵呵呵</strong>
</div>
```

虚拟DOM 真实DOM template VirtualDOM

```javascript
<div> div div
<P>
<i>哈哈哈哈</i>
<i>哈哈哈哈</i>
</p> P span strong span strong
<span>嘻嘻嘻嘻</span>
<strong>呵呵呵呵</strong>
</div>
```

## 插入F的案例

我们先来看一个案例：这个案例是当我点击按钮时会在中间插入一个f；

```javascript
<template id="my-app">
<ul> 我们可以确定的是，这次更新对于ul和button是不需要进行更新，需
<li v-for="item in letters">{{item}}</li>
</ul> 要更新的是我们的列表：
<button @click="insertF">insert f</button>
</template>
```

》在Vue中，对于相同父元素的子元素节点并不会重新渲染整个列

```javascript
<script src="../../list/vue.global.js"></script> 表;
<script>
const App
template:·'#ry-app', 因为对于列表中 a、b、c、d它们都是没有变化的;
data() {
return 在操作真实DOM的时候，我们只需要在中间插入一个f的li即可；
letters:·['a',·'b',·'c',·'d'
```

那么Vue中对于列表的更新究竟是如何操作的呢？ methods

```javascript
insertF() { 》Vue事实上会对于有key和没有key会调用两个不同的方法；
this.letters.splice(2, 0,·'f');
有key，那么就使用patchKeyedChildren方法;
没有key，那么久使用patchUnkeyedChildren方法;
Vue.createApp(App).mount('#app');
```

## Vue源码对于key的判断

renderet.ts-xue-ne-3.0.11 EXPLORER TSrenderer.ts M X

```javascript
VUE-NEXT-3.0.11 packages>runtime-core nderer [ patchChildren
```

components L686

```javascript
helpers 1687 (patchFlag>0)
apiAsyncComponent.ts 1688 (patchFLag &PatchFlags.KEYEDFRAGMENT)n[
```

TS apiComputed.ts 1689 could-be either-fulty

## sapiCreateApp.ts 如果列美中有&ey，那么会执行paichKeyedChinen方法

T5 apiDefineComponent.ts 1696 Il presence of patchFLog means children are guaranteed zo-be orra TSapilnject.ts patchKeyedchildren( es cl as VNode[], TS apiSetupHelpers.ts c2 as VNodeArrayChildren, TSapiWatch.ts 1694 container, TS component.ts M 1695 anchor, TS componentEmits.ts T5componentOptions.ts M 1696 parentComponent, TS componentProps.ts 1697 parentSuspense, TScomponentPubliclnstance.ts 1698 isSVG, TS componentRenderContext.ts 1699 slotScopeIds, TS componentRenderutis.ts 1700 optimized TS componentSlots.ts TS customFarmatter.ts 1701

```javascript
TS devtools.ts 1702 return
TS directives.ts 1703 else if(patchFLag&PatchFLags.UNKEYED_FRAGMENT)//没有key 名热行patchunkeyedchtldren
```

TS arrorHandling.ts 1784 TS featureFlags.ts 1705 patchUnkeyedchildren( 如果列表中们可以kcy，那么执行patchUnkeyedChildren方法 Tsh.ts 1706 cl as VNode[], Tshmr.ts Ts hydration.ts 1707 c2 as VNodeArrayChildren, TSindex.ts 1708 container, TS.profiling.ts 1709 anchor, renderer.ts M 1710 parentcomponent, scheduler.ts 1711 parentsuspense,

```javascript
>OUTLINE
>TIMELINE 1712 isSVG,
```

NPM SCRIPTS 1713 slotScopeIds,

## 没有key的操作(源码)

-vue-nexi-3.0.11 EXPLORER T5renderer.ts Mx VUE-NEXT-3.0.11 lpatchUnkeyedChildren

```javascript
components 1782 =>
helpers 1783 c1=c111 EMPTY_ARR
apiAsyncComponent.ts 1784 2=21 EMPTYARR
```

TsapiComputed.ts sapiCreateApp.ts 1785 111.获取旧节点的长度

```javascript
rS apiDefineComponent.ts 1786 const oldLength = cl.length
```

Ts-apilnject.ts 1787 (12.获取新节点的长度

```javascript
T'SapiLifecycle.ts 1788 const newLength = c2.length
```

TS apiSetupHelpers.ts 1789 11获取最小的孤个长度 TSapiWatch.ts

```javascript
TScomponent.ts M 1796 const commonLength =Math.min(oldLength,newLength)
T5componentEmits.ts 1791 leti
```

T5.componentOptions.ts M 1792 3：从o位善开始依次patch比较

```javascript
TScomponentProps.ts 1793 for(i=0;i<commonLength;i++)
T5componentPubliclnstance.ts 1794 const nextChild=(c2[i]=optimized
TS componentRenderContext.ts 1795 ?cloneIfMounted(c2[i] as VNode)
```

componentRenderUtils.ts

```javascript
TS componentSiots.ts 1796 ：normalizeVNode(c2[i]))
```

TS customFormatter.ts 1797 patch( Tsdevtools.ts 1807 Ts directives.ts 1808 TsarrorHanding.ts 1809 加果旧的节点数大于新的节点数 Ts featureFlags.ts

```javascript
Tshts 1810 if(oldLength>newLength){
```

Ts.hmr.ts 1811 remove:otd TShydration.ts 1812 11·移健剩余的节点 TS.index.ts 1813 unmountchildren( 75 profiling.ts 1820 ronderer.ts

```javascript
TSscheduler.ts 1821 belse{
```

OUTLINE 1822 mountnew TIMELINE 1823 1创建新的节点 NPMSCRIPTS 1824 mountchildren( vun-naxt-3.0.11

## 没有key的过程如下

我们会发现上面的diff算法效率并不高：

- c和d来说它们事实上并不需要有任何的改动；
- 但是因为我们的c被f所使用了，所有后续所有的内容都要一次进行改动，并且最后进行新增；
I旧VNode列表 a b p patch patch patch patch 新增 新VNode列表 a C d

## 有key执行操作(源码)

EXPLORER TSrenderer.ts Mx

```javascript
VUE-NEXT-3.0.11 packages>runtime-core>src>TS rendarer.ts>baseCreateRenderer>(e)patchKeyedChildren
```

components 1855 //-1.·sync-from-start helpers 1856 11从头部开始追历 TS apiAsyncComponent.ts 1从头部开始遍历，遇到相同的节点就继绩，遇到不同的就跳出循环

```javascript
TsapiComputed.ts 1857) while(i<=el&&i(=e2)[
```

TSapiCreateApp.ts 1880 TS apiDefineComponent.ts 1881 TSapinject.ts 1882 //2. sync from end T5apiLifecycle.ts 1883 11-从尾部开始遍历 2从尾部开始遍历，遇到相同的节点就继续，遇到不同的就跳出循环 T5 apiSetupHelpers.ts

```javascript
TSapiWatch.ts 1884 while(i<=el&&1<=e2){
```

TS.component.ts 1908 TScomponentEmits.ts 1909 网 TS.componentOptions.ts 1910 //-3.·common-sequence:-mount

```javascript
TS.componentProps.ts 1911 (/如果旧节点遍历完了，依然有新的节点，那么新的节点就是添加(mount) 3如果最后新节点更多，那么就添加新节点
```

TS componentPubliclnstance.ts

```javascript
T5.componentRenderContext.ts 1912 f(i>e1)
```

TS componentRenderUtis.ts 1933 TS componentSlots.ts 1934 TS customFormatter.ts 1935 /4.commonsequencet-unmount TSdevtools.ts 1936 1姐果新的节点遍历完了，还有旧的节点，那么旧的节点就是移除的 Ts directives.ts 4.如果旧节点更多，那么就移除旧节点

```javascript
TSerrorHandling.ts 1937 else if(1>e2){
```

TSfeatureFlags.ts 1942 Tsh.ts 1943 T5hmr.ts 1944 //-s.·unknown sequence Tshydration.ts 1945 1如果是位置的节点序列， T5index.ts 1946 1-如果有多余的节点，那么就移除节点 5.如果中间存在不知道如何排列的位置序列，那么就使用key建立索引图 T'S profiling.ts 1947 1之后是移动节点和挂载新节点 最大限度的使用旧节点 TSrenderer.ts M

```javascript
T5scheduler.ts 1948 else...
```

OUTLINE 2071 TIMELINE 2072

```javascript
>NPMSCRIPTS
```

master'0vu-ext-3.0.11 Prettier

## 有key的diff算法如下(一)

1第一步的操作是从头开始进行遍历、比较：

- a和b是一致的会继续进行比较;
- c和f因为key不一致，所以就会break跳出循环;
I旧VNode列表 a b patch patch break 新VNode列表 a b 旧VNode列表 a P break patch patch 新VNode列表 a a P

## 有key的diff算法如下(二)

第三步是如果旧节点遍历完毕，但是依然有新的节点，那么就新增节点： l旧VNode列表 a b d patch patch patch patch 新增 新VNode列表 a 第四步是如果新的节点遍历完毕，但是依然有旧的节点，那么就移除旧节点： 旧VNode列表 a b C d patch patch 移除c patch 新VNode列表 q d

## 有key的diff算法如下(三)

第五步是最特色的情况，中间还有很多未知的或者乱序的节点： I旧VNode列表 a b h m n d patch patch 移除i patch patch patch patch patch 新增f 新VNode列表 a n m 所以我们可以发现，Vue在进行diff算法的时候，会尽量利用我们的key来进行优化操作：

- 在没有key的时候我们的效率是非常低效的；
- 在进行插入或者重置顺序的时候，保持相同的key可以让diff算法更加的高效；
