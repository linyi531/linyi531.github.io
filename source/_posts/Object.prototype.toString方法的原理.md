---
title: Object.prototype.toString方法的原理
date: 2019-10-15 12:33:37
tags:
  - JavaScript
categories: JavaScript
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g85nx9sg0wj30u0190x6p.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g85nx9sg0wj30u0190x6p.jpg
---

# Object.prototype.toString 方法的原理

## ECMAScript 3

### Object.prototype.toString 方法的规范

在**toString**方法被调用时,会执行下面的操作步骤:

1. 获取 this 对象的[[Class]]属性的值.
2. 计算出三个字符串**"[object ",**第一步的操作结果 Result(1), 以及 **"]"**连接后的新字符串.
3. 返回第二步的操作结果 Result(2).

### [[Class]]

[[Class]]是一个内部属性,所有的对象(原生对象和宿主对象)都拥有该属性.在规范中,[[Class]]是这么定义的

| 内部属性  | 描述                             |
| --------- | -------------------------------- |
| [[Class]] | 一个字符串值,表明了该对象的类型. |

然后给了一段解释:

> 所有内置对象的[[Class]]属性的值是由本规范定义的.所有宿主对象的[[Class]]属性的值可以是任意值,甚至可以是内置对象使用过的[[Class]]属性的值.[[Class]]属性的值可以用来判断一个原生对象属于哪种内置类型.需要注意的是,除了通过**Object.prototype.toString**方法之外,本规范没有提供任何其他方式来让程序访问该属性的值

也就是说,把 Object.prototype.toString 方法返回的字符串,去掉前面固定的**"[object "**和后面固定的**"]",**就是内部属性[[class]]的值,也就达到了判断对象类型的目的。

### [[Class]]的值

在 ES3 中,规范文档并没有总结出[[class]]内部属性一共有几种,不过我们可以自己统计一下,原生对象的[[class]]内部属性的值一共有 10 种.分别是:`"Array"`, `"Boolean"`, `"Date"`, `"Error"`, `"Function"`, `"Math"`, `"Number"`, `"Object"`, `"RegExp"`, `"String".`

## ECMAScript 5

### Object.prototype.toString 方法的规范

在**toString**方法被调用时,会执行下面的操作步骤:

1. 如果**this**的值为**undefined**,则返回`"[object Undefined]"`.
2. 如果**this**的值为**null**,则返回`"[object Null]"`.
3. 让*O*成为调用 ToObject(**this)**的结果.
4. 让*class*成为*O*的内部属性[[Class]]的值.
5. 返回三个字符串**"[object ",** _class_, 以及 **"]"**连接后的新字符串.

可以看出,ES5 比 ES3 多了 1,2,3 步.第 1,2 步属于新规则,比较特殊,因为"`Undefined"`和"`Null"`并不属于[[class]]属性的值,需要注意的是,这里和严格模式无关(大部分函数在严格模式下,this 的值才会保持 undefined 或 null,非严格模式下会自动成为全局对象).第 3 步并不算是新规则,因为在 ES3 的引擎中,也都会在这一步将三种原始值类型转换成对应的包装对象,只是规范中没写出来.

### [[Class]]

ES5 中,[[Class]]属性的解释更加详细:

> 所有内置对象的[[Class]]属性的值是由本规范定义的.所有宿主对象的[[Class]]属性的值可以是除了"Arguments", "Array", "Boolean", "Date", "Error", "Function", "JSON", "Math", "Number", "Object", "RegExp", "String"之外的的任何字符串.[[Class]]内部属性是引擎内部用来判断一个对象属于哪种类型的值的.需要注意的是,除了通过**Object.prototype.toString**方法之外,本规范没有提供任何其他方式来让程序访问该属性的值

### 对比 ES3

- 第一个差别就是[[class]]内部属性的值多了两种,成了 12 种
  - 一种是 arguments 对象的[[class]]成了"Arguments",而不是以前的"Object"
  - 多个了全局对象 JSON,它的[[class]]值为"JSON".
- 第二个差别就是,宿主对象的[[class]]内部属性的值,不能和这 12 种值冲突（不过在支持 ES3 的浏览器中,貌似也没有发现哪些宿主对象故意使用那 10 个值）

## ECMAScript 6

**[[class]]内部属性没有了**,取而代之的是另外一个内部属性[[NativeBrand]].[[NativeBrand]]属性是这么定义的:

| 内部属性        | 属性值                       | 描述                                                            |
| --------------- | ---------------------------- | --------------------------------------------------------------- |
| [[NativeBrand]] | 枚举 NativeBrand 的一个成员. | 该属性的值对应一个标志值(tag value),可以用来区分原生对象的类型. |

### [[NativeBrand]]属性（`internal slot`）

[[NativeBrand]]内部属性用来识别某个原生对象是否为符合本规范的某一种特定类型的对象.[[NativeBrand]]内部属性的值为下面这些枚举类型的值中的一个:NativeFunction, NativeArray, StringWrapper, BooleanWrapper, NumberWrapper, NativeMath, NativeDate, NativeRegExp, NativeError, NativeJSON, NativeArguments, NativePrivateName.[[NativeBrand]]内部属性仅用来区分区分特定类型的 ECMAScript 原生对象.只有在表 10 中明确指出的对象类型才有[[NativeBrand]]内部属性.

| 属性值            | 对应类型             |
| ----------------- | -------------------- |
| NativeFunction    | Function objects     |
| NativeArray       | Array objects        |
| StringWrapper     | String objects       |
| BooleanWrapper    | Boolean objects      |
| NumberWrapper     | Number objects       |
| NativeMath        | The Math object      |
| NativeDate        | Date objects         |
| NativeRegExp      | RegExp objects       |
| NativeError       | Error objects        |
| NativeJSON        | The JSON object      |
| NativeArguments   | Arguments objects    |
| NativePrivateName | Private Name objects |

可见,和[[class]]不同的是,并不是每个对象都拥有[[NativeBrand]].

### Object.prototype.toString 方法的规范:

在**toString**方法被调用时,会执行下面的操作步骤:

1. 如果**this**的值为**undefined**,则返回`"[object Undefined]"`.
2. 如果**this**的值为**null**,则返回`"[object Null]"`.
3. 让*O*成为调用 ToObject(**this)**的结果.
4. 如果*O*有[[NativeBrand]]内部属性,让*tag*成为表 29 中对应的值.
5. 否则
   1. 让*hasTag*成为调用*O*的[[HasProperty]]内部方法后的结果,参数为@@toStringTag.
   2. 如果*hasTag*为**false**,则让*tag*为`"Object"`.
   3. 否则,
      1. 让*tag*成为调用*O*的[[Get]]内部方法后的结果,参数为@@toStringTag.
      2. 如果*tag*是一个 abrupt completion,则让*tag*成为 NormalCompletion(`"???"`).
      3. 让*tag*成为*tag*.[[value]].
      4. 如果 Type(_tag_)不是字符串,则让*tag 成为*`"???"`.
      5. 如果*tag*的值为`"Arguments"`, `"Array"`, `"Boolean"`, `"Date"`, `"Error"`, `"Function"`, `"JSON"`, `"Math"`, `"Number"`, `"Object"`, `"RegExp"`,`或者"String"中的任一个,则让`*tag*成为字符串`"~"和`*tag*当前的值连接后的结果.
6. 返回三个字符串"[object ", tag, and "]"连接后的新字符串.

### ES6 里的新类型 Map,Set

ES6 里的新类型 Map,Set 等,都没有在表 29 中.它们在执行 toString 方法的时候返回的是什么?

```javascript
console.log(Object.prototype.toString.call(Map())); //"[object Map]"
console.log(Object.prototype.toString.call(Set())); //"[object Set]"
```

Map.prototype.@@toStringTag

@@toStringTag 属性的初始值为字符串**"Map"**.
