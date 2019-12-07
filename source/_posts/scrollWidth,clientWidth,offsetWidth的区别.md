---
title: scrollWidth,clientWidth,offsetWidth的区别
date: 2019-12-07 22:01:34
tags:
  - CSS
categories: CSS
cover_img: https://tva1.sinaimg.cn/large/006tNbRwgy1g9oibg5zl2j31900u0b29.jpg
feature_img: https://tva1.sinaimg.cn/large/006tNbRwgy1g9oibg5zl2j31900u0b29.jpg
---

# scrollWidth,clientWidth,offsetWidth的区别

## 总体说明

### 元素对象：

- offsetLeft、offsetTop属性：获取元素相对于文档左上角的坐标位置。

* scrollWidth：对象的**实际内容**的宽度，不包括边线宽度，会随对象中内容超过可视区后而变大。

  scrollWidth=元素的width+padding

* clientWidth：对象内容的**可视区的宽度**，不包括滚动条等边线，会随对象显示大小的变化而改变。

  clientWidth=元素的width+padding

* offsetWidth：对象**整体的实际宽度**，包括滚动条等边线，会随对象显示大小的变化而改变。

  offsetWidth=元素的width+padding+border

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g7aq0kktmyj30gx0gqtam.jpg)

###window对象：

* innerWidth：窗口中文档显示区域的宽度，不包括菜单栏、工具栏等部分。该属性可读可写。浏览器窗口的内部宽度（对于IE9+、Chrome、Firefox、Opera 以及 Safari）
* pageXOffset：整数只读属性，表示文档向右滚动过的像素数。IE不支持该属性，使用body元素的scrollLeft属性替代。

* 

##情况一

元素内无内容或者内容不超过可视区，滚动不出现或不可用的情况下。

* scrollWidth=clientWidth，两者皆为内容可视区的宽度。
* offsetWidth为元素的实际宽度。

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g7aoj7ezc9j30eo07hjs6.jpg)

##情况二

元素的内容超过可视区，滚动条出现和可用的情况下。

* scrollWidth>clientWidth。
* scrollWidth为实际内容的宽度。
* clientWidth是内容可视区的宽度。
* offsetWidth是元素的实际宽度。

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g7aokjodirj30h40a1q41.jpg)

## 针对文档(document)的各个height、width、top、left的说明

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g7apu7784cj30dw0fa0tw.jpg)

## 针对网页中一个div的各个属性值的说明

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g7apvi51ygj30jg0ga400.jpg)