---
title: CSS3新特性
date: 2019-04-28 12:37:08
tags:
  - CSS
categories: CSS
---

# CSS3 新特性

## 1. 选择器

CSS3 中新添加了很多选择器，解决了很多之前需要用 javascript 才能解决的布局问题。

- element1~element2: 选择前面有 element1 元素的每个 element2 元素。
- [attribute^=value] ：选择某元素 attribute 属性是以 value 开头的。
- [attribute$=value]：选择某元素 attribute 属性是以 value 结尾的。
- [attribute*=value]：选择某元素 attribute 属性包含 value 字符串的。
- E:first-of-type: 选择属于其父元素的首个 E 元素的每个 E 元素。
- E:last-of-type: 选择属于其父元素的最后 E 元素的每个 E 元素。
- E:only-of-type: 选择属于其父元素唯一的 E 元素的每个 E 元素。
- E:only-child: 选择属于其父元素的唯一子元素的每个 E 元素。
- E:nth-child(n): 选择属于其父元素的第 n 个子元素的每个 E 元素。
- E:nth-last-child(n): 选择属于其父元素的倒数第 n 个子元素的每个 E 元素。
- E:nth-of-type(n): 选择属于其父元素第 n 个 E 元素的每个 E 元素。
- E:nth-last-of-type(n): 选择属于其父元素倒数第 n 个 E 元素的每个 E 元素。
- E:last-child: 选择属于其父元素最后一个子元素每个 E 元素。
- :root: 选择文档的根元素。
- E:empty: 选择没有子元素的每个 E 元素（包括文本节点)。
- E:target: 选择当前活动的 E 元素。
- E:enabled: 选择每个启用的 E 元素。
- E:disabled: 选择每个禁用的 E 元素。
- E:checked: 选择每个被选中的 E 元素。
- E:not(selector): 选择非 selector 元素的每个元素。
- E::selection: 选择被用户选取的元素部分。
  <!-- more -->

## 2. Transition,Transform 和 Animation

这三个特性是 CSS3 新增的和动画相关的特性。

- Transition

  Transition 可以在当元素从一种样式变换为另一种样式时为元素添加效果，而不用使用 Flash 动画或 JavaScript。
  Transition 有如下属性：

  - transition-property: 规定应用过渡的 CSS 属性的名称。
  - transition-duration: 规定完成过渡效果需要多长时间。
  - transition-delay: 规定过渡效果何时开始，默认是 0。
  - transition-timing-function: 规定过渡效果的时间曲线，默认是”ease”，还有 linear、ease-in、ease-out、ease-in-out 和 cubic-bezier 等过渡类型。
  - transition: 简写属性，用于在一个属性中设置四个过渡属性。

- Transform

  Transform 用来向元素应用各种 2D 和 3D 转换，该属性允许我们对元素进行旋转、缩放、移动或倾斜等操作。

  变换类型：

  - none: 定义不进行转换。
  - matrix(n,n,n,n,n,n): 定义 2D 转换，使用六个值的矩阵。
  - matrix3d(n,n,n,n,n,n,n,n,n,n,n,n,n,n,n,n): 定义 3D 转换，使用 16 个值的 4x4 矩阵。
  - translate(x,y): 定义 2D 位移转换。
  - translate3d(x,y,z): 定义 3D 位移转换。
  - translateX(x): 定义位移转换，只是用 X 轴的值。
  - translateY(y): 定义位移转换，只是用 Y 轴的值。
  - translateZ(z): 定义 3D 位移转换，只是用 Z 轴的值。
  - scale(x,y): 定义 2D 缩放转换。
  - scale3d(x,y,z): 定义 3D 缩放转换。
  - scaleX(x): 通过设置 X 轴的值来定义缩放转换。
  - scaleY(y): 通过设置 Y 轴的值来定义缩放转换。
  - scaleZ(z): 通过设置 Z 轴的值来定义 3D 缩放转换。
  - rotate(angle): 定义 2D 旋转，在参数中规定角度。
  - rotate3d(x,y,z,angle): 定义 3D 旋转。
  - rotateX(angle): 定义沿着 X 轴的 3D 旋转。
  - rotateY(angle): 定义沿着 Y 轴的 3D 旋转。
  - rotateZ(angle): 定义沿着 Z 轴的 3D 旋转。
  - skew(x-angle,y-angle): 定义沿着 X 和 Y 轴的 2D 倾斜转换。
  - skewX(angle): 定义沿着 X 轴的 2D 倾斜转换。
  - skewY(angle): 定义沿着 Y 轴的 2D 倾斜转换。
  - perspective(n): 为 3D 转换元素定义透视视图。

- Animation

  Animation 让 CSS 拥有了可以制作动画的功能。使用 CSS3 的 Animation 制作动画我们可以省去复杂的 js 代码。

## 3. 边框

CSS3 新增了三个边框属性，分别是 border-radius、box-shadow 和 border-image。

- border-radius 可以创建圆角边框
- box-shadow 可以为元素添加阴影
- border-image 可以使用图片来绘制边框。

## 4. 背景

CSS3 新增了几个关于背景的属性，分别是 background-clip、background-origin、background-size 和 background-break。

- background-clip

  background-clip 属性用于确定背景画区，有以下几种可能的属性：

  - background-clip: border-box; 背景从 border 开始显示
  - background-clip: padding-box; 背景从 padding 开始显示
  - background-clip: content-box; 背景显 content 区域开始显示
  - background-clip: no-clip; 默认属性，等同于 border-box

  通常情况，背景都是覆盖整个元素的，利用这个属性可以设定背景颜色或图片的覆盖范围。

- background-origin

  background-clip 属性用于确定背景的位置，它通常与 background-position 联合使用，可以从 border、padding、content 来计算 background-position（就像 background-clip）。

  - background-origin: border-box; 从 border 开始计算 background-position
  - background-origin: padding-box; 从 padding 开始计算 background-position
  - background-origin: content-box; 从 content 开始计算 background-position

- background-size

  background-size 属性常用来调整背景图片的大小，主要用于设定图片本身。有以下可能的属性：

  - background-size: contain; 缩小图片以适合元素（维持像素长宽比）
  - background-size: cover; 扩展元素以填补元素（维持像素长宽比）
  - background-size: 100px 100px; 缩小图片至指定的大小
  - background-size: 50% 100%; 缩小图片至指定的大小，百分比是相对包 含元素的尺寸

- background-break

  CSS3 中，元素可以被分成几个独立的盒子（如使内联元素 span 跨越多行），background-break 属性用来控制背景怎样在这些不同的盒子中显示。

  - background-break: continuous; 默认值。忽略盒之间的距离（也就是像元素没有分成多个盒子，依然是一个整体一样）
  - background-break: bounding-box; 把盒之间的距离计算在内；
  - background-break: each-box; 为每个盒子单独重绘背景。

## 5. 文字效果

- word-wrap

  CSS3 中，word-wrap 属性允许您允许文本强制文本进行换行，即这意味着会对单词进行拆分。所有主流浏览器都支持 word-wrap 属性。

- text-overflow

  它与 word-wrap 是协同工作的，word-wrap 设置或检索当当前行超过指定容器的边界时是否断开转行，而 text-overflow 则设置或检索当当前行超过指定容器的边界时如何显示。对于“text-overflow”属性，有“clip”和“ellipsis”两种可供选择。

- text-shadow

  CSS3 中，text-shadow 可向文本应用阴影。能够规定水平阴影、垂直阴影、模糊距离，以及阴影的颜色。

- text-decoration

  CSS3 里面开始支持对文字的更深层次的渲染，具体有三个属性可供设置：

  - text-fill-color: 设置文字内部填充颜色
  - text-stroke-color: 设置文字边界填充颜色
  - text-stroke-width: 设置文字边界宽度

## 6. 渐变

CSS3 新增了渐变效果，包括 linear-gradient(线性渐变)和 radial-gradient(径向渐变)。

## 7. @font-face 特性

通过 CSS3，web 设计师可以使用他们喜欢的任意字体。当您您找到或购买到希望使用的字体时，可将该字体文件存放到 web 服务器上，它会在需要时被自动下载到用户的计算机上。字体是在 CSS3 @font-face 规则中定义的。Firefox、Chrome、Safari 以及 Opera 支持 .ttf(True Type Fonts)和 .otf(OpenType Fonts)类型的字体。IE9+ 支持新的@font-face 规则，但是仅支持 .eot 类型的字体(Embedded OpenType)。

在新的@font-face 规则中，必须首先定义字体的名称（比如 myFont），然后指向该字体文件。
如需为 HTML 元素使用字体，请通过 font-family 属性来引用字体的名称 (myFont)

## 8. 多列布局

通过 CSS3，能够创建多个列来对文本进行布局，IE10 和 Opera 支持多列属性。Firefox 需要前缀-moz-，Chrome 和 Safari 需要前缀-webkit-。主要有如下三个属性：

- column-count: 规定元素应该被分隔的列数。
- column-gap: 规定列之间的间隔。
- column-rule: 设置列之间的宽度、样式和颜色规则

## 9. 用户界面

CSS3 中，新的用户界面特性包括重设元素尺寸、盒尺寸以及轮廓等。Firefox、Chrome 以及 Safari 支持 resize 属性。IE、Chrome、Safari 以及 Opera 支持 box-sizing 属性。Firefox 需要前缀-moz-。
所有主流浏览器都支持 outline-offset 属性，除了 IE。

- resize

  resize 属性规定是否可由用户调整元素尺寸。如果希望此属性生效，需要设置元素的 overflow 属性，值可以是 auto、hidden 或 scroll。

- box-sizing

  box-sizing 属性可设置的值有 content-box、border-box 和 inherit。

  - content-box: padding 和 border 不被包含在定义的 width 和 height 之内。对象的实际宽度等于设置的 width 值和 border、padding 之和，即 (Element width = width + border + padding)，此属性表现为标准模式下的盒模型。
  - border-box: padding 和 border 被包含在定义的 width 和 height 之内。对象的实际宽度就等于设置的 width 值，即使定义有 border 和 padding 也不会改变对象的实际宽度，即 (Element width = width)，此属性表现为怪异模式下的盒模型。

- outline-offset

  outline-offset 属性对轮廓进行偏移，并在超出边框边缘的位置绘制轮廓。
