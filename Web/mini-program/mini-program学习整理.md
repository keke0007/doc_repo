# 小程序学习整理

这份笔记基于 `mini-program` 目录下的 7 份 Markdown 归纳整理，目标不是简单拼接原文，而是按微信小程序的学习路径重新组织内容，并给每个核心知识点补上代码示例和应用场景，方便系统学习和回顾。

## 学习路线

建议按下面的顺序学习：

1. 先理解什么是小程序、它的定位和开发模型
2. 再掌握小程序的项目结构、配置文件和宿主环境
3. 接着学习 WXML、WXSS、WXS、数据绑定和列表渲染
4. 然后学习内置组件和事件处理
5. 再进入自定义组件、组件通信、插槽、behaviors
6. 最后学习系统 API、网络请求、存储、页面跳转和登录流程

## 原始资料对应关系

| 主题 | 原文件 |
|---|---|
| 小程序入门 | `01_邂逅小程序开发.md` |
| 小程序配置与架构 | `02_小程序配置和架构.md` |
| 小程序内置组件 | `03_小程序的内置组件.md` |
| WXSS / WXML / WXS | `04_WXSS-WXML-WXS.md` |
| 小程序事件处理 | `05_小程序的事件处理.md` |
| 小程序组件化开发 | `06_小程序组件化开发.md` |
| 小程序系统 API | `07_小程序系统API调用.md` |

## 一、认识小程序

### 1. 什么是小程序

小程序可以理解为运行在宿主平台中的轻量级应用，例如微信小程序、支付宝小程序等。

它的特点通常包括：

- 不需要单独下载安装 App
- 打开即用
- 依赖宿主平台能力
- 适合轻量业务和高频服务场景

应用场景：

- 外卖、商城、预约、打卡、工具类应用
- 企业内部轻应用
- 活动页、会员中心、积分商城

### 2. 为什么要学小程序

- 微信生态用户基数大
- 对于业务转化、拉新、留存很有价值
- 前端工程师上手相对快

### 3. 小程序的开发模型

小程序整体上也可以理解为一种 MVVM 思路：

- `WXML` 负责结构
- `WXSS` 负责样式
- `JS` 负责逻辑
- `JSON` 负责配置

代码示例：

```js
Page({
  data: {
    message: "Hello Mini Program"
  }
})
```

```xml
<view>{{message}}</view>
```

应用场景：

- 页面上展示动态数据
- 用户点击按钮后更新界面

## 二、小程序项目结构与配置

### 1. 项目结构

一个典型小程序项目通常包括：

```text
miniprogram
├─ app.js
├─ app.json
├─ app.wxss
├─ pages
│  ├─ home
│  │  ├─ home.js
│  │  ├─ home.json
│  │  ├─ home.wxml
│  │  └─ home.wxss
```

### 2. 全局配置 `app.json`

```json
{
  "pages": [
    "pages/home/home",
    "pages/profile/profile"
  ],
  "window": {
    "navigationBarTitleText": "小程序示例",
    "navigationBarBackgroundColor": "#ffffff"
  }
}
```

应用场景：

- 配置页面路径
- 配置导航栏标题、颜色
- 配置 tabBar

### 3. 页面配置

```json
{
  "navigationBarTitleText": "首页"
}
```

应用场景：

- 某个页面单独改标题、下拉刷新等配置

### 4. `App()` 注册应用

```js
App({
  onLaunch() {
    console.log("小程序启动")
  },
  globalData: {
    token: ""
  }
})
```

应用场景：

- 启动时读取本地缓存
- 保存全局共享数据

### 5. `Page()` 注册页面

```js
Page({
  data: {
    count: 0
  },
  onLoad() {
    console.log("页面加载")
  }
})
```

应用场景：

- 编写页面数据
- 编写页面事件和生命周期

## 三、小程序架构模型

### 1. 宿主环境

小程序不是运行在浏览器里，而是运行在微信客户端这个宿主环境中。

### 2. 逻辑层与渲染层

小程序通常分为：

- 逻辑层：运行 JS
- 渲染层：运行 WXML / WXSS

理解重点：

- 逻辑层和渲染层是分开的
- 数据更新通常通过 `setData` 同步到视图层

代码示例：

```js
Page({
  data: {
    count: 0
  },
  increment() {
    this.setData({
      count: this.data.count + 1
    })
  }
})
```

应用场景：

- 点击按钮后更新页面数字

## 四、WXML 基础

### 1. 数据绑定

```xml
<view>{{message}}</view>
```

```js
Page({
  data: {
    message: "Hello WXML"
  }
})
```

应用场景：

- 标题展示
- 用户昵称展示

### 2. 条件渲染 `wx:if`

```xml
<view wx:if="{{isLogin}}">欢迎回来</view>
<view wx:else>请先登录</view>
```

应用场景：

- 登录态切换
- 空状态/加载态展示

### 3. `hidden`

```xml
<view hidden="{{isHidden}}">这是一段内容</view>
```

理解：

- `hidden` 更像控制显隐
- `wx:if` 是是否渲染

### 4. 列表渲染 `wx:for`

```xml
<view wx:for="{{books}}" wx:key="id">
  {{item.name}}
</view>
```

```js
Page({
  data: {
    books: [
      { id: 1, name: "JavaScript" },
      { id: 2, name: "TypeScript" }
    ]
  }
})
```

应用场景：

- 商品列表
- 分类列表
- 订单列表

### 5. `block`

`block` 不会渲染成真实节点，但可以包裹一组模板。

```xml
<block wx:for="{{list}}" wx:key="id">
  <view>{{item.title}}</view>
  <text>{{item.desc}}</text>
</block>
```

应用场景：

- 多个节点一起参与条件渲染或列表渲染

## 五、WXSS 基础

### 1. 三种样式写法

- 行内样式
- 页面样式
- 全局样式

代码示例：

```xml
<view style="color: red;">行内样式</view>
```

```css
.title {
  color: blue;
}
```

### 2. `rpx`

`rpx` 是小程序中很常用的响应式尺寸单位。

```css
.card {
  width: 300rpx;
  height: 180rpx;
}
```

应用场景：

- 移动端适配
- 不同屏幕下相对稳定的布局

### 3. 页面与全局样式

全局 `app.wxss`：

```css
page {
  background-color: #f5f5f5;
}
```

页面 `home.wxss`：

```css
.container {
  padding: 24rpx;
}
```

应用场景：

- 全局重置样式
- 页面单独定制布局

## 六、WXS

### 1. 什么是 WXS

WXS 是运行在视图层的一种脚本语言，主要用于模板中的简单处理逻辑。

### 2. 基本写法

```xml
<wxs module="tools">
module.exports = {
  formatPrice(price) {
    return "￥" + price
  }
}
</wxs>

<view>{{tools.formatPrice(price)}}</view>
```

应用场景：

- 模板中做简单格式化
- 避免把特别细碎的显示逻辑全堆在页面 JS 中

注意：

- 更复杂的业务逻辑通常还是建议写在 JS 中

## 七、内置组件

### 1. `view`

最常见的容器组件，类似块级盒子。

```xml
<view class="container">内容区域</view>
```

应用场景：

- 页面布局
- 卡片容器
- 列表项容器

### 2. `text`

用于显示文本。

```xml
<text selectable="true">这是一段文字</text>
```

应用场景：

- 标题、描述、价格、时间等文字内容

### 3. `button`

```xml
<button type="primary">确定</button>
```

应用场景：

- 提交按钮
- 登录按钮
- 分享按钮

### 4. `image`

```xml
<image src="/assets/banner.png" mode="widthFix"></image>
```

应用场景：

- 商品图
- 头像
- 轮播图

### 5. `scroll-view`

```xml
<scroll-view scroll-y="true" class="list">
  <view wx:for="{{list}}" wx:key="id">{{item.name}}</view>
</scroll-view>
```

```css
.list {
  height: 400rpx;
}
```

应用场景：

- 可滚动区域
- 局部列表滚动

## 八、事件处理

### 1. 事件绑定

```xml
<button bindtap="handleClick">点击我</button>
```

```js
Page({
  handleClick() {
    console.log("按钮被点击")
  }
})
```

应用场景：

- 按钮点击
- 卡片点击
- 页面跳转

### 2. 事件对象

```js
Page({
  handleClick(event) {
    console.log(event)
  }
})
```

常见内容：

- `event.currentTarget`
- `event.target`
- `event.detail`

### 3. 传递参数

```xml
<button bindtap="handleItemClick" data-id="{{item.id}}">
  查看详情
</button>
```

```js
Page({
  handleItemClick(event) {
    const id = event.currentTarget.dataset.id
    console.log(id)
  }
})
```

应用场景：

- 列表项点击带上 id
- 按钮操作带上商品编号

### 4. 事件冒泡

```xml
<view bindtap="handleParent">
  <button bindtap="handleChild">按钮</button>
</view>
```

如果不希望冒泡，可以使用：

```xml
<button catchtap="handleChild">按钮</button>
```

应用场景：

- 弹层点击遮罩关闭
- 内部按钮点击不触发外层事件

## 九、自定义组件

### 1. 创建组件

组件目录：

```text
components
└─ nav-bar
   ├─ nav-bar.js
   ├─ nav-bar.json
   ├─ nav-bar.wxml
   └─ nav-bar.wxss
```

`nav-bar.json`：

```json
{
  "component": true
}
```

### 2. 组件基本写法

```js
Component({
  properties: {},
  data: {},
  methods: {}
})
```

应用场景：

- 导航栏
- 商品卡片
- 搜索框
- 评分组件

### 3. 使用组件

页面配置：

```json
{
  "usingComponents": {
    "nav-bar": "/components/nav-bar/nav-bar"
  }
}
```

页面模板：

```xml
<nav-bar></nav-bar>
```

## 十、组件通信与插槽

### 1. properties

```js
Component({
  properties: {
    title: {
      type: String,
      value: "默认标题"
    }
  }
})
```

父页面：

```xml
<nav-bar title="首页"></nav-bar>
```

应用场景：

- 自定义导航栏标题
- 商品卡片传入价格、封面、标签

### 2. 自定义事件

子组件：

```js
Component({
  methods: {
    handleClick() {
      this.triggerEvent("add", { count: 1 })
    }
  }
})
```

父页面：

```xml
<counter-button bind:add="handleAdd"></counter-button>
```

```js
Page({
  handleAdd(event) {
    console.log(event.detail.count)
  }
})
```

应用场景：

- 子组件通知父页面增加数量
- 组件内搜索后通知外部发请求

### 3. externalClasses

组件：

```js
Component({
  externalClasses: ["custom-class"]
})
```

使用：

```xml
<my-card custom-class="big-card"></my-card>
```

应用场景：

- 外部想控制组件样式，但又不想改组件内部结构

### 4. 插槽

组件：

```xml
<view class="card">
  <slot></slot>
</view>
```

使用：

```xml
<my-card>
  <view>这里是插入内容</view>
</my-card>
```

应用场景：

- 卡片容器
- 弹窗内容区
- 自定义导航栏左右区域

### 5. 多插槽

组件配置：

```json
{
  "component": true,
  "options": {
    "multipleSlots": true
  }
}
```

组件模板：

```xml
<view class="header"><slot name="header"></slot></view>
<view class="content"><slot></slot></view>
```

使用：

```xml
<my-layout>
  <view slot="header">头部内容</view>
  <view>主体内容</view>
</my-layout>
```

## 十一、behaviors 与组件生命周期

### 1. behaviors

```js
// behaviors/common.js
module.exports = Behavior({
  methods: {
    log() {
      console.log("公共逻辑")
    }
  }
})
```

组件中使用：

```js
const commonBehavior = require("../behaviors/common")

Component({
  behaviors: [commonBehavior]
})
```

应用场景：

- 多个组件复用公共逻辑
- 类似 Vue mixin / React 自定义 Hook 的部分用途

### 2. 组件生命周期

```js
Component({
  lifetimes: {
    attached() {
      console.log("组件进入页面节点树")
    },
    detached() {
      console.log("组件离开页面节点树")
    }
  }
})
```

应用场景：

- 组件挂载时初始化数据
- 组件卸载时清理定时器

## 十二、系统 API

### 1. 网络请求 `wx.request`

```js
wx.request({
  url: "https://api.example.com/banner",
  method: "GET",
  success(res) {
    console.log(res.data)
  },
  fail(err) {
    console.log(err)
  }
})
```

应用场景：

- 获取轮播图
- 获取商品列表
- 提交表单数据

### 2. 网络请求封装

```js
function request(url, data = {}, method = "GET") {
  return new Promise((resolve, reject) => {
    wx.request({
      url,
      data,
      method,
      success: resolve,
      fail: reject
    })
  })
}
```

应用场景：

- 项目里统一处理请求逻辑
- 方便加 token、错误提示、loading

### 3. 弹窗

```js
wx.showModal({
  title: "提示",
  content: "确定删除吗？",
  success(res) {
    if (res.confirm) {
      console.log("用户点击确定")
    }
  }
})
```

应用场景：

- 删除确认
- 风险提示

### 4. 分享

```js
Page({
  onShareAppMessage() {
    return {
      title: "分享标题",
      path: "/pages/home/home"
    }
  }
})
```

应用场景：

- 活动页分享
- 商品详情分享

### 5. 获取设备信息

```js
wx.getSystemInfo({
  success(res) {
    console.log(res)
  }
})
```

应用场景：

- 适配不同设备
- 判断屏幕宽高

### 6. 获取位置信息

```js
wx.getLocation({
  type: "gcj02",
  success(res) {
    console.log(res.latitude, res.longitude)
  }
})
```

应用场景：

- 附近门店
- 地图选点
- 定位服务

### 7. Storage

同步方式：

```js
wx.setStorageSync("token", "abc123")
const token = wx.getStorageSync("token")
```

异步方式：

```js
wx.setStorage({
  key: "city",
  data: "shanghai"
})
```

应用场景：

- 存 token
- 存用户偏好
- 存历史记录

## 十三、页面跳转

### 1. `navigateTo`

```js
wx.navigateTo({
  url: "/pages/detail/detail?id=1001"
})
```

应用场景：

- 从列表页跳详情页

### 2. `navigateBack`

```js
wx.navigateBack({
  delta: 1
})
```

### 3. 页面参数接收

```js
Page({
  onLoad(options) {
    console.log(options.id)
  }
})
```

应用场景：

- 详情页拿商品 id
- 搜索结果页拿 keyword

## 十四、小程序登录流程

### 1. 为什么需要登录

登录的核心目标通常是：

- 识别当前用户身份
- 获取后端授权 token
- 实现个性化数据能力

### 2. 获取 code

```js
wx.login({
  success(res) {
    console.log(res.code)
  }
})
```

### 3. 服务端换取登录态

通常流程：

1. 小程序端调用 `wx.login`
2. 获取 `code`
3. 把 `code` 发给后端
4. 后端去微信服务端换取 `openid / session_key`
5. 后端生成自己的登录 token 返回给小程序

应用场景：

- 用户系统
- 订单系统
- 收藏、购物车、会员信息

### 4. 手机号授权

在一些业务里，登录后还需要绑定手机号完成业务身份识别。

## 十五、这套小程序内容的核心学习重点

如果时间有限，建议优先掌握：

### 第一优先级

- 项目结构和配置文件
- WXML 数据绑定 / 条件渲染 / 列表渲染
- 事件处理
- 网络请求
- 页面跳转

### 第二优先级

- 内置组件
- 自定义组件
- 组件通信
- Storage
- 登录流程

### 第三优先级

- WXS
- behaviors
- 设备信息 / 定位 / 分享等扩展 API

## 十六、容易混淆的知识点

### 1. `wx:if` 和 `hidden`

- `wx:if`：控制是否渲染
- `hidden`：控制是否显示

### 2. `bindtap` 和 `catchtap`

- `bindtap`：会冒泡
- `catchtap`：阻止冒泡

### 3. 页面和组件

- 页面用 `Page()`
- 组件用 `Component()`

### 4. 同步存储和异步存储

- `setStorageSync` 简单直接
- `setStorage` 更适合异步流程统一处理

## 十七、建议你的复习方式

### 第一轮

- 顺着这份笔记走一遍
- 明白每个知识点解决什么问题

### 第二轮

- 把每个代码示例手敲一遍
- 重点练：数据绑定、`wx:for`、事件处理、组件通信、网络请求

### 第三轮

- 做一个小项目
- 比如：商品列表、小型商城首页、打卡页面、工具类应用

### 第四轮

- 在项目里真实接一次登录接口、列表接口、详情页跳转

## 十八、一个最小知识树

```text
Mini Program
├─ 入门
│  ├─ 认识小程序
│  ├─ 项目结构
│  ├─ app / page
│  └─ 架构模型
├─ 视图层
│  ├─ WXML
│  ├─ WXSS
│  ├─ WXS
│  ├─ 内置组件
│  └─ 事件处理
├─ 组件化
│  ├─ 自定义组件
│  ├─ properties
│  ├─ 自定义事件
│  ├─ 插槽
│  └─ behaviors
└─ 系统能力
   ├─ 网络请求
   ├─ Storage
   ├─ 页面跳转
   ├─ 分享
   ├─ 定位
   └─ 登录流程
```

## 十九、学完这部分后的目标

学完这部分后，你至少应该能做到：

- 独立搭建一个基础小程序项目
- 熟练使用 WXML / WXSS / Page / Component
- 实现列表渲染、事件交互、组件拆分
- 封装基础请求函数
- 完成页面跳转和参数传递
- 理解小程序登录和本地存储的基本流程
