# Vue3 与 React 对照学习

这份文档用来把 [Vue3学习整理.md](/C:/Users/ke/Desktop/learn_document/web/Vue3/Vue3学习整理.md) 和 [React学习整理.md](/C:/Users/ke/Desktop/learn_document/web/React/React学习整理.md) 里的同类知识点放在一起对照，方便你建立“同一问题，两种框架怎么做”的理解。

## 1. 框架定位

Vue3：

- 更强调模板驱动
- 上手更平滑
- 官方提供更完整的一体化方案

React：

- 更强调 JavaScript 驱动 UI
- JSX 灵活度更高
- 生态更开放，选择空间更大

## 2. Hello World

Vue3：

```html
<div id="app"></div>
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
  Vue.createApp({
    template: `<h2>Hello Vue3</h2>`
  }).mount("#app")
</script>
```

React：

```html
<div id="root"></div>
<script src="https://unpkg.com/react@18/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
<script type="text/babel">
  function App() {
    return <h2>Hello React</h2>
  }

  ReactDOM.createRoot(document.getElementById("root")).render(<App />)
</script>
```

## 3. 插值与模板表达式

Vue3：

```html
<h2>{{ message }}</h2>
```

React：

```jsx
<h2>{message}</h2>
```

理解：

- Vue 用模板插值
- React 用 JSX 表达式

## 4. 属性绑定

Vue3：

```html
<img :src="imgUrl" :alt="title" />
```

React：

```jsx
<img src={imgUrl} alt={title} />
```

## 5. 事件绑定

Vue3：

```html
<button @click="increment">+1</button>
```

React：

```jsx
<button onClick={increment}>+1</button>
```

## 6. 条件渲染

Vue3：

```html
<p v-if="isShow">显示内容</p>
<p v-else>隐藏内容</p>
```

React：

```jsx
{isShow ? <p>显示内容</p> : <p>隐藏内容</p>}
```

## 7. 列表渲染

Vue3：

```html
<li v-for="item in books" :key="item.id">{{ item.name }}</li>
```

React：

```jsx
{books.map(item => (
  <li key={item.id}>{item.name}</li>
))}
```

共同点：

- 都推荐稳定唯一的 `key`

## 8. 双向绑定与表单

Vue3：

```html
<input v-model="message" />
```

React：

```jsx
<input value={message} onChange={e => setMessage(e.target.value)} />
```

理解：

- Vue 内置 `v-model`
- React 一般自己写受控组件

## 9. 状态定义

Vue3 Options API：

```js
data() {
  return {
    count: 0
  }
}
```

Vue3 Composition API：

```js
const count = ref(0)
```

React：

```jsx
const [count, setCount] = useState(0)
```

## 10. 计算属性 vs 派生值

Vue3：

```js
const fullName = computed(() => firstName.value + " " + lastName.value)
```

React：

```jsx
const fullName = useMemo(() => firstName + " " + lastName, [firstName, lastName])
```

理解：

- Vue 有专门的 `computed`
- React 一般通过普通表达式或 `useMemo` 处理

## 11. 侦听变化

Vue3：

```js
watch(count, (newValue, oldValue) => {
  console.log(newValue, oldValue)
})
```

React：

```jsx
useEffect(() => {
  console.log(count)
}, [count])
```

理解：

- Vue `watch` 更像“显式监听某个源”
- React `useEffect` 更像“依赖变化后执行副作用”

## 12. 生命周期

Vue3：

```js
onMounted(() => {
  console.log("mounted")
})
```

React：

```jsx
useEffect(() => {
  console.log("mounted")
}, [])
```

卸载：

Vue3：

```js
onUnmounted(() => {
  console.log("unmounted")
})
```

React：

```jsx
useEffect(() => {
  return () => {
    console.log("unmounted")
  }
}, [])
```

## 13. 组件通信

父传子：

Vue3：

```vue
<Child :title="title" />
```

```js
defineProps({
  title: String
})
```

React：

```jsx
<Child title={title} />
```

```jsx
function Child(props) {
  return <h2>{props.title}</h2>
}
```

子传父：

Vue3：

```vue
<Child @add="handleAdd" />
```

React：

```jsx
<Child onAdd={handleAdd} />
```

## 14. 插槽 vs children

Vue3：

```vue
<Card>
  <template #header>
    <h3>标题</h3>
  </template>
</Card>
```

React：

```jsx
<Card>
  <h3>标题</h3>
</Card>
```

理解：

- Vue 有更完整的 slot 语法
- React 常用 `children` 或 render props

## 15. 全局共享

Vue3：

- `provide/inject`
- Pinia / Vuex

React：

- Context
- Redux Toolkit

轻量共享：

Vue3：

```js
provide("theme", "dark")
```

React：

```jsx
const ThemeContext = createContext("dark")
```

## 16. 路由

Vue3：

```js
createRouter({
  history: createWebHistory(),
  routes: [{ path: "/", component: Home }]
})
```

React：

```jsx
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Home />} />
  </Routes>
</BrowserRouter>
```

## 17. 状态管理

Vue3 Pinia：

```js
export const useCounterStore = defineStore("counter", {
  state: () => ({ count: 0 }),
  actions: {
    increment() {
      this.count++
    }
  }
})
```

React Redux Toolkit：

```js
const counterSlice = createSlice({
  name: "counter",
  initialState: { count: 0 },
  reducers: {
    increment(state) {
      state.count += 1
    }
  }
})
```

## 18. 样式方案

Vue3：

- `<style scoped>`
- CSS Modules
- CSS 预处理器

React：

- 普通 CSS
- CSS Modules
- styled-components
- CSS-in-JS

## 19. 响应式思维差异

Vue3：

- 更强调“数据变了，模板自动响应”
- `reactive` / `ref` 很直接

React：

- 更强调“状态更新后，组件重新执行渲染函数”
- `useState` / `useReducer` 是核心

一句话记忆：

- Vue 更像“响应式系统 + 模板”
- React 更像“状态驱动的 UI 函数”

## 20. 学习建议

如果你两个都学，推荐顺序：

1. 先把 HTML/CSS/JavaScript 基础打牢
2. 先深入一个框架作为主线
3. 再对照学习另一个框架

如果你已经学过 Vue3，再学 React 时重点关注：

- JSX 思维
- 受控组件
- Hooks
- Redux Toolkit

如果你已经学过 React，再学 Vue3 时重点关注：

- 模板语法
- `v-model`
- `computed` / `watch`
- `ref` / `reactive`
- slot
