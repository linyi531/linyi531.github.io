---
title: ES6类以及继承的实现原理
date: 2020-01-17 23:46:51
tags:
  - JavaScript
  - ES6
categories: ES6
cover_img: https://tva1.sinaimg.cn/large/0082zybply1gbpfdpe68uj31900u0x1b.jpg
feature_img: https://tva1.sinaimg.cn/large/0082zybply1gbpfdpe68uj31900u0x1b.jpg
---

# ES6类以及继承的实现原理

## ES6创建一个类的过程

ES6中通过class关键字，定义类

```javascript
class Parent {
    constructor(name,age){
        this.name = name;
        this.age = age;
    }
    speakSomething(){
        console.log("I can speek chinese");
    }
}
```

经过babel转码之后

```javascript
"use strict";

var _createClass = function () {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    }

    return function (Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
}();

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var Parent = function () {
    function Parent(name, age) {
        _classCallCheck(this, Parent);

        this.name = name;
        this.age = age;
    }

    _createClass(Parent, [{
        key: "speakSomething",
        value: function speakSomething() {
            console.log("I can speek chinese");
        }
    }]);

    return Parent;
}();
```

**ES6类的底层还是通过构造函数去创建的**

可见class的底层依然是构造函数：

1. 调用_classCallCheck方法判断当前函数调用前是否有new关键字。

   构造函数执行前有new关键字，会在构造函数内部创建一个空对象，将构造函数的proptype指向这个空对象的_proto_,并将this指向这个空对象。如上，_classCallCheck中：this instanceof Parent 返回true。

   若构造函数前面没有new则构造函数的proptype不会不出现在this的原型链上，返回false。

   通过ES6创建的类，是不允许你直接调用的。在ES5中，构造函数是可以直接运行的，比如`Parent()`。但是在ES6就不行。我们可以看到转码的构造函数中有`_classCallCheck(this, Parent)`语句,这句话是防止你通过构造函数直接运行的。你直接在ES6运行`Parent()`,这是不允许的,ES6中抛出`Class constructor Parent cannot be invoked without 'new'`错误。转码后的会抛出`Cannot call a class as a function`.我觉得这样的规范挺好的，能够规范化类的使用方式。

2. 将class内部的变量和函数赋给this。

   转码中`_createClass`方法，它调用`Object.defineProperty`方法去给新创建的`Parent`添加各种属性。`defineProperties(Constructor.prototype, protoProps)`是给原型添加属性。如果你有静态属性，会直接添加到构造函数上`defineProperties(Constructor, staticProps)`。

3. 执行constuctor内部的逻辑。

4. return this (构造函数默认在最后我们做了)。

## ES6实现继承

**1.调用_inherits函数继承父类的proptype。**

**2.用一个闭包保存父类引用，在闭包内部做子类构造逻辑。**

**3.new检查。**

**4.用当前this调用父类构造函数。**

**5.将行子类class内部的变量和函数赋给this。**

**6.执行子类constuctor内部的逻辑。**

es6实际上是为我们提供了一个“组合寄生继承”的简单写法。

```javascript
class Parent {
    static height = 12
    constructor(name,age){
        this.name = name;
        this.age = age;
    }
    speakSomething(){
        console.log("I can speek chinese");
    }
}
Parent.prototype.color = 'yellow'


//定义子类，继承父类
class Child extends Parent {
    static width = 18
    constructor(name,age){
        super(name,age);
    }
    coding(){
        console.log("I can code JS");
    }
}

var c = new Child("job",30);
c.coding()
```

转码之后的代码变成了这样

```javascript
"use strict";

var _createClass = function () {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    }

    return function (Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
}();

//校验this是否被初始化，super是否调用，并返回父类已经赋值完的this。
function _possibleConstructorReturn(self, call) {
    if (!self) {
        throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
    }
    return call && (typeof call === "object" || typeof call === "function") ? call : self;
}

function _inherits(subClass, superClass) {
    if (typeof superClass !== "function" && superClass !== null) {
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
    }
    subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var Parent = function () {
    function Parent(name, age) {
        _classCallCheck(this, Parent);

        this.name = name;
        this.age = age;
    }

    _createClass(Parent, [{
        key: "speakSomething",
        value: function speakSomething() {
            console.log("I can speek chinese");
        }
    }]);

    return Parent;
}();

Parent.height = 12;

Parent.prototype.color = 'yellow';

//定义子类，继承父类

var Child = function (_Parent) {
    _inherits(Child, _Parent);

    function Child(name, age) {
        _classCallCheck(this, Child);

      /**用当前this调用父类构造函数。
      * 这里的Child.proto || Object.getPrototypeOf(Child)实际上是父构造函数(_inherits最后的操作)，
      * 然后通过call将其调用方改为当前this，并传递参数。（这里感觉可以直接用参数传过来的Parent）*/
        return _possibleConstructorReturn(this, (Child.__proto__ || Object.getPrototypeOf(Child)).call(this, name, age));
    }

    _createClass(Child, [{
        key: "coding",
        value: function coding() {
            console.log("I can code JS");
        }
    }]);

    return Child;
}(Parent);

Child.width = 18;


var c = new Child("job", 30);
c.coding();
```

我们可以看到，构造类的方法都没变，只是添加了`_inherits`核心方法来实现继承

### `_inherits`核心方法

(1) 校验父构造函数。

(2) 典型的寄生继承：用父类构造函数的proptype创建一个空对象，并将这个对象指向子类构造函数的proptype。

(3) 将父构造函数指向子构造函数的_proto_（这步是做什么的不太明确，感觉没什么意义。）

首先是判断父类的类型，然后

```javascript
subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
```

这段代码翻译下来就是

```javascript
function F(){}
F.prototype = superClass.prototype
subClass.prototype = new F()
subClass.prototype.constructor = subClass
```

接下来`subClass.__proto__ = superClass`
`_inherits`核心思想就是下面两句

```javascript
subClass.prototype.__proto__ = superClass.prototype
subClass.__proto__ = superClass
```

首先 `subClass.prototype.__proto__ = superClass.prototype`保证了`c instanceof Parent`是true,Child的实例可以访问到父类的属性，包括内部属性，以及原型属性。其次，`subClass.__proto__ = superClass`，保证了Child.height也能访问到，也就是静态方法。

### super

super代表父类构造函数。

super.fun1() 等同于 Parent.fun1() 或 Parent.prototype.fun1()。

**super() 等同于Parent.prototype.construtor()**

默认的构造函数中会主动调用父类构造函数，并默认把当前constructor传递的参数传给了父类。

所以当我们声明了constructor后必须主动调用super(),否则无法调用父构造函数，无法完成继承。

典型的例子就是Reatc的Component中，我们声明constructor后必须调用super(props)，因为父类要在构造函数中对props做一些初始化操作。