---
title: JS事件机制
date: 2019-07-24 15:22:00
tags:
  - javascript
categories: javascript
---

# JS 事件机制

## js 里一般的阻止事件默认行为怎么做？

event.preventDefault()可以取消默认事件

event.stopPropagation()起到阻止捕获和冒泡阶段中当前事件的进一步传播。

<!-- more -->

```javascript
function cancelHandler(event) {
  var event = event || window.event; //兼容IE
  //取消事件相关的默认行为
  if (event.preventDefault)
    //标准技术
    event.preventDefault();
  if (event.returnValue)
    //兼容IE9之前的IE
    event.returnValue = false;
  return false; //用于处理使用对象属性注册的处理程序
}
```

## 冒泡的机制是什么？下面代码输出顺序是什么？（2->button->1）

- 触发顺序：button->1->2

```html
<div><button>aaa</button></div>

<script>
  document.getElementById("div").addEventListener(
    "click",
    function() {
      console.log("1");
    },
    false
  );
  document.getElementById("div").addEventListener(
    "click",
    function() {
      console.log("2");
    },
    false
  );
  document.getElementById("button").addEventListener(
    "click",
    function() {
      console.log("button");
    },
    false
  );
</script>
```

- 触发顺序：2->button->1

```html
<div><button>aaa</button></div>

<script>
  document.getElementById("div").addEventListener(
    "click",
    function() {
      console.log("1");
    },
    false
  );
  document.getElementById("div").addEventListener(
    "click",
    function() {
      console.log("2");
    },
    true
  );
  document.getElementById("button").addEventListener(
    "click",
    function() {
      console.log("button");
    },
    false
  );
</script>
```

- 触发顺序：2->button->1

```html
<div><button>aaa</button></div>

<script>
  document.getElementById("div").addEventListener(
    "click",
    function() {
      console.log("1");
    },
    false
  );
  document.getElementById("div").addEventListener(
    "click",
    function() {
      console.log("2");
    },
    true
  );
  document.getElementById("button").addEventListener(
    "click",
    function() {
      console.log("button");
    },
    true
  );
</script>
```
