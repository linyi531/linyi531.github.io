---
title: CSS多列布局
date: 2019-05-08 11:25:33
tags:
  - CSS
categories: CSS
cover_img: https://i.screenshot.net/vr7g1cv
feature_img: https://i.screenshot.net/vr7g1cv
---

## 1.单列布局

常见的单列布局有两种：

- header,content 和 footer 等宽的单列布局
- header 与 footer 等宽,content 略窄的单列布局
  <!-- more -->

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iysrsoe5j31100h2q3f.jpg)

### 第一种：

对于第一种，先通过对 header,content,footer 统一设置 width：1000px;或者 max-width：1000px(这两者的区别是当屏幕小于 1000px 时，前者会出现滚动条，后者则不会，显示出实际宽度);然后设置 margin:auto 实现居中即可得到。

```html
<div class="header"></div>
<div class="content"></div>
<div class="footer"></div>
```

```css
.header {
  margin: 0 auto;
  max-width: 960px;
  height: 100px;
  background-color: blue;
}
.content {
  margin: 0 auto;
  max-width: 960px;
  height: 400px;
  background-color: aquamarine;
}
.footer {
  margin: 0 auto;
  max-width: 960px;
  height: 100px;
  background-color: aqua;
}
```

### 第二种：

对于第二种，header、footer 的内容宽度不设置，块级元素充满整个屏幕，但 header、content 和 footer 的内容区设置同一个 width，并通过 margin:auto 实现居中。

```html
<div class="header">
  <div class="nav"></div>
</div>
<div class="content"></div>
<div class="footer"></div>
```

```css
.header {
  margin: 0 auto;
  max-width: 960px;
  height: 100px;
  background-color: blue;
}
.nav {
  margin: 0 auto;
  max-width: 800px;
  background-color: darkgray;
  height: 50px;
}
.content {
  margin: 0 auto;
  max-width: 800px;
  height: 400px;
  background-color: aquamarine;
}
.footer {
  margin: 0 auto;
  max-width: 960px;
  height: 100px;
  background-color: aqua;
}
```

## 2.两列自适应布局

**两列自适应布局是指一列由内容撑开，另一列撑满剩余宽度的布局方式**

### float+overflow:hidden

如果是普通的两列布局，**浮动+普通元素的 margin**便可以实现，但如果是自适应的两列布局，利用**float+overflow:hidden**便可以实现，**这种办法主要通过 overflow 触发 BFC,而 BFC 不会重叠浮动元素**。由于设置 overflow:hidden 并不会触发 IE6-浏览器的 haslayout 属性，所以需要设置 zoom:1 来兼容 IE6-浏览器。具体代码如下：

```html
<div class="parent" style="background-color: lightgrey;">
  <div class="left" style="background-color: lightblue;">
    <p>left</p>
  </div>
  <div class="right" style="background-color: lightgreen;">
    <p>right</p>
    <p>right</p>
  </div>
</div>
```

```css
.parent {
  overflow: hidden;
  zoom: 1;
}
.left {
  float: left;
  margin-right: 20px;
}
.right {
  overflow: hidden;
  zoom: 1;
}
```

注意点:如果侧边栏在右边时，注意渲染顺序。即在 HTML 中，先写侧边栏后写主内容

### Flex 布局

Flex 布局，也叫弹性盒子布局，区区简单几行代码就可以实现各种页面的的布局。

```css
//html部分同上
.parent {
  display: flex;
}
.right {
  margin-left: 20px;
  flex: 1;
}
```

### grid 布局

Grid 布局，是一个基于网格的二维布局系统，目的是用来优化用户界面设计。

```css
//html部分同上
.parent {
  display: grid;
  grid-template-columns: auto 1fr;
  grid-gap: 20px;
}
```

## 3.三栏布局

**特征：中间列自适应宽度，旁边两侧固定宽度**

### 圣杯布局

- #### 特点

**比较特殊的三栏布局，同样也是两边固定宽度，中间自适应，唯一区别是 dom 结构必须是先写中间列部分，这样实现中间列可以优先加载**。

```html
<article class="container">
  <div class="center">
    <h2>圣杯布局</h2>
  </div>
  <div class="left"></div>
  <div class="right"></div>
</article>
```

```css
.container {
  padding-left: 220px; //为左右栏腾出空间
  padding-right: 220px;
}
.left {
  float: left;
  width: 200px;
  height: 400px;
  background: red;
  margin-left: -100%;
  position: relative;
  left: -220px;
}
.center {
  float: left;
  width: 100%;
  height: 500px;
  background: yellow;
}
.right {
  float: left;
  width: 200px;
  height: 400px;
  background: blue;
  margin-left: -200px;
  position: relative;
  right: -220px;
}
```

- #### 实现步骤

  - 三个部分都设定为左浮动，**否则左右两边内容上不去，就不可能与中间列同一行**。然后设置 center 的宽度为 100%(**实现中间列内容自适应**)，此时，left 和 right 部分会跳到下一行
  - 通过设置 margin-left 为负值让 left 和 right 部分回到与 center 部分同一行
  - 通过设置父容器的 padding-left 和 padding-right，让左右两边留出间隙。
  - 通过设置相对定位，让 left 和 right 部分移动到两边。

- #### 缺点

  - center 部分的最小宽度不能小于 left 部分的宽度，否则会 left 部分掉到下一行
  - 如果其中一列内容高度拉长(如下图)，其他两列的背景并不会自动填充。(借助等高布局正 padding+负 margin 可解决，下文会介绍)

### 双飞翼布局

- #### 特点

**同样也是三栏布局，在圣杯布局基础上进一步优化，解决了圣杯布局错乱问题，实现了内容与布局的分离。而且任何一栏都可以是最高栏，不会出问题**。

```html
<article class="container">
  <div class="center">
    <div class="inner">双飞翼布局</div>
  </div>
  <div class="left"></div>
  <div class="right"></div>
</article>
```

```css
.container {
  min-width: 600px; //确保中间内容可以显示出来，两倍left宽+right宽
}
.left {
  float: left;
  width: 200px;
  height: 400px;
  background: red;
  margin-left: -100%;
}
.center {
  float: left;
  width: 100%;
  height: 500px;
  background: yellow;
}
.center .inner {
  margin: 0 200px; //新增部分
}
.right {
  float: left;
  width: 200px;
  height: 400px;
  background: blue;
  margin-left: -200px;
}
```

- #### 实现步骤(前两步与圣杯布局一样)

  - 三个部分都设定为左浮动，然后设置 center 的宽度为 100%，此时，left 和 right 部分会跳到下一行；
  - 通过设置 margin-left 为负值让 left 和 right 部分回到与 center 部分同一行；
  - center 部分增加一个内层 div，并设 margin: 0 200px；

- #### 缺点

  **多加一层 dom 树节点，增加渲染树生成的计算量**。

- #### 两种布局实现方式对比:

  - 两种布局方式都是把主列放在文档流最前面，使主列优先加载。
  - 两种布局方式在实现上也有相同之处，都是让三列浮动，然后通过负外边距形成三列布局。
  - 两种布局方式的不同之处在于如何处理中间主列的位置：
    **圣杯布局是利用父容器的左、右内边距+两个从列相对定位**；
    **双飞翼布局是把主列嵌套在一个新的父级块中利用主列的左、右外边距进行布局调整**

### 浮动布局

```html
<article class="left-right-center">
  <div class="left"></div>
  <div class="right"></div>
  // 右栏部分要写在中间内容之前
  <div class="center">
    <h2>浮动解决方案</h2>
    1.这是三栏布局的浮动解决方案； 2.这是三栏布局的浮动解决方案；
    3.这是三栏布局的浮动解决方案； 4.这是三栏布局的浮动解决方案；
    5.这是三栏布局的浮动解决方案； 6.这是三栏布局的浮动解决方案；
  </div>
</article>
```

```css
.layout.float .left {
  float: left;
  width: 300px;
  background: red;
}
.layout.float .center {
  background: yellow;
}
.layout.float .right {
  float: right;
  width: 300px;
  background: blue;
}
```

这种布局方式，dom 结构必须是先写浮动部分，然后再中间块，否则右浮动块会掉到下一行。
**浮动布局的优点就是比较简单，兼容性也比较好。但浮动布局是有局限性的，浮动元素脱离文档流，要做清除浮动，这个处理不好的话，会带来很多问题，比如父容器高度塌陷等**。

### 绝对布局

```html
<article class="left-center-right">
  <div class="left"></div>
  <div class="center">
    <h2>绝对定位解决方案</h2>
    1.这是三栏布局的浮动解决方案； 2.这是三栏布局的浮动解决方案；
    3.这是三栏布局的浮动解决方案； 4.这是三栏布局的浮动解决方案；
    5.这是三栏布局的浮动解决方案； 6.这是三栏布局的浮动解决方案；
  </div>
  <div class="right"></div>
</article>
```

```css
.layout.absolute .left-center-right > div {
  position: absolute; //三块都是绝对定位
}
.layout.absolute .left {
  left: 0;
  width: 300px;
  background: red;
}
.layout.absolute .center {
  right: 300px;
  left: 300px; //离左右各三百
  background: yellow;
}
.layout.absolute .right {
  right: 0;
  width: 300px;
  background: blue;
}
```

**绝对定位布局优点就是快捷，设置很方便，而且也不容易出问题。缺点就是，容器脱离了文档流，后代元素也脱离了文档流，高度未知的时候，会有问题，这就导致了这种方法的有效性和可使用性是比较差的。**

### flexbox 布局

```html
<article class="left-center-right">
  <div class="left"></div>
  <div class="center">
    <h2>flexbox解决方案</h2>
    1.这是三栏布局的浮动解决方案； 2.这是三栏布局的浮动解决方案；
    3.这是三栏布局的浮动解决方案； 4.这是三栏布局的浮动解决方案；
    5.这是三栏布局的浮动解决方案； 6.这是三栏布局的浮动解决方案；
  </div>
  <div class="right"></div>
</article>
```

```css
.layout.flexbox .left-center-right {
  display: flex;
}
.layout.flexbox .left {
  width: 300px;
  background: red;
}
.layout.flexbox .center {
  background: yellow;
  flex: 1;
}
.layout.flexbox .right {
  width: 300px;
  background: blue;
}
```

**flexbox 布局是 css3 里新出的一个，它就是为了解决上述两种方式的不足出现的，是比较完美的一个。目前移动端的布局也都是用 flexbox。 flexbox 的缺点就是 IE10 开始支持，但是 IE10 的是-ms 形式的。**

### 表格布局

```html
<article class="left-center-right">
  <div class="left"></div>
  <div class="center">
    <h2>表格布局解决方案</h2>
    1.这是三栏布局的浮动解决方案； 2.这是三栏布局的浮动解决方案；
    3.这是三栏布局的浮动解决方案； 4.这是三栏布局的浮动解决方案；
    5.这是三栏布局的浮动解决方案； 6.这是三栏布局的浮动解决方案；
  </div>
  <div class="right"></div>
</article>
```

```css
.layout.table .left-center-right {
  display: table;
  height: 150px;
  width: 100%;
}
.layout.table .left-center-right > div {
  display: table-cell;
}
.layout.table .left {
  width: 300px;
  background: red;
}
.layout.table .center {
  background: yellow;
}
.layout.table .right {
  width: 300px;
  background: blue;
}
```

**表格布局的兼容性很好(见下图)，在 flex 布局不兼容的时候，可以尝试表格布局。当内容溢出时会自动撑开父元素**。

**表格布局也是有缺陷:① 无法设置栏边距；② 对 seo 不友好；③ 当其中一个单元格高度超出的时候，两侧的单元格也是会跟着一起变高的，然而有时候这并不是我们想要的效果。**

### 网格布局

```html
<article class="left-center-right">
  <div class="left"></div>
  <div class="center">
    <h2>网格布局解决方案</h2>
    1.这是三栏布局的浮动解决方案； 2.这是三栏布局的浮动解决方案；
    3.这是三栏布局的浮动解决方案； 4.这是三栏布局的浮动解决方案；
    5.这是三栏布局的浮动解决方案； 6.这是三栏布局的浮动解决方案；
  </div>
  <div class="right"></div>
</article>
```

```css
.layout.grid .left-center-right {
  display: grid;
  width: 100%;
  grid-template-columns: 300px auto 300px;
  grid-template-rows: 150px; //行高
}
.layout.grid .left {
  background: red;
}
.layout.grid .center {
  background: yellow;
}
.layout.grid .right {
  background: blue;
}
```

**CSS Grid 是创建网格布局最强大和最简单的工具。就像表格一样，网格布局可以让 Web 设计师根据元素按列或行对齐排列，但他和表格不同，网格布局没有内容结构，从而使各种布局不可能与表格一样。例如，一个网格布局中的子元素都可以定位自己的位置，这样他们可以重叠和类似元素定位**。

**但网格布局的兼容性不好。IE10+上支持，而且也仅支持部分属性**。

## 4.等高布局

等高布局是指子元素在父元素中高度相等的布局方式。接下来我们介绍常见几种实现方式：

- #### 利用正 padding+负 margin

我们通过等高布局便可解决圣杯布局的第二点缺点，因为背景是在 padding 区域显示的，**设置一个大数值的 padding-bottom，再设置相同数值的负的 margin-bottom，并在所有列外面加上一个容器，并设置 overflow:hidden 把溢出背景切掉**。这种可能实现多列等高布局，并且也能实现列与列之间分隔线效果，结构简单，兼容所有浏览器。新增代码如下：

```css
.center,
.left,
.right {
  padding-bottom: 10000px;
  margin-bottom: -10000px;
}
.container {
  padding-left: 220px;
  padding-right: 220px;
  overflow: hidden; //把溢出背景切掉
}
```

- #### 利用背景图片

这种方法是我们实现等高列最早使用的一种方法，就是使用背景图片，在列的父元素上使用这个背景图进行 Y 轴的铺放，从而实现一种等高列的假象。实现方法简单，兼容性强，不需要太多的 css 样式就可以轻松实现,但此方法不适合流体布局等高列的布局。

在制作样式之前需要一张类似下面的背景图：

[![img](https://camo.githubusercontent.com/7b26b4a48392ca95709486320efc031dd8e9ea58/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f31322f32312f313637636633633031343664323436343f773d35303026683d313226663d67696626733d333133)](https://camo.githubusercontent.com/7b26b4a48392ca95709486320efc031dd8e9ea58/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f31322f32312f313637636633633031343664323436343f773d35303026683d313226663d67696626733d333133)

```html
<div class="”container" clearfix”>
  <div class="”left”"></div>
  <div class="”content”"></div>
  <div class="”right”"></div>
</div>
```

```css
.container {
  background: url("column.png") repeat-y;
  width: 960px;
  margin: 0 auto;
}
.left {
  float: left;
  width: 220px;
}
.content {
  float: left;
  width: 480px;
}
.right {
  float: left;
  width: 220px;
}
```

- #### 模仿表格布局

这是一种非常简单，易于实现的方法。不过兼容性不好，在 ie6-7 无法正常运行。

```html
<div class="container table">
  <div class="containerInner tableRow">
    <div class="column tableCell cell1">
      <div class="left aside">
        ....
      </div>
    </div>
    <div class="column tableCell cell2">
      <div class="content section">
        ...
      </div>
    </div>
    <div class="column tableCell cell3">
      <div class="right aside">
        ...
      </div>
    </div>
  </div>
</div>
```

```css
.table {
  width: auto;
  min-width: 1000px;
  margin: 0 auto;
  padding: 0;
  display: table;
}
.tableRow {
  display: table-row;
}
.tableCell {
  display: table-cell;
  width: 33%;
}
.cell1 {
  background: #f00;
  height: 800px;
}
.cell2 {
  background: #0f0;
}
.cell3 {
  background: #00f;
}
```

- #### 使用边框和定位

这种方法是使用边框和绝对定位来实现一个假的高度相等列的效果。结构简单，兼容各浏览器，容易掌握。假设你需要实现一个两列等高布局，侧栏高度要和主内容高度相等。

```html
<div id="wrapper">
  <div id="mainContent">...</div>
  <div id="sidebar">...</div>
</div>
```

```css
#wrapper {
  width: 960px;
  margin: 0 auto;
}
#mainContent {
  border-right: 220px solid #dfdfdf;
  position: absolute;
  width: 740px;
  height: 800px;
  background: green;
}
#sidebar {
  background: #dfdfdf;
  margin-left: 740px;
  position: absolute;
  height: 800px;
  width: 220px;
}
```

## 5.粘连布局

- #### 特点

  - 有一块内容`<main>`，当`<main>`的高康足够长的时候，紧跟在`<main>`后面的元素`<footer>`会跟在`<main>`元素的后面。
  - 当`<main>`元素比较短的时候(比如小于屏幕的高度),我们期望这个`<footer>`元素能够“粘连”在屏幕的底部

具体代码如下：

```html
<div id="wrap">
  <div class="main">
    main <br />
    main <br />
    main <br />
  </div>
</div>
<div id="footer">footer</div>
```

```css
* {
  margin: 0;
  padding: 0;
}
html,
body {
  height: 100%; //高度一层层继承下来
}
#wrap {
  min-height: 100%;
  background: pink;
  text-align: center;
  overflow: hidden;
}
#wrap .main {
  padding-bottom: 50px;
}
#footer {
  height: 50px;
  line-height: 50px;
  background: deeppink;
  text-align: center;
  margin-top: -50px;
}
```

- #### 实现步骤

  - footer 必须是一个独立的结构，与 wrap 没有任何嵌套关系
  - wrap 区域的高度通过设置 min-height，变为视口高度
  - footer 要使用 margin 为负来确定自己的位置
  - 在 main 区域需要设置 padding-bottom。这也是为了防止负 margin 导致 footer 覆盖任何实际内容。
