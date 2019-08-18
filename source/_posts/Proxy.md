---
title: Proxy
date: 2018-10-27 11:07:46
tags:
  - ES6
  - 前端
categories: ES6
cover_img: https://i.screenshot.net/zmxj2sj
feature_img: https://i.screenshot.net/zmxj2sj
---

# 概述

ES6 原生提供 Proxy 构造函数，用来生成 Proxy 实例。

```javascript
var proxy = new Proxy(target, handler);
```

Proxy 对象的所有用法，都是上面这种形式，不同的只是 handler 参数的写法。其中，new Proxy()表示生成一个 Proxy 实例，target 参数表示所要拦截的目标对象，handler 参数也是一个对象，用来定制拦截行为。

<!-- more -->

下面是一个拦截读取属性行为的例子。

```javascript
var proxy = new Proxy(
  {},
  {
    get: function(target, property) {
      return 35;
    }
  }
);

proxy.time; // 35
proxy.name; // 35
proxy.title; // 35
```

上面代码中，作为构造函数，Proxy 接受两个参数。第一个参数是所要代理的目标对象（上例是一个空对象），即如果没有 Proxy 的介入，操作原来要访问的就是这个对象；第二个参数是一个配置对象，对于每一个被代理的操作，需要提供一个对应的处理函数，该函数将拦截对应的操作。比如，上面代码中，配置对象有一个 get 方法，用来拦截对目标对象属性的访问请求。get 方法的两个参数分别是目标对象和所要访问的属性。可以看到，由于拦截函数总是返回 35，所以访问任何属性都得到 35。<font color="#DC143C">注意，要使得 Proxy 起作用，必须针对 Proxy 实例（上例是 proxy 对象）进行操作，而不是针对目标对象（上例是空对象）进行操作。</font>如果 handler 没有设置任何拦截，那就等同于直接通向原对象。
同一个拦截器函数，可以设置拦截多个操作。

```javascript
var handler = {
  get: function(target, name) {
    if (name === "prototype") {
      return Object.prototype;
    }
    return "Hello, " + name;
  },

  apply: function(target, thisBinding, args) {
    return args[0];
  },

  construct: function(target, args) {
    return { value: args[1] };
  }
};

var fproxy = new Proxy(function(x, y) {
  return x + y;
}, handler);

fproxy(1, 2); // 1
new fproxy(1, 2); // {value: 2}
fproxy.prototype === Object.prototype; // true
fproxy.foo === "Hello, foo"; // true
```

# Proxy 支持的拦截操作一览

Proxy 支持的拦截操作一览，一共 13 种。

## get(target, propKey, receiver)

拦截对象属性的读取，比如 proxy.foo 和 proxy['foo']。

## set(target, propKey, value, receiver)

拦截对象属性的设置，比如 proxy.foo = v 或 proxy['foo'] = v，返回一个布尔值。

## has(target, propKey)

拦截 propKey in proxy 的操作，返回一个布尔值。

## deleteProperty(target, propKey)

拦截 delete proxy[propKey]的操作，返回一个布尔值。

## ownKeys(target)

拦截 Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in 循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而 Object.keys()的返回结果仅包括目标对象自身的可遍历属性。

## getOwnPropertyDescriptor(target, propKey)

拦截 Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。

## defineProperty(target, propKey, propDesc)

拦截 Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。

## preventExtensions(target)

拦截 Object.preventExtensions(proxy)，返回一个布尔值。

## getPrototypeOf(target)

拦截 Object.getPrototypeOf(proxy)，返回一个对象。

## isExtensible(target)

拦截 Object.isExtensible(proxy)，返回一个布尔值。

## setPrototypeOf(target, proto)

拦截 Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。

## apply(target, object, args)

拦截 Proxy 实例作为函数调用的操作，比如 proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。

## construct(target, args)

拦截 Proxy 实例作为构造函数调用的操作，比如 new proxy(...args)。

# Proxy 实例的方法

下面是上面这些拦截方法的详细介绍。

## get()

get 方法用于拦截某个属性的读取操作，可以接受三个参数，依次为目标对象、属性名和 proxy 实例本身（严格地说，它总是指向原始的读操作所在的那个对象，一般情况下就是 Proxy 实例），其中最后一个参数可选。

```javascript
var person = {
  name: "张三"
};

var proxy = new Proxy(person, {
  get: function(target, property) {
    if (property in target) {
      return target[property];
    } else {
      throw new ReferenceError('Property "' + property + '" does not exist.');
    }
  }
});

proxy.name; // "张三"
proxy.age; // 抛出一个错误
```

get 方法可以继承。

```javascript
let proto = new Proxy(
  {},
  {
    get(target, propertyKey, receiver) {
      console.log("GET " + propertyKey);
      return target[propertyKey];
    }
  }
);

let obj = Object.create(proto);
obj.foo; // "GET foo"
```

上面代码中，拦截操作定义在 Prototype 对象上面，所以如果读取 obj 对象继承的属性时，拦截会生效。
利用 Proxy，可以将读取属性的操作（get），转变为执行某个函数，从而实现属性的链式操作。

```javascript
var pipe = (function() {
  return function(value) {
    var funcStack = [];
    var oproxy = new Proxy(
      {},
      {
        get: function(pipeObject, fnName) {
          if (fnName === "get") {
            return funcStack.reduce(function(val, fn) {
              return fn(val);
            }, value);
          }
          funcStack.push(window[fnName]);
          return oproxy;
        }
      }
    );

    return oproxy;
  };
})();

var double = n => n * 2;
var pow = n => n * n;
var reverseInt = n =>
  n
    .toString()
    .split("")
    .reverse()
    .join("") | 0;

pipe(3).double.pow.reverseInt.get; // 63
```

下面是一个 get 方法的第三个参数的例子

```javascript
const proxy = new Proxy(
  {},
  {
    get: function(target, property, receiver) {
      return receiver;
    }
  }
);

const d = Object.create(proxy);
d.a === d; // true
```

上面代码中，d 对象本身没有 a 属性，所以读取 d.a 的时候，会去 d 的原型 proxy 对象找。这时，receiver 就指向 d，代表原始的读操作所在的那个对象。
如果一个属性不可配置（configurable）且不可写（writable），则 Proxy 不能修改该属性，否则通过 Proxy 对象访问该属性会报错。

```javascript
const target = Object.defineProperties(
  {},
  {
    foo: {
      value: 123,
      writable: false,
      configurable: false
    }
  }
);

const handler = {
  get(target, propKey) {
    return "abc";
  }
};

const proxy = new Proxy(target, handler);

proxy.foo;
// TypeError: Invariant check failed
```

## set()

set 方法用来拦截某个属性的赋值操作，可以接受四个参数，依次为目标对象、属性名、属性值和 Proxy 实例本身，其中最后一个参数可选。
利用 set 方法，可以进行数据验证，还可以数据绑定，即每当对象发生变化时，会自动更新 DOM。
有时，我们会在对象上面设置内部属性，属性名的第一个字符使用下划线开头，表示这些属性不应该被外部使用。结合 get 和 set 方法，就可以做到防止这些内部属性被外部读写。

```javascript
const handler = {
  get(target, key) {
    invariant(key, "get");
    return target[key];
  },
  set(target, key, value) {
    invariant(key, "set");
    target[key] = value;
    return true;
  }
};
function invariant(key, action) {
  if (key[0] === "_") {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}
const target = {};
const proxy = new Proxy(target, handler);
proxy._prop;
// Error: Invalid attempt to get private "_prop" property
proxy._prop = "c";
// Error: Invalid attempt to set private "_prop" property
```

上面代码中，只要读写的属性名的第一个字符是下划线，一律抛错，从而达到禁止读写内部属性的目的。
下面是 set 方法第四个参数的例子。

```javascript
const handler = {
  set: function(obj, prop, value, receiver) {
    obj[prop] = receiver;
  }
};
const proxy = new Proxy({}, handler);
const myObj = {};
Object.setPrototypeOf(myObj, proxy);

myObj.foo = "bar";
myObj.foo === myObj; // true
```

上面代码中，设置 myObj.foo 属性的值时，myObj 并没有 foo 属性，因此引擎会到 myObj 的原型链去找 foo 属性。myObj 的原型对象 proxy 是一个 Proxy 实例，设置它的 foo 属性会触发 set 方法。这时，第四个参数 receiver 就指向原始赋值行为所在的对象 myObj。
如果目标对象自身的某个属性，不可写且不可配置，那么 set 方法将不起作用。

```javascript
const obj = {};
Object.defineProperty(obj, "foo", {
  value: "bar",
  writable: false
});

const handler = {
  set: function(obj, prop, value, receiver) {
    obj[prop] = "baz";
  }
};

const proxy = new Proxy(obj, handler);
proxy.foo = "baz";
proxy.foo; // "bar"
```

## apply()

apply 方法拦截函数的调用、call 和 apply 操作。apply 方法可以接受三个参数，分别是目标对象、目标对象的上下文对象（this）和目标对象的参数数组。

```javascript
var target = function() {
  return "I am the target";
};
var handler = {
  apply: function() {
    return "I am the proxy";
  }
};

var p = new Proxy(target, handler);

p();
// "I am the proxy"
```

上面代码中，变量 p 是 Proxy 的实例，当它作为函数调用时（p()），就会被 apply 方法拦截，返回一个字符串。
另外，直接调用 Reflect.apply 方法，也会被拦截。

## has()

has 方法用来拦截 HasProperty 操作，即判断对象是否具有某个属性时，这个方法会生效。典型的操作就是 in 运算符。has 方法可以接受两个参数，分别是目标对象、需查询的属性名。
下面的例子使用 has 方法隐藏某些属性，不被 in 运算符发现。

```javascript
var handler = {
  has(target, key) {
    if (key[0] === "_") {
      return false;
    }
    return key in target;
  }
};
var target = { _prop: "foo", prop: "foo" };
var proxy = new Proxy(target, handler);
"_prop" in proxy; // false
```

如果原对象不可配置或者禁止扩展，这时 has 拦截会报错。

```javascript
var obj = { a: 10 };
Object.preventExtensions(obj);

var p = new Proxy(obj, {
  has: function(target, prop) {
    return false;
  }
});

"a" in p; // TypeError is thrown
```

上面代码中，obj 对象禁止扩展，结果使用 has 拦截就会报错。也就是说，如果某个属性不可配置（或者目标对象不可扩展），则 has 方法就不得“隐藏”（即返回 false）目标对象的该属性。
has 方法拦截的是 HasProperty 操作，而不是 HasOwnProperty 操作，即 has 方法不判断一个属性是对象自身的属性，还是继承的属性。另外，虽然 for...in 循环也用到了 in 运算符，但是 has 拦截对 for...in 循环不生效。

## construct()

construct 方法用于拦截 new 命令，下面是拦截对象的写法。

```javascript
var handler = {
  construct(target, args, newTarget) {
    return new target(...args);
  }
};
```

construct 方法可以接受三个参数。target：目标对象。args：构造函数的参数对象。newTarget：创造实例对象时，new 命令作用的构造函数。
construct 方法返回的必须是一个对象，否则会报错。

```javascript
var p = new Proxy(function() {}, {
  construct: function(target, args) {
    console.log("called: " + args.join(", "));
    return { value: args[0] * 10 };
  }
});

new p(1).value;
// "called: 1"
// 10
```

```javascript
var p = new Proxy(function() {}, {
  construct: function(target, argumentsList) {
    return 1;
  }
});

new p(); // 报错
```

## deleteProperty()

deleteProperty 方法用于拦截 delete 操作，如果这个方法抛出错误或者返回 false，当前属性就无法被 delete 命令删除。

```javascript
var handler = {
  deleteProperty(target, key) {
    invariant(key, "delete");
    return true;
  }
};
function invariant(key, action) {
  if (key[0] === "_") {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}

var target = { _prop: "foo" };
var proxy = new Proxy(target, handler);
delete proxy._prop;
// Error: Invalid attempt to delete private "_prop" property
```

deleteProperty 方法拦截了 delete 操作符，删除第一个字符为下划线的属性会报错。
注意，目标对象自身的不可配置（configurable）的属性，不能被 deleteProperty 方法删除，否则报错。

## defineProperty()

defineProperty 方法拦截了 Object.defineProperty 操作。

```javascript
var handler = {
  defineProperty(target, key, descriptor) {
    return false;
  }
};
var target = {};
var proxy = new Proxy(target, handler);
proxy.foo = "bar"; // 不会生效
```

defineProperty 方法返回 false，导致添加新属性总是无效。
注意，如果目标对象不可扩展（extensible），则 defineProperty 不能增加目标对象上不存在的属性，否则会报错。另外，如果目标对象的某个属性不可写（writable）或不可配置（configurable），则 defineProperty 方法不得改变这两个设置。

## getOwnPropertyDescriptor()

getOwnPropertyDescriptor 方法拦截 Object.getOwnPropertyDescriptor()，返回一个属性描述对象或者 undefined。

```javascript
var handler = {
  getOwnPropertyDescriptor(target, key) {
    if (key[0] === "_") {
      return;
    }
    return Object.getOwnPropertyDescriptor(target, key);
  }
};
var target = { _foo: "bar", baz: "tar" };
var proxy = new Proxy(target, handler);
Object.getOwnPropertyDescriptor(proxy, "wat");
// undefined
Object.getOwnPropertyDescriptor(proxy, "_foo");
// undefined
Object.getOwnPropertyDescriptor(proxy, "baz");
// { value: 'tar', writable: true, enumerable: true, configurable: true }
```

handler.getOwnPropertyDescriptor 方法对于第一个字符为下划线的属性名会返回 undefined。

## getPrototypeOf()

getPrototypeOf 方法主要用来拦截获取对象原型。具体来说，拦截下面这些操作。

1. Object.prototype.\_\_proto\_\_
2. Object.prototype.isPrototypeOf()
3. Object.getPrototypeOf()
4. Reflect.getPrototypeOf()
5. instanceof

```javascript
var proto = {};
var p = new Proxy(
  {},
  {
    getPrototypeOf(target) {
      return proto;
    }
  }
);
Object.getPrototypeOf(p) === proto; // true
```

getPrototypeOf 方法的返回值必须是对象或者 null，否则报错。另外，如果目标对象不可扩展（extensible）， getPrototypeOf 方法必须返回目标对象的原型对象。

## isExtensible()

isExtensible 方法拦截 Object.isExtensible 操作

```javascript
var p = new Proxy(
  {},
  {
    isExtensible: function(target) {
      console.log("called");
      return true;
    }
  }
);

Object.isExtensible(p);
// "called"
// true
```

该方法只能返回布尔值，否则返回值会被自动转为布尔值。
这个方法有一个强限制，它的返回值必须与目标对象的 isExtensible 属性保持一致，否则就会抛出错误。

```javascript
var p = new Proxy(
  {},
  {
    isExtensible: function(target) {
      return false;
    }
  }
);

Object.isExtensible(p); // 报错
Object.isExtensible(proxy) === Object.isExtensible(target); //true
```

## ownKeys()

ownKeys 方法用来拦截对象自身属性的读取操作。具体来说，拦截以下操作。

1. Object.getOwnPropertyNames()
2. Object.getOwnPropertySymbols()
3. Object.keys()
4. for...in 循环

```javascript
let target = {
  a: 1,
  b: 2,
  c: 3
};

let handler = {
  ownKeys(target) {
    return ["a"];
  }
};

let proxy = new Proxy(target, handler);

Object.keys(proxy);
// [ 'a' ]
```

注意，使用 Object.keys 方法时，有三类属性会被 ownKeys 方法自动过滤，不会返回。

1. 目标对象上不存在的属性
2. 属性名为 Symbol 值
3. 不可遍历（enumerable）的属性

```javascript
let target = {
  a: 1,
  b: 2,
  c: 3,
  [Symbol.for("secret")]: "4"
};

Object.defineProperty(target, "key", {
  enumerable: false,
  configurable: true,
  writable: true,
  value: "static"
});

let handler = {
  ownKeys(target) {
    return ["a", "d", Symbol.for("secret"), "key"];
  }
};

let proxy = new Proxy(target, handler);

Object.keys(proxy);
// ['a']
```

上面代码中，ownKeys 方法之中，显式返回不存在的属性（d）、Symbol 值（Symbol.for('secret')）、不可遍历的属性（key），结果都被自动过滤掉。

```javascript
//ownKeys方法还可以拦截Object.getOwnPropertyNames()
var p = new Proxy(
  {},
  {
    ownKeys: function(target) {
      return ["a", "b", "c"];
    }
  }
);

Object.getOwnPropertyNames(p);
// [ 'a', 'b', 'c' ]

//for...in循环也受到ownKeys方法的拦截
const obj = { hello: "world" };
const proxy = new Proxy(obj, {
  ownKeys: function() {
    return ["a", "b"];
  }
});

for (let key in proxy) {
  console.log(key); // 没有任何输出
}
```

上面代码中，ownkeys 指定只返回 a 和 b 属性，由于 obj 没有这两个属性，因此 for...in 循环不会有任何输出。
ownKeys 方法返回的数组成员，只能是字符串或 Symbol 值。如果有其他类型的值，或者返回的根本不是数组，就会报错。

```javascript
var obj = {};

var p = new Proxy(obj, {
  ownKeys: function(target) {
    return [123, true, undefined, null, {}, []];
  }
});

Object.getOwnPropertyNames(p);
// Uncaught TypeError: 123 is not a valid property name
```

如果目标对象自身包含不可配置的属性，则该属性必须被 ownKeys 方法返回，否则报错。

```javascript
var obj = {};
Object.defineProperty(obj, "a", {
  configurable: false,
  enumerable: true,
  value: 10
});

var p = new Proxy(obj, {
  ownKeys: function(target) {
    return ["b"];
  }
});

Object.getOwnPropertyNames(p);
// Uncaught TypeError: 'ownKeys' on proxy: trap result did not include 'a'
```

如果目标对象是不可扩展的（non-extensition），这时 ownKeys 方法返回的数组之中，必须包含原对象的所有属性，且不能包含多余的属性，否则报错。

```javascript
var obj = {
  a: 1
};

Object.preventExtensions(obj);

var p = new Proxy(obj, {
  ownKeys: function(target) {
    return ["a", "b"];
  }
});

Object.getOwnPropertyNames(p);
// Uncaught TypeError: 'ownKeys' on proxy: trap returned extra keys but proxy target is non-extensible
```

## preventExtensions()

preventExtensions 方法拦截 Object.preventExtensions()。该方法必须返回一个布尔值，否则会被自动转为布尔值。这个方法有一个限制，只有目标对象不可扩展时（即 Object.isExtensible(proxy)为 false），proxy.preventExtensions 才能返回 true，否则会报错。

```javascript
var p = new Proxy(
  {},
  {
    preventExtensions: function(target) {
      return true;
    }
  }
);

Object.preventExtensions(p); // 报错

//为了防止出现这个问题，通常要在proxy.preventExtensions方法里面，调用一次Object.preventExtensions
var p = new Proxy(
  {},
  {
    preventExtensions: function(target) {
      console.log("called");
      Object.preventExtensions(target);
      return true;
    }
  }
);

Object.preventExtensions(p);
// "called"
// true
```

## setPrototypeOf()

setPrototypeOf 方法主要用来拦截 Object.setPrototypeOf 方法。

```javascript
var handler = {
  setPrototypeOf(target, proto) {
    throw new Error("Changing the prototype is forbidden");
  }
};
var proto = {};
var target = function() {};
var proxy = new Proxy(target, handler);
Object.setPrototypeOf(proxy, proto);
// Error: Changing the prototype is forbidden
```

上面代码中，只要修改 target 的原型对象，就会报错。
注意，该方法只能返回布尔值，否则会被自动转为布尔值。另外，如果目标对象不可扩展（extensible），setPrototypeOf 方法不得改变目标对象的原型。

# Proxy.revocable()

Proxy.revocable 方法返回一个可取消的 Proxy 实例

```javascript
let target = {};
let handler = {};

let { proxy, revoke } = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo; // 123

revoke();
proxy.foo; // TypeError: Revoked
```

Proxy.revocable 方法返回一个对象，该对象的 proxy 属性是 Proxy 实例，revoke 属性是一个函数，可以取消 Proxy 实例。上面代码中，当执行 revoke 函数之后，再访问 Proxy 实例，就会抛出一个错误。
Proxy.revocable 的一个使用场景是，目标对象不允许直接访问，必须通过代理访问，一旦访问结束，就收回代理权，不允许再次访问。

# this 问题

虽然 Proxy 可以代理针对目标对象的访问，但它不是目标对象的透明代理，即不做任何拦截的情况下，也无法保证与目标对象的行为一致。主要原因就是在 Proxy 代理的情况下，目标对象内部的 this 关键字会指向 Proxy 代理。

```javascript
const target = {
  m: function() {
    console.log(this === proxy);
  }
};
const handler = {};

const proxy = new Proxy(target, handler);

target.m(); // false
proxy.m(); // true
```
