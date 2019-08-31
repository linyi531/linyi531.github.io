---
title: JS引擎执行机制
date: 2019-07-30 13:13:29
tags:
  - javascript
categories: javascript
cover_img: https://i.screenshot.net/zj17wa6
feature_img: https://i.screenshot.net/zj17wa6
---

# JS 引擎执行机制

**(1) JS 是单线程语言**

**(2) JS 的 Event Loop 是 JS 的执行机制。深入了解 JS 的执行,就等于深入了解 JS 里的 event loop**

<!-- more -->

## JS 为什么是单线程的? 为什么需要异步? 单线程又是如何实现异步的呢?

- JS 最初被设计用在浏览器中,那么想象一下,如果浏览器中的 JS 是多线程的。

  场景描述:

  那么现在有 2 个线程,process1 process2,由于是多线程的 JS,所以他们对同一个 dom,同时进行操作。process1 删除了该 dom,而 process2 编辑了该 dom,同时下达 2 个矛盾的命令,浏览器究竟该如何执行呢?

- JS 为什么需要异步?

  如果 JS 中不存在异步,只能自上而下执行,如果上一行解析时间很长,那么下面的代码就会被阻塞。
  对于用户而言,阻塞就意味着"卡死",这样就导致了很差的用户体验

- JS 单线程又是如何实现异步的呢?

  **是通过的事件循环(event loop),理解了 event loop 机制,就理解了 JS 的执行机制**

## JS 中的 event loop

### event loop（1）

JS 里的一种分类方式,就是将任务分为: 同步任务和异步任务

JS 的执行机制是：

- 首先判断 JS 是同步还是异步,同步就进入主线程,异步就进入 event table
- 异步任务在 event table 中注册函数,当满足触发条件后,被推入 event queue
- 同步任务进入主线程后一直执行,直到主线程空闲时,才会去 event queue 中查看是否有可执行的异步任务,如果有就推入主线程中

### event loop（2）

准确的划分方式是:

- macro-task(宏任务)：包括整体代码 script，setTimeout，setInterval
- micro-task(微任务)：Promise.then，process.nextTick

JS 的执行机制是：

- 执行一个宏任务,过程中如果遇到微任务,就将其放到微任务的【事件队列】里
- 当前宏任务执行完成后,会查看微任务的【事件队列】,并将里面全部的微任务依次执行完

**重复以上 2 步骤,结合 event loop(1) event loop(2) ,就是更为准确的 JS 执行机制了。**

## 理解 JavaScript 的 async/await

### async 和 await 在干什么

先从字面意思来理解。async 是“异步”的简写，而 await 可以认为是 async wait 的简写。所以应该很好理解 async 用于申明一个 function 是异步的，而 await 用于等待一个异步方法执行完成。

- async 函数返回的是一个 **Promise** 对象。
  - 如果在函数中 `return` 一个直接量，async 会把这个直接量通过 `Promise.resolve()` 封装成 Promise 对象。
  - 如果 async 函数没有返回值，又该如何？很容易想到，它会返回 `Promise.resolve(undefined)`。
  - Promise 的特点——无等待，所以在没有 `await` 的情况下执行 async 函数，它会立即执行，返回一个 Promise 对象，并且，绝不会阻塞后面的语句。这和普通返回 Promise 对象的函数并无二致。
- await 等待的是一个表达式。
  - 这个表达式的计算结果是 Promise 对象或者其它值（换句话说，就是没有特殊限定）。
  - await 不仅仅用于等 Promise 对象，它可以等任意表达式的结果，所以，await 后面实际是可以接普通函数调用或者直接量的。
- await 等到了要等的，然后呢？
  - `await` 是个运算符，用于组成表达式，await 表达式的运算结果取决于它等的东西。
  - 如果它等到的不是一个 Promise 对象，那 await 表达式的运算结果就是它等到的东西。
  - 如果它等到的是一个 Promise 对象，await 就忙起来了，它会阻塞后面的代码，等着 Promise 对象 resolve，然后得到 resolve 的值，作为 await 表达式的运算结果。

这就是 await 必须用在 async 函数中的原因。async 函数调用不会造成阻塞，它内部所有的阻塞都被封装在一个 Promise 对象中异步执行。

### await 等待的表达式详解

**await 等的是右侧「表达式」的结果**

await 是从**右向左执行的**

```javascript
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
async function async2() {
  console.log("async2");
}
async1();
console.log("script start");
```

先执行 async2 后，发现有 await 关键字，于是让出线程，阻塞代码

右侧表达式的结果:

- 如果不是 promise , await 会阻塞后面的代码，先执行 async 外面的同步代码，同步代码执行完，再回到 async 内部，把这个非 promise 的东西，作为 await 表达式的结果
- 如果它等到的是一个 promise 对象，await 也会暂停 async 后面的代码，先执行 async 外面的同步代码，等着 Promise 对象 fulfilled，然后把 resolve 的参数作为 await 表达式的运算结果。

### async/await 的优势在于处理 then 链

单一的 Promise 链并不能发现 async/await 的优势，但是，如果需要处理由多个 Promise 组成的 then 链的时候，优势就能体现出来了。

- Async/await 代码清晰很多，几乎跟同步代码一样。
- Promise 方案的死穴—— 链式调用参数传递太麻烦

## async/await 和 promise 的执行顺序

### 例子

```javascript
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
async function async2() {
  console.log("async2");
}
console.log("script start");
setTimeout(function() {
  console.log("setTimeout");
}, 0);
async1();
new Promise(function(resolve) {
  console.log("promise1");
  resolve();
}).then(function() {
  console.log("promise2");
});
console.log("script end");
```

```javascript
script start
async1 start
async2
promise1
script end
promise2
async1 end
setTimeout
```

宏任务和微任务的慨念，在我脑海中宏任务和为微任务如图所示

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6izm5nu3ij30f204qq2z.jpg)

也就是「宏任务」、「微任务」都是队列。

一段代码执行时，会先执行宏任务中的同步代码，

- 如果执行中遇到 setTimeout 之类宏任务，那么就把这个 setTimeout 内部的函数推入「宏任务的队列」中，下一轮宏任务执行时调用。
- 如果执行中遇到 promise.then()之类的微任务，就会推入到「当前宏任务的微任务队列」中，在本轮宏任务的同步代码执行都完成后，依次执行所有的微任务 1、2、3

### 例子分析执行顺序

#### 直接打印同步代码 console.log(‘script start’)

```javascript
// 首先是2个函数声明，虽然有async关键字，但不是调用我们就不看。然后首先是打印同步代码
console.log("script start");
```

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6izmivm1ij30m801x745.jpg)

#### 将 setTimeout 放入宏任务队列

默认所包裹的代码，其实可以理解为是第一个宏任务，所以这里是宏任务 2

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6izmvovzjj30m8034dft.jpg)

#### 调用 async1，打印 同步代码 console.log( ‘async1 start’ )

我们说过看到带有 async 关键字的函数，不用害怕，它的仅仅是把 return 值包装成了 promise，其他并没有什么不同的地方。所以就很普通的打印 console.log( ‘async1 start’ )

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6izn7f29dj30m803dt8o.jpg)

#### 分析一下 await async2()

前文提过 await，1.它先计算出右侧的结果，2.然后看到 await 后，中断 async 函数

- 先得到 await 右侧表达式的结果。执行 async2()，打印同步代码 console.log(‘async2’), 并且 return Promise.resolve(undefined)
- await 后，中断 async 函数，先执行 async 外的同步代码

目前就直接打印 console.log(‘async2’)

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iznit5adj30m803edfu.jpg)

#### 被阻塞后，要执行 async 之外的代码

执行 new Promise()，Promise 构造函数是直接调用的同步代码，所以 console.log( ‘promise1’ )

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iznu7o6jj30m803l74a.jpg)

#### 代码运行到 promise.then()

代码运行到 promise.then()，发现这个是微任务，所以暂时不打印，只是推入当前宏任务的微任务队列中。

**注意：这里只是把 promise2 推入微任务队列，并没有执行。微任务会在当前宏任务的同步代码执行完毕，才会依次执行**

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6izo8rmlkj30m803c0sp.jpg)

#### 打印同步代码 console.log(‘script end’)

执行完这个同步代码后，「async 外的代码」终于走了一遍

下面该回到 await 表达式那里，执行 await Promise.resolve(undefined)了
![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6izoj1zjkj30m803c749.jpg)

#### 回到 async 内部，执行 await Promise.resolve(undefined)

这部分可能不太好理解，我尽量表达我的想法。

对于 await Promise.resolve(undefined) 如何理解呢？

根据 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/await) 原话我们知道

**如果一个 Promise 被传递给一个 await 操作符，await 将等待 Promise 正常处理完成并返回其处理结果。**

在我们这个例子中，就是 Promise.resolve(undefined)正常处理完成，并返回其处理结果。那么 await async2()就算是执行结束了。

目前这个 promise 的状态是 fulfilled，等其处理结果返回就可以执行 await 下面的代码了。

那何时能拿到处理结果呢？

回忆平时我们用 promise，调用 resolve 后，何时能拿到处理结果？是不是需要在 then 的第一个参数里，才能拿到结果。

（调用 resolve 时，会把 then 的参数推入微任务队列，等主线程空闲时，再调用它）

所以这里的 await Promise.resolve() 就类似于

```javascript
Promise.resolve(undefined).then(undefined => {});
```

把 then 的第一个回调参数 (undefined) => {} 推入微任务队列。

then 执行完，才是 await async2()执行结束。

await async2()执行结束，才能继续执行后面的代码

如图

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6izovbp1aj30m8037q2y.jpg)

#### 此时当前宏任务 1 都执行完了，要处理微任务队列里的代码。

微任务队列，先进选出的原则，

1. 执行微任务 1，打印 promise2
2. 执行微任务 2，没什么内容..

但是微任务 2 执行后，await async2()语句结束，后面的代码不再被阻塞，所以打印

console.log(‘async1 end’)

#### 宏任务 1 执行完成后,执行宏任务 2

宏任务 2 的执行比较简单，就是打印

console.log(‘setTimeout’)
