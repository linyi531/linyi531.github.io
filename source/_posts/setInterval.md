---
title: setInterval
date: 2020-06-02 08:15:39
tags:
  - JavaScript
categories: JavaScript
cover_img: https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbqjmromqj31ag0u0e81.jpg
feature_img: https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbqjmromqj31ag0u0e81.jpg
---

# setInterval

## setInterval()基础

### 用法

setInterval函数的用法与setTimeout完全一致，区别仅仅在于setInterval指定某个任务每隔一段时间就执行一次，也就是无限次的定时执行。

### 间隔

`setInterval` 指定的是“开始执行”之间的间隔，并不考虑每次任务执行本身所消耗的事件。因此实际上，两次执行之间的间隔会小于指定的时间。比如，`setInterval` 指定每 100ms 执行一次，每次执行需要 5ms，那么第一次执行结束后 95 毫秒，第二次执行就会开始。如果某次执行耗时特别长，比如需要 105 毫秒，那么它结束后，下一次执行就会立即开始。

```javascript
var i = 1;
var timer = setInterval(function() {
  alert(i++);
}, 2000);
```

上面代码每隔2000毫秒，就跳出一个alert对话框。如果用户一直不点击“确定”，整个浏览器就处于“堵塞”状态，后面的执行就一直无法触发，将会累积起来。举例来说，第一次跳出alert对话框后，用户过了6000毫秒才点击“确定”，那么第二次、第三次、第四次执行将累积起来，它们之间不会再有等待间隔。

**HTML 5标准规定，`setInterval` 的最短间隔时间是10毫秒，也就是说，小于10毫秒的时间间隔会被调整到10毫秒。**

## **setInterval运行机制**

**setInterval** 的运行机制是，将指定的代码移出本次执行，等到下一轮Event Loop时，再检查是否到了指定时间。如果到了，就执行对应的代码；如果不到，就等到再下一轮Event Loop时重新判断。这意味着，`setTimeout` 和 `setInterval` 指定的代码，必须等到本次执行的所有代码都执行完，才会执行。

每一轮Event Loop时，都会将“任务队列”中需要执行的任务，一次执行完。setTimeout 和 setInterval 都是把任务添加到“任务队列”的尾部。因此，它们实际上要等到当前脚本的所有同步任务执行完，然后再等到本次 Event Loop 的“任务队列”的所有任务执行完，才会开始执行。由于前面的任务到底需要多少时间执行完，是不确定的，所以没有办法保证 `setTimeout` 和 `setInterval` 指定的任务，一定会按照预定时间执行。

```javascript
setInterval(function(){
  console.log(2);
},1000);
(function (){
  sleeping(3000);
})();
```

上面的第一行语句要求每隔1000毫秒，就输出一个2。但是，第二行语句需要3000毫秒才能完成，请问会发生什么结果？

<u>结果就是等到第二行语句运行完成以后，立刻连续输出三个2，然后开始每隔1000毫秒，输出一个2。也就是说，setIntervel具有**累积效应**，如果某个操作特别耗时，超过了setInterval的时间间隔，排在后面的操作会被累积起来，然后在很短的时间内连续触发，这可能或造成性能问题（比如集中发出Ajax请求）。</u>

**如果执行时间大于预设间隔时间，很可能导致连续执行，中间没有时间间隔，这是很糟糕的，很可能会耗费大量cpu.**

> 如果你在一个大的JavaScript代码块正在执行的时候把所有的interval回调函数都囤起来的话，其结果就是在JavaScript代码块执行完 了之后会有一堆的interval事件被执行，而执行过程中不会有间隔。因此，取代的作法是浏览器情愿先等一等，以确保在一个interval进入队列的 时候队列中没有别的interval。

**setInterval去排队时，如果发现自己还在队列中未执行，则会被drop调，同一个interval，在队列里只会有一个**

> interval是不管当前在执行些什么的，在任何情况下它都会进入到队列中去，即使这样意味着每次回调之间的时间就不准确了。https://www.cnblogs.com/youxin/p/3354924.html

但上一示例，在Nodejs环境下测试，并不会连续输出三个2；理由：setInterval 在准备把回调函数加入到事件队列的时候，会判断队列中是否还有未执行的回调，如果有的话，它就不会再往队列中添加回调函数。

## 弹窗会让 Chrome/Opera/Safari 内的时钟停止

```javascript
// 每 2 秒重复一次
let timerId = setInterval(() => alert('tick'), 2000);

// 5 秒之后停止
setTimeout(() => { clearInterval(timerId); alert('stop'); }, 5000);
```

在众多浏览器中，IE 和 Firefox 在显示 `alert/confirm/prompt` 时，内部的定时器仍旧会继续滴答，但是在 Chrome、Opera 和 Safari 中，内部的定时器会暂停/冻结。

所以，在执行以上代码时，如果在一定时间内没有关掉 `alert` 弹窗，那么在你关闭弹窗后，Firefox/IE 会立即显示下一个 `alert` 弹窗（前提是距离上一次执行超过了 2 秒），而 Chrome/Opera/Safari 这三个则需要再等待 2 秒以上的时间才会再显示（因为在 `alert` 弹窗期间，定时器并没有滴答）。

## 标签处于非活动状态时不同浏览器的表现

**如果网页不在浏览器的当前窗口（或tab），许多浏览器限制 `setInteral` 指定的反复运行的任务最多每秒执行一次。**

### chrome

当标签处于非活动状态时，Chrome会将最小间隔`setInterval`限制为大约1000毫秒。 如果间隔高于1000毫秒，它将以指定的间隔运行。 窗口是否失焦并不重要，只有当您切换到不同的选项卡时才会限制间隔。 

```javascript
// Provides control over the minimum timer interval for background tabs.
const double kBackgroundTabTimerInterval = 1.0;
```

[https://codereview.chromium.org/6546021/patch/1001/2001]

### 火狐

与Chrome类似，当标签页（非窗口）处于非活动状态时，Firefox会将最小间隔`setInterval`限制为大约1000毫秒。 

```javascript
// The default shortest interval/timeout we permit
#define DEFAULT_MIN_TIMEOUT_VALUE 4 // 4ms
#define DEFAULT_MIN_BACKGROUND_TIMEOUT_VALUE 1000 // 1000ms
```

[https://hg.mozilla.org/releases/mozilla-release/file/0bf1cadfb004/dom/base/nsGlobalWindow.cpp#l296]

### IE浏览器

当选项卡处于非活动状态时，IE不会限制`setInterval`中的延迟，窗口是否失焦并不重要。

### Edge

从Edge 14开始，`setInterval`在非活动选项卡中的上限为1000毫秒。

### safari

就像Chrome一样，当标签处于非活动状态时，Safari会在1000毫秒时限制`setInterval`。 

### Opera

自从采用Webkit引擎以来，Opera表现出与Chrome相同的行为。 `setInterval`上限为1000毫秒。

