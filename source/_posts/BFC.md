---
title: BFC
date: 2019-04-10 08:11:20
tags:
  - CSS
  - 布局
categories: CSS
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g904fe6mqoj31900u0qvp.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g904fe6mqoj31900u0qvp.jpg
---

# BFC

## BFC 的定义：

BFC(Block formatting context)直译为"块级格式化上下文"。它**是一个独立的渲染区域**，只有**Block-level box**参与（在下面有解释）， 它规定了内部的 Block-level Box 如何布局，并且与这个区域外部毫不相干。**通俗地讲，BFC 是一个容器，用于管理块级元素。**

 <!-- more -->

## 触发 BFC 的方式（以下任意一条就可以）

1. 根元素，即 HTML 元素
2. float 的值不为 none（为 `left`或`right`）
3. overflow 的值不为 visible（为`hidden`或`auto`或`scroll`）
4. display 的值为`table-cell`、`table-caption`、`inline-flex`、`flex`和`inline-block`之一
5. position 的值不为 static 或者 releative 中任何一个(为`absolute`或`fixed`)

## BFC 的布局规则

1. 内部的 Box 会在垂直方向，一个接一个地放置。
2. Box 垂直方向的距离由 margin 决定。属于同一个 BFC 的两个相邻 Box 的 margin 会发生重叠（**margin 重叠三个条件:同属于一个 BFC;相邻;块级元素**），两个相邻的 BFC 上下 margin 不会重叠
3. 每个元素的 margin box 的左边， 与包含块 border box 的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。
4. BFC 的区域不会与 float box 重叠。非浮动元素不会覆盖浮动元素的位置。
5. BFC 就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。（触发 BFC，回流的局部渲染）
6. 计算 BFC 的高度时，浮动元素也参与计算（清除浮动 haslayout）
7. margin 不会传递给父级（父级触发了 BFC）

### 对比普通文档流的布局规则

1. 浮动的元素是不会被父级计算高度
2. 非浮动元素会覆盖浮动元素的位置
3. margin 会传递给父级
4. 两个相邻的元素上下 margin 会重叠

## BFC 有哪些作用：

1. 自适应两栏布局（规则 4）
2. 可以阻止元素被浮动元素覆盖（规则 4）
3. 可以包含浮动元素——清除内部浮动（规则 6）
4. 分属于不同的 BFC 时可以阻止 margin 重叠（规则 2）
