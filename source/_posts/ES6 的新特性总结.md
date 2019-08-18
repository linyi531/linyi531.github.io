---
title: ES6 的新特性总结
date: 2019-06-01 22:23:01
tags:
  - ES6
  - javascript
categories: ES6
---

# ES6 的新特性

## 1. 数组的拓展

### 数组的方法

- `array.concat(array1, array2,…arrayN)`合并多个数组，返回合并后的新数组，原数组没有变化。
- `array.every(callback[, thisArg])`检测数组中的每一个元素是否都通过了 callback 测试，全部通过返回 true，否则返回 false。
- `array.filter(callback[, thisArg])`返回一个新数组，包含通过 callback 函数测试的所有元素。(callback 定义，三个参数： element:当前元素值；index：当前元素下标； array:当前数组)
- `array.find(callback[, thisArg])`返回通过 callback 函数测试的第一个元素，否则返回 undefined，callback 函数定义同上。
  <!-- more -->
- `array.findIndex(callback[, thisArg])`返回通过 callback 函数测试的第一个元素的索引，否则返回-1，callback 函数定义同上。
- `array.includes(searchElement, fromIndex)`includes() 方法用来判断一个数组是否包含一个指定的值，返回 true 或 false。searchElement：要查找的元素；fromIndex：开始查找的索引位置。
- `array.indexOf(searchElement[, fromIndex = 0])`返回在数组中可以找到一个给定元素的第一个索引，如果不存在，则返回-1。searchElement：要查找的元素；fromIndex：开始查找的索引位置。
- `array.join(separator=',')`将数组中的元素通过 separator 连接成字符串，并返回该字符串，separator 默认为","。
- `array.map(callback[, thisArg])`返回一个新数组，新数组中的每个元素都是调用 callback 函数后返回的结果。注意：如果没有 return 值，则新数组会插入一个 undefined 值。array.map 由于不具有过滤的功能，因此 array 调用 map 函数时，如果 array 中的数据并不是每一个都会 return，则必须先 filter，然后再 map，即 map 调用时必须是对数组中的每一个元素都有效。
- `array.pop() 与 array.shift()`pop 为从数组中删除最后一个元素，并返回最后一个元素的值，原数组的最后一个元素被删除。数组为空时返回 undefined。shift 删除数组的第一个元素，并返回第一个元素，原数组的第一个元素被删除。数组为空返回 undefined。
- `array.push(element1, element2, ....elementN) 与 array.unshift(element1, element2, ...elementN)`push 是将一个或多个元素添加到数组的末尾，并返回新数组的长度; unshift 将一个或多个元素添加到数组的开头，并返回新数组的长度。唯一的区别就是插入的位置不同。
- `array.reduce(callback[, initialValue])`对数组中的每个元素（从左到右）执行 callback 函数累加，将其减少为单个值。
- `array.reverse()`将数组中元素的位置颠倒。
- `array.slice(begin, end)`返回一个新数组，包含原数组从 begin 到 end(不包含 end)索引位置的所有元素。
- `array.some(callback[, thisArg])`判断数组中是否包含可以通过 callback 测试的元素，与 every 不同的是，这里只要某一个元素通过测试，即返回 true。callback 定义同上。
- `array.sort([compareFunction])`对数组中的元素进行排序，compareFunction 不存在时，元素按照转换为的字符串的诸个字符的 Unicode 位点进行排序，慎用！请使用时一定要加 compareFunction 函数，而且该排序是不稳定的。
- `array.splice(start[, deleteCount, item1, item2, ...])`通过删除现有元素和/或添加新元素来更改一个数组的内容。start:指定修改的开始位置；deleteCount：从 start 位置开始要删除的元素个数；item...：要添加进数组的元素,从 start 位置开始。返回值是由被删除的元素组成的一个数组。如果只删除了一个元素，则返回只包含一个元素的数组。如果没有删除元素，则返回空数组。如果 deleteCount 大于 start 之后的元素的总数，则从 start 后面的元素都将被删除（含第 start 位）。

注：push、 shift、 pop、 unshift、 reverse、 sort、 splice 方法会对原来的数组进行修改，其他的数组操作方法只有返回值不同，对原数组都没有影响，即原数组不变。

### ES6 中对数组的扩展

- Array.from() : 将伪数组对象或可遍历对象转换为真数组

- Array.of(v1, v2, v3) : 将一系列值转换成数组。

  当使用单个数值参数来调用 Array 构造器时，数组的长度属性会被设置为该参数。 如果使用多个参数(无论是否为数值类型)来调用，这些参数也会成为目标数组的项。数组的这种行为既混乱又有风险，因为有时可能不会留意所传参数的类型。

  Array.of( )方法总会创建一个包含所有传入参数的数组，而不管参数的数量与类型

  ```javascript
  let items = Array.of(1, 2);
  console.log(items.length); // 2
  console.log(items[0]); // 1
  console.log(items[1]); // 2
  items = Array.of(2);
  console.log(items.length); // 1
  console.log(items[0]); // 2
  ```

  Array.of 基本上可以用来替代 Array()或 newArray()，并且不存在由于参数不同而导致的重载，而且他们的行为非常统一。

- 数组实例的 find() 和 findIndex()

  - find 方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为 true 的成员，然后返回该成员。如果没有符合条件的成员，则返回 undefined。

    ```javascript
    [1, 4, -5, 10].find(n => n < 0); // -5
    ```

* findIndex 方法的用法与 find 方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1。

  ```javascript
  [1, 5, 10, 15].findIndex(function(value, index, arr) {
    return value > 9;
  }); // 2
  ```

* 数组实例的 includes()

  Array.prototype.includes 方法返回一个布尔值，表示某个数组是否包含给定的值。该方法的第二个参数表示搜索的起始位置，默认为 0。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为-4，但数组长度为 3），则会重置为从 0 开始。

  ```javascript
  [1, 2, 3]
    .includes(2) // true
    [(1, 2, 3)].includes(3, -1); // true
  [1, 2, 3, 5, 1].includes(1, 2); // true
  ```

  没有该方法之前，我们通常使用数组的 indexOf 方法，检查是否包含某个值。它内部使用严格相等运算符（===）进行判断，这会导致对 NaN 的误判。

  ```javascript
  [NaN]
    .indexOf(NaN) // -1
    [NaN].includes(NaN); // true
  ```

* 数组实例的 entries()，keys() 和 values()

  它们都返回一个遍历器对象，可以用 for...of 循环进行遍历，唯一的区别是 keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。

  ```javascript
  for (let index of ["a", "b"].keys()) {
    console.log(index);
  }
  // 0
  // 1

  for (let elem of ["a", "b"].values()) {
    console.log(elem);
  }
  // 'a'
  // 'b'

  for (let [index, elem] of ["a", "b"].entries()) {
    console.log(index, elem);
  }
  // 0 "a"
  // 1 "b"
  ```

## 2. 箭头函数

- 缩减代码和改变 this 指向
- 使用注意点：
  - 函数体内的 this 对象，就是定义时所在的对象，而不是使用时所在的对象。
  - 不可以当作构造函数，也就是说，不可以使用 new 命令，否则会抛出一个错误。
  - 不可以使用 arguments 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。
  - 不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。

## 3. rest 参数

- 用于获取函数的多余参数，这样就不需要使用 arguments 对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。
- 注意点：
  - 每个函数最多只能声明一个 rest 参数，而且 rest 参数必须是最后一个参数，否则报错。
  - rest 参数不能用于对象字面量 setter 之中

## 4. 展开运算符

剩余参数允许你把多个独立的参数合并到一个数组中；而扩展运算符则允许将一个数组分割，并将各个项作为分离的参数传给函数。

## 5. 解构赋值----更方便的数据访问

## 6. 模板字符串

用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。模板字符串中嵌入变量和函数，需要将变量名写在\${}之中。

## 7. class 类

## 8. promise

## 9. Iterator 和 for...of 循环

JavaScript 原有的表示“集合”的数据结构，主要是数组（Array）和对象（Object），ES6 添加了 Map 和 Set。

任何数据结构只要部署 Iterator 接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。

- Iterator 的作用：
  - 为各种数据结构，提供一个统一的、简便的访问接口；
  - 使得数据结构的成员能够按某种次序排列
  - ES6 创造了一种新的遍历命令 for...of 循环，Iterator 接口主要供 for...of 消费。
- 原生具备 iterator 接口的数据(可用 for of 遍历)
  - Array
  - set 容器
  - map 容器
  - String
  - 函数的 arguments 对象
  - NodeList 对象
- 几种遍历方式比较
  - for of 循环不仅支持数组、大多数伪数组对象，也支持字符串遍历，此外还支持 Map 和 Set 对象遍历
  - for in 循环可以遍历字符串、对象、数组，不能遍历 Set/Map
  - forEach 循环不能遍历字符串、对象,可以遍历 Set/Map

## 10. ES6 模块化

其模块功能主要由两个命令构成：export 和 import。export 命令用于规定模块的对外接口，import 命令用于输入其他模块提供的功能。
