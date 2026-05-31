# 知识梳理 - HTML与CSS

## 简介

本文档是对 HTML 与 CSS 基础课程的完整知识体系整理，融合了 15 份课堂笔记内容。原始文件清单如下：

- Day01_HTM课堂笔记.md
- Day02_HTML课堂笔记.md
- Day03_HTML课堂笔记.md
- Day04_HTML&CSS课堂笔记.md
- Day05_CSS课堂笔记.md
- Day06_CSS课堂笔记.md
- Day07_CSS课堂笔记.md
- Day08_CSS课堂笔记.md
- Day09_页面布局.md
- Day11_HTML5&CSS3课堂笔记.md
- Day12_CSS3课堂笔记.md
- Day13_CSS3课堂笔记.md
- Day14_CSS3课堂笔记.md
- Day15_CSS3课堂笔记.md
- Day16_CSS3课堂笔记.md

---

## 第一章 HTML 基础

### 1.1 HTML 基本概念与语法

#### HTML 文档声明与页面结构

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>页面标题</title>
    </head>
    <body>
    </body>
</html>
```

**关键要点：**
- `<!DOCTYPE html>` 告诉浏览器采用标准模式渲染，避免进入怪异模式
- `<meta charset="UTF-8">` 设置字符集编码（推荐使用 UTF-8）
- 标签分为双标签（`<tag>内容</tag>`）和单标签（自闭合标签）
- 标签具有语义性，应根据内容选择合适的标签

#### HTML 标签分类

**主体结构标签：**
- `<html>` 根元素
- `<head>` 头部，包含元信息
- `<body>` 主体内容

**排版标签：**
- `<h1>~<h6>` 标题（块级）
- `<p>` 段落（块级）
- `<hr>` 分隔线（单标签，块级）
- `<br>` 换行（单标签，行内）
- `<pre>` 原格式显示（块级）
- `<div>` 无语义布局容器（块级）

**文本标签：**
- `<em>` 强调，默认斜体（行内，有语义）
- `<strong>` 强调，默认粗体（行内，有语义）
- `<ins>` 插入文本，下划线（行内）
- `<del>` 删除文本，删除线（行内）
- `<sub>` 下标字（行内）
- `<sup>` 上标字（行内）
- `<span>` 无语义文字容器（行内）

**全局属性：**
- `id` 唯一标识，用作锚点或 CSS 选择器
- `class` 类名，用于 CSS 选择器（可多个）
- `style` 行内样式
- `title` 鼠标悬浮提示文字
- `name` 表单控件标识
- `lang` 语言设置（如 `zh-CN`）
- `hidden` HTML5 属性，隐藏元素

### 1.2 列表

**无序列表：**
```html
<ul>
    <li>列表项</li>
    <li>列表项</li>
</ul>
```

**有序列表：**
```html
<ol>
    <li>列表项</li>
    <li>列表项</li>
</ol>
```

**定义列表：**
```html
<dl>
    <dt>术语</dt>
    <dd>定义</dd>
</dl>
```

### 1.3 表格

#### 表格结构

```html
<table border="1">
    <caption>表格标题</caption>
    <thead>
        <tr>
            <th>表头单元格</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>数据单元格</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td>表脚</td>
        </tr>
    </tfoot>
</table>
```

#### 表格相关属性

| 属性 | 应用元素 | 作用 |
|------|--------|------|
| `border` | table | 边框宽度 |
| `cellspacing` | table | 单元格间距 |
| `cellpadding` | table | 单元格内补白 |
| `rowspan` | td/th | 跨行数 |
| `colspan` | td/th | 跨列数 |
| `align` | thead/tbody/tfoot/tr/td | 水平对齐（left/center/right） |
| `valign` | thead/tbody/tfoot/tr/td | 垂直对齐（top/middle/bottom） |

### 1.4 图片与路径

#### img 标签

```html
<img src="图片地址" alt="替代文字" width="宽度" height="高度">
```

**属性说明：**
- `src` 图片地址（必需）
- `alt` 替代文字（图片加载失败时显示）
- `width/height` 只设置其一则自动等比缩放；同时设置易造成变形

#### 路径类型

**绝对路径：**
- 网络绝对路径：`https://example.com/image.jpg`
- 本地绝对路径：`D:/folder/image.jpg`（后端开发使用）

**相对路径：**
- `./` 当前目录（可省略）
- `../` 上级目录
- `../../` 上上级目录

### 1.5 超链接与锚点

#### a 标签

```html
<a href="目标地址" target="_self">链接文字</a>
```

**属性：**
- `href` 目标地址
- `target` 打开方式（`_self` 本窗口，`_blank` 新窗口）
- `download` HTML5 属性，下载文件

**特殊链接：**
```html
<a href="tel:10086">打电话</a>
<a href="sms:10010">发短信</a>
<a href="mailto:email@example.com">发邮件</a>
```

#### 锚点

```html
<!-- 设置锚点 -->
<div id="section1"></div>

<!-- 跳转锚点 -->
<a href="#section1">跳转到部分1</a>
<a href="page2.html#section2">跳转到其他页面的锚点</a>
```

### 1.6 表单

#### 表单基本结构

```html
<form action="提交地址" target="_blank" method="POST">
    <!-- 表单控件 -->
</form>
```

#### 表单控件

| 控件类型 | 代码 | 说明 |
|---------|------|------|
| 文本框 | `<input type="text">` | 单行文本 |
| 密码框 | `<input type="password">` | 密码输入 |
| 单选框 | `<input type="radio" name="group">` | 同name为一组 |
| 复选框 | `<input type="checkbox">` | 多选 |
| 提交按钮 | `<input type="submit">` 或 `<button type="submit">` | - |
| 重置按钮 | `<input type="reset">` 或 `<button type="reset">` | - |
| 普通按钮 | `<input type="button">` 或 `<button type="button">` | - |
| 文本域 | `<textarea rows="10" cols="60"></textarea>` | 多行文本 |
| 下拉选项 | `<select><option>选项</option></select>` | 单选列表 |

#### 表单控件属性

| 属性 | 作用 |
|------|------|
| `name` | 表单控件标识，用于后端获取数据 |
| `value` | 表单控件的值 |
| `disabled` | 设置不可用 |
| `checked` | 单选框/复选框默认选中 |
| `selected` | 下拉选项默认选中 |
| `maxlength` | 最大输入长度 |
| `placeholder` | HTML5，输入提示文字 |
| `required` | HTML5，必填项 |
| `autofocus` | HTML5，自动获取焦点 |
| `pattern` | HTML5，正则表达式验证 |
| `autocomplete` | HTML5，是否显示输入记录 |

#### label 标签

```html
<!-- 方式1：for 属性关联 -->
<label for="username">用户名：</label>
<input type="text" id="username">

<!-- 方式2：包裹方式 -->
<label>
    <input type="checkbox">
    同意条款
</label>
```

### 1.7 音视频

```html
<!-- 视频 -->
<video src="video.mp4" width="400" height="300" controls></video>

<!-- 音频 -->
<audio src="audio.mp3" controls></audio>

<!-- 支持多格式 -->
<video width="400" height="300" controls>
    <source src="video.mp4" type="video/mp4">
    <source src="video.webm" type="video/webm">
</video>
```

**常见属性：**
- `controls` 显示控制条
- `autoplay` 自动播放
- `muted` 默认静音
- `loop` 循环播放
- `preload` 预加载
- `poster` 视频封面（仅视频）

### 1.8 内联框架

```html
<iframe src="页面地址" width="400" height="300" frameborder="0"></iframe>
```

### 1.9 字符实体

| 实体 | 含义 |
|------|------|
| `&nbsp;` | 空格 |
| `&lt;` | < |
| `&gt;` | > |
| `&amp;` | & |
| `&yen;` | ¥ |
| `&copy;` | © |

---

## 第二章 HTML5 新增特性

### 2.1 HTML5 新增语义化标签

| 标签 | 语义 | 备注 |
|------|------|------|
| `<header>` | 页头 | - |
| `<footer>` | 页脚 | - |
| `<nav>` | 导航 | - |
| `<section>` | 页面区域或文章分节 | - |
| `<aside>` | 侧边栏 | - |
| `<article>` | 文章/博客/帖子等 | - |
| `<main>` | 主要内容部分 | IE 不支持 |
| `<figure>` | 独立流内容 | - |
| `<figcaption>` | 图片标题 | - |

**状态标签（了解）：**
- `<meter>` 静态度量（如电量、磁盘用量）
- `<progress>` 动态进度条

### 2.2 HTML5 表单新增功能

#### 新增 input 类型

**输入框类（5个）：**
- `type="email"` 邮箱验证
- `type="number"` 数字输入
- `type="url"` URL 验证
- `type="tel"` 电话（手机端弹键盘）
- `type="search"` 搜索框

**范围选择：**
- `type="range"` 滑块选择

**颜色选择：**
- `type="color"` 颜色选择器

**日期时间类（5个）：**
- `type="date"` 日期
- `type="month"` 年月
- `type="week"` 周
- `type="time"` 时间
- `type="datetime-local"` 日期时间

#### 新增表单属性

| 属性 | 作用 |
|------|------|
| `placeholder` | 输入提示文字 |
| `autofocus` | 自动获取焦点 |
| `required` | 必填项 |
| `pattern` | 正则表达式验证 |
| `autocomplete` | 是否显示输入记录 |
| `form` | 关联到指定表单 |
| `novalidate` | form 标签，禁用验证 |

#### 搜索建议

```html
<input type="text" list="searchData">
<datalist id="searchData">
    <option value="选项1"></option>
    <option value="选项2"></option>
</datalist>
```

### 2.3 HTML5 新增全局属性

- `hidden` 隐藏元素（不占据空间）
- a 标签的 `download` 属性下载文件

### 2.4 meta 元信息

```html
<!-- 字符集 -->
<meta charset="UTF-8">

<!-- 网页关键词 -->
<meta name="keywords" content="关键词1,关键词2">

<!-- 网页描述 -->
<meta name="description" content="网页描述信息">

<!-- 移动端完美视口 -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- 网页自动刷新 -->
<meta http-equiv="refresh" content="3">

<!-- 定时跳转 -->
<meta http-equiv="refresh" content="10,url=http://example.com">
```

---

## 第三章 CSS 基础

### 3.1 CSS 引入方式

#### 行内样式
```html
<p style="color: red; font-size: 16px;">文本</p>
```

#### 内嵌样式
```html
<head>
    <style>
        p { color: red; }
    </style>
</head>
```

#### 外链样式
```html
<link rel="stylesheet" href="style.css">
```

**建议：** 优先使用外链式，便于维护和复用。

### 3.2 CSS 语法基础

```css
选择器 {
    属性名: 属性值;
    属性名: 属性值;
}
```

### 3.3 CSS 选择器

#### 基本选择器（4个）

```css
/* 标签名选择器 */
p { }

/* 类名选择器 */
.classname { }

/* ID 选择器 */
#idname { }

/* 全局（通配）选择器 */
* { }
```

#### 组合选择器

```css
/* 后代元素选择器（空格） */
div p { }

/* 子元素选择器 */
div > p { }

/* 交集选择器（无空格） */
p.active { }
div.container { }

/* 并集选择器（逗号） */
h1, h2, h3 { }
```

#### CSS3 新增选择器

**相邻兄弟、通用兄弟：**
```css
/* 相邻兄弟 */
p + span { }

/* 通用兄弟 */
p ~ span { }
```

**属性选择器：**
```css
[attr]                  /* 包含属性 */
[attr="value"]          /* 属性值等于 */
[attr^="value"]         /* 属性值以...开头 */
[attr$="value"]         /* 属性值以...结尾 */
[attr*="value"]         /* 属性值包含 */
```

**结构伪类：**
```css
:first-child            /* 首个子元素 */
:last-child             /* 末个子元素 */
:nth-child(n)           /* 第 n 个子元素 */
:nth-last-child(n)      /* 倒数第 n 个 */
:only-child             /* 唯一子元素 */

:first-of-type          /* 首个同类元素 */
:last-of-type           /* 末个同类元素 */
:nth-of-type(n)         /* 第 n 个同类元素 */

:root                   /* 根元素 */
:empty                  /* 空元素 */
```

**动态伪类：**
```css
:link                   /* 未访问链接 */
:visited                /* 已访问链接 */
:hover                  /* 鼠标悬浮 */
:active                 /* 鼠标按下 */
:focus                  /* 获取焦点 */
```

**其他伪类：**
```css
:checked                /* 被选中的表单控件 */
:disabled               /* 禁用的表单控件 */
:enabled                /* 启用的表单控件 */
:not(selector)          /* 否定伪类 */
:target                 /* 锚点对应元素 */
```

**伪元素：**
```css
::before                /* 前插入元素（需设置 content） */
::after                 /* 后插入元素（需设置 content） */
::first-letter          /* 首字母 */
::first-line            /* 首行 */
::selection             /* 选中文字 */
::placeholder           /* 占位符文字 */
```

#### 选择器优先级（权重）

```
!important              > 1, 0, 0, 0, 0
行内样式                > 1, 0, 0, 0
ID 选择器               > 0, 1, 0, 0
类、伪类、属性选择器     > 0, 0, 1, 0
标签名、伪元素选择器     > 0, 0, 0, 1
全局选择器              > 0, 0, 0, 0
```

**计算规则：**
1. 比较 ID 选择器个数
2. ID 个数相同，比较类选择器个数
3. 类个数相同，比较标签选择器个数
4. 权重相同，后写的覆盖前面的

### 3.4 CSS 长度单位

| 单位 | 说明 |
|------|------|
| `px` | 像素 |
| `em` | 相对于元素字体大小的倍数 |
| `%` | 百分比（参照父元素对应属性） |
| `rem` | 相对于根元素字体大小的倍数 |
| `vw` | 视口宽度的百分比（1vw = 1%视口宽度） |
| `vh` | 视口高度的百分比 |

### 3.5 CSS 颜色设置

#### 颜色名
```css
color: red;
color: deeppink;
color: skyblue;
```

#### RGB 颜色
```css
color: rgb(255, 0, 0);
color: rgb(100%, 0%, 0%);
```

#### 十六进制颜色
```css
color: #FF0000;
color: #f00;  /* 相同数字可简写 */
```

#### RGBA 颜色（带透明度）
```css
background: rgba(255, 0, 0, 0.5);  /* 透明度 0~1 */
```

#### HSL/HSLA 颜色
```css
background: hsl(0, 100%, 50%);
background: hsla(0, 100%, 50%, 0.5);
/* H: 0~360, S: 0%~100%, L: 0%~100% */
```

### 3.6 元素显示模式

#### 块级元素（block）
- 独占一行
- 可设置宽高
- 默认宽度 100%
- 常见：`<div>`, `<p>`, `<h1>~<h6>`, `<ul>`, `<ol>`, `<li>` 等

#### 行内元素（inline）
- 在行内显示
- 无法设置宽高
- 宽高被内容撑开
- 常见：`<span>`, `<a>`, `<em>`, `<strong>` 等

#### 行内块元素（inline-block）
- 在行内显示，但可设置宽高
- 常见：`<img>`, `<input>`, `<button>` 等

#### 修改显示模式
```css
display: block;
display: inline;
display: inline-block;
```

### 3.7 文字相关属性

#### 字体样式

| 属性 | 值 |
|------|-----|
| `font-size` | 字体大小（px、em、%） |
| `font-weight` | 字体粗细（normal、bold、100~900） |
| `font-style` | 斜体（normal、italic） |
| `font-family` | 字体族科 |

```css
font-family: "Microsoft YaHei", 微软雅黑, 黑体, sans-serif;
font: bold italic 16px "Microsoft YaHei", sans-serif;  /* 复合属性 */
```

#### 文本样式

| 属性 | 说明 |
|------|------|
| `color` | 文字颜色 |
| `text-align` | 水平对齐（left、center、right、justify） |
| `text-indent` | 首行缩进 |
| `text-decoration` | 文本修饰（none、underline、overline、line-through） |
| `letter-spacing` | 字间距 |
| `word-spacing` | 词间距 |
| `line-height` | 行高（影响垂直居中） |
| `vertical-align` | 行内元素垂直对齐（baseline、top、middle、bottom） |
| `text-transform` | 文字转换（uppercase、lowercase、capitalize） |

#### 行高与垂直居中

```css
/* 单行文字垂直居中 */
.container {
    height: 100px;
    line-height: 100px;  /* 与高度相同 */
}
```

**行高概念：**
- 上一行文字中线与下一行文字中线的距离
- 第一行文字中线距离元素顶部 = 行高 / 2
- 最后一行文字中线距离元素底部 = 行高 / 2

### 3.8 背景属性

| 属性 | 说明 |
|------|------|
| `background-color` | 背景颜色 |
| `background-image` | 背景图像 |
| `background-repeat` | 重复方式（repeat、repeat-x、repeat-y、no-repeat） |
| `background-position` | 背景位置 |
| `background-attachment` | 固定方式（scroll、fixed） |
| `background-size` | 背景尺寸（contain、cover、具体尺寸） |
| `background-origin` | 定位原点（padding-box、border-box、content-box） |
| `background-clip` | 裁剪区域（border-box、padding-box、content-box、text） |

```css
/* 背景位置 */
background-position: left top;           /* 关键字 */
background-position: 50% 50%;            /* 百分比 */
background-position: 100px 100px;        /* 坐标 */

/* 背景大小 */
background-size: 200px 100px;
background-size: 50%;
background-size: contain;                /* 完整显示 */
background-size: cover;                  /* 铺满容器 */

/* 复合属性 */
background: url(bg.jpg) no-repeat center/cover;
```

---

## 第四章 盒子模型

### 4.1 盒子模型组成

```
┌─────────────────────────────────────┐
│          margin（外边距）            │
│  ┌──────────────────────────────┐   │
│  │    border（边框）             │   │
│  │  ┌────────────────────────┐  │   │
│  │  │  padding（内边距）     │  │   │
│  │  │  ┌────────────────────┐  │   │
│  │  │  │ content（内容）    │  │   │
│  │  │  └────────────────────┘  │   │
│  │  └────────────────────────┘  │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

### 4.2 内容区域

```css
width: 400px;           /* 内容宽度 */
height: 300px;          /* 内容高度 */
max-width: 1200px;      /* 最大宽度 */
min-width: 320px;       /* 最小宽度 */
max-height: 500px;
min-height: 100px;
```

### 4.3 内边距（padding）

```css
padding: 20px;              /* 四边 */
padding: 10px 20px;         /* 上下 左右 */
padding: 10px 20px 30px;    /* 上 左右 下 */
padding: 10px 20px 30px 40px;  /* 上 右 下 左 */

padding-top: 10px;
padding-right: 20px;
padding-bottom: 30px;
padding-left: 40px;
```

**性质：**
- 不能为负值
- 百分比参照父元素内容宽度
- 行内元素左右 padding 正常，上下 padding 不完美

### 4.4 边框（border）

```css
border-width: 2px;
border-style: solid;          /* solid、dashed、dotted、double */
border-color: #333;

border: 2px solid #333;       /* 复合属性 */

border-left: 2px solid red;
border-right: 3px dashed blue;
border-top: 1px dotted green;
border-bottom: 4px solid orange;
```

### 4.5 外边距（margin）

```css
margin: 20px;
margin: 10px 20px;
margin: 10px 20px 30px;
margin: 10px 20px 30px 40px;

margin-top: 10px;
margin-right: 20px;
margin-bottom: 30px;
margin-left: 40px;

/* 水平居中 */
margin: 0 auto;  /* 块级元素 */
```

**性质：**
- 可以为负值
- 百分比参照父元素内容宽度
- 行内元素只能设置左右 margin，上下 margin 无效

#### margin 塌陷问题

```
现象：子元素的上/下 margin 会塌陷到父元素外面
解决方案：
1. 父元素设置 border
2. 父元素设置 padding
3. 父元素设置 overflow: hidden（开启BFC）
```

#### margin 合并问题

```
现象：相邻块级元素的上下 margin 会合并，取较大值
解决方案：不需要解决（正常现象）
```

### 4.6 盒子模型公式

```
盒子总宽度 = content-width + padding-left + padding-right + border-left + border-right
盒子总高度 = content-height + padding-top + padding-bottom + border-top + border-bottom

块级元素默认总宽度 = 父元素内容宽度 - margin-left - margin-right
```

### 4.7 box-sizing 属性

```css
box-sizing: content-box;      /* 默认值，width 指 content 宽度 */
box-sizing: border-box;       /* width 指 content + padding + border 的总宽度 */
```

### 4.8 溢出处理

```css
overflow: visible;            /* 默认值，内容溢出 */
overflow: hidden;             /* 溢出隐藏 */
overflow: scroll;             /* 显示滚动条 */
overflow: auto;               /* 自动，内容溢出才显示滚动条 */

overflow-x: auto;             /* X 轴方向 */
overflow-y: auto;             /* Y 轴方向 */
```

### 4.9 隐藏元素

```css
display: none;                /* 彻底隐藏，不占据空间 */
visibility: hidden;           /* 隐藏但占据空间 */
```

---

## 第五章 CSS 布局

### ASCII 流程图：HTML 渲染流程

```
┌──────────────────────────────────────────┐
│     HTML 文本                            │
└───────────────┬──────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────┐
│  1. HTML 解析器解析                      │
│     构建 DOM 树                          │
└───────────────┬──────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────┐
│  2. CSS 解析器解析                       │
│     构建 CSSOM 树                        │
└───────────────┬──────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────┐
│  3. 合并 DOM + CSSOM                    │
│     生成渲染树（仅可见元素）            │
│     剔除 display:none 元素               │
└───────────────┬──────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────┐
│  4. 布局（Layout）                      │
│     计算元素的大小和位置                 │
│     考虑盒子模型、定位、浮动等          │
└───────────────┬──────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────┐
│  5. 绘制（Paint）                       │
│     绘制元素的视觉效果                   │
│     文字、背景、边框、阴影等            │
└───────────────┬──────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────┐
│  6. 合成（Composite）                   │
│     多图层合成最终画面                   │
└──────────────────────────────────────────┘
```

### 5.1 浮动布局

#### 浮动属性

```css
float: left;                  /* 左浮动 */
float: right;                 /* 右浮动 */
float: none;                  /* 不浮动 */

clear: left;                  /* 清除左浮动影响 */
clear: right;                 /* 清除右浮动影响 */
clear: both;                  /* 清除所有浮动影响 */
```

#### 浮动元素特点

```
1. 脱离文档流
2. 多个浮动元素水平排列，自动换行
3. 不存在 margin 塌陷和合并
4. 设置左右 margin:auto 无法居中
```

#### 浮动问题与解决

**问题1：浮动元素覆盖相邻元素**
```css
/* 解决方案：给后面的兄弟元素设置 clear:both */
.next-sibling {
    clear: both;
}
```

**问题2：浮动元素造成父元素高度塌陷**
```css
/* 方案1：父元素固定高度 */
.parent {
    height: 300px;
}

/* 方案2：父元素浮动 */
.parent {
    float: left;
}

/* 方案3：父元素 overflow:hidden（推荐） */
.parent {
    overflow: hidden;  /* 开启 BFC */
}

/* 方案4：伪元素清浮动（推荐） */
.parent::after {
    content: "";
    display: block;
    clear: both;
}
```

### 5.2 定位布局

#### 定位类型

```css
position: static;             /* 默认，无定位 */
position: relative;           /* 相对定位 */
position: absolute;           /* 绝对定位 */
position: fixed;              /* 固定定位 */
position: sticky;             /* 粘连定位（CSS3） */
```

#### 相对定位

```css
position: relative;
top: 10px;
left: 20px;
```

**特点：**
- 参照自己原来的位置定位
- 不脱离文档流
- 相邻元素按原位置排列

#### 绝对定位

```css
position: absolute;
top: 50px;
left: 100px;
```

**特点：**
- 脱离文档流
- 参照**包含块**定位
- 包含块：第一个定位的祖先元素（或整个页面）
- 不存在 margin 塌陷

#### 固定定位

```css
position: fixed;
top: 0;
right: 0;
```

**特点：**
- 脱离文档流
- 参照视口定位
- 随页面滚动而固定

#### 定位层级

```css
z-index: 10;              /* 数字越大越靠前，默认值 auto = 0 */
z-index: -1;              /* 可以为负值 */
```

**规则：**
- 定位元素默认显示在不定位元素上方
- 定位元素之间，z-index 大的在上方
- 不定位元素的 z-index 无效

#### ASCII 流程图：定位层级比较

```
┌─────────────────────────────────────────┐
│   元素1（z-index: 10）                  │
│   ┌───────────────────────────────────┐ │
│   │  定位元素，显示在上方              │ │
│   └───────────────────────────────────┘ │
└─────────────────────────────────────────┘
             ▲
             │ z-index 大
             │
┌─────────────────────────────────────────┐
│   元素2（z-index: 5）                   │
│   ┌───────────────────────────────────┐ │
│   │  定位元素，显示在中间              │ │
│   └───────────────────────────────────┘ │
└─────────────────────────────────────────┘
             ▲
             │ z-index 大
             │
┌─────────────────────────────────────────┐
│   元素3（position: static）             │
│   ┌───────────────────────────────────┐ │
│   │  不定位元素，显示在最下方          │ │
│   └───────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

#### 定位元素居中方法

**方法1：margin 负值法**
```css
position: absolute;
left: 50%;
top: 50%;
margin-left: -自身总宽度/2;  /* 需提前知道宽度 */
margin-top: -自身总高度/2;
```

**方法2：margin:auto 法（推荐）**
```css
position: absolute;
left: 0;
right: 0;
top: 0;
bottom: 0;
margin: auto;
```

### 5.3 行内块布局

**注意事项：**

```css
/* 问题1：元素间空白 */
/* 原因：HTML 代码中的换行产生空格 */
/* 解决：父元素 font-size:0，子元素单独设置字体大小 */
.parent {
    font-size: 0;
}
.child {
    font-size: 16px;
}

/* 问题2：图片底部空白（幽灵空白） */
/* 原因：行内块元素与基线对齐 */
/* 解决：vertical-align:bottom 或将 img 设为 block */
img {
    vertical-align: bottom;
    /* 或 */
    display: block;
}
```

---

## 第六章 CSS 选择器优先级深度解析

### ASCII 流程图：CSS 层叠优先级计算

```
┌─────────────────────────────────────────────────┐
│        CSS 属性优先级匹配流程                    │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│  第1层：!important 规则                         │
│  ┌────────────────────────────────────────────┐ │
│  │ color: red !important;  (优先级: 无限)    │ │
│  └────────────────────────────────────────────┘ │
└────────────────────┬────────────────────────────┘
                     │ (如果无 !important)
                     ▼
┌─────────────────────────────────────────────────┐
│  第2层：行内样式                               │
│  ┌────────────────────────────────────────────┐ │
│  │ style="color: red;"  (优先级: 1,0,0,0)   │ │
│  └────────────────────────────────────────────┘ │
└────────────────────┬────────────────────────────┘
                     │ (如果无行内样式)
                     ▼
┌─────────────────────────────────────────────────┐
│  第3层：ID 选择器                              │
│  ┌────────────────────────────────────────────┐ │
│  │ #myid { }  (优先级: 0,1,0,0)            │ │
│  └────────────────────────────────────────────┘ │
└────────────────────┬────────────────────────────┘
                     │ (如果 ID 个数相同)
                     ▼
┌─────────────────────────────────────────────────┐
│  第4层：类、伪类、属性选择器                    │
│  ┌────────────────────────────────────────────┐ │
│  │ .myclass {}  (优先级: 0,0,1,0)           │ │
│  │ :hover {}    (优先级: 0,0,1,0)           │ │
│  │ [type="text"] {}  (优先级: 0,0,1,0)      │ │
│  └────────────────────────────────────────────┘ │
└────────────────────┬────────────────────────────┘
                     │ (如果类个数相同)
                     ▼
┌─────────────────────────────────────────────────┐
│  第5层：标签名、伪元素选择器                    │
│  ┌────────────────────────────────────────────┐ │
│  │ p {}       (优先级: 0,0,0,1)             │ │
│  │ ::before {} (优先级: 0,0,0,1)            │ │
│  └────────────────────────────────────────────┘ │
└────────────────────┬────────────────────────────┘
                     │ (如果标签个数相同)
                     ▼
┌─────────────────────────────────────────────────┐
│  第6层：全局选择器                             │
│  ┌────────────────────────────────────────────┐ │
│  │ * {}  (优先级: 0,0,0,0)                  │ │
│  └────────────────────────────────────────────┘ │
└────────────────────┬────────────────────────────┘
                     │ (权重相同)
                     ▼
┌─────────────────────────────────────────────────┐
│  第7层：继承样式 / 浏览器默认样式             │
│  ┌────────────────────────────────────────────┐ │
│  │ (继承的样式最低)                          │ │
│  └────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

---

## 第七章 Flexbox 布局（伸缩盒）

### ASCII 流程图：Flex 容器与项目关系

```
Flex 容器 (display: flex)
┌─────────────────────────────────────────┐
│                                         │
│  ┌──────────┬──────────┬──────────┐    │
│  │ 项目1    │ 项目2    │ 项目3    │    │
│  │          │          │          │    │
│  │ flex-grow:1     flex-grow:2     │    │
│  └──────────┴──────────┴──────────┘    │
│                                         │
│  ◄───────  justify-content  ────────►  │
│                                         │
│  ▲                                      │
│  │ align-items                          │
│  ▼                                      │
│                                         │
└─────────────────────────────────────────┘
```

### 7.1 Flex 容器属性

```css
display: flex;              /* 创建 Flex 容器 */
display: inline-flex;       /* 行内 Flex 容器 */

/* 主轴方向 */
flex-direction: row;              /* 默认，水平从左到右 */
flex-direction: row-reverse;      /* 水平从右到左 */
flex-direction: column;           /* 竖直从上到下 */
flex-direction: column-reverse;   /* 竖直从下到上 */

/* 换行方式 */
flex-wrap: nowrap;                /* 默认，不换行 */
flex-wrap: wrap;                  /* 自动换行 */
flex-wrap: wrap-reverse;          /* 换行并翻转 */

/* 复合属性 */
flex-flow: row wrap;              /* flex-direction + flex-wrap */

/* 主轴对齐（单行） */
justify-content: flex-start;      /* 起点对齐 */
justify-content: flex-end;        /* 终点对齐 */
justify-content: center;          /* 居中对齐 */
justify-content: space-between;   /* 两端对齐 */
justify-content: space-around;    /* 两端是中间间距一半 */
justify-content: space-evenly;    /* 间距均匀分布 */

/* 侧轴对齐（单行） */
align-items: stretch;             /* 默认，拉伸 */
align-items: flex-start;          /* 起点对齐 */
align-items: flex-end;            /* 终点对齐 */
align-items: center;              /* 居中对齐 */
align-items: baseline;            /* 基线对齐 */

/* 侧轴对齐（多行） */
align-content: stretch;           /* 默认，拉伸 */
align-content: flex-start;        /* 起点对齐 */
align-content: flex-end;          /* 终点对齐 */
align-content: center;            /* 居中对齐 */
align-content: space-between;     /* 两端对齐 */
align-content: space-around;      /* 两端是中间间距一半 */
align-content: space-evenly;      /* 间距均匀分布 */
```

### 7.2 Flex 项目属性

```css
/* 项目在主轴上的长度 */
flex-basis: auto;                 /* 默认值，按 width/height */
flex-basis: 100px;

/* 扩展比率（富余空间分配） */
flex-grow: 0;                     /* 默认值，不扩展 */
flex-grow: 1;                     /* 等分富余空间 */
flex-grow: 2;                     /* 占富余空间 2/n */

/* 收缩比率（不足空间分配） */
flex-shrink: 1;                   /* 默认值，等分不足空间 */
flex-shrink: 0;                   /* 不收缩 */

/* 复合属性：flex: grow shrink basis */
flex: 1;                          /* 计算值：1 1 0% */
flex: auto;                       /* 计算值：1 1 auto */
flex: none;                       /* 计算值：0 0 auto */
flex: 0 1 auto;                   /* 初始值 */

/* 项目排序 */
order: 0;                         /* 默认值 */
order: 1;                         /* 值越大排序越靠后 */
order: -1;                        /* 可为负值 */

/* 单独设置侧轴对齐 */
align-self: auto;                 /* 默认值，按容器 align-items */
align-self: flex-start;
align-self: flex-end;
align-self: center;
align-self: stretch;
align-self: baseline;
```

### ASCII 流程图：Flex 项目扩展计算

```
Flex 容器宽度: 1000px
项目基础宽度总和: 600px (100 + 200 + 300)
富余空间: 400px (1000 - 600)

┌────────────────────────────────────────────┐
│  项目扩展分配规则                         │
├────────────────────────────────────────────┤
│                                            │
│  项目1                                     │
│  └─ 基础宽度: 100px                       │
│  └─ flex-grow: 1                          │
│  └─ 扩展比重: 1/(1+6+3) = 1/10            │
│  └─ 分配: 100 + 400×(1/10) = 140px       │
│                                            │
│  项目2                                     │
│  └─ 基础宽度: 200px                       │
│  └─ flex-grow: 6                          │
│  └─ 扩展比重: 6/(1+6+3) = 6/10            │
│  └─ 分配: 200 + 400×(6/10) = 440px       │
│                                            │
│  项目3                                     │
│  └─ 基础宽度: 300px                       │
│  └─ flex-grow: 3                          │
│  └─ 扩展比重: 3/(1+6+3) = 3/10            │
│  └─ 分配: 300 + 400×(3/10) = 420px       │
│                                            │
│  验证: 140 + 440 + 420 = 1000px ✓        │
└────────────────────────────────────────────┘
```

---

## 第八章 Grid 布局

Grid 是 CSS3 提供的网格布局系统，相比 Flex 更强大，可以同时控制行列。

```css
display: grid;                    /* 创建网格容器 */

/* 定义列 */
grid-template-columns: 100px 200px 100px;  /* 固定列宽 */
grid-template-columns: repeat(3, 1fr);     /* 3 列等分 */
grid-template-columns: 1fr 2fr 1fr;        /* 比例分配 */
grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));

/* 定义行 */
grid-template-rows: 100px 200px 100px;

/* 行列间距 */
gap: 10px;                        /* 行列间距都是 10px */
gap: 10px 20px;                   /* 行间距 10px，列间距 20px */
row-gap: 10px;
column-gap: 20px;

/* 项目对齐 */
justify-items: start;             /* 项目在单元格内水平对齐 */
align-items: start;               /* 项目在单元格内垂直对齐 */
justify-content: center;          /* 网格在容器内水平对齐 */
align-content: center;            /* 网格在容器内垂直对齐 */
```

---

## 第九章 CSS3 新增样式

### 9.1 变换（Transform）

#### ASCII 流程图：2D/3D 变换坐标系

```
2D 坐标系：               3D 坐标系：
       ▲ Y                    ▲ Y
       │                      │  ╱ Z
       │                      │╱
  ─────┼───► X          ─────┼───► X
       │
       │
      
变换原点（transform-origin）：
默认值为元素中心 (50%, 50%)
```

#### Transform 属性

**2D 变换：**
```css
transform: translateX(100px);         /* X 轴位移 */
transform: translateY(100px);         /* Y 轴位移 */
transform: translate(100px, 100px);   /* XY 轴位移 */

transform: scaleX(1.5);               /* X 轴缩放 */
transform: scaleY(0.8);               /* Y 轴缩放 */
transform: scale(1.5, 0.8);           /* XY 轴缩放 */

transform: rotate(45deg);             /* 旋转角度 */
```

**3D 变换：**
```css
transform: translateZ(100px);         /* Z 轴位移 */
transform: translate3d(0, 0, 100px);  /* 3D 位移 */

transform: rotateX(45deg);            /* 绕 X 轴旋转 */
transform: rotateY(45deg);            /* 绕 Y 轴旋转 */
transform: rotateZ(45deg);            /* 绕 Z 轴旋转 */
```

**其他属性：**
```css
transform-origin: center;            /* 变换原点 */
transform-style: preserve-3d;        /* 子元素保持 3D 空间 */
perspective: 1000px;                 /* 3D 透视距离 */
backface-visibility: hidden;         /* 隐藏背面 */
```

### 9.2 过渡（Transition）

```css
transition-property: all;            /* 过渡属性（all、color 等） */
transition-duration: 0.5s;           /* 过渡时间 */
transition-delay: 0.1s;              /* 过渡延迟 */
transition-timing-function: ease;    /* 过渡曲线 */

/* 复合属性 */
transition: all 0.5s ease 0.1s;
```

**timing-function 值：**
```css
ease                    /* 默认，先快后慢 */
linear                  /* 匀速 */
ease-in                 /* 加速 */
ease-out                /* 减速 */
ease-in-out             /* 先加速后减速 */
cubic-bezier(0,0,1,1)   /* 贝塞尔曲线 */
steps(4, end)           /* 分步过渡 */
```

### 9.3 动画（Animation）

#### 关键帧定义

```css
@keyframes myAnimation {
    from { }              /* 起始帧 */
    to { }                /* 结束帧 */
}

@keyframes myAnimation {
    0% { }
    50% { }
    100% { }
}
```

#### 动画属性

```css
animation-name: myAnimation;              /* 关键帧名字 */
animation-duration: 2s;                   /* 动画时间 */
animation-delay: 0.5s;                    /* 延迟时间 */
animation-timing-function: ease;         /* 动画曲线 */
animation-iteration-count: infinite;      /* 循环次数 */
animation-direction: alternate;           /* 方向 */
animation-play-state: running;            /* 运动状态 */
animation-fill-mode: forwards;            /* 完成后保持状态 */

/* 复合属性 */
animation: myAnimation 2s ease 0.5s infinite alternate forwards;
```

**animation-direction 值：**
```css
normal                  /* 默认 */
reverse                 /* 反向 */
alternate               /* 交替（正反反正） */
alternate-reverse       /* 反向交替 */
```

**animation-fill-mode 值：**
```css
none                    /* 默认 */
forwards                /* 保持最后一帧 */
backwards               /* 保持第一帧 */
both                    /* 兼有 forwards 和 backwards */
```

### 9.4 边框圆角（border-radius）

```css
border-radius: 10px;                      /* 四个角 */
border-radius: 10px 20px;                 /* 对角 */
border-radius: 10px 20px 30px;            /* 左上、右上+左下、右下 */
border-radius: 10px 20px 30px 40px;       /* 左上、右上、右下、左下 */

border-top-left-radius: 20px;
border-top-right-radius: 20px;
border-bottom-right-radius: 20px;
border-bottom-left-radius: 20px;

/* 椭圆角（x半径 / y半径） */
border-radius: 50px / 25px;
```

### 9.5 盒子阴影（box-shadow）

```css
box-shadow: 5px 5px 10px rgba(0,0,0,0.3);  /* x偏移 y偏移 模糊 颜色 */
box-shadow: 5px 5px 10px 2px rgba(0,0,0,0.3);  /* 加外延值 */
box-shadow: inset 5px 5px 10px rgba(0,0,0,0.3);  /* 内阴影 */
box-shadow: 5px 0 10px red, -5px 0 10px blue;  /* 多阴影 */
```

### 9.6 文本阴影（text-shadow）

```css
text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
text-shadow: 2px 2px 4px red, 4px 4px 8px blue;
```

### 9.7 渐变

#### 线性渐变

```css
background: linear-gradient(to right, red, blue);
background: linear-gradient(45deg, red 0%, blue 50%, green 100%);
background: linear-gradient(90deg, #fff 0%, #ccc 100%);
```

#### 径向渐变

```css
background: radial-gradient(circle, red, blue);
background: radial-gradient(ellipse, red 0%, blue 100%);
```

#### 重复渐变

```css
background: repeating-linear-gradient(90deg, red 0%, blue 10%, red 20%);
background: repeating-radial-gradient(circle, red 0%, red 10%, blue 20%);
```

### 9.8 滤镜（Filter）

```css
filter: blur(10px);                   /* 模糊 */
filter: brightness(150%);             /* 亮度 */
filter: contrast(120%);               /* 对比度 */
filter: grayscale(100%);              /* 灰度 */
filter: hue-rotate(90deg);            /* 色相旋转 */
filter: invert(100%);                 /* 反色 */
filter: saturate(150%);               /* 饱和度 */
filter: sepia(100%);                  /* 深褐色 */
filter: drop-shadow(5px 5px 10px rgba(0,0,0,0.3));  /* 阴影 */
```

---

## 第十章 CSS3 选择器扩展

### 10.1 属性选择器

```css
[type]                   /* 包含 type 属性 */
[type="text"]            /* type 属性值为 "text" */
[type^="text"]           /* type 属性值以 "text" 开头 */
[type$="text"]           /* type 属性值以 "text" 结尾 */
[type*="text"]           /* type 属性值包含 "text" */

/* 结合其他选择器 */
input[type="text"] { }
a[href^="http"] { }
```

### 10.2 目标伪类

```css
:target                  /* 当前锚点对应的元素 */
```

### 10.3 UI 伪类

```css
:enabled                 /* 启用的表单控件 */
:disabled                /* 禁用的表单控件 */
:checked                 /* 被选中的表单控件 */
```

---

## 第十一章 响应式设计

### 11.1 视口与完美视口

```html
<!-- 移动端完美视口设置 -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

**参数解释：**
- `width=device-width` 视口宽度等于设备宽度
- `initial-scale=1.0` 初始缩放比为 1（不缩放）

### 11.2 媒体查询

```css
/* 媒体类型 */
@media screen { }       /* 屏幕 */
@media print { }        /* 打印 */
@media speech { }       /* 语音 */

/* 媒体特性 */
@media (min-width: 768px) { }     /* 最小宽度 */
@media (max-width: 1024px) { }    /* 最大宽度 */
@media (orientation: portrait) { } /* 竖屏 */

/* 运算符 */
@media (min-width: 768px) and (max-width: 1024px) { }  /* 并且 */
@media (max-width: 640px), (min-width: 1024px) { }     /* 或者 */
@media not print { }                                    /* 否定 */
```

### 11.3 响应式断点建议

```
常见方案：
小屏幕：< 768px
中等屏幕：768px ~ 992px
大屏幕：992px ~ 1200px
超大屏幕：> 1200px
```

### 11.4 响应式图片

```html
<!-- 方案1：picture 标签 -->
<picture>
    <source srcset="small.jpg" media="(max-width: 640px)">
    <source srcset="medium.jpg" media="(max-width: 1024px)">
    <img src="large.jpg" alt="">
</picture>

<!-- 方案2：srcset 属性 -->
<img srcset="small.jpg 640w, medium.jpg 1024w, large.jpg 1440w"
     src="large.jpg" alt="">
```

---

## 第十二章 BFC（块级格式化上下文）

### 12.1 BFC 定义

**Block Formatting Context**，是 Web 页面的可视 CSS 渲染中一部分，可看作一个独立的容器，内部元素布局不影响外部。

### 12.2 创建 BFC 的方式

```css
/* 方式1：根元素 */
html { }

/* 方式2：浮动元素 */
float: left;
float: right;

/* 方式3：绝对/固定定位 */
position: absolute;
position: fixed;

/* 方式4：行内块元素 */
display: inline-block;

/* 方式5：表格单元格 */
display: table-cell;

/* 方式6：overflow 非 visible */
overflow: hidden;
overflow: auto;
overflow: scroll;

/* 方式7：伸缩项目 */
display: flex;

/* 方式8：多列容器 */
column-count: 2;
```

### 12.3 BFC 应用

**解决浮动造成的高度塌陷：**
```css
.parent {
    overflow: hidden;    /* 创建 BFC */
}
```

**解决 margin 塌陷：**
```css
.parent {
    overflow: hidden;    /* 创建 BFC */
}
```

---

## 第十三章 常见错误与易错点

### > ⚠️ 原文有误：第一处 - 默认显示模式混淆

**原文错误位置：** Day04、Day05 讲述块级元素默认宽度

原文描述：
```
默认总宽度 = 父元素内容宽度 - 自身的左右外边距
```

**正确** 应该是：
```
默认总宽度 = 父元素内容宽度 - 自身左右margin - 自身左右padding - 自身左右border
默认内容宽度 = 父元素内容宽度 - 自身左右margin

设置了 width 后：
实际占用宽度 = width + 左右padding + 左右border + 左右margin
```

---

### > ⚠️ 原文有误：第二处 - box-sizing 默认值

**原文错误位置：** Day05、Day12 讲述盒子模型时

原文容易混淆的点：
```
容易误认为 IE 盒子模型是怪异的，W3C 是标准的
```

**正确** 应该是：
```
W3C 标准盒子模型（默认）：
  width = content 宽度
  元素占用宽度 = width + padding + border + margin

IE 盒子模型（怪异模式）：
  width = content + padding + border
  元素占用宽度 = width + margin

现代浏览器用 box-sizing 切换：
  box-sizing: content-box;    (默认，W3C)
  box-sizing: border-box;     (IE 模型)
```

---

### > ⚠️ 原文有误：第三处 - Flex 项目默认值

**原文错误位置：** Day15 伸缩盒布局

原文的模糊点：
```
flex: 1 表示的计算值
```

**正确** 应该是：
```
flex: 1          = flex: 1 1 0%
                  grow:1  shrink:1  basis:0%

flex: auto       = flex: 1 1 auto
flex: none       = flex: 0 0 auto
flex: 0 1 auto   = 初始值

特别注意：flex: 1 的 basis 是 0%，不是 auto
这意味着在分配富余空间时，不考虑项目本身的宽度
```

---

## 第十四章 易错点速查

### 盒子模型

| 易错点 | 正确做法 |
|--------|---------|
| 混淆 content-box 和 border-box | 使用 `box-sizing: border-box` 确保 width 包含 padding 和 border |
| margin 负值的作用不清 | margin 可为负值，常用于重叠和偏移 |
| 行内元素 margin 无效 | 行内元素只有左右 margin，上下无效 |
| 不知道 margin 塌陷解决方案 | 父元素 `overflow:hidden`（创建 BFC）最简单 |

### 浮动

| 易错点 | 正确做法 |
|--------|---------|
| 浮动元素高度塌陷 | 伪元素 `::after { clear: both; }` 或 `overflow:hidden` |
| 浮动元素不能设置宽高 | 浮动元素可以设置宽高 |
| 浮动元素间有空格 | 代码中的换行导致，HTML 写在一行或用 font-size:0 |

### 定位

| 易错点 | 正确做法 |
|--------|---------|
| 绝对定位包含块理解错误 | 包含块是**第一个定位的祖先**，不是父元素 |
| z-index 对不定位元素有效 | z-index 仅对定位元素有效 |
| 多个 z-index 值比较错误 | 比较时考虑层级关系，不只是数字大小 |

### Flex

| 易错点 | 正确做法 |
|--------|---------|
| flex-basis 与 width 冲突 | flex-basis 优先级更高，会覆盖 width |
| flex: 1 的含义 | 等于 `1 1 0%`，0% 很关键 |
| 项目不换行的原因 | 默认 `flex-wrap: nowrap`，需手动设置 wrap |
| align-items vs align-content | items 作用于单行，content 作用于多行 |

### CSS 优先级

| 易错点 | 正确做法 |
|--------|---------|
| ID 选择器权重计算错误 | ID (0,1,0,0) > 类 (0,0,1,0) > 标签 (0,0,0,1) |
| 并集选择器权重合并 | 并集各自计算，不合并：`p, .class { }` |
| !important 滥用 | !important 优先级最高，应避免过度使用 |

### 其他

| 易错点 | 正确做法 |
|--------|---------|
| HTML5 新标签个数 | 主要新增 9 个语义标签（header/footer/nav/section/article/aside/main/figure/figcaption） |
| CSS Reset vs Normalize | Reset 重置所有样式，Normalize 保留有用默认样式 |
| 行内元素基线对齐问题 | 图片底部空白用 `vertical-align: bottom` 或 `display: block` |
| 表单 placeholder 颜色设置 | 使用 `::placeholder` 伪元素设置 |
| 浏览器前缀必要性 | 某些 CSS3 属性在旧浏览器需要前缀（-webkit-, -moz- 等） |

---

## 文档生成日期：2026-05-25

**本文档整理自 15 份课堂笔记，涵盖 HTML、CSS 基础及 HTML5、CSS3 新特性的完整知识体系。**
