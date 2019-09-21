---
title: 双向数据绑定Proxy和Object.defineProperty对比
date: 2019-03-11 11:52:12
tags:
  - vue
categories: vue
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hankac9j30u0190b2b.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hankac9j30u0190b2b.jpg
---

# 双向数据绑定`Proxy`和`Object.defineProperty`对比

## 数据劫持的优势所在。

对比其他双向绑定的实现方法,数据劫持的优势所在：

1. 无需显示调用: 例如 Vue 运用数据劫持+发布订阅,直接可以通知变化并驱动视图,上面的例子也是比较简单的实现`data.name = '渣渣辉'`后直接触发变更,而比如 Angular 的脏检测则需要显示调用`markForCheck`(可以用 zone.js 避免显示调用,不展开),react 需要显示调用`setState`。
2. 可精确得知变化数据：还是上面的小例子，我们劫持了属性的 setter,当属性值改变,我们可以精确获知变化的内容`newVal`,因此在这部分不需要额外的 diff 操作,否则我们只知道数据发生了变化而不知道具体哪些数据变化了,这个时候需要大量 diff 来找出变化值,这是额外性能损耗。
   <!-- more -->

## 基于数据劫持双向绑定的实现思路

**数据劫持**是双向绑定各种方案中比较流行的一种,最著名的实现就是 Vue。

基于数据劫持的双向绑定离不开`Proxy`与`Object.defineProperty`等方法对对象/对象属性的"劫持",我们要实现一个完整的双向绑定需要以下几个要点。

1. 利用`Proxy`或`Object.defineProperty`生成的 Observer 针对对象/对象的属性进行"劫持",在属性发生变化后通知订阅者
2. 解析器 Compile 解析模板中的`Directive`(指令)，收集指令所依赖的方法和数据,等待数据变化然后进行渲染
3. Watcher 属于 Observer 和 Compile 桥梁,它将接收到的 Observer 产生的数据变化,并根据 Compile 提供的指令进行视图渲染,使得数据变化促使视图变化

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy2rxn2jj310q0jugn4.jpg)

- 在 `new Vue()` 后， Vue 会调用`_init` 函数进行初始化，也就是 init 过程，在 这个过程 Data 通过 Observer 转换成了 getter/setter 的形式，来对数据追踪变化，当被设置的对象被读取的时候会执行`getter` 函数，而在当被赋值的时候会执行 `setter`函数。
- 当 render function 执行的时候，因为会读取所需对象的值，所以会触发`getter`函数从而将 Watcher 添加到依赖中进行依赖收集。
- 在修改对象的值的时候，会触发对应的`setter`， `setter`通知之前**依赖收集**得到的 Dep 中的每一个 Watcher，告诉它们自己的值改变了，需要重新渲染视图。这时候这些 Watcher 就会开始调用 `update` 来更新视图。

## 基于 Object.defineProperty 双向绑定的特点

Vue 通过设定对象属性的 setter/getter 方法来监听数据的变化，通过 getter 进行依赖收集，而每个 setter 方法就是一个观察者，在数据变更的时候通知订阅者更新视图。

**在 getter 中收集依赖，在 setter 中触发依赖。**

当外界通过 Watcher 读取数据时，便会触发 getter 从而将 Watcher 添加到依赖中，哪个 Watcher 触发了 getter，就把哪个 Watcher 收集到 Dep 中。当数据发生变化时，会循环依赖列表，把所有的 Watcher 都通知一遍。

### 极简版的双向绑定

`Object.defineProperty`的作用就是劫持一个对象的属性,通常我们对属性的`getter`和`setter`方法进行劫持,在对象的属性发生变化时进行特定的操作。

我们就对对象`obj`的`text`属性进行劫持,在获取此属性的值时打印`'get val'`,在更改属性值的时候对 DOM 进行操作,这就是一个极简的双向绑定。

```javascript
const obj = {};
Object.defineProperty(obj, "text", {
  get: function() {
    console.log("get val");
  },
  set: function(newVal) {
    console.log("set val:" + newVal);
    document.getElementById("input").value = newVal;
    document.getElementById("span").innerHTML = newVal;
  }
});

const input = document.getElementById("input");
input.addEventListener("keyup", function(e) {
  obj.text = e.target.value;
});
```

### 升级改造

我们很快会发现，这个所谓的*双向绑定*貌似并没有什么乱用。。。

原因如下:

1. 我们只监听了一个属性,一个对象不可能只有一个属性,我们需要对对象每个属性进行监听。
2. 违反开放封闭原则,我们如果了解[开放封闭原则](https://link.juejin.im?target=https%3A%2F%2Fzh.wikipedia.org%2Fzh-hans%2F%E5%BC%80%E9%97%AD%E5%8E%9F%E5%88%99)的话,上述代码是明显违反此原则,我们每次修改都需要进入方法内部,这是需要坚决杜绝的。
3. 代码耦合严重,我们的数据、方法和 DOM 都是耦合在一起的，就是传说中的面条代码。

那么如何解决上述问题？

Vue 的操作就是加入了**发布订阅**模式，结合`Object.defineProperty`的劫持能力，实现了可用性很高的双向绑定。

首先，我们以**发布订阅**的角度看我们第一部分写的那一坨代码,会发现它的*监听*、*发布*和*订阅*都是写在一起的,我们首先要做的就是解耦。

我们先实现**一个订阅发布中心，即消息管理员（Dep）,它负责储存订阅者和消息的分发,不管是订阅者还是发布者都需要依赖于它**。

```javascript
let uid = 0;
// 用于储存订阅者并发布消息
class Dep {
  constructor() {
    // 设置id,用于区分新Watcher和只改变属性值后新产生的Watcher
    this.id = uid++;
    // 储存订阅者的数组
    this.subs = [];
  }
  // 触发target上的Watcher中的addDep方法,参数为dep的实例本身
  depend() {
    Dep.target.addDep(this);
  }
  // 添加订阅者
  addSub(sub) {
    this.subs.push(sub);
  }
  notify() {
    // 通知所有的订阅者(Watcher)，触发订阅者的相应逻辑处理
    this.subs.forEach(sub => sub.update());
  }
}
// 为Dep类设置一个静态属性,默认为null,工作时指向当前的Watcher
Dep.target = null;
```

现在我们需要实现监听者(Observer),用于监听属性值的变化。

```javascript
// 监听者,监听对象属性值的变化
class Observer {
  constructor(value) {
    this.value = value;
    this.walk(value);
  }
  // 遍历属性值并监听
  walk(value) {
    Object.keys(value).forEach(key => this.convert(key, value[key]));
  }
  // 执行监听的具体方法
  convert(key, val) {
    defineReactive(this.value, key, val);
  }
}

function defineReactive(obj, key, val) {
  const dep = new Dep();
  // 给当前属性的值添加监听
  let chlidOb = observe(val);
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: () => {
      // 如果Dep类存在target属性，将其添加到dep实例的subs数组中
      // target指向一个Watcher实例，每个Watcher都是一个订阅者
      // Watcher实例在实例化过程中，会读取data中的某个属性，从而触发当前get方法
      if (Dep.target) {
        dep.depend();
      }
      return val;
    },
    set: newVal => {
      if (val === newVal) return;
      val = newVal;
      // 对新值进行监听
      chlidOb = observe(newVal);
      // 通知所有订阅者，数值被改变了
      dep.notify();
    }
  });
}

function observe(value) {
  // 当值不存在，或者不是复杂数据类型时，不再需要继续深入监听
  if (!value || typeof value !== "object") {
    return;
  }
  return new Observer(value);
}
```

那么接下来就简单了,我们需要实现一个订阅者(Watcher)。

```javascript
class Watcher {
  constructor(vm, expOrFn, cb) {
    this.depIds = {}; // hash储存订阅者的id,避免重复的订阅者
    this.vm = vm; // 被订阅的数据一定来自于当前Vue实例
    this.cb = cb; // 当数据更新时想要做的事情
    this.expOrFn = expOrFn; // 被订阅的数据
    this.val = this.get(); // 维护更新之前的数据
  }
  // 对外暴露的接口，用于在订阅的数据被更新时，由订阅者管理员(Dep)调用
  update() {
    this.run();
  }
  addDep(dep) {
    // 如果在depIds的hash中没有当前的id,可以判断是新Watcher,因此可以添加到dep的数组中储存
    // 此判断是避免同id的Watcher被多次储存
    if (!this.depIds.hasOwnProperty(dep.id)) {
      dep.addSub(this);
      this.depIds[dep.id] = dep;
    }
  }
  run() {
    const val = this.get();
    console.log(val);
    if (val !== this.val) {
      this.val = val;
      this.cb.call(this.vm, val);
    }
  }
  get() {
    // 当前订阅者(Watcher)读取被订阅数据的最新更新后的值时，通知订阅者管理员收集当前订阅者
    Dep.target = this;
    const val = this.vm._data[this.expOrFn];
    // 置空，用于下一个Watcher使用
    Dep.target = null;
    return val;
  }
}
```

那么我们最后完成 Vue,将上述方法挂载在 Vue 上。

```javascript
class Vue {
    constructor(options = {}) {
      // 简化了$options的处理
      this.$options = options;
      // 简化了对data的处理
      let data = (this._data = this.$options.data);
      // 将所有data最外层属性代理到Vue实例上
      Object.keys(data).forEach(key => this._proxy(key));
      // 监听数据
      observe(data);
    }
    // 对外暴露调用订阅者的接口，内部主要在指令中使用订阅者
    $watch(expOrFn, cb) {
      new Watcher(this, expOrFn, cb);
    }
    _proxy(key) {
      Object.defineProperty(this, key, {
        configurable: true,
        enumerable: true,
        get: () => this._data[key],
        set: val => {
          this._data[key] = val;
        },
      });
    }
  }
]
```

至此,一个简单的双向绑定算是被我们实现了。

### Object.defineProperty 的缺陷

- `Object.defineProperty`的第一个缺陷,无法监听数组变化。[Vue 的文档](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Flist.html%23%E6%95%B0%E7%BB%84%E6%9B%B4%E6%96%B0%E6%A3%80%E6%B5%8B)提到了 Vue 是可以检测到数组变化的，但是只有以下八种方法,`vm.items[indexOfItem] = newValue`这种是无法检测的。`push()`、`pop()`、`shift()`、`unshift()`、`splice()`、`sort()`、`reverse()`

- `Object.defineProperty`的第二个缺陷,只能劫持对象的属性,因此我们需要对每个对象的每个属性进行遍历，如果属性值也是对象那么需要深度遍历,显然能劫持一个完整的对象是更好的选择。

  `Object.keys(value).forEach(key => **this**.convert(key, value[key]));`

- **无法检测到对象属性的添加或删除**(如`data.location.a=1`)。

  这是因为 Vue 通过`Object.defineProperty`来将对象的 key 转换成`getter/setter`的形式来追踪变化，但`getter/setter`只能追踪一个数据是否被修改，无法追踪新增属性和删除属性。如果是删除属性，我们可以用`vm.$delete`实现，那如果是新增属性，该怎么办呢？
  1）可以使用 `Vue.set(location, a, 1)` 方法向嵌套对象添加响应式属性;
  2）也可以给这个对象重新赋值，比如`data.location = {...data.location,a:1}`

## Proxy 实现的双向绑定的特点

Proxy 在 ES2015 规范中被正式发布,它在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写,我们可以这样认为,Proxy 是`Object.defineProperty`的全方位加强版

### Proxy 可以直接监听对象而非属性

我们还是以上文中用`Object.defineProperty`实现的极简版双向绑定为例,用 Proxy 进行改写。

```javascript
const input = document.getElementById("input");
const p = document.getElementById("p");
const obj = {};

const newObj = new Proxy(obj, {
  get: function(target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function(target, key, value, receiver) {
    console.log(target, key, value, receiver);
    if (key === "text") {
      input.value = value;
      p.innerHTML = value;
    }
    return Reflect.set(target, key, value, receiver);
  }
});

input.addEventListener("keyup", function(e) {
  newObj.text = e.target.value;
});
```

我们可以看到,Proxy 直接可以劫持整个对象,并返回一个新对象,不管是操作便利程度还是底层功能上都远强于`Object.defineProperty`。

### Proxy 可以直接监听数组的变化

当我们对数组进行操作(push、shift、splice 等)时，会触发对应的方法名称和*length*的变化，我们可以借此进行操作,以上文中`Object.defineProperty`无法生效的列表渲染为例。

```javascript
const list = document.getElementById("list");
const btn = document.getElementById("btn");

// 渲染列表
const Render = {
  // 初始化
  init: function(arr) {
    const fragment = document.createDocumentFragment();
    for (let i = 0; i < arr.length; i++) {
      const li = document.createElement("li");
      li.textContent = arr[i];
      fragment.appendChild(li);
    }
    list.appendChild(fragment);
  },
  // 我们只考虑了增加的情况,仅作为示例
  change: function(val) {
    const li = document.createElement("li");
    li.textContent = val;
    list.appendChild(li);
  }
};

// 初始数组
const arr = [1, 2, 3, 4];

// 监听数组
const newArr = new Proxy(arr, {
  get: function(target, key, receiver) {
    console.log(key);
    return Reflect.get(target, key, receiver);
  },
  set: function(target, key, value, receiver) {
    console.log(target, key, value, receiver);
    if (key !== "length") {
      Render.change(value);
    }
    return Reflect.set(target, key, value, receiver);
  }
});

// 初始化
window.onload = function() {
  Render.init(arr);
};

// push数字
btn.addEventListener("click", function() {
  newArr.push(6);
});
```

很显然,Proxy 不需要那么多 hack（即使 hack 也无法完美实现监听）就可以无压力监听数组的变化,我们都知道,标准永远优先于 hack。

### Proxy 的其他优势

Proxy 有多达 13 种拦截方法,不限于 apply、ownKeys、deleteProperty、has 等等是`Object.defineProperty`不具备的。

Proxy 返回的是一个新对象,我们可以只操作新的对象达到目的,而`Object.defineProperty`只能遍历对象属性直接修改。

Proxy 作为新标准将受到浏览器厂商重点持续的性能优化，也就是传说中的新标准的性能红利。

当然,Proxy 的劣势就是兼容性问题,而且无法用 polyfill 磨平,因此 Vue 的作者才声明需要等到下个大版本(3.0)才能用 Proxy 重写。

### 基础 proxy 的双向数据绑定的实现

#### 发布订阅中心(Dep)

`Dep`保存订阅者,并在 Observer 发生变化时通知保存在 Dep 中的订阅者,让订阅者得知变化并更新视图,这样才能保证视图与状态的同步。

```javascript
/**
 * [subs description] 订阅器,储存订阅者,通知订阅者
 * @type {Map}
 */
export default class Dep {
  constructor() {
    // 我们用 hash 储存订阅者
    this.subs = new Map();
  }
  // 添加订阅者
  addSub(key, sub) {
    // 取出键为 key 的订阅者
    const currentSub = this.subs.get(key);
    // 如果能取出说明有相同的 key 的订阅者已经存在,直接添加
    if (currentSub) {
      currentSub.add(sub);
    } else {
      // 用 Set 数据结构储存,保证唯一值
      this.subs.set(key, new Set([sub]));
    }
  }
  // 通知
  notify(key) {
    // 触发键为 key 的订阅者们
    if (this.subs.get(key)) {
      this.subs.get(key).forEach(sub => {
        sub.update();
      });
    }
  }
}
```

#### 监听者的实现(Observer)

我们在订阅器 `Dep` 中实现了一个`notify`方法来通知相应的订阅这们,然而`notify`方法到底什么时候被触发呢?

当然是当状态发生变化时,即 MVVM 中的 Modal 变化时触发通知,然而`Dep` 显然无法得知 Modal 是否发生了变化,因此我们需要创建一个监听者`Observer`来监听 Modal, 当 Modal 发生变化的时候我们就执行通知操作。

与`Object.defineProperty`监听属性不同, Proxy 可以监听(实际是代理)整个对象,因此就不需要遍历对象的属性依次监听了,但是如果对象的属性依然是个对象,那么 Proxy 也无法监听,所以我们实现了一个`observify`进行递归监听即可。

```javascript
/**
 * [Observer description] 监听器,监听对象,触发后通知订阅
 * @param {[type]}   obj [description] 需要被监听的对象
 */
const Observer = obj => {
  const dep = new Dep();
  return new Proxy(obj, {
    get: function(target, key, receiver) {
      // 如果订阅者存在，直接添加订阅
      if (Dep.target) {
        dep.addSub(key, Dep.target);
      }
      return Reflect.get(target, key, receiver);
    },
    set: function(target, key, value, receiver) {
      // 如果对象值没有变,那么不触发下面的操作直接返回
      if (Reflect.get(receiver, key) === value) {
        return;
      }
      const res = Reflect.set(target, key, observify(value), receiver);
      // 当值被触发更改的时候,触发 Dep 的通知方法
      dep.notify(key);
      return res;
    }
  });
};

/**
 * 将对象转为监听对象
 * @param {*} obj 要监听的对象
 */
export default function observify(obj) {
  if (!isObject(obj)) {
    return obj;
  }

  // 深度监听
  Object.keys(obj).forEach(key => {
    obj[key] = observify(obj[key]);
  });

  return Observer(obj);
}
```

#### 订阅者的实现(watcher)

我们目前已经解决了两个问题,一个是如何得知 Modal 发生了改变(利用监听者 Observer 监听 Modal 对象),一个是如何收集订阅者并通知其变化(利用订阅器收集订阅者,并用 notify 通知订阅者)。

我们目前还差一个订阅者（Watcher）

```javascript
// 订阅者
export default class Watcher {
  constructor(vm, exp, cb) {
    this.vm = vm; // vm 是 vue 的实例
    this.exp = exp; // 被订阅的数据
    this.cb = cb; // 触发更新后的回调
    this.value = this.get(); // 获取老数据
  }
  get() {
    const exp = this.exp;
    let value;
    Dep.target = this;
    if (typeof exp === "function") {
      value = exp.call(this.vm);
    } else if (typeof exp === "string") {
      value = this.vm[exp];
    }
    Dep.target = null;
    return value;
  }
  // 将订阅者放入待更新队列等待批量更新
  update() {
    pushQueue(this);
  }
  // 触发真正的更新操作
  run() {
    const val = this.get(); // 获取新数据
    this.cb.call(this.vm, val, this.value);
    this.value = val;
  }
}
```

#### 批量更新的实现

我们在上一节中实现了订阅者( Watcher),但是其中的`update`方法是将订阅者放入了一个待更新的队列中,而不是直接触发,原因如下:

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy4crpf3j31260l6qbp.jpg)

因此这个队列需要做的是**异步**且**去重**,因此我们用 `Set`作为数据结构储存 Watcher 来去重,同时用`Promise`模拟异步更新。

```javascript
// 创建异步更新队列
let queue = new Set();

// 用Promise模拟nextTick
function nextTick(cb) {
  Promise.resolve().then(cb);
}

// 执行刷新队列
function flushQueue(args) {
  queue.forEach(watcher => {
    watcher.run();
  });
  // 清空
  queue = new Set();
}

// 添加到队列
export default function pushQueue(watcher) {
  queue.add(watcher);
  // 下一个循环调用
  nextTick(flushQueue);
}
```
