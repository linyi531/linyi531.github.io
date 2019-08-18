---
title: var，let和const声明及区别
date: 2018-07-04 11:29:59
tags: 
- ES6
- 前端
categories: ES6
---
# var声明
在函数作用域或全局作用域中通过var声明的变量，都会被当成在当前作用域顶部声明的变量。这就是提升（Hoisting）机制。
<!-- more -->
例如：
```javascript
fuction getValue(condition){
    if(condition){
        var value="blue";
        //其他代码
        return value;
    }
    else{
        //此处可以访问变量value，其值为undefined
        return null;
    }
    //此处可以访问变量value，其值为undefined
}
```
事实上，在预编译阶段，JavaScript引擎会将上面的函数修改为下面这样：
```javascript
fuction getValue(condition){
    var value;
    if(condition){
        value="blue";
        //其他代码
        return value;
    }
    else{
        //此处可以访问变量value，其值为undefined
        return null;
    }
    //此处可以访问变量value，其值为undefined
}
```
变量value的声明会被提升至函数顶部，而初始化操作依然留在原处执行。这样，就意味着，在函数的其他部分，else子句中或者if-else外，也能访问到value变量，而由于此时value变量并未被初始化赋值，所以访问到值为undefined。

# 块级声明
ES6中引入块级作用域来强化对变量生命周期的控制。
块级声明用于声明在指定块的作用域之外无法访问的变量。块级作用域（词法作用域）存在于：
1. 函数内部
2. 块中（字符{和}之间的区域）

## let声明
let声明的用法与var相同。用let代替var来声明变量，就可以把变量的作用域限制在当前代码块中。
let声明<font color="#DC143C">不会被提升</font>，因此通常将let声明语句放在封闭代码块的顶部，以便整个代码块都可以访问。
```javascript
fuction getValue(condition){
    if(condition){
        var value="blue";
        //其他代码
        return value;
    }
    else{
        //变量value在此处不存在
        return null;
    }
    //变量value在此处不存在
}
```
  let声明后，不会被提升至函数顶部。因此执行流离开if块之后，value立刻被销毁，如果condition的值为false，就永远不会声明并初始化value。并且，假设作用域中已经存在了某个标识符，此时再用let关键字声明它，就会抛出错误：
  ```javascript
  var count=30;
  //抛出语法错误
  let count=40;
  ```
  在同一作用域中不能用let重复定义已经存在的标识符，所以此处使用let声明会抛出错误。但如果当前作用域内内嵌另一个作用域，就可在内嵌的作用域中使用let声明同名变量。
  例如：
  ```javascript
  var count=30;
  if(condition){
      //不会抛出错误
      let count=40;
  }
  ```
  此时，if内部块中的count会遮蔽全局作用域中的count，而var声明的count只能在if块外访问到。

## const声明
使用const声明的是常量，其值一旦被设定后不可更改。因此每个通过const声明的常量必须在声明的同时被初始化。
```javascript
//有效的常量
const max=30;

//语法错误，常量未初始化
const name;
```
const和let声明都是块级标识符，所以常量也只在当前的代码块内有效，一旦执行到代码块外会被立即销毁。并且，常量也不会被提升至作用域顶部。同样，与let相似，在同一作用域用const声明已经存在的标识符，也会导致语法错误。

如上所述，const定义的值一旦被设定后不可更改，无论在严格模式还是非严格模式下，都不可以为const定义的常量再赋值，否则会抛出语法错误：
```javascript
const max=50;
//抛出语法错误
max=30;
```
然而，与其他语言中的常量不同的是，JavaScript中的常量，如果是对象，则对象的值可以修改，也就是const声明<font color="#DC143C">不允许修改绑定，但允许修改值</font>这也意味着，const声明对象后，可以修改对象的属性。const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指针，const只能保证这个指针是固定的，至于它指向的数据结构是不是可变的，就完全不能控制了。
```javascript
const person={
    name:"Nicholas";
};

//可以修改对象属性的值
person.name="Greg";

//抛出语法错误
person={
    name:"Greg";
};
```
如果真的想将对象冻结，应该使用Object.freeze方法。
```javascript
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```
上面代码中，常量foo指向一个冻结的对象，所以添加新属性不起作用，严格模式时还会报错。
除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。
```javascript
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```

## 临时死区（Temporal Dead Zone）
临时死区常被描述let和const的不提升的效果。JavaScript引擎在扫描代码发现变量声明时，要么将它们提升至作用域顶部（遇到var声明时），要么将声明放到TDZ中（遇到let和const声明时）。访问TDZ中的变量会触发运行错误。只有在执行过变量声明语句后，变量才会从TDZ中移出，然后方可正常访问。ES6 明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。
```javascript
if(condition){
    console.log(typeof value);//引用错误！
    let value="blue";
}
```
但在let声明的作用域外对该变量使用typeof则不会报错：
```javascript
console.log(typeof value);   //"undefined"

if(condition){
    let value="blue";
}
```
typeof是在声明变量value的代码块外执行的，此时value并不在TDZ中，也就意味着不存在value这个绑定，typeof操作最终返回"undefined"。

# 循环中的块级作用域绑定
先看这段代码：
```javascript
var funcs=[];

for(var i=0;i<10;i++){
    funcs.push(function(){
        console.log(i);
    });
}

funcs.forEach(function(func){
    func();   //输出10次数字10
});
```
预想结果是输出数字0～9，但是因为循环里的每次迭代同时共享着i，循环内部创建的函数全部都保存了对相同变量的引用。循环结束时变量i的值为10，所以每次调用console.log（i）时都会输出数字10。
而使用let声明，每次迭代循环都会创建一个新变量，并以之前的迭代中同名变量的值将其初始化，得到预期的效果。
```javascript
var funcs=[];

for(let i=0;i<10;i++){
    funcs.push(function(){
        console.log(i);
    });
}

funcs.forEach(function(func){
    func();   //输出0，1，2，……，9
});
```
当前的i只在本轮循环有效，所以每一次循环的i其实都是一个新的变量，JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量i时，就在上一轮循环的基础上进行计算。另外，for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。
const也是同样，但在循环中不能修改const声明的变量，否则会抛出错误。

# 全局作用域绑定
当var被用在全局作用域时，它会创建一个新的全局变量作为全局对象（浏览器环境中的window对象）的属性。这意味着，用var很可能会无意中覆盖一个已经存在的全局属性。例如：
```javascript
//在浏览器中
var RegExp="Hello!";
console.log(window.RegExp);  //"Hello!"
```
全局对象定义在RegExp定义在window上，但不能幸免<font color="#DC143C">被var覆盖，成为window的属性。</font>
但如果在全局作用域中使用let或者const，会在全局作用域下创建一个新的绑定，但该绑定不会添加全局对象的属性。<font color="#DC143C">用let或const不能覆盖全局变量，只能遮蔽它。</font>
```javascript
//在浏览器中
let RegExp="Hello!";
console.log(RegExp);  //"Hello!"
console.log(window.RegExp===RegExp);  //false
```
这里let声明的RegExp创建了一个绑定并遮蔽了全局的RegExp变量，但window.RegExp和RegExp并不相同，说明它不会破坏全局作用域，不会为全局对象创建属性。

# ES6声明变量的六种方法
ES5只有两种声明变量的方法：var命令和function命令。ES6除了添加let和const命令，还有另外两种声明变量的方法：import命令和class命令。所以，ES6 一共有 6 种声明变量的方法。


