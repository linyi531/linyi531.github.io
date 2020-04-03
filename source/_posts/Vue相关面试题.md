---
title: Vue相关面试题
date: 2020-02-17 10:45:21
tags:
  - VUE
categories: VUE
cover_img: https://tva1.sinaimg.cn/large/00831rSTgy1gdgdxdta1vj30u0151kjl.jpg
feature_img: https://tva1.sinaimg.cn/large/00831rSTgy1gdgdxdta1vj30u0151kjl.jpg
---

# vue

Vue.js（读音 /vjuː/, 类似于 view） 是一套构建用户界面的 渐进式框架。与其他重量级框架不同的是，Vue 采用自底向上增量开发的设计。Vue 的核心库只关注视图层，并且非常容易学习，非常容易与其它库或已有项目整合。另一方面，Vue 完全有能力驱动采用[单文件组件](http://link.zhihu.com/?target=http%3A//cn.vuejs.org/v2/guide/single-file-components.html)和 [Vue 生态系统支持的库](http://link.zhihu.com/?target=http%3A//github.com/vuejs/awesome-vue%23libraries--plugins)开发的复杂单页应用。

Vue.js 的目标是通过尽可能简单的 API 实现响应的数据绑定和组合的视图组件。

* 响应式编程（双向数据绑定 MVVM）
* 组件化
* 模块化（配合 [Webpack](https://link.zhihu.com/?target=http%3A//webpack.github.io/) 或者 [Browserify](https://link.zhihu.com/?target=http%3A//browserify.org/)等打包工具，然后再加上 ES2015。每一个 Vue 组件都可以看做一个独立的模块）
* 动画（Vue 自带简洁易用的[过渡动画系统](https://link.zhihu.com/?target=http%3A//vuejs.org/guide/transitions.html)。Vue 的反应式系统也使得它可以用来开发高效的数据驱动的逐帧动画。这一类逐帧动画在基于脏检查或是 Virtual DOM 的框架中，往往会导致性能问题，因为即使只是改了一个值，整个所处的子树（scope 或是 component）都需要重新计算。而 Vue 则是改了多少，计算多少，不会有无谓的浪费。在小 demo 中，脏检查或是 Virtual DOM 往往也足够快，但是在大型应用中可就不一定了。）
* Virtual DOM
* Vuex
* Vue-router

## 1. 对于MVVM的理解？

MVVM 是 Model-View-ViewModel 的缩写。 **Model**代表数据模型，也可以在Model中定义数据修改和操作的业务逻辑。 **View** 代表UI 组件，它负责将数据模型转化成UI 展现出来。 **ViewModel** 监听模型数据的改变和控制视图行为、处理用户交互，简单理解就是一个同步View 和 Model的对象，连接Model和View。 在MVVM架构下，View 和 Model 之间并没有直接的联系，而是通过ViewModel进行交互，Model 和 ViewModel 之间的交互是双向的， 因此View 数据的变化会同步到Model中，而Model 数据的变化也会立即反应到View 上。 **ViewModel** 通过双向数据绑定把 View 层和 Model 层连接了起来，而View 和 Model 之间的同步工作完全是自动的，无需人为干涉，因此开发者只需关注业务逻辑，不需要手动操作DOM, 不需要关注数据状态的同步问题，复杂的数据状态维护完全由 MVVM 来统一管理。

###怎么实现MVVM

1. 脏值检查：angularangular.js 是通过脏值检测的方式比对数据是否有变更，来决定是否更新视图。
2. 数据劫持：使用Object.defineProperty()方法把这些vm.data属性全部转成setter、getter方法。

[Vue MVVM的实现(defineProperty)](https://jancat.github.io/post/2019/vue-mvvm/)

[Vue3.0 MVVM的实现(proxy)](https://jancat.github.io/post/2019/vue-mvvm-proxy/)（Vue 3.0 使用了 **Proxy** 后，将会消除之前 Vue 2.x 中基于 `Object.defineProperty` 的实现所存在的一些限制：无法监听 **属性的添加和删除**、**数组索引值和长度的变更**。）

### MVVM 各模块职责

- **MVVM**: Vue 实例初始化，调用 **Observer** 数据劫持，调用 **Compiler** 解析模板；
- **Observer**: 利用Object.defineProperty数据劫持data全部属性，定义 setter 、getter，添加和通知订阅者；（所以vue不能新增属性必须事先定义，model->vm.data）
- **Compiler**: 解析模板初始化视图，收集模板中的数据依赖，创建订阅者订阅变化，绑定更新函数；（在文档碎片中操作dom节点，遍历正则匹配替换data属性，view->vm.$el）
- **Dep**：订阅中心，提供添加、移除、通知订阅的接口；
- **Watcher**: data 属性的订阅者，收到变化通知后调用更新函数更新视图。

![image-20190709192722478](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j5vbvssij31460m4n93.jpg)

###Object.defineProperty() 和 Proxy 实现 MVVM 数据劫持的区别

`Object.defineProperty()` 和 `Proxy` 实现 MVVM 数据劫持的区别是：`Object.defineProperty()` 需要为每个 key 定义 getter 和 setter，也就是需要遍历 `data` 下的全部子属性；而 `Proxy` 只需要为 `data` 本身和内部嵌套的对象创建代理，每个对象统一代理访问内部属性，对外提供代理的引用。

## 2.vue生命周期：

### **1. 在beforeCreate和created钩子函数之间的生命周期**

在这个生命周期之间，进行**初始化事件，进行数据的观测**，可以看到在**created**的时候数据已经和**data属性进行绑定**（放在data中的属性当值发生改变的同时，视图也会改变）。
注意看下：此时还是没有el选项

组件实例刚刚创建，**还未进行数据观测和事件配置**//这里不要被beforeCreate误导，实际上组件实例已经创建了

### **2. created钩子函数和beforeMount间的生命周期**

实例已经创建完成，并且**已经完成数据观测，属性和方法的运算，watch/event 事件回调。//常用！！！**

首先会判断对象是否有**el选项**。**如果有的话就继续向下编译，如果没有**el选项**，则停止编译，也就意味着停止了生命周期，直到在该vue实例上调用vm.$mount(el)。**

然后，我们往下看，**template**参数选项的有无对生命周期的影响。
（1）.如果vue实例对象中有template参数选项，则将其作为模板编译成render函数。
（2）.如果没有template选项，则将外部HTML作为模板编译。
（3）.可以看到template中的模板优先级要高于outer HTML的优先级。

所以综合排名优先级：
render函数选项 > template选项 > outer HTML.

### **3. beforeMount和mounted 钩子函数间的生命周期**

可以看到此时是给vue实例对象添加**$el成员**，并且替换掉挂在的DOM元素。因为在之前console中打印的结果可以看到**beforeMount**之前el上还是undefined。

模板编译之前，还没挂载，页面仍未展示，但**虚拟Dom已经配置**//先把坑占住了，到后面mounted挂载的时候再把值渲染进去

### **4. mounted**

模板编译之后，已经挂载，**此时才会渲染页面，才能看到页面上数据的展示//常用！！！**

**注意:** mounted 不会承诺所有的子组件也都一起被挂载。如果你希望等到整个视图都渲染完毕，可以用 **vm.$nextTick 替换掉 mounted**

在mounted之前h1中还是通过`{{message}}`进行占位的，因为此时还有挂在到页面上，还是JavaScript中的虚拟DOM形式存在的。在mounted之后可以看到h1中的内容发生了变化。

el 替换，并挂载到实例上去之后调用。实例已完成以下的配置：用上面编译好的html内容替换el属性指向的DOM对象。完成模板中的html渲染到html页面中。此过程中进行ajax交互。

### **5. beforeUpdate**（更新前）

在数据更新之前调用，发生在虚拟DOM重新渲染和打补丁之前。可以在该钩子中进一步地更改状态，不会触发附加的重渲染过程。 

### **6. updated**（更新后）    

在由于数据更改导致的虚拟DOM重新渲染和打补丁之后调用。调用时，组件DOM已经更新，所以可以执行依赖于DOM的操作。然而在大多数情况下，应该避免在此期间更改状态，因为这可能会导致更新无限循环。该钩子在服务器端渲染期间不被调用。

### **7. beforeDestroy**（销毁前）    

在实例销毁之前调用。实例仍然完全可用。 

### **8. destroyed**（销毁后）    

在实例销毁之后调用。调用后，所有的事件监听器会被移除，所有的子实例也会被销毁。该钩子在服务器端渲染期间不被调用。 

### 关于生命周期的问题：

1.什么是vue生命周期？ 答： Vue 实例从创建到销毁的过程，就是生命周期。从开始创建、初始化数据、编译模板、挂载Dom→渲染、更新→渲染、销毁等一系列过程，称之为 Vue 的生命周期。

2.vue生命周期的作用是什么？ 答：它的生命周期中有多个事件钩子，让我们在控制整个Vue实例的过程时更容易形成好的逻辑。

3.vue生命周期总共有几个阶段？ 答：它可以总共分为8个阶段：创建前/后, 载入前/后,更新前/后,销毁前/销毁后。

4.第一次页面加载会触发哪几个钩子？ 答：会触发 下面这几个beforeCreate, created, beforeMount, mounted 

5.DOM 渲染在 哪个周期中就已经完成？ 答：DOM 渲染在 mounted 中就已经完成了。

## 3.Vue实现数据双向绑定的原理：Object.defineProperty（）

vue实现数据双向绑定主要是：采**用数据劫持结合发布者-订阅者模式**的方式，通过**Object.defineProperty（）**来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应监听回调。当把一个普通 Javascript 对象传给 Vue 实例来作为它的 data 选项时，Vue 将遍历它的属性，用 Object.defineProperty 将它们转为 getter/setter。用户看不到 getter/setter，但是在内部它们让 Vue 追踪依赖，在属性被访问和修改时通知变化。

vue的数据双向绑定 将MVVM作为数据绑定的入口，整合Observer，Compile和Watcher三者，通过Observer来监听自己的model的数据变化，通过Compile来解析编译模板指令（vue中是用来解析双花括号的），最终利用watcher搭起observer和Compile之间的通信桥梁，达到数据变化 —>视图更新；视图交互变化（input）—>数据model变更双向绑定效果。

### 关于Object.defineProperty

- 语法

  `Object.defineProperty(obj,prop,descriptor)`

- 参数

  obj:目标对象

  prop:需要定义的属性或方法的名称

  descriptor:目标属性所拥有的特性

- 可供定义的特性列表

  value:属性的值

  writable:如果为false，属性的值就不能被重写。

  get: 一旦目标属性被访问就会调回此方法，并将此方法的运算结果返回用户。

  set:一旦目标属性被赋值，就会调回此方法。

  configurable:如果为false，则任何尝试删除目标属性或修改属性性以下特性（writable, configurable, enumerable）的行为将被无效化。

  enumerable:是否能在for...in循环中遍历出来或在Object.keys中列举出来。



**js实现简单的双向绑定**

```html
<body>
  <div id="app">
    <input type="text"id="txt"/>
    <p id="show"></p>
  </div>
</body>
<script type="text/javascript">
  	var obj = {}
    Object.defineProperty(obj, 'txt', {
        get: function () {
            return obj
        },
        set: function (newValue) {
            document.getElementById('txt').value = newValue
            document.getElementById('show').innerHTML = newValue
        }
    })
    document.addEventListener('keyup', function (e) {
        obj.txt = e.target.value
    })
</script>
```

## 4. Vue的路由实现：hash模式 和 history模式

**hash模式：**在浏览器中符号“#”，#以及#后面的字符称之为hash，用window.location.hash读取； 特点：hash虽然在URL中，但不被包括在HTTP请求中；用来指导浏览器动作，对服务端安全无用，hash不会重加载页面。 hash 模式下，仅 hash 符号之前的内容会被包含在请求中，如 [www.xxx.com](https://link.juejin.im?target=http%3A%2F%2Fwww.xxx.com)，因此对于后端来说，即使没有做到对路由的全覆盖，也不会返回 404 错误。

**history模式：**history采用HTML5的新特性；且提供了两个新方法：pushState（），replaceState（）可以对浏览器历史记录栈进行修改，以及popState事件的监听到状态变更。 history 模式下，前端的 URL 必须和实际向后端发起请求的 URL 一致，如 [www.xxx.com/items/id](https://link.juejin.im?target=http%3A%2F%2Fwww.xxx.com%2Fitems%2Fid)。后端如果缺少对 /items/id 的路由处理，将返回 404 错误。**Vue-Router 官网里如此描述：**“不过这种模式要玩好，还需要后台配置支持……所以呢，你要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面。”

## 5. Vue与Angular以及React的区别？

**1.与AngularJS的区别** 

相同点： 都支持指令：内置指令和自定义指令；都支持过滤器：内置过滤器和自定义过滤器；都支持双向数据绑定；都不支持低端浏览器。

不同点： AngularJS的学习成本高，比如增加了Dependency Injection特性，而Vue.js本身提供的API都比较简单、直观；在性能上，AngularJS依赖对数据做脏检查，所以Watcher越多越慢；Vue.js使用基于依赖追踪的观察并且使用异步队列更新，所有的数据都是独立触发的。

**2.与React的区别** （详见vue和react区别）

相同点： 

* React采用特殊的JSX语法，Vue.js在组件开发中也推崇编写.vue特殊文件格式，对文件内容都有一些约定，两者都需要编译后使用；
* 中心思想相同：一切都是组件，组件实例之间可以嵌套；
* 都提供合理的钩子函数，可以让开发者定制化地去处理需求；
* 在组件开发中都支持mixins的特性。
* 使用 Virtual DOM，有自己的diff渲染算法
* 提供了响应式 (Reactive) 和组件化 (Composable) 的视图组件。
* 将注意力集中保持在核心库，而将其他功能如路由和全局状态管理交给相关的库。都不内置列数AJAX，Route等功能到核心包，而是以插件的方式加载。

 不同点：

* React采用的Virtual DOM会对渲染出来的结果做脏检查；在 React 应用中，当某个组件的状态发生变化时，它会以该组件为根，重新渲染整个组件子树。

  如要避免不必要的子组件的重渲染，你需要在所有可能的地方使用 PureComponent，或是手动实现 shouldComponentUpdate 方法。同时你可能会需要使用不可变的数据结构来使得你的组件更容易被优化。

  然而，使用 PureComponent 和 shouldComponentUpdate 时，需要保证该组件的整个子树的渲染输出都是由该组件的 props 所决定的。如果不符合这个情况，那么此类优化就会导致难以察觉的渲染结果不一致。这使得 React 中的组件优化伴随着相当的心智负担。

  在 Vue 应用中，组件的依赖是在渲染过程中自动追踪的，所以系统能精确知晓哪个组件确实需要被重渲染。你可以理解为每一个组件都已经自动获得了 shouldComponentUpdate，并且没有上述的子树问题限制。

* Vue.js在模板中提供了指令，过滤器等，可以非常方便，快捷地操作Virtual DOM。

## 6. vue路由的钩子函数

首页可以控制导航跳转，beforeEach，afterEach等，一般用于页面title的修改。一些需要登录才能调整页面的重定向功能。

**beforeEach**主要有3个参数to，from，next：

**to**：route即将进入的目标路由对象，

**from**：route当前导航正要离开的路由

**next**：function一定要调用该方法resolve这个钩子。执行效果依赖next方法的调用参数。可以控制网页的跳转。

## 7. vue中常用的指令有哪些？

### 指令

- 解释：指令 (Directives) 是带有 `v-` 前缀的特殊属性
- 作用：当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM

### 常用指令

#### 1.v-text：更新元素的 textContent

```
<h1 v-text="msg"></h1>
```

#### 2.v-html：更新元素的 innerHTML

```
<h1 v-html="msg"></h1>
```

#### 3.v-bind

- 作用：当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM
- 语法：`v-bind:title="msg"`
- 简写：`:title="msg"`

```vue
<!-- 完整语法 -->
<a v-bind:href="url"></a>
<!-- 缩写 -->
<a :href="url"></a>
<script>
    // 2 创建 Vue 的实例对象
    var vm = new Vue({
      // el 用来指定vue挂载到页面中的元素，值是：选择器
      // 理解：用来指定vue管理的HTML区域
      el: '#app',
      // 数据对象，用来给视图中提供数据的
      data: {
        url: 'http://www.baidu.com'
      }
    })
  </script>
```

#### 4.v-on

- 作用：绑定事件
- 语法：`v-on:click="say"` or `v-on:click="say('参数', $event)"`
- 简写：`@click="say"`
- 说明：绑定的事件从`methods`中获取

```vue
<!-- 完整语法 -->
<a v-on:click="doSomething"></a>
<!-- 缩写 -->
<a @click="doSomething"></a>
<!-- 方法传参 -->
<a @click="doSomething（“123”）"></a>

 <script>
    // 2 创建 Vue 的实例对象
    var vm = new Vue({
      el: '#app',
      // methods属性用来给vue实例提供方法（事件）
      methods: {
        doSomething: function(str) {
          //接受参数，并输出
          console.log(str);
        }
      }
    })
  </script>
```

#### 5.v-model

数据响应，是通过数据的改变去驱动 DOM 视图的变化，而双向绑定除了数据驱动 DOM 外， DOM 的变化反过来影响数据，是一个双向关系，在 Vue 中，我们可以通过 `v-model` 来实现双向绑定。

- 作用：在表单元素上创建双向数据绑定
- 说明：监听用户的输入事件以更新数据

```vue
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```

##### v-model只不过是一个语法糖而已,真正的实现靠的还是

* v-bind:绑定响应式数据
* 触发 input 事件，并传递数据 (重点)

##### v-model 在不同的 HTML 标签上使用会监控不同的属性和抛出不同的事件：

- text 和 textarea 元素使用 `value` 属性和 `input` 事件；
- checkbox 和 radio 使用 `checked` 属性和 `change` 事件；
- select 字段将 `value` 作为 prop 并将 `change` 作为事件。

[v-model源码分析]([https://ustbhuangyi.github.io/vue-analysis/extend/v-model.html#%E8%A1%A8%E5%8D%95%E5%85%83%E7%B4%A0](https://ustbhuangyi.github.io/vue-analysis/extend/v-model.html#表单元素))

#### 6.v-for

- 作用：基于源数据多次渲染元素或模板块
- key属性
  - 推荐：使用 `v-for` 的时候提供 `key` 属性，以获得性能提升。
  - 说明：使用 key，VUE会基于 key 的变化重新排列元素顺序，并且会移除 key 不存在的元素。

```vue
<!-- 1 基础用法 -->
<div v-for="item in items">
  {{ item.text }}
</div>

<!-- item 为当前项，index 为索引 -->
<p v-for="(item, index) in list">{{item}} -- {{index}}</p>
<!-- item 为值，key 为键，index 为索引 -->
<p v-for="(item, key, index) in obj">{{item}} -- {{key}}</p>
<p v-for="item in 10">{{item}}</p>
```

#### 7.v-if 和 v-show

- [条件渲染](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Fconditional.html)
- `v-if`：根据表达式的值的真假条件，销毁或重建元素
- `v-show`：根据表达式之真假值，切换元素的 display CSS 属性

#### 8.提升用户体验：v-cloak

- 这个指令保持在元素上直到关联实例结束编译。和 CSS 规则如 [v-cloak] { display: none } 一起用时，这个指令可以隐藏未编译的 Mustache 标签直到实例准备完毕。
- 防止刷新页面，网速慢的情况下出现`{{ message }}`等数据格式

```vue
<div v-cloak>
  {{ message }}
</div>
```

#### 9.提升性能：v-pre

- 说明：跳过这个元素和它的子元素的编译过程。可以用来显示原始 Mustache 标签。跳过大量没有指令的节点会加快编译。

```
<span v-pre>{{ this will not be compiled }}</span>
```

#### 10.提升性能：v-once

- 说明：只渲染元素和组件一次。随后的重新渲染，元素/组件及其所有的子节点将被视为静态内容并跳过。这可以用于优化更新性能。

```
<span v-once>This will never change: {{msg}}</span>
```

### 自定义指令

除了核心功能默认内置的指令 (`v-model` 和 `v-show`)，Vue 也允许注册自定义指令。注意，在 Vue2.0 中，代码复用和抽象的主要形式是组件。然而，有的情况下，你仍然需要对普通 DOM 元素进行底层操作，这时候就会用到自定义指令。

例如：

当页面加载时，该元素将获得焦点 (注意：`autofocus` 在移动版 Safari 上不工作)。事实上，只要你在打开这个页面后还没点击过任何内容，这个输入框就应当还是处于聚焦状态。现在让我们用指令来实现这个功能：

```vue
// 注册一个全局自定义指令 `v-focus`
Vue.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时……
  inserted: function (el) {
    // 聚焦元素
    el.focus()
  }
})
```

如果想注册局部指令，组件中也接受一个 `directives` 的选项：

```vue
directives: {
  focus: {
    // 指令的定义
    inserted: function (el) {
      el.focus()
    }
  }
}
```

然后你可以在模板中任何元素上使用新的 `v-focus` 属性，如下：

```vue
<input v-focus>
```

#### 钩子函数

一个指令定义对象可以提供如下几个钩子函数 (均为可选)：

- `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
- `inserted`：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
- `update`：所在组件的 VNode 更新时调用，**但是可能发生在其子 VNode 更新之前**。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。
- `componentUpdated`：指令所在组件的 VNode **及其子 VNode** 全部更新后调用。
- `unbind`：只调用一次，指令与元素解绑时调用。

#### 钩子函数参数

指令钩子函数会被传入以下参数：

- `el`：指令所绑定的元素，可以用来直接操作 DOM 。

- `binding`：一个对象，包含以下属性：

  - `name`：指令名，不包括 `v-` 前缀。
  - `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 `2`。
  - `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
  - `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 `"1 + 1"`。
  - `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
  - `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`。

- `vnode`：Vue 编译生成的虚拟节点。移步 [VNode API](https://cn.vuejs.org/v2/api/#VNode-接口) 来了解更多详情。

- `oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用。

  除了 `el` 之外，其它参数都应该是只读的，切勿进行修改。如果需要在钩子之间共享数据，建议通过元素的 [`dataset`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/dataset) 来进行。

```vue
<div id="dynamicexample">
  <h3>Scroll down inside this section ↓</h3>
  <p v-pin:[direction]="200">I am pinned onto the page at 200px to the left.</p>
</div>
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    var s = (binding.arg == 'left' ? 'left' : 'top')
    el.style[s] = binding.value + 'px'
  }
})

new Vue({
  el: '#dynamicexample',
  data: function () {
    return {
      direction: 'left'
    }
  }
})
```

## 8. 一句话就能回答的面试题

**1.css只在当前组件起作用** 答：在style标签中写入**scoped**即可 例如：

**2.v-if 和 v-show 区别** 答：v-if按照条件是否渲染，v-show是display的block或none；

**3.route和router的区别** 答：route是"路由信息对象"，包括path，params，hash，query，fullPath，matched，name等路由信息参数。而router是“路由实例”对象包括了路由的跳转方法，钩子函数等。

**4.vue.js的两个核心是什么？** 答：数据驱动、组件系统

**5.vue几种常用的指令** 答：v-for 、 v-if 、v-bind、v-on、v-show、v-else

**6.vue常用的修饰符？** 答：.prevent: 提交事件不再重载页面；.stop: 阻止单击事件冒泡；.self: 当事件发生在该元素本身而不是子元素的时候会触发；.capture: 事件侦听，事件发生的时候会调用

**7.v-on 可以绑定多个方法吗？** 答：可以

**8.vue中 key 值的作用？** 答：当 Vue.js 用 v-for 正在更新已渲染过的元素列表时，它默认用“就地复用”策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序， 而是简单复用此处每个元素，并且确保它在特定索引下显示已被渲染过的每个元素。key的作用主要是为了高效的更新虚拟DOM。

**9.什么是vue的计算属性？** 答：在模板中放入太多的逻辑会让模板过重且难以维护，在需要对数据进行复杂处理，且可能多次使用的情况下，尽量采取计算属性的方式。好处：①使得数据处理结构清晰；②依赖于数据，数据更新，处理结果自动更新；③计算属性内部this指向vm实例；④在template调用时，直接写计算属性名即可；⑤常用的是getter方法，获取数据，也可以使用set方法改变数据；⑥相较于methods，不管依赖的数据变不变，methods都会重新计算，但是依赖数据不变的时候computed从缓存中获取，不会重新计算。

**10.vue等单页面应用及其优缺点** 答：优点：Vue 的目标是通过尽可能简单的 API 实现响应的数据绑定和组合的视图组件，核心是一个响应的数据绑定系统。MVVM、数据驱动、组件化、轻量、简洁、高效、快速、模块友好。 ==**缺点：**不支持低版本的浏览器，最低只支持到IE9；不利于SEO的优化（如果要支持SEO，建议通过服务端来进行渲染组件）；第一次加载首页耗时相对长一些；不可以使用浏览器的导航按钮需要自行实现前进、后退。==

