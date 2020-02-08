---
title: symbol
date: 2020-01-22 15:20:01
tags:
  - JavaScript
  - ES6
categories: ES6
cover_img: https://tva1.sinaimg.cn/large/0082zybply1gbpfhxuqjij30u0190qvf.jpg
feature_img: https://tva1.sinaimg.cn/large/0082zybply1gbpfhxuqjij30u0190qvf.jpg
---

# symbol

## symbol的特性。

- Symbol 值通过`Symbol`函数生成。这就是说，对象的属性名现在可以有两种类型，一种是原来就有的字符串，另一种就是新增的 Symbol 类型。

- 凡是属性名属于 Symbol 类型，就都是独一无二的，可以保证不会与其他属性名产生冲突。（Symbol 值作为对象属性名时，不能用点运算符。因为点运算符后面总是字符串，所以不会读取`mySymbol`作为标识名所指代的那个值，导致`a`的属性名实际上是一个字符串，而不是一个 Symbol 值。同理，在对象的内部，使用 Symbol 值定义属性时，Symbol 值必须放在方括号之中。）Symbol 值作为属性名时，该属性还是公开属性，不是私有属性。

- 注意，`Symbol`函数前不能使用`new`命令，否则会报错。这是因为生成的 Symbol 是一个原始类型的值，不是对象。也就是说，由于 Symbol 值不是对象，所以不能添加属性。基本上，它是一种类似于字符串的数据类型。

- symbol一旦创建后就不可更改。

- 如果 Symbol 的参数是一个对象，就会调用该对象的`toString`方法，将其转为字符串，然后才生成一个 Symbol 值。`Symbol`函数的参数只是表示对当前 Symbol 值的描述，因此相同参数的`Symbol`函数的返回值是不相等的。

- Symbol 值不能与其他类型的值进行运算，会报错。（Symbol 不能自动被转换为字符串，当尝试将一个 Symbol 强制转换为字符串时，将返回一个 TypeError。）

- Symbol 值可以显式转为字符串。

  ```javascript
  let sym = Symbol('My symbol');
  
  String(sym) // 'Symbol(My symbol)'
  sym.toString() // 'Symbol(My symbol)'
  ```

  另外，Symbol 值也可以转为布尔值，但是不能转为数值。

  ```javascript
  let sym = Symbol();
  Boolean(sym) // true
  !sym  // false
  
  if (sym) {
    // ...
  }
  
  Number(sym) // TypeError
  sym + 2 // TypeError
  ```

- Symbol属性名遍历：Symbol 作为属性名，该属性不会出现在`for...in`、`for...of`循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames()`、`JSON.stringify()`返回。但是，它也不是私有属性，有一个`Object.getOwnPropertySymbols`方法，可以获取指定对象的所有 Symbol 属性名。

  `Object.getOwnPropertySymbols`方法返回一个数组，成员是当前对象的所有用作属性名的 Symbol 值。

  ```javascript
  const obj = {};
  
  let foo = Symbol("foo");
  
  Object.defineProperty(obj, foo, {
    value: "foobar",
  });
  
  for (let i in obj) {
    console.log(i); // 无输出
  }
  
  Object.getOwnPropertyNames(obj)
  // []
  
  Object.getOwnPropertySymbols(obj)
  // [Symbol(foo)]
  ```

  `Reflect.ownKeys`方法可以返回所有类型的键名，包括常规键名和 Symbol 键名。

  ```javascript
  let obj = {
    [Symbol('my_key')]: 1,
    enum: 2,
    nonEnum: 3
  };
  
  Reflect.ownKeys(obj)
  //  ["enum", "nonEnum", Symbol(my_key)]
  ```

  由于以 Symbol 值作为名称的属性，不会被常规方法遍历得到。我们可以利用这个特性，为对象定义一些非私有的、但又希望只用于内部的方法。

## 获取 Symbol 的三种方法

- **Symbol()** 每次调用时都返回一个唯一的 Symbol。
- **Symbol.for(string) 、Symbol.keyFor()**`Symbol.For`从 Symbol 注册表中返回相应的 Symbol，与上个方法不同的是，Symbol 注册表中的 Symbol 是共享的。也就是说，如果你调用 `Symbol.for("cat")` 三次，都将返回相同的 Symbol。当不同页面或同一页面不同模块需要共享 Symbol 时，注册表就非常有用。（`Symbol.for()`与`Symbol()`这两种写法，都会生成新的 Symbol。它们的区别是，前者会被登记在全局环境中供搜索，后者不会。比如，如果你调用`Symbol.for("cat")`30 次，每次都会返回同一个 Symbol 值，但是调用`Symbol("cat")`30 次，会返回 30 个不同的 Symbol 值。）`Symbol.keyFor`方法返回一个已登记的 Symbol 类型值的`key`。
- **Symbol.iterator** 返回语言预定义的一些 Symbol，每个都有其特殊的用途。

## Symbol 在 ES6 规范中的应用

我们已经知道可以使用 Symbol 来避免代码冲突。之前在[介绍 iterator](https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/) 时，我们还解析了 `for (var item of myArray)` 内部是以调用 `myArray[Symbol.iterator]()` 开始的，当时我提到这个方法可以使用 `myArray.iterator()` 来代替，但是使用 Symbol 的后向兼容性更好。

在 ES6 中还有一些地方使用到了 Symbol。（这些特性还没有在 FireFox 中实现。）

- **使 instanceof 可扩展**。在 ES6 中，`object instanceof constructor` 表达式被标准化为构造函数的一个方法：`constructor[Symbol.hasInstance](object)`，这意味着它是可扩展的。
- **消除新特性和旧代码之间的冲突**。
- **支持新类型的字符串匹配**。在 ES5 中，调用 `str.match(myObject)` 时，首先会尝试将 `myObject` 转换为 `RegExp` 对象。在 ES6 中，首先将检查 `myObject` 中是否有 `myObject[Symbol.match](str)` 方法，在所有正则表达式工作的地方都可以提供一个自定义的字符串解析方法。

##symbol的11个内置值

- `Symbol.hasInstance`方法，会被`instanceof`运算符调用。构造器对象用来识别一个对象是否是其实例。
- `Symbol.isConcatSpreadable`布尔值，表示当在一个对象上调用`Array.prototype.concat`时，这个对象的数组元素是否可展开。
- `Symbol.iterator`方法，被`for-of`语句调用。返回对象的默认迭代器。
- `Symbol.match`方法，被`String.prototype.match`调用。正则表达式用来匹配字符串。
- `Symbol.replace`方法，被`String.prototype.replace`调用。正则表达式用来替换字符串中匹配的子串。
- `Symbol.search`方法，被`String.prototype.search`调用。正则表达式返回被匹配部分在字符串中的索引。
- `Symbol.species`函数值，为一个构造函数。用来创建派生对象。
- `Symbol.split`方法，被`String.prototype.split`调用。正则表达式来用分割字符串。
- `Symbol.toPrimitive`方法，被`ToPrimitive`抽象操作调用。把对象转换为相应的原始值。
- `Symbol.toStringTag`方法，被内置方法`Object.prototype.toString`调用。返回创建对象时默认的字符串描述。
- `Symbol.unscopables`对象，它自己拥有的属性会被`with`作用域排除在外。

（详解可见阮一峰ES6入门http://es6.ruanyifeng.com/#docs/symbol）

## 模拟实现symbol

### 回顾特性

**1. Symbol 值通过 Symbol 函数生成，使用 typeof，结果为 "symbol"**

```javascript
var s = Symbol();
console.log(typeof s); // "symbol"
```

**2. Symbol 函数前不能使用 new 命令，否则会报错。这是因为生成的 Symbol 是一个原始类型的值，不是对象。**

**3. instanceof 的结果为 false**

```javascript
var s = Symbol('foo');
console.log(s instanceof Symbol); // false
```

**4. Symbol 函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。**

```javascript
var s1 = Symbol('foo');
console.log(s1); // Symbol(foo)
```

**5. 如果 Symbol 的参数是一个对象，就会调用该对象的 toString 方法，将其转为字符串，然后才生成一个 Symbol 值。**

```javascript
const obj = {
  toString() {
    return 'abc';
  }
};
const sym = Symbol(obj);
console.log(sym); // Symbol(abc)
```

**6. Symbol 函数的参数只是表示对当前 Symbol 值的描述，相同参数的 Symbol 函数的返回值是不相等的。**

```javascript
// 没有参数的情况
var s1 = Symbol();
var s2 = Symbol();

console.log(s1 === s2); // false

// 有参数的情况
var s1 = Symbol('foo');
var s2 = Symbol('foo');

console.log(s1 === s2); // false
```

**7. Symbol 值不能与其他类型的值进行运算，会报错。**

```javascript
var sym = Symbol('My symbol');

console.log("your symbol is " + sym); // TypeError: can't convert symbol to string
```

**8. Symbol 值可以显式转为字符串。**

```javascript
var sym = Symbol('My symbol');

console.log(String(sym)); // 'Symbol(My symbol)'
console.log(sym.toString()); // 'Symbol(My symbol)'
```

**9. Symbol 值可以作为标识符，用于对象的属性名，可以保证不会出现同名的属性。**

```javascript
var mySymbol = Symbol();

// 第一种写法
var a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
var a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
var a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上写法都得到同样结果
console.log(a[mySymbol]); // "Hello!"
```

**10. Symbol 作为属性名，该属性不会出现在 for...in、for...of 循环中，也不会被 Object.keys()、Object.getOwnPropertyNames()、JSON.stringify() 返回。但是，它也不是私有属性，有一个 Object.getOwnPropertySymbols 方法，可以获取指定对象的所有 Symbol 属性名。**

```javascript
var obj = {};
var a = Symbol('a');
var b = Symbol('b');

obj[a] = 'Hello';
obj[b] = 'World';

var objectSymbols = Object.getOwnPropertySymbols(obj);

console.log(objectSymbols);
// [Symbol(a), Symbol(b)]
```

**11. 如果我们希望使用同一个 Symbol 值，可以使用 Symbol.for。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建并返回一个以该字符串为名称的 Symbol 值。**

```javascript
var s1 = Symbol.for('foo');
var s2 = Symbol.for('foo');

console.log(s1 === s2); // true
```

**12. Symbol.keyFor 方法返回一个已登记的 Symbol 类型值的 key。**

```javascript
var s1 = Symbol.for("foo");
console.log(Symbol.keyFor(s1)); // "foo"

var s2 = Symbol("foo");
console.log(Symbol.keyFor(s2) ); // undefined
```

### 当调用 Symbol 的时候，发生什么？

当调用 Symbol 的时候，会采用以下步骤：

1. 如果使用 new ，就报错
2. 如果 description 是 undefined，让 descString 为 undefined
3. 否则 让 descString 为 ToString(description)
4. 如果报错，就返回
5. 返回一个新的唯一的 Symbol 值，它的内部属性 [[Description]] 值为 descString

### 第一版

```javascript
// 第一版
(function() {
    var root = this;

    var SymbolPolyfill = function Symbol(description) {

        // 实现第2点特性：Symbol 函数前不能使用 new 命令
        if (this instanceof SymbolPolyfill) throw new TypeError('Symbol is not a constructor');

        // 实现第 5 点特性：如果 Symbol 的参数是一个对象，就会调用该对象的 toString 方法，将其转为字符串，然后才生成一个 Symbol 值。
        var descString = description === undefined ? undefined : String(description)

        var symbol = Object.create(null)

        Object.defineProperties(symbol, {
            '__Description__': {
                value: descString,
                writable: false,
                enumerable: false,
                configurable: false
            }
        });

        // 实现第 6 点特性，因为调用该方法，返回的是一个新对象，两个对象之间，只要引用不同，就不会相同
        return symbol;
    }

    root.SymbolPolyfill = SymbolPolyfill;
})();
```

### 第二版

**1. 使用 typeof，结果为 "symbol"。**

利用 ES5，我们并不能修改 typeof 操作符的结果，所以这个无法实现。

**3. instanceof 的结果为 false**

因为不是通过 new 的方式实现的，所以 instanceof 的结果自然是 false。

**4.Symbol 函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述。主要是为了在控制台显示，或者转为字符串时，比较容易区分。**

当我们打印一个原生 Symbol 值的时候：

```
console.log(Symbol('1')); // Symbol(1)
```

可是我们模拟实现的时候返回的却是一个对象，所以这个也是无法实现的，当然你修改 console.log 这个方法是另讲。

**8.Symbol 值可以显式转为字符串。**

```javascript
var sym = Symbol('My symbol');

console.log(String(sym)); // 'Symbol(My symbol)'
console.log(sym.toString()); // 'Symbol(My symbol)'
```

当调用 String 方法的时候，如果该对象有 toString 方法，就会调用该 toString 方法，所以我们只要给返回的对象添加一个 toString 方法，即可实现这两个效果。

```javascript
// 第二版

// 前面面代码相同 ……

var symbol = Object.create({
    toString: function() {
        return 'Symbol(' + this.__Description__ + ')';
    },
});

// 后面代码相同 ……
```

### 第三版

**Symbol 值可以作为标识符，用于对象的属性名，可以保证不会出现同名的属性。**

看着好像没什么，这点其实和第 8 点是冲突的，这是因为当我们模拟的所谓 Symbol 值其实是一个有着 toString 方法的 对象，当对象作为对象的属性名的时候，就会进行隐式类型转换，还是会调用我们添加的 toString 方法，对于 Symbol('foo') 和 Symbol('foo')两个 Symbol 值，虽然描述一样，但是因为是两个对象，所以并不相等，但是当作为对象的属性名的时候，都会隐式转换为 `Symbol(foo)` 字符串，这个时候就会造成同名的属性。举个例子：

```javascript
var a = SymbolPolyfill('foo');
var b = SymbolPolyfill('foo');

console.log(a ===  b); // false

var o = {};
o[a] = 'hello';
o[b] = 'hi';

console.log(o); // {Symbol(foo): 'hi'}
```

为了防止不会出现同名的属性，毕竟这是一个非常重要的特性，迫不得已，我们需要修改 toString 方法，让它返回一个唯一值，所以第 8 点就无法实现了，而且我们还需要再写一个用来生成 唯一值的方法，就命名为 generateName，我们将该唯一值添加到返回对象的 __Name__ 属性中保存下来。

```javascript
// 第三版
(function() {
    var root = this;

    var generateName = (function(){
        var postfix = 0;
        return function(descString){
            postfix++;
            return '@@' + descString + '_' + postfix
        }
    })()

    var SymbolPolyfill = function Symbol(description) {

        if (this instanceof SymbolPolyfill) throw new TypeError('Symbol is not a constructor');

        var descString = description === undefined ? undefined : String(description)

        var symbol = Object.create({
            toString: function() {
                return this.__Name__;
            }
        })

        Object.defineProperties(symbol, {
            '__Description__': {
                value: descString,
                writable: false,
                enumerable: false,
                configurable: false
            },
            '__Name__': {
                value: generateName(descString),
                writable: false,
                enumerable: false,
                configurable: false
            }
        });

        return symbol;
    }


    root.SymbolPolyfill = SymbolPolyfill;

})()
```

此时再看下这个例子：

```javascript
var a = SymbolPolyfill('foo');
var b = SymbolPolyfill('foo');

console.log(a ===  b); // false

var o = {};
o[a] = 'hello';
o[b] = 'hi';

console.log(o); // Object { "@@foo_1": "hello", "@@foo_2": "hi" }
```

### 第四版

**7.Symbol 值不能与其他类型的值进行运算，会报错。**

以 `+` 操作符为例，当进行隐式类型转换的时候，会先调用对象的 valueOf 方法，如果没有返回基本值，就会再调用 toString 方法，所以我们考虑在 valueOf 方法中进行报错，比如：

```javascript
var symbol = Object.create({
    valueOf: function() {
        throw new Error('Cannot convert a Symbol value')
    }
})

console.log('1' + symbol); // 报错
```

看着很简单的解决了这个问题，可是如果我们是显式调用 valueOf 方法呢？对于一个原生的 Symbol 值：

```javascript
var s1 = Symbol('foo')
console.log(s1.valueOf()); // Symbol(foo)
```

是的，对于原生 Symbol，显式调用 valueOf 方法，会直接返回该 Symbol 值，而我们又无法判断是显式还是隐式的调用，所以这个我们就只能实现一半，要不然实现隐式调用报错，要不然实现显式调用返回该值，那……我们选择不报错的那个吧，即后者。

我们迫不得已的修改 valueOf 函数：

```javascript
// 第四版
// 前面面代码相同 ……

var symbol = Object.create({
    toString: function() {
        return this.__Name__;
    },
    valueOf: function() {
        return this;
    }
});
// 后面代码相同 ……
```

### 第五版

**10. Symbol 作为属性名，该属性不会出现在 for...in、for...of 循环中，也不会被 Object.keys()、Object.getOwnPropertyNames()、JSON.stringify() 返回。但是，它也不是私有属性，有一个 Object.getOwnPropertySymbols 方法，可以获取指定对象的所有 Symbol 属性名。**

嗯，无法实现。

**11. 有时，我们希望重新使用同一个Symbol值，Symbol.for方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的Symbol值。如果有，就返回这个Symbol值，否则就新建并返回一个以该字符串为名称的Symbol值。**

这个实现类似于函数记忆，我们建立一个对象，用来储存已经创建的 Symbol 值即可。

**12. Symbol.keyFor 方法返回一个已登记的 Symbol 类型值的 key。**

遍历 forMap,查找该值对应的键值即可。

```javascript
// 第五版
// 前面代码相同 ……
var SymbolPolyfill = function() { ... }

var forMap = {};

Object.defineProperties(SymbolPolyfill, {
    'for': {
        value: function(description) {
            var descString = description === undefined ? undefined : String(description)
            return forMap[descString] ? forMap[descString] : forMap[descString] = SymbolPolyfill(descString);
        },
        writable: true,
        enumerable: false,
        configurable: true
    },
    'keyFor': {
        value: function(symbol) {
            for (var key in forMap) {
                if (forMap[key] === symbol) return key;
            }
        },
        writable: true,
        enumerable: false,
        configurable: true
    }
});
// 后面代码相同 ……
```

### 完整实现

综上所述：

无法实现的特性有：1、4、7、8、10

可以实现的特性有：2、3、5、6、9、11、12

最后的实现如下:

```javascript
(function() {
    var root = this;

    var generateName = (function(){
        var postfix = 0;
        return function(descString){
            postfix++;
            return '@@' + descString + '_' + postfix
        }
    })()

    var SymbolPolyfill = function Symbol(description) {

        if (this instanceof SymbolPolyfill) throw new TypeError('Symbol is not a constructor');

        var descString = description === undefined ? undefined : String(description)

        var symbol = Object.create({
            toString: function() {
                return this.__Name__;
            },
            valueOf: function() {
                return this;
            }
        })

        Object.defineProperties(symbol, {
            '__Description__': {
                value: descString,
                writable: false,
                enumerable: false,
                configurable: false
            },
            '__Name__': {
                value: generateName(descString),
                writable: false,
                enumerable: false,
                configurable: false
            }
        });

        return symbol;
    }

    var forMap = {};

    Object.defineProperties(SymbolPolyfill, {
        'for': {
            value: function(description) {
                var descString = description === undefined ? undefined : String(description)
                return forMap[descString] ? forMap[descString] : forMap[descString] = SymbolPolyfill(descString);
            },
            writable: true,
            enumerable: false,
            configurable: true
        },
        'keyFor': {
            value: function(symbol) {
                for (var key in forMap) {
                    if (forMap[key] === symbol) return key;
                }
            },
            writable: true,
            enumerable: false,
            configurable: true
        }
    });

    root.SymbolPolyfill = SymbolPolyfill;

})()
```