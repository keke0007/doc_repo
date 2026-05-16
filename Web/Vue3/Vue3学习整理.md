# Vue3 学习整理

这份笔记基于 `Vue3` 目录下的 21 份 Markdown 归纳整理，目标不是简单拼接原文，而是按 Vue3 的学习路径重新组织内容，并给每个核心知识点补上最小可运行示例，方便你系统学习和回顾。

## 学习路线

建议按下面的顺序学习：

1. 先理解 Vue 是什么、怎么引入、声明式开发是什么
2. 再掌握模板语法、指令、事件、列表渲染
3. 接着学习 Options API、`v-model`
4. 然后进入组件化、组件通信、插槽、生命周期
5. 再学习 Composition API、`script setup`
6. 然后补 Vue3 高级特性、动画、响应式原理
7. 最后学习 Router、状态管理、axios、项目打包部署

## 原始资料对应关系

| 主题 | 原文件 |
|---|---|
| Vue 入门 | `01_邂逅Vue.js开发.md` |
| 模板语法与指令 | `02_`、`03_`、`04_`、`05_` |
| 组件化与通信 | `06_`、`07_`、`08_`、`09_` |
| Composition API | `10_`、`11_` |
| Vue3 高级特性 | `12_`、`18_`、`19_`、`20_` |
| Vue Router | `13_` |
| Vuex / Pinia | `14_`、`15_` |
| axios / 项目实战 / 部署 | `16_`、`17_`、`21_` |

## 一、认识 Vue3

应用场景：

- 用在建立 Vue 项目的整体认知，知道它适合开发什么类型的前端应用。
- 适合后台管理系统、内容展示站点、商城前台、移动端 H5 等组件化项目。

### 1. Vue 是什么

Vue 是一套用于构建用户界面的渐进式 JavaScript 框架。

可以先记住三个关键词：

- 声明式
- 组件化
- 渐进式

### 2. 为什么学 Vue3

- Vue3 是 Vue 生态的现代主线
- 性能和 Tree Shaking 更好
- Composition API 更适合复杂逻辑复用
- 生态已经比较成熟，Router、Pinia、Vite 都很常用

### 3. 如何引入 Vue

最简单的方式是通过 CDN。

```html
<div id="app"></div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
  const App = {
    template: `<h2>Hello Vue3</h2>`
  }

  Vue.createApp(App).mount("#app")
</script>
```

### 4. 声明式开发

原生 JS 往往更偏命令式，Vue 更偏声明式。

```html
<div id="app">
  <h2>{{ message }}</h2>
  <button @click="changeMessage">修改文本</button>
</div>

<script>
  Vue.createApp({
    data() {
      return {
        message: "Hello Vue3"
      }
    },
    methods: {
      changeMessage() {
        this.message = "你好，Vue3"
      }
    }
  }).mount("#app")
</script>
```

## 二、模板语法与基础指令

应用场景：

- 用在把数据渲染到页面、绑定属性、绑定事件、控制显示隐藏和循环输出列表时。
- 适合商品列表、表格、详情页、按钮联动这类最常见的页面开发。

### 1. Mustache 插值

```html
<div id="app">
  <h2>{{ title }}</h2>
  <p>{{ count + 1 }}</p>
</div>
```

### 2. `v-bind`

动态绑定属性。

```html
<div id="app">
  <img :src="imgUrl" :alt="title" />
</div>

<script>
  Vue.createApp({
    data() {
      return {
        title: "图片",
        imgUrl: "https://via.placeholder.com/120"
      }
    }
  }).mount("#app")
</script>
```

### 3. 绑定 class

对象语法：

```html
<div :class="{ active: isActive, danger: isDanger }">内容</div>
```

数组语法：

```html
<div :class="['card', sizeClass]">内容</div>
```

### 4. 绑定 style

```html
<div :style="{ color: textColor, fontSize: fontSize + 'px' }">
  动态样式
</div>
```

### 5. `v-on`

绑定事件，简写为 `@`。

```html
<button @click="increment">+1</button>
```

```js
methods: {
  increment() {
    this.count++
  }
}
```

### 6. 事件传参

```html
<button @click="add(5)">增加 5</button>
```

```js
methods: {
  add(num) {
    this.count += num
  }
}
```

### 7. 事件修饰符

```html
<a href="https://example.com" @click.prevent="handleClick">链接</a>
<div @click.stop="innerClick">阻止冒泡</div>
```

### 8. `v-if` 和 `v-show`

```html
<p v-if="isShow">v-if 控制显示</p>
<p v-show="isShow">v-show 控制显示</p>
```

区别：

- `v-if` 是条件渲染，切换有销毁/重建
- `v-show` 只是切换 `display`

### 9. `v-for`

```html
<ul>
  <li v-for="(item, index) in books" :key="item.id">
    {{ index + 1 }} - {{ item.name }}
  </li>
</ul>
```

```js
data() {
  return {
    books: [
      { id: 1, name: "Vue3" },
      { id: 2, name: "JavaScript" }
    ]
  }
}
```

### 10. 为什么要写 key

`key` 能帮助 Vue 更准确地识别节点变化，优化 diff。

建议：

- 列表渲染尽量使用稳定唯一值
- 不要随手用索引当长期 key

## 三、Options API

应用场景：

- 用在中小型 Vue 页面里组织 `data`、`methods`、`computed`、`watch` 这类经典结构。
- 适合学习 Vue 基础、维护老项目、快速写业务页和管理后台表单页。

### 1. `data`

```js
data() {
  return {
    title: "Vue 学习",
    count: 0
  }
}
```

### 2. `methods`

```js
methods: {
  increment() {
    this.count++
  },
  decrement() {
    this.count--
  }
}
```

### 3. `computed`

适合根据已有数据派生新值。

```html
<h2>{{ fullName }}</h2>
```

```js
data() {
  return {
    firstName: "Tom",
    lastName: "Lee"
  }
},
computed: {
  fullName() {
    return this.firstName + " " + this.lastName
  }
}
```

### 4. `computed` vs `methods`

- `computed` 有缓存
- `methods` 每次调用都会重新执行

### 5. `watch`

```js
watch: {
  keyword(newValue, oldValue) {
    console.log("keyword changed:", oldValue, "->", newValue)
  }
}
```

对象写法：

```js
watch: {
  info: {
    handler(newValue) {
      console.log(newValue)
    },
    deep: true,
    immediate: true
  }
}
```

## 四、v-model 与表单

应用场景：

- 用在登录框、搜索框、筛选器、资料编辑页、设置页等需要双向绑定输入的地方。
- 适合处理文本框、单选框、多选框、下拉框和表单修饰符联动。

### 1. 输入框双向绑定

```html
<input v-model="message" />
<p>{{ message }}</p>
```

### 2. textarea

```html
<textarea v-model="content"></textarea>
```

### 3. checkbox

单个：

```html
<input type="checkbox" v-model="isAgree" />
```

多个：

```html
<label><input type="checkbox" value="篮球" v-model="hobbies" />篮球</label>
<label><input type="checkbox" value="足球" v-model="hobbies" />足球</label>
```

### 4. radio

```html
<label><input type="radio" value="男" v-model="gender" />男</label>
<label><input type="radio" value="女" v-model="gender" />女</label>
```

### 5. select

```html
<select v-model="city">
  <option value="shanghai">上海</option>
  <option value="beijing">北京</option>
</select>
```

### 6. 修饰符

```html
<input v-model.lazy="username" />
<input v-model.number="age" />
<input v-model.trim="keyword" />
```

## 五、组件化开发

应用场景：

- 用在把页面拆成导航栏、卡片、弹窗、列表项、表单块等可复用模块时。
- 适合团队协作开发，减少重复代码，让页面结构更清晰。

### 1. 认识组件

组件就是可复用的 UI 片段。

### 2. 全局组件

```js
const App = {
  template: `
    <div>
      <my-button></my-button>
    </div>
  `
}

const app = Vue.createApp(App)

app.component("my-button", {
  template: `<button>全局按钮</button>`
})

app.mount("#app")
```

### 3. 局部组件

```js
const HelloWorld = {
  template: `<h2>Hello World</h2>`
}

const App = {
  components: {
    HelloWorld
  },
  template: `<HelloWorld />`
}
```

### 4. 单文件组件 SFC

```vue
<template>
  <h2>{{ title }}</h2>
</template>

<script>
export default {
  data() {
    return {
      title: "SFC 组件"
    }
  }
}
</script>

<style scoped>
h2 {
  color: #42b883;
}
</style>
```

## 六、组件通信

应用场景：

- 用在父子组件之间传递数据、通知事件、共享依赖时。
- 适合商品卡片点击详情、弹窗开关控制、表单子项回传、全局主题配置下发等场景。

先记住一句话：

- 父传子：父组件把数据交给子组件。
- 子传父：子组件把“发生了什么”通知给父组件。

如果你把组件通信想成人和人说话：

- `props` 像“父组件下发信息”。
- `emit` 像“子组件向上汇报”。

这两种是 Vue 里最核心、最常见、最应该先掌握的通信方式。

## 父传子和子传父的核心区别

| 方式 | 谁发起 | 传什么 | 关键词 | 常见用途 |
|---|---|---|---|---|
| 父传子 | 父组件 | 数据 | `props` | 标题、列表、用户信息、开关状态 |
| 子传父 | 子组件 | 事件或结果 | `$emit` / `emits` | 按钮点击、输入变化、删除确认、提交结果 |

可以这样记：

- 父传子传的是“值”。
- 子传父传的是“消息”和“结果”。

再说得更直白一点：

- 父组件决定“给你什么”。
- 子组件决定“告诉你我做了什么”。

## 一、父传子 props

最常见的实现方式：

- 父组件在子组件标签上绑定属性。
- 子组件通过 `props` 接收。
- 数据流方向是单向的，从父到子。

### 1. 基本写法

子组件：

```vue
<template>
  <h3>{{ title }}</h3>
</template>

<script>
export default {
  props: {
    title: String
  }
}
</script>
```

父组件：

```vue
<Child title="Vue3 组件通信" />
```

也可以传动态值：

```vue
<Child :title="pageTitle" :count="count" />
```

```js
export default {
  data() {
    return {
      pageTitle: "商品列表",
      count: 10
    }
  }
}
```

特点：

- 写法简单，最适合传展示数据和配置项。
- 子组件接收到数据后可以使用，但不应该直接修改父组件传进来的 `props`。

适用场景：

- 页面标题传给 Header 组件
- 商品列表传给 ProductList 组件
- 用户信息传给 UserCard 组件
- 按钮类型、颜色、尺寸传给通用 Button 组件

案例 1：父组件把标题传给子组件

```vue
<!-- Child.vue -->
<template>
  <h2>{{ title }}</h2>
</template>

<script>
export default {
  props: {
    title: String
  }
}
</script>
```

```vue
<!-- Parent.vue -->
<template>
  <Child title="首页标题" />
</template>
```

案例 2：父组件把商品数组传给列表组件

```vue
<!-- ProductList.vue -->
<template>
  <ul>
    <li v-for="item in products" :key="item.id">
      {{ item.name }} - ￥{{ item.price }}
    </li>
  </ul>
</template>

<script>
export default {
  props: {
    products: {
      type: Array,
      default() {
        return []
      }
    }
  }
}
</script>
```

```vue
<!-- Parent.vue -->
<template>
  <ProductList :products="goods" />
</template>

<script>
export default {
  data() {
    return {
      goods: [
        { id: 1, name: "键盘", price: 199 },
        { id: 2, name: "鼠标", price: 99 }
      ]
    }
  }
}
</script>
```

案例 3：父组件控制弹窗显示文本

```vue
<!-- ConfirmBox.vue -->
<template>
  <div class="box">
    <h3>{{ title }}</h3>
    <p>{{ message }}</p>
  </div>
</template>

<script>
export default {
  props: {
    title: String,
    message: String
  }
}
</script>
```

```vue
<ConfirmBox
  title="删除提示"
  message="删除后将无法恢复"
/>
```

案例 4：父组件把布尔状态传给子组件

```vue
<!-- LoadingMask.vue -->
<template>
  <div v-if="loading" class="mask">加载中...</div>
</template>

<script>
export default {
  props: {
    loading: Boolean
  }
}
</script>
```

```vue
<LoadingMask :loading="isLoading" />
```

### 2. props 的几个关键规则

1. `props` 是单向数据流。
   父组件变，子组件会跟着变。

2. 子组件不要直接修改 `props`。
   如果子组件真要改，通常应该：
   - 复制成自己的本地数据
   - 或通过 `emit` 通知父组件改

3. `props` 更适合传“数据”和“配置”，不适合让子组件直接控制父组件状态。

### 3. 为什么不能直接改 props

错误思路：

```js
this.title = "新标题"
```

这样做的问题是：

- `title` 的真正来源是父组件。
- 子组件直接改它，会破坏数据来源的清晰性。
- Vue 也会给出相关警告。

正确思路通常是：

- 如果只是展示，直接用 `props`。
- 如果子组件要改，通知父组件去改。

## 二、子传父 emits

最常见的实现方式：

- 子组件内部触发一个事件。
- 父组件监听这个事件。
- 如果需要，还可以顺带把数据一起传出去。

### 1. 基本写法

子组件：

```vue
<template>
  <button @click="sendData">发送数据</button>
</template>

<script>
export default {
  emits: ["add"],
  methods: {
    sendData() {
      this.$emit("add", 10)
    }
  }
}
</script>
```

父组件：

```vue
<Child @add="handleAdd" />
```

父组件方法通常会接收子组件传过来的参数：

```vue
<template>
  <Child @add="handleAdd" />
</template>

<script>
export default {
  methods: {
    handleAdd(num) {
      console.log("子组件传来的值：", num)
    }
  }
}
</script>
```

特点：

- 子组件不直接修改父组件，而是发消息给父组件。
- 这种方式非常符合组件职责划分，子组件专注触发行为，父组件决定怎么处理。

适用场景：

- 点击删除按钮，通知父组件删除当前项
- 表单组件输入完成，把值同步给父组件
- 弹窗点击确认，把结果交给父组件
- 子组件某个操作结束后，通知父组件刷新列表

案例 1：点击按钮，子组件通知父组件加 1

```vue
<!-- CounterButton.vue -->
<template>
  <button @click="handleClick">+1</button>
</template>

<script>
export default {
  emits: ["increment"],
  methods: {
    handleClick() {
      this.$emit("increment", 1)
    }
  }
}
</script>
```

```vue
<!-- Parent.vue -->
<template>
  <h3>当前数量：{{ count }}</h3>
  <CounterButton @increment="addCount" />
</template>

<script>
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    addCount(step) {
      this.count += step
    }
  }
}
</script>
```

案例 2：子组件把输入框内容传给父组件

```vue
<!-- SearchInput.vue -->
<template>
  <input
    :value="modelValue"
    @input="handleInput"
    placeholder="请输入关键字"
  />
</template>

<script>
export default {
  props: {
    modelValue: String
  },
  emits: ["update:modelValue"],
  methods: {
    handleInput(event) {
      this.$emit("update:modelValue", event.target.value)
    }
  }
}
</script>
```

```vue
<!-- Parent.vue -->
<template>
  <SearchInput v-model="keyword" />
  <p>当前搜索词：{{ keyword }}</p>
</template>
```

这个例子本质上也是子传父，只是借助了 `v-model` 的语法糖。

案例 3：商品卡片点击删除，通知父组件

```vue
<!-- ProductItem.vue -->
<template>
  <div>
    <span>{{ product.name }}</span>
    <button @click="$emit('remove', product.id)">删除</button>
  </div>
</template>

<script>
export default {
  props: {
    product: Object
  },
  emits: ["remove"]
}
</script>
```

```vue
<!-- Parent.vue -->
<template>
  <ProductItem
    v-for="item in goods"
    :key="item.id"
    :product="item"
    @remove="handleRemove"
  />
</template>

<script>
export default {
  data() {
    return {
      goods: [
        { id: 1, name: "键盘" },
        { id: 2, name: "鼠标" }
      ]
    }
  },
  methods: {
    handleRemove(id) {
      this.goods = this.goods.filter(item => item.id !== id)
    }
  }
}
</script>
```

案例 4：弹窗确认后，子组件把结果告诉父组件

```vue
<!-- ConfirmDialog.vue -->
<template>
  <div>
    <button @click="$emit('confirm')">确认</button>
    <button @click="$emit('cancel')">取消</button>
  </div>
</template>

<script>
export default {
  emits: ["confirm", "cancel"]
}
</script>
```

```vue
<ConfirmDialog
  @confirm="handleConfirm"
  @cancel="handleCancel"
/>
```

### 2. 子传父的本质

子传父并不是把父组件的数据“直接改掉”，而是：

- 子组件触发事件
- 父组件接收事件
- 父组件决定是否更新自己的数据

这点非常重要，因为它保证了数据来源清晰，组件职责清晰。

## 三、父传子和子传父怎么选

你可以用一个很实用的判断方法：

1. 我要把数据展示给子组件吗？
   用父传子 `props`。

2. 我要让子组件告诉我“用户做了什么”吗？
   用子传父 `emit`。

3. 子组件需要既拿到数据，又把变化回传吗？
   通常是 `props + emit` 组合使用。

最典型的组合场景就是：

- 父组件传一个值进去
- 子组件操作后通过 `emit` 把新值发回来

这也是很多表单组件和 `v-model` 的底层思路。

## 四、父传子和子传父的组合案例

案例 1：输入框双向协作

- 父组件传 `modelValue`
- 子组件触发 `update:modelValue`

这本质上就是：

- 父传子负责显示旧值
- 子传父负责同步新值

案例 2：分页组件

```vue
<!-- Pager.vue -->
<template>
  <div>
    <button @click="$emit('change', currentPage - 1)">上一页</button>
    <span>第 {{ currentPage }} 页</span>
    <button @click="$emit('change', currentPage + 1)">下一页</button>
  </div>
</template>

<script>
export default {
  props: {
    currentPage: Number
  },
  emits: ["change"]
}
</script>
```

```vue
<!-- Parent.vue -->
<template>
  <Pager :current-page="page" @change="handlePageChange" />
</template>

<script>
export default {
  data() {
    return {
      page: 1
    }
  },
  methods: {
    handlePageChange(newPage) {
      if (newPage < 1) return
      this.page = newPage
    }
  }
}
</script>
```

这个场景特别经典：

- 当前页码来自父组件
- 页码变化动作由子组件触发
- 真正修改页码的权力仍然在父组件

案例 3：评分组件

```vue
<!-- RateStar.vue -->
<template>
  <div>
    <button @click="$emit('update:score', 1)">1分</button>
    <button @click="$emit('update:score', 2)">2分</button>
    <button @click="$emit('update:score', 3)">3分</button>
  </div>
</template>

<script>
export default {
  props: {
    score: Number
  },
  emits: ["update:score"]
}
</script>
```

```vue
<RateStar :score="score" @update:score="score = $event" />
```

## 五、除了 props 和 emits，还要知道什么

### 1. 非 prop attribute

```vue
<MyInput class="large" placeholder="请输入内容" />
```

如果组件没有声明这些 props，它们可能会透传到根元素。

适用场景：

- 给组件根节点追加 `class`
- 传原生 `placeholder`
- 追加 `id`、`style`、`disabled` 这类原生属性

它不属于严格意义上的“主要通信方式”，更像是组件包装时的补充能力。

### 2. provide / inject

父组件：

```js
provide() {
  return {
    theme: "dark"
  }
}
```

子组件：

```js
inject: ["theme"]
```

适用场景：

- 祖先组件给很深层的后代传主题、语言、配置项
- 组件库内部传递表单上下文、布局上下文

注意：

- 它更适合跨层级依赖注入。
- 如果只是普通父子传值，优先还是用 `props` 和 `emit`，因为更直观。

## 六、常见误区

1. 不要把 `props` 和 `emit` 当成双向随便乱改。
   Vue 推荐的是单向数据流，父传子，子通知父。

2. 不要在子组件里直接改 `props`。
   这会让数据来源混乱。

3. 不要本来只是父子通信，却一上来就用 `provide / inject`。
   会让代码绕远，降低可读性。

4. 不要把“传数据”和“传模板”混在一起。
   传数据主要是 `props`，传模板主要是 `slot`。

## 七、一个最终理解

你可以把父子组件通信想成这一套分工：

- 父组件负责提供数据和状态。
- 子组件负责展示、交互和触发事件。
- 父组件最后决定要不要更新数据。

所以最常见的模型就是：

- `props` 向下传值
- `emit` 向上传递结果

这套模型一旦吃透，后面的 `v-model`、表单组件、分页组件、弹窗组件都会变得非常好理解。

## 七、插槽 Slot

应用场景：

- 用在封装通用组件同时保留内容扩展能力，比如弹窗、表格列、自定义卡片头尾部。
- 适合 UI 组件库和业务组件里做结构复用但内容不固定的场景。

先记住一句话：

- `props` 是父组件把“数据”传给子组件。
- `slot` 是父组件把“模板内容”传给子组件。

如果你发现一个组件的外壳很固定，但内部某一块内容希望由外部决定，这时候往往就该考虑插槽。

三种插槽的核心区别：

| 类型 | 解决的问题 | 数据来自谁 | 适合场景 |
|---|---|---|---|
| 默认插槽 | 只有一块内容要插进去 | 父组件提供内容 | 卡片、弹窗正文、按钮文本 |
| 具名插槽 | 有多块不同位置内容要插进去 | 父组件提供内容 | 页面布局、卡片头部/底部、弹窗头尾 |
| 作用域插槽 | 子组件既要暴露数据，又要让父组件决定怎么渲染 | 子组件提供数据，父组件决定模板 | 列表项渲染、表格列渲染、下拉项自定义 |

可以这样快速区分：

- 默认插槽：只有“放内容进来”这一个需求。
- 具名插槽：不止一个坑位，要精确指定放到哪里。
- 作用域插槽：子组件手里有数据，但渲染方式想交给父组件。

### 1. 默认插槽

特点：

- 子组件只预留一个内容入口。
- 父组件不写名字，直接把内容包在组件标签内部。
- 最适合最简单的“内容填充”场景。

子组件：

```vue
<template>
  <div class="card">
    <slot>默认内容</slot>
  </div>
</template>
```

父组件：

```vue
<MyCard>
  <h3>这是插入的内容</h3>
</MyCard>
```

如果父组件没有传内容，就会显示子组件里写好的“默认内容”。

适用场景：

- 通用卡片组件的正文区域
- 弹窗中间的说明内容
- 按钮内部文字或图标
- 折叠面板的主要展示内容

案例 1：通用卡片正文

```vue
<!-- BaseCard.vue -->
<template>
  <div class="card">
    <slot>暂无内容</slot>
  </div>
</template>
```

```vue
<BaseCard>
  <p>这是文章摘要</p>
</BaseCard>
```

案例 2：弹窗正文

```vue
<!-- BaseDialog.vue -->
<template>
  <div class="dialog">
    <h3>提示</h3>
    <div class="dialog-body">
      <slot>默认提示内容</slot>
    </div>
  </div>
</template>
```

```vue
<BaseDialog>
  <p>确定要删除这条记录吗？</p>
</BaseDialog>
```

案例 3：按钮内容扩展

```vue
<!-- BaseButton.vue -->
<template>
  <button class="btn">
    <slot>提交</slot>
  </button>
</template>
```

```vue
<BaseButton>
  <span>保存修改</span>
</BaseButton>
```

什么时候不要只用默认插槽：

- 当组件内部不止一个位置需要外部插入内容时，不要继续硬塞默认插槽，要改成具名插槽。

### 2. 具名插槽

特点：

- 子组件里有多个插槽位置，每个位置有自己的名字。
- 父组件通过 `#名字` 或 `v-slot:名字` 精确指定内容放到哪里。
- 适合“组件结构固定，但每块区域内容不同”的情况。

子组件：

```vue
<template>
  <header><slot name="header"></slot></header>
  <main><slot></slot></main>
</template>
```

父组件：

```vue
<MyLayout>
  <template #header>
    <h2>头部内容</h2>
  </template>

  <p>默认内容</p>
</MyLayout>
```

适用场景：

- 页面布局组件：头部、侧边栏、主体、底部
- 卡片组件：标题区、内容区、操作区
- 弹窗组件：标题、正文、底部按钮
- 表格/表单容器组件：工具栏、表格主体、分页区

案例 1：卡片组件有头部和底部

```vue
<!-- UserCard.vue -->
<template>
  <div class="user-card">
    <div class="user-card__header">
      <slot name="header">默认标题</slot>
    </div>
    <div class="user-card__body">
      <slot>默认正文</slot>
    </div>
    <div class="user-card__footer">
      <slot name="footer">默认按钮</slot>
    </div>
  </div>
</template>
```

```vue
<UserCard>
  <template #header>
    <h3>用户资料</h3>
  </template>

  <p>这里是用户简介</p>

  <template #footer>
    <button>编辑</button>
  </template>
</UserCard>
```

案例 2：弹窗组件的标题和底部操作

```vue
<!-- ConfirmDialog.vue -->
<template>
  <div class="dialog">
    <div class="dialog-title">
      <slot name="title">默认标题</slot>
    </div>
    <div class="dialog-body">
      <slot>默认正文</slot>
    </div>
    <div class="dialog-footer">
      <slot name="footer">
        <button>关闭</button>
      </slot>
    </div>
  </div>
</template>
```

```vue
<ConfirmDialog>
  <template #title>
    <h3>删除确认</h3>
  </template>

  <p>删除后将无法恢复。</p>

  <template #footer>
    <button>取消</button>
    <button>确认删除</button>
  </template>
</ConfirmDialog>
```

案例 3：后台布局

```vue
<!-- AdminLayout.vue -->
<template>
  <header><slot name="header" /></header>
  <aside><slot name="sidebar" /></aside>
  <main><slot /></main>
</template>
```

```vue
<AdminLayout>
  <template #header>
    <TopNav />
  </template>

  <template #sidebar>
    <SideMenu />
  </template>

  <DashboardMain />
</AdminLayout>
```

默认插槽和具名插槽的区别：

- 默认插槽适合一个坑位。
- 具名插槽适合多个坑位。
- 具名插槽本质上也是插槽，只是多了“名字”来区分位置。

### 3. 作用域插槽

这是最容易绕的一种，但也最有价值。

核心理解：

- 默认插槽、具名插槽主要解决“父组件塞什么内容进去”。
- 作用域插槽进一步解决“子组件把自己的数据交给父组件来渲染”。

也就是说：

- 内容模板仍然是父组件写的。
- 但是模板里可以使用子组件暴露出来的数据。

子组件：

```vue
<template>
  <slot :item="item"></slot>
</template>
```

父组件：

```vue
<ListSlot v-slot="slotProps">
  <span>{{ slotProps.item.name }}</span>
</ListSlot>
```

这里的 `slotProps` 不是 Vue 固定写法，只是一个变量名，你也可以直接解构：

```vue
<ListSlot v-slot="{ item }">
  <span>{{ item.name }}</span>
</ListSlot>
```

适用场景：

- 列表组件把每一项数据交给父组件自定义渲染
- 表格组件把行数据、列数据暴露给外部
- 下拉框组件把 option 数据交给外部决定显示格式
- 分页组件把页码状态暴露出去做更灵活 UI

案例 1：列表项自定义渲染

```vue
<!-- BaseList.vue -->
<template>
  <ul>
    <li v-for="item in list" :key="item.id">
      <slot :item="item">
        {{ item.name }}
      </slot>
    </li>
  </ul>
</template>

<script setup>
defineProps({
  list: {
    type: Array,
    default: () => []
  }
})
</script>
```

```vue
<BaseList :list="users">
  <template #default="{ item }">
    <div>
      <strong>{{ item.name }}</strong>
      <span> - {{ item.age }} 岁</span>
    </div>
  </template>
</BaseList>
```

这个场景里：

- `BaseList` 负责循环。
- 父组件负责“每一项长什么样”。

案例 2：表格列自定义

```vue
<!-- BaseTable.vue -->
<template>
  <table>
    <tr v-for="row in rows" :key="row.id">
      <td>
        <slot name="name" :row="row">
          {{ row.name }}
        </slot>
      </td>
      <td>
        <slot name="status" :row="row">
          {{ row.status }}
        </slot>
      </td>
    </tr>
  </table>
</template>
```

```vue
<BaseTable :rows="tableData">
  <template #status="{ row }">
    <span :class="row.status">
      {{ row.status === 'online' ? '在线' : '离线' }}
    </span>
  </template>
</BaseTable>
```

这个模式在组件库里非常常见，因为表格组件本身不知道每一列最终要怎么展示。

案例 3：商品卡片列表，子组件给数据，父组件定样式

```vue
<!-- ProductList.vue -->
<template>
  <div class="product-list">
    <div v-for="product in products" :key="product.id">
      <slot :product="product">
        {{ product.title }}
      </slot>
    </div>
  </div>
</template>
```

```vue
<ProductList :products="goods">
  <template #default="{ product }">
    <div class="product-card">
      <h4>{{ product.title }}</h4>
      <p>￥{{ product.price }}</p>
      <button>加入购物车</button>
    </div>
  </template>
</ProductList>
```

案例 4：具名插槽 + 作用域插槽组合

```vue
<!-- BaseTable.vue -->
<template>
  <div>
    <div class="toolbar">
      <slot name="toolbar"></slot>
    </div>

    <table>
      <tr v-for="row in rows" :key="row.id">
        <td>
          <slot name="action" :row="row">
            <button>查看</button>
          </slot>
        </td>
      </tr>
    </table>
  </div>
</template>
```

```vue
<BaseTable :rows="rows">
  <template #toolbar>
    <button>新增用户</button>
  </template>

  <template #action="{ row }">
    <button @click="editRow(row)">编辑 {{ row.name }}</button>
  </template>
</BaseTable>
```

这个案例很重要，因为真实项目里经常不是单独用某一种插槽，而是混合使用。

## 插槽怎么选

可以按这个顺序判断：

1. 只有一个内容区域需要外部传入吗？
   用默认插槽。
2. 有多个固定位置都需要外部传内容吗？
   用具名插槽。
3. 子组件内部有数据，希望父组件来决定显示方式吗？
   用作用域插槽。

## 一组对比记忆

场景 1：卡片正文换内容

- 只需要一个区域，选默认插槽。

场景 2：弹窗有标题、正文、底部按钮

- 有多个位置，选具名插槽。

场景 3：表格每一列显示规则都可能不同

- 子组件提供行数据，父组件决定渲染，选作用域插槽。

## 常见误区

1. 不要把插槽和 props 混为一谈。
   `props` 传数据，`slot` 传模板结构。

2. 不要在只有一个坑位时强行用具名插槽。
   这样会让调用方写法变重。

3. 作用域插槽不是“子组件替父组件渲染”。
   恰好相反，是子组件把数据交出来，父组件自己决定怎么渲染。

4. 当你发现组件越来越像“固定外壳 + 外部决定内部内容”时，优先考虑插槽，而不是继续堆很多布尔 props。

## 一个最终理解

你可以把插槽理解成“组件预留的坑位”：

- 默认插槽：一个普通坑位。
- 具名插槽：多个带名字的坑位。
- 作用域插槽：坑位里还能把子组件的数据一起递给你。

## 八、生命周期与组件补充

应用场景：

- 用在组件挂载后请求数据、销毁前清理定时器、切换动态组件、缓存页面状态时。
- 适合图表初始化、tab 页面缓存、表单草稿保留、DOM 操作依赖节点存在等功能。

### 1. 生命周期

最常见的几个：

- `created`
- `mounted`
- `updated`
- `unmounted`

```js
export default {
  mounted() {
    console.log("组件挂载完成")
  },
  unmounted() {
    console.log("组件卸载")
  }
}
```

### 2. `$refs`

```vue
<template>
  <input ref="inputRef" />
  <button @click="focusInput">聚焦</button>
</template>

<script>
export default {
  methods: {
    focusInput() {
      this.$refs.inputRef.focus()
    }
  }
}
</script>
```

### 3. 动态组件

```vue
<component :is="currentTab"></component>
```

### 4. keep-alive

```vue
<keep-alive>
  <component :is="currentTab"></component>
</keep-alive>
```

### 5. 组件上的 v-model

子组件：

```vue
<template>
  <input :value="modelValue" @input="$emit('update:modelValue', $event.target.value)" />
</template>

<script>
export default {
  props: ["modelValue"],
  emits: ["update:modelValue"]
}
</script>
```

父组件：

```vue
<MyInput v-model="username" />
```

## 九、Composition API

应用场景：

- 用在复杂业务逻辑拆分、逻辑复用、自定义组合函数封装时。
- 适合大型页面、跨组件复用逻辑、项目逐步模块化和统一状态处理。

### 1. setup 基本使用

```vue
<script>
import { ref } from "vue"

export default {
  setup() {
    const count = ref(0)

    function increment() {
      count.value++
    }

    return {
      count,
      increment
    }
  }
}
</script>
```

### 2. reactive

```js
import { reactive } from "vue"

const state = reactive({
  name: "Tom",
  age: 18
})
```

### 3. ref

```js
import { ref } from "vue"

const message = ref("Hello Vue3")
console.log(message.value)
```

### 4. toRefs / toRef

```js
import { reactive, toRefs, toRef } from "vue"

const state = reactive({
  name: "Tom",
  age: 18
})

const { name, age } = toRefs(state)
const singleName = toRef(state, "name")
```

### 5. readonly

```js
import { reactive, readonly } from "vue"

const state = reactive({ count: 0 })
const copyState = readonly(state)
```

### 6. computed

```js
import { ref, computed } from "vue"

const firstName = ref("Tom")
const lastName = ref("Lee")

const fullName = computed(() => firstName.value + " " + lastName.value)
```

### 7. watch

```js
import { ref, watch } from "vue"

const count = ref(0)

watch(count, (newValue, oldValue) => {
  console.log(newValue, oldValue)
})
```

### 8. watchEffect

```js
import { ref, watchEffect } from "vue"

const name = ref("Tom")

watchEffect(() => {
  console.log("当前 name:", name.value)
})
```

### 9. 生命周期钩子

```js
import { onMounted, onUnmounted } from "vue"

onMounted(() => {
  console.log("mounted")
})

onUnmounted(() => {
  console.log("unmounted")
})
```

### 10. script setup

```vue
<script setup>
import { ref } from "vue"

const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">{{ count }}</button>
</template>
```

### 11. defineProps / defineEmits

```vue
<script setup>
const props = defineProps({
  title: String
})

const emit = defineEmits(["change"])

function handleClick() {
  emit("change", "new value")
}
</script>
```

## 十、高级特性

应用场景：

- 用在封装插件、自定义指令、异步加载组件、跨层级渲染弹窗和复杂渲染控制时。
- 适合权限指令、全局 Loading、消息提示、异步页面分包、组件库封装。

### 1. 自定义指令

```js
app.directive("focus", {
  mounted(el) {
    el.focus()
  }
})
```

使用：

```html
<input v-focus />
```

### 2. Teleport

```vue
<teleport to="body">
  <div class="modal">这是弹窗</div>
</teleport>
```

### 3. 异步组件

```js
import { defineAsyncComponent } from "vue"

const AsyncHome = defineAsyncComponent(() => import("./Home.vue"))
```

### 4. Suspense

```vue
<Suspense>
  <template #default>
    <AsyncHome />
  </template>
  <template #fallback>
    <div>加载中...</div>
  </template>
</Suspense>
```

### 5. 插件

```js
const MyPlugin = {
  install(app) {
    app.config.globalProperties.$format = function(value) {
      return "结果：" + value
    }
  }
}

app.use(MyPlugin)
```

### 6. h 函数

```js
import { h } from "vue"

export default {
  render() {
    return h("h2", { class: "title" }, "Hello Render")
  }
}
```

### 7. JSX

```jsx
export default {
  setup() {
    const message = "Hello JSX"
    return () => <h2>{message}</h2>
  }
}
```

## 十一、过渡动画

应用场景：

- 用在弹窗显示隐藏、列表增删、页面切换、提示消息过渡等提升体验的交互里。
- 适合后台管理、移动端 H5、营销页和引导页中的轻量动效。

### 1. Transition

```vue
<template>
  <button @click="isShow = !isShow">切换</button>
  <Transition name="fade">
    <p v-if="isShow">Hello Vue3</p>
  </Transition>
</template>

<script setup>
import { ref } from "vue"
const isShow = ref(true)
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```

### 2. TransitionGroup

```vue
<TransitionGroup name="list" tag="ul">
  <li v-for="item in list" :key="item.id">{{ item.name }}</li>
</TransitionGroup>
```

## 十二、Vue3 响应式原理

应用场景：

- 用在理解为什么数据变化会驱动页面更新，也适合排查响应式失效问题。
- 适合学习框架底层、实现简化版响应式系统、准备面试原理题。

### 1. 响应式的核心思想

响应式的本质是：

- 依赖收集
- 数据变化时触发更新

### 2. 简化版依赖收集

```js
let activeFn = null

function watchFn(fn) {
  activeFn = fn
  fn()
  activeFn = null
}

class Depend {
  constructor() {
    this.reactiveFns = new Set()
  }

  depend() {
    if (activeFn) {
      this.reactiveFns.add(activeFn)
    }
  }

  notify() {
    this.reactiveFns.forEach(fn => fn())
  }
}
```

### 3. Proxy 实现响应式

```js
const obj = { name: "Tom" }
const depend = new Depend()

const proxy = new Proxy(obj, {
  get(target, key) {
    depend.depend()
    return target[key]
  },
  set(target, key, value) {
    target[key] = value
    depend.notify()
    return true
  }
})
```

### 4. Vue2 和 Vue3 的差异

- Vue2 主要基于 `Object.defineProperty`
- Vue3 主要基于 `Proxy`
- Vue3 对新增属性、删除属性、数组等场景支持更自然

## 十三、Vue Router

应用场景：

- 用在多页面视图切换、详情页跳转、权限路由控制、菜单导航和懒加载优化里。
- 适合后台管理系统、博客、商城、文档站这类典型单页应用。

### 1. 创建路由

```js
import { createRouter, createWebHistory } from "vue-router"
import Home from "../views/Home.vue"
import About from "../views/About.vue"

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: "/", component: Home },
    { path: "/about", component: About }
  ]
})

export default router
```

### 2. router-link 与 router-view

```vue
<router-link to="/">首页</router-link>
<router-link to="/about">关于</router-link>

<router-view />
```

### 3. 动态路由

```js
{
  path: "/user/:id",
  component: User
}
```

获取参数：

```js
import { useRoute } from "vue-router"

const route = useRoute()
console.log(route.params.id)
```

### 4. 编程式导航

```js
import { useRouter } from "vue-router"

const router = useRouter()

router.push("/about")
router.replace("/login")
```

### 5. 路由守卫

```js
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem("token")
  if (to.path === "/profile" && !token) {
    next("/login")
  } else {
    next()
  }
})
```

### 6. 路由懒加载

```js
{
  path: "/about",
  component: () => import("../views/About.vue")
}
```

## 十四、状态管理

应用场景：

- 用在多个页面或多个组件共享用户信息、主题、购物车、权限、缓存数据时。
- 适合大型项目集中管理全局状态，避免层层传参和状态分散。

### 1. Vuex 基本使用

```js
import { createStore } from "vuex"

const store = createStore({
  state() {
    return {
      count: 0
    }
  },
  mutations: {
    increment(state) {
      state.count++
    }
  }
})

export default store
```

组件中使用：

```vue
<template>
  <button @click="$store.commit('increment')">{{ $store.state.count }}</button>
</template>
```

### 2. getters

```js
getters: {
  doubleCount(state) {
    return state.count * 2
  }
}
```

### 3. actions

```js
actions: {
  asyncIncrement(context) {
    setTimeout(() => {
      context.commit("increment")
    }, 1000)
  }
}
```

### 4. Pinia 基本使用

```js
import { defineStore } from "pinia"

export const useCounterStore = defineStore("counter", {
  state: () => ({
    count: 0
  }),
  getters: {
    doubleCount: state => state.count * 2
  },
  actions: {
    increment() {
      this.count++
    }
  }
})
```

组件中使用：

```vue
<script setup>
import { useCounterStore } from "@/stores/counter"

const counterStore = useCounterStore()
</script>

<template>
  <button @click="counterStore.increment()">
    {{ counterStore.count }}
  </button>
</template>
```

## 十五、axios 网络请求

应用场景：

- 用在请求用户信息、获取列表、提交表单、上传文件、统一处理 token 和错误时。
- 适合封装项目请求层，给不同页面复用拦截器和公共配置。

### 1. 基本请求

```js
import axios from "axios"

axios.get("/users").then(res => {
  console.log(res.data)
})
```

### 2. 创建实例

```js
import axios from "axios"

const request = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000
})

export default request
```

### 3. 请求拦截器和响应拦截器

```js
request.interceptors.request.use(config => {
  config.headers.Authorization = "Bearer token"
  return config
})

request.interceptors.response.use(res => {
  return res.data
})
```

## 十六、项目实战思路

应用场景：

- 用在把路由、状态管理、组件化、请求、表单、权限这些知识真正串成完整项目。
- 适合从练习题过渡到真实业务开发，建立页面拆分和模块设计能力。

做 Vue 项目时，建议重点关注这些方面：

1. 目录结构划分
2. 全局样式重置
3. 路由划分
4. 状态管理
5. API 封装
6. 组件复用
7. 权限控制

示例目录：

```text
src
├─ assets
├─ components
├─ views
├─ router
├─ stores
├─ service
├─ utils
└─ App.vue
```

## 十七、项目打包和部署

应用场景：

- 用在把本地开发完成的 Vue 项目发布到测试环境、正式环境或云服务器上。
- 适合个人作品上线、企业前端静态资源部署、自动化发布流程搭建。

### 1. 打包

通常通过：

```bash
npm run build
```

打包后会生成 `dist` 目录。

### 2. Nginx 部署

一个最简思路：

1. 把 `dist` 上传到服务器
2. 用 Nginx 指向静态目录
3. 配置 history 路由回退

示例配置：

```nginx
server {
  listen 80;
  server_name example.com;

  location / {
    root /usr/share/nginx/html;
    index index.html;
    try_files $uri $uri/ /index.html;
  }
}
```

### 3. 自动化部署

常见组合：

- Git + Jenkins
- GitHub Actions
- Docker + Nginx

## 十八、这套 Vue3 内容的核心学习重点

如果时间有限，优先掌握：

### 第一优先级

- 模板语法和常用指令
- 组件化和组件通信
- Composition API
- Router
- Pinia
- axios 基础封装

### 第二优先级

- 插槽、生命周期、keep-alive
- Teleport、异步组件、插件
- 过渡动画
- Vue3 响应式原理

### 第三优先级

- Vuex
- h 函数 / JSX
- 打包部署与自动化

## 十九、容易混淆的知识点

### 1. `v-if` 和 `v-show`

- `v-if`：控制渲染
- `v-show`：控制显示

### 2. `reactive` 和 `ref`

- `reactive`：更适合对象
- `ref`：更适合基本类型，也能包对象

### 3. `computed` 和 `watch`

- `computed`：派生值
- `watch`：监听变化后执行副作用

### 4. props 和 emits

- props：父传子
- emits：子传父

### 5. Vuex 和 Pinia

- Vuex 更传统
- Pinia 更现代、写法更轻量

## 二十、建议你的复习方式

### 第一轮

- 顺着这份文档走一遍
- 能说出每个知识点解决什么问题

### 第二轮

- 把每个核心示例手敲一遍
- 特别练：组件通信、Composition API、Router、Pinia、axios

### 第三轮

- 自己做小案例
- 例如：计数器、TodoList、Tab 切换、商品列表、登录页

### 第四轮

- 自己搭一个完整小项目
- 用到：Router + Pinia + axios + 组件拆分

## 二十一、一个最小知识树

```text
Vue3
├─ 基础
│  ├─ createApp
│  ├─ 模板语法
│  ├─ 指令
│  └─ v-model
├─ 组件化
│  ├─ props / emits
│  ├─ slot
│  ├─ 生命周期
│  └─ keep-alive
├─ Composition API
│  ├─ setup
│  ├─ reactive / ref
│  ├─ computed / watch
│  └─ script setup
├─ 高级特性
│  ├─ directive
│  ├─ Teleport
│  ├─ Suspense
│  ├─ h / JSX
│  └─ Transition
├─ 全家桶
│  ├─ Router
│  ├─ Vuex
│  ├─ Pinia
│  └─ axios
└─ 工程化
   ├─ 项目结构
   ├─ 打包
   └─ 部署
```

## 二十二、学完这部分后的目标

学完这部分后，你至少应该能做到：

- 用 Vue3 独立写中小型页面
- 用组件拆分页面结构
- 用 props、emits、slot 完成组件通信
- 用 Composition API 管理逻辑
- 用 Router 管理页面切换
- 用 Pinia 管理共享状态
- 用 axios 封装请求
- 看懂 Vue3 响应式和基本运行机制
