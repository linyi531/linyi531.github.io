---
title: 循环中的异步操作问题小结
date: 2019-11-26 13:29:24
tags:
  - JavaScript
categories: JavaScript
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g9bdv3db80j31900u0hdt.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g9bdv3db80j31900u0hdt.jpg
---

# 循环中的异步操作问题小结

循环的异步操作主要有两个问题：

- 如何确保循环的所有异步操作完成之后执行某个其他操作
- 循环中的下一步操作依赖于前一步的操作，如何解决

## 不需等待结果

要处理这个问题，我们可以把这个匿名函数定义为异步的：

```javascript
async function processArray(array){
  array.forEach(async (item)=>{
    await delayedLog(item)
  })
  console.log('Done!')
}
```

但是这样的话 forEach 方法就相当于异步的了，不会等待遍历完所有的 item 将会输出：

```text
Done!
1
2
3
```

如果你不需要等待这个循环完成，这样就已经可以了。但是大部分情况我们还是需要等待这个循环完成才进行之后的操作。

## 如何确保循环的所有异步操作完成之后执行某个其他操作

### 方法一：设置一个flag，在每个异步操作中对flag进行检测

```javascript
let flag = 0;
for(let i = 0; i < len; i++) {
  flag++;
  Database.save_method().exec().then((data) => {
      if(flag === len) {
            // your code
      }
  })
}
```

### 方法二：将所有的循环放在一个promise中，使用then处理

```javascript
 new Promise(function(resolve){
      resolve()
 }).then(()=> {
     for(let i = 0; i < len; i++) {
           Database.save_method().exec()
     }
}).then(() => {
    // your code
})
```

### 方法三：串行遍历

要等待所有的结果返回，我们还是要回到老式的 for 循环写法：

```javascript
async function processArray(array){
  for(const item of array){
    await delayedLog(item)
  }
  console.log('Done!')
}
```

最后的结果符合我们的预期：

```text
1
2
3
Done!
```

### 方法四：并行遍历

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g7c606vfgvj30tg081mxe.jpg)

```javascript
async function processArray(array){
  //map array to promise
  const promises=array.map(delayedLog)
  await Promise.all(promises)
  console.log('Done!')
}
```

（注意：对于特别大的数组不建议使用这种写法，太多的并行任务会加重 CPU 和内存的负荷）

## 循环中的下一步操作依赖于前一步的操作，如何解决

### 方法一：使用递归，在异步操作完成之后调用下一次异步操作

```javascript
function loop(i){
  i++;
  Database.save_method().exec().then(() => {
      loop(i)
    })
}
```

### 方法二：使用async和await（串行遍历）

```javascript
async function loop() {
    for(let i = 0; i < len; i++) {
         await Database.save_method().exec();
    }
}
```

# 如何优雅地写js异步循环

## 循环的方式

假设我们有个数组，包含 5 个数字：`let times = [100, 150, 200, 250, 300]`； 
还有一个异步的睡觉方法：`sleep(time, cb)`。

```javascript
import Promise from 'bluebird';

// 当没有 cb 时，返回一个 Promise 对象
export default function sleep(time, cb) {  
    if (cb) {
        setTimeout(cb, time);
    } else {
        return new Promise(resolve => {
            setTimeout(resolve, time);
        });
    }
};
```

现在要去循环睡这几个数字，问你有哪些睡法？🤔

为了方便交流，我就给这几个睡法起个名字：

1. All in：你如果赶时间又不担心消耗过度，你可以一次性都睡了；
2. One by one：你想细水长流，你可以一个一个睡；
3. With concurrency：你害羞地低下头，说一次能不能睡两个。

> 作为一段有节操的代码，肯定要告诉其他人你睡完了，也就是必须有全部完成的回调，否则我们接下来的交流会毫无意义。

本文目的是和大家探讨如何写出优雅的异步循环代码，并不是去实现这些循环控制的逻辑；而保持代码优雅，个人以为最好的办法是使用较新的语言特性，其次是使用优秀的开源项目，最后才是自己撸。下面会使用 [Async](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fcaolan%2Fasync)、[Promise(bluebird)](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fpetkaantonov%2Fbluebird) 和 ES7 中的 `async/await` 对比下实现这几种循环的区别。

### All in

![All in](https://tva1.sinaimg.cn/large/006y8mN6ly1g7c606vfgvj30tg081mxe.jpg)

这种方式效率是最高的，耗时取决于循环中最慢的那个异步方法。对资源的消耗也是最大的，如果大量循环请求后端服务，很有可能造成瞬时拥堵的情况。

如果自己实现，这也是最简单的场景，加一个完成计数器，每个异步方法完就给这个完成计数器加 1，然后检查完成数是不是等于数组长度，一旦相等就表示所有的异步方法执行完毕，通知全部完成的回调。

#### 使用 async.each：

```javascript
import { each } from 'async';  
import sleep from './sleep';

let times = [100, 150, 200, 250, 300];

console.log('sleep start');  
console.time('async all in');  
each(times, sleep, (err) => {  
    console.timeEnd('async all in');
    console.log('sleep complete');
});
// sleep start
// async all in: 304.627ms
// sleep complete
```

#### 使用 Promise.all：

```javascript
import Promise from 'bluebird';  
import sleep from './sleep';

let times = [100, 150, 200, 250, 300];

console.log('sleep start');  
console.time('promise all in');  
Promise.all(times.map(time => sleep(time))).then(() => {  
    console.timeEnd('promise all in');
    console.log('sleep complete');
});
// sleep start
// promise all in: 305.509ms
// sleep complete
```

#### 使用ES7 async/await：

```javascript
import sleep from './sleep';

let times = [100, 150, 200, 250, 300];

(async function() {
    console.log('sleep start');
    console.time('es7 all in');
    for await (let i of times.map(time => sleep(time))) {}
    console.timeEnd('es7 all in');
    console.log('sleep complete');
}());
// sleep start
// es7 all in: 305.986ms
// sleep complete
```

### One by one

![One by one](https://tva1.sinaimg.cn/large/006y8mN6ly1g7c65nlncwj30tg04374g.jpg)

这种方式效率最低，有点类似于同步语言中的循环，一个接着一个执行，耗时自然也就是所有异步方法耗时的总和。对资源的消耗最小。

这个实现起来也比较简单，把数组看做一个队列，每次从队列`shift`出一个代入异步方法执行，执行完成就开始递归调用这个过程，当队列长度为空就表示所有的异步方法执行完毕，结束递归，通知全部完成的回调。

#### 使用 async.eachSeries：

```javascript
import { eachSeries } from 'async';  
import sleep from './sleep';

let times = [100, 150, 200, 250, 300];

console.log('sleep start');  
console.time('async one by one');  
eachSeries(times, sleep, (err) => {  
    console.timeEnd('async one by one');
    console.log('sleep complete');
});
// sleep start
// async one by one: 1020.078ms
// sleep complete
```

#### 使用 Promise.reduce：

```javascript
import Promise from 'bluebird';  
import sleep from './sleep';

let times = [100, 150, 200, 250, 300];

console.log('sleep start');  
console.time('promise one by one');  
Promise.reduce(times, (last, curr) => {  
    return sleep(curr);
}, 0).then(() => {
    console.timeEnd('promise one by one');
    console.log('sleep complete');
});
// sleep start
// promise one by one: 1023.014ms
// sleep complete
```

#### 使用ES7 async/await：

```javascript
import sleep from './sleep';

let times = [100, 150, 200, 250, 300];

(async function() {
    console.log('sleep start');
    console.time('es7 one by one');
    for (let time of times) {
        await sleep(time);
    }
    console.timeEnd('es7 one by one');
    console.log('sleep complete');
}());
// sleep start
// es7 one by one: 1025.513ms
// sleep complete
```

### With concurrency

这种方式稍微复杂些，但也是最灵活的方式，可以随心控制并发数。效率和耗时取决于魔法数字 `concurrency`，当 `concurrency` 大于或等于数组长度时，它就等同于 **All in** 方式；当 `concurrency` 为 1 时，它就等同于 **One by one** 方式。所以耗时和对资源的消耗都会介于以上两种方式之间。

**With concurrency** 本身在实现上也会有不同的方式，分别是预分组和任务池。

#### 预分组

![Pre Group](https://tva1.sinaimg.cn/large/006y8mN6ly1g7c66v9wx3j30tg05074j.jpg)

顾名思义，就是提前将数组内容按 `concurrency` 分好组，组内是以 **All in** 方式执行，组之间则是以 **One by one** 的方式执行。

就以上文的例子，假如 `concurrency` 为 2，`times` 预先分组成：`[[100, 150], [200, 250], [300]]`，这样耗时会是 700（150 + 250 + 300）。

这个实现方式可以有效地控制并发数，优点就是简单，缺点是并不能达到效率最大化。

#### 任务池

![Task Pool](https://tva1.sinaimg.cn/large/006y8mN6ly1g7c67an02oj30tg066q38.jpg)

任务池的方式就是设置一个容量为 `concurrency` 的池子，比如容量为 2，初始化放入两个任务，每当有任务完成，就继续往池子添加新的任务，直到所有任务都完成。上文的例子执行过程大致如下：

1. `time = 0; pool = [100, 150]`：放入 `100` 和 `150`
2. `time = 100; pool = [150, 200]`：`100` 结束，放入 `200`
3. `time = 150; pool = [200, 250]`：`150` 结束，放入 `250`
4. `time = 300; pool = [250, 300]`：`200` 结束，放入 `300`
5. `time = 400; pool = [300]`：`250` 结束，没有更多任务
6. `time = 600; pool = []`：`300` 结束，循环完毕

得出来的耗时是 600，比预分组的方式效率更高，而且同样能有效控制并发个数。async 和 bluebird 也有相关的方法供直接使用。

#### 使用 async.eachLimit：

```javascript
import { eachLimit } from 'async';  
import sleep from './sleep';

let times = [100, 150, 200, 250, 300];

console.log('sleep start');  
console.time('async with concurrency');  
eachLimit(times, 2, sleep, (err) => {  
    console.timeEnd('async with concurrency');
    console.log('sleep complete');
});
// sleep start
// async with concurrency: 611.498ms
// sleep complete
```

#### 使用 Promise.map（bluebird 特有 api）：

```javascript
import Promise from 'bluebird';  
import sleep from './sleep';

let times = [100, 150, 200, 250, 300];

console.log('sleep start');  
console.time('promise one by one');  
Promise.map(times, (time) => {  
    return sleep(time);
}, {
    concurrency: 2
}).then(() => {
    console.timeEnd('promise one by one');
    console.log('sleep complete');
});
// sleep start
// promise with concurrency: 616.601ms
// sleep complete
```

#### 使用ES7 async/await：

> `pool` 方法来自[davetemplin/async-parallel](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fdavetemplin%2Fasync-parallel%2Fblob%2Fmaster%2Findex.ts%23L153)

```javascript
import sleep from './sleep';

let times = [100, 150, 200, 250, 300];

(async function() {
    console.log('sleep start');
    console.time('es7 with concurrency');
    await pool(2, async () => {
        await sleep(times.shift());
        return times.length > 0;
    });
    console.timeEnd('es7 with concurrency');
    console.log('sleep complete');
}());

async function pool(size, task) {  
    var active = 0;
    var done = false;
    var errors = [];
    return new Promise((resolve, reject) => {
        next();
        function next() {
            while (active < size && !done) {
                active += 1;
                task()
                    .then(more => {
                        if (--active === 0 && (done || !more))
                            errors.length === 0 ? resolve() : reject(errors);
                        else if (more)
                            next();
                        else
                            done = true;
                    })
                    .catch(err => {
                        errors.push(err);
                        done = true;
                        if (--active === 0)
                            reject(errors);
                    });
            }
        }
    });
}
// sleep start
// es7 with concurrency: 612.197ms
// sleep complete
```

## 总结

好了，到这应该可以给这三种循环方式打下分了：

| **循环方式**         | 效率 | 消耗 | 灵活度 | 复杂度 |
| -------------------- | ---- | ---- | ------ | ------ |
| **All in**           | 高   | 高   | 低     | 低     |
| **One by one**       | 低   | 低   | 低     | 低     |
| **With concurrency** | 中   | 中   | 高     | 高     |

乍一看 **With concurrency** 是完胜，其实并没有。**All in** 和 **One by one** 虽然灵活度低，但是应用的场景还是非常广泛的。要求效率优先就使用 **All in**；如果有下一次循环依赖上一次循环结果的场景，就必须使用 **One by One**。

再说下上面 async、bluebird、ES7 对这三种循环方式的实现。需求一直在变，async 需要修改的代码非常少，甚至只要改下方法名就可以，方法定义简单优雅，这可能也是 async 易上手的原因；bluebird 在 `Promise` 标准基础上添加的方法非常实用，如：map、join...，以至于我几乎是没有使用过原生 `Promise` 😂；ES7 新增的 `async/await` 语法特性确实减轻了编写异步代码的痛苦，同时还增强了代码的可读性。

