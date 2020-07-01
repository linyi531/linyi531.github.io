---
title: VUE和REACT简要对比
date: 2020-06-11 21:52:10
tags:
  - React
  - VUE
categories: React
cover_img: https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbqn2lfmhj31900u01l3.jpg
feature_img: https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbqn2lfmhj31900u01l3.jpg
---

# VUE和REACT简要对比

## 1.相同点： 

- React采用特殊的JSX语法，Vue.js在组件开发中也推崇编写.vue特殊文件格式，对文件内容都有一些约定，两者都需要编译后使用；
- 中心思想相同：一切都是组件，组件实例之间可以嵌套；
- 都提供合理的钩子函数，可以让开发者定制化地去处理需求；
- 在组件开发中都支持mixins的特性。
- 使用 Virtual DOM，有自己的diff渲染算法
- 提供了响应式 (Reactive) 和组件化 (Composable) 的视图组件。
- 将注意力集中保持在核心库，而将其他功能如路由和全局状态管理交给相关的库。都不内置列数AJAX，Route等功能到核心包，而是以插件的方式加载。

 ## 2.Virtual DOM渲染不同点：

Virtual DOM是一个映射真实DOM的JavaScript对象，如果需要改变任何元素的状态，那么是先在Virtual DOM上进行改变，而不是直接改变真实的DOM。当有变化产生时，一个新的Virtual DOM对象会被创建并计算新旧Virtual DOM之间的差别。之后这些差别会应用在真实的DOM上。

### React：

React采用的Virtual DOM会对渲染出来的结果做脏检查；在 React 应用中，当某个组件的状态发生变化时，它会以该组件为根，重新渲染整个组件子树。

如要避免不必要的子组件的重渲染，你需要在所有可能的地方使用 PureComponent，或是手动实现 shouldComponentUpdate 方法。同时你可能会需要使用不可变的数据结构来使得你的组件更容易被优化。

然而，使用 PureComponent 和 shouldComponentUpdate 时，需要保证该组件的整个子树的渲染输出都是由该组件的 props 所决定的。如果不符合这个情况，那么此类优化就会导致难以察觉的渲染结果不一致。这使得 React 中的组件优化伴随着相当的心智负担。

### Vue：

在 Vue 应用中，组件的依赖是在渲染过程中自动追踪的，所以系统能精确知晓哪个组件确实需要被重渲染。你可以理解为每一个组件都已经自动获得了 shouldComponentUpdate，并且没有上述的子树问题限制。

Vue.js在模板中提供了指令，过滤器等，可以非常方便，快捷地操作Virtual DOM。

## 3.状态管理 vs 对象属性

### React：

React在state状态管理存储数据的，不能修改数据，修改数据在Setstate中 setState是异步的，如果需要马上利用结果，需要在setState传入回调，具体可以看看[ React中setState几个现象---先知道再理解](https://link.juejin.im?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000014498196)

### Vue：

在Vue中，state对象并不是必须的，数据由data属性在Vue对象中进行管理

## 4.JSX vs Templates

### React：

在 React 中，所有的组件的渲染功能都依靠 JSX。 使用 JSX 的渲染函数有下面这些优势：

- 你可以使用完整的编程语言 JavaScript 功能来构建你的视图页面。比如你可以使用临时变量、JS 自带的流程控制、以及直接引用当前 JS 作用域中的值等等。
- 开发工具对 JSX 的支持相比于现有可用的其他 Vue 模板还是比较先进的 (比如，linting、类型检查、编辑器的自动完成)。

### Vue：

虽然Vue也可以使用JSX，但基本都使用模版语法，这也带来了一些特有的优势：

- 对于很多习惯了 HTML 的开发者来说，模板比起 JSX 读写起来更自然。这里当然有主观偏好的成分，但如果这种区别会导致开发效率的提升，那么它就有客观的价值存在。
- 基于 HTML 的模板使得将已有的应用逐步迁移到 Vue 更为容易。

## 5.组件作用域内的 CSS

### Vue：

设置样式的默认方法是单文件组件里类似 style 的标签。 单文件组件让你可以在同一个文件里完全控制 CSS，将其作为组件代码的一部分。

```css
<style scoped>
  .container{
      display:flex;
  }
</style>
```

这个可选 scoped 属性会自动添加一个唯一的属性 (比如 data-v-8123) 为组件内 CSS 指定作用域。

### React:

语法不太一样，React设置class是用className字段，而设置css是使用对象的形式，当然，一般还是引入外部的css(经过编译的sass或者less文件)比较合适。

## 6.规模

Vue 和 React 都提供了强大的路由来应对大型应用。React 社区在状态管理方面非常有创新精神 (比如 Flux、Redux)，而这些状态管理模式甚至 Redux 本身也可以非常容易的集成在 Vue 应用中。实际上，Vue 更进一步地采用了这种模式 (Vuex)，更加深入集成 Vue 的状态管理解决方案 Vuex 相信能为你带来更好的开发体验。

两者另一个重要差异是，Vue 的路由库和状态管理库都是由官方维护支持且与核心库同步更新的。React 则是选择把这些问题交给社区维护，因此创建了一个更分散的生态系统。但相对的，React 的生态系统相比 Vue 更加繁荣。

## 7.构建工具

React和Vue都有自己的构建工具，你可以使用它快速搭建开发环境。

### React：

React可以使用Create React App (CRA)，由于CRA有很多选项，使用起来会稍微麻烦一点。这个工具会逼迫你使用Webpack和Babel。

### Vue：

Vue对应的则是vue-cli。vue-cli则有模板列表可选，能按需创造不同模板，使用起来更灵活一点。

两个工具都能让你得到一个根据最佳实践设置的项目模板。都能为你建立一个好环境。



# react

## React是什么特性的，有什么用？配套的全家桶是什么？

- React不是一个MVC框架，它是构建易于可重复调用的web组件，侧重于UI, 也就是view层
- 其次React是单向的从数据到视图的渲染，非双向数据绑定
- 不直接操作DOM对象，而是通过虚拟DOM通过diff算法以最小的步骤作用到真实的DOM上。
- 不便于直接操作DOM，大多数时间只是对 virtual DOM 进行编程

配套redux