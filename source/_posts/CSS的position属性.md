---
title: CSS的position属性
date: 2019-08-31 21:48:53
tags:
  - CSS
categories: CSS
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g6j7eh7b5pj30u0190wyl.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g6j7eh7b5pj30u0190wyl.jpg
---

# CSS 的 position 属性

## static

默认值。该关键字指定元素使用正常的布局行为，即元素在文档常规流中当前的布局位置。此时 `top`, `right`, `bottom`, `left` 和 `z-index`属性无效。

## relative

### 定位类型

相对定位元素，相对定位的元素是在文档中的正常位置偏移给定的值，但是不影响其他元素的偏移。

### 定位方式

生成相对定位的元素，相对于其正常位置进行定位。该关键字下，元素先放置在未添加定位时的位置，再在不改变页面布局的前提下调整元素位置（因此会在此元素未添加定位时所在位置留下空白，"left:20" 会向元素的 LEFT 位置添加 20 像素。）。`position:relative` 对 `table-group`, `table-row`,`table-column`,`table-cell`,`table-caption` 元素无效。

```html
<div class="box" id="one">One</div>
<div class="box" id="two">Two</div>
<div class="box" id="three">Three</div>
<div class="box" id="four">Four</div>
```

```css
.box {
  display: inline-block;
  width: 100px;
  height: 100px;
  background: red;
  color: white;
}

#two {
  position: relative;
  top: 20px;
  left: 20px;
  background: blue;
}
```

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iyqx5px1j312q0b40t2.jpg)

## absolute

### 定位类型

绝对定位元素，相对定位的元素并未脱离文档流，而绝对定位的元素则脱离了文档流。在布置文档流中其它元素时，绝对定位元素不占据空间。绝对定位元素相对于*最近的非 static 祖先元素*定位。当这样的祖先元素不存在时，则相对于 ICB（inital container block, 初始包含块）。

### 定位方式

生成绝对定位的元素，不为元素预留空间，相对于 static 定位以外的第一个父元素进行定位。元素的位置通过 "left", "top", "right" 以及 "bottom" 属性进行规定。绝对定位的元素可以设置外边距（margins），且不会与其他边距合并。

```html
<div class="box" id="one">One</div>
<div class="box" id="two">Two</div>
<div class="box" id="three">Three</div>
<div class="box" id="four">Four</div>
```

```css
.box {
  display: inline-block;
  background: red;
  width: 100px;
  height: 100px;
  float: left;
  margin: 20px;
  color: white;
}

#three {
  position: absolute;
  top: 20px;
  left: 20px;
}
```

![image-20190823001646550](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iyrszydtj310809e0sz.jpg)

## fixed

### 定位类型

绝对定位元素，固定定位与绝对定位相似，但元素的包含块为 viewport 视口。该定位方式常用于创建在滚动屏幕时仍固定在相同位置的元素。

### 定位方式

生成绝对定位的元素，不为元素预留空间，相对于浏览器窗口进行定位。元素的位置通过 "left", "top", "right" 以及 "bottom" 属性进行规定。元素的位置在屏幕滚动时不会改变。打印时，元素会出现在的每页的固定位置。`fixed` 属性会创建新的层叠上下文。

**当元素祖先的 `transform` 属性非 `none` 时，容器由视口改为该祖先。**

## sticky

### 定位类型

粘性定位元素，粘性定位可以被认为是相对定位和固定定位的混合。元素在跨越特定阈值前为相对定位，之后为固定定位。

### 定位方式

盒位置根据正常流计算(这称为正常流动中的位置)，然后相对于该元素在流中的 flow root（BFC）和 containing block（最近的块级祖先元素）定位。在所有情况下（即便被定位元素为 `table 时`），该元素定位均不对后续元素造成影响。当元素 B 被粘性定位时，后续元素的位置仍按照 B 未定位时的位置来确定。`position: sticky`对 `table` 元素的效果与 `position: relative`相同。

```css
#one {
  position: sticky;
  top: 10px;
}
```

在 viewport 视口滚动到元素 top 距离小于 10px 之前，元素为相对定位。之后，元素将固定在与顶部距离 10px 的位置，直到 viewport 视口回滚到阈值以下。

粘性定位常用于定位字母列表的头部元素。标示 B 部分开始的头部元素在滚动 A 部分时，始终处于 A 的下方。而在开始滚动 B 部分时，B 的头部会固定在屏幕顶部，直到所有 B 的项均完成滚动后，才被 C 的头部替代。

### 例子

```html
<div>
  <dl>
    <dt>A</dt>
    <dd>Andrew W.K.</dd>
    <dd>Apparat</dd>
    <dd>Arcade Fire</dd>
    <dd>At The Drive-In</dd>
    <dd>Aziz Ansari</dd>
  </dl>
  <dl>
    <dt>C</dt>
    <dd>Chromeo</dd>
    <dd>Common</dd>
    <dd>Converge</dd>
    <dd>Crystal Castles</dd>
    <dd>Cursive</dd>
  </dl>
  <dl>
    <dt>E</dt>
    <dd>Explosions In The Sky</dd>
  </dl>
  <dl>
    <dt>T</dt>
    <dd>Ted Leo & The Pharmacists</dd>
    <dd>T-Pain</dd>
    <dd>Thrice</dd>
    <dd>TV On The Radio</dd>
    <dd>Two Gallants</dd>
  </dl>
</div>
```

```css
* {
  box-sizing: border-box;
}

dl {
  margin: 0;
  padding: 24px 0 0 0;
}

dt {
  background: #b8c1c8;
  border-bottom: 1px solid #989ea4;
  border-top: 1px solid #717d85;
  color: #fff;
  font: bold 18px/21px Helvetica, Arial, sans-serif;
  margin: 0;
  padding: 2px 0 0 12px;
  position: -webkit-sticky;
  position: sticky;
  top: -1px;
}

dd {
  font: bold 20px/45px Helvetica, Arial, sans-serif;
  margin: 0;
  padding: 0 0 0 12px;
  white-space: nowrap;
}

dd + dd {
  border-top: 1px solid #ccc;
}
```

### 生效规则

`position:sticky` 的生效是有一定的限制的，总结如下：

1. 须指定 top, right, bottom 或 left 四个阈值其中之一，才可使粘性定位生效。否则其行为与相对定位相同。
   - 并且 `top` 和 `bottom` 同时设置时，`top` 生效的优先级高，`left` 和 `right` 同时设置时，`left` 的优先级高。
2. 设定为 `position:sticky` 元素的任意父节点的 overflow 属性必须是 visible，否则 `position:sticky` 不会生效。这里需要解释一下：
   - 如果 `position:sticky` 元素的任意父节点定位设置为 `overflow:hidden`，则父容器无法进行滚动，所以 `position:sticky` 元素也不会有滚动然后固定的情况。
   - 如果 `position:sticky` 元素的任意父节点定位设置为 `position:relative | absolute | fixed`，则元素相对父元素进行定位，而不会相对 viewprot 定位。
3. 达到设定的阀值。这个还算好理解，也就是设定了 `position:sticky` 的元素表现为 `relative` 还是 `fixed` 是根据元素是否达到设定了的阈值决定的。

### 特性

1. 同一个父容器中的 sticky 元素，如果定位值相等，则会重叠；如果属于不同父元素，则会鸠占鹊巢，挤开原来的元素，形成依次占位的效果。

## initial

`initial` 关键字用于设置 CSS 属性为它的默认值，可作用于任何 CSS 样式。（IE 不支持该关键字）

## inherit：

规定应该从父元素继承 position 属性的值。

每一个 CSS 属性都有一个特性就是，这个属性必然是默认继承的 (`inherited: Yes`) 或者是默认不继承的 (`inherited: no`)其中之一，我们可以在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference) 上通过这个索引查找，判断一个属性的是否继承特性。

### 可继承属性

- 所有元素可继承：visibility 和 cursor
- 内联元素可继承：letter-spacing、word-spacing、white-space、line-height、color、font、 font-family、font-size、font-style、font-variant、font-weight、text- decoration、text-transform、direction
- 块状元素可继承：text-indent 和 text-align
- 列表元素可继承：list-style、list-style-type、list-style-position、list-style-image
- 表格元素可继承：border-collapse

## unset

`unset` 关键字我们可以简单理解为不设置。其实，它是关键字 `initial` 和 `inherit` 的组合。

什么意思呢？也就是当我们给一个 CSS 属性设置了 `unset` 的话：

1. 如果该属性是默认继承属性，该值等同于 `inherit`
2. 如果该属性是非继承属性，该值等同于 `initial`

### 例子

```html
<div class="father">
  <div class="children">子级元素一</div>
  <div class="children unset">子级元素二</div>
</div>

.father { color: red; border: 1px solid black; } .children { color: green;
border: 1px solid blue; } .unset { color: unset; border: unset; }
```

1. 由于 `color` 是可继承样式，设置了 `color: unset` 的元素，最终表现为了父级的颜色 `red`。
2. 由于 `border` 是不可继承样式，设置了 `border: unset` 的元素，最终表现为 `border: initial` ，也就是默认 border 样式，无边框。

## revert

revert 未列入规范
