# Node.js 服务端完整知识体系

> **文档生成日期：2026-05-25**
>
> 综合 Day06-Day13 课堂笔记，涵盖 Node.js、Express、MongoDB、会话控制全面体系

---

## 目录
1. [一、Node.js 基础](#一nodejs-基础)
2. [二、核心模块](#二核心模块)
3. [三、npm 与包管理](#三npm-与包管理)
4. [四、Express 框架](#四express-框架)
5. [五、MongoDB 数据库](#五mongodb-数据库)
6. [六、会话控制](#六会话控制)
7. [速查表](#速查表)

---

## 一、Node.js 基础

### 1.1 Node.js 概述

**定义**：基于 Chrome V8 引擎的 JavaScript 运行环境（宿主），与浏览器等价。

**为什么学 Node.js**：
- 后端开发，前端工程师变全栈
- 前端工程化开发（webpack、gulp、脚手架）
- 开发工具脚本、爬虫程序
- 跨平台桌面应用（Electron、React Native）

**Node.js 特点**：
- 单线程
- 非阻塞 I/O (non-blocking I/O)
- 事件驱动 (event-driven)

### 1.2 运行机制

#### Node.js 事件循环 (Event Loop) - 6 个阶段

```
┌─────────────────────────────────┐
│        Event Loop 完整流程        │
└─────────────────────────────────┘
                │
        ┌───────▼────────┐
        │  timers 阶段    │ setTimeout/setInterval 回调
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ pending callbacks│ 系统操作的回调
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │  idle, prepare  │ 内部准备
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │   poll 阶段    │ I/O 相关回调（fs、网络等）
        │  (可能阻塞)     │ 检查有无定时器就绪
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │   check 阶段   │ setImmediate 回调
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │  close callbacks│ socket.on('close') 等
        └───────┬────────┘
                │
         循环回到 timers
```

> **⚠️ 原文有误**：
> 
> 误：Node.js 是单线程、单进程单线程
> 
> 正：Node.js **主线程是单线程**，但底层 libuv 维护了一个**线程池**处理 I/O 操作。Node 本身是多进程架构（可用 cluster 模块创建多个进程），只不过开发时主要操作单线程事件循环。

---

### 1.3 全局对象与变量

#### 内置常量
```js
__dirname    // 脚本所在目录的绝对路径
__filename   // 脚本自身的绝对路径
```

#### 全局对象（global）
- 浏览器中是 `window`
- Node.js 中是 `global`

#### Process 对象
```js
process.argv          // 命令行参数数组
process.cwd()         // 当前工作目录
process.exit(code)    // 退出进程
process.pid           // 进程 ID
process.version       // Node 版本
process.platform      // 操作系统平台
```

---

### 1.4 模块化 - CommonJS 规范

#### 模块分类

| 分类 | 特点 | 示例 |
|------|------|------|
| 内置模块 | Node官方提供 | fs、path、http、events |
| 第三方模块 | 需要 npm 安装 | express、mongoose |
| 自定义模块 | 用户创建 | ./utils.js、../config/db.js |

#### 暴露数据

```js
// 方式1：module.exports 整体替换
module.exports = {
  say: function() {},
  eat: function() {}
};

// 方式2：module.exports 设置属性
module.exports.name = 'Tom';
module.exports.age = 18;

// 方式3：exports 设置属性（内部等同 module.exports）
exports.name = 'Tom';  // 正确
exports = {name:'Tom'} // 错误！脱钩
```

#### 引入模块

```js
// 自定义模块需要 ./ 或 ../
const utils = require('./utils');
const config = require('../config');

// 内置/第三方模块直接用名字
const fs = require('fs');
const express = require('express');
```

#### Module 查找算法

```
require('模块名') 查找流程
        │
        ├─ 是否 ./ 或 ../ 开头？
        │   ├─ 是 → 自定义模块查找
        │   │       └─ 查找路径 → .js → .json → 目录(package.json main) → index.js
        │   │
        │   └─ 否 → 内置或第三方模块
        │           ├─ 是否内置模块？(fs、path、http...)
        │           │   ├─ 是 → 加载内置模块
        │           │   └─ 否 → 第三方模块查找
        │           │
        │           └─ 第三方模块查找
        │               ├─ 当前目录 node_modules
        │               ├─ 上级目录 node_modules
        │               ├─ 再上级...
        │               └─ 一直查找到根目录
        │
        └─ 缓存命中？
            └─ 是 → 返回缓存
```

> **⚠️ 原文有误**：
> 
> 误：模块路径"参照命令行所在的目录"
> 
> 正：自定义模块的相对路径参照**所在文件的目录**，不是命令行目录。命令行的相对路径才参照当前工作目录。

---

### 1.5 Buffer 二进制数据

#### Buffer 特点
- 大小**固定**，创建时确定，无法调整
- 直接操作计算机内存，性能较好
- 每个元素 1 字节（8 bit）

#### 创建 Buffer

```js
Buffer.alloc(10);              // <Buffer 00 00 00 00...>
Buffer.alloc(2, 'a');          // <Buffer 61 61> (a 的 ASCII 是 0x61)
Buffer.from('hello');          // <Buffer 68 65 6c 6c 6f>
Buffer.from([1, 2, 3]);        // <Buffer 01 02 03>

// alloc 安全，allocUnsafe 快但可能包含旧数据
Buffer.allocUnsafe(10);
```

#### Buffer 读写

```js
const b = Buffer.from('hello');
b[0];                    // 104 (h 的 ASCII)
b.toString();            // 'hello'
b.toString('utf8', 1, 3) // 'el'

b.forEach((item, idx) => {
  console.log(item, idx);
});
```

#### 中文编码
一个 UTF-8 中文字符占 **3 个字节**（大多数情况）

---

## 二、核心模块

### 2.1 fs 模块 - 文件系统

#### 文件读取

```js
const fs = require('fs');
const path = require('path');

// 异步读取（推荐）
fs.readFile('./data.txt', 'utf-8', (err, data) => {
  if (err) {
    console.error('读取失败:', err.code);
    return;
  }
  console.log(data);
});

// 同步读取（阻塞）
try {
  const data = fs.readFileSync('./data.txt', 'utf-8');
  console.log(data);
} catch (err) {
  console.error('读取失败:', err.code);
}
```

#### 文件写入

```js
// 异步写入（覆盖）
fs.writeFile('./output.txt', '内容', (err) => {
  if (err) throw err;
  console.log('写入成功');
});

// 追加写入
fs.appendFile('./output.txt', '追加', (err) => {
  if (err) throw err;
  console.log('追加成功');
});

// 同步写入
fs.writeFileSync('./output.txt', '内容');
```

#### 文件重命名/移动

```js
fs.rename('./old.txt', './new.txt', (err) => {
  if (err) throw err;
  console.log('重命名成功');
});

// 也可用于移动文件
fs.rename('./a.txt', '../b.txt', (err) => {
  if (err) throw err;
  console.log('移动成功');
});
```

#### 文件删除

```js
fs.unlink('./file.txt', (err) => {
  if (err) throw err;
  console.log('删除成功');
});

fs.unlinkSync('./file.txt');
```

#### 目录操作

```js
// 创建目录（可递归）
fs.mkdir('./dir/sub', {recursive: true}, (err) => {
  if (err) throw err;
  console.log('创建成功');
});

// 删除目录（空目录或递归）
fs.rmdir('./dir', {recursive: true}, (err) => {
  if (err) throw err;
  console.log('删除成功');
});

// 读取目录
fs.readdir('./dir', (err, files) => {
  if (err) throw err;
  console.log(files); // ['file1.txt', 'file2.js', ...]
});
```

#### 文件/目录判断

```js
// 判断是否存在
fs.access('./file.txt', (err) => {
  if (err) {
    console.log('不存在');
  } else {
    console.log('存在');
  }
});

// 判断是文件还是目录
fs.stat('./target', (err, stats) => {
  if (err) throw err;
  console.log('是目录:', stats.isDirectory());
  console.log('是文件:', stats.isFile());
});
```

#### 流式读写

```js
// 大文件使用流，省内存
const rs = fs.createReadStream('./big-file.txt');
rs.on('data', (chunk) => {
  console.log('读取块:', chunk.length, '字节');
});
rs.on('end', () => {
  console.log('读取完毕');
});
rs.on('error', (err) => {
  console.error('读取出错:', err);
});

// 流式写入
const ws = fs.createWriteStream('./output.txt');
for (let i = 0; i < 100000; i++) {
  ws.write(`行 ${i}\n`);
}
ws.end();
ws.on('close', () => {
  console.log('写入完毕');
});

// 流式复制
const rs = fs.createReadStream('./src.txt');
const ws = fs.createWriteStream('./dst.txt');
rs.pipe(ws);  // 管道，自动处理缓冲
```

---

### 2.2 path 模块 - 路径处理

```js
const path = require('path');

path.join(__dirname, './data', 'a.txt')
// → C:\project\data\a.txt (Windows)
// → /home/project/data/a.txt (Linux)

path.resolve('./file.txt')
// 返回绝对路径，相对于当前工作目录

path.isAbsolute('/home/user')        // true
path.isAbsolute('./relative')        // false

path.dirname('/home/user/file.txt')  // /home/user
path.basename('/home/user/file.txt') // file.txt
path.basename('/home/user/file.txt', '.txt') // file
path.extname('/home/user/file.txt')  // .txt

path.parse('/home/user/file.txt')
// {
//   root: '/',
//   dir: '/home/user',
//   base: 'file.txt',
//   name: 'file',
//   ext: '.txt'
// }
```

---

### 2.3 url 模块 - URL 解析

```js
const url = require('url');

const urlStr = 'http://www.example.com:8080/path?id=1&name=tom#anchor';

// 方式1：url.parse() (已废弃，但仍可用)
const parsed = url.parse(urlStr, true);
console.log(parsed.query);  // {id: '1', name: 'tom'}

// 方式2：new url.URL() (推荐)
const urlObj = new url.URL(urlStr);
console.log(urlObj.searchParams.get('id'));    // '1'
console.log(urlObj.searchParams.get('name'));  // 'tom'
console.log(urlObj.pathname);  // /path
console.log(urlObj.protocol);  // http:
```

---

### 2.4 querystring 模块 - 查询字符串

```js
const qs = require('querystring');

// 解析查询字符串
const str = 'name=tom&age=18&city=beijing';
const obj = qs.parse(str);
// {name: 'tom', age: '18', city: 'beijing'}

// 对象转查询字符串
const queryStr = qs.stringify({name: 'tom', age: 18});
// 'name=tom&age=18'
```

---

### 2.5 events 模块 - 事件系统

```js
const EventEmitter = require('events');

const emitter = new EventEmitter();

// 监听事件
emitter.on('say', (name) => {
  console.log(`Hello ${name}`);
});

// 触发事件
emitter.emit('say', 'Tom');  // Hello Tom

// 一次性监听
emitter.once('hello', () => {
  console.log('仅执行一次');
});

// 移除监听
emitter.off('say', callback);
emitter.removeAllListeners('say');
```

---

### 2.6 HTTP 模块 - 服务器

#### HTTP 请求生命周期流程

```
┌──────────────────────────────────────────┐
│         HTTP 请求完整生命周期              │
└──────────────────────────────────────────┘
              │
    ┌─────────▼────────────┐
    │ 1. 客户端发送请求     │
    │ (浏览器输入 URL)      │
    └─────────┬────────────┘
              │
    ┌─────────▼────────────┐
    │ 2. TCP 三次握手      │
    │ 建立连接             │
    └─────────┬────────────┘
              │
    ┌─────────▼────────────┐
    │ 3. 发送请求报文      │
    │ 请求行/请求头/请求体  │
    └─────────┬────────────┘
              │
    ┌─────────▼────────────┐
    │ 4. 服务器接收处理    │
    │ Node 路由分发        │
    │ 业务逻辑处理         │
    └─────────┬────────────┘
              │
    ┌─────────▼────────────┐
    │ 5. 发送响应报文      │
    │ 响应行/响应头/响应体  │
    └─────────┬────────────┘
              │
    ┌─────────▼────────────┐
    │ 6. 客户端接收处理    │
    │ 浏览器渲染           │
    └─────────┬────────────┘
              │
    ┌─────────▼────────────┐
    │ 7. TCP 四次挥手      │
    │ 连接关闭             │
    └──────────────────────┘
```

#### 创建 HTTP 服务

```js
const http = require('http');

const server = http.createServer((req, res) => {
  // req: 请求对象 (IncomingMessage)
  // res: 响应对象 (ServerResponse)
  
  // 获取请求信息
  console.log(req.method);         // GET、POST 等
  console.log(req.url);            // /path?query=1
  console.log(req.headers);        // 请求头对象
  console.log(req.socket.remoteAddress);  // 客户端 IP
  
  // 获取 URL 查询字符串
  const url = require('url');
  const query = url.parse(req.url, true).query;
  
  // 获取请求体（POST 数据）
  let body = '';
  req.on('data', chunk => {
    body += chunk;
  });
  req.on('end', () => {
    const qs = require('querystring');
    const data = qs.parse(body);
    // 处理 data
  });
  
  // 设置响应
  res.statusCode = 200;
  res.statusMessage = 'OK';
  res.setHeader('Content-Type', 'text/html; charset=utf-8');
  
  // 或使用 writeHead 一次性设置
  res.writeHead(200, 'OK', {
    'Content-Type': 'text/html; charset=utf-8'
  });
  
  // 写入响应体
  res.write('<h1>Hello</h1>');
  res.write('<p>World</p>');
  
  // 结束响应
  res.end();  // 或 res.end('内容')
});

server.listen(8080, '127.0.0.1', () => {
  console.log('服务运行在 http://127.0.0.1:8080');
});

// 获取 GET 和 POST 有区别
// GET：通过 URL 查询字符串传递数据
// POST：通过请求体传递数据，有请求体
```

---

## 三、npm 与包管理

### 3.1 npm 基本命令

```bash
# 查看版本
npm -v
npm --version

# 初始化项目
npm init           # 交互式
npm init -y        # 快速初始化

# 安装依赖
npm install package-name      # 产品依赖
npm install package-name -D   # 开发依赖
npm install package-name -g   # 全局安装
npm install package-name@1.2.3 # 指定版本

# 删除依赖
npm remove package-name
npm uninstall package-name

# 更新依赖
npm update package-name
npm outdated      # 查看可更新的包

# 清除缓存
npm cache clean --force

# 安装所有依赖（从 package.json）
npm install
npm i
```

> **⚠️ 原文有误**：
>
> 误：npm install 遵循"参照命令行目录"
>
> 正：npm install 等价于 require，会自动向上级查找 node_modules，不仅基于当前目录

---

### 3.2 package.json 详解

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "项目描述",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "server": "node server.js",
    "test": "jest"
  },
  "author": "Your Name",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "^6.0.0"
  },
  "devDependencies": {
    "webpack": "^5.0.0",
    "babel": "^7.0.0"
  }
}
```

#### 版本号规则

| 符号 | 含义 | 示例 |
|------|------|------|
| `^` | 锁定大版本 | ^3.0.0 → 3.x.x（最新） |
| `~` | 锁定小版本 | ~3.1.x → 3.1.x（最新） |
| 无 | 锁定完整版本 | 3.1.1 → 必须 3.1.1 |

#### 配置脚本别名

```json
{
  "scripts": {
    "start": "node index.js",
    "dev": "node app.js",
    "server": "node server.js"
  }
}
```

执行：
```bash
npm run start   # 运行 node index.js
npm start       # start 特殊，可省略 run
npm run dev
npm run server
```

---

### 3.3 使用镜像加速

```bash
# 方式1：配置 cnpm 命令
npm install -g cnpm --registry=https://registry.npm.taobao.org

# 方式2：直接修改 npm 源
npm config set registry https://registry.npm.taobao.org

# 恢复官方源
npm config set registry https://registry.npmjs.org/
```

---

### 3.4 npx 工具

npx 用于直接执行 npm 包中的命令，无需全局安装。

```bash
# 无需全局安装 create-react-app，直接运行
npx create-react-app my-app

# 运行本地 node_modules 中的命令
npx webpack  # 相当于 ./node_modules/.bin/webpack
```

---

## 四、Express 框架

### 4.1 Express 基础

Express 是基于 Node.js 的极简、灵活的 Web 框架，由**路由**和**中间件**构成。

#### 安装与基本使用

```bash
npm install express
```

```js
const express = require('express');
const path = require('path');

const app = express();

// 静态文件托管
app.use(express.static(path.join(__dirname, 'public')));

// GET 路由
app.get('/index', (req, res) => {
  res.send('<h1>首页</h1>');
});

// POST 路由
app.post('/login', (req, res) => {
  res.send('登录成功');
});

// 启动服务
app.listen(8080, () => {
  console.log('Express 服务运行在 http://localhost:8080');
});
```

---

### 4.2 路由详解

#### 路由方法

```js
app.get(path, callback)      // GET 请求
app.post(path, callback)     // POST 请求
app.put(path, callback)      // PUT 请求
app.delete(path, callback)   // DELETE 请求
app.all(path, callback)      // 所有方法
app.route(path)              // 链式定义
```

#### 路由匹配

```js
// 1. 精确匹配
app.get('/home', (req, res) => {
  res.send('home');
});

// 2. 字符串模糊匹配（? 表示可选）
app.get('/admin/?a?', (req, res) => {
  // 匹配 /admin, /admina, /adminab
});

// 3. 通配符匹配（* 表示任意）
app.get('/admin/*', (req, res) => {
  // 匹配 /admin/a, /admin/user/list 等
});

// 4. 正则匹配
app.get(/\.html$/, (req, res) => {
  // 匹配所有 .html 结尾的路由
});

// 5. 路径参数
app.get('/news/:id', (req, res) => {
  console.log(req.params.id);  // 获取参数值
  res.send(`新闻 ID: ${req.params.id}`);
});

app.get('/user/:id/post/:postId', (req, res) => {
  console.log(req.params.id);      // 用户 ID
  console.log(req.params.postId);  // 文章 ID
});

// 6. 404 处理（放在最后）
app.all('*', (req, res) => {
  res.status(404).send('404 页面不存在');
});
```

#### 链式路由

```js
app.route('/login')
  .get((req, res) => {
    res.send('登录页面');
  })
  .post((req, res) => {
    res.send('处理登录');
  })
  .put((req, res) => {
    res.send('更新登录信息');
  });
```

---

### 4.3 请求与响应对象

#### 请求对象 (Request)

```js
app.get('/user', (req, res) => {
  // 基本信息
  req.method           // 请求方法 (GET、POST...)
  req.url              // 请求 URL (/user?id=1)
  req.path             // 请求路径 (/user)
  req.ip               // 客户端 IP
  req.hostname         // 主机名
  
  // 查询字符串参数
  req.query            // {id: '1', name: 'tom'}
  
  // 路径参数
  app.get('/news/:id', (req, res) => {
    req.params.id      // 路径中的参数值
  });
  
  // 请求头
  req.headers          // 所有请求头对象
  req.get('content-type')  // 获取特定请求头
  
  // 请求体（需要中间件处理）
  req.body             // {username: 'tom', password: '123'}
});
```

#### 响应对象 (Response)

```js
app.get('/demo', (req, res) => {
  // 设置状态码
  res.status(200);
  res.status(404);
  
  // 设置响应头
  res.set('Content-Type', 'text/html; charset=utf-8');
  res.setHeader('key', 'value');
  
  // 发送响应体
  res.send('<h1>Hello</h1>');          // 自动设置 Content-Type
  res.send({name: 'tom'});             // 自动 JSON.stringify
  res.end('text');                     // 原始响应
  
  // 发送文件
  res.sendFile(__dirname + '/index.html');
  res.download(__dirname + '/file.zip');
  
  // 发送 JSON
  res.json({name: 'tom', age: 18});
  
  // 渲染模板
  res.render('index', {title: 'Home'}); // 需配置模板引擎
  
  // 重定向
  res.redirect('/home');
  res.redirect(301, '/home');
  
  // 链式调用
  res.status(200).set({'X-Custom': 'value'}).send('OK');
});
```

---

### 4.4 中间件 (Middleware)

中间件本质是**函数**，形式：`(req, res, next) => {}`

#### 中间件执行顺序 - 洋葱模型

```
     ┌────────────────────────┐
     │   客户端发送请求        │
     └────────────┬───────────┘
                  │
        ┌─────────▼──────────┐
        │  中间件1 前置逻辑   │ (日志记录)
        ├─────────┬──────────┤
        │  中间件2 前置逻辑   │ (身份验证)
        ├─────────┬──────────┤
        │  路由处理器         │ (主业务逻辑)
        ├─────────┬──────────┤
        │  中间件2 后置逻辑   │ (返回响应)
        ├─────────┬──────────┤
        │  中间件1 后置逻辑   │ (日志记录)
        └────────┬───────────┘
                 │
     ┌───────────▼──────────┐
     │   发送响应给客户端   │
     └──────────────────────┘
```

#### 应用级中间件

```js
const express = require('express');
const app = express();

// 全局中间件（所有路由都会执行）
app.use((req, res, next) => {
  console.log(`[${new Date().toLocaleTimeString()}] ${req.method} ${req.url}`);
  // 必须调用 next() 传递控制权
  next();
});

// 路由前的中间件（特定路由）
app.use('/admin', (req, res, next) => {
  // /admin 开头的所有路由都会执行
  if (req.session && req.session.user) {
    next();
  } else {
    res.redirect('/login');
  }
});

// 路由
app.get('/admin/list', (req, res) => {
  res.send('管理列表');
});

// 路由后的中间件
app.use((req, res) => {
  res.status(404).send('404 Not Found');
});
```

#### 错误处理中间件

> **⚠️ 原文有误**：
>
> 误：错误中间件可以接收 3 个或 4 个参数
>
> 正：错误处理中间件**必须**定义 4 个参数 `(err, req, res, next)`，少一个都不会被识别为错误中间件

```js
// 必须定义 4 个参数
app.use((err, req, res, next) => {
  console.error('错误:', err.stack);
  res.status(500).send('服务器错误');
});
```

#### 自定义中间件示例

```js
// middleware/accessLog.js - 访问日志
const moment = require('moment');
const fs = require('fs');
const path = require('path');

module.exports = (req, res, next) => {
  const ip = req.ip.slice(7);  // 去掉 ::ffff: 前缀
  const method = req.method;
  const url = req.url;
  const dt = moment().format('YYYY-MM-DD HH:mm:ss');
  
  const logMsg = `${ip} ${dt} ${method} ${url}\n`;
  console.log(logMsg);
  
  fs.appendFile(path.resolve(__dirname, '../logs/access.log'), logMsg, err => {
    if (err) throw err;
    next();  // 继续执行下一个中间件或路由
  });
};

// 在应用中使用
const accessLog = require('./middleware/accessLog');
app.use(accessLog);
```

---

### 4.5 Body 解析中间件

#### 处理 POST 数据

> **⚠️ 原文有误**：
>
> 误：Express 4.x 需要额外的 body-parser 中间件
>
> 正：Express 4.16.0+ 内置 `express.json()` 和 `express.urlencoded()`，不需要装单独的 body-parser

```js
const express = require('express');
const app = express();

// 解析 application/json
app.use(express.json());

// 解析 application/x-www-form-urlencoded
app.use(express.urlencoded({extended: false}));

app.post('/login', (req, res) => {
  const {username, password} = req.body;
  res.json({username, password});
});
```

---

### 4.6 静态文件服务

```js
const express = require('express');
const path = require('path');
const app = express();

// 方式1：单个目录
app.use(express.static(path.join(__dirname, 'public')));

// 方式2：多个目录
app.use(express.static(path.join(__dirname, 'public')));
app.use(express.static(path.join(__dirname, 'images')));

// 方式3：指定虚拟路径前缀
app.use('/static', express.static(path.join(__dirname, 'public')));
// 访问 http://localhost:8080/static/css/style.css
```

---

### 4.7 路由模块化 - express.Router

```js
// routes/users.js
const express = require('express');
const router = express.Router();

// 路由中间件
router.use((req, res, next) => {
  console.log('User route middleware');
  next();
});

// 路由定义
router.get('/', (req, res) => {
  res.send('用户列表');
});

router.get('/:id', (req, res) => {
  res.send(`用户 ID: ${req.params.id}`);
});

router.post('/', (req, res) => {
  res.send('创建用户');
});

module.exports = router;

// app.js
const express = require('express');
const usersRouter = require('./routes/users');
const productsRouter = require('./routes/products');

const app = express();

// 挂载路由模块
app.use('/users', usersRouter);      // /users, /users/:id
app.use('/products', productsRouter); // /products, /products/:id

app.listen(8080);
```

---

### 4.8 模板引擎 - EJS

#### 配置模板引擎

```js
const express = require('express');
const ejs = require('ejs');
const path = require('path');

const app = express();

// 方式1：使用 EJS（默认扩展名 .ejs）
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// 方式2：使用 HTML 扩展名
app.engine('html', ejs.renderFile);
app.set('view engine', 'html');
app.set('views', path.join(__dirname, 'views'));

// 渲染模板
app.get('/', (req, res) => {
  res.render('index', {
    title: '首页',
    message: 'Hello World',
    items: ['apple', 'banana', 'cherry']
  });
});
```

#### EJS 模板语法

```ejs
<!-- views/index.ejs -->

<!-- 1. 执行代码（不输出） -->
<% if (user) { %>
  <h1>欢迎 <%= user.name %></h1>
<% } %>

<!-- 2. 输出转义（防止 XSS） -->
<p><%= message %></p>  <!-- Hello World -->

<!-- 3. 输出原始 HTML（不转义） -->
<div><%- htmlContent %></div>

<!-- 4. 循环 -->
<ul>
  <% items.forEach(item => { %>
    <li><%= item %></li>
  <% }); %>
</ul>

<!-- 5. 三元运算符 -->
<p><%= age >= 18 ? '成人' : '未成年' %></p>

<!-- 6. 双向分支 -->
<% if (score >= 90) { %>
  <span>优秀</span>
<% } else if (score >= 80) { %>
  <span>良好</span>
<% } else { %>
  <span>及格</span>
<% } %>
```

---

## 五、MongoDB 数据库

### 5.1 MongoDB 概述

MongoDB 是文档型 NoSQL 数据库，特点：
- 灵活的文档结构（无需预定义 Schema）
- 高效的读写性能
- 易于扩展和水平扩展
- 支持复杂查询

#### 三个核心概念

```
MongoDB 数据库结构
├── 数据库 (Database)
│   ├── 集合 (Collection) - 类似表
│   │   ├── 文档 (Document) - 类似行/记录
│   │   │   └── { _id, name, age, email, ... }
│   │   ├── 文档
│   │   │   └── { _id, name, age, email, ... }
│   │   └── ...
│   └── 集合
│       └── ...
└── 数据库
    └── ...
```

---

### 5.2 MongoDB 基本命令

```bash
# 启动 MongoDB 服务
mongod --dbpath D:\data\db --port 27017

# 连接 MongoDB
mongo

# 数据库操作
show dbs                    # 显示所有数据库
use myapp                   # 切换/创建数据库
db                          # 显示当前数据库
db.dropDatabase()           # 删除当前数据库

# 集合操作
db.createCollection('users')     # 创建集合
show collections                 # 显示所有集合
db.users.drop()                  # 删除集合
db.users.renameCollection('persons')  # 重命名集合

# 文档操作 (CRUD)
db.users.insert({name: 'Tom', age: 18})  # 插入
db.users.find()                    # 查询所有
db.users.find({name: 'Tom'})       # 条件查询
db.users.findOne({name: 'Tom'})    # 查询一个
db.users.update({name: 'Tom'}, {$set: {age: 19}})  # 更新
db.users.updateMany({age: {$gt: 18}}, {$set: {vip: true}})  # 批量更新
db.users.remove({name: 'Tom'})     # 删除
db.users.deleteMany({age: {$lt: 18}})  # 批量删除

# 条件操作符
$gt   # 大于
$lt   # 小于
$gte  # 大于等于
$lte  # 小于等于
$ne   # 不等于
$in   # 包含（满足其中任意一个）
$or   # 逻辑或
$and  # 逻辑与

# 排序、分页
db.users.find().sort({age: 1})         # 升序（1 升 -1 降）
db.users.find().skip(10).limit(10)     # 跳过 10 条，取 10 条

# 正则查询
db.users.find({name: /tom/})           # 包含 tom
db.users.find({name: /^tom/i})         # 以 tom 开头，不区分大小写
```

---

### 5.3 Mongoose ODM

Mongoose 是 Node.js 中操作 MongoDB 的对象数据模型库，在 MongoDB 原生驱动基础上进行了优化和增强。

#### 安装与连接

```bash
npm install mongoose
```

```js
const mongoose = require('mongoose');

// 连接 MongoDB
mongoose.connect('mongodb://127.0.0.1:27017/myapp');
// 或带身份验证
mongoose.connect('mongodb://user:password@localhost/myapp?authSource=admin');

// 监听连接成功
mongoose.connection.on('open', () => {
  console.log('数据库连接成功！');
  // 在这里进行数据库操作
});

// 监听连接失败
mongoose.connection.on('error', () => {
  console.log('数据库连接失败！');
});
```

---

### 5.4 Schema 与 Model

#### Mongoose Schema→Model→Document 关系流程

```
┌─────────────────────────────────────┐
│       Schema（文档结构）             │
│  定义字段、类型、验证规则、中间件    │
└────────────────┬────────────────────┘
                 │
                 │ mongoose.model()
                 ▼
┌─────────────────────────────────────┐
│       Model（文档模型）              │
│  映射到集合，提供 CRUD 方法          │
│  create、find、update、delete 等    │
└────────────────┬────────────────────┘
                 │
                 │ .create()、.find() 等
                 ▼
┌─────────────────────────────────────┐
│      Document（文档实例）            │
│  具体的数据对象，可直接操作           │
│  {_id, name, age, email, ...}      │
└─────────────────────────────────────┘
```

#### Schema 定义

```js
const mongoose = require('mongoose');

// 定义 Schema
const userSchema = new mongoose.Schema({
  name: String,                    // 字符串
  age: Number,                     // 数字
  active: Boolean,                 // 布尔值
  emails: [String],                // 字符串数组
  createdAt: Date,                 // 日期
  
  // 带选项
  username: {
    type: String,
    required: true,                // 必填
    unique: true,                  // 唯一
    minlength: 3,                  // 最小长度
    maxlength: 20                  // 最大长度
  },
  
  password: {
    type: String,
    required: true,
    minlength: 6
  },
  
  role: {
    type: String,
    enum: ['user', 'admin'],       // 枚举值
    default: 'user'                // 默认值
  },
  
  avatar: {
    type: String,
    default: '/images/default.jpg'
  }
});

// Schema 类型
/*
String          字符串
Number          数字
Boolean         布尔值
Array / []      数组
Date            日期
Buffer          Buffer
Mixed           任意类型 (mongoose.Schema.Types.Mixed)
ObjectId        对象 ID (mongoose.Schema.Types.ObjectId)
Decimal128      高精度数字
*/

module.exports = userSchema;
```

#### Model 创建

```js
const mongoose = require('mongoose');
const userSchema = require('./schema/user');

// 创建 Model（参数：集合名，Schema）
const User = mongoose.model('User', userSchema);
// 自动映射到 users 集合（Mongoose 会自动复数化）

module.exports = User;
```

---

### 5.5 CRUD 操作

#### 创建 (Create)

```js
const User = require('./models/User');

// 单条插入
User.create({
  username: 'tom',
  password: '123456',
  role: 'user'
}, (err, doc) => {
  if (err) {
    console.error('创建失败:', err);
    return;
  }
  console.log('创建成功:', doc);
});

// 批量插入
User.insertMany([
  {username: 'tom', password: '123456'},
  {username: 'jerry', password: '654321'},
  {username: 'spike', password: 'admin123'}
], (err, docs) => {
  if (err) throw err;
  console.log('批量插入成功:', docs);
});
```

#### 读取 (Read)

```js
// 查询一条（返回第一条匹配的）
User.findOne({username: 'tom'}, (err, doc) => {
  if (err) throw err;
  console.log(doc);
});

// 根据 ID 查询
User.findById('5dd662b5381fc316b44ce167', (err, doc) => {
  if (err) throw err;
  console.log(doc);
});

// 查询多条
User.find({}, (err, docs) => {
  if (err) throw err;
  console.log(docs);
});

// 条件查询
User.find({age: {$gte: 18}}, (err, docs) => {
  if (err) throw err;
  console.log(docs);
});

// 字段筛选（1 显示，0 隐藏）
User.find()
  .select({username: 1, email: 1, _id: 0})
  .exec((err, docs) => {
    console.log(docs);
  });

// 排序（1 升序，-1 降序）
User.find()
  .sort({age: -1})
  .exec((err, docs) => {
    console.log(docs);
  });

// 分页
User.find()
  .skip(20)        // 跳过 20 条
  .limit(10)       // 取 10 条
  .exec((err, docs) => {
    console.log(docs);
  });

// 链式调用
User.find({role: 'user'})
  .select({username: 1, email: 1})
  .sort({createdAt: -1})
  .limit(10)
  .exec((err, docs) => {
    if (err) throw err;
    console.log(docs);
  });
```

#### 更新 (Update)

```js
// 更新一条
User.updateOne(
  {username: 'tom'},
  {$set: {age: 20}},
  (err, result) => {
    if (err) throw err;
    console.log('更新成功:', result);
  }
);

// 批量更新
User.updateMany(
  {role: 'user'},
  {$set: {vip: false}},
  (err, result) => {
    if (err) throw err;
    console.log('批量更新成功:', result);
  }
);

// 替换整个文档
User.update(
  {username: 'tom'},
  {username: 'tom2', age: 25, role: 'admin'},
  (err, result) => {
    if (err) throw err;
    console.log(result);
  }
);
```

#### 删除 (Delete)

```js
// 删除一条
User.deleteOne({username: 'tom'}, (err, result) => {
  if (err) throw err;
  console.log('删除成功:', result);
});

// 根据 ID 删除
User.findByIdAndDelete('5dd662b5381fc316b44ce167', (err, doc) => {
  if (err) throw err;
  console.log('删除成功:', doc);
});

// 批量删除
User.deleteMany({role: 'user'}, (err, result) => {
  if (err) throw err;
  console.log('批量删除成功:', result);
});
```

---

### 5.6 高级查询

#### 运算符详解

```js
User.find({
  age: {$gt: 18}        // 大于
});

User.find({
  age: {$gte: 18}       // 大于等于
});

User.find({
  age: {$lt: 30}        // 小于
});

User.find({
  age: {$lte: 30}       // 小于等于
});

User.find({
  age: {$ne: 25}        // 不等于
});

User.find({
  age: {$in: [18, 20, 25]}  // 包含其中任意一个
});

User.find({
  $or: [
    {age: {$lt: 18}},
    {age: {$gt: 60}}
  ]
});

User.find({
  $and: [
    {age: {$gt: 18}},
    {age: {$lt: 30}}
  ]
});

// 正则查询
User.find({
  username: /tom/       // 包含 tom
});

User.find({
  username: /^admin/i   // 以 admin 开头，不区分大小写
});
```

#### 文档关联（关系）

MongoDB 支持两种关联方式：

**方式1：嵌入式（推荐用于一对一、一对少）**

```js
const articleSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: {
    name: String,
    email: String,
    avatar: String
  }
});
```

**方式2：引用式（推荐用于一对多、多对多）**

```js
const userSchema = new mongoose.Schema({
  username: String,
  email: String
});

const articleSchema = new mongoose.Schema({
  title: String,
  content: String,
  authorId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'    // 引用 User 集合
  }
});

// 查询时使用 populate 填充关联数据
Article.find()
  .populate('authorId')  // 自动填充作者信息
  .exec((err, articles) => {
    console.log(articles);
    // authorId 会被替换为完整的用户对象
  });
```

---

## 六、会话控制

### 6.1 会话控制概述

HTTP 是**无状态协议**，无法区分多个请求是否来自同一客户端。会话控制解决方案：
- **Cookie**：存储在客户端
- **Session**：存储在服务器端
- **Token**：无状态认证

---

### 6.2 Cookie

#### Cookie 原理

```
┌─────────────────────────────────────────┐
│      Cookie 完整流转过程                 │
└─────────────────────────────────────────┘
        │
┌───────▼───────────┐
│  1. 客户端请求    │ GET /login
│  无 Cookie        │
└───────┬───────────┘
        │
┌───────▼──────────────┐
│  2. 服务器响应      │
│  Set-Cookie 响应头  │
│  userId=abc123      │
└───────┬──────────────┘
        │
┌───────▼──────────────┐
│  3. 浏览器保存      │
│  在本地 Cookie 存储 │
└───────┬──────────────┘
        │
┌───────▼──────────────┐
│  4. 后续请求        │
│  Cookie 请求头      │
│  自动发送 Cookie    │
└───────┬──────────────┘
        │
┌───────▼──────────────┐
│  5. 服务器识别      │
│  通过 Cookie 识别   │
│  客户端身份         │
└──────────────────────┘
```

#### Cookie 使用

```bash
npm install cookie-parser
```

```js
const express = require('express');
const cookieParser = require('cookie-parser');

const app = express();

// 使用中间件
app.use(cookieParser());

// 设置 Cookie
app.get('/setCookie', (req, res) => {
  // 设置普通 Cookie（会话级，关闭浏览器即删除）
  res.cookie('userId', '12345');
  
  // 设置有过期时间的 Cookie（maxAge 单位毫秒）
  res.cookie('token', 'abc123', {
    maxAge: 1000 * 60 * 60 * 24,  // 24 小时
    httpOnly: true                 // 前端无法通过 JS 访问
  });
  
  res.send('Cookie 已设置');
});

// 读取 Cookie
app.get('/getCookie', (req, res) => {
  console.log(req.cookies);  // {userId: '12345', token: 'abc123'}
  console.log(req.cookies.userId);  // '12345'
  res.send('获取 Cookie 成功');
});

// 删除 Cookie
app.get('/clearCookie', (req, res) => {
  res.clearCookie('userId');
  res.clearCookie('token', {path: '/admin'});
  res.send('Cookie 已删除');
});
```

#### Cookie 缺点

1. 大小限制：数量不超过 50 个，单个不超过 4KB
2. 传输消耗：大 Cookie 会影响网络传输速度
3. 安全性低：存储在客户端，可被修改

---

### 6.3 Session

#### Session 原理

```
┌──────────────────────────────────────────┐
│       Session 完整工作流程                │
└──────────────────────────────────────────┘
        │
┌───────▼──────────────────┐
│ 1. 用户提交登录信息      │
│    POST /login           │
│    {username, password}  │
└───────┬──────────────────┘
        │
┌───────▼──────────────────┐
│ 2. 服务器验证          │
│    查询数据库            │
│    验证用户名密码        │
└───────┬──────────────────┘
        │
        ├─ 验证失败 → 响应 401
        │
┌───────▼──────────────────┐
│ 3. 创建 Session 对象    │
│    在服务器内存中        │
│    存储用户信息          │
│    生成唯一 sessionId   │
└───────┬──────────────────┘
        │
┌───────▼──────────────────┐
│ 4. 设置 Cookie          │
│    Set-Cookie           │
│    connect.sid=xxx      │
│    (不直接暴露用户信息)  │
└───────┬──────────────────┘
        │
┌───────▼──────────────────┐
│ 5. 浏览器保存 Cookie    │
│    connect.sid 值       │
└───────┬──────────────────┘
        │
┌───────▼──────────────────┐
│ 6. 后续请求             │
│    自动发送 Cookie      │
│    Cookie: connect.sid  │
└───────┬──────────────────┘
        │
┌───────▼──────────────────┐
│ 7. 服务器解析          │
│    通过 sessionId       │
│    查找对应 Session    │
│    验证用户身份         │
└───────┬──────────────────┘
        │
┌───────▼──────────────────┐
│ 8. 返回用户数据         │
│    req.session 可访问   │
│    用户信息             │
└──────────────────────────┘
```

#### Session 使用

```bash
npm install express-session
```

```js
const express = require('express');
const session = require('express-session');

const app = express();

// 配置 Session 中间件
app.use(session({
  name: 'sid',              // Cookie 名称，默认 connect.sid
  secret: 'mySecret',       // 参与加密的字符串（签名）
  saveUninitialized: false, // 不保存未初始化的 Session
  resave: false,            // 不强制保存未修改的 Session
  cookie: {
    httpOnly: true,         // 前端无法通过 JS 访问
    maxAge: 1000 * 60 * 30  // 30 分钟过期
  }
}));

// 设置 Session
app.get('/login', (req, res) => {
  // 验证用户信息...
  req.session.userId = 123;
  req.session.username = 'tom';
  req.session.role = 'user';
  res.send('登录成功');
});

// 读取 Session
app.get('/profile', (req, res) => {
  if (req.session.userId) {
    res.send(`欢迎 ${req.session.username}`);
  } else {
    res.redirect('/login');
  }
});

// 删除 Session
app.get('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) throw err;
    res.send('退出登录成功');
  });
});
```

> **⚠️ 原文有误**：
>
> 误：Session 默认存储在内存中，支持分布式/集群部署
>
> 正：express-session 默认确实存储在**内存**中，只能在**单机**使用。生产环境应配合 store（如 connect-mongo、redis）实现持久化存储，支持集群部署。

---

### 6.4 Cookie vs Session vs Token

| 对比项 | Cookie | Session | Token (JWT) |
|--------|--------|---------|------------|
| **存储位置** | 客户端浏览器 | 服务器端 | 客户端（token + payload） |
| **安全性** | 低（明文）| 高（服务器控制） | 中等（签名验证） |
| **网络流量** | 每次请求都发送 Cookie | 仅发送 ID，数据在服务器 | 发送完整 Token |
| **数据大小** | 单个 4KB，总数 50 个 | 理论无限 | 通常 1-2KB |
| **服务器存储** | 无 | 需要（占用服务器内存） | 无（无状态） |
| **跨域支持** | 不支持 | 不支持 | 支持（CORS） |
| **分布式** | 可以 | 需要 Session Store | 天生支持 |
| **撤销难度** | 难 | 易 | 难（不能撤销） |

---

### 6.5 Token & JWT

#### JWT (JSON Web Token) 三段式结构

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

│                                      │ │                                      │ │                                      │
└──────────────────────────────────────┘ └──────────────────────────────────────┘ └──────────────────────────────────────┘
         Header (头部)                           Payload (载荷)                        Signature (签名)
     Base64 编码                          Base64 编码                           HMACSHA256(header.payload, secret)

┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ {                │  │ {                │  │ HMACSHA256(      │
│ "alg": "HS256",  │  │ "sub": "1234...", │  │   base64(header) │
│ "typ": "JWT"     │  │ "name": "John",  │  │ + "." +          │
│ }                │  │ "iat": 1516...,  │  │   base64(payload)│
│                  │  │ "exp": 1516...   │  │   secret         │
│                  │  │ }                │  │ )                │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

#### JWT 校验流程

```
┌──────────────────────────────────┐
│   客户端发送 JWT Token           │
│   Authorization: Bearer xxx...   │
└────────────┬─────────────────────┘
             │
    ┌────────▼──────────┐
    │ 服务器接收 Token  │
    │ 解析三段          │
    └────────┬──────────┘
             │
    ┌────────▼────────────────┐
    │ 验证签名是否合法        │
    │ HMACSHA256(header.      │
    │ payload, secret) ?=     │
    │ signature               │
    └────────┬────────────────┘
             │
        ┌────┴────┐
        │          │
    ┌───▼──┐  ┌───▼──┐
    │ 无效  │  │ 有效  │
    │       │  │       │
    │ 401   │  │ 200   │
    │ 拒绝  │  │ 接受  │
    │       │  │ 继续  │
    └───────┘  └───────┘
```

#### JWT 使用示例

```bash
npm install jsonwebtoken
```

```js
const express = require('express');
const jwt = require('jsonwebtoken');

const app = express();
const secret = 'mySecretKey';

// 生成 Token
app.post('/login', (req, res) => {
  const user = {
    id: 1,
    username: 'tom',
    role: 'user'
  };
  
  // 创建 JWT
  const token = jwt.sign(user, secret, {
    expiresIn: '24h'  // 过期时间
  });
  
  res.json({
    token: token,
    message: '登录成功'
  });
});

// 验证 Token 中间件
function verifyToken(req, res, next) {
  const token = req.headers['authorization']?.slice(7);  // Bearer xxx
  
  if (!token) {
    return res.status(401).json({message: '无效 Token'});
  }
  
  jwt.verify(token, secret, (err, decoded) => {
    if (err) {
      return res.status(401).json({message: '无效或过期 Token'});
    }
    
    req.user = decoded;  // 保存用户信息
    next();
  });
}

// 使用验证中间件
app.get('/profile', verifyToken, (req, res) => {
  res.json({
    message: '获取成功',
    user: req.user
  });
});

app.listen(8080);
```

> **⚠️ 原文有误**：
>
> 误：JWT 可以撤销（通过 blacklist 等机制）
>
> 正：JWT 本质是**无状态**的，服务器无法主动撤销。只能通过 token blacklist（将过期 token 加入黑名单）、缩短过期时间等间接方案实现，但这违背了 JWT 无状态的初衷。

---

### 6.6 Cookie SameSite 属性

> **⚠️ 原文有误**：
>
> 误：Cookie SameSite 默认无限制
>
> 正：现代浏览器（Chrome 80+）**默认值为 SameSite=Lax**，限制跨站 Cookie 发送，防止 CSRF 攻击。需要跨站发送时显式设置 SameSite=None（需 Secure）。

```js
res.cookie('token', 'value', {
  sameSite: 'Strict',   // 完全禁止跨站发送
  // 或
  sameSite: 'Lax',      // 仅在安全跨站请求（GET）发送（默认）
  // 或
  sameSite: 'None',     // 允许跨站发送（需配合 Secure）
  secure: true          // 仅通过 HTTPS 发送
});
```

---

## 速查表

### Node.js 核心 API 速查

| 功能 | 代码片段 |
|------|--------|
| **读文件** | `fs.readFile(path, 'utf-8', (err, data) => {})` |
| **写文件** | `fs.writeFile(path, content, (err) => {})` |
| **追加文件** | `fs.appendFile(path, content, (err) => {})` |
| **删除文件** | `fs.unlink(path, (err) => {})` |
| **创建目录** | `fs.mkdir(path, {recursive: true}, (err) => {})` |
| **读取目录** | `fs.readdir(path, (err, files) => {})` |
| **判断文件** | `fs.stat(path, (err, stats) => { stats.isFile() })` |
| **路径拼接** | `path.join(__dirname, './file.txt')` |
| **绝对路径** | `path.resolve('./file.txt')` |
| **解析 URL** | `new URL(urlStr)` 或 `url.parse(urlStr, true)` |
| **创建服务器** | `http.createServer((req, res) => {}).listen(8080)` |
| **暴露模块** | `module.exports = {};` 或 `exports.name = value` |
| **引入模块** | `const mod = require('./path')` |

---

### Express 常用方法速查

| 功能 | 代码片段 |
|------|--------|
| **GET 路由** | `app.get('/path', (req, res) => {})` |
| **POST 路由** | `app.post('/path', (req, res) => {})` |
| **路径参数** | `app.get('/user/:id', (req, res) => req.params.id)` |
| **查询参数** | `req.query` 或 `req.query.name` |
| **请求体** | `req.body` (需中间件 express.json/urlencoded) |
| **发送字符串** | `res.send('text')` |
| **发送 JSON** | `res.json({name: 'tom'})` |
| **发送文件** | `res.sendFile(path)` |
| **设置状态码** | `res.status(404)` |
| **重定向** | `res.redirect('/path')` |
| **设置响应头** | `res.set('Content-Type', 'text/html')` |
| **全局中间件** | `app.use((req, res, next) => { next() })` |
| **路由前中间件** | `app.use('/path', middleware)` |
| **错误中间件** | `app.use((err, req, res, next) => {})` |
| **静态文件** | `app.use(express.static('./public'))` |
| **启动服务** | `app.listen(8080, () => {})` |

---

### MongoDB 查询速查

| 功能 | 代码片段 |
|------|--------|
| **插入一条** | `Model.create({...}, (err, doc) => {})` |
| **批量插入** | `Model.insertMany([...], (err, docs) => {})` |
| **查询所有** | `Model.find((err, docs) => {})` |
| **条件查询** | `Model.find({name: 'tom'}, (err, docs) => {})` |
| **查询一条** | `Model.findOne({...}, (err, doc) => {})` |
| **根据 ID 查** | `Model.findById(id, (err, doc) => {})` |
| **大于** | `Model.find({age: {$gt: 18}})` |
| **小于** | `Model.find({age: {$lt: 30}})` |
| **包含** | `Model.find({age: {$in: [18, 20]}})` |
| **逻辑或** | `Model.find({$or: [{age: 18}, {age: 20}]})` |
| **排序** | `Model.find().sort({age: -1})` |
| **分页** | `Model.find().skip(10).limit(10)` |
| **字段筛选** | `Model.find().select({name: 1, age: 0})` |
| **更新一条** | `Model.updateOne({...}, {$set: {...}})` |
| **批量更新** | `Model.updateMany({...}, {$set: {...}})` |
| **删除一条** | `Model.deleteOne({...})` |
| **批量删除** | `Model.deleteMany({...})` |

---

### 会话控制速查

| 功能 | 代码片段 |
|------|--------|
| **设置 Cookie** | `res.cookie('name', 'value', {maxAge: 3600000})` |
| **读取 Cookie** | `req.cookies.name` 或 `req.cookies` |
| **删除 Cookie** | `res.clearCookie('name')` |
| **设置 Session** | `req.session.userId = 123` |
| **读取 Session** | `req.session.userId` |
| **删除 Session** | `req.session.destroy()` |
| **生成 JWT** | `jwt.sign({id: 1}, secret, {expiresIn: '24h'})` |
| **验证 JWT** | `jwt.verify(token, secret, (err, decoded) => {})` |
| **中间件验证** | `function verify(req, res, next) { ... next() }` |

---

## 常见错误汇总

### 错误 1：Node.js 单线程误区

**错误说法**：Node.js 是单线程、单进程单线程架构

**正确理解**：
- Node.js **主线程单线程**（事件循环运行在主线程）
- 底层 **libuv 库维护线程池**（默认 4 线程）处理 I/O 操作
- Node.js 本身是**多进程架构**（可用 cluster 模块创建多进程）
- 只是开发时主要操作单线程事件循环

---

### 错误 2：模块路径参照混淆

**错误说法**：自定义模块相对路径"参照命令行所在的目录"

**正确理解**：
- **自定义模块相对路径**参照**所在文件的目录**
- **命令行相对路径**参照**当前工作目录**
- 两者无关，容易混淆

示例：
```js
// 即使在不同目录执行，仍是相对于文件位置
// src/db.js 中 require('./config') 永远找 src/config
```

---

### 错误 3：Express next() 参数混淆

**错误说法**：`next()` 和 `next(err)` 作用相同

**正确理解**：
- `next()` ：传递给下一个**普通中间件**
- `next(err)` ：跳过所有普通中间件，直接进入**错误处理中间件**
- `next('route')` ：跳过当前路由的其他回调

```js
app.get('/test', 
  (req, res, next) => {
    if (err) next(err);    // 进入错误中间件
    else next();           // 进入下一个普通中间件
  },
  (req, res) => {
    res.send('OK');
  }
);
```

---

### 错误 4：Session 存储误区

**错误说法**：express-session 默认支持分布式/集群部署

**正确理解**：
- express-session 默认存储在**内存**中
- 只能在**单机**使用
- 生产环境必须配合 **store**（connect-mongo、redis 等）
- 才能实现分布式/集群部署

```js
// 单机可以，但不推荐
app.use(session({...}));

// 生产环境应配合 store
const MongoStore = require('connect-mongo');
app.use(session({
  store: new MongoStore({mongoUrl: 'mongodb://...'}),
  ...
}));
```

---

### 错误 5：JWT 撤销误区

**错误说法**：JWT 可以通过服务器撤销

**正确理解**：
- JWT 是**无状态**的，服务器无法主动撤销
- 只能通过间接方案：
  - **Token blacklist**：将撤销的 token 加入黑名单（违背无状态）
  - **缩短过期时间**：减少 token 有效期
  - **Refresh Token**：发放短期 token，用 refresh token 更新

- 需要撤销功能时，应该用 **Session** 而非 JWT

---

### 其他常见问题

| 问题 | 解决方案 |
|------|--------|
| 端口被占用 | `netstat -ano \| findstr 端口号` 查找占用进程，kill 掉 |
| 模块缓存问题 | 删除 require 缓存：`delete require.cache[require.resolve('./mod')]` |
| 路由顺序问题 | 特定路由写在前面，通配符 * 写在最后 |
| 中间件不执行 | 检查是否调用了 next() |
| req.body 为 undefined | 检查是否使用了 express.json() 或 express.urlencoded() |
| CORS 跨域问题 | 设置响应头 Access-Control-Allow-Origin |
| MongoDB 连接不上 | 检查 mongod 服务是否启动，检查端口号 27017 |
| 文件找不到 | 使用绝对路径 `path.resolve` 或 `__dirname` |

---

## 项目实战建议

### 项目目录结构示例

```
myapp/
├── bin/
│   └── www                 # 启动脚本
├── routes/
│   ├── index.js            # 首页路由
│   ├── users.js            # 用户路由
│   └── articles.js         # 文章路由
├── models/
│   ├── user.js             # 用户模型
│   ├── article.js          # 文章模型
│   └── db.js               # 数据库连接
├── middleware/
│   ├── auth.js             # 认证中间件
│   └── errorHandler.js     # 错误处理中间件
├── views/
│   ├── index.ejs           # 首页模板
│   ├── user/
│   │   ├── login.ejs
│   │   └── profile.ejs
│   └── article/
│       └── list.ejs
├── public/
│   ├── css/
│   ├── js/
│   └── images/
├── app.js                  # 应用主文件
├── package.json
└── .gitignore
```

### 推荐依赖

```json
{
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "^6.0.0",
    "express-session": "^1.17.0",
    "cookie-parser": "^1.4.0",
    "jsonwebtoken": "^9.0.0",
    "ejs": "^3.1.0",
    "moment": "^2.29.0",
    "dotenv": "^16.0.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.0"
  }
}
```

---

**文档生成日期：2026-05-25**
