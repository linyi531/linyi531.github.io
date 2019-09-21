---
title: CSS常用布局方式
date: 2019-05-02 15:44:43
tags:
  - CSS
categories: CSS
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hfmiaxij31900u07wj.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hfmiaxij31900u07wj.jpg
---

# CSS 常用布局方式

## 1.静态布局（固定布局）

### 布局特点

不管浏览器尺寸具体是多少，网页布局始终按照最初写代码时的布局来显示。常规的 pc 的网站都是静态（定宽度）布局的，也就是设置了 min-width，这样的话，如果小于这个宽度就会出现滚动条，如果大于这个宽度则内容居中外加背景，这种设计常见于 pc 端。

  <!-- more -->

### 设计方法

- PC：居中布局，所有样式使用绝对宽度/高度(px)，设计一个 Layout，在屏幕宽高有调整时，使用横向和竖向的滚动条来查阅被遮掩部分；

```css
.wrap {
  width: 640px;
  overflow: hidden;
  margin: 0 auto;
}
```

​ 有固定的版型大小，例如 640px，然后设置 margin：0 auto；来居中。小于 640 时出现滚动条。

- 移动设备：另外建立移动网站，单独设计一个布局，使用不同的域名如 wap.或 m.。在 `<viewport meta>` 标签上设置 `width`，页面的各个元素也采用`px`作为单位。通过用 JS 动态修改标签的`initial-scale`使得页面等比缩放，从而刚好占满整个屏幕。

### 实现方法

- 普通/文档流 布局
- Float 布局
- 绝对布局

### 优缺点

优点：这种布局方式对设计师和 CSS 编写者来说都是最简单的，亦没有兼容性问题。

缺点：显而易见，不能根据用户的屏幕尺寸做出不同的展现。当前，大部分门户网站、大部分企业的 PC 宣传站点都采用了这种布局方式。固定像素尺寸的网页是匹配固定像素尺寸显示器的最简单办法。但这种方法不是一种完全兼容未来网页的制作方法，我们需要一些适应未知设备的方法。

- 窄屏幕滚动条体验很差
- 宽屏有大片空白，不利于空间利用

## 2.流式布局

流式布局（Liquid）的特点（也叫"Fluid") 是**页面元素的宽度**按照屏幕分辨率进行适配调整，但整体布局不变。代表作栅栏系统（网格系统）。网页中主要的划分区域的**尺寸使用百分数**（搭配 min-*、max-*属性使用），例如，设置网页主体的宽度为 80%，min-width 为 960px。图片也作类似处理（width:100%, max-width 一般设定为图片本身的尺寸，防止被拉伸而失真）。

### 布局特点

屏幕分辨率变化时，页面里元素的大小会变化而但布局不变。【这就导致如果屏幕太大或者太小都会导致元素无法正常显示。

### 设计方法

**使用%百分比定义宽度，而高度大都是用 px 来固定住**，可以根据可视区域 (viewport) 和父元素的实时尺寸进行调整，尽可能的适应各种分辨率。往往配合 max-width/min-width 等属性控制尺寸流动范围以免过大或者过小影响阅读。

**这种布局方式在 Web 前端开发的早期历史上，用来应对不同尺寸的 PC 屏幕**（那时屏幕尺寸的差异不会太大），**在当今的移动端开发也是常用布局方式**。流式布局目的是在不同大小的设备上**满屏呈现同样网页**。它是用于解决类似的设备不同分辨率之间的兼容(一般分辨率差异较少)。

**百分比能够设置的属性是 width、height、padding、margin。其他属性比如 border、font-size 不能用百分比设置的。**

- 如果用百分比写 width，那么指的是父元素 width 的百分之多少。
- 如果用百分比写 height，那么指的是父元素 height 的百分之多少。
- 如果用百分比写 padding，那么指的是**父元素 width **的百分之多少，无论是水平的 padding 还是竖直的 padding。
- 如果用百分比写 margin，那么指的是**父元素 width** 的百分之多少，无论是水平的 margin 还是竖直的 margin。
- 不能用百分比写 border 的宽度

### 实现方法

- 允许网页宽度自动调整：`<meta name="viewport" content="width=device-width, initial-scale=1" />`
- 不使用绝对尺寸（包括容器/字体/图片），使用百分比、em、rem、vw、vh 等
- 可使用 flex 等弹性盒子（不要使用 px 定尺寸）

### 优缺点

优点：页面左右满屏。

但缺点明显：

主要的问题\*\*是如果屏幕尺度跨度太大，那么在相对其原始设计而言过小或过大的屏幕上不能正常显示。因为宽度使用%百分比定义，但是高度和文字大小等大都是用 px 来固定，所以在大屏幕的手机下显示效果会变成有些页面元素宽度被拉的很长，但是高度、文字大小还是和原来一样（即，这些东西无法变得“流式”），显示非常不协调

- 使用百分比定义，所以在大屏幕的手机/Pad 下（或者横屏下）显示效果会变成有些页面元素被拉的很大，但是内容数量却不变，显得稀疏不紧凑，空间利用率低下。
- 如果文字也按照百分比放大，则整体效果会非常不协调（老人机效果）。

### 例子

https://www.trip.com/flightsh5/status/

## 3.自适应布局

自适应布局的特点是分别为不同的屏幕分辨率定义布局，即创建多个静态布局，每个静态布局对应一个屏幕分辨率范围。改变屏幕分辨率可以切换不同的静态局部（页面元素位置发生改变），但在每个静态布局中，页面元素不随窗口大小的调整发生变化。可以把自适应布局看作是静态布局的一个合集。

### 布局特点

屏幕分辨率变化时，页面里面元素的位置会变化而大小不会变化。

### 设计方法

使用 @media 媒体查询给不同尺寸和介质的设备切换不同的样式。在优秀的响应范围设计下可以给适配范围内的设备最好的体验，在同一个设备下实际还是固定的布局。

### 实现方式

- 静态布局方法
- 分辨率 detector（media query/server-side detector/UA）

### 优缺点

优点：自适应布局**页面里面元素的位置会变化，很好的解决了流式布局中的大屏空间利用率不高弊端**。

缺点：单个布局容器无法灵活伸缩，未触发布局切换的情况下，容器仍然容易出现静态布局中提到的问题。

### 例子

[www.baidu.com/](https://link.juejin.im/?target=https%3A%2F%2Fwww.baidu.com%2F)

### 自适应设计（AWD）

自适应设计是通过**服务端检测设备类型、从 site 的不同版本中选择最合适该设备类型的设计布局/尺寸的版本进行展示。**它可以使用到所有（包括响应式布局）布局方案。

实现方式：

- server-side detection

- different versions to different devices

  对于 PC: 可使用流式布局；

  对于 Mobile: 可使用流式布局。推荐一个 Rem 解决方案：

  - 设置元素（可以包括字体等）大小为 `rem` （`rem` 是以跟元素`font-size`为基准的单位）
  - 按照屏幕宽度的不同，JS 动态设置 `<html>` 的 `font-size` 大小，元素同样会按照屏幕宽度等比例放大缩小

举个栗子：[www.trip.com/](https://link.juejin.im/?target=https%3A%2F%2Fwww.trip.com%2F)

## 4.响应式布局（媒体查询）

随着 CSS3 出现了媒体查询技术，又出现了响应式设计的概念。响应式设计的目标是确保一个页面在所有终端上（各种尺寸的 PC、手机、手表、冰箱的 Web 浏览器等等）都能显示出令人满意的效果，对 CSS 编写者而言，在实现上不拘泥于具体手法，但通常是糅合了流式布局+弹性布局，再搭配媒体查询技术使用。——分别为不同的屏幕分辨率定义布局，同时，在每个布局中，应用流式布局的理念，即页面元素宽度随着窗口调整而自动适配。即：创建多个流体式布局，分别对应一个屏幕分辨率范围。改变屏幕分辨率可以通过 CSS Media query 实时地切换不同的布局（页面元素位置可能发生改变），在每个布局中，页面元素会随窗口大小的调整发生流式布局中的自动尺寸变化。可以把响应式布局看作是流式布局和自适应布局设计理念的融合。

响应式几乎已经成为优秀页面布局的标准。

### 布局特点

每个屏幕分辨率下面会有一个布局样式，即元素位置和大小都会变。

### 设计方法

媒体查询+流式布局。通常使用 @media 媒体查询和网格系统 (Grid System) 配合相对布局单位进行布局，实际上就是综合响应式、流动等上述技术通过 CSS 给单一网页不同设备返回不同样式的技术统称。

### 实现方式

- 流式布局
- CSS media query

### 优缺点

优点：适应 pc 和移动端，如果足够耐心，效果完美。融合了流式布局和自适应布局的优势。

缺点：

- 媒体查询是有限的，也就是可以枚举出来的，只能适应主流的宽高。
- 要匹配足够多的屏幕大小，工作量不小，设计也需要多个版本。
- CSS 代码繁琐，对于特定的设备有较多冗余，适用于对于各个终端（特别是移动端）性能要求不高的 Blog Dos 站点。

响应式页面在头部会加上这一段代码：

```html
<meta name="applicable-device" content="pc,mobile" />
<meta http-equiv="Cache-Control" content="no-transform " />
```

### 例子

[elevenbeans.github.io/](https://link.juejin.im/?target=http%3A%2F%2Felevenbeans.github.io%2F)

### 响应式设计（RWD）

响应式设计基于响应式布局，**使用同一套页面在各种各样不同大小的设备上进行大小合适、布局（甚至功能）合理的展现。**

响应式设计会根据识别屏幕宽度对于展示的具体内容块进行位置调整，甚至展示和隐藏。

实现方式：

- 响应式布局
- 特性检测 （用于网页功能的渐进增强）

> 举个栗子：[elevenbeans.github.io/，](https://link.juejin.im/?target=http%3A%2F%2Felevenbeans.github.io%2F，)

### RWD 和 AWD 的异同

相同点：

- 均针对不同的分辨率/device 采用不同的样式和布局达到页面展示最优
- 布局方式本质没有差别（AWD 也 including responsive layout）

不同点：

- 前者强调同一套页面多端兼容展示，而后者给出多套页面，对于不同 device 进行了分类处理
- 前者是通过 CSS Media query 进行分辨率检测，可以实时的响应浏览器尺寸变化，改变元素尺寸/布局，而后者一般是 server side detection，一次性渲染既定布局和样式

### 媒体查询用法

- #### 开始在 html 中写入 Media

在 html 头部添加以下代码，用来显示兼容移动设备的显示效果

```html
<meta
  name="viewport"
  content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no"
/>
```

参数详解：

`width=device-width` ：宽度等于当前设备的宽度

`initial-scale=1` ：初始的缩放比例。（默认为 1）

`minimum-scale=1` ：允许用户缩放到的最小比例。（默认为 1）

`maximum-scale=1` ：允许用户缩放到的最大比例。（默认为 1）

`user-scalable=no` ：用户是否可以手动缩放（默认为 no）

- #### 引入包含 Media 的 css 文件

  **我们在媒体查询外面写的第一条规则，是“基本的”样式，它适用于任何设备。在此基础上，我们再为不同视口、不同能力的设备，渐进增加不同的视觉效果和功能。**(**IE6、7、8 不支持媒体查询，也为了防止手机端的某些浏览器不支持媒体查询，所以不要把所有的选择器都放在媒体查询里面。**)

  ```css
  body {
    background-color: grey;
  }
  @media screen and (min-width: 1200px) {
    body {
      background-color: pink;
    }
  }
  @media screen and (min-width: 700px) and (max-width: 1200px) {
    body {
      background-color: blue;
    }
  }
  @media screen and (max-width: 700px) {
    body {
      background-color: orange;
    }
  }
  ```

1. 媒体类型

   | 值         | 描述                                                                  |
   | :--------- | :-------------------------------------------------------------------- |
   | all        | 用于所有设备                                                          |
   | aural      | 已废弃。用于语音和声音合成器                                          |
   | braille    | 已废弃。 应用于盲文触摸式反馈设备                                     |
   | embossed   | 已废弃。 用于打印的盲人印刷设备                                       |
   | handheld   | 已废弃。 用于掌上设备或更小的装置，如 PDA 和小型电话                  |
   | print      | 用于打印机和打印预览                                                  |
   | projection | 已废弃。 用于投影设备                                                 |
   | screen     | 用于电脑屏幕，平板电脑，智能手机等。                                  |
   | speech     | 应用于屏幕阅读器等发声设备                                            |
   | tty        | 已废弃。 用于固定的字符网格，如电报、终端设备和对字符有限制的便携设备 |
   | tv         | 已废弃。 用于电视和网络电视                                           |

2. 逻辑操作符

   - `and`操作符用来把多个[媒体属性](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Media_queries#Media_features)组合成一条媒体查询，对成链式的特征进行请求，只有当每个属性都为真时，结果才为真。
   - `not`操作符用来对一条媒体查询的结果进行取反。
   - `only`操作符仅在媒体查询匹配成功的情况下被用于应用一个样式，这对于防止让选中的样式在老式浏览器中被应用到。
   - 也可以将多个媒体查询以逗号分隔放在一起；只要其中任何一个为真，整个媒体语句就返回真。相当于`or`操作符。
   - 若使用了`not`或`only`操作符，必须明确指定一个媒体类型。

3. 媒体功能

   | 值                  | 描述                                                                             | 值                      | 媒体           | 是否接受 min/max 前缀 |
   | :------------------ | :------------------------------------------------------------------------------- | ----------------------- | -------------- | --------------------- |
   | aspect-ratio        | 定义输出设备中的页面可见区域宽度与高度的比率                                     | <ratio>                 | visual/tactile | 是                    |
   | color               | 定义输出设备每一组彩色原件的个数。如果不是彩色设备，则值等于 0                   | <integer>               | visual         | 是                    |
   | color-index         | 定义在输出设备的彩色查询表中的条目数。如果没有使用彩色查询表，则值等于 0         | <integer>               | visual         | 是                    |
   | device-aspect-ratio | 定义输出设备的屏幕可见宽度与高度的比率。                                         | <ratio>                 | visual/tactile | 是                    |
   | device-height       | 定义输出设备的屏幕可见高度。                                                     | <length>                | visual/tactile | 是                    |
   | device-width        | 定义输出设备的屏幕可见宽度。                                                     | <length>                | visual/tactile | 是                    |
   | grid                | 用来查询输出设备是否使用栅格或点阵。                                             | <integer>               | all            | 否                    |
   | height              | 定义输出设备中的页面可见区域高度。                                               | <length>                | visual/tactile | 是                    |
   | monochrome          | 定义在一个单色框架缓冲区中每像素包含的单色原件个数。如果不是单色设备，则值等于 0 | <integer>               | visual         | 是                    |
   | orientation         | 定义输出设备中的页面可见区域高度是否大于或等于宽度。                             | landscape`|`portrait    | visual         | 否                    |
   | resolution          | 定义设备的分辨率。如：96dpi, 300dpi, 118dpcm                                     | <resolution>            | bitmap         | 是                    |
   | scan                | 定义电视类设备的扫描工序。                                                       | progressive`|`interlace | tv             | 否                    |
   | width               | 定义输出设备中的页面可见区域宽度。                                               | <length>                | visual/tactile | 是                    |

## 5.弹性布局（flex）

Flex 是 Flexible Box 的缩写，意为"弹性布局"，用来为盒状模型提供最大的灵活性。引入弹性盒布局模型的目的是提供一种更加有效的方式来对一个容器中的子元素进行排列、对齐和分配空白空间（他会根据页面的剩余宽度自动分配空间）。

采用 Flex 布局的元素，称为 Flex 容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为 Flex 项目（flex item），简称"项目"。

容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做`main start`，结束位置叫做`main end`；交叉轴的开始位置叫做`cross start`，结束位置叫做`cross end`。

项目默认沿主轴排列。单个项目占据的主轴空间叫做`main size`，占据的交叉轴空间叫做`cross size`。

### 容器的属性

**基本语法：**

```css
.box {
  display: flex; /* 或者 inline-flex */
}
```

上述写法，定义了一个 flex 容器，根据设值的不同可以是块状容器或内联容器。这使得直接子结点拥有了一个 flex 上下文。

- flex-direction
- flex-wrap
- flex-flow
- justify-content
- align-items
- align-content

#### 1.flex-direction

属性决定主轴的方向（即项目的排列方向）。

- `row`（默认值）：主轴为水平方向，起点在左端。
- `row-reverse`：主轴为水平方向，起点在右端。
- `column`：主轴为垂直方向，起点在上沿。
- `column-reverse`：主轴为垂直方向，起点在下沿。

#### 2.flex-wrap

默认情况下，项目都排在一条线（又称"轴线"）上。`flex-wrap`属性定义，如果一条轴线排不下，如何换行。

- `nowrap`（默认）：不换行。
- `wrap`：换行，第一行在上方。
- `wrap-reverse`：换行，第一行在下方。

#### 3.flex-flow

`flex-flow`属性是`flex-direction`属性和`flex-wrap`属性的简写形式，默认值为`row nowrap`。

```css
.box {
  flex-flow: <flex-direction> || <flex-wrap>;
}
```

#### 4.justify-content

定义了项目在主轴上的对齐方式。

- `flex-start`（默认值）：左对齐
- `flex-end`：右对齐
- `center`： 居中
- `space-between`：两端对齐，项目之间的间隔都相等。
- `space-around`：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iyhrshycj30hp0l7dg9.jpg)

#### 5.align-items

定义项目在交叉轴上如何对齐。

- `flex-start`：交叉轴的起点对齐。
- `flex-end`：交叉轴的终点对齐。
- `center`：交叉轴的中点对齐。
- `baseline`: 项目的第一行文字的基线对齐。
- `stretch`（默认值）：如果项目未设置高度或设为 auto，将占满整个容器的高度。
  ![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iyikbcvij30h50lujrx.jpg)

#### 6.align-content

定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。

- `flex-start`：与交叉轴的起点对齐。
- `flex-end`：与交叉轴的终点对齐。
- `center`：与交叉轴的中点对齐。
- `space-between`：与交叉轴两端对齐，轴线之间的间隔平均分布。
- `space-around`：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
- `stretch`（默认值）：轴线占满整个交叉轴。
  ![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iyj94lshj30h80lumxy.jpg)

### 项目的属性

- `order`
- `flex-grow`
- `flex-shrink`
- `flex-basis`
- `flex`
- `align-self`

#### 1.order

定义项目的排列顺序。数值越小，排列越靠前，默认为 0。

```css
.item {
  order: <integer>;
}
```

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iyk0vxo7j30kv0dct8w.jpg)

#### 2.flex-grow

定义项目的放大比例，默认为`0`，即如果存在剩余空间，也不放大。

如果所有项目的`flex-grow`属性都为 1，则它们将等分剩余空间（如果有的话）。如果一个项目的`flex-grow`属性为 2，其他项目都为 1，则前者占据的剩余空间将比其他项多一倍。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iylyev7sj30ma05v0sq.jpg)

#### 3.flex-shrink

定义了项目的缩小比例，默认为 1，即如果空间不足，该项目将缩小。

如果所有项目的`flex-shrink`属性都为 1，当空间不足时，都将等比例缩小。如果一个项目的`flex-shrink`属性为 0，其他项目都为 1，则空间不足时，前者不缩小。

负值对该属性无效。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iymkslb9j30jg041glp.jpg)

#### 4.flex-basis

定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为`auto`，即项目的本来大小。

它可以设为跟`width`或`height`属性一样的值（比如 350px），则项目将占据固定空间。

#### 5.flex

`flex`属性是`flex-grow`, `flex-shrink` 和 `flex-basis`的简写，默认值为`0 1 auto`。后两个属性可选。

该属性有两个快捷值：`auto` (`1 1 auto`) 和 none (`0 0 auto`)。

建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值。

#### 6.align-self

允许单个项目有与其他项目不一样的对齐方式，可覆盖`align-items`属性。默认值为`auto`，表示继承父元素的`align-items`属性，如果没有父元素，则等同于`stretch`。

该属性可能取 6 个值，除了 auto，其他都与 align-items 属性完全一致。

```css
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iynadp42j30kn0aut8r.jpg)

### 兼容性

| Chrome | Safari | Firefox | Opera | IE  | Android | iOS  |
| ------ | ------ | ------- | ----- | --- | ------- | ---- |
| 21+    | 6.1+   | 22+     | 12.1+ | 11+ | 4.4+    | 7.1+ |

Flexbox 需要一些特定的前缀以支持大多数的浏览器。甚至还存在完全不同的属性名称或属性值。这就需要[Autoprefixer](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fpostcss%2Fautoprefixer)或类似的 CSS 后处理器的辅助，具体内容请参考相关文档。

## 6.REM 布局

### rem/em 区别

**rem:当前页面中元素的 REM 单位的样式值都是针对于 HTML 元素的 font-size 的值进行动态计算的**

**em:表示父元素的字号的倍数。(特例：在 text-indent 属性中，表示文字宽度)**em 单位不仅仅可以用来设置字号，还可以设置任何盒模型的属性，比如 width、height、padding、margin、border。

rem 作用于非根元素时，相对于根元素字体大小；rem 作用于根元素字体大小时，相对于其出初始字体大小（16px）。

**rem 有一点优势就是可以和媒体查询配合，实现响应式布局：**

使用 em 或 rem 单位进行相对布局，相对%百分比更加灵活，同时可以支持浏览器的字体大小调整和缩放等的正常显示，因为 em 是相对父级元素的原因没有得到推广。【中国站点制作网页的时候，习惯用 CSS 强制定义字体大小，保证每个人都看到一致的效果，包括网易、搜狐这些门户网站在内的大部分站点，用的都是绝对单位 px（像素）。但是，如果从网站**易用性**方面考虑，字体大小应该是可变的，一些视力不是那么好的人需要放大字体才能看得清页面内容。然而，占据大部分浏览器市场的 IE 无法调整那些使用 px 作为单位的字体大小。国外人士非常重视网站的易用性，相当一部分外国站点已经使用 em 作为字体单位。

### 布局特点

**包裹文字的各元素的尺寸采用 em/rem 做单位，而页面的主要划分区域的尺寸仍使用百分数或 px 做单位（同「流式布局」或「静态/固定布局」）**。**早期浏览器不支持整个页面按比例缩放**，仅支持网页内文字尺寸的放大，这种情况下。使用 em/rem 做单位，可以使包裹文字的元素随着文字的缩放而缩放。

浏览器的默认字体高度一般为`16px`，即 1em:16px，但是 1:16 的比例不方便计算，为了使单位 em/rem 更直观，CSS 编写者常常将页面跟节点字体设为 62.5%，比如选择用 rem 控制字体时，先需要设置根节点 html 的字体大小，因为浏览器默认字体大小 16px\*62.5%=10px。这样 1rem 便是 10px，方便了计算。

### 设计思想

1. 一般不要给元素设置具体的宽度,但是对于一些小图标可以设定具体宽度值
2. 高度值可以设置固定值,设计稿有多大,我们就严格写多大
3. 所有设置的固定值都用 REM 做单位(首先在 HTML 中设置一个基准值：PX 和 REM 的对应比例,然后在效果图上获取 PX 值,布局的时候转化为 REM 值)
4. JS 获取真实屏幕的宽度,让其除以设计稿的宽度,算出比例,把之前的基准值按照比例进行重新的设定,这样项目就可以在移动端自适应了

### 优点

更能适应缩进/以字体单位 padding 或 margin／浏览器设置字体尺寸等情况（因为 em/rem 相对于字体大小，会同步改变）。例如：p{ text-indent: 2em; }。

```css
p {
  text-indent: 2em;
}
```

rem 单位对于（根据屏幕尺寸）调整页面的各元素的尺寸、文字大小时比较好用

### Rem 布局的 js 实现

```javascript
// px转rem，方便模拟小程序 rpx
px2rem($px) {
  $px / 750 * 10 * 1rem;
}
```

```javascript
if remlayout
      script.
        (function flexible (window, document) {
          var docEl = document.documentElement
          var dpr = window.devicePixelRatio || 1
          function setBodyFontSize () {
            if (document.body) {
              document.body.style.fontSize = (12 * dpr) + 'px'
            }
            else {
              document.addEventListener('DOMContentLoaded', setBodyFontSize)
            }
          }
          setBodyFontSize()
          function setRemUnit () {
            var rem = docEl.clientWidth / 10
            docEl.style.fontSize = rem + 'px'
          }
          setRemUnit()
          window.addEventListener('resize', setRemUnit)
          window.addEventListener('pageshow', function (e) {
            if (e.persisted) { setRemUnit() }
          })

          if (dpr >= 2) {
            var fakeBody = document.createElement('body')
            var testElement = document.createElement('div')
            testElement.style.border = '.5px solid transparent'
            fakeBody.appendChild(testElement)
            docEl.appendChild(fakeBody)
            if (testElement.offsetHeight === 1) {
              docEl.classList.add('hairlines')
            }
            docEl.removeChild(fakeBody)
          }
        }(window, document))
```

### 对比三种方式（响应式&&REM&&viewport）

#### 响应式的优缺点

优点：兼容性好，@media 在 ie9 以上是支持的，PC 和 MOBILE 是同一套代码的，不用分开。

缺点：要写得 css 相对另外两个多很多，而且各个断点都要做好。css 样式会稍微大点，更麻烦。

#### REM 优缺点

优点：能维持能整体的布局效果，移动端兼容性好，不用写多个 css 代码，而且还可以利用@media 进行优化。

缺点：开头要引入一段 js 代码，单位都要改成 rem(font-size 可以用 px)，计算 rem 比较麻烦(可以引用预处理器，但是增加了编译过程，相对麻烦了点)。pc 和 mobile 要分开。

#### 设置 viewport 中的 width

```html
<meta name="viewport" content="width=750" />
```

优点：和 REM 相同，而且不用写 rem，直接使用 px，更加快捷。

缺点：效果可能没 rem 的好，图片可能会相对模糊，而且无法使用@media 进行断点，不同 size 的手机上显示，高度间距可能会相差很大。

## 7.Grid 布局（BOOTSTRAP 布局）

网格布局（Grid）是最强大的 CSS 布局方案。

它将网页划分成一个个网格，可以任意组合不同的网格，做出各种各样的布局。以前，只能通过复杂的 CSS 框架达到的效果，现在浏览器内置了。

Flex 布局是轴线布局，只能指定"项目"针对轴线的位置，可以看作是**一维布局**。Grid 布局则是将容器划分成"行"和"列"，产生单元格，然后指定"项目所在"的单元格，可以看作是**二维布局**。Grid 布局远比 Flex 布局强大。

采用网格布局的区域，称为"容器"（container）。容器内部采用网格定位的子元素，称为"项目"（item）。容器里面的水平区域称为"行"（row），垂直区域为"列"（column）。行和列的交叉区域，称为"单元格"（cell）。划分网格的线，称为"网格线"（grid line）。水平网格线划分出行，垂直网格线划分出列。

### 容器的属性

#### 1.display

`display: grid`指定一个容器采用网格布局。

默认情况下，容器元素都是块级元素，但也可以设成行内元素。`display: inline-grid;`

注意，设为网格布局以后，容器子元素（项目）的`float`、`display: inline-block`、`display: table-cell`、`vertical-align`和`column-*`等设置都将失效。

#### 2.grid-template-columns 属性， grid-template-rows 属性

容器指定了网格布局以后，接着就要划分行和列。`grid-template-columns`属性定义每一列的列宽，`grid-template-rows`属性定义每一行的行高。

```css
.container {
  display: grid;
  grid-template-columns: 100px 100px 100px;
  grid-template-rows: 100px 100px 100px;
}
```

除了使用绝对单位，也可以使用百分比

- repeat()：接受两个参数，第一个参数是重复的次数（上例是 3），第二个参数是所要重复的值。重复某种模式也是可以的。

  ```css
  grid-template-columns: repeat(2, 100px 20px 80px);
  ```

- auto-fill 关键字：有时，单元格的大小是固定的，但是容器的大小不确定。如果希望每一行（或每一列）容纳尽可能多的单元格，这时可以使用`auto-fill`关键字表示自动填充。

  ```css
  .container {
    display: grid;
    grid-template-columns: repeat(auto-fill, 100px);
  }
  ```

- fr 关键字：为了方便表示比例关系，网格布局提供了`fr`关键字（fraction 的缩写，意为"片段"）。如果两列的宽度分别为`1fr`和`2fr`，就表示后者是前者的两倍。（`fr`可以与绝对长度的单位结合使用）

- minmax()：`minmax()`函数产生一个长度范围，表示长度就在这个范围之中。它接受两个参数，分别为最小值和最大值。

  ```css
  grid-template-columns: 1fr 1fr minmax(100px, 1fr);
  ```

  上面代码中，`minmax(100px, 1fr)`表示列宽不小于`100px`，不大于`1fr`。

- auto 关键字：表示由浏览器自己决定长度

- 网格线的名称：`grid-template-columns`属性和`grid-template-rows`属性里面，还可以使用方括号，指定每一根网格线的名字，方便以后的引用。（网格布局允许同一根线有多个名字，比如`[fifth-line row-5]`。）

  ```css
  .container {
    display: grid;
    grid-template-columns: [c1] 100px [c2] 100px [c3] auto [c4];
    grid-template-rows: [r1] 100px [r2] 100px [r3] auto [r4];
  }
  ```

#### 3.grid-row-gap 属性， grid-column-gap 属性， grid-gap 属性

- `grid-row-gap`属性设置行与行的间隔（行间距）
- `grid-column-gap`属性设置列与列的间隔（列间距）。
- `grid-gap`属性是`grid-column-gap`和`grid-row-gap`的合并简写形式：`grid-gap: <grid-row-gap> <grid-column-gap>;`(如果`grid-gap`省略了第二个值，浏览器认为第二个值等于第一个值。)

1.  根据最新标准，上面三个属性名的`grid-`前缀已经删除，`grid-column-gap`和`grid-row-gap`写成`column-gap`和`row-gap`，`grid-gap`写成`gap`。

#### 4.grid-template-areas 属性

网格布局允许指定"区域"（area），一个区域由单个或多个单元格组成。

```css
grid-template-areas:
  "header header header"
  "main main sidebar"
  "footer footer footer";
```

区域的命名会影响到网格线。每个区域的起始网格线，会自动命名为`区域名-start`，终止网格线自动命名为`区域名-end`。

#### 5.grid-auto-flow 属性

划分网格以后，容器的子元素会按照顺序，自动放置在每一个网格。默认的放置顺序是"先行后列"，即先填满第一行，再开始放入第二行，即下图数字的顺序。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iyo19otvj30au0b6glp.jpg)

- 这个顺序由`grid-auto-flow`属性决定，默认值是`row`，即"先行后列"。
- 也可以将它设成`column`，变成"先列后行"。
- 设为`row dense`，表示"先行后列"，并且尽可能紧密填满，尽量不出现空格。
- `column dense`，表示"先列后行"，并且尽量填满空格。

#### 6.justify-items 属性， align-items 属性， place-items 属性

- `justify-items`属性设置单元格内容的水平位置（左中右）

- `align-items`属性设置单元格内容的垂直位置（上中下）。

  这两个属性的写法完全相同，都可以取下面这些值。

  - start：对齐单元格的起始边缘。
  - end：对齐单元格的结束边缘。
  - center：单元格内部居中。
  - stretch：拉伸，占满单元格的整个宽度（默认值）。

- `place-items`属性是`align-items`属性和`justify-items`属性的合并简写形式。（如果省略第二个值，则浏览器认为与第一个值相等。）

  ```css
  place-items: <align-items> <justify-items>;
  ```

#### 7.justify-content 属性， align-content 属性， place-content 属性

- `justify-content`属性是整个内容区域在容器里面的水平位置（左中右）

- `align-content`属性是整个内容区域的垂直位置（上中下）。

  这两个属性的写法完全相同，都可以取下面这些值。

  只是将水平方向改成垂直方向。）

  - start - 对齐容器的起始边框。
  - end - 对齐容器的结束边框。
  - center - 容器内部居中。
  - stretch - 项目大小没有指定时，拉伸占据整个网格容器。
  - space-around - 每个项目两侧的间隔相等。所以，项目之间的间隔比项目与容器边框的间隔大一倍。
  - space-between - 项目与项目的间隔相等，项目与容器边框之间没有间隔。
  - space-evenly - 项目与项目的间隔相等，项目与容器边框之间也是同样长度的间隔。

- `place-content`属性是`align-content`属性和`justify-content`属性的合并简写形式。（如果省略第二个值，浏览器就会假定第二个值等于第一个值。）

  ```css
  place-content: <align-content> <justify-content>;
  ```

#### 8.grid-auto-columns 属性， grid-auto-rows 属性

有时候，一些项目的指定位置，在现有网格的外部。比如网格只有 3 列，但是某一个项目指定在第 5 行。这时，浏览器会自动生成多余的网格，以便放置项目。

`grid-auto-columns`属性和`grid-auto-rows`属性用来设置，浏览器自动创建的多余网格的列宽和行高。它们的写法与`grid-template-columns`和`grid-template-rows`完全相同。如果不指定这两个属性，浏览器完全根据单元格内容的大小，决定新增网格的列宽和行高。

划分好的网格是 3 行 x 3 列，但是，8 号项目指定在第 4 行，9 号项目指定在第 5 行。

```css
.container {
  display: grid;
  grid-template-columns: 100px 100px 100px;
  grid-template-rows: 100px 100px 100px;
  grid-auto-rows: 50px;
}
```

上面代码指定新增的行高统一为 50px（原始的行高为 100px）。

#### 9.grid-template 属性， grid 属性

`grid-template`属性是`grid-template-columns`、`grid-template-rows`和`grid-template-areas`这三个属性的合并简写形式。

`grid`属性是`grid-template-rows`、`grid-template-columns`、`grid-template-areas`、 `grid-auto-rows`、`grid-auto-columns`、`grid-auto-flow`这六个属性的合并简写形式。

### 项目属性

#### 1.grid-column-start 属性， grid-column-end 属性， grid-row-start 属性， grid-row-end 属性

项目的位置是可以指定的，具体方法就是指定项目的四个边框，分别定位在哪根网格线。

- `grid-column-start`属性：左边框所在的垂直网格线
- `grid-column-end`属性：右边框所在的垂直网格线
- `grid-row-start`属性：上边框所在的水平网格线
- `grid-row-end`属性：下边框所在的水平网格线

```css
.item-1 {
  grid-column-start: 2;
  grid-column-end: 4;
}
```

这四个属性的值还可以使用`span`关键字，表示"跨越"，即左右边框（上下边框）之间跨越多少个网格。

使用这四个属性，如果产生了项目的重叠，则使用`z-index`属性指定项目的重叠顺序。

#### 2.grid-column 属性， grid-row 属性

- `grid-column`属性是`grid-column-start`和`grid-column-end`的合并简写形式

- `grid-row`属性是`grid-row-start`属性和`grid-row-end`的合并简写形式。

  这两个属性之中，也可以使用`span`关键字，表示跨越多少个网格。斜杠以及后面的部分可以省略，默认跨越一个网格。

  ```css
  .item-1 {
    background: #b03532;
    grid-column: 1 / 3;
    grid-row: 1 / 3;
  }
  /* 等同于 */
  .item-1 {
    background: #b03532;
    grid-column: 1 / span 2;
    grid-row: 1 / span 2;
  }
  ```

#### 3.grid-area 属性

`grid-area`属性指定项目放在哪一个区域。

```css
.item-1 {
  grid-area: e;
}
```

`grid-area`属性还可用作`grid-row-start`、`grid-column-start`、`grid-row-end`、`grid-column-end`的合并简写形式，直接指定项目的位置。

```css
.item {
  grid-area: <row-start> / <column-start> / <row-end> / <column-end>;
}
```

#### 4.justify-self 属性， align-self 属性， place-self 属性

- `justify-self`属性设置单元格内容的水平位置（左中右），跟`justify-items`属性的用法完全一致，但只作用于单个项目。

- `align-self`属性设置单元格内容的垂直位置（上中下），跟`align-items`属性的用法完全一致，也是只作用于单个项目。

  这两个属性都可以取下面四个值。

  - start：对齐单元格的起始边缘。
  - end：对齐单元格的结束边缘。
  - center：单元格内部居中。
  - stretch：拉伸，占满单元格的整个宽度（默认值）。

- `place-self`属性是`align-self`属性和`justify-self`属性的合并简写形式。

  ```css
  place-self: <align-self> <justify-self>;
  ```

### 兼容性

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iyqd0osoj326m0rok1w.jpg)

### 对比 Bootstrap

- 标签会更加简洁：相比`Bootstrap`，使用 grid 会使你的 HTML 更加干净。`Bootstrap`需要创建的标签，每个 row 都需要一个`<div>`标签，使用了 class name 来指定布局(`col-xs-2`)。grid 用来布局看起来更简单，丑陋的类名和每行所需的额外的 div 标签一去不复返了，简简单单一个 container 和里面的 item。与`Bootstrap`不同的是，随着布局复杂度的增加，Grid 布局标签的复杂度将不会增加太多。
- 更灵活：用`CSS Grid`的话会非常简单，我们只需要添加一个`media query`就可以重新排列布局。而如果想在`Bootstrap`中做同样的事情，就必须得修改 HTML 了，需要调整标签的顺序。
- 不再限死 12 列：`Bootstrap`的 grid 系统分为了 12 列，如果你想要一个 5 列的布局就会纠结，或是 7 列、9 列、任何不会合为 12 列的。`CSS Grid`就没有任何限制，你可以让 grid 正好有你想要的数量。
- 浏览器支持：全球 75%的网站流量支持`CSS Grid`

## 结论：

1. 如果只做 pc 端，那么静态布局（定宽度）是最好的选择；
2. 如果做移动端，且设计对高度和元素间距要求不高，那么弹性布局（rem+js）是最好的选择，一份 css+一份 js 调节 font-size 搞定；
3. 如果 pc，移动要兼容，而且要求很高那么响应式布局还是最好的选择，前提是设计根据不同的高宽做不同的设计，响应式根据媒体查询做不同的布局.
