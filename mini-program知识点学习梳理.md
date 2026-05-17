# mini-program 知识点学习梳理

本文根据 `mini-program` 文件夹中的课程主题整理，覆盖内容包括：

- 邂逅小程序开发
- 小程序配置和架构
- 小程序的内置组件
- WXSS / WXML / WXS
- 小程序的事件处理
- 小程序组件化开发
- 小程序系统 API 调用

目标是把这些内容重组成一份适合学习、复习和查漏补缺的文档。

---

## 1. mini-program 主要学什么

一句话理解：

`mini-program 这一阶段，主要是在学如何开发微信小程序页面和交互。`

它解决的问题包括：

- 如何搭建和运行一个小程序项目
- 如何组织页面和配置
- 如何使用小程序组件搭页面
- 如何做数据绑定和事件交互
- 如何拆分组件
- 如何调用微信提供的系统能力

如果你已经学过 HTML、CSS、JavaScript、Vue，那么可以把小程序理解成：

`一种有自己语法和运行环境的前端开发体系。`

---

## 2. 这套 mini-program 内容的学习主线

这套内容大致可以分成 5 个层次：

1. 认识小程序和开发环境
2. 理解项目结构、页面结构、配置方式
3. 学会 WXML、WXSS、内置组件
4. 学会事件处理和数据交互
5. 学会组件化和系统 API 调用

如果先抓住一句话：

`小程序开发的核心是：页面结构 + 数据绑定 + 事件处理 + API 调用。`

---

## 3. 哪些知识点是必须掌握的

下面这些内容，是后面继续做小程序项目时最常用、最应该优先掌握的基础。

---

## 4. 小程序基础认知

### 4.1 小程序是什么

**必须掌握**

- 小程序是运行在微信生态中的应用形态
- 不需要用户下载安装传统 App
- 有自己的开发框架、组件体系和 API

**应用场景**

- 电商小程序
- 点餐小程序
- 社区团购
- 企业服务工具
- 预约和报名系统

---

### 4.2 开发工具与运行方式

**必须掌握**

- 微信开发者工具
- 项目创建
- 预览和真机调试

**应用场景**

- 本地开发
- 调试页面
- 发布前测试

---

## 5. 小程序配置和架构

### 5.1 项目基本结构

**必须掌握**

常见文件和目录：

- `app.js`
- `app.json`
- `app.wxss`
- `pages/`
- 各页面的 `.wxml`、`.wxss`、`.js`、`.json`

**必须理解**

- `app.json` 负责全局配置
- 页面通常由 4 类文件组成

**应用场景**

- 新建页面
- 配置路由
- 配置窗口样式
- 维护项目结构

---

### 5.2 全局配置与页面配置

**必须掌握**

- 页面路径配置
- 窗口配置
- tabBar 配置
- 页面单独配置

**学习示例**

```json
{
  "pages": [
    "pages/home/home",
    "pages/profile/profile"
  ],
  "window": {
    "navigationBarTitleText": "小程序学习"
  }
}
```

**应用场景**

- 配置首页
- 配置导航栏标题
- 配置 tabBar 页面切换

---

## 6. WXML、WXSS、WXS

### 6.1 WXML

**必须掌握**

- WXML 是小程序的页面结构语言
- 类似 HTML，但不是完全一样
- 使用内置组件来描述页面

**学习示例**

```xml
<view class="container">
  <text>{{ title }}</text>
</view>
```

**应用场景**

- 写页面结构
- 渲染文本、图片、列表、表单

---

### 6.2 数据绑定

**必须掌握**

- 文本插值：`{{ }}`
- 属性绑定
- 列表渲染
- 条件渲染

**学习示例**

```xml
<view>{{ username }}</view>
<image src="{{ avatarUrl }}"></image>
```

```js
Page({
  data: {
    username: '小明',
    avatarUrl: '/assets/avatar.png'
  }
})
```

**应用场景**

- 展示用户信息
- 展示商品信息
- 动态控制页面内容

---

### 6.3 列表渲染与条件渲染

**必须掌握**

- `wx:for`
- `wx:key`
- `wx:if`
- `wx:elif`
- `wx:else`
- `hidden` 需要知道适用场景

**学习示例**

```xml
<view wx:if="{{ isLogin }}">欢迎回来</view>
<view wx:else>请先登录</view>

<view wx:for="{{ list }}" wx:key="id">
  {{ item.name }}
</view>
```

**应用场景**

- 登录状态切换
- 商品列表
- 评论列表
- 菜单列表

**必须理解**

- 列表渲染必须关注 `wx:key`
- 条件渲染是小程序页面逻辑的基础

---

### 6.4 WXSS

**必须掌握**

- WXSS 是小程序样式语言
- 和 CSS 很相似
- 支持尺寸单位 `rpx`

**学习示例**

```css
.container {
  padding: 20rpx;
}

.title {
  font-size: 32rpx;
  color: #333;
}
```

**应用场景**

- 页面排版
- 响应式适配
- 样式复用

**必须理解**

- `rpx` 是小程序布局中非常关键的单位

---

### 6.5 WXS

**建议掌握程度：理解**

- WXS 是小程序中的脚本语言模块
- 用于模板中的简单逻辑处理

**应用场景**

- 模板中的简单格式化
- 轻量逻辑处理

**学习建议**

- 初学阶段知道作用即可
- 真实项目中不一定高频使用

---

## 7. 小程序内置组件

### 7.1 基础容器和文本组件

**必须掌握**

- `view`
- `text`
- `image`

**学习示例**

```xml
<view class="card">
  <image src="{{ cover }}" class="cover"></image>
  <text class="title">{{ title }}</text>
</view>
```

**应用场景**

- 列表项
- 商品卡片
- 文章信息展示

---

### 7.2 表单和交互组件

**必须掌握**

- `button`
- `input`
- `textarea`
- `switch`
- `checkbox`
- `radio`

**应用场景**

- 登录注册
- 搜索框
- 用户信息填写
- 设置页面

---

### 7.3 滚动和布局组件

**必须掌握程度：常用**

- `scroll-view`
- `swiper`

**应用场景**

- 横向或纵向滚动区域
- 轮播图

---

## 8. 小程序事件处理

### 8.1 事件绑定

**必须掌握**

- `bindtap`
- `catchtap`
- 其他常见表单事件

**学习示例**

```xml
<button bindtap="handleClick">点击我</button>
```

```js
Page({
  handleClick() {
    console.log('按钮被点击了')
  }
})
```

**应用场景**

- 按钮点击
- 卡片点击
- 页面跳转

**必须理解**

- `bind` 事件会冒泡
- `catch` 事件会阻止冒泡

---

### 8.2 事件对象与传参

**必须掌握**

- 事件对象 `event`
- `data-*` 传参

**学习示例**

```xml
<button bindtap="handleItem" data-id="{{ item.id }}">查看详情</button>
```

```js
Page({
  handleItem(event) {
    const id = event.currentTarget.dataset.id
    console.log(id)
  }
})
```

**应用场景**

- 获取当前点击项的 id
- 列表操作
- 删除、编辑、查看详情

---

### 8.3 更新数据

**必须掌握**

- 小程序更新页面数据要用 `setData`

**学习示例**

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

**应用场景**

- 计数器
- 切换状态
- 表单内容同步
- 列表更新

**必须理解**

- 不能像普通对象那样直接改 `data` 后指望页面自动刷新
- 页面更新依赖 `setData`

---

## 9. 页面生命周期

### 9.1 常见页面生命周期

**必须掌握**

- `onLoad`
- `onShow`
- `onHide`
- `onUnload`

**学习示例**

```js
Page({
  onLoad() {
    console.log('页面加载')
  },
  onShow() {
    console.log('页面显示')
  }
})
```

**应用场景**

- 页面初始化请求数据
- 页面切回来时刷新数据
- 页面离开时清理逻辑

---

## 10. 小程序组件化开发

### 10.1 自定义组件基础

**必须掌握**

- 自定义组件的意义
- 组件目录结构
- 组件注册与使用

**应用场景**

- 商品卡片组件
- 导航栏组件
- 搜索框组件
- 公共弹窗组件

---

### 10.2 组件通信

**必须掌握**

- `properties`
- 组件事件触发

**学习示例**

```js
Component({
  properties: {
    title: String
  },
  methods: {
    handleTap() {
      this.triggerEvent('change', { value: '子组件数据' })
    }
  }
})
```

**应用场景**

- 父组件给子组件传值
- 子组件通知父组件操作结果
- 组件间交互

---

## 11. 小程序系统 API 调用

### 11.1 常见 API 能力

**必须掌握**

- `wx.navigateTo`
- `wx.redirectTo`
- `wx.switchTab`
- `wx.showToast`
- `wx.showModal`
- `wx.request`
- `wx.getStorageSync`
- `wx.setStorageSync`

**学习示例**

```js
wx.showToast({
  title: '保存成功',
  icon: 'success'
})
```

```js
wx.navigateTo({
  url: '/pages/detail/detail?id=1'
})
```

**应用场景**

- 页面跳转
- 成功提示
- 确认弹窗
- 请求接口
- 缓存 token 或用户信息

---

### 11.2 网络请求

**必须掌握**

- `wx.request`
- 成功失败处理
- 请求封装思路

**学习示例**

```js
wx.request({
  url: 'https://api.example.com/list',
  method: 'GET',
  success(res) {
    console.log(res.data)
  }
})
```

**应用场景**

- 商品列表
- 用户信息
- 订单提交
- 登录接口

---

### 11.3 本地存储

**必须掌握**

- 同步存储
- 读取缓存
- 删除缓存

**学习示例**

```js
wx.setStorageSync('token', '123456')
const token = wx.getStorageSync('token')
```

**应用场景**

- 保存登录状态
- 保存用户偏好
- 保存历史记录

---

## 12. mini-program 必须掌握到什么程度

### 12.1 必须会默写

- 小程序页面基本结构
- `app.json` 基础配置
- WXML 数据绑定
- `wx:for`
- `wx:if`
- `bindtap`
- `setData`
- `onLoad`
- `properties`
- `wx.navigateTo`
- `wx.showToast`
- `wx.request`

### 12.2 必须会独立写

- 一个简单首页
- 一个商品列表页
- 一个登录表单页
- 一个点击切换内容的小案例
- 一个自定义卡片组件
- 一个请求接口并渲染列表的小页面

### 12.3 必须能看懂并改动

- 页面目录结构
- 页面配置
- 基础 WXML / WXSS
- 基础事件逻辑
- 基础组件通信
- 基础系统 API 调用

---

## 13. 推荐学习顺序

建议按这个顺序学：

1. 认识小程序和开发环境
2. 小程序配置和项目结构
3. WXML
4. WXSS
5. 内置组件
6. 数据绑定
7. 事件处理
8. `setData`
9. 页面生命周期
10. 组件化开发
11. 系统 API
12. 网络请求和本地存储
13. WXS 作为补充理解

---

## 14. 一组最实用的综合示例

下面这个例子把小程序开发中的几个核心知识点串起来了：

```xml
<view class="page">
  <input placeholder="请输入任务" bindinput="handleInput" />
  <button bindtap="handleAdd">添加</button>

  <view wx:for="{{ list }}" wx:key="id">
    {{ item.name }}
  </view>
</view>
```

```js
Page({
  data: {
    keyword: '',
    list: []
  },

  handleInput(event) {
    this.setData({
      keyword: event.detail.value
    })
  },

  handleAdd() {
    if (!this.data.keyword) {
      wx.showToast({
        title: '请输入内容',
        icon: 'none'
      })
      return
    }

    const newItem = {
      id: Date.now(),
      name: this.data.keyword
    }

    this.setData({
      list: [...this.data.list, newItem],
      keyword: ''
    })
  }
})
```

这个例子涉及：

- WXML 结构
- 数据绑定
- `bindinput`
- `bindtap`
- `setData`
- `wx:for`
- `wx.showToast`

如果你能独立写出这个例子，说明小程序基础已经比较稳了。

---

## 15. 每个知识点的典型应用场景速查

| 知识点 | 典型场景 |
| --- | --- |
| 项目结构 | 页面管理、配置维护 |
| `app.json` | 路由配置、窗口设置、tabBar |
| WXML | 页面结构编写 |
| WXSS | 页面样式与适配 |
| `rpx` | 多设备尺寸适配 |
| `wx:for` | 商品列表、评论列表、菜单列表 |
| `wx:if` | 登录状态、权限控制、模块切换 |
| 内置组件 | 页面搭建、表单交互、轮播展示 |
| `bindtap` | 点击事件 |
| `setData` | 页面数据更新 |
| 生命周期 | 页面初始化和刷新 |
| 自定义组件 | 公共卡片、导航、弹窗 |
| `wx.navigateTo` | 页面跳转 |
| `wx.request` | 请求接口 |
| `wx.showToast` | 操作提示 |
| `storage` | 保存 token、缓存数据 |

---

## 16. 给你的学习建议

### 16.1 第一阶段：先把页面写出来

优先练：

- 页面结构
- 样式
- 列表渲染
- 条件渲染

### 16.2 第二阶段：尽快补事件和数据更新

小程序真正“动起来”的关键在：

- 事件绑定
- `setData`
- 页面生命周期

### 16.3 第三阶段：进入工程化思维

你要开始思考：

- 公共模块如何拆成组件
- 页面如何组织
- 请求如何封装
- 缓存如何管理

---

## 17. 最后总结

如果只用一句话概括 mini-program 阶段最重要的任务，那就是：

`先学会用 WXML、WXSS 和事件处理把页面写出来，再学会用组件和系统 API 做真实交互。`

优先掌握下面这些内容：

- 小程序项目结构
- 配置文件
- WXML
- WXSS
- 内置组件
- 数据绑定
- 事件处理
- `setData`
- 页面生命周期
- 组件化开发
- `wx.request`
- `wx.navigateTo`
- 本地存储

这些内容掌握后，你就已经具备继续做小程序实战项目的基础了。

