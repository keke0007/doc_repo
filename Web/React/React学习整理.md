# React 学习整理

这份笔记基于 `React` 目录下的 13 份 Markdown 归纳整理，目标不是简单拼接原文，而是按 React 的学习路径重新组织内容，并给每个核心知识点补上最小可运行示例，方便你系统学习和回顾。

## 学习路线

建议按下面的顺序学习：

1. 先理解 React 是什么、声明式开发是什么、为什么要组件化
2. 再掌握 JSX、事件、条件渲染、列表渲染
3. 接着学习类组件、函数组件、生命周期、组件通信
4. 然后学习表单、ref、Context、性能优化、Portal、StrictMode
5. 再学习 Hooks
6. 然后进入 Redux、React Router
7. 最后补 CSS 方案、动画、项目实战和工程化

## 原始资料对应关系

| 主题 | 原文件 |
|---|---|
| React 入门与地位 | `01_`、`02_` |
| JSX 语法 | `03_React的JSX语法解析.md` |
| 脚手架与工程化 | `04_React脚手架解析.md` |
| 组件化开发 | `05_`、`06_` |
| 动画与样式 | `07_`、`08_` |
| Redux | `09_`、`10_` |
| React Router | `11_React-Router路由.md` |
| Hooks | `12_React-Hooks解析.md` |
| 项目实战 | `13_项目实战-爱彼迎.md` |

## 一、认识 React

应用场景：

- 用在建立 React 项目的整体认知，知道它适合开发什么类型的前端应用。
- 适合后台管理、内容社区、商城前台、跨端生态项目和组件化团队协作开发。

### 1. React 是什么

React 是一个用于构建用户界面的 JavaScript 库。

可以先记住 React 的几个关键词：

- 声明式
- 组件化
- 单向数据流
- 多平台适配

### 2. 声明式开发

React 更关注“我想要什么界面”，而不是一步步操作 DOM。

```jsx
function App() {
  const message = "Hello React"
  return <h2>{message}</h2>
}
```

### 3. 为什么学 React

- React 生态非常成熟
- 在企业项目中很常见
- Hooks 写法已经成为主流
- React Native、Next.js 等生态延展能力强

## 二、搭建 React 开发环境

应用场景：

- 用在初始化 React 项目、选择脚手架、搭建开发环境和理解工程化流程时。
- 适合从 demo 过渡到正式项目，建立模块化、编译、构建、热更新的基础认知。

### 1. 最简单的 CDN 体验

```html
<div id="root"></div>

<script src="https://unpkg.com/react@18/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

<script type="text/babel">
  function App() {
    return <h2>Hello React</h2>
  }

  const root = ReactDOM.createRoot(document.getElementById("root"))
  root.render(<App />)
</script>
```

### 2. create-react-app / Vite

现代开发更常见的是脚手架。

```bash
npx create-react-app my-app
```

或：

```bash
npm create vite@latest
```

### 3. 工程化的意义

脚手架主要帮我们解决：

- 模块化
- JSX 编译
- 开发服务器
- 打包构建
- 目录结构组织

## 三、JSX 语法

应用场景：

- 用在把数据渲染到界面、绑定属性和事件、控制条件显示、渲染列表时。
- 适合商品列表、卡片页、表格、按钮交互、详情页这类日常页面开发。

### 1. JSX 是什么

JSX 是 JavaScript 的语法扩展，看起来像 HTML，但本质上会被编译成 `React.createElement`。

```jsx
const element = <h2>Hello JSX</h2>
```

### 2. JSX 中插入表达式

```jsx
function App() {
  const name = "Tom"
  const age = 18
  return <h2>{name} - {age}</h2>
}
```

### 3. JSX 中绑定属性

```jsx
function App() {
  const imgUrl = "https://via.placeholder.com/120"
  return <img src={imgUrl} alt="示例图片" />
}
```

注意：

- `class` 要写成 `className`
- 行内样式要写成对象

```jsx
function App() {
  return (
    <div className="box" style={{ color: "red", fontSize: "20px" }}>
      Hello
    </div>
  )
}
```

### 4. JSX 绑定事件

```jsx
function App() {
  function handleClick() {
    console.log("按钮点击")
  }

  return <button onClick={handleClick}>点击我</button>
}
```

### 5. 事件传参

```jsx
function App() {
  function handleItemClick(id) {
    console.log("当前点击：", id)
  }

  return <button onClick={() => handleItemClick(100)}>查看详情</button>
}
```

### 6. 条件渲染

```jsx
function App() {
  const isLogin = true

  return (
    <div>
      {isLogin ? <h2>欢迎回来</h2> : <h2>请先登录</h2>}
    </div>
  )
}
```

### 7. 列表渲染

```jsx
function App() {
  const books = [
    { id: 1, name: "React" },
    { id: 2, name: "Redux" }
  ]

  return (
    <ul>
      {books.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  )
}
```

### 8. key 的作用

`key` 用来帮助 React 更准确地识别列表项变化。

建议：

- 使用稳定唯一值
- 不要随手长期用索引

## 四、组件化开发

应用场景：

- 用在把页面拆成头部、侧边栏、卡片、弹窗、表单块等可复用模块时。
- 适合团队协作开发和中大型项目，减少重复代码，让页面结构更清晰。

### 1. 函数组件

```jsx
function Hello() {
  return <h2>Hello Function Component</h2>
}
```

### 2. 类组件

```jsx
import React from "react"

class Hello extends React.Component {
  render() {
    return <h2>Hello Class Component</h2>
  }
}
```

### 3. state

类组件中通过 `this.state` 管理状态。

```jsx
import React from "react"

class Counter extends React.Component {
  constructor() {
    super()
    this.state = {
      count: 0
    }
  }

  increment = () => {
    this.setState({ count: this.state.count + 1 })
  }

  render() {
    return (
      <div>
        <h2>{this.state.count}</h2>
        <button onClick={this.increment}>+1</button>
      </div>
    )
  }
}
```

### 4. props

```jsx
function Header(props) {
  return <h2>{props.title}</h2>
}

function App() {
  return <Header title="React 组件传值" />
}
```

### 5. prop-types

```jsx
import PropTypes from "prop-types"

function Product(props) {
  return <h3>{props.name}</h3>
}

Product.propTypes = {
  name: PropTypes.string.isRequired
}
```

### 6. defaultProps

```jsx
function Banner(props) {
  return <h2>{props.title}</h2>
}

Banner.defaultProps = {
  title: "默认标题"
}
```

## 五、生命周期

应用场景：

- 用在组件挂载后请求数据、更新后同步外部状态、卸载前清理定时器或订阅时。
- 适合图表初始化、事件监听、轮询、资源释放、页面切换清理等场景。

### 1. componentDidMount

适合：

- 操作 DOM
- 发送网络请求
- 添加事件监听

```jsx
class App extends React.Component {
  componentDidMount() {
    console.log("组件挂载完成")
  }

  render() {
    return <h2>Hello</h2>
  }
}
```

### 2. componentDidUpdate

```jsx
class App extends React.Component {
  componentDidUpdate(prevProps, prevState) {
    console.log("更新前：", prevState)
    console.log("更新后：", this.state)
  }
}
```

### 3. componentWillUnmount

```jsx
class App extends React.Component {
  componentWillUnmount() {
    console.log("组件即将卸载")
  }
}
```

## 六、组件通信

应用场景：

- 用在父子组件传值、触发回调、插入子内容、跨层级共享全局数据时。
- 适合弹窗开关、列表项操作、主题切换、用户信息共享等功能。

### 1. 父传子

```jsx
function Child(props) {
  return <p>{props.message}</p>
}

function App() {
  return <Child message="父组件传来的消息" />
}
```

### 2. 子传父

```jsx
function Child(props) {
  return <button onClick={() => props.onAdd(10)}>传给父组件</button>
}

function App() {
  function handleAdd(num) {
    console.log("收到：", num)
  }

  return <Child onAdd={handleAdd} />
}
```

### 3. children

```jsx
function Card(props) {
  return <div className="card">{props.children}</div>
}

function App() {
  return (
    <Card>
      <h3>卡片标题</h3>
      <p>卡片内容</p>
    </Card>
  )
}
```

### 4. Context

```jsx
import React from "react"

const ThemeContext = React.createContext("light")

function Child() {
  return (
    <ThemeContext.Consumer>
      {value => <p>当前主题：{value}</p>}
    </ThemeContext.Consumer>
  )
}

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Child />
    </ThemeContext.Provider>
  )
}
```

## 七、setState 与更新机制

应用场景：

- 用在组件状态更新、连续点击计数、异步批量更新和依赖旧状态计算新状态时。
- 适合表单输入、开关切换、局部 UI 刷新和理解 React 更新节奏。

### 1. 不能直接改 state

错误示例：

```jsx
this.state.count = 10
```

正确写法：

```jsx
this.setState({
  count: 10
})
```

### 2. setState 回调

```jsx
this.setState(
  { count: this.state.count + 1 },
  () => {
    console.log("更新后：", this.state.count)
  }
)
```

### 3. 函数式 setState

```jsx
this.setState(prevState => {
  return {
    count: prevState.count + 1
  }
})
```

## 八、性能优化与高级组件能力

应用场景：

- 用在减少无意义渲染、访问 DOM、透传 ref、跨层级挂载弹窗时。
- 适合大列表、复杂表单、模态框、性能敏感页面和组件库封装。

### 1. shouldComponentUpdate

```jsx
shouldComponentUpdate(nextProps, nextState) {
  return nextState.count !== this.state.count
}
```

### 2. PureComponent

```jsx
import React, { PureComponent } from "react"

class Counter extends PureComponent {
  render() {
    return <h2>{this.props.count}</h2>
  }
}
```

### 3. React.memo

```jsx
import React, { memo } from "react"

const Button = memo(function Button(props) {
  console.log("重新渲染")
  return <button>{props.title}</button>
})
```

### 4. ref

```jsx
import React, { createRef } from "react"

class App extends React.Component {
  inputRef = createRef()

  focusInput = () => {
    this.inputRef.current.focus()
  }

  render() {
    return (
      <div>
        <input ref={this.inputRef} />
        <button onClick={this.focusInput}>聚焦</button>
      </div>
    )
  }
}
```

### 5. forwardRef

```jsx
import React, { forwardRef } from "react"

const MyInput = forwardRef((props, ref) => {
  return <input ref={ref} />
})
```

### 6. Portal

```jsx
import ReactDOM from "react-dom"

function Modal() {
  return ReactDOM.createPortal(
    <div className="modal">这是弹窗</div>,
    document.getElementById("modal-root")
  )
}
```

### 7. StrictMode

```jsx
import React from "react"

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

## 九、受控组件与非受控组件

应用场景：

- 用在处理输入框、搜索框、表单提交、上传组件和第三方表单库接入时。
- 适合需要校验、联动、即时预览的表单优先用受控组件，简单取值场景可用非受控。

### 1. 受控组件

```jsx
import React, { useState } from "react"

function App() {
  const [username, setUsername] = useState("")

  return (
    <input
      value={username}
      onChange={e => setUsername(e.target.value)}
    />
  )
}
```

### 2. 非受控组件

```jsx
import React, { useRef } from "react"

function App() {
  const inputRef = useRef()

  function handleSubmit() {
    console.log(inputRef.current.value)
  }

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={handleSubmit}>提交</button>
    </div>
  )
}
```

## 十、高阶组件 HOC

应用场景：

- 用在给多个组件统一加权限校验、日志埋点、主题能力、数据注入时。
- 适合老项目复用横切逻辑，或理解 React 早期高级复用模式。

### 1. 基本定义

高阶组件是一个函数，接收组件，返回新组件。

```jsx
function withAuth(WrappedComponent) {
  return function(props) {
    const isLogin = true
    if (!isLogin) return <h2>请先登录</h2>
    return <WrappedComponent {...props} />
  }
}
```

### 2. 使用 HOC

```jsx
function Profile() {
  return <h2>个人中心</h2>
}

const AuthProfile = withAuth(Profile)
```

## 十一、React Hooks

应用场景：

- 用在函数组件里管理状态、副作用、上下文、性能优化和逻辑复用时。
- 适合现代 React 业务开发，是写列表页、详情页、表单页、弹窗页的核心工具集。

### 1. useState

```jsx
import { useState } from "react"

function Counter() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  )
}
```

### 2. useEffect

```jsx
import { useEffect, useState } from "react"

function App() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    console.log("effect 执行")
  }, [count])

  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

### 3. 清除副作用

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    console.log("timer running")
  }, 1000)

  return () => {
    clearInterval(timer)
  }
}, [])
```

### 4. useContext

```jsx
import { createContext, useContext } from "react"

const ThemeContext = createContext("light")

function Child() {
  const theme = useContext(ThemeContext)
  return <h2>{theme}</h2>
}
```

### 5. useReducer

```jsx
import { useReducer } from "react"

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 }
    default:
      return state
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, { count: 0 })

  return (
    <button onClick={() => dispatch({ type: "increment" })}>
      {state.count}
    </button>
  )
}
```

### 6. useCallback

```jsx
import { useCallback, useState } from "react"

function App() {
  const [count, setCount] = useState(0)

  const handleClick = useCallback(() => {
    console.log("clicked")
  }, [])

  return <button onClick={handleClick}>{count}</button>
}
```

### 7. useMemo

```jsx
import { useMemo, useState } from "react"

function App() {
  const [count, setCount] = useState(0)

  const doubleCount = useMemo(() => count * 2, [count])

  return <button onClick={() => setCount(count + 1)}>{doubleCount}</button>
}
```

### 8. useRef

```jsx
import { useRef } from "react"

function App() {
  const inputRef = useRef()

  return <input ref={inputRef} />
}
```

### 9. useLayoutEffect

```jsx
import { useLayoutEffect, useRef } from "react"

function App() {
  const boxRef = useRef()

  useLayoutEffect(() => {
    console.log(boxRef.current.getBoundingClientRect())
  }, [])

  return <div ref={boxRef}>box</div>
}
```

### 10. 自定义 Hook

```jsx
import { useEffect, useState } from "react"

function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  })

  useEffect(() => {
    function handleResize() {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      })
    }

    window.addEventListener("resize", handleResize)
    return () => window.removeEventListener("resize", handleResize)
  }, [])

  return size
}
```

### 11. useId

```jsx
import { useId } from "react"

function App() {
  const id = useId()

  return (
    <div>
      <label htmlFor={id}>用户名</label>
      <input id={id} />
    </div>
  )
}
```

### 12. useTransition

```jsx
import { useState, useTransition } from "react"

function App() {
  const [isPending, startTransition] = useTransition()
  const [text, setText] = useState("")

  function handleChange(e) {
    startTransition(() => {
      setText(e.target.value)
    })
  }

  return (
    <div>
      <input onChange={handleChange} />
      {isPending && <p>加载中...</p>}
      <p>{text}</p>
    </div>
  )
}
```

### 13. useDeferredValue

```jsx
import { useDeferredValue, useState } from "react"

function App() {
  const [keyword, setKeyword] = useState("")
  const deferredKeyword = useDeferredValue(keyword)

  return (
    <div>
      <input value={keyword} onChange={e => setKeyword(e.target.value)} />
      <p>{deferredKeyword}</p>
    </div>
  )
}
```

## 十二、React 中的 CSS 方案

应用场景：

- 用在给组件添加样式、隔离样式作用域、按模块维护样式文件时。
- 适合普通业务页、组件库、主题切换和不同团队协作风格下的样式管理。

### 1. 行内样式

```jsx
function App() {
  return <div style={{ color: "red", fontSize: "20px" }}>Hello</div>
}
```

### 2. 普通 CSS

```jsx
import "./App.css"

function App() {
  return <h2 className="title">Hello React</h2>
}
```

### 3. CSS Modules

```jsx
import styles from "./App.module.css"

function App() {
  return <h2 className={styles.title}>Hello React</h2>
}
```

### 4. styled-components

```jsx
import styled from "styled-components"

const Title = styled.h2`
  color: #42b883;
  font-size: 24px;
`

function App() {
  return <Title>Hello Styled</Title>
}
```

## 十三、过渡动画

应用场景：

- 用在弹窗出现消失、列表项增删、页面切换、提示消息动画等交互里。
- 适合提升后台系统和内容站点的体验，但应控制复杂度避免影响性能。

### 1. CSSTransition

```jsx
import { CSSTransition } from "react-transition-group"
import { useState } from "react"

function App() {
  const [show, setShow] = useState(true)

  return (
    <div>
      <button onClick={() => setShow(!show)}>切换</button>
      <CSSTransition in={show} timeout={300} classNames="fade" unmountOnExit>
        <div className="box">Hello</div>
      </CSSTransition>
    </div>
  )
}
```

### 2. TransitionGroup

```jsx
import { TransitionGroup, CSSTransition } from "react-transition-group"

function List({ list }) {
  return (
    <TransitionGroup component="ul">
      {list.map(item => (
        <CSSTransition key={item.id} timeout={300} classNames="item">
          <li>{item.name}</li>
        </CSSTransition>
      ))}
    </TransitionGroup>
  )
}
```

## 十四、Redux

应用场景：

- 用在多个页面共享用户信息、购物车、权限、筛选器、缓存数据等全局状态时。
- 适合大型项目统一状态流转，特别是状态变化复杂、异步逻辑较多的场景。

### 1. Redux 三要素

- store
- action
- reducer

### 2. 基础 Redux

```js
import { createStore } from "redux"

const initialState = {
  count: 0
}

function reducer(state = initialState, action) {
  switch (action.type) {
    case "increment":
      return { ...state, count: state.count + 1 }
    default:
      return state
  }
}

const store = createStore(reducer)

store.dispatch({ type: "increment" })
console.log(store.getState())
```

### 3. react-redux

```jsx
import { Provider, useDispatch, useSelector } from "react-redux"

function Counter() {
  const count = useSelector(state => state.count)
  const dispatch = useDispatch()

  return (
    <button onClick={() => dispatch({ type: "increment" })}>
      {count}
    </button>
  )
}
```

### 4. redux-thunk

```js
function fetchHomeData() {
  return async function(dispatch) {
    const res = await fetch("/api/home")
    const data = await res.json()
    dispatch({ type: "setHomeData", payload: data })
  }
}
```

### 5. Redux Toolkit

```js
import { configureStore, createSlice } from "@reduxjs/toolkit"

const counterSlice = createSlice({
  name: "counter",
  initialState: { count: 0 },
  reducers: {
    increment(state) {
      state.count += 1
    }
  }
})

export const { increment } = counterSlice.actions

const store = configureStore({
  reducer: {
    counter: counterSlice.reducer
  }
})

export default store
```

## 十五、React Router

应用场景：

- 用在页面导航、详情页跳转、动态路由、嵌套路由和权限拦截里。
- 适合后台管理、博客、商城、文档站这类典型单页应用。

### 1. 基本路由配置

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom"

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  )
}
```

### 2. Link / NavLink

```jsx
import { Link, NavLink } from "react-router-dom"

function Nav() {
  return (
    <nav>
      <Link to="/">首页</Link>
      <NavLink to="/about">关于</NavLink>
    </nav>
  )
}
```

### 3. useNavigate

```jsx
import { useNavigate } from "react-router-dom"

function Login() {
  const navigate = useNavigate()

  function handleLogin() {
    navigate("/profile")
  }

  return <button onClick={handleLogin}>登录</button>
}
```

### 4. useParams

```jsx
import { useParams } from "react-router-dom"

function User() {
  const params = useParams()
  return <h2>用户 ID：{params.id}</h2>
}
```

### 5. 查询参数

```jsx
import { useSearchParams } from "react-router-dom"

function SearchPage() {
  const [searchParams] = useSearchParams()
  return <h2>keyword: {searchParams.get("keyword")}</h2>
}
```

### 6. 路由嵌套

```jsx
<Routes>
  <Route path="/home" element={<Home />}>
    <Route path="recommend" element={<Recommend />} />
    <Route path="ranking" element={<Ranking />} />
  </Route>
</Routes>
```

## 十六、项目实战思路

应用场景：

- 用在把组件化、路由、状态管理、请求、样式、权限等知识真正串成完整项目。
- 适合从零散知识点过渡到独立完成中小型 React 应用开发。

做 React 项目时，建议重点关注：

1. 目录结构划分
2. 状态管理方案
3. 路由拆分
4. 网络请求封装
5. 公共组件抽离
6. 样式方案统一
7. 性能优化

一个常见结构：

```text
src
├─ assets
├─ components
├─ views
├─ router
├─ store
├─ service
├─ hooks
├─ utils
└─ App.jsx
```

## 十七、这套 React 内容的核心学习重点

如果时间有限，优先掌握：

### 第一优先级

- JSX
- 组件化与组件通信
- state / props / 生命周期
- Hooks
- React Router
- Redux Toolkit

### 第二优先级

- Context
- React.memo / PureComponent
- ref / forwardRef / Portal
- CSS Modules / styled-components
- 过渡动画

### 第三优先级

- Redux 原始写法
- 高阶组件
- 更深入的 Hook 和工程化补充

## 十八、容易混淆的知识点

### 1. state 和 props

- state：组件自己维护的数据
- props：父组件传进来的数据

### 2. 类组件和函数组件

- 类组件有传统生命周期
- 函数组件现在通常配合 Hooks 使用

### 3. 受控组件和非受控组件

- 受控组件：值由 React state 控制
- 非受控组件：值由 DOM 自己维护

### 4. useEffect 和 useLayoutEffect

- `useEffect` 更常用，异步执行副作用
- `useLayoutEffect` 更接近 DOM 更新后、浏览器绘制前

### 5. Redux 和 Context

- Context 更适合轻量级共享
- Redux 更适合复杂全局状态管理

## 十九、建议你的复习方式

### 第一轮

- 顺着这份文档走一遍
- 能说清楚每个知识点解决什么问题

### 第二轮

- 手敲每个核心示例
- 特别练：JSX、Hooks、Router、Redux Toolkit

### 第三轮

- 自己做小案例
- 例如：计数器、TodoList、登录页、音乐列表、商品列表

### 第四轮

- 自己搭一个小项目
- 用上 Router + Redux Toolkit + axios 封装 + 组件拆分

## 二十、一个最小知识树

```text
React
├─ 基础
│  ├─ JSX
│  ├─ state / props
│  ├─ 事件绑定
│  ├─ 条件渲染
│  └─ 列表渲染
├─ 组件化
│  ├─ 类组件
│  ├─ 函数组件
│  ├─ 生命周期
│  ├─ children / Context
│  └─ ref / Portal
├─ Hooks
│  ├─ useState
│  ├─ useEffect
│  ├─ useReducer
│  ├─ useMemo / useCallback
│  ├─ useRef
│  └─ 自定义 Hook
├─ 生态
│  ├─ Redux / Redux Toolkit
│  ├─ React Router
│  └─ CSS 方案
└─ 工程实践
   ├─ 脚手架
   ├─ 项目结构
   ├─ 动画
   └─ 项目实战
```

## 二十一、学完这部分后的目标

学完这部分后，你至少应该能做到：

- 用 React 独立写中小型页面
- 熟练使用 JSX 和组件拆分页面
- 理解 state、props、Hooks 的配合
- 用 Router 实现页面跳转
- 用 Redux Toolkit 管理共享状态
- 能写常见表单、列表、弹窗、卡片类组件
- 能看懂并维护常见 React 项目结构

## 二十二、章节练习题

### 1. JSX 与基础渲染

练习：

1. 写一个组件，页面上展示你的名字、年龄和一句介绍。
2. 准备一个图片地址数组，渲染出 3 张图片。
3. 实现一个按钮，点击后把标题从“Hello React”改成“你好 React”。

参考方向：

- 练 `{}`
- 练 `map`
- 练事件绑定和状态更新

### 2. 组件化与通信

练习：

1. 封装一个 `ProductItem` 组件，接收 `title`、`price` 两个 props。
2. 封装一个 `CounterButton` 子组件，点击时通知父组件增加数量。
3. 封装一个 `Card` 组件，使用 `children` 自定义中间内容。

参考方向：

- 练 props
- 练父传子、子传父
- 练 `children`

### 3. 生命周期与副作用

练习：

1. 类组件中在 `componentDidMount` 里打印一条日志。
2. 用函数组件 + `useEffect` 模拟 mounted 行为。
3. 给页面绑定一个 `resize` 事件，在组件卸载时移除它。

参考方向：

- 对比类组件生命周期和 Hooks
- 理解副作用和清理函数

### 4. 表单与 ref

练习：

1. 实现一个受控输入框，实时展示输入内容。
2. 实现一个非受控输入框，点击按钮时读取 DOM 中的值。
3. 实现一个“点击按钮自动聚焦输入框”的小例子。

### 5. Hooks

练习：

1. 用 `useState` 写一个计数器。
2. 用 `useReducer` 重写这个计数器。
3. 用 `useMemo` 计算一个列表过滤结果。
4. 写一个 `useTitle` Hook，根据传入标题更新网页标题。

示例：

```jsx
import { useEffect } from "react"

function useTitle(title) {
  useEffect(() => {
    document.title = title
  }, [title])
}
```

### 6. Router

练习：

1. 配置首页、关于页、用户页三个路由。
2. 用户页通过动态路由拿到 `id`。
3. 登录按钮点击后跳转到 `/profile`。
4. 写一个 404 页面。

### 7. Redux Toolkit

练习：

1. 创建一个 `counter` store，支持加一减一。
2. 再创建一个 `user` store，保存用户名。
3. 组件里同时读取两个 slice 的数据。
4. 用异步 thunk 请求一个列表数据。

### 8. CSS 与动画

练习：

1. 分别用普通 CSS、CSS Modules、styled-components 写一个按钮。
2. 用 `CSSTransition` 实现一个淡入淡出的提示框。
3. 用 `TransitionGroup` 实现一个待办事项列表的添加动画。

## 二十三、综合小项目路线

### 1. 入门项目：计数器 + TodoList

目标：

- 学会 `useState`
- 学会列表渲染
- 学会事件处理

建议功能：

- 新增任务
- 删除任务
- 切换完成状态

### 2. 进阶项目：文章列表页

目标：

- 学会组件拆分
- 学会 props 传值
- 学会条件渲染和分页展示

建议功能：

- 文章卡片组件
- 标签筛选
- 加载更多

### 3. 完整项目：后台管理 / 商城前台

目标：

- 学会 Router
- 学会 Redux Toolkit
- 学会请求封装
- 学会公共布局拆分

建议模块：

- 登录页
- 首页
- 列表页
- 详情页
- 个人中心

## 二十四、自测问题

1. JSX 为什么不能直接写多个根节点？常见解决方案有哪些？
2. `state` 和 `props` 的本质区别是什么？
3. 为什么列表渲染要加 `key`？
4. 类组件生命周期里，发送请求通常放在哪个阶段？
5. `useEffect` 的第二个参数分别传 `[]`、不传、传 `[count]` 有什么区别？
6. 受控组件和非受控组件分别适合什么场景？
7. `useMemo` 和 `useCallback` 的区别是什么？
8. Redux Toolkit 相比原始 Redux 简化了哪些内容？
9. `BrowserRouter` 和 `HashRouter` 的区别是什么？
10. Portal 解决了什么问题？
