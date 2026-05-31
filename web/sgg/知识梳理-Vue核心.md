# Vue核心知识梳理

## 第一章 Vue基础

### 1.1 模板语法与指令

Vue 模板使用 HTML 的超集语法，支持数据插值、条件渲染、列表渲染等。

**关键指令对比 (Vue2 vs Vue3):**
- `v-if` 优先级：Vue3 > v-for；Vue2 < v-for
- `v-for` 循环渲染
- `v-show` 条件显示（DOM 保留）
- `v-bind` 属性绑定
- `v-on` 事件监听

### 1.2 计算属性 (Computed)

```javascript
// Vue3 - Composition API
const getInfoCom = computed(function(){
    return "今年" + userName.value + "," + age.value + "岁了"
})

// 计算属性缓存 - 只有依赖的数据改变才重新计算
// 多次调用，函数只执行一次
```

**常见纠错 01:**
```
问题: computed vs watch 的选择
✓ computed: 处理一个数据依赖多个其他数据的场景，有缓存
✓ watch: 监听一个数据变化，需要执行副作用的场景
✗ 错误: 用 watch 替代 computed 导致缓存失效
```

### 1.3 侦听器 (Watch)

```javascript
// 基本侦听 ref
const stopWatch = watch(count,(newValue,oldValue)=> {
    console.log("watch", newValue, oldValue)
},{
    immediate:true  // 立即调用
})

// 侦听多个数据源
watch([info,count],(newValue,oldValue)=>{
    // newValue:[info更改后的值，count更改后的值]
})

// 侦听对象属性
watch(()=>info.age,(newValue,oldValue)=>{},{
    deep:true  // 深度侦听
})
```

**常见纠错 02:**
```
问题: deep:true 的代价
✓ deep:true 会遍历整个对象的所有属性，性能消耗大
✓ 尽量使用返回特定属性的函数形式: watch(()=>obj.prop, ...)
✗ 错误: 无脑使用 deep:true
```

### 1.4 watchEffect

```javascript
// 自动追踪依赖，立即执行
watchEffect(()=>{
    console.log(count.value, obj.age)  // 这些被访问的数据都会被追踪
})

// 清除副作用
watchEffect((onInvalidate)=>{
    const timer = setInterval(()=>{...}, 1000)
    onInvalidate(()=>{
        clearInterval(timer)  // 依赖变化或停止时清除
    })
})
```

**watch vs watchEffect:**
| 对比项 | watch | watchEffect |
|-----|------|-----------|
| 初次执行 | 需要 immediate:true | 自动立即执行 |
| 依赖指定 | 明确指定侦听数据 | 自动追踪访问的数据 |
| 新旧值 | 可获取 newValue/oldValue | 无法获取 |
| 清理副作用 | 需手动处理 | onInvalidate 处理 |

---

## 第二章 响应式原理

### 2.1 Vue2 - Object.defineProperty

```javascript
// 基本原理
const obj = { userName: "zhangsan" };
let value = obj.userName;

Object.defineProperty(obj, "userName", {
    get() {
        console.log("get");
        return "《" + value + "》";
    },
    set(v) {
        console.log("set", v);
        value = v;
    }
});

console.log(obj.userName);  // 《zhangsan》
obj.userName = "lisi";      // 触发 set
console.log(obj.userName);  // 《lisi》
```

**响应式依赖收集流程 - ASCII 图 1:**
```
┌─────────────────────────────────────────────┐
│         Vue2 Object.defineProperty 流程     │
├─────────────────────────────────────────────┤
│                                             │
│  data 中的属性 --> 为每个属性添加 getter   │
│       ↓                                     │
│  Dep 依赖对象 --> 收集访问该属性的 Watcher│
│       ↓                                     │
│  Watcher 收集 --> Dep.depend() 建立映射    │
│       ↓                                     │
│  属性变化 --> setter 触发 --> Dep.notify() │
│       ↓                                     │
│  通知所有 Watcher --> 重新渲染/计算        │
│                                             │
└─────────────────────────────────────────────┘
```

**常见纠错 03:**
```
问题: Vue2 数组下标修改不响应
✗ 错误: this.arr[0] = newValue  // 不会触发响应
✓ 正确: this.$set(this.arr, 0, newValue)
✓ 正确: this.arr.splice(0, 1, newValue)

原因: Object.defineProperty 只能拦截已存在的属性，
      数组下标为动态属性，无法预先定义。
```

**常见纠错 04:**
```
问题: $set 的用法
✓ $set(target, propertyName/index, value)
  - 第一个参数: 要修改的对象或数组
  - 第二个参数: 属性名或数组下标
  - 第三个参数: 新值
  
✗ 错误: $set(newValue) 参数不对
```

### 2.2 Vue3 - Proxy

```javascript
// Proxy 拦截所有操作
let obj = { userName: "zhangsan" };
const p = new Proxy(obj, {
    get(target, key, proxy) {
        console.log("proxy->get->key", key);
        return "《" + target[key] + "》";
    },
    set(target, key, newValue, proxy) {
        console.log("proxy-set->key", key);
        target[key] = newValue;
    }
});

console.log(p.userName);  // 《zhangsan》
p.userName = "lisi";      // 触发 set
```

**Vue3 Proxy 拦截方法 - ASCII 图 2:**
```
┌───────────────────────────────────────────────────────┐
│              Proxy 拦截操作流程                       │
├───────────────────────────────────────────────────────┤
│                                                       │
│  操作类型                拦截器                      │
│  ─────────────────────────────────────              │
│  读取 p.prop ──────→ get(target, key, receiver)    │
│  修改 p.prop = val ──→ set(target, key, val, receiver)
│  删除 delete p.prop ──→ deleteProperty(target, key)│
│  for...in ────────→ ownKeys(target)                │
│  Object.keys() ───→ ownKeys + getOwnPropertyDescriptor
│  new p() ──────→ construct(target, args)          │
│                                                       │
│  ↓                                                    │
│  Dep 依赖对象 --> 收集 Watcher                      │
│  ↓                                                    │
│  属性变化 --> set 拦截 --> Dep.notify()            │
│  ↓                                                    │
│  通知 Watcher --> 重新渲染                          │
│                                                       │
└───────────────────────────────────────────────────────┘
```

**Object.defineProperty vs Proxy - 对比:**
| 特性 | defineProperty | Proxy |
|------|----------------|-------|
| 拦截范围 | 单个属性 | 整个对象 |
| 新增属性 | 无法拦截 | 自动拦截 |
| 数组操作 | 需特殊处理 | 完全支持 |
| 性能 | 一对一映射 | 统一拦截 |
| 浏览器支持 | IE9+ | IE11+ |

---

## 第三章 虚拟 DOM 与 Diff

### 3.1 虚拟 DOM

虚拟 DOM 是真实 DOM 在 JavaScript 中的一个表示。

```javascript
// 虚拟 DOM 结构
{
    tag: 'div',
    props: { id: 'app', class: 'container' },
    children: [
        { tag: 'h3', children: 'Hello Vue' },
        { tag: 'p', children: 'Content' }
    ]
}
```

### 3.2 Diff 算法 - 同层比较

**虚拟 DOM Diff 同层比较 - ASCII 图 3:**
```
┌──────────────────────────────────────────────┐
│         虚拟 DOM Diff 同层比较流程           │
├──────────────────────────────────────────────┤
│                                              │
│  新旧虚拟 DOM 节点                          │
│  ↓                                          │
│  第一层: 比较根节点类型                     │
│  └─ 不同 ──→ 整个替换                      │
│  └─ 相同 ──↓                               │
│                                              │
│  第二层: 比较属性 (props)                   │
│  └─ 删除旧属性、更新新属性                 │
│                                              │
│  第三层: 比较子节点 (children)              │
│  ├─ 文本节点 ──→ 直接替换                  │
│  ├─ 元素节点 ──→ 递归 Diff                │
│  └─ key 优化 ──→ 快速定位对应节点          │
│                                              │
│  只比较同一层级，不跨级比较                │
│  (这是 Diff 的核心优化)                    │
│                                              │
└──────────────────────────────────────────────┘
```

**常见纠错 05:**
```
问题: key 的作用和误用
✓ key 用于标识节点身份，加速 Diff 算法
✗ 错误: 用 index 作为 key
  原因: 数组元素位置变化时，index 仍相同，
        导致复用错误的 DOM，引发状态混乱
  
✓ 正确: 用 id 或其他唯一标识符作为 key
```

---

## 第四章 ref 与 reactive

### 4.1 ref - 基本类型响应式

```javascript
const count = ref(100);
console.log(count.value);  // script 中需要 .value

// 在模板中自动解包
{{ count }}  // 无需写 .value

// ref 包裹对象时，内部使用 reactive
let obj = ref({ userName: "zhangsan" })
// obj.value 是一个 Proxy 实例
```

### 4.2 reactive - 引用类型响应式

```javascript
const obj = reactive({
    userName: "zhangsan",
    age: 12
})

// 模板中直接使用
{{ obj.userName }}
{{ obj.age }}

// script 中也不需要 .value
obj.userName = "lisi"
```

### 4.3 ref 与 reactive 的区别

| 对比项 | ref | reactive |
|-----|-----|---------|
| 适用数据类型 | 基本类型、对象 | 对象、数组 |
| 模板访问 | 自动解包，无需 .value | 直接访问 |
| Script 访问 | 需要 .value | 无需 .value |
| 响应原理 | defineProperty (基本) + Proxy (对象) | Proxy |
| 赋值更新 | 可直接赋值 value | 改变引用无效 |

**常见纠错 06:**
```
问题: ref vs reactive 解包规则
✗ 错误: const { userName } = reactive(obj)
       解构后失去响应性

✓ 正确: const obj = reactive({ userName })
✓ 正确: const { userName } = toRefs(obj)
✓ 正确: 使用 .value: const userName = ref('')

原因: 解构会打破响应式代理关系
```

### 4.4 ref 与 DOM 元素结合

```vue
<template>
    <h3 ref="h3Ref">ref</h3>
    <input value="100" ref="userName" type="text">
    <p ref="pRef" v-for="item in arr" :key="item">{{item}}</p>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";

const userName = ref<HTMLInputElement | number>(1);
const h3Ref = ref();
const pRef = ref();

onMounted(()=>{
    // ref 与 DOM 结合: value 为真实 DOM 元素
    console.log(userName.value === document.querySelector("input"));
    
    // ref 与 v-for 结合: value 为 DOM 数组
    pRef.value.forEach(item => {
        item.style.color = "red";
    })
})
</script>
```

### 4.5 ref 与组件结合

```vue
<template>
    <button @click="childRef.setCount(200)">更改子组件中的 count</button>
    <Child ref="childRef"/>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import Child from "./components/Child.vue";

const childRef = ref();

onMounted(function(){
    // ref 与组件结合: value 为组件 proxy 实例
    console.log(childRef.value);
})
</script>
```

**子组件必须暴露:**
```vue
<script setup lang="ts">
let count = ref(100);
let setCount = function(n: number){
    count.value = n;
}
defineExpose({ count, setCount })  // 必须显式暴露
</script>
```

---

## 第五章 生命周期

### 5.1 生命周期钩子时序图 - ASCII 图 4:

```
┌─────────────────────────────────────────────┐
│        Vue3 生命周期执行顺序                │
├─────────────────────────────────────────────┤
│                                             │
│  ① setup() 函数执行 (或 <script setup>)  │
│     └─ 在 beforeCreate 之前执行            │
│                                             │
│  ② beforeCreate (Vue2 兼容)               │
│                                             │
│  ③ created (Vue2 兼容)                    │
│                                             │
│  ④ beforeMount --> onBeforeMount          │
│     └─ 响应式状态已设置，DOM 未创建       │
│                                             │
│  ⑤ DOM 渲染 (虚拟 DOM → 真实 DOM)      │
│                                             │
│  ⑥ mounted --> onMounted                  │
│     └─ 组件挂载完成，可访问 DOM           │
│                                             │
│  ─────────────────────────────────         │
│  ※ 数据变化时 (更新阶段)                  │
│  ─────────────────────────────────         │
│                                             │
│  ⑦ beforeUpdate --> onBeforeUpdate        │
│     └─ 即将因响应式状态变更而更新 DOM    │
│                                             │
│  ⑧ DOM 更新                               │
│                                             │
│  ⑨ updated --> onUpdated                  │
│     └─ DOM 已更新                          │
│                                             │
│  ─────────────────────────────────         │
│  ※ 组件销毁时                             │
│  ─────────────────────────────────         │
│                                             │
│  ⑩ beforeUnmount --> onBeforeUnmount      │
│     └─ 组件实例功能完整，即将卸载        │
│                                             │
│  ⑪ unmount --> onUnmounted                │
│     └─ 组件已卸载，事件监听已移除         │
│                                             │
└─────────────────────────────────────────────┘
```

### 5.2 Composition API 生命周期

```javascript
import {
    onBeforeMount,
    onMounted,
    onBeforeUpdate,
    onUpdated,
    onBeforeUnmount,
    onUnmounted
} from 'vue';

onBeforeMount(() => {
    console.log("组件挂载前");
})

onMounted(() => {
    console.log("组件挂载后，可操作 DOM");
})

onBeforeUpdate(() => {
    console.log("更新前，访问最新的响应式数据");
})

onUpdated(() => {
    console.log("更新后，可访问更新后的 DOM");
})

onBeforeUnmount(() => {
    console.log("卸载前");
})

onUnmounted(() => {
    console.log("卸载后，清理资源");
})
```

---

## 第六章 插槽 (Slot)

### 6.1 基本插槽

```vue
<!-- 父组件 -->
<Child>
    <h3>这是插槽内容</h3>
</Child>

<!-- 子组件 -->
<template>
    <div>
        <h2>子组件标题</h2>
        <slot></slot>  <!-- 插槽占位符 -->
    </div>
</template>
```

### 6.2 具名插槽

```vue
<!-- 父组件 -->
<Child>
    <template #header>
        <h3>头部内容</h3>
    </template>
    <template #default>
        <p>默认内容</p>
    </template>
    <template #footer>
        <footer>底部内容</footer>
    </template>
</Child>

<!-- 子组件 -->
<template>
    <div>
        <slot name="header"></slot>
        <slot></slot>
        <slot name="footer"></slot>
    </div>
</template>
```

### 6.3 作用域插槽

```vue
<!-- 父组件 - 可访问子组件数据 -->
<Child :info="info">
    <template #default="{ suibian, $index }">
        <p :style="{ background: suibian.done ? 'green' : 'red' }">
            {{ $index }}: {{ suibian.do }}
        </p>
    </template>
</Child>

<!-- 子组件 - 通过 slot 属性传递数据 -->
<template>
    <div v-for="(item, index) in info" :key="item.id">
        <slot :suibian="item" :$index="index"></slot>
    </div>
</template>
```

---

## 第七章 自定义指令、过滤器、Mixin

### 7.1 自定义指令

```javascript
// 全局指令
app.directive('focus', {
    mounted(el) {
        el.focus();
    }
})

// 局部指令
const vFocus = {
    mounted(el) {
        el.focus();
    }
}

// 使用
<input v-focus />
```

### 7.2 过滤器

```javascript
// 全局过滤器
app.config.globalProperties.$filters = {
    capitalize: (value) => {
        if (!value) return ''
        value = value.toString()
        return value.charAt(0).toUpperCase() + value.slice(1)
    }
}

// 使用
{{ message | capitalize }}
```

### 7.3 Mixin

```javascript
// mixin 对象
const myMixin = {
    data() {
        return { message: 'Hello' }
    },
    methods: {
        sayHello() {
            console.log(this.message);
        }
    }
}

// 组件中使用
export default {
    mixins: [myMixin],
    // 组件自身属性会覆盖 mixin 中同名属性
}
```

---

## 第八章 渲染函数与 JSX

### 8.1 渲染函数 (h 函数)

```javascript
import { h } from 'vue';

export default {
    render() {
        return h('div', { class: 'container' }, [
            h('h3', 'Title'),
            h('p', 'Content')
        ])
    }
}
```

### 8.2 JSX 语法

```jsx
// 需要配置 @vitejs/plugin-vue-jsx

export default {
    render() {
        return (
            <div class="container">
                <h3>Title</h3>
                <p>Content</p>
            </div>
        )
    }
}
```

---

## 第九章 Vue3 Composition API

### 9.1 setup 函数

```javascript
export default {
    setup(props, context) {
        // props: 父组件传递的属性
        // context.emit: 触发自定义事件
        // context.slots: 插槽对象
        // context.attrs: 未被 props 接收的属性
        
        return {
            // 返回的对象中的属性在模板中可直接使用
        }
    }
}
```

### 9.2 setup 关键要点

```javascript
// 1. setup 在 beforeCreate 之前执行
// 2. setup 中 this 指向 undefined
// 3. setup 不能是 async 函数 (除非使用 <Suspense>)
// 4. setup 返回的对象属性优先级高于 data()
// 5. 返回对象的属性可在模板中直接使用
```

### 9.3 `<script setup>` 语法糖

```vue
<script setup lang="ts">
// 此处的代码等价于在 setup() 中编写
import { ref, computed } from 'vue'

const count = ref(0)

const doubled = computed(() => count.value * 2)

// 变量、函数直接暴露给模板，无需 return
</script>

<template>
    <div>
        <p>{{ count }}</p>
        <p>{{ doubled }}</p>
    </div>
</template>
```

---

## 第十章 组件通信全景图 - ASCII 图 5:

```
┌──────────────────────────────────────────────────────────┐
│           Vue 组件通信全景图                             │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ✓ Props (父 → 子)                                      │
│    └─ 单向数据流，子组件只读                           │
│                                                          │
│  ✓ Emit + defineEmits (子 → 父)                        │
│    └─ 触发自定义事件，父组件监听                       │
│                                                          │
│  ✓ v-model (父 ↔ 子)                                   │
│    └─ :modelValue + @update:modelValue 的语法糖      │
│                                                          │
│  ✓ Provide/Inject (祖先 → 后代)                       │
│    └─ 跨级数据传递，适合深层嵌套                      │
│                                                          │
│  ✓ $ref/$parent (直接访问)                            │
│    └─ 获取组件实例，不推荐过度使用                    │
│                                                          │
│  ✓ EventBus (任意组件通信)                            │
│    └─ 使用 mitt 库实现，发布-订阅模式                │
│                                                          │
│  ✓ Pinia/Vuex (全局状态管理)                          │
│    └─ 集中管理共享状态                                │
│                                                          │
│  ✓ $attrs (属性透传)                                  │
│    └─ 传递未被 props 接收的属性和事件                │
│                                                          │
│  ✓ useAttrs (Composition API 版本)                   │
│    └─ 在 setup 中访问 attrs                          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### 10.1 Props 和 Emit

```vue
<!-- 父组件 -->
<template>
    <Child :count="count" @setCount="setCount" />
</template>

<script setup lang="ts">
import { ref } from 'vue';
import Child from './Child.vue';

const count = ref(0);
const setCount = (n) => count.value += n;
</script>

<!-- 子组件 -->
<template>
    <button @click="$emit('setCount', 2)">Increase</button>
</template>

<script setup lang="ts">
defineProps(['count']);
defineEmits(['setCount']);
</script>
```

### 10.2 Provide/Inject

```vue
<!-- 祖先组件 -->
<script setup lang="ts">
import { ref, provide } from 'vue';

const token = ref("appToken");
provide("t", token);  // 提供数据
</script>

<!-- 后代组件 (任意层级) -->
<script setup lang="ts">
import { inject } from 'vue';

const t = inject("t");  // 注入数据
</script>
```

### 10.3 EventBus (mitt)

```javascript
// utils/index.ts
import mitt from "mitt";
export const bus = mitt();

// 组件A - 发送事件
import { bus } from "@/utils";
bus.emit("changenum", 200);

// 组件B - 监听事件
import { onMounted } from "vue";
import { bus } from "@/utils";

onMounted(() => {
    bus.on("changenum", (value) => {
        console.log("接收到:", value);
        bus.off("changenum");  // 取消监听
    })
})
```

---

## 第十一章 nextTick

`nextTick()` 等待 DOM 更新完成后执行回调。

```javascript
import { ref, nextTick } from 'vue';

const count = ref(1);

const changeNum = async () => {
    count.value++;
    
    // 方案 1: 回调函数
    nextTick(() => {
        console.log(document.querySelector('div').innerText);
    })
    
    // 方案 2: Promise
    nextTick().then(() => {
        console.log(document.querySelector('div').innerText);
    })
    
    // 方案 3: async/await
    await nextTick();
    console.log(document.querySelector('div').innerText);
}
```

**应用场景:**
- 更新 DOM 后立即操作 (设置样式、获取高度等)
- 获取更新后的真实 DOM 内容
- 在数据变化后需要同步操作 DOM

---

## 易错速查

| 编号 | 错误 | 正确做法 | 原因 |
|-----|------|--------|------|
| 01 | 用 watch 替代 computed | computed 有缓存机制 | computed 性能更好 |
| 02 | 无脑使用 deep:true | 针对性使用，监听单个属性 | deep:true 性能消耗大 |
| 03 | 数组下标直接赋值 | 使用 $set 或 splice | defineProperty 无法拦截动态属性 |
| 04 | $set 参数错误 | $set(obj, key, value) | 必须三个参数 |
| 05 | 用 index 作为 key | 用 id 或唯一标识符 | index 改变会导致 DOM 复用错误 |
| 06 | ref 解构失去响应性 | 使用 toRefs() 或 .value | 解构打破代理关系 |

---

**文档生成日期: 2026-05-25**
