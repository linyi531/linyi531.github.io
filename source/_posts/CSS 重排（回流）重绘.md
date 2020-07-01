---
title: CSS 重排（回流）重绘
date: 2020-05-03 20:03:26
tags:
  - CSS
  - 布局
categories: CSS
cover_img: https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbq8jym55j31900u0qv6.jpg
feature_img: https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbq8jym55j31900u0qv6.jpg
---

# 重排重绘

## 重排重绘的影响范围是基于什么的

- 重绘：某些元素的外观被改变，例如：元素的填充颜色

  当一个元素的外观发生改变，但没有改变布局,重新把元素外观绘制出来的过程，叫做重绘。

- 重排（回流）：重新生成布局，重新排列元素。

  当DOM的变化影响了元素的几何信息(DOM对象的位置和尺寸大小)，浏览器需要重新计算元素的几何属性，将其安放在界面中的正确位置，这个过程叫做重排。

  由于浏览器渲染界面是基于流失布局模型的，所以触发重排时会对周围DOM重新排列，影响的范围有两种：

  - 全局范围：从根节点`html`开始对整个渲染树进行重新布局。
  - 局部范围：对渲染树的某部分或某一个渲染对象进行重新布局

### 1）常见引起回流属性和方法

任何会改变元素几何信息(元素的位置和尺寸大小)的操作，都会触发回流，

- 添加或者删除可见的DOM元素；
- 元素尺寸改变——边距、填充、边框、宽度和高度
- 内容变化，比如用户在input框中输入文字
- 浏览器窗口尺寸改变——resize事件发生时
- 计算 offsetWidth 和 offsetHeight 属性
- 设置 style 属性的值

### 2）常见引起重绘属性和方法

![image-20191017183756722](https://tva1.sinaimg.cn/large/006y8mN6ly1g81dslpyruj30w40eegnb.jpg)

### 3）如何减少回流、重绘

- 使用 transform 替代 top
- 使用 visibility 替换 display: none ，因为前者只会引起重绘，后者会引发回流（改变了布局）
- 不要把节点的属性值放在一个循环里当成循环里的变量。

```javascript
for(let i = 0; i < 1000; i++) {
    // 获取 offsetTop 会导致回流，因为需要去获取正确的值
    console.log(document.querySelector('.test').style.offsetTop)
}
```

- 不要使用 table 布局，可能很小的一个小改动会造成整个 table 的重新布局
- 动画实现的速度的选择，动画速度越快，回流次数越多，也可以选择使用 requestAnimationFrame
- CSS 选择符从右往左匹配查找，避免节点层级过多
- 将频繁重绘或者回流的节点设置为图层，图层能够阻止该节点的渲染行为影响别的节点。比如对于 video 标签来说，浏览器会自动将该节点变为图层。

