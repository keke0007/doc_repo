# 15 - 使用 TypeScript 改造项目(整理版)

> 源文档:`typescript笔记/05-使用Typescript改造项目.md`

---

## 一、知识点清单

### 1. 改造总策略

- **新代码用 TS,老代码逐步迁移**(不要一次性大改)
- 渐进迁移:开启 `allowJs:true`,让 `.js` 与 `.ts` 并存
- 文件后缀:
  - 含 JSX 的 React 组件 → **`.tsx`**
  - 纯逻辑/工具函数 → `.ts`,或保留 `.js` + 补 `.d.ts`
- 启动顺序:① 改 `index` 与 `App` 入口 → ② 工具函数 → ③ 通用组件 → ④ 业务组件 → ⑤ Redux

### 2. 入口与第三方类型

- 安装常用类型包(devDeps):
  ```bash
  npm i -D @types/react @types/react-dom @types/react-router-dom @types/node @types/jest
  ```
- 没有官方类型的库 → 看 https://www.typescriptlang.org/dt 找 `@types/xxx`

### 3. 非空断言 `!`

```ts
const v = localStorage.getItem(K)!         // 告诉 TS:这里一定不为 null
JSON.parse(localStorage.getItem(K)!)
```

- 仅"绕过编译期检查",**运行时仍会真实报错**
- 用于"代码逻辑上一定有值,但 TS 推不出"的场景;能避免就避免

### 4. localStorage 取值的 TS 化范式

```ts
type Token = { token: string; refresh_token: string }

export const getToken = (): Token =>
  JSON.parse(localStorage.getItem(TOKEN_KEY) || '{}')   // 兜底字符串 → JSON.parse 不会炸

export const setToken = (t: Token): void =>
  localStorage.setItem(TOKEN_KEY, JSON.stringify(t))
```

### 5. React 函数组件 + Props 类型

```tsx
type IconProps = {
  type: string
  className?: string
  onClick?: () => void
}
const Icon: React.FC<IconProps> = ({ type, className, ...rest }) => (
  <svg {...rest}><use xlinkHref={`#${type}`}/></svg>
)
// 或者直接函数式签名 (推荐,React 18+ 不再推荐 React.FC)
function Icon({ type }: IconProps) { return <svg>...</svg> }
```

### 6. ref 在 TS 中的类型

```ts
const inputRef = useRef<HTMLInputElement>(null)
const imgRef   = useRef<HTMLImageElement>(null)

// 使用时类型已 narrow,可选链或非空断言
inputRef.current?.focus()
imgRef.current!.src = data.src
```

### 7. 扩展原生 HTML 属性 + 自定义 props

需求:组件透传所有 HTML `<input>` 属性,同时加自己的属性。

```ts
// 方式 1:&(交叉类型)
type Props = {
  extra?: string
  onExtraClick?: () => void
} & InputHTMLAttributes<HTMLInputElement>

// 方式 2:interface extends
interface Props extends InputHTMLAttributes<HTMLInputElement> {
  extra?: string
  onExtraClick?: () => void
}

// 方式 3:用 Omit 避免与原生属性冲突
type Props = Omit<InputHTMLAttributes<HTMLInputElement>, 'type' | 'autoFocus'> & {
  type?: 'text' | 'password'
  autoFocus?: boolean
}
```

### 8. 交叉类型 `&` vs 联合类型 `|`

| 操作符 | 含义 | 直觉 |
| --- | --- | --- |
| `A \| B` | 联合:**A 或 B** | 满足其一即可 |
| `A & B` | 交叉:**A 和 B 同时具备** | 必须同时满足 |

注意:`&` 在同名属性冲突时会取交集类型(可能成 `never`)。

### 9. Redux + TS 的核心套路

#### 9.1 reducer:用判别联合(Discriminated Union)定义 Action

```ts
type ProfileAction =
  | { type: 'profile/user'; payload: User }
  | { type: 'profile/profile'; payload: Profile }

export default function reducer(state = init, action: ProfileAction) {
  switch (action.type) {
    case 'profile/user':    return { ...state, user: action.payload }   // 类型自动 narrow 到 User
    case 'profile/profile': return { ...state, profile: action.payload }// 类型自动 narrow 到 Profile
    default: return state
  }
}
```

要点:**switch 上的 `action.type` 是字面量类型**,TS 会按 case 自动收窄 `action.payload` 的类型。

#### 9.2 actionCreator 与 `as const`

```ts
// 默认推断:type 会被推成 string,无法匹配联合
export const saveToken = (p: Token) => ({ type: 'login/token', payload: p })

// 修正方案 1:as const 锁字面量
export const saveToken = (p: Token) => ({ type: 'login/token' as const, payload: p })

// 修正方案 2:显式标返回类型
export const saveToken = (p: Token): LoginAction => ({ type: 'login/token', payload: p })
```

#### 9.3 从 store 自动派生 RootState 类型

```ts
const store = createStore(reducer, ...)
export type RootState = ReturnType<typeof store.getState>
```

- `typeof store.getState` → 函数类型 `() => StateShape`
- `ReturnType<...>` → 取返回值 → 即整个状态形状

#### 9.4 useSelector 类型

```ts
// 写法 A:泛型形式
const user = useSelector<RootState, User>(s => s.profile.user)

// 写法 B(更优):在回调参数上标注
const user = useSelector((s: RootState) => s.profile.user)
```

#### 9.5 redux-thunk 的 ThunkAction

```ts
import { ThunkAction, AnyAction } from 'redux-thunk'
export type AppThunk = ThunkAction<Promise<void>, RootState, unknown, AnyAction>

// 业务 thunk 只需标返回 AppThunk
export const fetchUser = (): AppThunk => async dispatch => {
  const res = await api.get('/user')
  dispatch(saveUser(res.data))
}
```

`ThunkAction<R, S, E, A>` 四个泛型:返回类型、State、Extra arg、Action 类型。

### 10. AuthRoute(路由守卫)在 TS 下的写法

```tsx
interface PrivateRouteProps extends RouteProps {
  component: React.ComponentType<any>
}
export default function AuthRoute({ component: C, ...rest }: PrivateRouteProps) {
  const loc = useLocation()
  return (
    <Route {...rest} render={() =>
      hasToken()
        ? <C/>
        : <Redirect to={{ pathname: '/login', state: { from: loc.pathname } }}/>
    }/>
  )
}
```

注意:`useLocation<LocationState>()` 可以给 `location.state` 标类型。

---

## 二、多文件调用流程图

### 2.1 渐进式迁移:JS 和 TS 共存

```
src/
 ├─ index.tsx              (改后)
 ├─ App.tsx                (改后)
 ├─ pages/
 │   ├─ Login.js           (待迁移)
 │   └─ Home.tsx           (改后)
 ├─ store/
 │   ├─ index.ts           ── createStore + 派生 RootState
 │   ├─ actions/
 │   │   ├─ login.ts       ── 用 RootThunkAction 标注
 │   │   └─ profile.ts
 │   └─ reducers/
 │       ├─ login.ts       ── 判别联合 ActionType
 │       └─ profile.ts
 ├─ types/
 │   └─ index.d.ts         ── 公共业务类型
 └─ utils/
     ├─ request.ts
     └─ storage.ts

   开 allowJs:true 让 .js 与 .ts 互相 import
   tsc 对 .js 不强求,但 .ts 互相 import 时享受全链路检查
```

### 2.2 Action → Reducer 的类型链路

```
actions/login.ts
  ┌────────────────────────────────────┐
  │ export const saveToken = (p:Token)=> ({
  │   type: 'login/token' as const,    │ ← 字面量类型 'login/token'
  │   payload: p                       │
  │ })                                  │
  └───────────────┬────────────────────┘
                  │ dispatch 时被识别为 LoginAction
                  ▼
reducers/login.ts
  type LoginAction =
    | { type:'login/token';  payload:Token }
    | { type:'login/logout'; payload:Token }

  function reducer(s, a: LoginAction) {
    switch (a.type) {
      case 'login/token':   // 此处 a.payload 自动 narrow 到 Token
      case 'login/logout':  // 此处 a.payload 自动 narrow 到 Token
    }
  }
                  │
                  ▼
store/index.ts
  combineReducers({ login: loginReducer, ... })
                  │
                  ▼
store/index.ts
  export type RootState = ReturnType<typeof store.getState>
```

### 2.3 RootState / useSelector / dispatch 类型流

```
                store/index.ts
                  ┌───────────────────────────┐
                  │ const store = createStore │
                  │ export type RootState =   │
                  │   ReturnType<typeof       │
                  │     store.getState>       │
                  └─────────────┬─────────────┘
                                │ import RootState
       ┌────────────────────────┴───────────────────────┐
       ▼                                                ▼
组件 Home.tsx                                  组件 Profile.tsx
  const list = useSelector(                     const user = useSelector(
    (s: RootState) => s.home.list)               (s: RootState) => s.profile.user)
              ▲                                              ▲
              │ TS 完整推断到具体字段类型,改字段名时编辑器立即报错 │
              └─────────────────────────────────────────────┘
```

### 2.4 ThunkAction 流水线

```
组件 dispatch(getUser())
         │
         ▼
actions/profile.ts:
   getUser = (): AppThunk => async dispatch => {
     const res = await http.get('/user')
     dispatch(saveUser(res.data))   ← 同步 action,触发 reducer
   }
         │ thunk 中间件识别"action 是 function" → 调用它
         ▼
   被调用时注入 dispatch / getState
         │
         ▼
   API 返回 → dispatch(saveUser(...))
         │
         ▼
reducer 处理 → state 更新 → useSelector 触发 re-render
```

---

## 三、勘误与修正

| # | 原文 | 问题 | 正确表述 |
| --- | --- | --- | --- |
| 1 | "非空断言 仅仅是让 TS 中的类型检查不再校验 null 或 undefined" 但例子中 `JSON.parse(localStorage.getItem(TOKEN_KEY)!)` 紧跟着 `|| {}` | 写法矛盾且有 bug | 如果用了 `!` 表示"一定不为 null",那 `|| {}` 永远走不到;反之若可能为 null,应该是 `JSON.parse(...) ?? {}` 或 `localStorage.getItem(K) ?? '{}'`(后者更稳)。**推荐 `JSON.parse(localStorage.getItem(K) ?? '{}')`,不使用非空断言** |
| 2 | "讲 App.js 改成 app.tsx" / "讲 js 改成 ts" 多处 | 错别字 | 应为 **"将"** |
| 3 | "TS 中的 `&`(交叉类型),作用:**取多个类型的并集**" | 概念颠倒 | `&` 在 **类型层面是交集**(必须同时满足);在 **属性维度上是并集**(同时拥有所有属性)。原文用"并集"描述容易混淆。准确表述见上文表格 |
| 4 | "推荐使用 type 替代 interface" 出现在 Input 章节 | 又一次武断推荐 | 见 12 文档勘误 #1。在描述"扩展 HTML 元素全部属性"时,两种写法等价;社区也无定论 |
| 5 | `import { TextareaHTMLAttributes } from 'hoist-non-react-statics/node_modules/@types/react'` | 错误导入路径 | 应改为 **`import type { TextareaHTMLAttributes } from 'react'`**。从 node_modules 子路径导入会随依赖版本变化而失效 |
| 6 | "useSelector 第一个泛型是整个 Redux 状态、第二个是当前要获取的状态" | 没说官方推荐 | 官方现已推荐使用 **预先封装好的 hooks**:`export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector`,这样每次调用不必再标 `RootState`。Redux Toolkit 文档明确推这一做法 |
| 7 | "Dispatch 类型的分析" 一节 | 解释不够 | `Dispatch` 是 redux 提供的泛型类型 `Dispatch<A extends Action = AnyAction>`,泛型表示能 dispatch 的 action 类型。配合判别联合可以做到 "只能 dispatch 已声明的 action" |
| 8 | "useHistory" 与 react-router 版本 | 已过时 | `useHistory` 仅在 **react-router v5**;v6 已改为 `useNavigate`。整篇的 `Route component / Redirect` 都属 v5 写法,迁移到 v6 需要大改 |
| 9 | reducer 函数 `function reducer(state = initValue, action)` 中 `action` 未标类型 | 类型不安全 | 必须显式标 `action: ProfileAction`(判别联合),否则 switch 内 `action.payload` 会是 any |
| 10 | "as InitType" 把空对象强转 | 风险 | `{} as InitType` 绕过初始化校验,使用时若访问字段会拿到 undefined。更稳的做法:**给类型加 `?` 把字段标可选**,或拆为 `Partial<InitType>` |
| 11 | "解决方案:`type: 'login/token' as const`" | 表述对,但缺关键讲解 | `as const` 的作用是把字面量 **锁** 为字面量类型而非 string。这样这条 action 才能匹配判别联合中的 `type: 'login/token'` 那一支。**没有 as const,联合判别就完全失效** |
| 12 | logout 强行加 payload | 设计不合理 | logout 不需要 payload,正确做法是把 `LoginAction` 联合写成 `\| { type: 'login/logout' }`(没有 payload 字段),让 logout action 不传 payload 也能通过类型检查 |
| 13 | RootThunkAction 泛型解释顺序 | 描述细节有偏差 | `ThunkAction<R, S, E, A>` 正确含义:**R = 返回值;S = State;E = Extra(传给 thunk 的第三参);A = Action**。原文虽列了字母,但说明的措辞顺序略有混乱 |

