---
title: 深入浅出Node.js学习笔记（二）
date: 2019-12-19 17:25:43
tags:
  - Node.js
categories: Node.js
cover_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gadm4v0imcj31900u0b29.jpg
feature_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gadm4v0imcj31900u0b29.jpg
---

# 深入浅出Node.js学习笔记（二）

## 第四章 异步编程

### 函数式编程

#### 高阶函数

高阶函数是可以把函数作为参数，或是将函数作为返回值的函数

```javascript
function foo(x) { 
  return function () {
    return x; 
  };
}
```

后续传递风格(Continuation Passing Style)的结果接收 方式，而非单一的返回值形式。后续传递风格的程序编写将函数的业务重点从返回值转移到了回调函数中。

##### 例子

ECMAScript5中提供的一些数组方法(sort()、forEach()、 map()、reduce()、reduceRight()、filter()、every()、some())十分典型

#### 偏函数用法

偏函数用法是指创建一个调用另外一个部分——参数或变量已经预置的函数——的函数的用法

```javascript
var toString = Object.prototype.toString;
var isString = function (obj) {
  return toString.call(obj) == '[object String]';
};
var isFunction = function (obj) {
  return toString.call(obj) == '[object Function]'; 
};
```

我们需要重复去定义一些相似的函数，如果有更多的isXXX()，就会出现更多的冗余代码.

```javascript
var isType = function (type) { 
  return function (obj) {
    return toString.call(obj) == '[object ' + type + ']'; 
  };
};
var isString = isType('String');
var isFunction = isType('Function');
```

这 种通过指定部分参数来产生一个新的定制函数的形式就是偏函数

### 异步编程的优势与难点

#### 优势

最大的特性：基于事件驱动的非阻塞I/O模型

非阻塞I/O可以使CPU与I/O并不相互依赖等待，让资源得到更好的利用

并行带来的想象空间更大，延展而开的是分布式和云。并行使得各个单点之间能够更有效地组织起来

![image-20191213111035164](https://tva1.sinaimg.cn/large/006tNbRwgy1g9ux6kboyzj30qc0iu3zv.jpg)

#### 难点

##### 难点1:异常处理

异步I/O的实现主要包含两个阶段:提交请求和处理结果。这两个阶段中间有事件循环的调度，两者彼此不关联。异步方法则通常在第一个阶段提交请求后立即返回，因为异常并不一定发生在这个阶段，try/catch的功效在此处不会发挥任何作用。

```javascript
var async = function (callback) {
  process.nextTick(callback);
};
```

```javascript
try {
  async(callback); 
} catch (e) { 
  // TODO
}
```

调用async()方法后，callback被存放起来，直到下一个事件循环(Tick)才会取出来执行。尝试对异步方法进行try/catch操作只能捕获当次事件循环内的异常，对callback执行时抛出的异常将无能为力。

Node在处理异常上形成了一种约定，将异常作为回调函数的第一个实参传回，如果为空值，则表明异步调用没有异常抛出：

```javascript
async(function (err, results) { 
  // TODO
});
```

自行编写的异步方法上，需要去遵循这样一些原则: 

* 原则一:必须执行调用者传入的回调函数; 
* 原则二:正确传递回异常供调用者判断。

##### 难点2:函数嵌套过深

##### 难点3:阻塞代码

没有sleep()这样的线程沉睡功能

setInterval()和setTimeout()并不能阻塞后续代码的持续执行

##### 难点4:多线程编程

对于服务器端而言，如果服务器是多核CPU，单个Node进程实质上是没有充分利用多核CPU的。

Node借鉴了这个模式，child_process是其基础API，cluster模块是更深层次的应用。借助Web Workers的模式，开发人员要更多地去面临跨线程的编程，这对于以往的JavaScript编程经验是较少考虑的。

##### 难点5:异步转同步

Node中试图同步式编程，但并不能得到原生支持，需要借助库或者编译等手段来实现

### 异步编程解决方案

#### 事件发布/订阅模式

事件监听器模式是一种广泛用于异步编程的模式，是回调函数的事件化，又称发布/订阅模式

```javascript
// 订阅
emitter.on("event1", function (message) {
  console.log(message); 
});
// 发布
emitter.emit('event1', "I am message!");
```

事件发布/订阅模式可以实现一个事件与多 个回调函数的关联，这些回调函数又称为事件侦听器。通过emit()发布事件后，消息会立即传递给当前事件的所有侦听器执行。侦听器可以很灵活地添加和删除，使得事件和具体处理逻辑之间可以很轻松地关联和解耦。

Node对事件发布/订阅的机制做了一些额外的处理，这大多是基于健壮性而考虑的:

* 如果对一个事件添加了超过10个侦听器，将会得到一条警告。这一处设计与Node自身单线程运行有关，设计者认为侦听器太多可能导致内存泄漏，所以存在这样一条警告。调用emitter.setMaxListeners(0);可以将这个限制去掉。另一方面，由于事件发布会引起一系列侦听器执行，如果事件相关的侦听器过多，可能存在过多占用CPU的情景。
* 为了处理异常，EventEmitter对象对error事件进行了特殊对待。如果运行期间的错误触发了error事件，EventEmitter会检查是否有对error事件添加过侦听器。如果添加了，这个错误将会交由该侦听器处理，否则这个错误将会作为异常抛出。如果外部没有捕获这个异常，将会引起线程退出。一个健壮的EventEmitter实例应该对error事件做处理。

##### 1. 继承events模块

```javascript
var events = require('events');
function Stream() { 
  events.EventEmitter.call(this);
}
util.inherits(Stream, events.EventEmitter);
```

Node在util模块中封装了继承的方法，所以此处可以很便利地调用。开发者可以通过这样的方式轻松继承EventEmitter类，利用事件机制解决业务问题。在Node提供的核心模块中，有近半数都继承自EventEmitter。

##### 2. 利用事件队列解决雪崩问题

雪崩问题，就是在高访问量、大并发量的情况下缓存失效的情景，此时大量的请求同时涌入数据库中，数据库无法同时承受如此大的查询请求，进而往前影响到网站整体的响应速度

```javascript
var proxy = new events.EventEmitter();
var status = "ready";
var select = function (callback) {
  proxy.once("selected", callback); 
  if (status === "ready") {
    status = "pending";
    db.select("SQL", function (results) {
      proxy.emit("selected", results);
      status = "ready"; 
    });
 } 
};
```

利用了once()方法，将所有请求的回调都压入事件队列中，利用其执行一次就会将监视器移除的特点，保证每一个回调只会被执行一次。对于相同的SQL语句，保证在同一个查询开始到结束的过程中永远只有一次。

##### 3. 多异步之间的协作方案

由于多个异步场景中回调函数的执行并不能保证顺序，且回调函数之间互相没有任何交集，所以需要借助一个第三方函数和第三方变量来处理异步协作的结果。通常，我们把这个用于检测次数的变量叫做哨兵变量

##### 4. EventProxy的原理

EventProxy来自于Backbone的事件模块，Backbone的事件模块是Model、View模块的基础功能，在前端有广泛的使用。它在每个非all事件触发时都会触发一次all事件

```javascript
// Trigger an event, firing all bound callbacks. Callbacks are passed the 
// same arguments as `trigger` is, apart from the event name.
// Listening for `"all"` passes the true event name as the first argument 
trigger : function(eventName) {
  var list, calls, ev, callback, args;
  var both = 2;
  if (!(calls = this._callbacks)) return this;
  while (both--) {
    ev = both ? eventName : 'all'; 
    if (list = calls[ev]) {
      for (var i = 0, l = list.length; i < l; i++) {
        if (!(callback = list[i])) {
          list.splice(i, 1); i--; l--; 
        } else {
          args = both ? Array.prototype.slice.call(arguments, 1) : arguments;

          callback[0].apply(callback[1] || this, args); 
        }
      }
    }
  }
  return this; 
}
```

EventProxy则是将all当做一个事件流的拦截层，在其中注入一些业务来处理单一事件无法解决的异步处理问题。类似的扩展方法还有all()、tail()、after()、not()和any()等。

##### 5. EventProxy的异常处理

```javascript
exports.getContent = function (callback) { 
  var ep = new EventProxy();
  ep.all('tpl', 'data', function (tpl, data) { 
    // 成功回调
    callback(null, {
      template: tpl,
      data: data 
    });
}); 
  //绑定错误处理函数 
  ep.fail(callback);
  fs.readFile('template.tpl', 'utf-8', ep.done('tpl'));
  db.get('some sql', ep.done('data')); 
};
```

EventProxy提供了fail()和done()这两个实例方法来优化异常处理，使得开发者将精力关注在业务部分，而不是在异常捕获上。

###### fail()方法的实现

```javascript
ep.fail(callback);
```

等价于：

```javascript
ep.fail(function (err) { 
  callback(err);
});
```

又等价于:

```javascript
ep.bind('error', function (err) { 
  // 卸载掉所有处理函数 
  ep.unbind();
  // 异常回调
  callback(err); 
});
```

###### done()方法的实现

```javascript
ep.done('tpl');
```

等价于:

```javascript
function (err, content) { 
  if (err) {
  // 一旦发生异常，一律交给error事件处理函数处理
  return ep.emit('error', err); 
  }
  ep.emit('tpl', content); 
}
```

#### Promise/Deferred模式

先执行异步调用，延迟传递处理的方式

```javascript
$.get('/api') 
  .success(onSuccess) 
  .error(onError) 
  .complete(onComplete);
```

这使得即使不调用success()、error()等方法，Ajax也会执行，这样的调用方式比预先传入回调让人觉得舒适一些。

在原始的API中，一个事件只能处理一个回调，而通过Deferred对象，可以对事件加入任意的业务处理逻辑。

```javascript
$.get('/api') 
  .success(onSuccess1) 
  .success(onSuccess2);
```

CommonJS草案目前已经抽象出了Promises/A、 Promises/B、Promises/D这样典型的异步Promise/Deferred模型，这使得异步操作可以以一种优雅的方式出现。

##### Promises/A

- Promise操作只会处在3种状态的一种:未完成态、完成态和失败态。
- Promise的状态只会出现从未完成态向完成态或失败态转化，不能逆反。完成态和失败态不能互相转化。
- Promise的状态一旦转化，将不能被更改。

![image-20191216113903367](https://tva1.sinaimg.cn/large/006tNbRwgy1g9yev5y4i1j30gs0c4q3v.jpg)

###### Promise对象的then()

一个Promise对象只要具备then()方法即可

* 接受完成态、错误态的回调方法。在操作完成或出现错误时，将会调用对应方法。
* 可选地支持progress事件回调作为第三个方法。
* then()方法只接受function对象，其余对象将被忽略。 
* then()方法继续返回Promise对象，以实现链式调用。

###### Deferred，延迟对象

触发执行这些回调函数的地方，实现这些功能的对象通常被称为Deferred，即延迟对象

###### Promise和Deferred的差别

Deferred主要是用于内部， 用于维护异步模型的状态;Promise则作用于外部，通过then()方法暴露给外部以添加自定义逻辑。

![image-20191216114535955](https://tva1.sinaimg.cn/large/006tNbRwgy1g9yf1x7tvnj30ws0coabq.jpg)

##### Promise中的多异步协作

通过all()方法抽象多个异步操作。只有所有异步操作成功，这个异步操作才算成功， 一旦其中一个异步操作失败，整个异步操作就失败。

##### Promise的进阶知识

* 支持序列执行的Promise

  ```javascript
  promise()
    .then(obj.api1) 
    .then(obj.api2) 
    .then(obj.api3) 
    .then(obj.api4) 
    .then(function (value4) {
    // Do something with value4 
  }, function (error) {
    // Handle any error from step1 through step4 
  })
    .done();
  
  ```

  改造一下代码以实现链式调用

  ```javascript
  var Deferred = function () { 
    this.promise = new Promise();
  };
  
  // 完成态
  Deferred.prototype.resolve = function (obj) {
    var promise = this.promise;
    var handler;
    while ((handler = promise.queue.shift())) {
      if (handler && handler.fulfilled) { 
        var ret = handler.fulfilled(obj); 
        if (ret && ret.isPromise) {
          ret.queue = promise.queue; 
          this.promise = ret; 
          return;
        } 
      }
    } 
  };
  
  // 失败态
  Deferred.prototype.reject = function (err) {
    var promise = this.promise;
    var handler;
    while ((handler = promise.queue.shift())) {
      if (handler && handler.error) { 
        var ret = handler.error(err); 
        if (ret && ret.isPromise) {
          ret.queue = promise.queue; 
          this.promise = ret;
          return; 
        }
      } 
    }
  };
  
   // 生成回调函数 
  Deferred.prototype.callback = function () {
    var that = this;
    return function (err, file) {
      if (err) {
        return that.reject(err);
      }
      that.resolve(file); 
    };
  };
  
  var Promise = function () {
    // 队列用于存储待执行的回调函数 
    this.queue = [];
    this.isPromise = true;
  };
  
  Promise.prototype.then = function (fulfilledHandler, errorHandler, progressHandler) { 
    var handler = {};
    if (typeof fulfilledHandler === 'function') {
      handler.fulfilled = fulfilledHandler; 
    }
    if (typeof errorHandler === 'function') { 
      handler.error = errorHandler;
    } 
    this.queue.push(handler); 
    return this;
  };
  ```

读取第二个文件是依 赖于第一个文件中的内容的

```javascript
var readFile1 = function (file, encoding) {
  var deferred = new Deferred();
  fs.readFile(file, encoding, deferred.callback());
  return deferred.promise;
};
var readFile2 = function (file, encoding) {
  var deferred = new Deferred();
  fs.readFile(file, encoding, deferred.callback());
  return deferred.promise;
};
readFile1('file1.txt', 'utf8').then(function (file1) { 
  return readFile2(file1.trim(), 'utf8');
}).then(function (file2) { 
  console.log(file2);
});
```

要让Promise支持链式执行，主要通过以下两个步骤。
 (1) 将所有的回调都存到队列中。
 (2) Promise完成时，逐个执行回调，一旦检测到返回了新的Promise对象，停止执行，然后将当前Deferred对象的promise引用改变为新的Promise对象，并将队列中余下的回调转交给它。

* 将API Promise化

可以 批量将方法Promise化

```javascript
// smooth(fs.readFile);
var smooth = function (method) {
  return function () {
    var deferred = new Deferred();
    var args = Array.prototype.slice.call(arguments, 0); 
    args.push(deferred.callback());
    method.apply(null, args);
    return deferred.promise;
  }; 
};
```

于是前面的两次文件读取的构造可以简化为:

```javascript
var readFile = smooth(fs.readFile);
```

于是代码锐减到：

```javascript
var readFile = smooth(fs.readFile); readFile('file1.txt', 'utf8').then(function (file1) {
  return readFile(file1.trim(), 'utf8'); }).then(function (file2) {
  // file2 => I am file2
  console.log(file2); 
});
```

#### 流程控制库

##### 1. 尾触发与Next

需要手工调用才能持续执行后续调用的

![image-20191216150057148](https://tva1.sinaimg.cn/large/006tNbRwgy1g9ykp64h0pj314a0bggn2.jpg)

中间件机制使得在处理网络请求时，可以像面向切面编程一样进行过滤、验证、日志等功能， 而不与具体业务逻辑产生关联，以致产生耦合

尽管中间件这种尾触发模式并不要求每个中间方法都是异步的，但是如果每 个步骤都采用异步来完成，实际上只是串行化的处理，没办法通过并行的异步调用来提升业务的 处理效率。流式处理可以将一些串行的逻辑扁平化，但是并行逻辑处理还是需要搭配事件或者 Promise完成的，这样业务在纵向和横向都能够各自清晰。

在Connect中，尾触发十分适合处理网络请求的场景。将复杂的处理逻辑拆解为简洁、单一 的处理单元，逐层次地处理请求对象和响应对象。

##### 2. async

###### 异步的串行执行

```javascript
async.series([
  function (callback) {
    fs.readFile('file1.txt', 'utf-8', callback); 
  },
  function (callback) {
    fs.readFile('file2.txt', 'utf-8', callback);
  }
], function (err, results) {
  // results => [file1.txt, file2.txt] 
});
```

这段代码等价于:

```javascript
fs.readFile('file1.txt', 'utf-8', function (err, content) { 
  if (err) {
    return callback(err); 
  }
  fs.readFile('file2.txt ', 'utf-8', function (err, data) {
    if (err) {
      return callback(err);
    }
    callback(null, [content, data]); 
  });
});
```

series()方法中传入的函数callback()并非由使用者指定。事实上，此处的回调函数由async通过高阶函数的方式注入，这里隐含了特殊的逻 辑。每个callback()执行时会将结果保存起来，然后执行下一个调用，直到结束所有调用。最终的回调函数执行时，队列里的异步调用保存的结果以数组的方式传入。这里的异常处理规则是一 旦出现异常，就结束所有调用，并将异常传递给最终回调函数的第一个参数。

###### 异步的并行执行

当我们需要通过并行来提升性能时，async提供了parallel()方法，用以并行执行一些异步操作。

```javascript
async.parallel([ 
  function (callback) {
    fs.readFile('file1.txt', 'utf-8', callback); 
  },
  function (callback) {
    fs.readFile('file2.txt', 'utf-8', callback);
  }
], function (err, results) {
  // results => [file1.txt, file2.txt] 
});
```

上面这段代码等价于下面的代码:

```javascript
var counter = 2;
var results = [];
var done = function (index, value) {
  results[index] = value; 
  counter--;
  if (counter === 0) {
    callback(null, results); 
  }
};

// 只传递第一个异常
var hasErr = false;
var fail = function (err) {
  if (!hasErr) { 
    hasErr = true; 
    callback(err);
  } 
};
fs.readFile('file1.txt', 'utf-8', function (err, content) { 
  if (err) {
    return fail(err); 
  }
  done(0, content); 
});
fs.readFile('file2.txt', 'utf-8', function (err, data) { 
  if (err) {
    return fail(err); 
  }
  done(1, data); 
});
```

通过async编写的代码既没有深度的嵌套，也没有复杂的状态判断，它的诀窍依然来 自于注入的回调函数

parallel()方法对于异常的判断依然是一旦某个异步调用产生了异常，就 会将异常作为第一个参数传入给最终的回调函数。只有所有异步调用都正常完成时，才会将结果 以数组的方式传入。

###### 异步调用的依赖处理

series()适合无依赖的异步串行执行，但当前一个的结果是后一个调用的输入时，series()方法就无法满足需求了

async提供了**waterfall()**方法来满足

```javascript
async.waterfall([ 
  function (callback) {
    fs.readFile('file1.txt', 'utf-8', function (err, content) { 
      callback(err, content);
    }); 
  },
  function (arg1, callback) {
    // arg1 => file2.txt
    fs.readFile(arg1, 'utf-8', function (err, content) {
      callback(err, content); 
    });
  },
  function(arg1, callback){ 
    // arg1 => file3.txt
    fs.readFile(arg1, 'utf-8', function (err, content) {
      callback(err, content); 
    });
  }
], function (err, result) {
  // result => file4.txt 
});
```

* 自动依赖处理

auto()方法能根据依赖关系自动分析，以最佳的顺序执行业务

```javascript
async.auto(deps);
```

##### 3. Step

```javascript
Step(task1, task2, task3);
```

Step接受任意数量的任务，所有的任务都将会串行依次执行。

```javascript
Step(
  function readFile1() {
    fs.readFile('file1.txt', 'utf-8', this);
  },
  function readFile2(err, content) {
    fs.readFile('file2.txt', 'utf-8', this);
  },
  function done(err, content) {
    console.log(content); 
  }
);
```

Step用到了this关键字。事实上，它是Step内部的一个next()方法，将异步调用的结果传递给下一个任务作为参 数，并调用执行。

* 并行任务执行

this具有一个parallel()方法，它告诉Step，需要等所有任务完成时才进行下一个任务

```javascript
Step(
  function readFile1() {
    fs.readFile('file1.txt', 'utf-8', this.parallel());
    fs.readFile('file2.txt', 'utf-8', this.parallel()); 
  },
  function done(err, content1, content2) { 
    // content1 => file1
    // content2 => file2 
    console.log(arguments);
  } 
);
```

使用parallel()的时候需要小心的是，如果异步方法的结果传回的是多个参数，Step将只会取前两个参数

Step的parallel()方法的原理是每次执行时将内部的计数器加1，然后返回一个回调函数，这个回调函数在异步调用结束时才执行。当回调函数执行时，将计数器减1。当计数器为0的时候， 告知Step所有异步调用结束了，Step会执行下一个方法。

Step与async相同的是异常处理，一旦有一个异常产生，这个异常会作为下一个方法的第一个 参数传入

* 结果分组

```javascript
Step(
  function readDir() {
    fs.readdir(__dirname, this); 
  },
  function readFiles(err, results) {
    if (err) throw err;
    // Create a new group
    var group = this.group();
    results.forEach(function (filename) {
      if (/\.js$/.test(filename)) {
        fs.readFile(__dirname + "/" + filename, 'utf8', group());
      } 
    });
  },
  function showAll(err, files) {
    if (err) throw err;
    console.dir(files); 
  }
);
```

我们注意到有两次group()的调用。第一次调用是告知Step要并行执行，第二次调用的结果将会生成一个回调函数，而回调函数接受的返回值将会按组存储。

parallel()传递给下一个任务的 结果是如下形式:

```javascript
function (err, result1, result2, ...);
```

group()传递的结果是:

```javascript
function (err, results);
```

这个函数返回的数据保存在数组中。

##### 4. wind

* 异步任务定义

```javascript
eval(Wind.compile("async", function() {}));
Wind.Async.sleep(20);
```

Wind.compile()会将普通的函数进行编译，然后交给eval()执行。

eval(Wind.compile("async", function () {}));定义了异步任务。Wind.Async.sleep();内置了对setTimeout()的封装。

除了通过eval(Wind.compile("async", function () {}));定义任务外，正式的任务创建方法为Task.create()。

* $await()与任务模型

```javascript
$await()
```

事实上，它并不是一个方法，也不存在于上下文中，只是一个等待的占位符，告之编译器这里需要等待。

$await()接受的参数是一个任务对象，表示等待任务结束后才会执行后续操作。每一个异步 操作都可以转化为一个任务，wind正是基于任务模型实现的。

wind提供了whenAll()来处理并发，通过$await关键字将等待配置的所有任务完成后才向下继续执行。

```javascript
var parallel = eval(Wind.compile("async", function () { 
  var result = $await(Task.whenAll({
    file1: readFileAsync('file1.txt', 'utf-8'),
    file2: readFileAsync('file2.txt', 'utf-8') 
  }));
  console.log(result.file1);
  
  console.log(result.file2); }));
parallel().start();
//得到输出:
file1 file2
```

* 异步方法转换辅助函数

这种近同步编程的体验需要我们额外 或者提前完成的事情是:将异步方法任务化。

wind提供了两个 方法来辅助转换:

1. Wind.Async.Binding.fromCallback 用于转换这类无异常的异步调用为wind中的任务
2. Wind.Async.Binding.fromStandard 用于转换这类带异常的异步调用到wind中的任务。

### 异步并发控制

同步I/O因为每个I/O都是彼此阻塞的，在循环体 中，总是一个接着一个调用，不会出现耗用文件描述符太多的情况，同时性能也是低下的;对于 异步I/O，虽然并发容易实现，但是由于太容易实现，依然需要控制。换言之，尽管是要压榨底 层系统的性能，但还是需要给予一定的过载保护，以防止过犹不及。

#### bagpipe的解决方案

* 通过一个队列来控制并发量。
* 如果当前活跃(指调用发起但未执行回调)的异步调用量小于限定值，从队列中取出执行。 
* 如果活跃调用达到限定值，调用暂时存放在队列中。
* 每个异步调用结束时，从队列中取出新的异步调用执行

用户传入的回调函数被真正执行前，被封装替换过。这个封装的回调函数内部的逻辑将活跃 值的计数器减1后，主动调用next()执行后续等待的异步调用。

bagpipe类似于打开了一道窗口，允许异步调用并行进行，但是严格限定上限。仅仅在调用 push()时分开传递，并不对原有API有任何侵入。

##### 拒绝模式

```javascript
// 设定最大并发数为10
var bagpipe = new Bagpipe(10, {
  refuse: true 
});
```

在拒绝模式下，如果等待的调用队列也满了之后，新来的调用就直接返给它一个队列太忙的 拒绝异常。

##### 超时控制

造成队列拥塞的主要原因是异步调用耗时太久，调用产生的速度远远高于执行的速度。为了防止某些异步调用使用了太多的时间，我们需要设置一个时间基线，将那些执行时间太久的异步调用 清理出活跃队列，让排队中的异步调用尽快执行。否则在拒绝模式下，会有太多的调用因为某个执 行得慢，导致得到拒绝异常。

超时控制是为异步调用设置一个时间阈值，如果异步调用 没有在规定时间内完成，我们先执行用户传入的回调函数，让用户得到一个超时异常，以尽早返 回。然后让下一个等待队列中的调用执行。

```javascript
// 设定最大并发数为10
var bagpipe = new Bagpipe(10, {
  timeout: 3000 
});
```

#### async的解决方案

async也提供了一个方法用于处理异步调用的限制:parallelLimit()

```javascript
async.parallelLimit([ 
  function (callback) {
    fs.readFile('file1.txt', 'utf-8', callback); 
  },
  function (callback) {
    fs.readFile('file2.txt', 'utf-8', callback);
  }
], 1, function (err, results) {
  // TODO 
});
```

parallelLimit()与parallel()类似，但多了一个用于限制并发数量的参数，使得任务只能同 时并发一定数量，而不是无限制并发。

parallelLimit()方法的缺陷在于无法动态地增加并行任务。async提供了queue()方法 来满足该需求.

```javascript
var q = async.queue(function (file, callback) {
  fs.readFile(file, 'utf-8', callback);
},2);
q.drain=function(){
  // 完成了队列中的所有任务 
};
fs.readdirSync('.').forEach(function (file) {
  q.push(file, function (err, data) {
    // TODO 
  });
});
```

尽管queue()实现了动态添加并行任务，但是相比parallelLimit()，由于queue()接收的参数是固定的，它丢失了parallelLimit()的多样性

## 第五章 内存控制

### V8 的垃圾回收机制与内存限制

#### V8 的内存限制

在Node中通过JavaScript 使用内存时就会发现只能使用部分内存(64位系统下约为1.4 GB，32位系统下约为0.7 GB)。在 这样的限制下，将会导致Node无法直接操作大内存对象。

造成这个问题的主要原因在于Node基于V8构建，所以在Node中使用的JavaScript对象基本上 都是通过V8自己的方式来进行分配和管理的。V8的这套内存管理机制在浏览器的应用场景下使 用起来绰绰有余，足以胜任前端页面中的所有需求。但在Node中，这却限制了开发者随心所欲使 用大内存的想法。

#### V8 的对象分配

##### 内存使用量的查看

在V8中，所有的JavaScript对象都是通过堆来进行分配的。Node提供了V8中内存使用量的查 看方式，执行下面的代码，将得到输出的内存信息:

```javascript
$ node
> process.memoryUsage(); 
{ rss: 14958592,
  heapTotal: 7195904, 
  heapUsed: 2821496 
}
```

heapTotal和heapUsed是V8的堆内存使用情况，前者是已申请到的堆内存，后者是当前使用的量。

当我们在代码中声明变量并赋值时，所使用对象的内存就分配在堆中。如果已申请的堆空闲 内存不够分配新的对象，将继续申请堆内存，直到堆的大小超过V8的限制为止。

##### V8限制堆大小的原因

以1.5 GB的垃圾回收堆内存为例，V8做一次小的垃圾回收需要50毫秒以上，做一 次非增量式的垃圾回收甚至要1秒以上。这是垃圾回收中引起JavaScript线程暂停执行的时间，在 这样的时间花销下，应用的性能和响应能力都会直线下降。这样的情况不仅仅后端服务无法接受， 前端浏览器也无法接受。因此，在当时的考虑下直接限制堆内存是一个好的选择。

##### 调整内存限制的大小

```javascript
node --max-old-space-size=1700 test.js // 单位为MB 
// 或者
node --max-new-space-size=1024 test.js // 单位为KB
```

#### V8 的垃圾回收机制

##### V8主要的垃圾回收算法

V8的垃圾回收策略主要基于分代式垃圾回收机制

* V8的内存分代

  在V8中，主要将内存分为新生代和老生代两代。新生代中的对象为存活时间较短的对象，老生代中的对象为存活时间较长或常驻内存的对象。

  ![image-20191219183341084](https://tva1.sinaimg.cn/large/006tNbRwgy1ga27phiuq7j30r005q74q.jpg)

  V8堆的整体大小就是新生代所用内存空间加上老生代的内存空间。前面我们提及的 --max-old-space-size命令行参数可以用于设置老生代内存空间的最大值，--max-new-space-size 命令行参数则用于设置新生代内存空间的大小的。比较遗憾的是，这两个最大值需要在启动时就 指定。这意味着V8使用的内存没有办法根据使用情况自动扩充，当内存分配过程中超过极限值 时，就会引起进程出错。

  * 对于新生代内存，它由两个reserved_semispace_size_所构成.按机器位数不同，reserved_semispace_size_在64位系统和32位系统上分别为16 MB和8 MB。所以新生 代内存的最大值在64位系统和32位系统上分别为32 MB和16 MB。
  * 默认情况下，V8堆内存的最大值在64位系统上为1464 MB，32位系统上则为732 MB。 这个数值可以解释为何在64位系统下只能使用约1.4 GB内存和在32位系统下只能使用约0.7 GB 内存。

* Scavenge算法

  新生代中的对象主要通过Scavenge算法进行垃圾回收。在Scavenge的具体实现中，主要采用了Cheney算法。

  * Cheney算法是一种采用复制的方式实现的垃圾回收算法。它将堆内存一分为二，每一部分空间称为semispace。在这两个semispace空间中，只有一个处于使用中，另一个处于闲置状态。处 于使用状态的semispace空间称为From空间，处于闲置状态的空间称为To空间。当我们分配对象 时，先是在From空间中进行分配。当开始进行垃圾回收时，会检查From空间中的存活对象，这 些存活对象将被复制到To空间中，而非存活对象占用的空间将会被释放。完成复制后，From空 间和To空间的角色发生对换。简而言之，在垃圾回收的过程中，就是通过将存活对象在两个 semispace空间之间进行复制。

  * Scavenge的缺点是只能使用堆内存中的一半，这是由划分空间和复制机制所决定的。但 Scavenge由于只复制存活的对象，并且对于生命周期短的场景存活对象只占少部分，所以它在时 间效率上有优异的表现。

  ![image-20191219184025403](https://tva1.sinaimg.cn/large/006tNbRwgy1ga27wgek1uj30rm07wq3m.jpg)

  当一个对象经过多次复制依然存活时，它将会被认为是生命周期较长的对象。这种较长生命周期的对象随后会被移动到老生代中，采用新的算法进行管理。对象从新生代中移动到老生代中 的过程称为晋升。

  在分代式垃圾回收的前提下，From空间中的存活对 象在复制到To空间之前需要进行检查。在一定条件下，需要将存活周期长的对象移动到老生代中， 也就是完成对象晋升。

  对象晋升的条件主要有两个，一个是对象是否经历过Scavenge回收，一个是To空间的内存占用比超过限制。当要从From空间复制一个对象到To空间时，如果 To空间已经使用了超过25%，则这个对象直接晋升到老生代空间中

  （设置25%这个限制值的原因是当这次Scavenge回收完成后，这个To空间将变成From空间，接 下来的内存分配将在这个空间中进行。如果占比过高，会影响后续的内存分配。）

* Mark-Sweep & Mark-Compact

  Mark-Sweep在标记阶段遍历堆中的所有对象，并标记活着的对象，在随后的清除阶段中，只清除没有被标记的对象。**Scavenge中只复制活着的对象，而Mark-Sweep只清理死亡对象。**

  Mark-Sweep最大的问题是在进行一次标记清除回收后，内存空间会出现不连续的状态。这种 内存碎片会对后续的内存分配造成问题，因为很可能出现需要分配一个大对象的情况，这时所有 的碎片空间都无法完成此次分配，就会提前触发垃圾回收，而这次回收是不必要的。

  在整理的 过程中，将活着的对象往一端移动，移动完成后，直接清理掉边界外的内存。

  ![image-20191219184724646](https://tva1.sinaimg.cn/large/006tNbRwgy1ga283qesgnj317y08sq4f.jpg)

  V8主要使用Mark-Sweep，在空间不足以对从新 生代中晋升过来的对象进行分配时才使用Mark-Compact。

* Incremental Marking

  为了避免出现JavaScript应用逻辑与垃圾回收器看到的不一致的情况，垃圾回收的3种基本算法都需要将应用逻辑暂停下来，待执行完垃圾回收后再恢复执行应用逻辑，这种行为被称为“全 停顿”(stop-the-world)。

  为了降低全堆垃圾回收带来的停顿时间，V8先从标记阶段入手，将原本要一口气停顿完成 的动作改为增量标记(incremental marking)，也就是拆分为许多小“步进”，每做完一“步进” 就让JavaScript应用逻辑执行一小会儿，垃圾回收与应用逻辑交替执行直到标记阶段完成。

  ![image-20191219185100781](https://tva1.sinaimg.cn/large/006tNbRwgy1ga287h5jwqj30sk09qt9k.jpg)

  V8在经过增量标记的改进后，垃圾回收的最大停顿时间可以减少到原本的1/6左右。

  V8后续还引入了延迟清理(lazy sweeping)与增量式整理(incremental compaction)，让清 理与整理动作也变成增量式的。同时还计划引入并行标记与并行清理，进一步利用多核性能降低 每次停顿的时间。

#### 查看垃圾回收日志

查看垃圾回收日志的方式主要是在启动时添加--trace_gc参数。在进行垃圾回收时，将会从 标准输出中打印垃圾回收的日志信息。通过分析垃圾回收日志，可以了解垃圾回收的运行状况，找出垃圾回收的哪些阶段比较耗时， 触发的原因是什么。

通过在Node启动时使用--prof参数，可以得到V8执行时的性能分析数据，其中包含了垃圾 回收执行时占用的时间。

（V8提供了linux-tick-processor工具用于统计日志信息。该工具可以从Node源码的 deps/v8/tools目录下找到，Windows下的对应命令文件为windows-tick-processor.bat。将该目录添 加到环境变量PATH中，即可直接调用）

### 高效使用内存

#### 作用域

##### 1. 标识符查找

标识符，可以理解为变量名。

##### 2. 作用域链

JavaScript在执行时会去查找该变量定义在哪里。它最先查找的是当前作用域，如果在当前作 用域中无法找到该变量的声明，将会向上级的作用域里查找，直到查到为止。这样的查找方式使得作 用域像一个链条。由于标识符的查找方向是向上的，所以变量只能向外访问，而不能向内访问。

##### 3. 变量的主动释放

* 如果变量是全局变量(不通过var声明或定义在global变量上)，由于全局作用域需要直到 进程退出才能释放，此时将导致引用的对象常驻内存(常驻在老生代中)。如果需要释放常驻内 存的对象，可以通过delete操作来删除引用关系。或者将变量重新赋值，让旧的对象脱离引用关系。在接下来的老生代内存清除和整理的过程中，会被回收释放。

* 在非全局作用域中，想主动释放变量引用的对象，也可以通过这样的方式。
* 虽然 delete操作和重新赋值具有相同的效果，但是在V8中通过delete删除对象的属性有可能干扰V8 的优化，所以通过赋值方式解除引用更好。

#### 闭包

实现外部作用域访问内部作用域中变量的方法叫做闭包(closure)

这得益 于高阶函数的特性:函数可以作为参数或者返回值。

一旦有变量引用这个中间函数，这个中间函数将不会释放，同时也会使原始的作用域不会得到释放，作用域 中产生的内存占用也不会得到释放。除非不再有引用，才会逐步释放。

### 内存指标

os模块中的 totalmem()和freemem()方法也可以查看内存使用情况

#### 查看内存使用情况

##### 1. 查看进程的内存占用

调用process.memoryUsage()可以看到Node进程的内存占用情况

```javascript
$ node
> process.memoryUsage() 
{ rss: 13852672,
  heapTotal: 6131200, 
  heapUsed: 2757120 
}
```

rss是resident set size的缩写，即进程的常驻内存部分。进程的内存总共有几部分，一部分是 rss，其余部分在交换区(swap)或者文件系统(filesystem)中。

除了rss外，heapTotal和heapUsed对应的是V8的堆内存信息。heapTotal是堆中总共申请的内 存量，heapUsed表示目前堆中使用中的内存量。这3个值的单位都是字节。

##### 2. 查看系统的内存占用

os模块中的totalmem()和freemem()这两个方法用于查看操作系统的内存使用情况，它们分别返回系统的总内存和闲置内存，以字节为单位。

```javascript
$ node
> os.totalmem() 
8589934592
> os.freemem() 
4527833088
>
```

#### 堆外内存

堆中的内存用量总是小于进程的常驻内存用 量，这意味着Node中的内存使用并非都是通过V8进行分配的。我们将那些不是通过V8分配的内存称为堆外内存。

Buffer对象不同于其他对象，它不经过V8的内存分配机制，所以也不 会有堆内存的大小限制。

利用堆外内存可以突破内存限制的问题。

**Node的内存构成主要由通过V8进行分配的部分和Node自行分配的 部分。受V8的垃圾回收限制的主要是V8的堆内存。**

### 内存泄漏

造成内存泄漏的原因：

* 缓存。
* 队列消费不及时。
* 作用域未释放。

#### 慎将内存当做缓存

* 在Node中，缓存并非物美价廉。一旦一个对象被当做缓存来使用，那就意味着它将会常 驻在老生代中。缓存中存储的键越多，长期存活的对象也就越多，这将导致垃圾回收在进行扫描 10 和整理时，对这些对象做无用功

* JavaScript开发者通常喜欢用对象的键值对来缓存东西，但这与严格意义上 的缓存又有着区别，严格意义的缓存有着完善的过期策略，而普通对象的键值对并没有。

##### 1. 缓存限制策略

为了解决缓存中的对象永远无法释放的问题，需要加入一种策略来限制缓存的无限增长。

模块机制：为了加速模块的引入，所有模块都会通 过编译执行，然后被缓存起来。由于通过exports导出的函数，可以访问文件模块中的私有变量， 这样每个文件模块在编译执行后形成的作用域因为模块缓存的原因，不会被释放。

##### 2. 缓存的解决方案

进 程之间无法共享内存。如果在进程内使用缓存，这些缓存不可避免地有重复，对物理内存的使用 是一种浪费。

采用进程外的缓存，进程自身不存储状态。外 部的缓存软件有着良好的缓存过期淘汰策略以及自有的内存管理，不影响Node进程的性能。

在Node中主要可以解决以下两个问题。

(1) 将缓存转移到外部，减少常驻内存的对象的数量，让垃圾回收更高效。
 (2) 进程之间可以共享缓存。

市面上较好的缓存有Redis和Memcached

#### 关注队列状态

队列在消费者-生产者模型中经常充当中间产物。一旦消费速度低于生产速度， 将会形成堆积。

深度的解决方案应该是监控队列的长度，一旦堆积，应当通过监控系统产生报警并通知相关 人员。另一个解决方案是任意异步调用都应该包含超时机制，一旦在限定的时间内未完成响应， 通过回调函数传递超时异常，使得任意异步调用的回调都具备可控的响应时间，给消费速度一个 下限值。

### 内存泄漏排查

常见的定位Node应用的内存泄漏的工具：

* v8-profiler。由Danny Coates提供，它可以用于对V8堆内存抓取快照和对CPU进行分析，但该项目已经有3年没有维护了。
* node-heapdump。这是Node核心贡献者之一Ben Noordhuis编写的模块，它允许对V8堆内存抓取快照，用于事后分析。

- node-mtrace。由Jimb Esser提供，它使用了GCC的mtrace工具来分析堆的使用。
- dtrace。在Joyent的SmartOS系统上，有完善的dtrace工具用来分析内存泄漏。
- node-memwatch。来自Mozilla的Lloyd Hilaiel贡献的模块，采用WTFPL许可发布。

#### node-heapdump

先构造如下一份包含内存泄 漏的代码示例，并将其存为server.js文件：

```javascript
var leakArray = [];
var leak = function () {
  leakArray.push("leak" + Math.random()); 
};
http.createServer(function (req, res) { 
  leak();
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n'); 
}).listen(1337);
  console.log('Server running at http://127.0.0.1:1337/');
```

在上面这段代码中，每次访问服务进程都将引起leakArray数组中的元素增加，而且得不到回收。我们可以用curl工具输入http://127.0.0.1:1337/命令来模拟用户访问。

##### 安装node-heapdump

```markdown
$ npm install heapdump
```

安装node-heapdump后，在代码的第一行添加如下代码将其引入:

```
var heapdump = require('heapdump');
```

引入node-heapdump后，就可以启动服务进程，并接受客户端的请求。访问多次之后， leakArray中就会具备大量的元素。这个时候我们通过向服务进程发送SIGUSR2信号，让 10 node-heapdump抓拍一份堆内存的快照。发送信号的命令如下:

```
$ kill -USR2 <pid>
```

这份抓取的快照将会在文件目录下以heapdump-<sec>.<usec>.heapsnapshot的格式存放。这是一份较大的JSON文件，需要通过Chrome的开发者工具打开查看。

![image-20191220142253073](https://tva1.sinaimg.cn/large/006tNbRwgy1ga362sgilrj31400qmwq7.jpg)

可以看到有大量的leak字符串存在，这些字符串就是一直未能得到回收的数据。 通过在开发者工具的面板中查看内存分布，我们可以找到泄漏的数据，然后根据这些信息找到造 成泄漏的代码。

#### node-memwatch

```javascript
var memwatch = require('memwatch'); memwatch.on('leak', function (info) {
  console.log('leak:');
  console.log(info); 
});
memwatch.on('stats', function (stats) {
  console.log('stats:') 
  console.log(stats);
});
var http = require('http');
var leakArray = [];
var leak = function () {
  leakArray.push("leak" + Math.random()); 
};
http.createServer(function (req, res) { 
  leak();
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n'); 
}).listen(1337);
console.log('Server running at http://127.0.0.1:1337/');
```

##### 1. stats事件

在进程中使用node-memwatch之后，每次进行全堆垃圾回收时，将会触发一次stats事件，这 4 个事件将会传递内存的统计信息。

![image-20191220142653590](https://tva1.sinaimg.cn/large/006tNbRwgy1ga366z9jroj30mi0a6q4r.jpg)

num_full_gc和num_inc_gc比较直观地反应了垃圾回收的情况

##### 2. leak事件

如果经过连续5次垃圾回收后，内存仍然没有被释放，这意味着有内存泄漏的产生，node-memwatch会出发一个leak事件。

![image-20191220143429405](https://tva1.sinaimg.cn/large/006tNbRwgy1ga36evbve4j30uk05wgmj.jpg)

这个数据能显示5次垃圾回收的过程中内存增长了多少。

##### 3. 堆内存比较

最终得到的leak事件的信息只能告知我们应用中存在内存泄漏，具体问题产生在何处还需要从V8的堆内存上定位。node-memwatch提供了抓取快照和比较快照的功能，它能够比较堆上对象 的名称和分配数量，从而找出导致内存泄漏的元凶。

```javascript
var memwatch = require('memwatch');
var leakArray = [];
var leak = function () { 
  leakArray.push("leak" + Math.random());
};

// Take first snapshot
var hd = new memwatch.HeapDiff();
for (var i = 0; i < 10000; i++) { 
  leak();
}
// Take the second snapshot and compute the diff 
var diff = hd.end(); 
console.log(JSON.stringify(diff, null, 2));
```

执行上面这段代码，得到的输出结果如下所示:

```json
$ node diff.js 
{
  "before": {
    "nodes": 11719,
    "time": "2013-10-07T06:32:07.000Z",
    "size_bytes": 1493304,
    "size": "1.42 mb"
  }, 
  "after": {
    "nodes": 31618,
    "time": "2013-10-07T06:32:07.000Z", 
    "size_bytes": 2684864,
    "size": "2.56 mb"
  }, 
  "change": {
    "size_bytes": 1191560, 
    "size": "1.14 mb", 
    "freed_nodes": 129, 
    "allocated_nodes": 20028,
    "details": [
      {
        "what": "Array", 
        "size_bytes": 323720, 
        "size": "316.13 kb", 
        "+": 15,
        "-": 65
      }, 
      {
        "what": "Code", 
        "size_bytes": -10944,
        "size": "-10.69 kb",
        "+": 8,
        "-": 28 
      },
      {
        "what": "String",
        "size_bytes": 879424,
        "size": "858.81 kb",
        "+": 20001,
        "-": 1
      } 
    ]
  }
}
```

change节点下的freed_nodes和allocated_nodes，它们记录了 释放的节点数量和分配的节点数量。这里由于有内存泄漏，分配的节点数量远远多余释放的节点 数量。在details下可以看到具体每种类型的分配和释放数量。

加号和减号分别表示分配和释放的字符串对象数量。

### 大内存应用

**Node提供了stream模块用于处理大文件**

stream模块是Node的原生模块，直接引用即可。stream继承自EventEmitter，具备基本的自 定义事件功能，同时抽象出标准的事件和方法。它分可读和可写两种。Node中的大多数模块都有 stream的应用，比如fs的createReadStream()和createWriteStream()方法可以分别用于创建文件 的可读流和可写流，process模块中的stdin和stdout则分别是可读流和可写流的示例。

## 第六章 理解Buffer

### Buffer 结构

Buffer是一个像Array的对象，但它主要用于操作字节。

#### 模块结构

Buffer是一个典型的JavaScript与C++结合的模块，它将性能相关部分用C++实现，将非性能相关的部分用JavaScript实现

![image-20191220145535868](https://tva1.sinaimg.cn/large/006tNbRwgy1ga370u7oazj30ja09ewf3.jpg)

Node在进程启动时就已经加载了它，并将其放在全局对象(global) 上。所以在使用Buffer时，无须通过require()即可直接使用

#### Buffer 对象

Buffer对象类似于数组，它的元素为16进制的两位数，即0到255的数值。

```javascript
var str = "深入浅出node.js";
var buf = new Buffer(str, 'utf-8');
console.log(buf);
// => <Buffer e6 b7 b1 e5 85 a5 e6 b5 85 e5 87 ba 6e 6f 64 65 2e 6a 73>
```

不同编码的字符串占用的元素个数各不相同，上面代码中的中文字在 UTF-8编码下占用3个元素，字母和半角标点符号占用1个元素。

Buffer可以访问length属性得到长度，也可以通过下标访问元素，在构造对象时也与Array相似。

```javascript
var buf = new Buffer(100); 
console.log(buf.length); // => 100
console.log(buf[10]);//会得到一个比较奇怪的结果，它的元素值是一个0到255的随机值。
```

如果给元素赋值不是0到255的整数而是小数时会怎样呢?

```javascript
buf[20] = -100;
console.log(buf[20]); // 156 
buf[21] = 300; 
console.log(buf[21]); // 44 
buf[22] = 3.1415; 
console.log(buf[22]); // 3
```

给元素的赋值如果小于0，就将该值逐次加256，直到得到一个0到255之间的整数。如果得到 的数值大于255，就逐次减256，直到得到0~255区间内的数值。如果是小数，舍弃小数部分，只 保留整数部分。

#### Buffer 内存分配

Buffer对象的内存分配不是在V8的堆内存中，而是在Node的C++层面实现内存的申请的。因 为处理大量的字节数据不能采用需要一点内存就向操作系统申请一点内存的方式，这可能造成大 量的内存申请的系统调用，对操作系统有一定压力。为此Node在内存的使用上应用的是在C++ 层面申请内存、在JavaScript中分配内存的策略。

Node采用了slab分配机制，slab是一种动态内存管理机制。slab就是一块申请好的固定大小的内存区域。slab具有如下3种状态。 

* full:完全分配状态。
* partial:部分分配状态。
* empty:没有被分配状态。

```javascript
new Buffer(size);
```

Node以8 KB为界限来区分Buffer是大对象还是小对象。8 KB的值也就是每个slab的大小值，在JavaScript层面，以它作为单位单元进行内存的分配。

##### 1. 分配小Buffer对象

如果指定Buffer的大小少于8 KB，Node会按照小对象的方式进行分配。Buffer的分配过程中主要使用一个局部变量pool作为中间处理对象，处于分配状态的slab单元都指向它

```javascript
var pool;
function allocPool() {
  pool = new SlowBuffer(Buffer.poolSize); 
  pool.used = 0;
}
```

![image-20191220200835681](https://tva1.sinaimg.cn/large/006tNbRwgy1ga3g2jgvmhj30x408ot97.jpg)

**slab处于empty状态。**

```javascript
new Buffer(1024);
```

这次构造将会去检查pool对象，如果pool没有被创建，将会创建一个新的slab单元指向它:

```javascript
if (!pool || pool.length - pool.used < this.length) allocPool();
```

同时当前Buffer对象的parent属性指向该slab，并记录下是从这个slab的哪个位置(offset) 开始使用的，slab对象自身也记录被使用了多少字节

```javascript
this.parent = pool;
this.offset = pool.used;
pool.used += this.length;
if (pool.used & 7) pool.used = (pool.used + 8) & ~7;
```

![image-20191220201258838](https://tva1.sinaimg.cn/large/006tNbRwgy1ga3g75sbkjj30w80ccmy1.jpg)

**这时候的slab状态为partial。**

当再次创建一个Buffer对象时，构造过程中将会判断这个slab的剩余空间是否足够。如果足 够，使用剩余空间，并更新slab的分配状态。如果slab剩余的空间不够，将会构造新的slab，原slab中剩余的空间会造成浪费。

由于同一个slab可能分配给多个Buffer对象使用，只有这些小Buffer对 象在作用域释放并都可以回收时，slab的8 KB空间才会被回收。尽管创建了1个字节的Buffer对象， 但是如果不释放它，实际可能是8 KB的内存没有释放。

##### 2. 分配大Buffer对象

如果需要超过8 KB的Buffer对象，将会直接分配一个SlowBuffer对象作为slab单元，这个slab 单元将会被这个大Buffer对象独占。

```javascript
// Big buffer, just alloc one
this.parent = new SlowBuffer(this.length); this.offset = 0;
```

### Buffer 的转换

Buffer对象可以与字符串之间相互转换。目前支持的字符串编码类型有如下这几种。

* ASCII
* UTF-8
* UTF-16LE/UCS-2
* Base64
* Binary
* Hex

#### 字符串转Buffer

字符串转Buffer对象主要是通过构造函数完成的

```javascript
new Buffer(str, [encoding]);
```

通过构造函数转换的Buffer对象，存储的只能是一种编码类型。encoding参数不传递时，默认按UTF-8编码进行转码和存储。

一个Buffer对象可以存储不同编码类型的字符串转码的值，调用write()方法可以实现该目的

```javascript
buf.write(string, [offset], [length], [encoding])
```

由于可以不断写入内容到Buffer对象中，并且每次写入可以指定编码，所以Buffer对象中可 以存在多种编码转化后的内容。需要小心的是，每种编码所用的字节长度不同，将Buffer反转回 字符串时需要谨慎处理。

#### Buffer 转字符串

Buffer对象的toString()可以将Buffer对象转换为字符串

```javascript
buf.toString([encoding], [start], [end])
```

可以设置encoding(默认为UTF-8)、start、end这3个参数实现整体或局部的转换。如果Buffer对象由多种编码写入，就需要在局部指定不同的编码，才能转换回正常的编码。

#### Buffer 不支持的编码类型

Buffer提供了一个isEncoding()函数来判断编码是否支持转换

```javascript
Buffer.isEncoding(encoding)
```

将编码类型作为参数传入上面的函数，如果支持转换返回值为true，否则为false。

### Buffer 的拼接

Buffer在使用场景中，通常是以一段一段的方式传输。

#### 乱码是如何产生的

```javascript
var fs = require('fs');
var rs = fs.createReadStream('test.md');
var data = '';
rs.on("data", function (chunk){ 
  data += chunk; });
rs.on("end", function () { 
  console.log(data);
});
```

`data += chunk;`这句代码里隐藏了toString()操作，等价于`data = data.toString() + chunk.toString();`

```javascript
var rs = fs.createReadStream('test.md', {highWaterMark: 11});
```

搭配该代码的测试数据为李白的《静夜思》。执行该程序，将会得到以下输出:

```text
床前明���光，疑���地上霜;举头���明月，���头思故乡。
```

产生这个输出结果的原因在于文件可读流在读取时会逐个读取Buffer。这首诗的原始Buffer应存储为:

```javascript
<Buffer e5 ba 8a e5 89 8d e6 98 8e e6 9c 88 e5 85 89 ef bc 8c e7 96 91 e6 98 af e5 9c b0 e4 b8 8a e9 9c 9c ef bc 9b e4 b8 be e5 a4 b4 e6 9c 9b e6 98 8e e6 9c 88 ...>
```

由于我们限定了Buffer对象的长度为11，因此只读流需要读取7次才能完成完整的读取，结果 是以下几个Buffer对象依次输出:

```javascript
<Buffer e5 ba 8a e5 89 8d e6 98 8e e6 9c> 
<Buffer 88 e5 85 89 ef bc 8c e7 96 91 e6> 
...
```

buf.toString()方法默认以UTF-8为编码，中文字在UTF-8下占3个字节。所以第 一个Buffer对象在输出时，只能显示3个字符，Buffer中剩下的2个字节(e6 9c)将会以乱码的形 式显示。第二个Buffer对象的第一个字节也不能形成文字，只能显示乱码。于是形成一些文字无 法正常显示的问题。

在这个示例中我们构造了11这个限制，但是对于任意长度的Buffer而言，宽字节字符串都有 可能存在被截断的情况，只不过Buffer的长度越大出现的概率越低而已，但该问题依然不可忽视。

#### setEncoding()与 string_decoder()

可读流还有一个设置编码的方法setEncoding()，该方法的作用是让data事件中传递的不再是一个Buffer对象，而是编码后的字符串。

```javascript
readable.setEncoding(encoding)
```

```javascript
var rs = fs.createReadStream('test.md', { highWaterMark: 11});
rs.setEncoding('utf8');
```

重新执行程序，得到输出:

`床前明月光，疑是地上霜;举头望明月，低头思故乡。`

**设置编码并未改变按 段读取的基本方式。**在调用setEncoding()时，可读流对象在内部设置了一个decoder对象。每次data事 件都通过该decoder对象进行Buffer到字符串的解码，然后传递给调用者。置编码后，data 不再收到原始的Buffer对象。

decoder对象来自于string_decoder 模块StringDecoder的实例对象。StringDecoder在得到编码后，知道宽字节字符串在UTF-8编码下是 以3个字节的方式存储的，所以第一次write()时，只输出前9个字节转码形成的字符，“月”字的 前两个字节被保留在StringDecoder实例内部。第二次write()时，会将这2个剩余字节和后续11 个字节组合在一起，再次用3的整数倍字节进行转码。于是乱码问题通过这种中间形式被解决了。

**string_decoder目前只能处理UTF-8、Base64和 UCS-2/UTF-16LE这3种编码。**

#### 正确拼接Buffer

```javascript
var chunks = [];
var size = 0;
res.on('data', function (chunk) {
  chunks.push(chunk);
  size += chunk.length;
})
res.on('end', function(){
  var buf = Buffer.concat(chunks, size); 
  var str = iconv.decode(buf, 'utf8');
  console.log(str);
})
```

正确的拼接方式是用一个数组来存储接收到的所有Buffer片段并记录下所有片段的总长度， 然后调用Buffer.concat()方法生成一个合并的Buffer对象。Buffer.concat()方法封装了从小 Buffer对象向大Buffer对象的复制过程，实现十分细腻

```javascript
Buffer.concat = function(list, length) { 
  if (!Array.isArray(list)) {
    throw new Error('Usage: Buffer.concat(list, [length])'); 
  }
  if (list.length === 0) { 
    return new Buffer(0);
  } else if (list.length === 1) { 
    return list[0];
  }
  if (typeof length !== 'number') {
    length = 0;
    for (var i = 0; i < list.length; i++) {
      var buf = list[i];
      length += buf.length; 
    }
  }
  var buffer = new Buffer(length);
  var pos = 0;
  for (var i = 0; i < list.length; i++) {
    var buf = list[i]; 
    buf.copy(buffer, pos); 
    pos += buf.length;
  }
  return buffer; 
};
```

### Buffer 与性能

一旦在网络中传输，都需要转换为Buffer，以进行二进制数据传输。

通过预先转换静态内容为Buffer对象，可以有效地减少CPU的重复使用，节省服务器资源。 在Node构建的Web应用中，可以选择将页面中的动态内容和静态内容分离，静态内容部分可以通 过预先转换为Buffer的方式，使性能得到提升。由于文件自身是二进制数据，所以在不需要改变 内容的场景下，尽量只读取Buffer，然后直接传输，不做额外的转换，避免损耗。

#### 文件读取

在文件的读取时，有一个highWaterMark设置对性能的影响至关重要。

在fs.createReadStream(path, opts)时，我们可以传入一些参数

```javascript
{
  flags: 'r',
  encoding: null,
  fd: null,
  mode: 0666, 
  highWaterMark: 64 * 1024
}
```

还可以传递start和end来指定读取文件的位置范围

```javascript
{start: 90, end: 99}
```

fs.createReadStream()的工作方式是在内存中准备一段Buffer，然后在fs.read()读取时逐步 从磁盘中将字节复制到Buffer中。完成一次读取时，则从这个Buffer中通过slice()方法取出部分 数据作为一个小Buffer对象，再通过data事件传递给调用方。如果Buffer用完，则重新分配一个; 如果还有剩余，则继续使用。

分配一个新的Buffer对象的操作:

```javascript
var pool;
function allocNewPool(poolSize) { 
  pool = new Buffer(poolSize); 
  pool.used = 0;
}
```

在理想的状况下，每次读取的长度就是用户指定的highWaterMark。但是有可能读到了文件结尾，或者文件本身就没有指定的highWaterMark那么大，这个预先指定的Buffer对象将会有部分 剩余，不过好在这里的内存可以分配给下次读取时使用。pool是常驻内存的，只有当pool单元剩 余数量小于128(kMinPoolSpace)字节时，才会重新分配一个新的Buffer对象。

highWaterMark的大小对性能有两个影响的点

* highWaterMark设置对Buffer内存的分配和使用有一定影响。
* highWaterMark设置过小，可能导致系统调用次数过多。

文件流读取基于Buffer分配，Buffer则基于SlowBuffer分配

由于fs.createReadStream()内部采用fs.read()实现，将会引起对磁盘的系统调用，对于大 文件而言，highWaterMark的大小决定会触发系统调用和data事件的次数。

读取一个相同的大文件时，highWaterMark值的大小与读取速 度的关系:该值越大，读取速度越快。