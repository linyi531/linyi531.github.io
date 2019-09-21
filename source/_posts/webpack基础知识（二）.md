---
title: webpack基础知识（二）
date: 2018-12-05 15:12:28
tags:
  - webpack
categories: 前端工具
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hpi9legj30u0190x6u.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hpi9legj30u0190x6u.jpg
---

## 代码分离

代码分离是 webpack 中最引人注目的特性之一。此特性能够把代码分离到不同的 bundle 中，然后可以按需加载或并行加载这些文件。代码分离可以用于获取更小的 bundle，以及控制资源加载优先级，如果使用合理，会极大影响加载时间。

有三种常用的代码分离方法：

1. 入口起点：使用 entry 配置手动地分离代码。
2. 防止重复：使用 CommonsChunkPlugin 去重和分离 chunk。
3. 动态导入：通过模块的内联函数调用来分离代码。

<!-- more -->

### 入口起点(entry points)

问题：

1. 如果入口 chunks 之间包含重复的模块，那些重复模块都会被引入到各个 bundle 中。
2. 这种方法不够灵活，并且不能将核心应用程序逻辑进行动态拆分代码。

### 防止重复(prevent duplication)

CommonsChunkPlugin 插件可以将公共的依赖模块提取到已有的入口 chunk 中，或者提取到一个新生成的 chunk。
使用这个插件，可将重复的 lodash 模块去除。
需要注意的是，CommonsChunkPlugin 插件将 lodash 分离到单独的 chunk，并且将其从 main bundle 中移除，减轻了大小。

以下是由社区提供的，一些对于代码分离很有帮助的插件和 loaders：

1. ExtractTextPlugin: 用于将 CSS 从主应用程序中分离。
2. bundle-loader: 用于分离代码和延迟加载生成的 bundle。
3. promise-loader: 类似于 bundle-loader ，但是使用的是 promises。

CommonsChunkPlugin 插件还可以通过使用显式的 vendor chunks 功能，从应用程序代码中分离 vendor 模块。

### 动态导入(dynamic imports)

当涉及到动态代码拆分时，webpack 提供了两个类似的技术。对于动态导入，第一种，也是优先选择的方式是，使用符合 ECMAScript 提案 的 import() 语法。第二种，则是使用 webpack 特定的 require.ensure。

## 懒加载

懒加载或者按需加载，是一种很好的优化网页或应用的方式。这种方式实际上是先把你的代码在一些逻辑断点处分离开，然后在一些代码块中完成某些操作后，立即引用或即将引用另外一些新的代码块。这样加快了应用的初始加载速度，减轻了它的总体体积，因为某些代码块可能永远不会被加载。

## 缓存

我们使用 webpack 来打包我们的模块化后的应用程序，webpack 会生成一个可部署的 /dist 目录，然后把打包后的内容放置在此目录中。只要 /dist 目录中的内容部署到服务器上，客户端（通常是浏览器）就能够访问网站此服务器的网站及其资源。而最后一步获取资源是比较耗费时间的。因此我们使用缓存技术。
以通过命中缓存，以降低网络流量，使网站加载速度更快，然而，如果我们在部署新版本时不更改资源的文件名，浏览器可能会认为它没有被更新，就会使用它的缓存版本。由于缓存的存在，当你需要获取新的代码时，就会显得很棘手。
所以需要通过必要的配置，以确保 webpack 编译生成的文件能够被客户端缓存，而在文件内容变化后，能够请求到新的文件。

### 输出文件的文件名(Output Filenames)

通过使用 output.filename 进行文件名替换，可以确保浏览器获取到修改后的文件。[hash] 替换可以用于在文件名中包含一个构建相关(build-specific)的 hash，但是更好的方式是使用 [chunkhash] 替换，在文件名中包含一个 chunk 相关(chunk-specific)的哈希。

### 提取模板(Extracting Boilerplate)

CommonsChunkPlugin 可以用于将模块分离到单独的文件中，还能够在每次修改后的构建结果中，将 webpack 的样板(boilerplate)和 manifest 提取出来。通过指定 entry 配置中未用到的名称，此插件会自动将我们需要的内容提取到单独的包中：

```javascript
const path = require('path');
+ const webpack = require('webpack');
  const CleanWebpackPlugin = require('clean-webpack-plugin');
  const HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: './src/index.js',
    plugins: [
      new CleanWebpackPlugin(['dist']),
      new HtmlWebpackPlugin({
        title: 'Caching'
-     })
+     }),
+     new webpack.optimize.CommonsChunkPlugin({
+       name: 'manifest'
+     })
    ],
    output: {
      filename: '[name].[chunkhash].js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```

将第三方库(library)（例如 lodash 或 react）提取到单独的 vendor chunk 文件中，是比较推荐的做法，这是因为，它们很少像本地的源代码那样频繁修改。因此通过实现以上步骤，利用客户端的长效缓存机制，可以通过命中缓存来消除请求，并减少向服务器获取资源，同时还能保证客户端代码和服务器端代码版本一致。

### 模块标识符(Module Identifiers)

每个 module.id 会基于默认的解析顺序(resolve order)进行增量。也就是说，当解析顺序发生变化，ID 也会随之改变。

可以使用两个插件来解决这个问题。第一个插件是 NamedModulesPlugin，将使用模块的路径，而不是数字标识符。虽然此插件有助于在开发过程中输出结果的可读性，然而执行时间会长一些。第二个选择是使用 HashedModuleIdsPlugin，推荐用于生产环境构建。

## library

可以通过以下方式暴露 library：

1. 变量：作为一个全局变量，通过 script 标签来访问（libraryTarget:'var'）。
2. this：通过 this 对象访问（libraryTarget:'this'）。
3. window：通过 window 对象访问，在浏览器中（libraryTarget:'window'）。
4. UMD：在 AMD 或 CommonJS 的 require 之后可访问（libraryTarget:'umd'）。
   如果设置了 library 但没设置 libraryTarget，则 libraryTarget 默认为 var

## shimming

一些第三方的库(library)可能会引用一些全局依赖（例如 jQuery 中的 \$）。这些库也可能创建一些需要被导出的全局变量。这些“不符合规范的模块”就是 shimming 发挥作用的地方。

### shimming 全局变量

使用 ProvidePlugin 后，能够在通过 webpack 编译的每个模块中，通过访问一个变量来获取到 package 包。如果 webpack 知道这个变量在某个模块中被使用了，那么 webpack 将在最终 bundle 中引入我们给定的 package。
我们还可以使用 ProvidePlugin 暴露某个模块中单个导出值，只需通过一个“数组路径”进行配置（例如 [module, child, ...children?]）

src/index.js

```javascript
  function component() {
    var element = document.createElement('div');

-   element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+   element.innerHTML = join(['Hello', 'webpack'], ' ');

    return element;
  }

  document.body.appendChild(component());
```

webpack.config.js

```javascript
  const path = require('path');
  const webpack = require('webpack');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    plugins: [
      new webpack.ProvidePlugin({
-       _: 'lodash'
+       join: ['lodash', 'join']
      })
    ]
  };
```

这样，无论 join 方法在何处调用，我们都只会得到的是 lodash 中提供的 join 方法。与 tree shaking 配合，能够很好的将 lodash 库中的其他没用到的部分去除。

### 细粒度 shimming

一些传统的模块依赖的 this 指向的是 window 对象。当模块运行在 CommonJS 环境下这将会变成一个问题，也就是说此时的 this 指向的是 module.exports。
此时，可以通过使用 imports-loader 覆写 this：

```javascript
  const path = require('path');
  const webpack = require('webpack');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
+   module: {
+     rules: [
+       {
+         test: require.resolve('index.js'),
+         use: 'imports-loader?this=>window'
+       }
+     ]
+   },
    plugins: [
      new webpack.ProvidePlugin({
        join: ['lodash', 'join']
      })
    ]
  };
```

### 全局 exports

使用 exports-loader，将一个全局变量作为一个普通的模块来导出。

```javascript
  const path = require('path');
  const webpack = require('webpack');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    module: {
      rules: [
        {
          test: require.resolve('index.js'),
          use: 'imports-loader?this=>window'
-       }
+       },
+       {
+         test: require.resolve('globals.js'),
+         use: 'exports-loader?file,parse=helpers.parse'
+       }
      ]
    },
    plugins: [
      new webpack.ProvidePlugin({
        join: ['lodash', 'join']
      })
    ]
  };
```

## 渐进式网络应用程序

渐进式网络应用程序(Progressive Web Application - PWA)，是一种可以提供类似于原生应用程序(native app)体验的网络应用程序(web app)。PWA 可以用来做很多事。其中最重要的是，在离线(offline)时应用程序能够继续运行功能。这是通过使用名为 Service Workers 的网络技术来实现的。

1. 添加 Workbox
2. 注册 Service Worker

停止服务器并刷新页面。如果浏览器能够支持 Service Worker，你应该可以看到你的应用程序还在正常运行。然而，服务器已经停止了服务，此刻是 Service Worker 在提供服务。

## 构建性能

### chunks

减少编译的整体大小，以提高构建性能。尽量保持 chunks 小巧。

1. 使用 更少/更小 的库。
2. 在多页面应用程序中使用 CommonsChunksPlugin。
3. 在多页面应用程序中以 async 模式使用 CommonsChunksPlugin 。
4. 移除不使用的代码。
5. 只编译你当前正在开发部分的代码。

最小化入口 chunk
webpack 只会在文件系统中生成已经更新的 chunk 。对于某些配置选项(HMR, [name]/[chunkhash] in output.chunkFilename, [hash])来说，除了更新的 chunks 无效之外，入口 chunk 也不会生效。
应当在生成入口 chunk 时，尽量减少入口 chunk 的体积，以提高性能。下述代码块将只提取包含 runtime 的 chunk ，其他 chunk 都作为子模块:

```javascript
new CommonsChunkPlugin({
  name: "manifest",
  minChunks: Infinity
});
```

### Worker Pool

thread-loader 可以将非常消耗资源的 loaders 转存到 worker pool 中

### 持久化缓存

使用 cache-loader 启用持久化缓存。使用 package.json 中的 "postinstall" 清除缓存目录。

### Dlls

使用 DllPlugin 将更改不频繁的代码进行单独编译。这将改善引用程序的编译速度，即使它增加了构建过程的复杂性。

## 公共路径(public path)

webpack 提供一个非常有用的配置，该配置能帮助你为项目中的所有资源指定一个基础路径。它被称为公共路径(publicPath)。
webpack 提供一个全局变量供你设置，它名叫 **webpack_public_path**

```javascript
__webpack_public_path__ = process.env.ASSET_PATH;
```
