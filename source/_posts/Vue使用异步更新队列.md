---
title: Vue使用异步更新队列
date: 2020-02-09 00:06:21
tags:
  - VUE
categories: VUE
cover_img: https://tva1.sinaimg.cn/large/0082zybply1gbpfv349k5j31900u0u0y.jpg
feature_img: https://tva1.sinaimg.cn/large/0082zybply1gbpfv349k5j31900u0u0y.jpg
---

# Vue使用异步更新队列

## DOM的异步更新

异步更新队列指的是当状态发生变化时，Vue异步执行DOM更新。

我们在项目开发中会遇到这样一种场景：当我们将状态改变之后想获取更新后的DOM，往往我们获取到的DOM是更新前的旧DOM，我们需要使用`vm.$nextTick`方法异步获取DOM，例如：

```javascript
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function () {
    return {
      message: '没有更新'
    }
  },
  methods: {
    updateMessage: function () {
      this.message = '更新完成'
      console.log(this.$el.textContent) // => '没有更新'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => '更新完成'
      })
    }
  }
})
```

我们都知道这样做很麻烦，但为什么Vue还要这样做呢？

首先我们假设Vue是同步执行DOM更新，会有什么问题？

如果同步更新DOM将会有这样一个问题，我们在代码中同步更新数据N次，DOM也会更新N次，伪代码如下：

```javascript
this.message = '更新完成' // DOM更新一次
this.message = '更新完成2' // DOM更新两次
this.message = '更新完成3' // DOM更新三次
this.message = '更新完成4' // DOM更新四次
```

但事实上，我们真正想要的其实只是最后一次更新而已，也就是说前三次DOM更新都是可以省略的，我们只需要等所有状态都修改好了之后再进行渲染就可以减少一些无用功。

而这种无用功在Vue2.0开始变得更为重要，Vue2.0开始引入了Virtualdom，每一次状态发生变化后，状态变化的信号会发送给组件，组件内部使用VirtualDOM进行计算得出需要更新的具体的DOM节点，然后对DOM进行更新操作，每次更新状态后的渲染过程需要更多的计算，而这种无用功也将浪费更多的性能，所以异步渲染变得更加至关重要。

组件内部使用VIrtualDOM进行渲染，也就是说，组件内部其实是不关心哪个状态发生了变化，它只需要计算一次就可以得知哪些节点需要更新。也就是说，如果更改了N个状态，其实只需要发送一个信号就可以将DOM更新到最新。例如：

```javascript
this.message = '更新完成'
this.age =  23
this.name = berwin
```

代码中我们分三次修改了三种状态，但其实Vue只会渲染一次。因为VIrtualDOM只需要一次就可以将整个组件的DOM更新到最新，它根本不会关心这个更新的信号到底是从哪个具体的状态发出来的。

那如何才能将渲染操作推迟到所有状态都修改完毕呢？很简单，只需要将渲染操作推迟到本轮事件循环的最后或者下一轮事件循环。也就是说，只需要在本轮事件循环的最后，等前面更新状态的语句都执行完之后，执行一次渲染操作，它就可以无视前面各种更新状态的语法，无论前面写了多少条更新状态的语句，只在最后渲染一次就可以了。

将渲染推迟到本轮事件循环的最后执行渲染的时机会比推迟到下一轮快很多，所以Vue优先将渲染操作推迟到本轮事件循环的最后，如果执行环境不支持会降级到下一轮。

当然，Vue的变化侦测机制决定了它必然会在每次状态发生变化时都会发出渲染的信号，但Vue会在收到信号之后检查队列中是否已经存在这个任务，保证队列中不会有重复。如果队列中不存在则将渲染操作添加到队列中。

之后通过异步的方式延迟执行队列中的所有渲染的操作并清空队列，当同一轮事件循环中反复修改状态时，并不会反复向队列中添加相同的渲染操作。

所以我们在使用Vue时，修改状态后更新DOM都是异步的。

## Watcher队列

### update

在`Watcher`的源码中，我们发现`watcher`的`update`其实是异步的。（注：`sync`属性默认为`false`，也就是异步）

```javascript
update () {
    /* istanbul ignore else */
    if (this.lazy) {
        this.dirty = true
    } else if (this.sync) {
        /*同步则执行run直接渲染视图*/
        this.run()
    } else {
        /*异步推送到观察者队列中，下一个tick时调用。*/
        queueWatcher(this)
    }
}
```

### queueWatcher

`queueWatcher(this)`函数的代码如下：

```javascript
/*将一个观察者对象push进观察者队列，在队列中已经存在相同的id则该观察者对象将被跳过，除非它是在队列被刷新时推送*/
export function queueWatcher (watcher: Watcher) {
    /*获取watcher的id*/
    const id = watcher.id
    /*检验id是否存在，已经存在则直接跳过，不存在则标记哈希表has，用于下次检验*/
    if (has[id] == null) {
        has[id] = true
        if (!flushing) {
            /*如果没有flush掉，直接push到队列中即可*/
            queue.push(watcher)
        } else {
        ...
        }
        // queue the flush
        if (!waiting) {
            waiting = true
            nextTick(flushSchedulerQueue)
        }
    }
}
```

这段源码有几个需要注意的地方：

1. 首先需要知道的是`watcher`执行`update`的时候，默认情况下肯定是异步的，它会做以下的两件事：
   - 判断`has`数组中是否有这个`watcher`的`id`
   - 如果有的话是不需要把`watcher`加入`queue`中的，否则不做任何处理。
2. 这里面的`nextTick(flushSchedulerQueue)`中，`flushScheduleQueue`函数的作用主要是执行视图更新的操作，它会把`queue`中所有的`watcher`取出来并执行相应的视图更新。
3. 核心其实是`nextTick`函数了，下面我们具体看一下`nextTick`到底有什么用。

### nextTick

`nextTick`函数其实做了两件事情，一是生成一个`timerFunc`，把回调作为`microTask`或`macroTask`参与到事件循环中来。二是把回调函数放入一个`callbacks`队列，等待适当的时机执行。（这个时机和`timerFunc`不同的实现有关）

首先我们先来看它是怎么生成一个`timerFunc`把回调作为`microTask`或`macroTask`的。

```javascript
if (typeof Promise !== 'undefined' && isNative(Promise)) {
    /*使用Promise*/
    var p = Promise.resolve()
    var logError = err => { console.error(err) }
    timerFunc = () => {
        p.then(nextTickHandler).catch(logError)
        // in problematic UIWebViews, Promise.then doesn't completely break, but
        // it can get stuck in a weird state where callbacks are pushed into the
        // microTask queue but the queue isn't being flushed, until the browser
        // needs to do some other work, e.g. handle a timer. Therefore we can
        // "force" the microTask queue to be flushed by adding an empty timer.
        if (isIOS) setTimeout(noop)
    }
} else if (typeof MutationObserver !== 'undefined' && (
    isNative(MutationObserver) ||
    // PhantomJS and iOS 7.x
    MutationObserver.toString() === '[object MutationObserverConstructor]'
    )) {
    // use MutationObserver where native Promise is not available,
    // e.g. PhantomJS IE11, iOS7, Android 4.4
    /*新建一个textNode的DOM对象，用MutationObserver绑定该DOM并指定回调函数，在DOM变化的时候则会触发回调,该回调会进入主线程（比任务队列优先执行），即textNode.data = String(counter)时便会触发回调*/
    var counter = 1
    var observer = new MutationObserver(nextTickHandler)
    var textNode = document.createTextNode(String(counter))
    observer.observe(textNode, {
        characterData: true
    })
    timerFunc = () => {
        counter = (counter + 1) % 2
        textNode.data = String(counter)
    }
} else {
    // fallback to setTimeout
    /* istanbul ignore next */
    /*使用setTimeout将回调推入任务队列尾部*/
    timerFunc = () => {
        setTimeout(nextTickHandler, 0)
    }
}
```

值得注意的是，它会按照`Promise`、`MutationObserver`、`setTimeout`优先级去调用传入的回调函数。前两者会生成一个`microTask`任务，而后者会生成一个`macroTask`。（微任务和宏任务）

之所以会设置这样的优先级，主要是考虑到浏览器之间的兼容性（`IE`没有内置`Promise`）。另外，设置`Promise`最优先是因为`Promise.resolve().then`回调函数属于一个**微任务**，浏览器在一个`Tick`中执行完`macroTask`后会清空当前`Tick`所有的`microTask`再进行`UI`渲染，把`DOM`更新的操作放在`Tick`执行`microTask`的阶段来完成，相比使用`setTimeout`生成的一个`macroTask`会少一次`UI`的渲染。

而`nextTickHandler`函数，其实才是我们真正要执行的函数。

```javascript
function nextTickHandler () {
    pending = false
    /*执行所有callback*/
    const copies = callbacks.slice(0)
    callbacks.length = 0
    for (let i = 0; i < copies.length; i++) {
        copies[i]()
    }
}
```

这里的`callbacks`变量供`nextTickHandler`消费。而前面我们所说的`nextTick`函数第二点功能中“等待适当的时机执行”，其实就是因为`timerFunc`的实现方式有差异，如果是`Promise\MutationObserver`则`nextTickHandler`回调是一个`microTask`，它会在当前`Tick`的末尾来执行。如果是`setTiemout`则`nextTickHandler`回调是一个`macroTask`，它会在下一个`Tick`来执行。

还有就是`callbacks`中的成员是如何被`push`进来的？从源码中我们可以知道，`nextTick`是一个自执行的函数，一旦执行是`return`了一个`queueNextTick`，所以我们在调用`nextTick`其实就是在调用`queueNextTick`这个函数。它的源代码如下：

```javascript
return function queueNextTick (cb?: Function, ctx?: Object) {
    let _resolve
    /*cb存到callbacks中*/
    callbacks.push(() => {
        if (cb) {
            try {
            cb.call(ctx)
            } catch (e) {
            handleError(e, ctx, 'nextTick')
            }
        } else if (_resolve) {
            _resolve(ctx)
        }
    })
    if (!pending) {
        pending = true
        timerFunc()
    }
    if (!cb && typeof Promise !== 'undefined') {
        return new Promise((resolve, reject) => {
            _resolve = resolve
        })
    }
}
```

可以看到，一旦调用`nextTick`函数时候，传入的`function`就会被存放到`callbacks`闭包中，然后这个`callbacks`由`nextTickHandler`消费，而`nextTickHandler`的执行时间又是由`timerFunc`来决定。

回看`Watcher`这里面的`nextTick(flushSchedulerQueue)`中的`flushSchedulerQueue`函数其实就是`watcher`的视图更新。调用的时候会把它`push`到`callbacks`中来异步执行。

另外，关于`waiting`变量，这是很重要的一个标志位，它保证`flushSchedulerQueue`回调只允许被置入`callbacks`一次。

也就是说，默认`waiting`变量为`false`，执行一次后`waiting`为`true`，后续的`this.xxx`不会再次触发`nextTick`的执行，而是把`this.xxx`相对应的`watcher`推入`flushSchedulerQueue`的`queue`队列中。

**所以，也就是说DOM确实是异步更新，但是具体是在下一个Tick更新还是在当前Tick执行microTask的时候更新，具体要看nextTcik的实现方式，也就是具体跑的是Promise/MutationObserver还是setTimeout。**

## 为什么要异步更新

```html
<template>
  <div>
    <div>{{test}}</div>
  </div>
</template>
```

```javascript
export default {
    data () {
        return {
            test: 0
        };
    },
    mounted () {
      for(let i = 0; i < 1000; i++) {
        this.test++;
      }
    }
}
```

现在有这样的一种情况，`mounted`的时候`test`的值会被`++`循环执行`1000`次。 每次`++`时，都会根据响应式触发`setter->Dep->Watcher->update->run`。 如果这时候没有异步更新视图，那么每次`++`都会直接操作`DOM`更新视图，这是非常消耗性能的。 所以`Vue`实现了一个`queue`队列，在下一个`Tick`（或者是当前`Tick`的微任务阶段）的时候会统一执行`queue`中`Watcher`的`run`。同时，拥有相同`id`的`Watcher`不会被重复加入到该`queue`中去，所以不会执行`1000`次`Watcher`的`run`。最终更新视图只会直接将`test`对应的`DOM`的`0`变成`1000`。 保证更新视图操作`DOM`的动作是在当前栈执行完以后下一个`Tick`（或者是当前`Tick`的微任务阶段）的时候调用，大大优化了性能。

