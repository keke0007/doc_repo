# 知识梳理：Ajax、Promise、Axios、Fetch 与 RESTful API

> 文档生成日期：2026-05-25

---

## 目录
1. [Ajax 原理](#一ajax-原理)
2. [Promise 规范](#二promise-规范)
3. [手写 Promise](#三手写-promise)
4. [Fetch API](#四fetch-api)
5. [Axios 用法](#五axios-用法)
6. [Axios 源码核心](#六axios-源码核心)
7. [RESTful API 设计](#七restful-api-设计原则)
8. [网易云音乐 API 案例](#八网易云音乐-api-案例)
9. [速查表](#速查表)

---

## 一、Ajax 原理

### 1.1 XMLHttpRequest（XHR）概述

**XHR** 是浏览器提供的用于与服务器通信的对象，它提供了对 HTTP 协议的完全访问权限。

#### 创建 XHR 对象

```javascript
const xhr = new XMLHttpRequest();
```

### 1.2 XHR 的五种状态

XHR 对象通过 `readyState` 属性表示请求的当前状态：

```
readyState 值    常量名              含义
0              UNSENT            XHR对象已创建或被abort()重置
1              OPENED            open()已调用
2              HEADERS_RECEIVED  send()已调用，响应头和状态已可获得
3              LOADING           响应体下载中，responseText包含部分数据
4              DONE              响应完成，所有数据接收完毕
```

#### ASCII 流程图：XHR 状态变迁

```
创建XHR          调用open()      调用send()     服务器响应      响应完成
   |               |               |              |              |
   |--readyState:0-|--readyState:1--|--readyState:2--readyState:3--readyState:4|
   |    UNSENT     |    OPENED      |   HEADERS    |   LOADING   |    DONE    |
   |              |               |               |             |            |
   +-----------loadstart-------+                              |            |
                                                        progress(多次)    |
                                                                    load/error
                                                                        |
                                                                    loadend
```

### 1.3 基本使用流程

```javascript
// 第一步：创建 XMLHttpRequest 对象
const xhr = new XMLHttpRequest();

// 第二步：监听响应成功事件
xhr.onload = () => {
    console.log(xhr.responseText);
};

// 第三步：请求初始化
xhr.open('GET', '/getData', true);  // 方法、URL、是否异步（默认true）

// 第四步：发送请求
xhr.send();  // GET请求无请求体，可不传参
```

### 1.4 发送请求数据的三种方式

#### 方式一：URL 查询字符串（所有请求方法都可用）

```javascript
const qs = `username=${nameInp.value}&userpwd=${pwdInp.value}`;
xhr.open('GET', '/addData?' + qs);
xhr.send();
```

#### 方式二：请求体 - 字符串形式

```javascript
const qs = `username=john&userpwd=123`;
xhr.open('POST', '/addData');
xhr.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');
xhr.send(qs);
```

> 注意：`setRequestHeader()` 必须在 `open()` 之后、`send()` 之前调用

#### 方式三：请求体 - JSON 形式

```javascript
const data = { username: 'john', userpwd: '123' };
xhr.open('POST', '/addData');
xhr.setRequestHeader('Content-type', 'application/json');
xhr.send(JSON.stringify(data));
```

### 1.5 处理响应

#### 获取响应数据

```javascript
// 响应行
xhr.status;                      // 响应状态码 (如 200, 404)
xhr.statusText;                  // 响应状态文本 (如 "OK")

// 响应头
xhr.getResponseHeader('key');    // 获取指定响应头
xhr.getAllResponseHeaders();     // 获取所有响应头

// 响应体
xhr.responseText;                // 响应体字符串
xhr.response;                    // 根据responseType处理后的响应
xhr.responseType = 'json';       // 设置自动JSON解析
```

### 1.6 FormData 上传文件

```javascript
// 创建FormData对象
const fd = new FormData();
fd.append('file', fileInput.files[0]);
fd.append('username', 'john');

// 浏览器自动设置Content-Type为multipart/form-data
xhr.open('POST', '/upload');
xhr.send(fd);
```

### 1.7 超时控制

```javascript
xhr.timeout = 5000;  // 超时时间（毫秒）

xhr.ontimeout = () => {
    console.log('请求超时');
};
```

### 1.8 跨域问题：CORS 与 JSONP

#### CORS（跨域资源共享）- 官方方案

```javascript
// 前端无需额外处理，由后端设置响应头：
// Access-Control-Allow-Origin: http://localhost:8080
// 或
// Access-Control-Allow-Origin: *
```

#### JSONP - 古老方案（只支持GET）

```javascript
// 动态创建script标签，利用script天生跨域能力
const script = document.createElement('script');
script.src = 'http://example.com/data?callback=handleData';
document.body.appendChild(script);

// 定义回调函数
function handleData(data) {
    console.log(data);
}

// 服务端响应：handleData({name: "John"})
```

> **原文有误纠正①**：JSONP 不是真正的 Ajax，它绕过了 XMLHttpRequest，而是利用 script 标签的跨域能力执行服务器返回的 JavaScript 代码。

---

## 二、Promise 规范

### 2.1 Promise 三种状态

```javascript
// 初始状态
pending      // 进行中

// 终止状态（不可逆）
fulfilled    // 已成功，可通过 resolve() 改变
rejected     // 已失败，可通过 reject() 改变
```

一旦状态改变，就永远无法再改变！

### 2.2 基本语法

```javascript
// 创建Promise
const p = new Promise((resolve, reject) => {
    // resolve(value)  - 改为fulfilled状态
    // reject(reason)  - 改为rejected状态
    // throw error     - 改为rejected状态
});

// 处理结果
p.then(
    value => { /* fulfilled时执行 */ },
    reason => { /* rejected时执行 */ }
);

p.catch(reason => { /* rejected时执行 */ });

p.finally(() => { /* 总是执行 */ });
```

### 2.3 then() 方法返回值规则

then() 返回一个**新的Promise对象**，其状态由回调函数的返回值决定：

```
情况1：回调函数无返回值或返回undefined
  => 返回的Promise状态为fulfilled，PromiseResult为undefined

情况2：回调函数返回非Promise的值
  => 返回的Promise状态为fulfilled，PromiseResult为该返回值

情况3：回调函数返回Promise对象
  => 返回的Promise状态和结果与该Promise保持一致

情况4：回调函数抛出异常
  => 返回的Promise状态为rejected，PromiseResult为异常对象
```

#### ASCII 流程图：Promise 状态机

```
┌─────────────────────────────────────┐
│         new Promise()               │
│       执行器函数立即执行             │
└────────────┬────────────────────────┘
             │
        ┌────┴─────┐
        │           │
        ▼           ▼
    resolve()   reject()
        │           │
        │           │
        ▼           ▼
   pending -----> fulfilled      pending -----> rejected
                   ▲                                 ▲
                   │                                 │
              (状态不可逆)                      (状态不可逆)
                   │                                 │
               PromiseResult                    PromiseResult
                   成功值                          失败原因
```

### 2.4 then() 链式调用与微任务

**重要**：then() 的两个回调都是**异步执行**，加入**微任务队列**。

#### ASCII 流程图：then 链的微任务调度

```
调用 p.then(onResolved, onRejected)
        │
        ├─> 检查Promise状态
        │
        ├─ fulfilled ──┐
        │              ├─> 将 onResolved 加入微任务队列
        │
        ├─ rejected ───┤
        │              ├─> 将 onRejected 加入微任务队列
        │
        └─ pending ────┤
                       ├─> 将callbacks保存，待状态改变后入队
                       │
                       ▼
            返回新的Promise对象（状态待定）
                       │
                  主线程同步代码执行完
                       │
                ▼─────────────────────▼
            执行微任务队列
            (Promise回调、MutationObserver)
                       │
            ┌──────────┴──────────┐
            │                     │
          微任务1              微任务2
          执行回调            执行回调
            │                     │
            └─────────┬───────────┘
                      │
                 开始下一个宏任务
```

> **原文有误纠正②**：Promise 的 then/catch 回调是**微任务**，不是宏任务！微任务的执行优先级高于宏任务（定时器、Ajax、DOM事件）。

### 2.5 Promise 静态方法

#### Promise.resolve(value)

```javascript
// 情况1：无参数 => fulfilled，PromiseResult为undefined
Promise.resolve()

// 情况2：非Promise值 => fulfilled，PromiseResult为该值
Promise.resolve(42)              // fulfilled, 42
Promise.resolve({name: 'John'})  // fulfilled, {name: 'John'}

// 情况3：Promise对象 => 直接返回该Promise
const p = Promise.resolve(somePromise)  // p === somePromise

// 情况4：thenable对象 => 调用其then方法
const obj = {
    then(resolve, reject) {
        resolve('success');
    }
};
Promise.resolve(obj)  // fulfilled, 'success'
```

#### Promise.reject(reason)

```javascript
// 总是返回rejected状态的Promise
Promise.reject('error')  // rejected, 'error'
```

#### Promise.all(iterable)

```javascript
// 都成功才成功，一个失败全盘皆输
const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);
const p3 = Promise.resolve(3);

Promise.all([p1, p2, p3])
    .then(results => console.log(results))  // [1, 2, 3]
    .catch(err => console.log('失败:', err));
```

> **原文有误纠正③**：Promise.all() 是"一败全败"的逻辑 - 一旦某个Promise rejected，整个all()立即rejected，不等待其他Promise完成。

#### Promise.race(iterable)

```javascript
// 谁先完成就是谁（不管成功还是失败）
const p1 = new Promise(r => setTimeout(() => r('1'), 100));
const p2 = new Promise(r => setTimeout(() => r('2'), 50));

Promise.race([p1, p2])
    .then(result => console.log(result));  // '2'（50ms更快）
```

#### Promise.allSettled(iterable)

```javascript
// 所有Promise都完成（成功或失败都算），返回结果数组
const p1 = Promise.resolve(1);
const p2 = Promise.reject('error');

Promise.allSettled([p1, p2])
    .then(results => console.log(results));
    // [
    //   { status: 'fulfilled', value: 1 },
    //   { status: 'rejected', reason: 'error' }
    // ]
```

> **原文有误纠正④**：Promise.all() 和 Promise.allSettled() 的区别：
> - Promise.all()：一个失败整体失败
> - Promise.allSettled()：无论成功失败都完等待，返回所有结果

#### Promise.any(iterable)

```javascript
// 至少一个成功就成功，全部失败才失败
const p1 = Promise.reject('error1');
const p2 = Promise.resolve('success');

Promise.any([p1, p2])
    .then(result => console.log(result));  // 'success'
```

---

## 三、手写 Promise

### 3.1 Promise 基础实现

```javascript
class MyPromise {
    // 私有属性
    #state = 'pending';        // pending | fulfilled | rejected
    #result;                   // 成功值或失败原因
    #callbackList = [];        // 待执行的回调列表

    constructor(executor) {
        // 定义改变状态的函数
        const resolve = (value) => {
            if (this.#state !== 'pending') return;
            this.#state = 'fulfilled';
            this.#result = value;
            // 执行所有待执行的回调
            this.#callbackList.forEach(cb => cb.onResolved(value));
        };

        const reject = (reason) => {
            if (this.#state !== 'pending') return;
            this.#state = 'rejected';
            this.#result = reason;
            this.#callbackList.forEach(cb => cb.onRejected(reason));
        };

        // 执行器函数中可能抛异常
        try {
            executor(resolve, reject);
        } catch (error) {
            reject(error);
        }
    }

    then(onResolved, onRejected) {
        // 参数默认值处理（值穿透）
        if (typeof onResolved !== 'function') {
            onResolved = value => value;  // 值穿透
        }
        if (typeof onRejected !== 'function') {
            onRejected = reason => { throw reason; };  // 异常穿透
        }

        // 返回新Promise（链式调用）
        return new MyPromise((resolve, reject) => {
            // 封装处理函数
            const handler = (cb) => {
                try {
                    const res = cb(this.#result);
                    if (res instanceof MyPromise) {
                        res.then(resolve, reject);
                    } else {
                        resolve(res);
                    }
                } catch (error) {
                    reject(error);
                }
            };

            if (this.#state === 'fulfilled') {
                // 状态已改变，异步执行回调
                setTimeout(() => handler(onResolved));
            } else if (this.#state === 'rejected') {
                setTimeout(() => handler(onRejected));
            } else {
                // 状态未改变，保存回调
                this.#callbackList.push({
                    onResolved: () => handler(onResolved),
                    onRejected: () => handler(onRejected)
                });
            }
        });
    }

    catch(onRejected) {
        return this.then(undefined, onRejected);
    }

    finally(onFinally) {
        return this.then(
            value => MyPromise.resolve(onFinally()).then(() => value),
            reason => MyPromise.resolve(onFinally()).then(() => { throw reason; })
        );
    }

    static resolve(value) {
        if (value instanceof MyPromise) return value;
        if (typeof value?.then === 'function') {
            return new MyPromise((resolve, reject) => value.then(resolve, reject));
        }
        return new MyPromise(resolve => resolve(value));
    }

    static reject(reason) {
        return new MyPromise((_, reject) => reject(reason));
    }

    static all(promises) {
        return new MyPromise((resolve, reject) => {
            const results = [];
            let count = 0;
            for (let i = 0; i < promises.length; i++) {
                MyPromise.resolve(promises[i]).then(
                    value => {
                        results[i] = value;
                        if (++count === promises.length) resolve(results);
                    },
                    reject
                );
            }
        });
    }

    static race(promises) {
        return new MyPromise((resolve, reject) => {
            for (const promise of promises) {
                MyPromise.resolve(promise).then(resolve, reject);
            }
        });
    }
}
```

### 3.2 关键实现点

#### then 的微任务调度

```javascript
// 使用 setTimeout 模拟微任务（实际应用中可用 queueMicrotask）
setTimeout(() => handler(onResolved));  // 放入宏任务队列

// 更准确的方式：
queueMicrotask(() => handler(onResolved));  // 放入微任务队列
```

#### 链式 resolve 处理

当 then 的回调返回 Promise 时，新 Promise 的状态与返回的 Promise 保持一致：

```javascript
const p = new MyPromise(resolve => resolve(1))
    .then(val => {
        return new MyPromise(resolve => resolve(val + 1));
    })
    .then(val => console.log(val));  // 2
```

#### 值穿透

```javascript
Promise.resolve(1)
    .then()                        // 无参数
    .then()                        // 无参数
    .then(val => console.log(val)); // 1（值穿透到最后）
```

---

## 四、Fetch API

### 4.1 基本语法

```javascript
fetch(url, options)
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error(error));
```

### 4.2 请求配置项

```javascript
fetch('https://api.example.com/data', {
    method: 'POST',                    // GET, POST, PUT, DELETE
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({            // POST/PUT/PATCH需要
        name: 'John'
    }),
    mode: 'cors',                      // cors, no-cors, same-origin
    credentials: 'include',            // include, omit, same-origin
    cache: 'no-cache'                  // no-cache, reload, force-cache
});
```

### 4.3 Response 对象

```javascript
fetch(url)
    .then(response => {
        console.log(response.status);        // HTTP状态码
        console.log(response.statusText);    // HTTP状态文本
        console.log(response.headers);       // Headers对象
        console.log(response.ok);            // status 200-299 ? true : false
        
        // 解析响应体（不同格式）
        return response.json();              // Promise<Object>
        // return response.text();            // Promise<String>
        // return response.blob();            // Promise<Blob>
        // return response.arrayBuffer();     // Promise<ArrayBuffer>
    });
```

### 4.4 Fetch 特殊注意

> **原文有误纠正⑤**：
> 1. **Fetch 不会因为 HTTP 错误状态码（如 404、500）reject**，只有网络故障才会 reject
> 2. **Fetch 默认不发送 Cookie**，需要设置 `credentials: 'include'` 才能跨域携带 Cookie

```javascript
// 404 不会触发 catch！
fetch('https://api.example.com/notfound')
    .then(response => {
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
        }
        return response.json();
    })
    .catch(error => console.error(error));

// 跨域时携带 Cookie
fetch(url, {
    credentials: 'include',  // 关键
    mode: 'cors'
});
```

---

## 五、Axios 用法

### 5.1 基本使用

```javascript
// 方式1：通用方式
axios({
    method: 'GET',
    url: 'https://api.example.com/users'
})

// 方式2：URL + 配置
axios('https://api.example.com/users', { method: 'GET' })

// 方式3：方法别名
axios.get('https://api.example.com/users')
axios.post('https://api.example.com/users', { name: 'John' })
axios.put('https://api.example.com/users/1', { name: 'Jane' })
axios.delete('https://api.example.com/users/1')
axios.patch('https://api.example.com/users/1', { name: 'Jack' })
```

### 5.2 请求配置项

```javascript
axios({
    url: '/user',
    method: 'get',
    
    // baseURL 拼接规则：非绝对URL时才拼接
    baseURL: 'https://api.example.com/',
    
    // URL参数（GET）
    params: { id: 1 },
    
    // 请求头
    headers: {
        'Authorization': 'Bearer token'
    },
    
    // 请求体（POST/PUT/PATCH）
    data: { name: 'John' },
    
    // 响应类型
    responseType: 'json',  // 'json', 'text', 'blob', 'arraybuffer'
    
    // 超时设置
    timeout: 5000,
    
    // 跨域是否带Cookie
    withCredentials: false
});
```

> **原文有误纠正⑤**：baseURL 拼接规则 - 当 url 是绝对URL（以http://或https://开头）时，baseURL 被忽略；否则拼接。

### 5.3 创建实例

```javascript
// 不同接口可能需要不同配置
const instance = axios.create({
    baseURL: 'https://api.example.com/',
    timeout: 3000,
    headers: {
        'Authorization': 'Bearer myToken'
    }
});

instance.get('/users')
    .then(response => console.log(response.data));
```

### 5.4 拦截器

#### ASCII 流程图：拦截器洋葱模型

```
                 请求拦截器
                     │
          ┌──────────┴──────────┐
          │  拦截器2（后注册）   │
          │  拦截器1（先注册）   │  <- 后进先出
          └──────────┬──────────┘
                     │
                  (config)
                     │
                     ▼
              发送 Ajax 请求
                     │
                     ▼ (response)
          ┌──────────┬──────────┐
          │  拦截器1（先注册）   │
          │  拦截器2（后注册）   │  <- 先进先出
          └──────────┴──────────┘
                     │
                响应拦截器
                     │
                 回调函数
```

#### 添加拦截器

```javascript
// 请求拦截器
axios.interceptors.request.use(
    config => {
        // config 是本次请求的配置对象
        config.headers.Authorization = 'Bearer token';
        return config;
    },
    error => {
        return Promise.reject(error);
    }
);

// 响应拦截器
axios.interceptors.response.use(
    response => {
        // 只返回数据部分
        return response.data;
    },
    error => {
        // 统一错误处理
        console.error(error.response?.status);
        return Promise.reject(error);
    }
);
```

#### 移除拦截器

```javascript
const reqId = axios.interceptors.request.use(...);
const resId = axios.interceptors.response.use(...);

// 移除
axios.interceptors.request.eject(reqId);
axios.interceptors.response.eject(resId);
```

### 5.5 取消请求

```javascript
// 创建取消令牌
const controller = new AbortController();

axios.get('/users', {
    signal: controller.signal
});

// 在需要时取消
controller.abort();

// 或者使用旧方式（CancelToken）
let cancel = null;
axios.get('/users', {
    cancelToken: new axios.CancelToken(c => {
        cancel = c;
    })
})
.catch(err => {
    if (axios.isCancel(err)) {
        console.log('请求已取消');
    }
});

cancel();  // 取消请求
```

### 5.6 并发请求

```javascript
const r1 = axios.get('/users/1');
const r2 = axios.get('/users/2');
const r3 = axios.get('/users/3');

// 方式1：Promise.all
Promise.all([r1, r2, r3])
    .then(([res1, res2, res3]) => {
        console.log(res1.data, res2.data, res3.data);
    });

// 方式2：axios.all + axios.spread
axios.all([r1, r2, r3])
    .then(axios.spread((res1, res2, res3) => {
        console.log(res1.data, res2.data, res3.data);
    }));
```

### 5.7 响应结构

```javascript
{
    data: {},              // 响应体数据
    status: 200,           // HTTP状态码
    statusText: 'OK',      // HTTP状态文本
    headers: {},           // 响应头
    config: {},            // 请求配置
    request: XMLHttpRequest // XHR对象
}
```

---

## 六、Axios 源码核心

### 6.1 源码目录结构

```
axios/
├── /adapters/
│   ├── xhr.js              # 浏览器适配器（基于XMLHttpRequest）
│   └── http.js             # Node.js适配器
├── /core/
│   ├── Axios.js            # 核心类
│   ├── InterceptorManager.js # 拦截器管理
│   ├── dispatchRequest.js   # 请求分发
│   └── settle.js            # 响应处理
├── /cancel/
│   └── CancelToken.js       # 取消令牌
└── defaults.js              # 默认配置
```

### 6.2 InterceptorManager 链式调用原理

#### ASCII 流程图：Axios 请求拦截器→适配器→响应拦截器洋葱模型

```
axios({...})
    │
    ▼ 1. 创建Promise链
    │
    ├─ 添加请求拦截器→适配器→响应拦截器
    │
    Request.use(...)   Request.use(...)
         │                   │
      (后注册)             (先注册)
         │                   │
    ┌────▼──────────────────▼───┐
    │   Promise 链式调用        │
    │   (后进先出 LIFO)          │
    └────┬──────────────────┬───┘
         │                  │
    拦截器2.then(...)  拦截器1.then(...)
         │                  │
    ┌────▼──────────────────▼───┐
    │   发送请求                │
    │   dispatchRequest(config) │
    └────┬──────────────────────┘
         │
    ┌────▼──────────────────┬───┐
    │   处理响应             │    │
    └────┬──────────────────▼───┘
         │
    拦截器1.then(...)  拦截器2.then(...)
         │                  │
    ┌────▼──────────────────▼───┐
    │   Promise 链式调用        │
    │   (先进先出 FIFO)          │
    └────┬──────────────────────┘
         │
       finally()
         │
    用户回调函数
```

### 6.3 InterceptorManager 实现

```javascript
class InterceptorManager {
    constructor() {
        this.handlers = [];
    }

    use(resolved, rejected) {
        this.handlers.push({
            resolved,
            rejected
        });
        // 返回ID，用于移除拦截器
        return this.handlers.length - 1;
    }

    eject(id) {
        if (this.handlers[id]) {
            this.handlers[id] = null;
        }
    }
}

// 使用示例
class Axios {
    constructor(config) {
        this.defaults = config;
        this.interceptors = {
            request: new InterceptorManager(),
            response: new InterceptorManager()
        };
    }

    request(config) {
        // 创建Promise链
        let promise = Promise.resolve(config);

        // 添加请求拦截器（后进先出）
        this.interceptors.request.handlers.forEach(handler => {
            if (handler) {
                promise = promise
                    .then(handler.resolved, handler.rejected);
            }
        });

        // 发送请求
        promise = promise.then(config => {
            return dispatchRequest(config);
        });

        // 添加响应拦截器（先进先出）
        this.interceptors.response.handlers.forEach(handler => {
            if (handler) {
                promise = promise
                    .then(handler.resolved, handler.rejected);
            }
        });

        return promise;
    }
}
```

### 6.4 CancelToken 取消传播原理

#### ASCII 流程图：CancelToken 取消传播机制

```
创建CancelToken
    │
    ├─ new Promise((resolve, reject) => {...})
    │
    └─ 将resolve函数保存到cancel属性
         │
         ▼
    调用 cancel() 函数
         │
         ├─ 改变cancelToken的Promise状态
         │
         └─ 触发then中的回调
              │
              ├─ 执行 xhr.abort()
              │
              └─ 中止请求
```

```javascript
class CancelToken {
    constructor(executor) {
        let resolveCancel;
        
        this.promise = new Promise(resolve => {
            resolveCancel = resolve;
        });

        // 调用executor，传入cancel函数
        executor((message) => {
            resolveCancel({
                message
            });
        });
    }

    static source() {
        let cancel;
        const token = new CancelToken(c => {
            cancel = c;
        });
        return {
            token,
            cancel
        };
    }
}

// 在dispatchRequest中
config.cancelToken.promise.then(() => {
    xhr.abort();
});
```

### 6.5 Defaults 合并策略

```javascript
// 全局config > 实例config > 默认config
const config = Object.assign(
    {},
    this.defaults,    // 实例的默认配置
    userConfig        // 用户传入的配置
);

// baseURL 拼接（非绝对URL才拼接）
if (config.baseURL && !config.url.startsWith('http')) {
    config.url = config.baseURL + config.url;
}
```

### 6.6 Adapter 模式

```javascript
// 核心思想：适配不同运行环境
function getAdapter(config) {
    if (typeof XMLHttpRequest !== 'undefined') {
        return xhrAdapter;  // 浏览器环境
    } else if (typeof http !== 'undefined') {
        return httpAdapter; // Node.js环境
    }
}

// 执行请求
function dispatchRequest(config) {
    const adapter = getAdapter(config);
    return adapter(config);  // 返回Promise
}
```

---

## 七、RESTful API 设计原则

### 7.1 REST 概念

**REST** (Representational State Transfer) 是一种网络应用架构风格，强调：
- 使用 HTTP 方法表示操作
- 使用 URL 表示资源
- 无状态通信

### 7.2 RESTful vs 非RESTful

#### 非 RESTful API（URL 表示操作）

```
增加用户：POST http://api.example.com/user/create
删除用户：GET  http://api.example.com/user/delete?id=1
修改用户：POST http://api.example.com/user/modify
查询用户：GET  http://api.example.com/user/list
```

#### RESTful API（HTTP 方法表示操作，URL 表示资源）

```
增加用户：POST   http://api.example.com/users
删除用户：DELETE http://api.example.com/users/1
修改用户：PUT    http://api.example.com/users/1
查询用户：GET    http://api.example.com/users
查询用户1：GET   http://api.example.com/users/1
```

### 7.3 HTTP 方法语义

| 方法 | CRUD | 幂等性 | 安全性 | 说明 |
|------|------|--------|--------|------|
| GET | Read | 是 | 是 | 获取资源，无副作用 |
| POST | Create | 否 | 否 | 创建资源，每次调用创建新资源 |
| PUT | Update | 是 | 否 | 替换资源，幂等操作 |
| PATCH | Update | 否 | 否 | 部分更新资源 |
| DELETE | Delete | 是 | 否 | 删除资源，幂等操作 |

- **幂等性**：多次调用结果相同（GET/PUT/DELETE是幂等的）
- **安全性**：不修改服务器数据（GET是安全的）

### 7.4 状态码规范

```
2xx 成功
  200 OK
  201 Created
  204 No Content

3xx 重定向
  301 Moved Permanently
  302 Found
  304 Not Modified

4xx 客户端错误
  400 Bad Request
  401 Unauthorized
  403 Forbidden
  404 Not Found

5xx 服务器错误
  500 Internal Server Error
  502 Bad Gateway
  503 Service Unavailable
```

---

## 八、网易云音乐 API 案例

### 8.1 API 架构

网易云音乐 API 是一个完整的 RESTful 接口系统，提供：

- **用户系统**：登录、注册、账户管理
- **音乐资源**：歌曲、歌单、专辑、电台
- **交互功能**：评论、点赞、收藏
- **搜索推荐**：搜索、排行榜、推荐

### 8.2 登录认证机制

#### 登录方式

```javascript
// 方式1：手机号登录
GET /login/cellphone?phone=13xxx&password=xxx

// 方式2：邮箱登录
GET /login?email=xxx@163.com&password=xxx

// 方式3：二维码登录（流程）
// 1. GET /login/qr/key            -> 获取key
// 2. GET /login/qr/create?key=xxx -> 生成二维码
// 3. GET /login/qr/check?key=xxx  -> 轮询检测状态
//    801: 等待扫码  802: 待确认  803: 登录成功
```

#### ASCII 流程图：网易云加密流程

```
用户输入
    │
    ├─ 手机号/邮箱
    ├─ 密码
    │
    ▼
参数加密处理
    │
    ├─ MD5 加密密码（可选）
    │
    ├─ base64编码
    │
    ├─ AES/RSA 加密敏感参数
    │
    ▼
发送登录请求
    │
    ├─ POST /login
    │
    ▼
服务器验证
    │
    ├─ 校验用户身份
    ├─ 生成 Session/JWT
    ├─ 下发 Cookie
    │
    ▼
浏览器保存 Cookie
    │
    ├─ 后续请求自动携带
    ├─ 需要登录的接口才能调用
    │
    ▼
获取用户数据
```

### 8.3 签名与反爬虫

#### 签名参数生成

网易云音乐 API 通过以下方式防止爬虫：

```javascript
// 1. 时间戳参数
?timestamp=${Date.now()}  // 每次请求不同，防缓存

// 2. 加密签名
// - 部分接口使用encSecKey加密敏感数据
// - 需要计算params的加密值

// 3. 实现示例（伪代码）
function createSign(data) {
    // 1. 将参数转为JSON字符串
    const params = JSON.stringify(data);
    
    // 2. 使用AES加密
    const encParams = encryptAES(params, secretKey);
    
    // 3. 使用RSA加密对称密钥
    const encSecKey = encryptRSA(secretKey, publicKey);
    
    return {
        params: encParams,
        encSecKey: encSecKey
    };
}
```

### 8.4 典型调用模式

#### 获取歌曲详情

```javascript
// 1. 获取榜单
axios.get('/toplist')
    .then(res => res.data.list[0].id)  // 榜单ID

// 2. 获取榜单歌曲
.then(playlistId => 
    axios.get(`/playlist/detail?id=${playlistId}`)
)
.then(res => res.data.playlist.trackIds)  // 歌曲ID列表

// 3. 获取歌曲详情
.then(trackIds => 
    axios.get(`/song/detail?ids=${trackIds.slice(0, 10).join(',')}`)
)
.then(res => console.log(res.data.songs))

// async/await 方式更清晰
async function getMusicDetails() {
    try {
        // 获取榜单
        const toplistRes = await axios.get('/toplist');
        const playlistId = toplistRes.data.list[0].id;
        
        // 获取歌曲列表
        const playlistRes = await axios.get(`/playlist/detail?id=${playlistId}`);
        const trackIds = playlistRes.data.playlist.trackIds;
        
        // 获取歌曲详情
        const songRes = await axios.get(`/song/detail?ids=${trackIds.join(',')}`);
        return songRes.data.songs;
    } catch (error) {
        console.error('获取数据失败:', error);
    }
}
```

#### 登录流程

```javascript
// 使用学生提供的测试API
async function login(phone, password) {
    try {
        const response = await axios.get(
            `http://api.fuming.site:54255/login/cellphone?phone=${phone}&password=${password}`
        );
        
        // 保存cookie用于后续请求
        const cookie = response.data.cookie;
        
        // 后续请求带上cookie
        return axios.get('/user/detail', {
            params: { uid: response.data.profile.userId },
            headers: { Cookie: cookie }
        });
    } catch (error) {
        console.error('登录失败:', error);
    }
}

// 跨域请求需要withCredentials
axios.get(url, {
    withCredentials: true,  // 关键！
    headers: {
        'Access-Control-Allow-Credentials': 'true'
    }
});
```

### 8.5 防爬虫规避策略

```javascript
// 1. 添加时间戳防缓存
const url = `/user/detail?uid=123&timestamp=${Date.now()}`;

// 2. 添加User-Agent
axios.interceptors.request.use(config => {
    config.headers['User-Agent'] = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)...';
    return config;
});

// 3. 添加Referer
config.headers['Referer'] = 'https://music.163.com/';

// 4. 合理控制请求频率（限流）
const delay = (ms) => new Promise(r => setTimeout(r, ms));
for (let i = 0; i < ids.length; i++) {
    await axios.get(`/song/detail?id=${ids[i]}`);
    await delay(1000);  // 每次请求间隔1秒
}

// 5. 部分接口不需要缓存处理
// - 登录接口：不要频繁调用（可能被风控）
// - 搜索接口：添加时间戳参数避免缓存
// - 下载接口：需要关注是否有频率限制
```

---

## 速查表

### Ajax 常用方法

| 方法/属性 | 说明 |
|----------|------|
| `xhr.open(method, url, async)` | 初始化请求 |
| `xhr.send(body)` | 发送请求 |
| `xhr.setRequestHeader(key, value)` | 设置请求头 |
| `xhr.onload` | 响应成功事件 |
| `xhr.onerror` | 请求失败事件 |
| `xhr.ontimeout` | 超时事件 |
| `xhr.responseText` | 响应体字符串 |
| `xhr.response` | 根据responseType的响应 |
| `xhr.abort()` | 中止请求 |
| `xhr.timeout` | 超时时间（毫秒） |

### Promise 常用方法

| 方法 | 说明 |
|------|------|
| `Promise.resolve(value)` | 返回fulfilled的Promise |
| `Promise.reject(reason)` | 返回rejected的Promise |
| `Promise.all(iterable)` | 全部成功才成功 |
| `Promise.race(iterable)` | 谁先完成算谁 |
| `Promise.allSettled(iterable)` | 等待全部完成 |
| `Promise.any(iterable)` | 至少一个成功就成功 |
| `promise.then(onFulfilled, onRejected)` | 处理结果 |
| `promise.catch(onRejected)` | 处理异常 |
| `promise.finally(onFinally)` | 总是执行 |

### Axios 常用方法

| 方法 | 说明 |
|------|------|
| `axios.get(url, config)` | 发送GET请求 |
| `axios.post(url, data, config)` | 发送POST请求 |
| `axios.put(url, data, config)` | 发送PUT请求 |
| `axios.delete(url, config)` | 发送DELETE请求 |
| `axios.all(promises)` | 并发请求 |
| `axios.create(config)` | 创建实例 |
| `axios.interceptors.request.use()` | 请求拦截器 |
| `axios.interceptors.response.use()` | 响应拦截器 |
| `axios.CancelToken` | 取消令牌 |
| `axios.isCancel(error)` | 判断是否取消 |

### Fetch 常用方法

| 方法/属性 | 说明 |
|----------|------|
| `fetch(url, options)` | 发送请求 |
| `response.ok` | status 200-299 ? true : false |
| `response.status` | HTTP状态码 |
| `response.json()` | 解析为JSON |
| `response.text()` | 解析为文本 |
| `response.blob()` | 解析为Blob |
| `credentials: 'include'` | 跨域携带Cookie |

---

## 纠错总结

| 序号 | 常见误解 | 正确理解 |
|------|---------|---------|
| ① | JSONP是真正的Ajax | JSONP利用script标签，不是XMLHttpRequest |
| ② | Promise回调是宏任务 | **Promise回调是微任务**，优先级高于宏任务 |
| ③ | Promise.all()等待全部 | Promise.all()是"一败全败"，一个失败立即失败 |
| ④ | .all()和.allSettled()一样 | allSettled等待全部完成无论成功失败 |
| ⑤ | Fetch 404会reject | **Fetch只在网络故障时reject**，HTTP错误不reject |
| ⑥ | Fetch默认发送Cookie | **Fetch默认不发送Cookie**，需credentials配置 |
| ⑦ | baseURL总是拼接 | **baseURL仅对非绝对URL拼接** |

---

文档生成日期：2026-05-25
