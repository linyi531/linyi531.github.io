---
title: Promise
date: 2019-08-05 19:44:02
tags:
  - JavaScript
  - ES6
categories: ES6
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hlxxwk7j31900u01kz.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hlxxwk7j31900u01kz.jpg
---

# promise

## 创建 XHR 的 promise 对象（Promise 包装 XHR[ajax]处理）

方法一：

创建一个用 Promise 把 XHR 处理包装起来的名为 `getURL` 的函数。

<!-- more -->

```javascript
function getURL(URL) {
  return new Promise(function(resolve, reject) {
    var req = new XMLHttpRequest();
    req.open("GET", URL, true);
    req.onload = function() {
      if (req.status === 200) {
        resolve(req.responseText);
      } else {
        reject(new Error(req.statusText));
      }
    };
    req.onerror = function() {
      reject(new Error(req.statusText));
    };
    req.send();
  });
}
// 运行示例
var URL = "http://httpbin.org/get";
getURL(URL)
  .then(function onFulfilled(value) {
    console.log(value);
  })
  .catch(function onRejected(error) {
    console.error(error);
  });
```

方法二：

```javascript
function ajaxPromise(url, data, callback) {
  return new Promise(function(resolve, reject) {
    $.ajax({
      url: url,
      type: data == null ? "GET" : "POST",
      dataType: "json",
      data: data == null ? "" : JSON.stringify(data),
      async: true,
      contentType: "application/json",
      success: function(res) {
        callback(res);
        resolve();
      },
      error: function(XMLHttpRequest, textStatus, errorThrown) {
        if (XMLHttpRequest.status == "401") {
          window.parent.location = "/enterprise/enterprise_login.html";
          self.location = "/enterprise/enterprise_login.html";
        } else {
          alert(XMLHttpRequest.responseText);
        }
        reject();
      }
    });
  });
}
```

调用

```javascript
ajaxPromise('/prefix/entity1/action1',null, function(res){
     //do something on response
}).then(
     ajaxPromise('/prefix/entity2/action2', someData, function(res){
          //do something on response
     }
).then(
     initVue() ;
).then(
     //do  something else
)
```

## promise 的实现和原理（用 js）

[promise 实现原理](https://juejin.im/post/5b83cb5ae51d4538cc3ec354)

```javascript
// 判断变量否为function
const isFunction = variable => typeof variable === "function";
// 定义Promise的三种状态常量
const PENDING = "PENDING";
const FULFILLED = "FULFILLED";
const REJECTED = "REJECTED";

class MyPromise {
  constructor(handle) {
    if (!isFunction(handle)) {
      throw new Error("MyPromise must accept a function as a parameter");
    }
    // 添加状态
    this._status = PENDING;
    // 添加状态
    this._value = undefined;
    // 添加成功回调函数队列
    this._fulfilledQueues = [];
    // 添加失败回调函数队列
    this._rejectedQueues = [];
    // 执行handle
    try {
      handle(this._resolve.bind(this), this._reject.bind(this));
    } catch (err) {
      this._reject(err);
    }
  }
  // 添加resovle时执行的函数
  _resolve(val) {
    const run = () => {
      if (this._status !== PENDING) return;
      // 依次执行成功队列中的函数，并清空队列
      const runFulfilled = value => {
        let cb;
        while ((cb = this._fulfilledQueues.shift())) {
          cb(value);
        }
      };
      // 依次执行失败队列中的函数，并清空队列
      const runRejected = error => {
        let cb;
        while ((cb = this._rejectedQueues.shift())) {
          cb(error);
        }
      };
      /* 如果resolve的参数为Promise对象，则必须等待该Promise对象状态改变后,
          当前Promsie的状态才会改变，且状态取决于参数Promsie对象的状态 */
      if (val instanceof MyPromise) {
        val.then(
          value => {
            this._value = value;
            this._status = FULFILLED;
            runFulfilled(value);
          },
          err => {
            this._value = err;
            this._status = REJECTED;
            runRejected(err);
          }
        );
      } else {
        this._value = val;
        this._status = FULFILLED;
        runFulfilled(val);
      }
    };
    // 为了支持同步的Promise，这里采用异步调用
    setTimeout(run, 0);
  }
  // 添加reject时执行的函数
  _reject(err) {
    if (this._status !== PENDING) return;
    // 依次执行失败队列中的函数，并清空队列
    const run = () => {
      this._status = REJECTED;
      this._value = err;
      let cb;
      while ((cb = this._rejectedQueues.shift())) {
        cb(err);
      }
    };
    // 为了支持同步的Promise，这里采用异步调用
    setTimeout(run, 0);
  }
  // 添加then方法
  then(onFulfilled, onRejected) {
    const { _value, _status } = this;
    // 返回一个新的Promise对象
    return new MyPromise((onFulfilledNext, onRejectedNext) => {
      // 封装一个成功时执行的函数
      let fulfilled = value => {
        try {
          if (!isFunction(onFulfilled)) {
            onFulfilledNext(value);
          } else {
            let res = onFulfilled(value);
            if (res instanceof MyPromise) {
              // 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调
              res.then(onFulfilledNext, onRejectedNext);
            } else {
              //否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
              onFulfilledNext(res);
            }
          }
        } catch (err) {
          // 如果函数执行出错，新的Promise对象的状态为失败
          onRejectedNext(err);
        }
      };
      // 封装一个失败时执行的函数
      let rejected = error => {
        try {
          if (!isFunction(onRejected)) {
            onRejectedNext(error);
          } else {
            let res = onRejected(error);
            if (res instanceof MyPromise) {
              // 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调
              res.then(onFulfilledNext, onRejectedNext);
            } else {
              //否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
              onFulfilledNext(res);
            }
          }
        } catch (err) {
          // 如果函数执行出错，新的Promise对象的状态为失败
          onRejectedNext(err);
        }
      };
      switch (_status) {
        // 当状态为pending时，将then方法回调函数加入执行队列等待执行
        case PENDING:
          this._fulfilledQueues.push(fulfilled);
          this._rejectedQueues.push(rejected);
          break;
        // 当状态已经改变时，立即执行对应的回调函数
        case FULFILLED:
          fulfilled(_value);
          break;
        case REJECTED:
          rejected(_value);
          break;
      }
    });
  }
  // 添加catch方法
  catch(onRejected) {
    return this.then(undefined, onRejected);
  }
  // 添加静态resolve方法
  static resolve(value) {
    // 如果参数是MyPromise实例，直接返回这个实例
    if (value instanceof MyPromise) return value;
    return new MyPromise(resolve => resolve(value));
  }
  // 添加静态reject方法
  static reject(value) {
    return new MyPromise((resolve, reject) => reject(value));
  }
  // 添加静态all方法
  static all(list) {
    return new MyPromise((resolve, reject) => {
      /**
       * 返回值的集合
       */
      let values = [];
      let count = 0;
      for (let [i, p] of list.entries()) {
        // 数组参数如果不是MyPromise实例，先调用MyPromise.resolve
        this.resolve(p).then(
          res => {
            values[i] = res;
            count++;
            // 所有状态都变成fulfilled时返回的MyPromise状态就变成fulfilled
            if (count === list.length) resolve(values);
          },
          err => {
            // 有一个被rejected时返回的MyPromise状态就变成rejected
            reject(err);
          }
        );
      }
    });
  }
  // 添加静态race方法
  static race(list) {
    return new MyPromise((resolve, reject) => {
      for (let p of list) {
        // 只要有一个实例率先改变状态，新的MyPromise的状态就跟着改变
        this.resolve(p).then(
          res => {
            resolve(res);
          },
          err => {
            reject(err);
          }
        );
      }
    });
  }
  finally(cb) {
    return this.then(
      value => MyPromise.resolve(cb()).then(() => value),
      reason =>
        MyPromise.resolve(cb()).then(() => {
          throw reason;
        })
    );
  }
}
```

## promise.race 和超时处理

函数 `timeoutPromise(比较对象promise, ms)` 接收两个参数，第一个是需要使用超时机制的 promise 对象，第二个参数是超时时间，它返回一个由 `Promise.race` 创建的相互竞争的 promise 对象。

```javascript
function delayPromise(ms) {
  return new Promise(function(resolve) {
    setTimeout(resolve, ms);
  });
}
function timeoutPromise(promise, ms) {
  var timeout = delayPromise(ms).then(function() {
    throw new Error("Operation timed out after " + ms + " ms");
  });
  return Promise.race([promise, timeout]);
}
```

例子：

```javascript
function delayPromise(ms) {
  return new Promise(function(resolve) {
    setTimeout(resolve, ms);
  });
}
function timeoutPromise(promise, ms) {
  var timeout = delayPromise(ms).then(function() {
    throw new Error("Operation timed out after " + ms + " ms");
  });
  return Promise.race([promise, timeout]);
}
// 运行示例
var taskPromise = new Promise(function(resolve) {
  // 随便一些什么处理
  var delay = Math.random() * 2000;
  setTimeout(function() {
    resolve(delay + "ms");
  }, delay);
});
timeoutPromise(taskPromise, 1000)
  .then(function(value) {
    console.log("taskPromise在规定时间内结束 : " + value);
  })
  .catch(function(error) {
    console.log("发生超时", error);
  });
```

虽然在发生超时的时候抛出了异常，但是这样的话我们就不能区分这个异常到底是*普通的错误*还是*超时错误*了。

为了能区分这个 `Error` 对象的类型，我们再来定义一个`Error` 对象的子类 `TimeoutError`。

- #### 定制 Error 对象

```javascript
//TimeoutError.js
function copyOwnFrom(target, source) {
  Object.getOwnPropertyNames(source).forEach(function(propName) {
    Object.defineProperty(
      target,
      propName,
      Object.getOwnPropertyDescriptor(source, propName)
    );
  });
  return target;
}
function TimeoutError() {
  var superInstance = Error.apply(null, arguments);
  copyOwnFrom(this, superInstance);
}
TimeoutError.prototype = Object.create(Error.prototype);
TimeoutError.prototype.constructor = TimeoutError;
```

我们定义了 `TimeoutError` 类和构造函数，这个类继承了 Error 的 prototype。

它的使用方法和普通的 `Error` 对象一样，使用 `throw` 语句即可，如下所示。

```javascript
var promise = new Promise(function() {
  throw TimeoutError("timeout");
});

promise.catch(function(error) {
  console.log(error instanceof TimeoutError); // true
});
```

- #### 应用

取消 XHR 操作本身的话并不难，只需要调用 `XMLHttpRequest` 对象的 `abort()` 方法就可以了。

为了能在外部调用 `abort()` 方法，我们先对之前本节出现的 [`getURL`](http://liubin.org/promises-book/#xhr-promise.js) 进行简单的扩展，`cancelableXHR` 方法除了返回一个包装了 XHR 的 promise 对象之外，还返回了一个用于取消该 XHR 请求的`abort`方法。

大体的流程就像下面这样。

1. 通过 `cancelableXHR` 方法取得包装了 XHR 的 promise 对象和取消该 XHR 请求的方法
2. 在 `timeoutPromise` 方法中通过 `Promise.race` 让 XHR 的包装 promise 和超时用 promise 进行竞争。
   - XHR 在超时前返回结果的话
     1. 和正常的 promise 一样，通过 `then` 返回请求结果
   - 发生超时的时候
     1. 抛出 `throw TimeoutError` 异常并被 `catch`
     2. catch 的错误对象如果是 `TimeoutError` 类型的话，则调用 `abort` 方法取消 XHR 请求

```javascript
function copyOwnFrom(target, source) {
  Object.getOwnPropertyNames(source).forEach(function(propName) {
    Object.defineProperty(
      target,
      propName,
      Object.getOwnPropertyDescriptor(source, propName)
    );
  });
  return target;
}
function TimeoutError() {
  var superInstance = Error.apply(null, arguments);
  copyOwnFrom(this, superInstance);
}
TimeoutError.prototype = Object.create(Error.prototype);
TimeoutError.prototype.constructor = TimeoutError;
function delayPromise(ms) {
  return new Promise(function(resolve) {
    setTimeout(resolve, ms);
  });
}
function timeoutPromise(promise, ms) {
  var timeout = delayPromise(ms).then(function() {
    return Promise.reject(
      new TimeoutError("Operation timed out after " + ms + " ms")
    );
  });
  return Promise.race([promise, timeout]);
}
function cancelableXHR(URL) {
  var req = new XMLHttpRequest();
  var promise = new Promise(function(resolve, reject) {
    req.open("GET", URL, true);
    req.onload = function() {
      if (req.status === 200) {
        resolve(req.responseText);
      } else {
        reject(new Error(req.statusText));
      }
    };
    req.onerror = function() {
      reject(new Error(req.statusText));
    };
    req.onabort = function() {
      reject(new Error("abort this request"));
    };
    req.send();
  });
  var abort = function() {
    // 如果request还没有结束的话就执行abort
    // https://developer.mozilla.org/en/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest
    if (req.readyState !== XMLHttpRequest.UNSENT) {
      req.abort();
    }
  };
  return {
    promise: promise,
    abort: abort
  };
}
var object = cancelableXHR("http://httpbin.org/get");
// main
timeoutPromise(object.promise, 1000)
  .then(function(contents) {
    console.log("Contents", contents);
  })
  .catch(function(error) {
    if (error instanceof TimeoutError) {
      object.abort();
      return console.log(error);
    }
    console.log("XHR Error :", error);
  });
```

## promise.all 和顺序处理

- 在 [重复使用多个 then 的方法](http://liubin.org/promises-book/#multiple-xhr.js) 中的实现方法如下。

```javascript
function getURL(URL) {
  return new Promise(function(resolve, reject) {
    var req = new XMLHttpRequest();
    req.open("GET", URL, true);
    req.onload = function() {
      if (req.status === 200) {
        resolve(req.responseText);
      } else {
        reject(new Error(req.statusText));
      }
    };
    req.onerror = function() {
      reject(new Error(req.statusText));
    };
    req.send();
  });
}
var request = {
  comment: function getComment() {
    return getURL("http://azu.github.io/promises-book/json/comment.json").then(
      JSON.parse
    );
  },
  people: function getPeople() {
    return getURL("http://azu.github.io/promises-book/json/people.json").then(
      JSON.parse
    );
  }
};
function main() {
  function recordValue(results, value) {
    results.push(value);
    return results;
  }
  // [] 用来保存初始化的值
  var pushValue = recordValue.bind(null, []);
  return request
    .comment()
    .then(pushValue)
    .then(request.people)
    .then(pushValue);
}
// 运行示例
main()
  .then(function(value) {
    console.log(value);
  })
  .catch(function(error) {
    console.error(error);
  });
```

使用这种写法的话那么随着 `request` 中元素数量的增加，我们也需要不断增加对 `then` 方法的调用

- 将处理内容统一放到数组里，再配合 for 循环进行处理

```javascript
function getURL(URL) {
  return new Promise(function(resolve, reject) {
    var req = new XMLHttpRequest();
    req.open("GET", URL, true);
    req.onload = function() {
      if (req.status === 200) {
        resolve(req.responseText);
      } else {
        reject(new Error(req.statusText));
      }
    };
    req.onerror = function() {
      reject(new Error(req.statusText));
    };
    req.send();
  });
}
var request = {
  comment: function getComment() {
    return getURL("http://azu.github.io/promises-book/json/comment.json").then(
      JSON.parse
    );
  },
  people: function getPeople() {
    return getURL("http://azu.github.io/promises-book/json/people.json").then(
      JSON.parse
    );
  }
};
function main() {
  function recordValue(results, value) {
    results.push(value);
    return results;
  }
  // [] 用来保存初始化值
  var pushValue = recordValue.bind(null, []);
  // 返回promise对象的函数的数组
  var tasks = [request.comment, request.people];
  var promise = Promise.resolve();
  // 开始的地方
  for (var i = 0; i < tasks.length; i++) {
    var task = tasks[i];
    promise = promise.then(task).then(pushValue);
  }
  return promise;
}
// 运行示例
main()
  .then(function(value) {
    console.log(value);
  })
  .catch(function(error) {
    console.error(error);
  });
```

`promise = promise.then(task).then(pushValue);` 的代码就是通过不断对 promise 进行处理，不断的覆盖 `promise` 变量的值，以达到对 promise 对象的累积处理效果。

但是这种方法需要 `promise` 这个临时变量，从代码质量上来说显得不那么简洁。

- Promise chain 和 reduce

```javascript
function getURL(URL) {
  return new Promise(function(resolve, reject) {
    var req = new XMLHttpRequest();
    req.open("GET", URL, true);
    req.onload = function() {
      if (req.status === 200) {
        resolve(req.responseText);
      } else {
        reject(new Error(req.statusText));
      }
    };
    req.onerror = function() {
      reject(new Error(req.statusText));
    };
    req.send();
  });
}
var request = {
  comment: function getComment() {
    return getURL("http://azu.github.io/promises-book/json/comment.json").then(
      JSON.parse
    );
  },
  people: function getPeople() {
    return getURL("http://azu.github.io/promises-book/json/people.json").then(
      JSON.parse
    );
  }
};
function main() {
  function recordValue(results, value) {
    results.push(value);
    return results;
  }
  var pushValue = recordValue.bind(null, []);
  var tasks = [request.comment, request.people];
  return tasks.reduce(function(promise, task) {
    return promise.then(task).then(pushValue);
  }, Promise.resolve());
}
// 运行示例
main()
  .then(function(value) {
    console.log(value);
  })
  .catch(function(error) {
    console.error(error);
  });
```

`Array.prototype.reduce` 的第二个参数用来设置盛放计算结果的初始值。在这个例子中， `Promise.resolve()` 会赋值给 `promise`，此时的 `task` 为 `request.comment` 。

在 reduce 中第一个参数中被 `return` 的值，则会被赋值为下次循环时的 `promise` 。也就是说，通过返回由 `then` 创建的新的 promise 对象，就实现了和 for 循环类似的 [Promise chain](http://liubin.org/promises-book/#promise-chain) 了。

使用 reduce 和 for 循环不同的地方是 reduce 不再需要临时变量 `promise` 了，因此也不用编写 `promise = promise.then(task).then(pushValue);` 这样冗长的代码了，这是非常大的进步。

- 定义进行顺序处理的函数

```javascript
function sequenceTasks(tasks) {
  function recordValue(results, value) {
    results.push(value);
    return results;
  }
  var pushValue = recordValue.bind(null, []);
  return tasks.reduce(function(promise, task) {
    return promise.then(task).then(pushValue);
  }, Promise.resolve());
}
```

需要注意的一点是，和 `Promise.all` 等不同，这个函数接收的参数是一个函数的数组。为什么传给这个函数的不是一个 promise 对象的数组呢？这是因为 promise 对象创建的时候，XHR 已经开始执行了，因此再对这些 promise 对象进行顺序处理的话就不能正常工作了。

重写上例：

```javascript
function sequenceTasks(tasks) {
  function recordValue(results, value) {
    results.push(value);
    return results;
  }
  var pushValue = recordValue.bind(null, []);
  return tasks.reduce(function(promise, task) {
    return promise.then(task).then(pushValue);
  }, Promise.resolve());
}
function getURL(URL) {
  return new Promise(function(resolve, reject) {
    var req = new XMLHttpRequest();
    req.open("GET", URL, true);
    req.onload = function() {
      if (req.status === 200) {
        resolve(req.responseText);
      } else {
        reject(new Error(req.statusText));
      }
    };
    req.onerror = function() {
      reject(new Error(req.statusText));
    };
    req.send();
  });
}
var request = {
  comment: function getComment() {
    return getURL("http://azu.github.io/promises-book/json/comment.json").then(
      JSON.parse
    );
  },
  people: function getPeople() {
    return getURL("http://azu.github.io/promises-book/json/people.json").then(
      JSON.parse
    );
  }
};
function main() {
  return sequenceTasks([request.comment, request.people]);
}
// 运行示例
main()
  .then(function(value) {
    console.log(value);
  })
  .catch(function(error) {
    console.error(error);
  });
```

## 如何实现 Promise.all ?

```javascript
Promise.all = function(promises) {
  return new Promise((resolve, reject) => {
    let index = 0;
    let result = [];
    if (promises.length === 0) {
      resolve(result);
    } else {
      function processValue(i, data) {
        result[i] = data;
        if (++index === promises.length) {
          resolve(result);
        }
      }
      for (let i = 0; i < promises.length; i++) {
        //promises[i] 可能是普通值
        Promise.resolve(promises[i]).then(
          data => {
            processValue(i, data);
          },
          err => {
            reject(err);
            return;
          }
        );
      }
    }
  });
};
```

## 如何实现 Promise.finally ?

不管成功还是失败，都会走到 finally 中,并且 finally 之后，还可以继续 then。并且会将值原封不动的传递给后面的 then.

```javascript
Promise.prototype.finally = function(callback) {
  return this.then(
    value => {
      return Promise.resolve(callback()).then(() => {
        return value;
      });
    },
    err => {
      return Promise.resolve(callback()).then(() => {
        throw err;
      });
    }
  );
};
```

## 常见问题

1. 输出结果：success

   解题思路：Promise 状态一旦改变，无法在发生变更。

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("success");
    reject("error");
  }, 1000);
});
promise.then(
  res => {
    console.log(res);
  },
  err => {
    console.log(err);
  }
);
```

2. 输出结果：1

   解题思路：Promise 的 then 方法的参数期望是函数，传入非函数则会发生值穿透。

```javascript
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log);
```

3. 结果是：3 4 6 8 7 5 2 1

   优先级关系如下：`process.nextTick > promise.then > setTimeout > setImmediate`

```javascript
setImmediate(function() {
  console.log(1);
}, 0);
setTimeout(function() {
  console.log(2);
}, 0);
new Promise(function(resolve) {
  console.log(3);
  resolve();
  console.log(4);
}).then(function() {
  console.log(5);
});
console.log(6);
process.nextTick(function() {
  console.log(7);
});
console.log(8);
```

V8 实现中，两个队列各包含不同的任务：

`macrotasks: script(整体代码),setTimeout, setInterval, setImmediate, I/O, UI rendering`
`microtasks: process.nextTick, Promises, Object.observe, MutationObserver`

4. 实现一个简单的 Promise

   ```javascript
   function Promise(fn) {
     var status = "pending";
     function successNotify() {
       status = "fulfilled"; //状态变为fulfilled
       toDoThen.apply(undefined, arguments); //执行回调
     }
     function failNotify() {
       status = "rejected"; //状态变为rejected
       toDoThen.apply(undefined, arguments); //执行回调
     }
     function toDoThen() {
       setTimeout(() => {
         // 保证回调是异步执行的
         if (status === "fulfilled") {
           for (let i = 0; i < successArray.length; i++) {
             successArray[i].apply(undefined, arguments); //执行then里面的回掉函数
           }
         } else if (status === "rejected") {
           for (let i = 0; i < failArray.length; i++) {
             failArray[i].apply(undefined, arguments); //执行then里面的回掉函数
           }
         }
       });
     }
     var successArray = [];
     var failArray = [];
     fn.call(undefined, successNotify, failNotify);
     return {
       then: function(successFn, failFn) {
         successArray.push(successFn);
         failArray.push(failFn);
         return undefined; // 此处应该返回一个Promise
       }
     };
   }
   ```

解题思路：Promise 中的 resolve 和 reject 用于改变 Promise 的状态和传参，then 中的参数必须是作为回调执行的函数。因此，当 Promise 改变状态之后会调用回调函数，根据状态的不同选择需要执行的回调函数。
