# CSS 知识点梳理

这份文档基于 `CSS/day06` 到 `CSS/day21` 的笔记内容整理，目标不是重复原笔记，而是把知识点按“学习路径 + 实战关系”重新归纳，方便复习和查漏补缺。

## 1. 整体知识地图

CSS 这套内容大致可以分成 6 个层次：

1. 基础认知
   CSS 是什么、怎么引入、怎么查文档、怎么调试、颜色怎么表示、浏览器怎么渲染。
2. 文字与外观
   文本属性、字体属性、背景属性、边框、阴影、鼠标样式、图标字体、精灵图。
3. 选择器与规则系统
   选择器、伪类、伪元素、继承、层叠、权重、display、overflow。
4. 盒子模型与元素类型
   width/height、padding、border、margin、box-sizing、块级元素与行内元素。
5. 页面布局
   标准流、定位、浮动、Flex 布局。
6. 工程化与补充
   Emmet、代码规范、组件化思维、meta/link、字符编码、动画与 transform。

---

## 2. CSS 入门基础

### 2.1 CSS 的本质

- CSS 是层叠样式表，用来控制网页的表现。
- HTML 负责结构，CSS 负责样式，二者分离是前端开发的基本原则。
- CSS 不是通用编程语言，更像一门声明式样式语言。

### 2.2 CSS 的三种引入方式

- 行内样式：写在元素的 `style` 属性里。
- 内部样式表：写在页面的 `<style>` 标签里。
- 外部样式表：写在独立 `.css` 文件里，再通过 `<link>` 引入。

结论：

- 实际开发最常用的是外部样式表。
- 行内样式通常只在少量动态样式场景使用。
- `@import` 也能导入 CSS，但工程上一般优先使用 `<link>`。

### 2.3 注释与文档

- CSS 注释写法：`/* 注释内容 */`
- 学 CSS 一定要养成查 MDN 的习惯，很多属性是否继承、默认值、兼容性都能直接查到。

---

## 3. 颜色、进制与浏览器基础

### 3.1 颜色表示方式

- 关键字：`red`、`blue`
- 十六进制：`#ff0000`、`#f00`
- RGB / RGBA：`rgb(255, 0, 0)`、`rgba(255, 0, 0, .5)`

重点理解：

- 十六进制本质上是在用 16 进制表达 RGB 三原色。
- `rgba` 比 `opacity` 更适合只给颜色增加透明度。

### 3.2 link 元素

- `link` 用于引入外部资源。
- 常见 `rel`：
  - `stylesheet`：样式表
  - `icon`：网站图标

### 3.3 浏览器渲染与调试

- 开发工具要会看：
  - Elements：结构和样式
  - Computed：最终生效样式
  - Box Model：盒模型
- 理解浏览器渲染流程有助于理解：
  - 为什么样式覆盖会生效
  - 为什么引入顺序很重要
  - 为什么某些属性修改成本高

---

## 4. 文本与字体系统

### 4.1 常见文本属性

- `text-decoration`
  - 去掉下划线：`none`
  - 下划线：`underline`
  - 删除线：`line-through`
- `text-transform`
  - 首字母大写：`capitalize`
  - 全大写：`uppercase`
  - 全小写：`lowercase`
- `text-indent`
  - 首行缩进常写：`2em`
- `text-align`
  - `left` / `center` / `right` / `justify`
- `letter-spacing` / `word-spacing`
  - 控制字母或单词间距

### 4.2 常见字体属性

- `font-size`
  - 推荐优先使用 `px` / `rem`
- `font-family`
  - 可设置多个字体，浏览器按顺序回退
- `font-weight`
  - `normal = 400`
  - `bold = 700`
- `font-style`
  - `normal` / `italic` / `oblique`
- `line-height`
  - 控制行高
  - 单行文字垂直居中常见技巧：`line-height` 等于容器高度
- `font`
  - 字体简写属性

### 4.3 字体相关易错点

- `height` 是盒子高度，`line-height` 是行盒高度，两者不是一个概念。
- 字体最好设置“字体栈”，不要只写一个字体。
- `em`、百分比会受父元素影响，层级复杂时容易失控。

---

## 5. 选择器体系

### 5.1 基础选择器

- 通配选择器：`*`
- 元素选择器：`div`
- 类选择器：`.box`
- id 选择器：`#header`
- 属性选择器：`[type]`、`[type="text"]`

建议：

- 开发中优先使用类选择器。
- 尽量少依赖 id 做样式控制。

### 5.2 组合选择器

- 后代选择器：`A B`
- 子代选择器：`A > B`
- 相邻兄弟选择器：`A + B`
- 通用兄弟选择器：`A ~ B`
- 交集选择器：`.btn.primary`
- 并集选择器：`h1, h2, h3`

### 5.3 伪类与伪元素

- 动态伪类：
  - `:hover`
  - `:active`
  - `:focus`
- 结构伪类：
  - `:nth-child()`
  - `:nth-of-type()`
  - `:first-child`
  - `:last-child`
  - `:not()`
- 伪元素：
  - `::before`
  - `::after`
  - `::first-line`
  - `::first-letter`

重点区别：

- `:nth-child()` 是按“所有子元素”编号。
- `:nth-of-type()` 是按“同类型元素”编号。

---

## 6. 继承、层叠、权重与 display

### 6.1 继承

- 常见可继承属性：
  - `font-size`
  - `font-family`
  - `font-weight`
  - `line-height`
  - `color`
  - `text-align`
- 不要死记，学会看文档的 `Inherited` 字段。

### 6.2 层叠与优先级

相同属性冲突时，通常按下面顺序决定：

1. `!important`
2. 行内样式
3. 选择器权重
4. 后写的覆盖先写的

常见权重大致可记为：

- `!important`：10000
- 行内样式：1000
- id：100
- 类 / 属性 / 伪类：10
- 元素 / 伪元素：1
- 通配符：0

### 6.3 元素类型与 display

元素大致分为：

- 块级元素：独占一行
- 行内元素：同一行排列
- 行内块元素：既能同排又能设宽高

常见 `display`：

- `block`
- `inline`
- `inline-block`
- `none`
- `flex`
- `inline-flex`

### 6.4 隐藏元素的方式

- `display: none`
  - 直接不占位
- `visibility: hidden`
  - 保留位置但不可见
- `opacity: 0`
  - 视觉透明，但仍可能响应事件

### 6.5 overflow

- `visible`
- `hidden`
- `scroll`
- `auto`

常见用途：

- 裁切超出内容
- 触发滚动区域
- 某些清除浮动场景可作为技巧使用

---

## 7. 盒子模型

### 7.1 盒模型组成

一个盒子由四部分组成：

- content
- padding
- border
- margin

### 7.2 核心属性

- `width` / `height`
- `padding`
- `border`
- `margin`
- `outline`
- `box-shadow`
- `text-shadow`
- `border-radius`

### 7.3 box-sizing

- `content-box`
  - `width/height` 只算内容区
- `border-box`
  - `width/height` 包含 padding 和 border

实战建议：

- 项目里一般更推荐统一使用 `box-sizing: border-box`

### 7.4 margin 的关键点

- 水平方向可用于块级盒子居中：`margin: 0 auto`
- 垂直方向存在 margin 合并问题
- 浮动、定位、BFC 等因素会影响 margin 表现

---

## 8. 背景、边框与视觉效果

### 8.1 背景相关

- `background-image`
- `background-repeat`
- `background-size`
- `background-position`
- `background-attachment`
- `background` 简写

### 8.2 background-image 与 img 的区别

- `img` 是内容，带语义
- 背景图是装饰，不参与文档内容语义

判断标准：

- 对内容理解有意义，用 `img`
- 纯视觉修饰，用 `background-image`

### 8.3 边框与图形

- `border` 除了画边框，还能做三角形等简单图形
- `border-radius` 用于圆角和圆形

### 8.4 Web 字体与图标

- 用 `@font-face` 可引入自定义字体
- 字体格式要考虑兼容性：`ttf`、`otf`、`woff`、`woff2`
- 图标字体适合单色、可缩放图标场景

### 8.5 CSS Sprite

- 把多个小图合成一张大图
- 再通过 `background-position` 精确显示局部
- 主要作用是减少请求数

### 8.6 cursor

- 鼠标样式常见值：
  - `pointer`
  - `default`
  - `text`
  - `move`
  - `not-allowed`

---

## 9. HTML 高级元素补充

虽然这部分不全是 CSS，但和页面样式开发紧密相关。

### 9.1 列表

- 有序列表：`ol > li`
- 无序列表：`ul > li`
- 定义列表：`dl > dt + dd`

### 9.2 表格

- 常见表格标签：
  - `table`
  - `tr`
  - `td`
  - `th`
  - `thead`
  - `tbody`
  - `tfoot`
- 单元格合并：
  - `rowspan`
  - `colspan`

### 9.3 表单

- 常见表单元素：
  - `input`
  - `textarea`
  - `select`
  - `option`
  - `button`
  - `label`
- 常见类型：
  - `text`
  - `password`
  - `radio`
  - `checkbox`
- 需要理解：
  - `label for`
  - `name`
  - `value`
  - `checked`
  - `selected`

---

## 10. 页面布局三大阶段

### 10.1 标准流

- 默认布局方式
- 元素从上到下、从左到右依次排列
- 最适合做基础文档排版

### 10.2 定位

`position` 的 5 个常见值：

- `static`
- `relative`
- `absolute`
- `fixed`
- `sticky`

重点理解：

- `relative`：相对自己原来的位置移动，不脱离标准流
- `absolute`：脱离标准流，参照最近的定位祖先元素
- `fixed`：脱离标准流，参照视口
- `sticky`：介于相对定位和固定定位之间

核心口诀：

- 子绝父相：子元素绝对定位时，父元素通常设为相对定位

### 10.3 z-index

- 控制定位元素的层级
- 只对定位元素更有意义
- 数值越大越靠上，但也受层叠上下文影响

### 10.4 浮动

- `float: left/right`
- 早期主要用于多列布局，现在更多用于图文环绕或旧项目维护

浮动特点：

- 会脱离标准流
- 会影响后续元素布局
- 可能造成父元素高度塌陷

清除浮动常见做法：

- `clear: both`
- 额外清除元素
- 伪元素清除浮动
- 某些情况下通过 BFC/overflow 解决

### 10.5 Flex 布局

这是当前最重要的一维布局方案。

#### 容器属性

- `display: flex`
- `flex-direction`
- `flex-wrap`
- `flex-flow`
- `justify-content`
- `align-items`
- `align-content`

#### 项目属性

- `order`
- `flex-grow`
- `flex-shrink`
- `flex-basis`
- `flex`
- `align-self`

重点理解：

- 主轴和交叉轴
- 剩余空间如何分配
- 子项如何拉伸、压缩、换行和对齐

布局选择建议：

- 简单文档排版：标准流
- 精确叠放：定位
- 老式多列兼容：浮动
- 现代组件和页面：Flex

---

## 11. 工具、规范与工程思维

### 11.1 Emmet

常见语法：

- `>` 子代
- `+` 兄弟
- `*` 多个
- `^` 返回上一级
- `()` 分组
- `{}` 文本内容
- `$` 编号

作用：

- 大幅提升 HTML / CSS 编写效率

### 11.2 CSS 编写顺序

推荐思路：

1. 先写布局方式
2. 再写显示与可见性
3. 再写盒模型
4. 再写文字
5. 再写背景
6. 最后写动画、变换、溢出等补充属性

### 11.3 组件化思想

- 页面不要一整块写到底
- 应拆成头部、导航、卡片、按钮、列表、弹层等可复用块
- 每个模块内部再管理自己的结构和样式

### 11.4 meta / favicon / 编码

- `meta charset`：声明字符编码
- `meta name`：描述页面级元信息
- `meta http-equiv`：兼容性等浏览器指令
- `link rel="icon"`：设置网站图标

---

## 12. 动效与补充方向

这部分在原笔记里主要是标题级提醒，属于扩展学习方向：

- `transform`
  - 位移、缩放、旋转、倾斜
- `transition`
  - 属性变化的过渡动画
- `animation`
  - 基于关键帧的复杂动画
- `vertical-align`
  - 处理行内级、inline-block、表格单元格等对齐问题

建议把它们放到布局和视觉效果掌握之后再系统补。

---

## 13. 高频易混点总结

### 13.1 `display: none` / `visibility: hidden` / `opacity: 0`

- `display: none`：不显示，不占位
- `visibility: hidden`：不显示，占位
- `opacity: 0`：透明，占位，可能还能点击

### 13.2 `text-align: center` 和 `margin: 0 auto`

- `text-align: center`：让行内内容居中
- `margin: 0 auto`：让定宽块级元素水平居中

### 13.3 `height` 和 `line-height`

- `height`：盒子高度
- `line-height`：文字行高

### 13.4 `absolute` 和 `fixed`

- `absolute`：相对最近定位祖先
- `fixed`：相对视口

### 13.5 `nth-child` 和 `nth-of-type`

- `nth-child`：看第几个孩子
- `nth-of-type`：看同类型中的第几个

### 13.6 `content-box` 和 `border-box`

- `content-box`：宽高不含内边距和边框
- `border-box`：宽高包含内边距和边框

### 13.7 `background-image` 和 `img`

- 背景图是装饰
- `img` 是内容

---

## 14. 建议的复习顺序

如果要重新过一遍，建议顺序如下：

1. CSS 是什么、三种引入方式、颜色表示
2. 文本属性、字体属性
3. 选择器、伪类、伪元素
4. 继承、层叠、权重、display
5. 盒子模型、margin/padding/border、box-sizing
6. 背景、圆角、阴影
7. 定位
8. 浮动
9. Flex
10. Web 字体、精灵图、Emmet、规范、组件化
11. transform / transition / animation

---

## 15. 一句话总纲

CSS 学习的核心不是“背属性”，而是建立下面这套判断顺序：

1. 先判断元素属于什么类型，是否在标准流中。
2. 再判断选择器能否准确选中它，以及谁的优先级更高。
3. 再判断盒模型、尺寸、间距如何影响布局。
4. 最后才是定位、浮动、Flex、动画这些具体实现手段。

一旦这条链路清楚，绝大多数样式问题都能定位出来。
