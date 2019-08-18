---
title: this和super关键字
date: 2018-09-24 18:41:35
tags:
  - ES6
  - 前端
categories: ES6
---

# this 关键字

this 的指向：

## 作为普通对象的方法调用

作为普通对象的方法调用时，this 指向这个对象本身

<!-- more -->

```javascript
var obj = {
  a: 1,
  getA: function() {
    console.log(this === obj);
    console.log(this.a);
  }
};

//this指向obj对象
obj.getA();
```

## 作为普通函数调用

作为普通函数调用时，this 指向全局对象，在浏览器中全局对象是 window，在 NodeJs 中全局对象是 global。

```javascript
var obj = {
  a: 1,
  getA: function() {
    console.log(this === obj);
    console.log(this.a);
  }
};

//this指向window对象
var getA = obj.getA;
getA();
```

这里需要注意的一点是，直接调用并不是指在全局作用域下进行调用，在任何作用域下，直接通过 函数名(...) 来对函数进行调用的方式，都称为直接调用。

## 构造器调用

构造器调用时，this 指向返回的对象。用 new 调用一个构造函数，会创建一个新对象，而其中的 this 就指向这个新对象。

```javascript
var a = 10;
var b = 20;
function Point(x, y) {
  this.x = x;
  this.y = y;
}

var a = new Point(1, 2);
console.log(a.x); // 1
console.log(x); // 10

var b = new Point(1, 2);
console.log(a === b); // false
```

## call apply bind

当函数通过 call()和 apply()方法绑定时，this 指向两个方法的第一个参数对象上，若第一个参数不是对象，JS 内部会尝试将其转化为对象然后再指向它。
通过 bind 方法绑定后，无论其在什么情况下被调用，函数将被永远绑定在其第一个参数对象上，bind 绑定后返回的是一个函数。

### call, apply 的用途

1. 改变 this 的指向
2. Function.prototype.bind

```javascript
Function.prototype.bind = function() {
  var self = this;
  var context = [].shift.call(arguments);
  var args = [].slice.call(arguments);
  return function() {
    return self.apply(context, args.concat([].slice.call(arguments)));
  };
};
```

### 三者区别

call 只能一个一个传入参数
apply 可直接传入参数数组
bind 会返回一个新的函数

# super 关键字

关键字 super，指向当前对象的原型对象。

```javascript
const proto = {
  foo: "hello"
};

const obj = {
  foo: "world",
  find() {
    return super.foo;
  }
};

Object.setPrototypeOf(obj, proto);
obj.find(); // "hello"
```

上面代码中，对象 obj 的 find 方法之中，通过 super.foo 引用了原型对象 proto 的 foo 属性。
注意，super 关键字表示原型对象时，只能用在对象的方法之中，用在其他地方都会报错。

```javascript
// 报错
const obj = {
  foo: super.foo
};

// 报错
const obj = {
  foo: () => super.foo
};

// 报错
const obj = {
  foo: function() {
    return super.foo;
  }
};
```

上面三种 super 的用法都会报错，因为对于 JavaScript 引擎来说，这里的 super 都没有用在对象的方法之中。第一种写法是 super 用在属性里面，第二种和第三种写法是 super 用在一个函数里面，然后赋值给 foo 属性。目前，只有对象方法的简写法可以让 JavaScript 引擎确认，定义的是对象的方法。
JavaScript 引擎内部，super.foo 等同于 Object.getPrototypeOf(this).foo（属性）或 Object.getPrototypeOf(this).foo.call(this)（方法）。

```javascript
const proto = {
  x: "hello",
  foo() {
    console.log(this.x);
  }
};

const obj = {
  x: "world",
  foo() {
    super.foo();
  }
};

Object.setPrototypeOf(obj, proto);

obj.foo(); // "world"
```

上面代码中，super.foo 指向原型对象 proto 的 foo 方法，但是绑定的 this 却还是当前对象 obj，因此输出的就是 world。
