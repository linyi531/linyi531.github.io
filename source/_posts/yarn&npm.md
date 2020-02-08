---
title: yarn&npm
date: 2020-01-07 11:40:41
tags:
  - yarn
  - npm
categories: git
cover_img: https://tva1.sinaimg.cn/large/0082zybply1gbpf9ihoepj30u01601kx.jpg
feature_img: https://tva1.sinaimg.cn/large/0082zybply1gbpf9ihoepj30u01601kx.jpg
---

# yarn&npm

Facebook、Google、Exponent 和 Tilde 联合推出了一个新的 JS 包管理工具 — Yarn，是为了弥补 npm 的一些缺陷而出现的：

* npm 安装包（packages）的速度不够快，拉取的 packages 可能版本不同
* npm 允许在安装 packages 时执行代码，这就埋下了安全隐患

Yarn 没想要完全替代 npm，它只是一个新的 CLI 工具，拉取的 packages 依然来自 npm 仓库。仓库本身不会变，所以获取或者发布模块的时候和原来一样。

## yarn vs npm：特性差异

### **yarn.lock 文件**

npm 和 Yarn 都是通过 package.json 记录项目需要拉取的依赖模块，不过在使用时，往往 package.json 中模块的版本号不太会写得非常确切，通常是定个版本范围。这样你就能自行选择使用模块的大版本或者小版本，也允许 npm 拉取模块最新的修复了 bug 的版本。

#### npm

*给定一个版本号：主版本号.次版本号.补丁版本号， 以下这三种情况需要增加相应的版本号:*

- *主版本号： 当API发生改变，并与之前的版本不兼容的时候*
- *次版本号： 当增加了功能，但是向后兼容的时候*
- *补丁版本号： 当做了向后兼容的缺陷修复的时候*

```json
"dependencies": {
    "lodash": "^4.17.4"
}
```

在版本号lodash之前有个^字符。这个字符告诉npm，安装主版本等于4的任意一个版本即可。所以如果我现在运行npm进行安装，npm将安装lodash的主版本为4的最新版，可能是 lodash@4.25.5（@是npm约定用来确定包名的指定版本的）。

在理想的[语义化版本](https://link.zhihu.com/?target=http%3A//semver.org/lang/zh-CN/)世界中，新版是不会有颠覆旧版本的改变，然而现实并非如此。这就导致了**使用 npm 拉取依赖时，即使用的是相同的 package.json，在不同的设备上拉到的 packages 版本不一，这就可能为项目引入 bug。**

#### yarn

为了防止拉取到不同的版本，Yarn 有一个锁定文件 (lock file) 记录了被确切安装上的模块的版本号。每次只要新增了一个模块，Yarn 就会创建（或更新）yarn.lock 这个文件。这么做就保证了，每一次拉取同一个项目依赖时，使用的都是一样的模块版本。

npm 其实也有办法实现处处使用相同版本的 packages，但需要开发者执行 **npm shrinkwrap** 命令。这个命令将会生成一个锁定文件，在执行 npm install 的时候，该锁定文件会先被读取，和 Yarn 读取 yarn.lock 文件一个道理。npm 和 Yarn 两者的不同之处在于，Yarn 默认会生成这样的锁定文件，而 npm 要通过 shrinkwrap 命令生成 npm-shrinkwrap.json 文件，只有当这个文件存在的时候，packages 版本信息才会被记录和更新。

### **并行安装**

无论 npm 还是 Yarn 在执行包的安装时，都会执行一系列任务。

#### npm

npm 是按照队列执行每个 package，也就是说必须要等到当前 package 成功安装之后，才能继续后面的安装。

这种方法的缺点是，npm必须首先遍历所有的项目依赖关系，然后再决定如何生成扁平的node_modules目录结构。npm必须为所有使用到的模块构建一个完整的依赖关系树，这是一个耗时的操作，是[npm安装速度慢的一个很重要的原因](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fnpm%2Fnpm%2Fissues%2F8826)。

#### yarn

Yarn 是同步执行所有任务，提高了性能。

由于yarn是崭新的经过重新设计的npm客户端，它能让开发人员并行化处理所有必须的操作，并添加了一些其他改进，这使得运行速度得到了显著的提升，整个安装时间也变得更少。

#### 比较

通过拉取 [express](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/express) 依赖，我比较了 npm 和 Yarn 的效率，在没有用任何锁定文件（也就是没有缓存）的前提下，一共安装 42 个依赖：

1. npm 耗时 9 秒
2. Yarn 耗时 1.37 秒

这耗时……我没法相信自己的眼睛了，反复尝试几次，得到的结果也差不多。于是我又试着安装了有195个依赖的 [gulp](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/gulp)，这一次：

1. npm 耗时 11 秒
2. Yarn 耗时 7.81 秒

看来 npm 和 Yarn 在安装包的速度差异和要安装的包个数强相关，不过不管怎么样，Yarn 都比 npm 要快。

### **更简洁的输出**

#### npm

npm 的输出信息比较冗长。在执行 npm install <package> 的时候，命令行里会不断地打印出所有被安装上的依赖。

#### yarn

相比之下，Yarn 简洁太多：默认情况下，结合了 emoji （Windows 上 emoji 不可见）直观且直接地打印出必要的信息，也提供了一些命令供开发者查询额外的安装信息。

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g7go968d0mj30k00dp75u.jpg)

### **多注册来源处理**

所有的依赖包，不管他被不同的库间接关联引用多少次，安装这个包时，只会从一个注册来源去装，要么是 npm 要么是 bower, 防止出现混乱不一致。

### **更好的语义化**

 yarn改变了一些npm命令的名称，比如 yarn add/remove，感觉上比 npm 原本的 install/uninstall 要更清晰。

## CLI 区别

除了特性上的区别，相比于 npm 的命令，Yarn 命令有增有减还有一些更改。

### **yarn global**

npm 的全局操作命令要加上 -g 或者 --global 参数，Yarn 的全局命令则需要加上 global。和 npm 类似，项目特定的依赖，就不需要全局安装了。

当执行 yarn add、yarn bin、yarn ls 和 yarn remove 时添加 global 前缀才是有全局作用。除了 yarn add 之外，其他三个命令和 npm 的一样。

[yarn global 文档](https://link.zhihu.com/?target=https%3A//yarnpkg.com/en/docs/cli/global)

### **yarn install**

npm install 命令安装的是 package.json 中的依赖，如果开发者在 package.json 中添加了新的依赖，npm install 也一样安装。然而，yarn install 会优先安装 yarn.lock 中记录的依赖，没有这样的锁定文件时，才会去安装 package.json 中的依赖。

[yarn install 文档](https://link.zhihu.com/?target=https%3A//yarnpkg.com/en/docs/cli/install)

[npm install 文档](https://link.zhihu.com/?target=https%3A//docs.npmjs.com/cli/install)

### **yarn add [–dev]**

和 npm install 类似，yarn add 命令允许你添加并安装依赖。通过这个命令添加的依赖都会被自动加到 package.json 中，和我们在 npm 命令中使用 --save 参数一样。Yarn 的-dev 则等同于 npm 的 --save-dev。

[yarn add 文档](https://link.zhihu.com/?target=https%3A//yarnpkg.com/en/docs/cli/add)

[npm install 文档](https://link.zhihu.com/?target=https%3A//docs.npmjs.com/cli/install)

### **yarn licenses [ls|generate-disclaimer]**

在写这篇文章的时候，npm 没有等同的命令。yarn licenses ls 用于罗列出所有被安装的 package 所持有的执照情况。yarn licenses generate-disclaimer 将生成一个对所有依赖的免责声明。有些执照要求开发者一定要在项目中包含这些它们，这个命令就是为这样的场景存在的。

[yarn licenses 文档](https://link.zhihu.com/?target=https%3A//yarnpkg.com/en/docs/cli/licenses)

### **yarn why**

这条命令能帮助开发者理清安装的 package 之间的关系。拉取了各种依赖以后，有些 package 是你显式安装的，有些包则是递归依赖的。

[yarn why 文档](https://link.zhihu.com/?target=https%3A//yarnpkg.com/en/docs/cli/why)

### **yarn upgrade [package]**

这条命令将根据 package.json 将 package 升级到最新版本，并更新 yarn.lock，和 npm update 相似。

有意思的是，如果指定了 [package] 参数，Yarn 会将 package 升级到最新版本，并更新 package.json 中该 package 的版本号字段。

[yarn upgrade 文档](https://link.zhihu.com/?target=https%3A//yarnpkg.com/en/docs/cli/upgrade)

### **yarn generate-lock-entry**

这条命令将会生成一份基于 package.json 的 yarn.lock 文件，作用和 npm shrinkwrap 类似。不过由于执行 yarn add andyarn upgrade 时都会更新 yarn.lock 文件，所以要慎重执行 yarn generate-lock-entry 命令

[yarn generate-lock-entry 文档](https://link.zhihu.com/?target=https%3A//yarnpkg.com/en/docs/cli/generate-lock-entry)

[npm shrinkwrap 文档](https://link.zhihu.com/?target=https%3A//docs.npmjs.com/cli/shrinkwrap)

## yarn和npm命令对比

```text
npm install === yarn 
npm install taco --save === yarn add taco
npm uninstall taco --save === yarn remove taco
npm install taco --save-dev === yarn add taco --dev
npm update --save === yarn upgrade
```

## npm的未来：npm5.0

有了yarn的压力之后，npm做了一些类似的改进。

1. 默认新增了类似yarn.lock的 package-lock.json；
2. git 依赖支持优化：这个特性在需要安装大量内部项目（例如在没有自建源的内网开发），或需要使用某些依赖的未发布版本时很有用。在这之前可能需要使用指定 commit*id 的方式来控制版本。*
3. *文件依赖优化：在之前的版本，如果将本地目录作为依赖来安装，将会把文件目录作为副本拷贝到 node*modules 中。而在 npm5 中，将改为使用创建 symlinks 的方式来实现（使用本地 tarball 包除外），而不再执行文件拷贝。这将会提升安装速度。目前yarn还不支持。