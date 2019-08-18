---
title: webpack基础知识（一）
date: 2018-11-25 12:03:08
tags:
  - webpack
categories: 前端工具
---

# 基本概念

webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。
通过声明模块所需的依赖，webpack 能够利用这些信息去构建依赖图，然后使用图生成一个优化过的，会以正确顺序执行的 bundle。

<!-- more -->

## 依赖图(dependency graph)

任何时候，一个文件依赖于另一个文件，webpack 就把此视为文件之间有 依赖关系 。这使得 webpack 可以接收非代码资源(non-code asset)（例如图像或 web 字体），并且可以把它们作为 _依赖_ 提供给你的应用程序。

webpack 从命令行或配置文件中定义的一个模块列表开始，处理你的应用程序。 从这些 入口起点 开始，webpack 递归地构建一个 依赖图 ，这个依赖图包含着应用程序所需的每个模块，然后将所有这些模块打包为少量的 bundle - 通常只有一个 - 可由浏览器加载。

## 入口

入口起点(entry point)指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。
每个依赖项随即被处理，最后输出到称之为 bundles 的文件中。

```javascript
module.exports = {
  entry: "./path/to/my/entry/file.js"
};
```

入口配置支持：

1. 单个入口语法
   注：向 entry 传入一个「文件路径(file path)数组」时将创建“多个主入口(multi-main entry)”。在你想要多个依赖文件一起注入，并且将它们的依赖导向(graph)到一个“chunk”时，传入数组的方式就很有用。
2. 对象语法
   这是应用程序中定义入口的最可扩展的方式。

## 出口(output)

output 属性告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件，默认值为 ./dist。基本上，整个应用程序结构，都会被编译到你指定的输出路径的文件夹中。你可以通过在配置中指定一个 output 字段，来配置这些处理过程：

```javascript
const path = require("path");

module.exports = {
  entry: "./path/to/my/entry/file.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "my-first-webpack.bundle.js"
  }
};
```

filename 用于输出文件的文件名。目标输出目录 path 的绝对路径。

注：如果配置创建了多个单独的 "chunk"（例如，使用多个入口起点或使用像 CommonsChunkPlugin 这样的插件），则应该使用占位符(substitutions)来确保每个文件具有唯一的名称。

设定 HtmlWebpackPlugin，虽然在 dist/ 文件夹我们已经有 index.html 这个文件，然而 HtmlWebpackPlugin 还是会默认生成 index.html 文件。这就是说，它会用新生成的 index.html 文件，把我们的原来的替换。

### 使用 source map

当 webpack 打包源代码时，可能会很难追踪到错误和警告在源代码中的原始位置。例如，如果将三个源文件（a.js, b.js 和 c.js）打包到一个 bundle（bundle.js）中，而其中一个源文件包含一个错误，那么堆栈跟踪就会简单地指向到 bundle.js。
为了更容易地追踪错误和警告，JavaScript 提供了 source map 功能，将编译后的代码映射回原始源代码。如果一个错误来自于 b.js，source map 就会明确的告诉你。

## loader

loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块，然后你就可以利用 webpack 的打包能力，对它们进行处理。

本质上，webpack loader 将所有类型的文件，转换为应用程序的依赖图（和最终的 bundle）可以直接引用的模块。

### 在 webpack 的配置中 loader 有两个目标：

1. test 属性，用于标识出应该被对应的 loader 进行转换的某个或某些文件。
2. use 属性，表示进行转换时，应该使用哪个 loader。

```javascript
const path = require("path");

const config = {
  output: {
    filename: "my-first-webpack.bundle.js"
  },
  module: {
    rules: [{ test: /\.txt$/, use: "raw-loader" }]
  }
};

module.exports = config;
```

### 在你的应用程序中，有三种使用 loader 的方式：

1. 配置（推荐）：在 webpack.config.js 文件中指定 loader。
   module.rules 允许你在 webpack 配置中指定多个 loader。 这是展示 loader 的一种简明方式，并且有助于使代码变得简洁。同时让你对各个 loader 有个全局概览
2. 内联：在每个 import 语句中显式指定 loader。
   可以在 import 语句或任何等效于 "import" 的方式中指定 loader。使用 ! 将资源中的 loader 分开。分开的每个部分都相对于当前目录解析。
3. CLI：在 shell 命令中指定它们。

### loader 特性：

1. loader 支持链式传递。能够对资源使用流水线(pipeline)。一组链式的 loader 将按照相反的顺序执行。loader 链中的第一个 loader 返回值给下一个 loader。在最后一个 loader，返回 webpack 所预期的 JavaScript。
2. loader 可以是同步的，也可以是异步的。
3. loader 运行在 Node.js 中，并且能够执行任何可能的操作。
4. loader 接收查询参数。用于对 loader 传递配置。
5. loader 也能够使用 options 对象进行配置。
6. 除了使用 package.json 常见的 main 属性，还可以将普通的 npm 模块导出为 loader，做法是在 package.json 里定义一个 loader 字段。
7. 插件(plugin)可以为 loader 带来更多特性。
8. loader 能够产生额外的任意文件。

## 插件(plugins)

loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务。
插件目的在于解决 loader 无法实现的其他事。

```javascript
const HtmlWebpackPlugin = require("html-webpack-plugin"); // 通过 npm 安装
const webpack = require("webpack"); // 用于访问内置插件

const config = {
  module: {
    rules: [{ test: /\.txt$/, use: "raw-loader" }]
  },
  plugins: [new HtmlWebpackPlugin({ template: "./src/index.html" })]
};

module.exports = config;
```

### 用法：

用 require（）引入
由于插件可以携带参数/选项，你必须在 webpack 配置中，向 plugins 属性传入 new 实例。

## 模式

提供 mode 配置选项，告知 webpack 使用相应模式的内置优化。
通过选择 development 或 production 之中的一个，来设置 mode 参数，可以启用相应模式下的 webpack 内置的优化

### 用法：

1. 只在配置中提供 mode 选项：

```javascript
module.exports = {
  mode: "production"
};
```

2. 从 CLI 参数中传递：

```javascript
webpack --mode=production
```

![](/image/webpack.png)

## 模块热替换(hot module replacement)

模块热替换(HMR - Hot Module Replacement)功能会在应用程序运行过程中替换、添加或删除模块，而无需重新加载整个页面。主要是通过以下几种方式，来显著加快开发速度：

1. 保留在完全重新加载页面时丢失的应用程序状态。
2. 只更新变更内容，以节省宝贵的开发时间。
3. 调整样式更加快速 - 几乎相当于在浏览器调试器中更改样式。

## tree shaking

通常用于描述移除 JavaScript 上下文中的未引用代码(dead-code)。它依赖于 ES2015 模块系统中的静态结构特性，例如 import 和 export。

如果所有代码都不包含副作用，我们就可以简单地将 package.json 的 "sideEffects" 属性标记为 false，来告知 webpack，它可以安全地删除未用到的 export 导出。
注：「副作用」的定义是，在导入时会执行特殊行为的代码，而不是仅仅暴露一个 export 或多个 export。

如果你的代码确实有一些副作用，那么可以改为提供一个数组：

```javascript
{
  "name": "your-project",
  "sideEffects": [
    "./src/some-side-effectful-file.js"
  ]
}
```

数组方式支持相关文件的相对路径、绝对路径和 glob 模式。它在内部使用 micromatch。

注意，任何导入的文件都会受到 tree shaking 的影响。这意味着，如果在项目中使用类似 css-loader 并导入 CSS 文件，则需要将其添加到 side effect 列表中，以免在生产模式中无意中将它删除。

从 webpack 4 开始，也可以通过 "mode" 配置选项轻松切换到压缩输出，只需设置为 "production"。
