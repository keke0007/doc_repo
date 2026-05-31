# VueRouter 知识梳理

## 第一章 路由模式

### 1.1 Hash 模式 vs History 模式

```javascript
// Vue Router 创建
const router = createRouter({
    // Hash 模式: URL 中有 #
    // history: createWebHashHistory(),
    
    // History 模式: 干净的 URL，需要后端配置
    history: createWebHistory(),
    routes
})
```

**Hash vs History 原理对比 - ASCII 图 1:**
```
┌────────────────────────────────────────────────┐
│           Hash vs History 对比                 │
├────────────────────────────────────────────────┤
│                                                │
│  Hash 模式 (#)                                │
│  ──────────────────────────────              │
│  URL: http://example.com/#/home/list        │
│       └─ # 后面的内容不发送给服务器         │
│       └─ 前端完全控制路由                   │
│  机制: hashchange 事件监听                   │
│  兼容: IE8+                                  │
│  优点: 无需后端配置，完全前端路由           │
│  缺点: URL 不够美观                          │
│                                                │
│  ─────────────────────────────────────────   │
│                                                │
│  History 模式                                │
│  ──────────────────────────────              │
│  URL: http://example.com/home/list          │
│       └─ 完整 URL，所有内容发送给服务器    │
│  机制: History API (pushState/replaceState) │
│       用 popstate 事件监听返回/前进         │
│  兼容: IE10+                                 │
│  优点: URL 美观，符合 RESTful               │
│  缺点: 需要后端配置 (404 重定向到根)       │
│                                                │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━        │
│                                                │
│  ※ 后端配置 History 模式:                    │
│    所有非文件请求都重定向到 /index.html    │
│    Nginx 配置:                              │
│    location / {                             │
│        try_files $uri /index.html;          │
│    }                                        │
│                                                │
└────────────────────────────────────────────────┘
```

**常见纠错 01:**
```
问题: History 模式刷新 404
✗ 错误: 前端未配置后端，所有请求直接到后端
✓ 正确: 后端需配置：非文件请求都返回 index.html
      前端再由 Vue Router 处理路由
```

---

## 第二章 路由基础配置

### 2.1 创建路由

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import Home from '@/pages/Home/index.vue'
import About from '@/pages/About/index.vue'
import NotFound from '@/pages/NotFound/index.vue'

const routes = [
    {
        path: '/',
        redirect: '/home'  // 重定向
    },
    {
        path: '/home',
        component: Home
    },
    {
        path: '/about',
        component: About
    },
    {
        // Vue Router 4 使用正则实现 404
        path: '/:pathMatch(.*)*',
        component: NotFound
    }
]

const router = createRouter({
    history: createWebHistory(),
    routes,
    // 滚动行为
    scrollBehavior() {
        return {
            left: 0,
            top: 0
        }
    }
})

export default router
```

### 2.2 在 main.ts 安装

```typescript
import { createApp } from 'vue'
import App from '@/App.vue'
import router from '@/router'

createApp(App)
    .use(router)
    .mount('#app')
```

---

## 第三章 嵌套路由

### 3.1 二级路由

```typescript
// 路由配置
const routes = [
    {
        path: '/home',
        component: Home,
        children: [
            {
                path: '/home',  // 默认重定向
                redirect: '/home/newsList'
            },
            {
                path: '/home/newsList',
                component: NewsList
            },
            {
                path: '/home/message',
                component: Message
            }
        ]
    }
]
```

### 3.2 在父组件中使用 `<router-view>`

```vue
<!-- Home.vue 父组件 -->
<template>
    <h2>Home 组件内容</h2>
    <div>
        <ul class="nav nav-tabs">
            <li><router-link to="/home/newsList">News</router-link></li>
            <li><router-link to="/home/message">Message</router-link></li>
        </ul>
        <!-- 二级路由出口 -->
        <router-view />
    </div>
</template>
```

### 3.3 三级路由

```typescript
const routes = [
    {
        path: '/home',
        component: Home,
        children: [
            {
                path: '/home/message',
                component: Message,
                children: [
                    {
                        path: '/home/message/:filmId',
                        component: Detail
                    }
                ]
            }
        ]
    }
]
```

---

## 第四章 路由参数传递

### 4.1 路径参数 (params)

```typescript
// 路由定义
{ path: '/home/message/:filmId', component: Detail }

// 模板中跳转
<router-link :to="'/home/message/' + item.filmId">
    {{ item.name }}
</router-link>

// 获取参数 - Script 中
import { useRoute } from 'vue-router'
const $route = useRoute()
console.log($route.params.filmId)  // 获取参数

// 获取参数 - 模板中
{{ $route.params.filmId }}
```

### 4.2 查询参数 (query)

```typescript
// 跳转时传递
<router-link :to="{ path: '/about', query: { id: 1 } }">
    About
</router-link>

// 编程式导航
this.$router.push({ path: '/about', query: { id: 1 } })

// 获取参数
console.log(this.$route.query.id)
```

### 4.3 使用 watch 监听参数变化

```vue
<script setup lang="ts">
import { watch } from 'vue'
import { useRoute } from 'vue-router'

const $route = useRoute()

watch(() => $route.params.filmId, async (filmId) => {
    // 参数变化时重新获取数据
    const response = await fetch(`/api/film/${filmId}`)
    // ...
}, {
    immediate: true  // 初次加载时也执行
})
</script>
```

---

## 第五章 编程式导航

### 5.1 push vs replace

```javascript
import { useRouter } from 'vue-router'

const $router = useRouter()

// push: 添加历史记录 (可返回)
$router.push('/home/message/1')
$router.push({ path: '/home/message/1' })
$router.push({ name: 'message', params: { id: 1 } })

// replace: 替换当前记录 (不可返回)
$router.replace('/home/message/1')

// back: 返回
$router.back()

// forward: 前进
$router.forward()

// go: 前进/返回指定步数
$router.go(-1)  // 返回 1 步
$router.go(2)   // 前进 2 步
```

**常见纠错 02:**
```
问题: push 与 replace 区别
✓ push: 添加到历史栈，用户可点返回按钮返回
✓ replace: 替换当前记录，用户无法返回到前一个路由
✗ 错误: 不区分，导致用户体验差异

应用: 登录页通常用 replace，防止登录后返回登录页
```

---

## 第六章 命名路由与命名视图

### 6.1 命名路由

```typescript
// 定义命名路由
const routes = [
    {
        path: '/home',
        name: 'home',  // 给路由取名
        component: Home
    },
    {
        path: '/home/message/:filmId',
        name: 'detail',
        component: Detail
    }
]

// 使用命名路由
<router-link :to="{ name: 'detail', params: { filmId: 1 } }">
    Detail
</router-link>

// 编程式导航
$router.push({ name: 'detail', params: { filmId: 1 } })
```

### 6.2 命名视图

```typescript
// 一个路由对应多个组件
const routes = [
    {
        path: '/layout',
        components: {
            default: LayoutMain,
            sidebar: LayoutSidebar,
            footer: LayoutFooter
        }
    }
]
```

```vue
<!-- 模板中使用多个 router-view -->
<template>
    <router-view name="sidebar"></router-view>
    <router-view></router-view>  <!-- default -->
    <router-view name="footer"></router-view>
</template>
```

---

## 第七章 路由懒加载

### 7.1 动态导入

```typescript
// 普通导入 - 所有组件都打包到一个文件
import Home from '@/pages/Home/index.vue'

// 懒加载 - 按需加载，拆分成多个文件
const Home = () => import('@/pages/Home/index.vue')

// 完整配置
const routes = [
    {
        path: '/home',
        component: () => import('@/pages/Home/index.vue')
    },
    {
        path: '/about',
        component: () => import('@/pages/About/index.vue')
    }
]
```

### 7.2 路由懒加载 + 异步组件加载流程 - ASCII 图 2:

```
┌──────────────────────────────────────────────────┐
│    路由懒加载 + 异步组件加载流程                  │
├──────────────────────────────────────────────────┤
│                                                  │
│  用户导航到路由                                 │
│  ↓                                              │
│  Vue Router 检查组件是否加载                    │
│  ├─ 已加载 ──→ 直接展示                        │
│  └─ 未加载 ──↓                                 │
│                                                  │
│  执行 () => import(...) 动态导入                │
│  ↓                                              │
│  Webpack/Vite 开始打包拆分的 JS 文件            │
│  ↓                                              │
│  浏览器网络请求该文件                          │
│  ├─ 加载中: 显示 loading 或 Suspense fallback │
│  ├─ 加载失败: 显示错误页面                     │
│  └─ 加载成功 ──↓                              │
│                                                  │
│  Promise resolve，获得组件定义                 │
│  ↓                                              │
│  渲染组件，显示内容                            │
│  ↓                                              │
│  组件缓存，后续导航不再加载                    │
│                                                  │
└──────────────────────────────────────────────────┘
```

### 7.3 与 Suspense 配合

```vue
<template>
    <Suspense>
        <template #default>
            <router-view />
        </template>
        <template #fallback>
            <div>加载中...</div>
        </template>
    </Suspense>
</template>
```

---

## 第八章 导航守卫

### 8.1 全局前置守卫

```typescript
// beforeEach: 每次路由切换前触发
router.beforeEach((to, from, next) => {
    // to: 要跳转到的路由对象
    // from: 从哪个路由来
    // next: 是否继续导航
    
    if (to.path === '/about') {
        // 检查权限
        const hasPermission = checkPermission()
        if (hasPermission) {
            next()  // 继续导航
        } else {
            next(false)  // 取消导航
        }
    } else {
        next()
    }
})

// afterEach: 路由切换后触发
router.afterEach((to, from) => {
    console.log('路由已切换')
})
```

**常见纠错 03:**
```
问题: beforeEach 中调用 next() 多次
✗ 错误: next(); next('/home');  // 调用多次
✓ 正确: 只调用一次 next()

问题: beforeEach 死循环
✗ 错误: 在守卫中跳转到同一个路由
        router.beforeEach(() => {
            router.push('/home')  // 死循环!
        })
✓ 正确: 使用条件判断，避免循环导航
```

### 8.2 路由级别守卫

```typescript
// beforeEnter: 只有进入该路由时触发
const routes = [
    {
        path: '/about',
        component: About,
        beforeEnter: (to, from, next) => {
            console.log('进入 about 路由')
            next()
        }
    }
]
```

### 8.3 组件内守卫

```typescript
// setup 中使用
import { onBeforeRouteUpdate, useRoute } from 'vue-router'

export default {
    setup() {
        // 在组件重用时 (参数变化) 触发
        onBeforeRouteUpdate(async (to, from, next) => {
            // 处理参数变化，重新获取数据
            next()
        })
    }
}

// Options API 中使用
export default {
    beforeRouteEnter(to, from, next) {
        // 进入组件前触发
        // 注意: 此时 this 还未定义!
        next((vm) => {
            // vm 是组件实例，现在可以使用 this
        })
    },
    beforeRouteUpdate(to, from, next) {
        // 路由参数变化时触发
        next()
    },
    beforeRouteLeave(to, from, next) {
        // 离开组件前触发
        next()
    }
}
```

**常见纠错 04:**
```
问题: beforeRouteEnter 中使用 this
✗ 错误: 在 beforeRouteEnter 中直接用 this
      beforeRouteEnter(to, from, next) {
          console.log(this.xxx)  // undefined!
      }

✓ 正确: 使用 next 回调函数
      beforeRouteEnter(to, from, next) {
          next((vm) => {
              console.log(vm.xxx)  // 现在可以了
          })
      }

原因: beforeRouteEnter 在组件挂载前执行，
      此时组件实例还未创建，所以 this 为 undefined
```

---

## 第九章 完整导航解析流程 (12 步) - ASCII 图 3:

```
┌────────────────────────────────────────────────────────┐
│      Vue Router 完整导航解析流程 (12 步)              │
├────────────────────────────────────────────────────────┤
│                                                        │
│  1. 用户点击 router-link 或调用 router.push()        │
│                                                        │
│  2. 执行旧组件的 beforeRouteLeave 守卫               │
│     └─ 可在此提示用户："确定离开? 有未保存数据"    │
│                                                        │
│  3. 执行全局 beforeEach 守卫                         │
│     └─ 常用于验证权限、token 等                     │
│                                                        │
│  4. 在被激活的新组件中执行 beforeRouteEnter 守卫    │
│     └─ 获取新组件数据                               │
│                                                        │
│  5. 在被激活的新组件中执行 beforeRouteUpdate 守卫  │
│     └─ 如果是重用组件，参数变化时触发              │
│                                                        │
│  6. 在新路由配置中的 beforeEnter 守卫               │
│     └─ 路由级别的守卫                               │
│                                                        │
│  7. 确认所有守卫都通过                               │
│                                                        │
│  8. 如果新组件配置了 component 或使用了异步组件    │
│     则进行异步组件加载                              │
│                                                        │
│  9. 激活被激活的视图 (调用组件的 setup/data)       │
│                                                        │
│  10. 将新组件挂载到 DOM (mount)                      │
│      └─ 触发 onMounted/mounted 钩子                 │
│                                                        │
│  11. 执行全局 afterEach 守卫                         │
│      └─ 通常用于结束 loading、清除状态等           │
│                                                        │
│  12. 导航成功，DOM 已更新                            │
│      └─ 用户看到新路由对应的内容                   │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## 第十章 路由元信息与滚动行为

### 10.1 路由元信息 (meta)

```typescript
const routes = [
    {
        path: '/admin',
        component: Admin,
        meta: {
            requiresAuth: true,  // 需要认证
            title: 'Admin Panel',
            roles: ['admin']  // 需要的角色
        }
    }
]

// 在守卫中使用
router.beforeEach((to, from, next) => {
    if (to.meta.requiresAuth) {
        // 检查用户是否已认证
        if (checkAuth()) {
            next()
        } else {
            next('/login')
        }
    } else {
        next()
    }
})
```

### 10.2 滚动行为

```typescript
const router = createRouter({
    history: createWebHistory(),
    routes,
    scrollBehavior(to, from, savedPosition) {
        // savedPosition: 如果是点返回按钮，返回之前的滚动位置
        if (savedPosition) {
            return savedPosition
        }
        // 否则滚到顶部
        return { left: 0, top: 0 }
    }
})
```

---

## 第十一章 动态添加/删除路由

### 11.1 添加路由

```typescript
import router from '@/router'

// 添加顶级路由
router.addRoute({
    path: '/new',
    name: 'new',
    component: () => import('@/pages/New.vue')
})

// 添加子路由 (必须指定父路由的 name)
router.addRoute('parentName', {
    path: '/new',
    component: () => import('@/pages/New.vue')
})
```

### 11.2 删除路由

```typescript
// 方案 1: 使用 removeRoute
if (router.hasRoute('admin')) {
    router.removeRoute('admin')
}

// 方案 2: 使用 addRoute 的返回值
const removeRoute = router.addRoute({
    path: '/temp',
    component: Temp
})
removeRoute()  // 调用返回函数删除
```

### 11.3 权限路由实现

```typescript
// 根据用户权限动态添加路由
import router from '@/router'

const asyncRoutes = [
    {
        path: '/admin',
        name: 'admin',
        component: () => import('@/pages/Admin.vue')
    }
]

// 用户登录后获取权限信息
function setupRoutes(userPermissions) {
    // 过滤出用户有权限的路由
    const routes = asyncRoutes.filter(route => {
        return userPermissions.includes(route.name)
    })
    
    // 动态添加路由
    routes.forEach(route => {
        router.addRoute(route)
    })
}

// 登录成功时调用
setupRoutes(userInfo.permissions)
```

---

## 第十二章 Vue Router 4 与 3 的差异

| 差异项 | Vue Router 3 | Vue Router 4 |
|--------|-------------|-------------|
| 404 处理 | path: "*" | path: "/:pathMatch(.*)*" |
| History 创建 | mode: "history" | history: createWebHistory() |
| 路由懒加载 | component: () => import() | 同左 |
| 获取路由对象 | this.$route | useRoute() |
| 获取路由实例 | this.$router | useRouter() |
| 导航守卫 | 全局/路由/组件三种 | 同左，但用法略不同 |

---

## 第十三章 与 keep-alive 配合

```vue
<template>
    <!-- 将已加载的组件缓存 -->
    <keep-alive>
        <router-view />
    </keep-alive>
</template>

<!-- 可加 include/exclude 指定缓存哪些组件 -->
<keep-alive include="Home,About" exclude="Temp">
    <router-view />
</keep-alive>
```

**配置组件名:**
```typescript
export default {
    name: 'Home',  // keep-alive 会根据这个名字匹配
    // ...
}
```

**keep-alive 生命周期:**
```typescript
export default {
    setup() {
        // 组件第一次加载时触发
        onMounted(() => {
            console.log('组件挂载')
        })
        
        // 组件从缓存激活时触发
        onActivated(() => {
            console.log('组件从缓存激活')
        })
        
        // 组件进入缓存时触发
        onDeactivated(() => {
            console.log('组件进入缓存')
        })
    }
}
```

---

## 易错速查

| 编号 | 错误 | 正确做法 | 原因 |
|-----|------|--------|------|
| 01 | History 模式刷新 404 | 后端配置重定向到 index.html | 后端需支持前端路由 |
| 02 | push 和 replace 混淆 | push 有历史记录，replace 无 | 用户体验差异 |
| 03 | beforeEach 调用 next 多次 | 只调用一次 | 多次调用导致错误 |
| 04 | beforeRouteEnter 中用 this | 使用 next 回调函数 | 此时组件未挂载 |
| 05 | 路由懒加载失败处理 | 使用 error 字段或 Suspense | 异步加载需容错 |

---

**文档生成日期: 2026-05-25**
