---
title: Vue计算属性(computed)和侦听属性(watch)
date: 2019-12-29 09:52:21
tags:
  - VUE
categories: VUE
cover_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gadm9k5sqbj31900u0x6q.jpg
feature_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gadm9k5sqbj31900u0x6q.jpg
---

# Vue计算属性(computed)和侦听属性(watch)

## 计算属性

### 介绍

计算属性是自动监听依赖值的变化，从而动态返回内容，监听是一个过程，在监听的值变化时，可以触发一个回调，并做一些事情。它有以下几个特点：

- 数据可以进行逻辑处理，减少模板中计算逻辑。
- 对计算属性中的数据进行监视
- 依赖固定的数据类型（响应式数据）

计算属性由两部分组成：get和set，分别用来获取计算属性和设置计算属性。默认只有get，如果需要set，要自己添加。另外set设置属性，并不是直接修改计算属性，而是修改它的依赖。

```javascript
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      //this.fullName = newValue 这种写法会报错
      var names = newValue.split(' ')
      this.firstName = names[0]//对它的依赖进行赋值
      this.lastName = names[names.length - 1]
    }
  }
}
```

### 计算属性 vs 普通属性

可以像绑定普通属性一样在模板中绑定计算属性，在定义上有区别：计算属性的属性值必须是一个函数。

```javascript
data:{ //普通属性
  msg:'浪里行舟',
},
computed:{ //计算属性
  msg2:function(){ //该函数必须有返回值，用来获取属性，称为get函数
    return '浪里行舟';
  },
  reverseMsg:function(){
  //可以包含逻辑处理操作，同时reverseMsg依赖于msg,一旦msg发生变化，reverseMsg也会跟着变化
    return this.msg.split(' ').reverse().join(' ');
 }
```

### 计算属性 vs 方法

**两者最主要的区别：computed 是可以缓存的，methods 不能缓存；**

**只要相关依赖没有改变，多次访问计算属性得到的值是之前缓存的计算结果，不会多次执行。**网上有种说法就是方法可以传参，而计算属性不能，其实并不准确，计算属性可以通过闭包来实现传参：

```javascript
:data="closure(item, itemName, blablaParams)"
computed: {
 closure () {
   return function (a, b, c) {
        /** do something */
        return data
    }
 }
}
```

## 侦听属性

Vue 提供了一种更通用的方式来观察和响应 Vue 实例上的数据变动：侦听属性watch。**watch中可以执行任何逻辑，如函数节流，Ajax异步获取数据，甚至操作 DOM（不建议）。**

### 常规用法

```vue
<template>
  <div class="attr">
    <h1>watch属性</h1>
    <h2>{{ $data }}</h2>
    <button @click="() => (a += 1)">修改a的值</button>
  </div>
</template>
<script>
export default {
  data() {
    return {
      a: 1,
      b: { c: 2, d: 3 },
      e: {
        f: {
          g: 4
        }
      },
      h: []
    };
  },
  watch: {
    a: function(val, oldVal) {
      this.b.c += 1;
    },
    "b.c": function(val, oldVal) {
      this.b.d += 1;
    },
    "b.d": function(val, oldVal) {
      this.e.f.g += 1;
    },
    e: {
      handler: function(val, oldVal) {
        this.h.push("浪里行舟");
      },
      deep: true //用于监听e对象内部值的变化
    }
  }
};
</script>
```

### 使用 watch 的深度遍历和立即调用功能

使用 watch 来监听数据变化的时候除了常用到 handler 回调，其实其还有两个参数，便是：

- deep 设置为 true 用于监听对象内部值的变化
- immediate 设置为 true 将立即以表达式的当前值触发回调

```vue
<template>
    <button @click="obj.a = 2">修改</button>
</template>
<script>
export default {
    data() {
        return {
            obj: {
                a: 1,
            }
        }
    },
    watch: {
        obj: {
            handler: function(newVal, oldVal) {
                console.log(newVal); 
            },
            deep: true,
            immediate: true 
        }
    }
}
</script>
```

以上代码我们修改了 obj 对象中 a 属性的值，我们可以触发其 watch 中的 handler 回调输出新的对象，而如果不加 deep: true，我们只能监听 obj 的改变，并不会触发回调。同时我们也添加了 immediate: true 配置，其会立即以 obj 的当前值触发回调。

## computed和watch两者之间对比

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j5tpjgacj30ip078q2z.jpg)

从上面流程图中，我们可以看出它们之间的区别：

- watch：监测的是属性值， 只要属性值发生变化，其都会触发执行回调函数来执行一系列操作。
- computed：监测的是依赖值，依赖值不变的情况下其会直接读取缓存进行复用，变化的情况下才会重新计算。

除此之外，有点很重要的区别是：**计算属性不能执行异步任务，计算属性必须同步执行**。也就是说计算属性不能向服务器请求或者执行异步任务。如果遇到异步任务，就交给侦听属性。watch也可以检测computed属性。

计算属性适合用在模板渲染中，某个值是依赖了其它的响应式对象甚至是计算属性计算而来；而侦听属性适用于观测某个值的变化去完成一段复杂的业务逻辑。

- computed能做的，watch都能做，反之则不行
- 能用computed的尽量用computed