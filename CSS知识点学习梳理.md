# CSS 知识点学习梳理

本文根据 `css/day06` 到 `css/day21` 的讲义内容整理，目标是把零散的 CSS 知识点重组成一份适合学习、复习和查漏补缺的文档。

---

## 1. CSS 主要是干什么的

一句话理解：

`HTML 负责内容和结构，CSS 负责样式和布局。`

CSS 主要解决两类问题：

1. 美化页面
2. 控制页面布局

比如：

- 设置文字大小、颜色、字体
- 设置背景、边框、圆角、阴影
- 控制元素的宽高、间距、位置
- 做导航栏、卡片、两栏布局、三栏布局
- 做水平居中、垂直居中、响应式排列

---

## 2. 哪些 CSS 知识点是必须掌握的

如果你现在是 HTML 之后准备继续学前端，下面这些知识点是必须优先掌握的。

---

## 3. CSS 基础语法与引入方式

### 3.1 CSS 基础语法

**必须掌握**

- 选择器
- 属性名
- 属性值
- 一条声明以 `;` 结尾

**学习示例**

```css
p {
  color: red;
  font-size: 18px;
}
```

这里：

- `p` 是选择器
- `color`、`font-size` 是属性名
- `red`、`18px` 是属性值

**应用场景**

- 所有 CSS 编写的基础

---

### 3.2 CSS 的三种引入方式

**必须掌握**

- 行内样式：写在标签的 `style` 属性里
- 内部样式：写在 `<style>` 标签里
- 外部样式：写在独立 `.css` 文件里，再通过 `<link>` 引入

**学习示例**

```html
<link rel="stylesheet" href="./style.css" />
```

```css
h1 {
  color: blue;
}
```

**应用场景**

- 实际项目中最常用的是外部样式表
- 行内样式常见于临时测试或少量动态样式

**学习建议**

- 初学时三种都要认识
- 真正开发时优先使用外部样式表

---

## 4. 选择器

### 4.1 基础选择器

**必须掌握**

- 元素选择器：`p`
- 类选择器：`.title`
- id 选择器：`#header`
- 通配符选择器：`*`

**学习示例**

```css
p {
  color: #333;
}

.title {
  font-size: 24px;
}

#banner {
  background-color: #f5f5f5;
}
```

**应用场景**

- 给页面中不同元素加样式
- 通过 `class` 做样式复用
- 通过 `id` 标记特殊区域

**必须记住**

- `class` 是开发中最常用的
- `id` 一般要求页面唯一

---

### 4.2 组合选择器

**必须掌握**

- 后代选择器：`.box p`
- 子代选择器：`.box > p`
- 交集选择器：`p.title`
- 并集选择器：`h1, h2, h3`

**学习示例**

```css
.article p {
  line-height: 1.8;
}

.nav > a {
  text-decoration: none;
}

p.desc {
  color: gray;
}

h1, h2, h3 {
  font-weight: 700;
}
```

**应用场景**

- 精准选中某个模块中的元素
- 给多个标题统一样式
- 控制局部区域样式

---

### 4.3 常用伪类

**必须掌握**

- `:hover`
- `:focus`
- `:first-child`
- `:last-child`
- `:nth-child()`
- `:not()`

**学习示例**

```css
a:hover {
  color: orange;
}

input:focus {
  border-color: #409eff;
}

li:nth-child(2n) {
  background: #f7f7f7;
}
```

**应用场景**

- 鼠标悬停效果
- 输入框聚焦效果
- 斑马纹列表
- 特定序号元素样式控制

**学习建议**

- `:hover`、`:focus`、`:nth-child()` 优先掌握
- 更复杂的结构伪类先理解即可

---

### 4.4 常用伪元素

**建议掌握程度：常用**

- `::before`
- `::after`

**学习示例**

```css
.tag::before {
  content: "#";
  color: red;
}
```

**应用场景**

- 添加装饰符号
- 清除浮动
- 添加小图标、角标

---

## 5. 文本与字体

### 5.1 文本颜色与大小

**必须掌握**

- `color`
- `font-size`

**学习示例**

```css
p {
  color: #333;
  font-size: 16px;
}
```

**应用场景**

- 正文
- 标题
- 按钮文字
- 提示文字

---

### 5.2 字体相关属性

**必须掌握**

- `font-family`
- `font-weight`
- `font-style`
- `line-height`

**学习示例**

```css
body {
  font-family: "Microsoft YaHei", sans-serif;
  font-size: 16px;
  line-height: 1.6;
}

h1 {
  font-weight: 700;
}
```

**应用场景**

- 页面整体排版
- 标题加粗
- 正文阅读体验优化

**必须理解**

- `line-height` 是行高，不是元素总高度
- 很多页面的可读性问题，本质上是字体和行高没设好

---

### 5.3 文本排版属性

**必须掌握**

- `text-align`
- `text-decoration`
- `text-indent`

**学习示例**

```css
h1 {
  text-align: center;
}

a {
  text-decoration: none;
}

p {
  text-indent: 2em;
}
```

**应用场景**

- 标题居中
- 去掉超链接下划线
- 中文段落首行缩进

**补充**

- `letter-spacing`、`word-spacing` 了解即可
- `text-transform` 英文项目更常见

---

## 6. 继承、层叠、优先级

### 6.1 继承

**必须掌握**

常见可继承属性：

- `color`
- `font-size`
- `font-family`
- `line-height`
- `text-align`

**应用场景**

- 给 `body` 设置统一字体和颜色
- 子元素自动继承排版风格

**学习示例**

```css
body {
  color: #333;
  font-family: "Microsoft YaHei", sans-serif;
}
```

---

### 6.2 层叠和优先级

**必须掌握**

CSS 冲突时看两件事：

1. 选择器优先级
2. 谁写在后面

**基础优先级概念**

- `!important`
- 行内样式
- `id`
- `class` / 属性选择器 / 伪类
- 元素选择器 / 伪元素

**应用场景**

- 样式冲突排查
- 改样式不生效时定位问题

**必须建立的意识**

- 不要乱堆 `!important`
- 项目里优先用合理的选择器结构解决问题

---

## 7. 元素显示类型

### 7.1 `display`

**必须掌握**

- `block`
- `inline`
- `inline-block`
- `none`
- `flex`

**学习示例**

```css
a {
  display: inline-block;
}

.hidden {
  display: none;
}
```

**应用场景**

- 让链接像按钮一样设置宽高
- 隐藏元素
- 切换布局方式

**必须理解**

- `inline` 默认不能随意设置宽高
- `inline-block` 很适合做横向按钮、标签、小卡片

---

## 8. 盒子模型

### 8.1 盒子模型的四部分

**必须掌握**

- `content`
- `padding`
- `border`
- `margin`

**必须理解**

一个元素在页面中占空间，看的不是只有内容宽高，而是整个盒子。

---

### 8.2 宽高：`width`、`height`

**必须掌握**

- `width`
- `height`
- `min-width`
- `max-width`

**学习示例**

```css
.card {
  width: 300px;
  min-height: 200px;
}
```

**应用场景**

- 卡片
- 弹窗
- 图片区域
- 页面主内容区

---

### 8.3 内边距：`padding`

**必须掌握**

- 控制内容和边框之间的距离
- 会写单值、双值、四值简写

**学习示例**

```css
.card {
  padding: 20px;
}
```

**应用场景**

- 卡片留白
- 按钮内部空间
- 输入框内部空间

---

### 8.4 边框：`border`

**必须掌握**

- `border-width`
- `border-style`
- `border-color`
- 简写 `border`
- `border-radius`

**学习示例**

```css
.box {
  border: 1px solid #ddd;
  border-radius: 8px;
}
```

**应用场景**

- 卡片边框
- 按钮边框
- 圆角输入框
- 圆形头像

---

### 8.5 外边距：`margin`

**必须掌握**

- 控制元素和元素之间的距离
- 会写简写
- 知道 `margin: 0 auto` 的作用

**学习示例**

```css
.container {
  width: 1200px;
  margin: 0 auto;
}
```

**应用场景**

- 模块之间留白
- 页面内容水平居中

**必须理解**

- `margin` 常用于兄弟元素之间
- `padding` 常用于父子元素之间

---

### 8.6 `box-sizing`

**必须掌握**

- `content-box`
- `border-box`

**学习示例**

```css
* {
  box-sizing: border-box;
}
```

**应用场景**

- 布局时更容易控制元素实际宽高
- 企业项目中非常常见

**必须理解**

- `border-box` 下，设置的宽高更接近你“看到的宽高”

---

## 9. 背景

### 9.1 背景常用属性

**必须掌握**

- `background-color`
- `background-image`
- `background-repeat`
- `background-position`
- `background-size`
- `background`

**学习示例**

```css
.banner {
  background: url("./images/banner.jpg") center/cover no-repeat;
}
```

**应用场景**

- banner 区域
- 卡片背景
- 页面装饰图
- 登录页背景

**必须理解**

- 内容图片优先用 `img`
- 装饰性图片优先用 `background-image`

---

## 10. 定位

### 10.1 `position`

**必须掌握**

- `static`
- `relative`
- `absolute`
- `fixed`
- `sticky`

---

### 10.2 相对定位 `relative`

**必须掌握**

- 元素仍保留原来的位置
- 通过 `top/right/bottom/left` 微调

**学习示例**

```css
.badge {
  position: relative;
  top: -4px;
}
```

**应用场景**

- 轻微调整元素位置
- 给绝对定位子元素提供参照

---

### 10.3 绝对定位 `absolute`

**必须掌握**

- 元素脱离标准流
- 相对最近的定位祖先元素定位
- 如果没有定位祖先，通常相对视口定位

**学习示例**

```css
.card {
  position: relative;
}

.card .tag {
  position: absolute;
  top: 10px;
  right: 10px;
}
```

**应用场景**

- 角标
- 浮层
- 图片上的按钮
- 弹层局部定位

**必须记住**

- 常见搭配是“子绝父相”

---

### 10.4 固定定位 `fixed`

**必须掌握**

- 相对视口定位
- 页面滚动时位置不变

**学习示例**

```css
.back-top {
  position: fixed;
  right: 20px;
  bottom: 20px;
}
```

**应用场景**

- 返回顶部按钮
- 固定客服按钮
- 固定导航

---

### 10.5 粘性定位 `sticky`

**建议掌握程度：常用**

**学习示例**

```css
.header {
  position: sticky;
  top: 0;
}
```

**应用场景**

- 吸顶导航
- 吸顶分类栏

---

### 10.6 `z-index`

**必须掌握**

- 控制定位元素的层叠顺序

**应用场景**

- 弹窗盖住页面
- 导航压住内容
- 遮罩层和弹层层级控制

---

## 11. 浮动

### 11.1 `float`

**建议掌握程度：必须理解，会维护旧代码**

- `left`
- `right`
- `none`

**应用场景**

- 老项目布局
- 图文环绕
- 一些历史代码维护

**结论**

- 现代布局首选 Flex
- 但浮动仍然必须看得懂

---

### 11.2 清除浮动

**必须掌握**

- 浮动会导致父元素高度塌陷
- 常用清除方式是伪元素清除浮动

**学习示例**

```css
.clearfix::after {
  content: "";
  display: block;
  clear: both;
}
```

**应用场景**

- 老式多列布局
- 浮动容器收起高度问题

---

## 12. Flex 布局

### 12.1 为什么 Flex 必须掌握

**必须掌握**

Flex 是现代前端最常用的布局方式之一，尤其适合：

- 横向排列
- 纵向排列
- 水平居中
- 垂直居中
- 等分布局
- 自适应布局

---

### 12.2 Flex 基本概念

**必须掌握**

- 容器：`display: flex`
- 项目：容器中的直接子元素
- 主轴
- 交叉轴

**学习示例**

```css
.nav {
  display: flex;
}
```

---

### 12.3 容器常用属性

**必须掌握**

- `flex-direction`
- `justify-content`
- `align-items`
- `flex-wrap`
- `gap` 如果后续项目使用，建议一并掌握

**学习示例**

```css
.nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
```

**应用场景**

- 顶部导航
- 按钮组
- 卡片横向排列
- 表单行布局

---

### 12.4 项目常用属性

**必须掌握**

- `flex`
- `order`
- `align-self`

**学习示例**

```css
.left {
  width: 200px;
}

.right {
  flex: 1;
}
```

**应用场景**

- 左侧固定右侧自适应
- 主内容区自动撑满
- 单个元素独立对齐

**必须理解**

- `flex: 1` 是开发中非常高频的写法

---

## 13. 这些知识点要掌握到什么程度

### 13.1 必须会默写

- CSS 基础语法
- 常见选择器
- `color`
- `font-size`
- `line-height`
- `text-align`
- `width`、`height`
- `padding`
- `margin`
- `border`
- `box-sizing`
- `display`
- `position`
- `background`
- `flex`

### 13.2 必须会独立写

- 一个按钮样式
- 一个卡片样式
- 一个导航栏
- 一个两栏布局
- 一个三列横向排列
- 一个固定定位按钮
- 一个基础 Flex 居中布局

### 13.3 必须能看懂并改动

- 选择器冲突
- 盒模型尺寸问题
- 元素为什么没居中
- 为什么宽高没生效
- 为什么 `z-index` 不起作用
- 为什么父元素高度塌陷

---

## 14. 推荐学习顺序

按这个顺序学，效果最好：

1. CSS 是做什么的
2. CSS 基础语法和三种引入方式
3. 基础选择器和组合选择器
4. 文本和字体属性
5. 继承、层叠、优先级
6. `display`
7. 盒模型：`width`、`height`、`padding`、`border`、`margin`
8. `box-sizing`
9. 背景
10. 定位
11. 浮动与清除浮动
12. Flex 布局

---

## 15. 一组最实用的综合示例

```html
<div class="card">
  <img class="card-cover" src="./images/course.jpg" alt="课程封面" />
  <div class="card-body">
    <h2 class="card-title">CSS 学习卡片</h2>
    <p class="card-desc">掌握盒模型、定位和 Flex，是 CSS 入门阶段的核心。</p>
    <a class="card-btn" href="#">开始学习</a>
  </div>
</div>
```

```css
* {
  box-sizing: border-box;
}

.card {
  display: flex;
  width: 720px;
  margin: 40px auto;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  overflow: hidden;
  background-color: #fff;
}

.card-cover {
  width: 260px;
  height: 180px;
  object-fit: cover;
}

.card-body {
  flex: 1;
  padding: 20px;
}

.card-title {
  margin: 0 0 12px;
  font-size: 24px;
  color: #111827;
}

.card-desc {
  margin: 0 0 16px;
  line-height: 1.7;
  color: #4b5563;
}

.card-btn {
  display: inline-block;
  padding: 10px 16px;
  border-radius: 8px;
  text-decoration: none;
  background-color: #2563eb;
  color: #fff;
}

.card-btn:hover {
  background-color: #1d4ed8;
}
```

这个综合示例包含了：

- 类选择器
- 盒模型
- `display: flex`
- `flex: 1`
- 字体与文本
- 背景色
- 边框和圆角
- `margin`、`padding`
- `a:hover`

如果你能自己写出这个例子，并理解每个属性的作用，说明你的 CSS 基础已经很稳了。

---

## 16. 每个知识点的典型应用场景速查

| 知识点 | 典型场景 |
| --- | --- |
| 选择器 | 给不同模块和元素设置样式 |
| `color` / `font-size` | 标题、正文、按钮文字 |
| `line-height` | 提高正文阅读体验 |
| `text-align` | 标题居中、表格文本对齐 |
| `display` | 隐藏元素、改显示类型、布局切换 |
| `width` / `height` | 卡片、图片、容器尺寸 |
| `padding` | 按钮内边距、卡片留白 |
| `margin` | 模块间距、内容居中 |
| `border` / `border-radius` | 卡片边框、按钮圆角 |
| `box-sizing` | 控制真实占位尺寸 |
| `background` | banner、装饰背景、登录页背景 |
| `position` | 角标、弹窗、固定按钮、吸顶导航 |
| `z-index` | 弹层层级控制 |
| `float` | 维护旧项目、图文环绕 |
| 清除浮动 | 修复父元素高度塌陷 |
| `flex` | 导航栏、列表、卡片、常规布局 |

---

## 17. 给你的学习建议

### 17.1 第一阶段：先把“看得见的样式”练熟

优先练：

- 字体大小
- 颜色
- 背景色
- 边框
- 间距
- 宽高

### 17.2 第二阶段：把盒模型真正吃透

很多 CSS 问题本质都出在：

- 为什么这个盒子比想象中大
- 为什么间距不对
- 为什么居中失败

这些都和盒模型强相关。

### 17.3 第三阶段：重点突破布局

布局学习建议优先级：

1. `display`
2. 盒模型
3. `position`
4. `flex`
5. `float` 以理解和维护为主

---

## 18. 最后总结

如果只用一句话概括 CSS 初学阶段最重要的任务，那就是：

`学会给页面加样式，并能用盒模型、定位和 Flex 把页面布局出来。`

优先掌握下面这些内容：

- CSS 基础语法
- 选择器
- 文本和字体属性
- 继承、层叠、优先级
- `display`
- 盒模型
- 背景
- 定位
- 浮动与清除浮动
- Flex 布局

这些内容掌握后，你就已经具备继续做静态页面和进入 JavaScript 阶段的 CSS 基础了。

