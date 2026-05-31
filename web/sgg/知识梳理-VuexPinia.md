# Vuex & Pinia 知识梳理

## 第一章 为什么需要状态管理

### 1.1 状态管理的问题

在没有状态管理库的项目中，多个组件之间共享状态非常困难：

```javascript
// 问题 1: Props Drilling (逐层传递)
// App → A → B → C → D
// 多层嵌套时，D 组件需要的数据需要从 App 逐层传递

// 问题 2: 兄弟组件通信困难
// 需要通过 EventBus 或状态提升，容易导致代码混乱

// 问题 3: 全局状态难以维护
// 没有统一的数据管理方案，容易产生 Bug
```

### 1.2 状态管理的核心思想

```
集中管理应用的全局状态
     ↓
通过统一的接口 (mutations/actions) 修改状态
     ↓
组件订阅状态变化，自动更新
```

---

## 第二章 Vuex 详解

### 2.1 Vuex 5 大模块

```typescript
// 1. State - 数据状态
state: () => ({
    num: 200,
    arr: [1, 2, 3]
})

// 2. Getters - 派生状态 (计算属性)
getters: {
    sum() {
        return this.arr.reduce((s, item) => s + item, 0)
    },
    // 接收其他 getter 作为第二个参数
    doubleSum(state, getters) {
        return getters.sum * 2
    }
}

// 3. Mutations - 同步修改状态
mutations: {
    addOne(state, payload) {
        state.num += payload
    }
}

// 4. Actions - 异步操作
actions: {
    async fetchData(context, payload) {
        const response = await fetch('/api/data')
        const data = await response.json()
        context.commit('updateData', data)
    }
}

// 5. Modules - 模块化管理
modules: {
    user: userModule,
    product: productModule
}
```

### 2.2 使用 Vuex Store

```typescript
// src/store/modules/counter.ts
import { defineStore } from 'pinia'  // 注: 这是 Pinia 写法，Vuex 类似

const useCounterStore = defineStore('counter', {
    state: () => ({
        num: 100
    }),
    getters: {
        getNum() {
            return this.num
        }
    },
    mutations: {
        // ※ 重要: mutations 必须同步!
        addOne() {
            this.num += 1
        }
    },
    actions: {
        async fetchAndAdd() {
            const response = await fetch('/api/num')
            const data = await response.json()
            this.num += data.value
        }
    }
})

export default useCounterStore
```

**常见纠错 01:**
```
问题: mutation 中执行异步操作
✗ 错误: 
    mutations: {
        async fetchData(state) {
            const res = await fetch('/api')  // 异步!
        }
    }

✓ 正确: mutations 必须同步
    mutations: {
        updateData(state, data) {
            state.data = data
        }
    }
    actions: {
        async fetchData(context) {
            const data = await fetch('/api')
            context.commit('updateData', data)  // 提交 mutation
        }
    }

原因: Vuex 需要追踪状态变化的时序，
     如果 mutations 异步，无法准确追踪
```

### 2.3 严格模式

```typescript
const store = new Vuex.Store({
    state: { count: 0 },
    mutations: { increment(state) { state.count++ } },
    strict: process.env.NODE_ENV !== 'production'  // 仅在开发环境启用
})

// 严格模式下，直接修改 state 会抛出错误
// store.state.count++  // Error!
```

**常见纠错 02:**
```
问题: 严格模式在生产环境中的性能问题
✗ 错误: strict: true  // 在生产环境也启用
✓ 正确: strict: process.env.NODE_ENV !== 'production'

原因: 严格模式会对每个 state 变化进行深度监听，
     生产环境中会造成明显的性能下降
```

### 2.4 Vuex 单向数据流 - ASCII 图 1:

```
┌────────────────────────────────────────────────┐
│          Vuex 单向数据流                       │
├────────────────────────────────────────────────┤
│                                                │
│           Vue Components (视图)                │
│           ↓                ↑                   │
│      dispatch actions    访问 state/getters   │
│           ↓                ↑                   │
│         Actions           State               │
│       (异步操作)         (数据状态)           │
│           ↓                ↑                   │
│     commit mutations       ↑                   │
│           ↓                ↑                   │
│       Mutations            ↑                   │
│    (同步更新状态)         ↑                   │
│           ↓                ↑                   │
│      State 变化 ──────────┘                   │
│                                                │
│  流程:                                         │
│  组件 dispatch action                         │
│  ↓                                             │
│  action 处理异步操作                          │
│  ↓                                             │
│  action 提交 mutation                         │
│  ↓                                             │
│  mutation 同步修改 state                      │
│  ↓                                             │
│  state 更新，视图自动响应                    │
│                                                │
└────────────────────────────────────────────────┘
```

### 2.5 模块化管理

```typescript
// src/store/modules/user.ts
const userModule = {
    namespaced: true,  // 启用命名空间
    state: () => ({
        name: 'zhangsan',
        age: 20
    }),
    mutations: {
        setName(state, name) {
            state.name = name
        }
    }
}

// src/store/index.ts
const store = new Vuex.Store({
    modules: {
        user: userModule
    }
})

// 使用时需要加上命名空间前缀
store.commit('user/setName', 'lisi')
store.state.user.name
```

**Vuex 模块命名空间解析 - ASCII 图 2:**

```
┌──────────────────────────────────────────────┐
│    Vuex 模块命名空间解析流程                 │
├──────────────────────────────────────────────┤
│                                              │
│  定义模块:                                   │
│  ──────────────────────────────────        │
│  const userModule = {                       │
│      namespaced: true,  ← 启用命名空间    │
│      state: { name: 'zhangsan' }           │
│      mutations: { setName(state, val) }   │
│  }                                         │
│                                              │
│  组册到 store:                               │
│  ────────────────────────────────         │
│  modules: { user: userModule }            │
│                                              │
│  访问 state:                                 │
│  ────────────────────────────────         │
│  store.state.user.name                    │
│  或模板中: $store.state.user.name         │
│                                              │
│  提交 mutation:                              │
│  ────────────────────────────────         │
│  store.commit('user/setName', 'lisi')    │
│  └─ 前缀 user/ 指定模块                   │
│                                              │
│  dispatch action:                           │
│  ────────────────────────────────         │
│  store.dispatch('user/fetchUser')        │
│  └─ 前缀 user/ 指定模块                   │
│                                              │
│  ※ 不启用命名空间:                         │
│    所有 mutations/actions 都在全局        │
│    无法避免名称冲突                       │
│                                              │
└──────────────────────────────────────────────┘
```

**常见纠错 03:**
```
问题: 模块命名空间默认行为
✗ 错误: 定义了模块，但未启用 namespaced: true
      然后用 store.commit('user/setName') 抛错

✓ 正确: 启用命名空间
      const module = { namespaced: true, ... }
      
✓ 或者: 不使用前缀
      store.commit('setName')  // 全局调用
      
原因: namespaced 默认为 false，
     所有模块的 mutations 都注册到全局
```

### 2.6 Vuex 数据持久化

```typescript
// 手动保存
const store = new Vuex.Store({
    state: { user: null },
    mutations: {
        setUser(state, user) {
            state.user = user
            // 保存到 localStorage
            localStorage.setItem('user', JSON.stringify(user))
        }
    }
})

// 初始化时从 localStorage 恢复
const savedUser = localStorage.getItem('user')
if (savedUser) {
    store.commit('setUser', JSON.parse(savedUser))
}

// 或使用 vuex-persist 插件
import VuexPersist from 'vuex-persist'

const vuexLocal = new VuexPersist({
    key: 'vuex',
    storage: window.localStorage
})

const store = new Vuex.Store({
    // ...
    plugins: [vuexLocal.plugin]
})
```

---

## 第三章 Pinia 详解

### 3.1 Pinia 的设计哲学

Pinia 是 Vuex 的现代化替代品，具有以下优点：

| 对比项 | Vuex | Pinia |
|--------|------|-------|
| 代码复杂度 | 需要定义 5 个模块 | 更简洁的 API |
| 模块化 | modules 概念 | 天然支持多 store |
| TypeScript | 类型推断差 | 类型推断完美 |
| 性能 | 开销较大 | 更轻量 |
| 学习曲线 | 陡峭 | 平缓 |

### 3.2 Pinia 安装与配置

```typescript
// src/store/index.ts
import { createPinia } from 'pinia'
export default createPinia()

// src/main.ts
import { createApp } from 'vue'
import App from '@/App.vue'
import store from '@/store'

createApp(App)
    .use(store)
    .mount('#app')
```

### 3.3 选项式 API (Pinia)

```typescript
// src/store/modules/counter.ts
import { defineStore } from 'pinia'

const useCounterStore = defineStore('counter', {
    // State
    state: () => ({
        num: 200,
        arr: [1, 2, 3, 4, 5]
    }),

    // Getters (计算属性)
    getters: {
        sum() {
            return this.arr.reduce((s, item) => s + item, 0)
        }
    },

    // Actions (方法，支持异步)
    actions: {
        addOne(a, b, c, d) {
            console.log(a, b, c, d)
            this.num += 1
        },
        async delaySet() {
            // 异步操作
            await new Promise(resolve => setTimeout(resolve, 2000))
            this.num = 900
        }
    }
})

export default useCounterStore
```

**使用选项式 API:**

```vue
<template>
    <h3>练习 Pinia</h3>
    <p>num: {{ counter.num }}</p>
    <p>arr: {{ counter.arr }}</p>
    <p>sum: {{ counter.sum }}</p>
    
    <!-- 直接修改 state -->
    <button @click="counter.num++">直接修改</button>
    
    <!-- 使用 $patch -->
    <button @click="patchNum">$patch 修改</button>
    
    <!-- 调用 action -->
    <button @click="counter.addOne(1,2,3,4)">调用 action</button>
    <button @click="counter.delaySet">异步更新</button>
</template>

<script setup lang="ts">
import useCounterStore from '@/store/modules/counter'
const counter = useCounterStore()

const patchNum = () => {
    counter.$patch({
        num: counter.num + 3
    })
}
</script>
```

### 3.4 Composition API (Pinia)

```typescript
// src/store/modules/todos.ts
import { defineStore } from 'pinia'
import { ref, reactive, computed, watch } from 'vue'

const useTodosStore = defineStore('todos', () => {
    // State (响应式数据)
    let taskList = ref([1, 2, 3, 4])
    let obj = reactive({
        userName: 'zhangsan',
        age: 12
    })

    // Actions (方法)
    const addTaskList = (num) => {
        taskList.value.push(num)
    }

    // Getters (计算属性)
    const sum = computed(() => {
        return taskList.value.reduce((v, item) => v + item, 0)
    })

    // Watch (侦听)
    watch(
        () => taskList.value,
        () => {
            console.log('taskList 改变了')
        },
        { immediate: true, deep: true }
    )

    // 必须返回!
    return {
        taskList,
        obj,
        addTaskList,
        sum
    }
})

export default useTodosStore
```

### 3.5 Pinia Store 注册到根 Store 流程 - ASCII 图 3:

```
┌──────────────────────────────────────────────────┐
│     Pinia Store 注册到根 Store 流程              │
├──────────────────────────────────────────────────┤
│                                                  │
│  1. 创建大仓库 (根 Store)                        │
│     ───────────────────────────────            │
│     const pinia = createPinia()                 │
│                                                  │
│  2. 在 main.ts 中安装                          │
│     ───────────────────────────────            │
│     app.use(pinia)                             │
│                                                  │
│  3. 定义模块 Store                              │
│     ───────────────────────────────            │
│     export const useCounterStore = defineStore(
│         'counter',  ← store 的唯一标识          │
│         {...}                                   │
│     )                                           │
│                                                  │
│  4. 在组件中使用 Store                          │
│     ───────────────────────────────            │
│     const counter = useCounterStore()          │
│     └─ 首次调用时，Pinia 自动注册到根 Store   │
│     └─ 后续调用返回同一个实例 (单例)           │
│                                                  │
│  5. 访问 State/Getters/Actions                 │
│     ───────────────────────────────            │
│     counter.num          // 访问 state        │
│     counter.sum          // 访问 getter        │
│     counter.addOne()     // 调用 action        │
│                                                  │
│  ※ 核心: useCounterStore() 负责创建/返回     │
│          该 store 实例，Pinia 管理其生命周期  │
│                                                  │
└──────────────────────────────────────────────────┘
```

### 3.6 Pinia 在组件外部使用

```typescript
// src/utils/api.ts
import { useCounterStore } from '@/store/modules/counter'
import store from '@/store'  // 大仓库

// 组件外部使用 store 时，必须传入大仓库
const counter = useCounterStore(store)
console.log(counter.num)
```

### 3.7 Pinia $reset 方法

```typescript
const counter = useCounterStore()

// 重置 store 到初始状态
counter.$reset()

// 等价于
counter.$patch({
    num: 200,
    arr: [1, 2, 3, 4, 5]
})
```

**常见纠错 04:**
```
问题: $reset 的限制
✗ 如果 state 中有复杂的初始化逻辑，$reset 无法恢复

✓ 解决方案: 手动定义 reset action
    actions: {
        reset() {
            // 手动恢复所有状态
            this.num = 200
            this.arr = [1, 2, 3]
        }
    }
```

---

## 第四章 Vuex vs Pinia 对比

| 对比项 | Vuex | Pinia |
|--------|------|-------|
| 代码量 | 多 | 少 |
| mutations | 有 | 无 (直接在 actions 中修改) |
| getters | 有 | 有 |
| modules | 需要手动配置 | 自动处理 |
| 模块化 | 通过 modules | 多个 store |
| 调用方式 | store.dispatch/commit | store.action() / store.state |
| TypeScript | 支持差 | 支持完美 |
| 学习难度 | 高 | 低 |
| 官方推荐 | Vue 2 | Vue 3 |

**常见纠错 05:**
```
问题: Pinia 中没有 mutations
✗ 错误: 在 Pinia 中定义 mutations
✓ 正确: 直接在 actions 中修改 state
        或在组件中直接修改 (store.num++)

原因: Pinia 简化了 API，
     不需要 mutations/actions 的区分
```

---

## 第五章 SSR 注意事项

### 5.1 状态序列化 (Hydration)

```typescript
// 在服务端
const store = useCounterStore()

// 将初始状态序列化到 HTML
const state = JSON.stringify(store.$state)
// <script>window.__INITIAL_STATE__ = {...}</script>

// 在客户端
const store = useCounterStore()

// 从初始状态恢复
if (window.__INITIAL_STATE__) {
    store.$patch(window.__INITIAL_STATE__)
}
```

### 5.2 避免跨请求污染

```typescript
// 不好的做法 - 会在请求间共享状态
const globalStore = useStore()  // 在服务端创建一次

// 好的做法 - 每个请求创建新的 store
app.get('/', (req, res) => {
    const pinia = createPinia()
    const store = useStore(pinia)  // 每个请求新建
    // ...
})
```

---

## 第六章 与 Composition API 联用

### 6.1 自定义 Composable 结合 Pinia

```typescript
// composables/useCounter.ts
import { useCounterStore } from '@/store/counter'
import { computed } from 'vue'

export function useCounter() {
    const store = useCounterStore()
    
    // 基于 store 派生新的计算属性
    const doubleNum = computed(() => store.num * 2)
    
    // 包装 action 添加额外逻辑
    const incrementWithLog = () => {
        console.log('Before:', store.num)
        store.increment()
        console.log('After:', store.num)
    }
    
    return {
        // 暴露 store 和派生属性
        num: computed(() => store.num),
        doubleNum,
        incrementWithLog
    }
}

// 在组件中使用
import { useCounter } from '@/composables/useCounter'

export default {
    setup() {
        const { num, doubleNum, incrementWithLog } = useCounter()
        return { num, doubleNum, incrementWithLog }
    }
}
```

---

## 易错速查

| 编号 | 错误 | 正确做法 | 原因 |
|-----|------|--------|------|
| 01 | mutations 中异步操作 | 异步逻辑放在 actions | Vuex 需要同步追踪 |
| 02 | 生产环境启用严格模式 | 仅在开发环境启用 | 性能问题 |
| 03 | 模块忘记启用命名空间 | namespaced: true | 避免全局冲突 |
| 04 | $reset 无法恢复复杂初始化 | 手写 reset action | 需要自定义逻辑 |
| 05 | Pinia 中定义 mutations | 直接在 actions 中修改 | Pinia 简化了 API |

---

**文档生成日期: 2026-05-25**
