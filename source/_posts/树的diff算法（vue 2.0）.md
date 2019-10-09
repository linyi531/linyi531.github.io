---
title: 树的diff算法（vue 2.0）
date: 2019-02-26 00:26:33
tags:
  - VUE
categories: VUE
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77ha49my6j32aq0u0npf.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77ha49my6j32aq0u0npf.jpg
---

# 树的 diff 算法（vue 2.0）

## 模板转换成视图的过程

- Vue.js 通过编译将 template 模板转换成渲染函数(render ) ，执行渲染函数就可以得到一个虚拟节点树
- 在对 Model 进行操作的时候，会触发对应 Dep 中的 Watcher 对象。Watcher 对象会调用对应的 update 来修改视图。这个过程主要是将新旧虚拟节点进行差异对比，然后根据对比结果进行 DOM 操作来更新视图。

简单点讲，在 Vue 的底层实现上，Vue 将模板编译成虚拟 DOM 渲染函数。结合 Vue 自带的响应系统，在状态改变时，Vue 能够智能地计算出重新渲染组件的最小代价并应到 DOM 操作上。

<!-- more -->

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6ixrpspcuj30qd07hdge.jpg)

- **渲染函数**：渲染函数是用来生成 Virtual DOM 的。Vue 推荐使用模板来构建我们的应用界面，在底层实现中 Vue 会将模板编译成渲染函数，当然我们也可以不写模板，直接写渲染函数，以获得更好的控制。
- **VNode 虚拟节点**：它可以代表一个真实的 dom 节点。通过 createElement 方法能将 VNode 渲染成 dom 节点。简单地说，vnode 可以理解成**节点描述对象**，它描述了应该怎样去创建真实的 DOM 节点。
- **patch(也叫做 patching 算法)**：虚拟 DOM 最核心的部分，它可以将 vnode 渲染成真实的 DOM，这个过程是对比新旧虚拟节点之间有哪些不同，然后根据对比结果找出需要更新的的节点进行更新。这点我们从单词含义就可以看出， patch 本身就有补丁、修补的意思，其实际作用是在现有 DOM 上进行修改来实现更新视图的目的。Vue 的 Virtual DOM Patching 算法是基于[Snabbdom](https://github.com/snabbdom/snabbdom)的实现，并在些基础上作了很多的调整和改进。

## Virtual DOM 是什么？

Virtual DOM 其实就是一棵以 JavaScript 对象( VNode 节点)作为基础的树，用对象属性来描述节点，实际上它只是一层对真实 DOM 的抽象。最终可以通过一系列操作使这棵树映射到真实环境上。

简单来说，可以把 Virtual DOM 理解为一个简单的 JS 对象，并且最少包含标签名( tag)、属性(attrs)和子元素对象( children)三个属性。不同的框架对这三个属性的命名会有点差别。

对于虚拟 DOM，咱们来看一个简单的实例，就是下图所示的这个，详细的阐述了`模板 → 渲染函数 → 虚拟DOM树 → 真实DOM`的一个过程
![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6ixs748iqj30yg06emxd.jpg)

## Virtual DOM 作用是什么？

**虚拟 DOM 的最终目标是将虚拟节点渲染到视图上**。但是如果直接使用虚拟节点覆盖旧节点的话，会有很多不必要的 DOM 操作。例如，一个 ul 标签下很多个 li 标签，其中只有一个 li 有变化，这种情况下如果使用新的 ul 去替代旧的 ul,因为这些不必要的 DOM 操作而造成了性能上的浪费。

为了避免不必要的 DOM 操作，虚拟 DOM 在虚拟节点映射到视图的过程中，将虚拟节点与上一次渲染视图所使用的旧虚拟节点（oldVnode）做对比，找出真正需要更新的节点来进行 DOM 操作，从而避免操作其他无需改动的 DOM。

其实虚拟 DOM 在 Vue.js 主要做了两件事：

- 提供与真实 DOM 节点所对应的虚拟节点 vnode
- 将虚拟节点 vnode 和旧虚拟节点 oldVnode 进行对比，然后更新视图

## 为何需要 Virtual DOM？

- 具备跨平台的优势

由于 Virtual DOM 是以 JavaScript 对象为基础而不依赖真实平台环境，所以使它具有了跨平台的能力，比如说浏览器平台、Weex、Node 等。

- 操作 DOM 慢，js 运行效率高。我们可以将 DOM 对比操作放在 JS 层，提高效率。

因为 DOM 操作的执行速度远不如 Javascript 的运算速度快，因此，把大量的 DOM 操作搬运到 Javascript 中，运用 patching 算法来计算出真正需要更新的节点，最大限度地减少 DOM 操作，从而显著提高性能。

Virtual DOM 本质上就是在 JS 和 DOM 之间做了一个缓存。可以类比 CPU 和硬盘，既然硬盘这么慢，我们就在它们之间加个缓存：既然 DOM 这么慢，我们就在它们 JS 和 DOM 之间加个缓存。CPU（JS）只操作内存（Virtual DOM），最后的时候再把变更写入硬盘（DOM）

- 提升渲染性能

Virtual DOM 的优势不在于单次的操作，而是在大量、频繁的数据更新下，能够对视图进行合理、高效的更新。

为了实现高效的 DOM 操作，一套高效的虚拟 DOM diff 算法显得很有必要。**我们通过 patch 的核心----diff 算法，找出本次 DOM 需要更新的节点来更新，其他的不更新**。

## VNode

### 抽象 Dom 树

把真实 Dom 树抽象成一棵以 javascript 对象构成的抽象树，在修改抽象树数据后将抽象树转化成真实 Dom 重绘到页面上呢？于是虚拟 Dom 出现了，它是真实 Dom 的一层抽象，用属性描述真实 Dom 的各个特性。当它发生变化的时候，就会去修改视图。

但是这样的 javascript 操作 Dom 进行重绘整个视图层是相当消耗性能的，我们是不是可以每次只更新它的修改呢？所以 Vue.js 将 Dom 抽象成一个以 javascript 对象为节点的虚拟 Dom 树，以 VNode 节点模拟真实 Dom，可以对这颗抽象树进行创建节点、删除节点以及修改节点等操作，在这过程中都不需要操作真实 Dom，只需要操作 javascript 对象，大大提升了性能。修改以后经过 diff 算法得出一些需要修改的最小单位，再将这些小单位的视图进行更新。这样做减少了很多不需要的 Dom 操作，大大提高了性能。

Vue 就使用了这样的抽象节点 VNode，它是对真实 Dom 的一层抽象，而不依赖某个平台，它可以是浏览器平台，也可以是 weex，甚至是 node 平台也可以对这样一棵抽象 Dom 树进行创建删除修改等操作，这也为前后端同构提供了可能。

### VNode 基类

先来看一下 Vue.js 源码中对 VNode 类的定义。

```javascript
export default class VNode {
  tag: string | void;
  data: VNodeData | void;
  children: ?Array<VNode>;
  text: string | void;
  elm: Node | void;
  ns: string | void;
  context: Component | void; // rendered in this component's scope
  functionalContext: Component | void; // only for functional component root nodes
  key: string | number | void;
  componentOptions: VNodeComponentOptions | void;
  componentInstance: Component | void; // component instance
  parent: VNode | void; // component placeholder node
  raw: boolean; // contains raw HTML? (server only)
  isStatic: boolean; // hoisted static node
  isRootInsert: boolean; // necessary for enter transition check
  isComment: boolean; // empty comment placeholder?
  isCloned: boolean; // is a cloned node?
  isOnce: boolean; // is a v-once node?

  constructor(
    tag?: string,
    data?: VNodeData,
    children?: ?Array<VNode>,
    text?: string,
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions
  ) {
    /*当前节点的标签名*/
    this.tag = tag;
    /*当前节点对应的对象，包含了具体的一些数据信息，是一个VNodeData类型，可以参考VNodeData类型中的数据信息*/
    this.data = data;
    /*当前节点的子节点，是一个数组*/
    this.children = children;
    /*当前节点的文本*/
    this.text = text;
    /*当前虚拟节点对应的真实dom节点*/
    this.elm = elm;
    /*当前节点的名字空间*/
    this.ns = undefined;
    /*编译作用域*/
    this.context = context;
    /*函数化组件作用域*/
    this.functionalContext = undefined;
    /*节点的key属性，被当作节点的标志，用以优化*/
    this.key = data && data.key;
    /*组件的option选项*/
    this.componentOptions = componentOptions;
    /*当前节点对应的组件的实例*/
    this.componentInstance = undefined;
    /*当前节点的父节点*/
    this.parent = undefined;
    /*简而言之就是是否为原生HTML或只是普通文本，innerHTML的时候为true，textContent的时候为false*/
    this.raw = false;
    /*静态节点标志*/
    this.isStatic = false;
    /*是否作为跟节点插入*/
    this.isRootInsert = true;
    /*是否为注释节点*/
    this.isComment = false;
    /*是否为克隆节点*/
    this.isCloned = false;
    /*是否有v-once指令*/
    this.isOnce = false;
  }

  // DEPRECATED: alias for componentInstance for backwards compat.
  /* istanbul ignore next https://github.com/answershuto/learnVue*/
  get child(): Component | void {
    return this.componentInstance;
  }
}
```

这是一个最基础的 VNode 节点，作为其他派生 VNode 类的基类，里面定义了下面这些数据。

- tag: 当前节点的标签名
- data: 当前节点对应的对象，包含了具体的一些数据信息，是一个 VNodeData 类型，可以参考 VNodeData 类型中的数据信息
- children: 当前节点的子节点，是一个数组
- text: 当前节点的文本
- elm: 当前虚拟节点对应的真实 dom 节点
- ns: 当前节点的名字空间
- context: 当前节点的编译作用域
- functionalContext: 函数化组件作用域
- key: 节点的 key 属性，被当作节点的标志，用以优化
- componentOptions: 组件的 option 选项
- componentInstance: 当前节点对应的组件的实例
- parent: 当前节点的父节点
- raw: 简而言之就是是否为原生 HTML 或只是普通文本，innerHTML 的时候为 true，textContent 的时候为 false
- isStatic: 是否为静态节点
- isRootInsert: 是否作为跟节点插入
- isComment: 是否为注释节点
- isCloned: 是否为克隆节点
- isOnce: 是否有 v-once 指令

打个比方，比如说我现在有这么一个 VNode 树

```json
{
    tag: 'div'
    data: {
        class: 'test'
    },
    children: [
        {
            tag: 'span',
            data: {
                class: 'demo'
            }
            text: 'hello,VNode'
        }
    ]
}
```

渲染之后的结果就是这样的

```html
<div class="test">
  <span class="demo">hello,VNode</span>
</div>
```

### 生成一个新的 VNode 的方法

下面这些方法都是一些常用的构造 VNode 的方法。

- **createEmptyVNode 创建一个空 VNode 节点**

```javascript
/*创建一个空VNode节点*/
export const createEmptyVNode = () => {
  const node = new VNode();
  node.text = "";
  node.isComment = true;
  return node;
};
```

- **createTextVNode 创建一个文本节点**

```javascript
/*创建一个文本节点*/
export function createTextVNode(val: string | number) {
  return new VNode(undefined, undefined, undefined, String(val));
}
```

- **createComponent 创建一个组件节点**

```javascript
 // plain options object: turn it into a constructor https://github.com/answershuto/learnVue
  if (isObject(Ctor)) {
    Ctor = baseCtor.extend(Ctor)
  }

  // if at this stage it's not a constructor or an async component factory,
  // reject.
  /*Github:https://github.com/answershuto*/
  /*如果在该阶段Ctor依然不是一个构造函数或者是一个异步组件工厂则直接返回*/
  if (typeof Ctor !== 'function') {
    if (process.env.NODE_ENV !== 'production') {
      warn(`Invalid Component definition: ${String(Ctor)}`, context)
    }
    return
  }

  // async component
  /*处理异步组件*/
  if (isUndef(Ctor.cid)) {
    Ctor = resolveAsyncComponent(Ctor, baseCtor, context)
    if (Ctor === undefined) {
      // return nothing if this is indeed an async component
      // wait for the callback to trigger parent update.
      /*如果这是一个异步组件则会不会返回任何东西（undifiened），直接return掉，等待回调函数去触发父组件更新。s*/
      return
    }
  }

  // resolve constructor options in case global mixins are applied after
  // component constructor creation
  resolveConstructorOptions(Ctor)

  data = data || {}

  // transform component v-model data into props & events
  if (isDef(data.model)) {
    transformModel(Ctor.options, data)
  }

  // extract props
  const propsData = extractPropsFromVNodeData(data, Ctor, tag)

  // functional component
  if (isTrue(Ctor.options.functional)) {
    return createFunctionalComponent(Ctor, propsData, data, context, children)
  }

  // extract listeners, since these needs to be treated as
  // child component listeners instead of DOM listeners
  const listeners = data.on
  // replace with listeners with .native modifier
  data.on = data.nativeOn

  if (isTrue(Ctor.options.abstract)) {
    // abstract components do not keep anything
    // other than props & listeners
    data = {}
  }

  // merge component management hooks onto the placeholder node
  mergeHooks(data)

  // return a placeholder vnode
  const name = Ctor.options.name || tag
  const vnode = new VNode(
    `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
    data, undefined, undefined, undefined, context,
    { Ctor, propsData, listeners, tag, children }
  )
  return vnode
}
```

- **cloneVNode 克隆一个 VNode 节点**

```javascript
export function cloneVNode(vnode: VNode): VNode {
  const cloned = new VNode(
    vnode.tag,
    vnode.data,
    vnode.children,
    vnode.text,
    vnode.elm,
    vnode.context,
    vnode.componentOptions
  );
  cloned.ns = vnode.ns;
  cloned.isStatic = vnode.isStatic;
  cloned.key = vnode.key;
  cloned.isCloned = true;
  return cloned;
}
```

### createElement

createElement 用来创建一个虚拟节点。当 data 上已经绑定**ob**的时候，代表该对象已经被 Oberver 过了，所以创建一个空节点。tag 不存在的时候同样创建一个空节点。当 tag 不是一个 String 类型的时候代表 tag 是一个组件的构造类，直接用 new VNode 创建。当 tag 是 String 类型的时候，如果是保留标签，则用 new VNode 创建一个 VNode 实例，如果在 vm 的 option 的 components 找得到该 tag，代表这是一个组件，否则统一用 new VNode 创建。

```javascript
// wrapper function for providing a more flexible interface
// without getting yelled at by flow
export function createElement(
  context: Component,
  tag: any,
  data: any,
  children: any,
  normalizationType: any,
  alwaysNormalize: boolean
): VNode {
  /*兼容不传data的情况*/
  if (Array.isArray(data) || isPrimitive(data)) {
    normalizationType = children;
    children = data;
    data = undefined;
  }
  /*如果alwaysNormalize为true，则normalizationType标记为ALWAYS_NORMALIZE*/
  if (isTrue(alwaysNormalize)) {
    normalizationType = ALWAYS_NORMALIZE;
  }
  /*Github:https://github.com/answershuto*/
  /*创建虚拟节点*/
  return _createElement(context, tag, data, children, normalizationType);
}

/*创建虚拟节点*/
export function _createElement(
  context: Component,
  tag?: string | Class<Component> | Function | Object,
  data?: VNodeData,
  children?: any,
  normalizationType?: number
): VNode {
  /*
    如果data未定义（undefined或者null）或者是data的__ob__已经定义（代表已经被observed，上面绑定了Oberver对象），
    https://cn.vuejs.org/v2/guide/render-function.html#约束
    那么创建一个空节点
  */
  if (isDef(data) && isDef((data: any).__ob__)) {
    process.env.NODE_ENV !== "production" &&
      warn(
        `Avoid using observed data object as vnode data: ${JSON.stringify(
          data
        )}\n` + "Always create fresh vnode data objects in each render!",
        context
      );
    return createEmptyVNode();
  }
  /*如果tag不存在也是创建一个空节点*/
  if (!tag) {
    // in case of component :is set to falsy value
    return createEmptyVNode();
  }
  // support single function children as default scoped slot
  /*默认默认作用域插槽*/
  if (Array.isArray(children) && typeof children[0] === "function") {
    data = data || {};
    data.scopedSlots = { default: children[0] };
    children.length = 0;
  }
  if (normalizationType === ALWAYS_NORMALIZE) {
    children = normalizeChildren(children);
  } else if (normalizationType === SIMPLE_NORMALIZE) {
    children = simpleNormalizeChildren(children);
  }
  let vnode, ns;
  if (typeof tag === "string") {
    let Ctor;
    /*获取tag的名字空间*/
    ns = config.getTagNamespace(tag);
    /*判断是否是保留的标签*/
    if (config.isReservedTag(tag)) {
      // platform built-in elements
      /*如果是保留的标签则创建一个相应节点*/
      vnode = new VNode(
        config.parsePlatformTagName(tag),
        data,
        children,
        undefined,
        undefined,
        context
      );
    } else if (
      isDef((Ctor = resolveAsset(context.$options, "components", tag)))
    ) {
      // component
      /*从vm实例的option的components中寻找该tag，存在则就是一个组件，创建相应节点，Ctor为组件的构造类*/
      vnode = createComponent(Ctor, data, context, children, tag);
    } else {
      // unknown or unlisted namespaced elements
      // check at runtime because it may get assigned a namespace when its
      // parent normalizes children
      /*未知的元素，在运行时检查，因为父组件可能在序列化子组件的时候分配一个名字空间*/
      vnode = new VNode(tag, data, children, undefined, undefined, context);
    }
  } else {
    // direct component options / constructor
    /*tag不是字符串的时候则是组件的构造类*/
    vnode = createComponent(tag, data, context, children);
  }
  if (isDef(vnode)) {
    /*如果有名字空间，则递归所有子节点应用该名字空间*/
    if (ns) applyNS(vnode, ns);
    return vnode;
  } else {
    /*如果vnode没有成功创建则创建空节点*/
    return createEmptyVNode();
  }
}
```

## diff 概解

### 1.当数据发生变化时，vue 是怎么更新节点的？

周所周知，Vue 通过数据绑定来修改视图，当某个数据被修改的时候，set 方法会让闭包中的 Dep 调用 notify 通知所有订阅者 Watcher，Watcher 通过 get 方法执行 vm.\_update(vm.\_render(), hydrating)。

这里看一下\_update 方法

```javascript
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    /*如果已经该组件已经挂载过了则代表进入这个步骤是个更新的过程，触发beforeUpdate钩子*/
    if (vm._isMounted) {
      callHook(vm, 'beforeUpdate')
    }
    const prevEl = vm.$el
    const prevVnode = vm._vnode
    const prevActiveInstance = activeInstance
    activeInstance = vm
    vm._vnode = vnode
    // Vue.prototype.__patch__ is injected in entry points
    // based on the rendering backend used.
    /*基于后端渲染Vue.prototype.__patch__被用来作为一个入口*/
    if (!prevVnode) {
      // initial render
      vm.$el = vm.__patch__(
        vm.$el, vnode, hydrating, false /* removeOnly */,
        vm.$options._parentElm,
        vm.$options._refElm
      )
    } else {
      // updates
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    activeInstance = prevActiveInstance
    // update __vue__ reference
    /*更新新的实例对象的__vue__*/
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
    // updated hook is called by the scheduler to ensure that children are
    // updated in a parent's updated hook.
  }复制代码
```

\_update 方法的第一个参数是一个 VNode 对象，在内部会将该 VNode 对象与之前旧的 VNode 对象进行**patch**。

要知道渲染真实 DOM 的开销是很大的，比如有时候我们修改了某个数据，如果直接渲染到真实 dom 上会引起整个 dom 树的重绘和重排，有没有可能我们只更新我们修改的那一小块 dom 而不要更新整个 dom 呢？diff 算法能够帮助我们。

我们先根据真实 DOM 生成一颗`virtual DOM`，当`virtual DOM`某个节点的数据改变后会生成一个新的`Vnode`，然后`Vnode`和`oldVnode`作对比，发现有不一样的地方就直接修改在真实的 DOM 上，然后使`oldVnode`的值为`Vnode`。

diff 的过程就是调用名为`patch`的函数，比较新旧节点，一边比较一边给**真实的 DOM**打补丁。

### 2. virtual DOM 和真实 DOM 的区别？

虚拟 dom 对应的是真实 dom， 使用`document.CreateElement` 和 `document.CreateTextNode`创建的就是真实节点。

virtual DOM 是将真实的 DOM 的数据抽取出来，以对象的形式模拟树形结构。比如 dom 是这样的：

```html
<div>
  <p>123</p>
</div>
```

对应的 virtual DOM（伪代码）：

```javascript
var Vnode = {
  tag: "div",
  children: [{ tag: "p", text: "123" }]
};
```

**（温馨提示：`VNode`和`oldVNode`都是对象，一定要记住）**

**virtual dom 很多时候都不是最优的操作，但它具有普适性，在效率、可维护性之间达平衡。**

### 3. diff 的比较方式？

在采取 diff 算法比较新旧节点的时候，比较只会在同层级进行, 不会跨层级比较。

```javascript
<div>
    <p>123</p>
</div>

<div>
    <span>456</span>
</div>
```

上面的代码会分别比较同一层的两个 div 以及第二层的 p 和 span，但是不会拿 div 和 span 作比较。在别处看到的一张很形象的图：

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6ixvc151vj30ah05j3yl.jpg)

## diff 流程图

当数据发生改变时，set 方法会让调用`Dep.notify`通知所有订阅者 Watcher，订阅者就会调用`patch`给真实的 DOM 打补丁，更新相应的视图。

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy0rlxm3j310c0syq4w.jpg)

diff 算法包括几个步骤：

- 用 JavaScript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中
- 当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异
- 把所记录的差异应用到所构建的真正的 DOM 树上，视图就更新了

## diff 算法具体分析

### 1. patch

来看看`patch`是怎么打补丁的（代码只保留核心部分）

```javascript
function patch(oldVnode, vnode) {
  // some code
  if (sameVnode(oldVnode, vnode)) {
    patchVnode(oldVnode, vnode);
  } else {
    const oEl = oldVnode.el; // 当前oldVnode对应的真实元素节点
    let parentEle = api.parentNode(oEl); // 父元素
    createEle(vnode); // 根据Vnode生成新元素
    if (parentEle !== null) {
      api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl)); // 将新元素添加进父元素
      api.removeChild(parentEle, oldVnode.el); // 移除以前的旧元素节点
      oldVnode = null;
    }
  }
  // some code
  return vnode;
}
复制代码;
```

patch 函数接收两个参数`oldVnode`和`Vnode`分别代表新的节点和之前的旧节点

- 判断两节点是否值得比较，值得比较则执行`patchVnode`

```javascript
function sameVnode(a, b) {
  return (
    a.key === b.key && // key值
    a.tag === b.tag && // 标签名
    a.isComment === b.isComment && // 是否为注释节点
    // 是否都定义了data，data包含一些具体信息，例如onclick , style
    isDef(a.data) === isDef(b.data) &&
    sameInputType(a, b) // 当标签是<input>的时候，type必须相同
  );
}
复制代码;
```

- 不值得比较则用`Vnode`替换`oldVnode`

如果两个节点都是一样的，那么就深入检查他们的子节点。如果两个节点不一样那就说明`Vnode`完全被改变了，就可以直接替换`oldVnode`。

虽然这两个节点不一样但是他们的子节点一样怎么办？别忘了，diff 可是逐层比较的，如果第一层不一样那么就不会继续深入比较第二层了。（我在想这算是一个缺点吗？相同子节点不能重复利用了...）

### 2. patchVnode

当我们确定两个节点值得比较之后我们会对两个节点指定`patchVnode`方法。那么这个方法做了什么呢？

```javascript
patchVnode (oldVnode, vnode) {
    const el = vnode.el = oldVnode.el
    let i, oldCh = oldVnode.children, ch = vnode.children
    if (oldVnode === vnode) return
    if (oldVnode.text !== null && vnode.text !== null && oldVnode.text !== vnode.text) {
        api.setTextContent(el, vnode.text)
    }else {
        updateEle(el, vnode, oldVnode)
    	if (oldCh && ch && oldCh !== ch) {
            updateChildren(el, oldCh, ch)
    	}else if (ch){
            createEle(vnode) //create el's children dom
    	}else if (oldCh){
            api.removeChildren(el)
    	}
    }
}
复制代码
```

这个函数做了以下事情：

- 找到对应的真实 dom，称为`el`
- 判断`Vnode`和`oldVnode`是否指向同一个对象，如果是，那么直接`return`
- 如果他们都有文本节点并且不相等，那么将`el`的文本节点设置为`Vnode`的文本节点。
- 如果`oldVnode`有子节点而`Vnode`没有，则删除`el`的子节点
- 如果`oldVnode`没有子节点而`Vnode`有，则将`Vnode`的子节点真实化之后添加到`el`
- 如果两者都有子节点，则执行`updateChildren`函数比较子节点，这一步很重要

其他几个点都很好理解，我们详细来讲一下 updateChildren

### 3. updateChildren

代码量很大，不方便一行一行的讲解，所以下面结合一些示例图来描述一下。

```javascript
updateChildren (parentElm, oldCh, newCh) {
    let oldStartIdx = 0, newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx
    let idxInOld
    let elmToMove
    let before
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        if (oldStartVnode == null) {   // 对于vnode.key的比较，会把oldVnode = null
            oldStartVnode = oldCh[++oldStartIdx]
        }else if (oldEndVnode == null) {
            oldEndVnode = oldCh[--oldEndIdx]
        }else if (newStartVnode == null) {
            newStartVnode = newCh[++newStartIdx]
        }else if (newEndVnode == null) {
            newEndVnode = newCh[--newEndIdx]
        }else if (sameVnode(oldStartVnode, newStartVnode)) {
            patchVnode(oldStartVnode, newStartVnode)
            oldStartVnode = oldCh[++oldStartIdx]
            newStartVnode = newCh[++newStartIdx]
        }else if (sameVnode(oldEndVnode, newEndVnode)) {
            patchVnode(oldEndVnode, newEndVnode)
            oldEndVnode = oldCh[--oldEndIdx]
            newEndVnode = newCh[--newEndIdx]
        }else if (sameVnode(oldStartVnode, newEndVnode)) {
            patchVnode(oldStartVnode, newEndVnode)
            api.insertBefore(parentElm, oldStartVnode.el, api.nextSibling(oldEndVnode.el))
            oldStartVnode = oldCh[++oldStartIdx]
            newEndVnode = newCh[--newEndIdx]
        }else if (sameVnode(oldEndVnode, newStartVnode)) {
            patchVnode(oldEndVnode, newStartVnode)
            api.insertBefore(parentElm, oldEndVnode.el, oldStartVnode.el)
            oldEndVnode = oldCh[--oldEndIdx]
            newStartVnode = newCh[++newStartIdx]
        }else {
           // 使用key时的比较
            if (oldKeyToIdx === undefined) {
                oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx) // 有key生成index表
            }
            idxInOld = oldKeyToIdx[newStartVnode.key]
            if (!idxInOld) {
                api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                newStartVnode = newCh[++newStartIdx]
            }
            else {
                elmToMove = oldCh[idxInOld]
                if (elmToMove.sel !== newStartVnode.sel) {
                    api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                }else {
                    patchVnode(elmToMove, newStartVnode)
                    oldCh[idxInOld] = null
                    api.insertBefore(parentElm, elmToMove.el, oldStartVnode.el)
                }
                newStartVnode = newCh[++newStartIdx]
            }
        }
    }
    if (oldStartIdx > oldEndIdx) {
        before = newCh[newEndIdx + 1] == null ? null : newCh[newEndIdx + 1].el
        addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx)
    }else if (newStartIdx > newEndIdx) {
        removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
    }
}
复制代码
```

先说一下这个函数做了什么

- 将`Vnode`的子节点`Vch`和`oldVnode`的子节点`oldCh`提取出来
- `oldCh`和`vCh`各有两个头尾的变量`StartIdx`和`EndIdx`，它们的 2 个变量相互比较，一共有 4 种比较方式。如果 4 种比较都没匹配，如果设置了`key`，就会用`key`进行比较，在比较的过程中，变量会往中间靠，一旦`StartIdx>EndIdx`表明`oldCh`和`vCh`至少有一个已经遍历完了，就会结束比较。

#### 图解 updateChildren

终于来到了这一部分，上面的总结相信很多人也看得一脸懵逼，下面我们好好说道说道。

粉红色的部分为 oldCh、黄色的部分为 vCh

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy0sb71dj311i0ccmyf.jpg)

我们将它们取出来并分别用 s 和 e 指针指向它们的头 child 和尾 child
![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy0tc3qkj31100h8gnl.jpg)

现在分别对`oldS、oldE、S、E`两两做`sameVnode`比较，有四种比较方式，当其中两个能匹配上那么真实 dom 中的相应节点会移到 Vnode 相应的位置，这句话有点绕，打个比方

- 如果是 oldS 和 E 匹配上了，那么真实 dom 中的第一个节点会移到最后
- 如果是 oldE 和 S 匹配上了，那么真实 dom 中的最后一个节点会移到最前，匹配上的两个指针向中间移动
- 如果四种匹配没有一对是成功的，分为两种情况
  - 如果新旧子节点都存在 key，那么会根据`oldChild`的 key 生成一张 hash 表，用`S`的 key 与 hash 表做匹配，匹配成功就判断`S`和匹配节点是否为`sameNode`，如果是，就在真实 dom 中将成功的节点移到最前面，否则，将`S`生成对应的节点插入到 dom 中对应的`oldS`位置，`oldS`和`S`指针向中间移动。
  - 如果没有 key,则直接将`S`生成新的节点插入`真实DOM`（ps：这下可以解释为什么 v-for 的时候需要设置 key 了，如果没有 key 那么就只会做四种匹配，就算指针中间有可复用的节点都不能被复用了）

再配个图（假设下图中的所有节点都是有 key 的，且 key 为自身的值）

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy0ulelpj311u0oy41x.jpg)

- 第一步

```json
oldS = a, oldE = d；
S = a, E = b;
```

`oldS`和`S`匹配，则将 dom 中的 a 节点放到第一个，已经是第一个了就不管了，此时 dom 的位置为：a b d

- 第二步

```json
oldS = b, oldE = d；
S = c, E = b;
```

`oldS`和`E`匹配，就将原本的 b 节点移动到最后，因为`E`是最后一个节点，他们位置要一致，这就是上面说的：**当其中两个能匹配上那么真实 dom 中的相应节点会移到 Vnode 相应的位置**，此时 dom 的位置为：a d b

- 第三步

```json
oldS = d, oldE = d；
S = c, E = d;

```

`oldE`和`E`匹配，位置不变此时 dom 的位置为：a d b

- 第四步

```json
oldS++;
oldE--;
oldS > oldE;

```

遍历结束，说明`oldCh`先遍历完。就将剩余的`vCh`节点根据自己的的 index 插入到真实 dom 中去，此时 dom 位置为：a c d b

一次模拟完成。

这个匹配过程的结束有两个条件：

- `oldS > oldE`表示`oldCh`先遍历完，那么就将多余的`vCh`根据 index 添加到 dom 中去（如上图）
- `S > E`表示 vCh 先遍历完，那么就在真实 dom 中将区间为`[oldS, oldE]`的多余节点删掉

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy0vmjjqj311m0ik75k.jpg)

下面再举一个例子，可以像上面那样自己试着模拟一下

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy0wmkh1j31180kmq5k.jpg)

当这些节点`sameVnode`成功后就会紧接着执行`patchVnode`了，可以看一下上面的代码

```javascript
if (sameVnode(oldStartVnode, newStartVnode)) {
  patchVnode(oldStartVnode, newStartVnode);
}
```

就这样层层递归下去，直到将 oldVnode 和 Vnode 中的所有子节点比对完。也将 dom 的所有补丁都打好啦。那么现在再回过去看 updateChildren 的代码会不会容易很多呢？

### 4. 操作 dom

这里我们只是将虚拟 DOM 映射成了真实的 DOM。那如何给这些 DOM 加入 attr、class、style 等 DOM 属性呢？

这要依赖于虚拟 DOM 的生命钩子。虚拟 DOM 提供了如下的钩子函数，分别在不同的时期会进行调用。

```javascript
const hooks = ["create", "activate", "update", "remove", "destroy"];

/*构建cbs回调函数，web平台上见/platforms/web/runtime/modules*/
for (i = 0; i < hooks.length; ++i) {
  cbs[hooks[i]] = [];
  for (j = 0; j < modules.length; ++j) {
    if (isDef(modules[j][hooks[i]])) {
      cbs[hooks[i]].push(modules[j][hooks[i]]);
    }
  }
}
```

同理，也会根据不同平台有自己不同的实现，我们这里以 Web 平台为例。Web 平台的钩子函数见[/platforms/web/runtime/modules](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fanswershuto%2FlearnVue%2Ftree%2Fmaster%2Fvue-src%2Fplatforms%2Fweb%2Fruntime%2Fmodules)。里面有对 attr、class、props、events、style 以及 transition（过渡状态）的 DOM 属性进行操作。

以 attr 为例，代码很简单。

```javascript
/* @flow */

import { isIE9 } from "core/util/env";

import { extend, isDef, isUndef } from "shared/util";

import {
  isXlink,
  xlinkNS,
  getXlinkProp,
  isBooleanAttr,
  isEnumeratedAttr,
  isFalsyAttrValue
} from "web/util/index";

/*更新attr*/
function updateAttrs(oldVnode: VNodeWithData, vnode: VNodeWithData) {
  /*如果旧的以及新的VNode节点均没有attr属性，则直接返回*/
  if (isUndef(oldVnode.data.attrs) && isUndef(vnode.data.attrs)) {
    return;
  }
  let key, cur, old;
  /*VNode节点对应的Dom实例*/
  const elm = vnode.elm;
  /*旧VNode节点的attr*/
  const oldAttrs = oldVnode.data.attrs || {};
  /*新VNode节点的attr*/
  let attrs: any = vnode.data.attrs || {};
  // clone observed objects, as the user probably wants to mutate it
  /*如果新的VNode的attr已经有__ob__（代表已经被Observe处理过了）， 进行深拷贝*/
  if (isDef(attrs.__ob__)) {
    attrs = vnode.data.attrs = extend({}, attrs);
  }

  /*遍历attr，不一致则替换*/
  for (key in attrs) {
    cur = attrs[key];
    old = oldAttrs[key];
    if (old !== cur) {
      setAttr(elm, key, cur);
    }
  }
  // #4391: in IE9, setting type can reset value for input[type=radio]
  /* istanbul ignore if */
  if (isIE9 && attrs.value !== oldAttrs.value) {
    setAttr(elm, "value", attrs.value);
  }
  for (key in oldAttrs) {
    if (isUndef(attrs[key])) {
      if (isXlink(key)) {
        elm.removeAttributeNS(xlinkNS, getXlinkProp(key));
      } else if (!isEnumeratedAttr(key)) {
        elm.removeAttribute(key);
      }
    }
  }
}

/*设置attr*/
function setAttr(el: Element, key: string, value: any) {
  if (isBooleanAttr(key)) {
    // set attribute for blank value
    // e.g. <option disabled>Select one</option>
    if (isFalsyAttrValue(value)) {
      el.removeAttribute(key);
    } else {
      el.setAttribute(key, key);
    }
  } else if (isEnumeratedAttr(key)) {
    el.setAttribute(
      key,
      isFalsyAttrValue(value) || value === "false" ? "false" : "true"
    );
  } else if (isXlink(key)) {
    if (isFalsyAttrValue(value)) {
      el.removeAttributeNS(xlinkNS, getXlinkProp(key));
    } else {
      el.setAttributeNS(xlinkNS, key, value);
    }
  } else {
    if (isFalsyAttrValue(value)) {
      el.removeAttribute(key);
    } else {
      el.setAttribute(key, value);
    }
  }
}

export default {
  create: updateAttrs,
  update: updateAttrs
};
```

attr 只需要在 create 以及 update 钩子被调用时更新 DOM 的 attr 属性即可。
