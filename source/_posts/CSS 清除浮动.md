---
title: CSS 清除浮动
date: 2019-09-29 15:26:33
tags:
  - CSS
  - 清除浮动
categories: CSS
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g7geypljooj31900u0qve.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g7geypljooj31900u0qve.jpg
---

# CSS 清除浮动

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6tbm9sca0j30op0fh0tp.jpg)

## 浮动是什么？

W3school 中给出的浮动定义为**浮动的框可以向左或向右移动，直到它的外边缘碰到包含框或另一个浮动框的边框为止。**由于浮动框脱离文档的普通流中，所以文档的普通流中的块框表现得就像浮动框不存在一样。

## 浮动的特点

浮动的特点，可以用八个字总结：**脱标、贴边、字围和收缩。**

为了更好说明，请看下图：
当框 1 向左浮动时，它脱离文档流（**脱标**）并且向左移动（**贴边**），直到它的左边缘碰到包含框的左边缘。因为它不再处于文档流中，所以它不占据空间，实际上覆盖住了框 2，使框 2 从视图中消失。如果框 2 中有文字，就会围着框 1 排开（**字围**）。

如果把所有三个框都向左浮动，那么框 1 向左浮动直到碰到包含框，另外两个框向左浮动直到碰到前一个浮动框。
![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6tc3n56n2j30em06haa4.jpg)
下面着重讲解下第四个特点--**收缩**

一个浮动的内联元素（比如 span img 标签）不需要设置 display：block 就可以设置宽度。

```html
<head>
  <style>
    div {
      float: left;
      background-color: greenyellow;
    }
  </style>
</head>
<body>
  <div>
    这是一段文字
  </div>
</body>
```

得到以下的效果：

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6tc6sc8yzj30qo01ft8h.jpg)
我们都知道 div 标签是块级元素，会独占一行，然而上面的例子中将 div 设置为左浮后，其宽度不再是占满一行，而是收紧为内部元素的宽度，这就是浮动第四个特征的含义。

## 浮动的缺点

先看下面这段代码：

```html
<head>
  <style>
    .parent {
      border: solid 5px;
      width: 300px;
    }
    .child:nth-child(1) {
      height: 100px;
      width: 100px;
      background-color: yellow;
      float: left;
    }
    .child:nth-child(2) {
      height: 100px;
      width: 100px;
      background-color: red;
      float: left;
    }
    .child:nth-child(3) {
      height: 100px;
      width: 100px;
      background-color: greenyellow;
      float: left;
    }
  </style>
</head>
<body>
  <div class="parent">
    <div class="child"></div>
    <div class="child"></div>
    <div class="child"></div>
  </div>
</body>
```

我们想让父容器包裹着三个浮动元素，然而事与愿违，得到却是这样的结果：
![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6tcc3apdwj30920393yb.jpg)
**这就是浮动带来副作用----父容器高度塌陷，于是清理浮动就显着至关重要。**

## 清理浮动

**清除浮动不是不用浮动，是清除浮动产生的父容器高度塌陷**。

### 套路 1：给浮动元素的父元素添加高度（扩展性不好）

如果一个元素要浮动，那么它的父元素一定要有高度。高度的盒子，才能关住浮动。可以通过直接给父元素设置 height，实际应用中我们不大可能给所有的盒子加高度，不仅麻烦，并且不能适应页面的快速变化；另外一种，父容器的高度可以通过内容撑开（比如 img 图片），实际当中此方法用的比较多。

### 套路 2：clear:both;

在最后一个子元素后新添加一个冗余元素，然后将其设置 clear:both,这样就可以清除浮动。这里强调一点，即**在父级元素末尾添加的元素必须是一个块级元素，否则无法撑起父级元素高度**。

```html
<div id="wrap">
  <div id="inner"></div>
  <div style="clear: both;"></div>
</div>
```

```css
#wrap {
  border: 1px solid;
}
#inner {
  float: left;
  width: 200px;
  height: 200px;
  background: pink;
}
```

### 套路 3：伪元素清除浮动

上面那种办法固然可以清除浮动，但是我们不想在页面中添加这些没有意义的冗余元素，此时如何清除浮动吗？
**结合 :after 伪元素和 IEhack ，可以完美兼容当前主流的各大浏览器，这里的 IEhack 指的是触发 hasLayout**。

```html
<div id="wrap" class="clearfix">
  <div id="inner"></div>
</div>
```

```css
#wrap {
  border: 1px solid;
}
#inner {
  float: left;
  width: 200px;
  height: 200px;
  background: pink;
}
/*开启haslayout*/
.clearfix {
  *zoom: 1;
}
/*ie6 7不支持伪元素*/
.clearfix:after {
  content: "";
  display: block;
  clear: both;
  height: 0;
  line-height: 0;
  visibility: hidden; //允许浏览器渲染它，但是不显示出来
}
```

给浮动元素的父容器添加一个 clearfix 的 class，然后给这个 class 添加一个:after 伪元素，实现元素末尾添加一个看不见的块元素来清理浮动。这是通用的清理浮动方案，推荐使用

### 套路 4：给父元素使用 overflow:hidden;

这种方案让父容器形成了 BFC（块级格式上下文），而 BFC 可以包含浮动，通常用来解决浮动父元素高度坍塌的问题。

**BFC 的触发方式**

我们可以给父元素添加以下属性来触发 BFC：

- float 为 left | right
- overflow 为 hidden | auto | scorll
- display 为 table-cell | table-caption | inline-block
- position 为 absolute | fixed

这里可以给父元素设置 overflow:auto，但是为了兼容 IE 最好使用 overflow:hidden。

**但这种办法有个缺陷：如果有内容出了盒子，用这种方法就会把多的部分裁切掉，所以这时候不能使用。**

**BFC 的主要特征:**

- BFC 容器是一个隔离的容器，和其他元素互不干扰；所以我们可以用触发两个元素的 BFC 来解决垂直边距折叠问题。
- BFC 不会重叠浮动元素
- BFC 可以包含浮动,这可以清除浮动。

### 套路 5：br 标签清浮动

**br 标签存在一个属性：clear。这个属性就是能够清除浮动的利器，在 br 标签中设置属性 clear，并赋值 all。即能清除掉浮动**。

```html
<div id="wrap">
  <div id="inner"></div>
  <br clear="all" />
</div>
```

```css
#wrap {
  border: 1px solid;
}
#inner {
  float: left;
  width: 200px;
  height: 200px;
  background: pink;
}
```
