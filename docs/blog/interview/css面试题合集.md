---
title: css面试题合集
---

## css选择器有哪些

1. id选择器（#myid）
2. 类选择器（.myclassname）
3. 标签选择器（div,h1,p）
4. 后代选择器（h1 p）
5. 相邻后代选择器（子）选择器（ul>li）
6. 兄弟选择器（li~a）
7. 相邻兄弟选择器（li+a）
8. 属性选择器（a[rel="external"]）
9. 伪类选择器（a:hover,li:nth-child）
10. 伪元素选择器（::before、::after）
11. 通配符选择器（*）

## display 有哪些值？说明他们的作用

* block 块类型。默认宽度为父元素宽度，可设置宽高，换行显示。
* none 元素不显示，并从文档流中移除。
* inline 行内元素类型。默认宽度为内容宽度，不可设置宽高，同行显示。
* inline-block 默认宽度为内容宽度，可以设置宽高，同行显示。
* list-item 像块类型元素一样显示，并添加样式列表标记。
* table 此元素会作为块级表格来显示。
* inherit 规定应该从父元素继承display属性的值。

## css 动画与 js 动画哪个性能更好

* CSS3 的动画

  1. 在性能上会稍微好一些，浏览器会对 CSS3 的动画做一些优化（比如专门新建一个图层用来跑动画）

  2. 代码相对简单

  3. 在动画控制上不够灵活

  4. 兼容性不好

  5. 部分动画功能无法实现（如滚动动画，视差滚动等）

* JavaScript 的动画

  正好弥补了 css 缺点，控制能力很强，可以单帧的控制、变换，同时写得好完全可以兼容 IE6，并且功能强大。

  总结： 对于一些复杂控制的动画，使用 javascript 会比较好。而在实现一些小的交互动效的时候，可以多考虑 CSS

## css 对一个非定长宽的块状元素如何做垂直水平居中

定位：

```css
.parent { 
  position: relative;
} 
.child {
   position: absolute; 
   top: 50%; 
   left: 50%; 
   transform: translate(-50%, -50%);
}
```

## css 如何画一个三角形

```css
.triangle {
  width: 0;
  border: 100px solid transparent;
  border-bottom: 100px solid rgba(66, 142, 212, 0.4);
}
```

## css 如何设置多行超出显示省略号

使用 `-webkit-line-clamp` 来设置多行超出显示省略号

```css
overflow: hidden;
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 2;
```

## css 如何设置一行超出显示省略号

```css
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
```

## css 对一个非定长宽的块状元素如何做垂直水平居中

通过 css3 中伪类 :nth-child 来实现。

其中 `:nth-child(an+b)` 匹配下标 { an + b; n = 0, 1, 2, ...} 且结果为整数的子元素

`nth-child(2n)/nth-child(even)`: 双行样式
`nth-child(2n+1)/nth-child(odd)`: 单行样式

其中 tr 在表格中代表行，实现表格中单双行样式就很简单了：

```css
tr:nth-child(2n) {
  background-color: red;
}

tr:nth-child(2n + 1) {
  background-color: blue;
}
```

同理：

如何匹配最前三个子元素: `:nth-child(-n+3)`
如何匹配最后三个子元素: `:nth-last-child(-n+3)`

## css 如何实现一个元素的水平垂直居中

flex：

```html
<style>
body {
    display: flex;
    justify-content: center;
    align-items: center;
}
.box {
  width: 50px;
  height: 50px;
}
</style>

<body>
<div class="box"><div>
</body>
```

## css 如何实现左侧固定 300px，右侧自适应的布局

使用 `flex` 布局，左侧 `300px`，右侧 `flex-grow: 1`：

```css
.container {
  display: flex;
}

.left {
  flex-basis: 300px;
  flex-shrink: 0;
}

.main {
  flex-grow: 1;
}
```

如果只使用 Grid 布局，则代码会更加简单，只需要控制容器的 CSS 属性：

```css
.container {
  display: grid;
  grid-template-columns: 300px 1fr;
}
```

使用 calc 方法 ：

```css
.left {
  width:330px;
} 
.main {
  width: calc(100% - 330px)
}
```

## css 隐藏页面中某个元素的几种方法

* display:none

  通过 CSS 操控 display，移出文档流

  ```css
  display: none;
  ```

* opacity: 0

  透明度为 0，仍在文档流中，当作用于其上的事件(如点击)仍有效

  ```css
  opacity: 0;
  ```

* visibility: hidden

  透明度为 0，仍在文档流中，但作用于其上的事件(如点击)无效，这也是 visibility:hidden 与 opacity: 0 的区别

  ```css
  visibility: hidden;
  ```

* content-visibility

  移出文档流，但是再次显示时消耗性能低

  ```css
  content-visibility: hidden;
  ```

* 绝对定位于当前页面的不可见位置

  ```css
  position: absolute;
  top: -9000px;
  left: -9000px;
  ```

* 字体大小设置为 0

  ```css
  font-size: 0;
  ```

## css 中'+' 与 '~' 选择器有什么不同

* \+ 选择器匹配紧邻的兄弟元素
* ~ 选择器匹配随后的所有兄弟元素

## css 中关于选择器的权重

`css specificity` 即 css 中关于选择器的权重，以下三种类型的选择器依次下降

* `id` 选择器，如 #app

* `class` 、`attribute` 与 `pseudo-classes` 选择器，如 `.header`、[type="radio"] 与 `:hover`
* type 标签选择器和伪元素选择器，如 `h1`、`p` 和 `::before`

其中通配符选择器 `*`，组合选择器 `+ ~ >`，否定伪类选择器 `:not()` 对优先级无影响

另有内联样式 `<div class="foo" style="color: red;"></div>` 及 `!important`(最高) 具有更高的权重

## flex 布局中的 flex-basis 与 width 有何区别

1. 当 `flex-direction` 为 `column` 时，主轴为纵轴，此时 `flex-basis` 与 `height` 对应
2. `flex-basis` 的值为理想情况，而在实际情况中可能被压缩

## css 如何自定义滚动条的样式

滚动条相关样式都是伪元素，以 scrollbar 打头，有以下伪元素，从 -webkit 中可见兼容性一般，不过无所谓，现在Chrome 浏览器占大头

* ::-webkit-scrollbar — 整个滚动条.
::-webkit-scrollbar-button — 滚动条上的按钮 (上下箭头).
* ::-webkit-scrollbar-thumb — 滚动条上的滚动滑块.
* ::-webkit-scrollbar-track — 滚动条轨道.
* ::-webkit-scrollbar-track-piece — 滚动条没有滑块的轨道部分.
* ::-webkit-scrollbar-corner — 当同时有垂直滚动条和水平滚动条时交汇的部分.
* ::-webkit-resizer — 某些元素的 corner 部分的部分样式(例:textarea 的可拖动按钮).

但其实最常用的是以下几个伪元素：滚动条、滑块、轨道，如下滚动条设置成功

```css
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

::-webkit-scrollbar-track {
  border-radius: 3px;
  background: rgba(0, 0, 0);
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.08);
}

::-webkit-scrollbar-thumb {
  border-radius: 3px;
  background: rgba(0, 0, 1);
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.2);
}
```

## css 如何实现容器中子元素三个三列布局，子元素两个则两列布局

> 问题描述：
容器中有三个及以上元素，三列布局
容器只有两个元素，两列布局平分
容器只有一个元素，一列布局全占

可以使用 flex 布局或者是 grid 布局

关健在于如何判断有多少元素。我们只需要判断有 1 个，有 2 个的情况，其他都是三列布局

```css
/* 有1个元素 */
.item:nth-last-child(1):first-child {
  flex: 1;
}

.item:nth-last-child(1):first-child ~ .item {
  flex: 1;
}

/* 有2个元素 */
.item:nth-last-child(2):first-child {
  flex: 1;
}

.item:nth-last-child(2):first-child ~ .item {
  flex: 1;
}
```

## css 网站设置字体时，如何设置优先使用系统默认字体

```css
font-family: system-ui;
```

`system-ui` 将会自动选取系统默认字体作为字体

## css 如何设置方格背景

```css
background: 
linear-gradient(90deg, rgba(200, 200, 200, 0.1) 3%, transparent 0),
linear-gradient(rgba(200, 200, 200, 0.1) 3%, transparent 0);
background-size: 20px 20px;
```

## 伪类和伪元素的区别

引入伪类和伪元素概念是为了格式化文档树以外的信息。也就是说，伪类和伪元素是用修饰不在文档树中的部分，比如，一句话中的第一个字母，或者是列表中的第一个元素。

**伪类用于当已有的元素处于某个状态时，为其添加对应的样式，这个状态是根据用户行为动态变化的**。比如说，当用户悬停在指定的元素时，我们可以通过 `:hover` 来描述这个元素的状态。

**伪元素用于创建一些不在文档树中的元素，并为其添加样式**。它们允许我们为元素的某些部分设置样式。比如说，我们可以通过 `::before` 来在一个元素前增加一些文本，并为这些文本添加样式。虽然用户可以看到这些文本，但是这些文本实际上不在文档树中。

有时你会发现伪元素使用了两个冒号（::）而不是一个冒号（:）。这是 `CSS3` 的一部分，并尝试区分伪类和伪元素。大多数浏览器都支持这两个值。按照规则应该使用（::）而不是（:），从而区分伪类和伪元素。但是，由于在旧版本的W3C规范并未对此进行特别区分，因此目前绝大多数的浏览器都支持使用这两种方式表示伪元素。

## css定位

* relative：生成相对定位的元素，相对于其元素本身所在正常位置进行定位。元素所占的空间还是在原来的位置
* absolute：绝对定位，相对于值不为static的第一个父元素的padding box进行定位，也可以理解为离自己这一级元素最近的
一级 position 设置为 absolute 或者 relative 的父元素的padding box的左上角为原点的。
* static：默认定位
* fixed：固定定位，包含元素是浏览器窗口，可以设置 `left:0;right:0`,将容器宽度占满窗口，也可以设置 `top:0;bottom:0` 将容器高度占满窗口
* sticky：相当于 relative 和 fixed 结合体
