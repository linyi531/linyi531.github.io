---
title: React组件
date: 2018-11-11 12:10:52
tags:
  - React
categories: React
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77ho2ktr0j30u011i1l0.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77ho2ktr0j30u011i1l0.jpg
---

# Web Copmpoents

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j6p2nvl3j314i0oqgm8.jpg)

# React 组件

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j6p1c7t9j31am0gejro.jpg)

React 组件：

1. 自定义元素是库自己构建的
2. 渲染过程包含了模版的概念
3. 实现均在方法与类中，相互隔离（不包括样式）
4. 引用方式遵循 ES6

构建：

1. React 方式：creatClass
2. ES6 方式：class
3. 无状态函数

# React 底层——合成事件

## 事件委派

把事件处理函数绑定到结构的最外层，使用一个统一的事件监听器。（不会把事件处理函数直接绑定到真实的节点上）

<!-- more -->

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j6p4jv10j31b20tmjse.jpg)

## 自动绑定

每个方法的上下文都会指向该组件的实例——自动绑定 this 为当前组件。React 会对这种引用进行缓存，达到 CPU 内存最优。（使用 ES6 class 或纯函数时，自动绑定不复存在，需要手动绑定 this）

1. bind 绑定
   绑定事件处理器内的 this，并可以向事件处理器中传参

2. 构造器内声明
   好处：仅需进行一次绑定

3. 箭头函数
   箭头函数自动绑定了定义此函数作用域的 this，因此不需要再用 bind 绑定

注意：React 中使用 DOM 原生事件，一定要在组件卸载时手动移除，否则内存泄漏。使用合成事件则不需要。

# React 合成事件与 JS 原生事件对比

原生 DOM 事件传播 3 个阶段：事件捕获阶段、目标对象本身的事件处理程序调用，以及事件冒泡。

1. 事件捕获阶段会优先调用结构树最外层的元素上绑定的事件侦听器，依次向内调用，一直调用到目标元素上的事件监听器为止。

```javascript
e.addEventListener("click", () => {}, false);
```

第三个参数，若传 true，为元素 e 注册捕获事件处理程序，并且在事件捕获阶段调用。

2. 事件冒泡与事件捕获相反，它会从目标元素向外传播，由内而外。
   React 的合成事件仅支持事件冒泡
   阻止原生事件冒泡用 e.preventDefault()

# React 受控组件更新 state 的流程

1. 可以通过在初始 state 中设置表单的默认值
2. 每当表单的值发生变化时，调用 onchange 事件处理器
3. 事件处理器通过合成事件对象 e 拿到改变后的状态，并更行应用的 state
4. setState 触发视图的重新渲染，完成表单的组件值更新

# React 非受控组件

是一种反模式，他的值不受组件自身的 state 或 props 控制。通常需要为其添加 ref prop 来访问渲染后的底层 DOM
