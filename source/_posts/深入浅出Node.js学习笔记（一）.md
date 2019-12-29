---
title: 深入浅出Node.js学习笔记（一）
date: 2019-12-13 15:04:52
tags:
  - Node.js
categories: Node.js
cover_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gadlwkk140j31960u0tnf.jpg
feature_img: https://tva1.sinaimg.cn/large/006tNbRwgy1gadlwkk140j31960u0tnf.jpg
---

# 深入浅出Node.js学习笔记（一）

高并发、高性能

## 第一章 Node简介

* 高性能、符合事件驱动、没有历史包袱这3个主要原因，JavaScript成为了Node的实现语言。

* Node发展为一个强制不共享任何资源的 单线程、单进程系统，包含十分适宜网络的库，为构建大型分布式应用程序提供基础设施，其目 标也是成为一个构建快速、可伸缩的网络应用平台。它自身非常简单，通过通信协议来组织许多 Node，非常容易通过扩展来达成构建大型网络应用的目的。每一个Node进程都构成这个网络应 用中的一个节点，这是它名字所含意义的真谛。

* 它们都是基于事件驱动的异步架构，浏览器通过事件驱动来服务界面上的交互，Node通过事件驱 动来服务I/O

  ![image-20191206112453524](https://tva1.sinaimg.cn/large/006tNbRwgy1g9mu9fwye3j30s80g875q.jpg)

### Node的特点

#### 异步I/O

* 在Node中，绝大多数的操作都以异步的方式进行调用。

* 在Node中，我们可 以从语言层面很自然地进行并行I/O操作。每个调用之间无须等待之前的I/O调用结束。

#### 事件与回调函数

* 事件的编程方式具有轻量级、松耦合、只关注事务点等优势，但是在多个异步任务的场景下， 事件与事件之间各自独立，如何协作是一个问题。

* 回调函数无处不在。回调函数是最好的接受异步调用返回数据的方式

#### 单线程

Node保持了JavaScript在浏览器中单线程的特点。

* 在Node中，JavaScript与其余线程是无 法共享任何状态的。（单线程的最大好处是不用像多线程编程那样处处在意状态的同步问题，这里 没有死锁的存在，也没有线程上下文交换所带来的性能上的开销。）
* 单线程的弱点：
  * 无法利用多核CPU
  * 错误会引起整个应用退出，应用的健壮性值得考验
  * 大量计算占用CPU导致无法继续调用异步I/O

* 在Node中，长时间的CPU占用也会导致后续的异步I/O发不出调用，已完成的异步I/O的 回调函数也会得不到及时执行。解决单线程中大计算量的问题——child_process
* 子进程的出现，意味着Node可以从容地应对单线程在健壮性和无法利用多核CPU方面的问 题。通过将计算分发到各个子进程，可以将大量计算分解掉，然后再通过进程之间的事件消息来 传递结果，这可以很好地保持应用模型的简单和低依赖。通过Master-Worker的管理方式，也可以 很好地管理各个工作进程，以达到更高的健壮性。

#### 跨平台

![image-20191206142014972](https://tva1.sinaimg.cn/large/006tNbRwgy1g9mzbqn9c1j30l80fct9g.jpg)

它在操作系统与Node上层模块 系统之间构建了一层平台层架构，即libuv。（libuv已经成为许多系统实现跨平台的基础组件）

### Node 的应用场景

#### I/O 密集型

Node擅长I/O密集型的应用场景。Node面向网络且擅长并行I/O，能够有效 地组织起更多的硬件资源，从而提供更多好的服务。

I/O密集的优势主要在于Node利用事件循环的处理能力，而不是启动每一个线程为每一个请 求服务，资源占用极少。

#### CPU密集型

CPU密集型应用给Node 带来的挑战主要是:由于JavaScript单线程的原因，如果有长时间运行的计算(比如大循环)，将 会导致CPU时间片不能释放，使得后续I/O无法发起。但是适当调整和分解大型运算任务为多个 小任务，使得运算能够适时释放，不阻塞I/O调用的发起，这样既可同时享受到并行异步I/O的好 处，又能充分利用CPU。

* Node可以通过编写C/C++扩展的方式更高效地利用CPU，将一些V8不能做到性能极致的地方通过C/C++来实现。由上面的测试结果可以看到，通过C/C++扩展的方式实现斐波那契数列计算，速度比Java还快。
* 如果单线程的Node不能满足需求，甚至用了C/C++扩展后还觉得不够，那么通过子进程的方式，将一部分Node进程当做常驻服务进程用于计算，然后利用进程间的消息来传递结果，将计算与I/O分离，这样还能充分利用多CPU。

**CPU密集不可怕，如何合理调度是诀窍。**

#### 分布式应用

Node高效利用并行I/O的过程，也是高效使用数 据库的过程

## 第二章 模块机制

### CommonJS

#### JavaScript缺陷

* 没有模块系统
* 标准库较少
* 没有标准接口
* 缺乏包管理系统

CommonJS规范的提出，主要是为了弥补当前JavaScript没有标准的缺陷

![image-20191206205027614](https://tva1.sinaimg.cn/large/006tNbRwgy1g9nalsobqcj310m0awq4f.jpg)

#### CommonJS 的模块规范

* 模块引用

```javascript
var math = require('math');
```

* 模块定义

上下文提供了 exports对象用于导出当前模块的方法或者变量，并且它是唯一导出的出口。

在模块中，还存在 一个module对象，它代表模块自身，而exports是module的属性。

在Node中，一个文件就是一个 模块，将方法挂载在exports对象上作为属性即可定义导出的方式。

```javascript
// math.js
exports.add = function () {
	var sum = 0, 
      i = 0,
			args = arguments,
			l = args.length; 
  while (i < l) {
		sum += args[i++]; 
  }
	return sum; 
};
```

* 模块标识

模块标识其实就是传递给require()方法的参数，它必须是符合小驼峰命名的字符串，或者 以.、..开头的相对路径，或者绝对路径。它可以没有文件名后缀.js。

它的意义在于将类聚的方法和变量等限定在私有的 作用域中，同时支持引入和导出功能以顺畅地连接上下游依赖。

![image-20191209111442609](https://tva1.sinaimg.cn/large/006tNbRwgy1g9qatnbzkaj30mi0byt98.jpg)

### Node 的模块实现

在Node中引入模块，需要经历如下3个步骤。

* 路径分析
* 文件定位
* 编译执行

在Node中，模块分为两类:一类是Node提供的模块，称为核心模块;另一类是用户编写的模块，称为文件模块。

* 核心模块部分在Node源代码的编译过程中，编译进了二进制执行文件。在Node进程启动 时，部分核心模块就被直接加载进内存中，所以这部分核心模块引入时，文件定位和编 译执行这两个步骤可以省略掉，并且在路径分析中优先判断，所以它的加载速度是最 快的。
* 文件模块则是在运行时动态加载，需要完整的路径分析、文件定位、编译执行过程，速 度比核心模块慢。

#### 优先从缓存加载

Node对引入过的模块都会进行缓存，以减少二次引入时的开销。Node缓存的事编译和执行之后的对象。

不论是核心模块还是文件模块，require()方法对相同模块的二次加载都一律采用缓存优先的 方式，这是第一优先级的。不同之处在于核心模块的缓存检查先于文件模块的缓存检查。

#### 路径分析和文件定位

##### 模块路径

模块路径是Node在定位文件模块的具体文件时制定的查找策略，具体表现为一个路径组成的数组。

模块路径的生成规则：

* 当前文件目录下的node_modules目录。
* 父目录下的node_modules目录。
* 父目录的父目录下的node_modules目录。
* 沿路径向上逐级递归，直到根目录下的node_modules目录。

##### 模块标识符分析

模块标识符在Node中主要分为以下几类。

* 核心模块，如http、fs、path等。
* .或..开始的相对路径文件模块。
* 以/开始的绝对路径文件模块。
* 非路径形式的文件模块，如自定义的connect模块。

1. 核心模块

   核心模块的优先级仅次于缓存加载，它在Node的源代码编译过程中已经编译为二进制代码， 其加载过程最快。

   （如果加载一个与核心模块标识符相同的自定义模块，不会成功。如果自己编写了一个http用户模块，想要加载成功，必须选择不同的标识符或换用路径方式）

2. 路径形式的文件模块

   在分析文件模块时，require()方法会将路径转为真实路径，并以真实路径作为索引，将编译执行后的结果存放到缓存中，以使二 次加载时更快。

   （文件模块指明了确切的文件位置，在查找中会节约时间，加载速度慢于核心模块）

3. 自定义模块

   它是一种特殊的文件模块，可能是一个文件或者包的形式。这类模块的查找是最费时的，也是所有方式中最慢的一种。

   在加载的过程中，Node 会逐个尝试模块路径中的路径，直到找到目标文件为止。可以看出，当前文件的路径越深，模块查找耗时会越多，这是自定义模块的加载速度是最慢的原因。

##### 文件定位

* 文件扩展名分析：

  require()在分析标识符的过程中，会出现标识符中不包含文件扩展名的情况。CommonJS模块规范也允许在标识符中不包含文件扩展名，这种情况下，**Node会按.js、.json、.node的次序补 足扩展名，依次尝试。**

  **在尝试的过程中，需要调用fs模块同步阻塞式地判断文件是否存在。**因为Node是单线程的， 所以这里是一个会引起性能问题的地方。

  （如果是.node和.json文件，在传递给require() 的标识符中带上扩展名，会加快一点速度。同步配合缓存，可以大幅度缓解Node 单线程中阻塞式调用的缺陷。）

* 目录分析和包

  在分析标识符的过程中，require()通过分析文件扩展名之后，可能**没有查找到对应文件，但却得到一个目录，这在引入自定义模块和逐个模块路径进行查找时经常会出现，此时Node会将目录当做一个包来处理。**

  * Node在当前目录下 查找package.json(CommonJS包规范定义的包描述文件)，通过JSON.parse()解析出包描述对象， 从中取出main属性指定的文件名进行定位。如果文件名缺少扩展名，将会进入扩展名分析的步骤。
  * 而如果main属性指定的文件名错误，或者压根没有package.json文件，Node会将index当做默 认文件名，然后依次查找index.js、index.json、index.node。
  * 如果在目录分析的过程中没有定位成功任何文件，则自定义模块进入下一个模块路径进行查 找。
  * 如果模块路径数组都被遍历完毕，依然没有查找到目标文件，则会抛出查找失败的异常。

#### 模块编译

在Node中，每个文件模块都是一个对象

定位到具体的文件后，Node会新建一个模块对 象，然后根据路径载入并编译。对于不同的文件扩展名，其载入方法有所不同

* .js文件。通过fs模块同步读取文件后编译执行。
* .node文件。这是用C/C++编写的扩展文件，通过dlopen()方法加载最后编译生成的文件。
* .json文件。通过fs模块同步读取文件后，用JSON.parse()解析返回结果。
* 其余扩展名文件。它们都被当做.js文件载入。

每一个编译成功的模块都会将其文件路径作为索引缓存在Module._cache对象上，以提高二 次引入的性能。

在确定文件的扩展名之后，Node将调用具体的编译方式来将文件执行后返回给调用者。

##### JavaScript模块的编译

在编译的过程中，Node对获取的JavaScript文件内容进行了头尾包装。

* 在头部添加 了(function (exports, require, module, __filename, __dirname) {\n，在尾部添加了\n});。

  ```javascript
  (function (exports, require, module, __filename, __dirname) { 
    var math = require('math');
    exports.area = function (radius) {
      return Math.PI * radius * radius; 
    };
   });
  ```

  这样每个模块文件之间都进行了作用域隔离。

* 包装之后的代码会通过vm原生模块的runInThisContext()方法执行(类似eval，只是具有明确上下文，不污染全局)，返回一个具体的 function对象。

* 将当前模块对象的exports属性、require()方法、module(模块对象自身)， 以及在文件定位中得到的完整文件路径和文件目录作为参数传递给这个function()执行。

exports对象是通过形参的方式传入的，直接赋值形参会改变形参的引用，但并不能改变作用域外的值。(**如果要达到require引入一个类的效果，请赋值给module.exports对象。**)

##### C/C++模块的编译

Node调用process.dlopen()方法进行加载和执行。在Node的架构下，dlopen()方法在Windows 和*nix平台下分别有不同的实现，通过libuv兼容层进行了封装。

它是编写C/C++模块之后编译生成的，所以这 里只有加载和执行的过程。在执行的过程中，模块的exports对象与.node模块产生联系，然后返 回给调用者。

* 优势：执行效率
* 劣势：编写门槛高

##### JSON文件的编译

Node利用fs模块同步读取JSON文件的内容之 后，调用JSON.parse()方法得到对象，然后将它赋给模块对象的exports，以供外部调用。

（定义了一个JSON文件作为配置，那就 不必调用fs模块去异步读取和解析，直接调用require()引入即可）

### 核心模块

核心模块C/C++文件存放在Node项目的src目录下， JavaScript文件存放在lib目录下。

#### JavaScript 核心模块的编译过程

##### 转存为C/C++代码

Node采用了V8附带的js2c.py工具，将所有内置的JavaScript代码(src/node.js和lib/*.js)转换 成C++里的数组，生成node_natives.h头文件

JavaScript代码以字符串的形式存储在node命名空间中，是不可直接执行的。

在启动Node进程时，JavaScript代码直接加载进内存中。在加载的过程中，JavaScript核心模块经 历标识符分析后直接定位到内存中，比普通的文件模块从磁盘中一处一处查找要快很多。

##### 编译JavaScript核心模块

lib目录下的所有模块文件也没有定义require、module、exports这些变量。在引入JavaScript 核心模块的过程中，也经历了头尾包装的过程，然后才执行和导出了exports对象。与文件模块有区别的地方在于:**获取源代码的方式(核心模块是从内存中加载的)以及缓存执行结果的位置。**

源文件通过process.binding('natives')取出， 编译成功的模块缓存到NativeModule._cache对象上，文件模块则缓存到Module._cache对象上

#### C/C++核心模块的编译过程

C++模块主内完成核心，JavaScript 主外实现封装的模式是Node能够提高性能的常见方式。

由纯C/C++编写的部分统一称为**内建模块**，因为它们通常不被用户直接调 用。

##### 内建模块

Node提供了get_builtin_module()方法从node_module_list 数组中取出这些模块

###### 内建模块的优势在于：

* 它们本身由C/C++编写，性能上优于脚本语言
* 在进行文件编译时，它们被编译进二进制文件。一旦Node开始执行，它们被直接加载进内存中，无须再 次做标识符定位、文件定位、编译等过程，直接就可执行。

在Node的所有模块类型中，存在着如图2-4所示的一种依赖层级关系，即文件模块可能会依 赖核心模块，核心模块可能会依赖内建模块。

![image-20191210195732891](https://tva1.sinaimg.cn/large/006tNbRwgy1g9rvjz8pitj30gc0gsmxx.jpg)

###### 加载内建模块：

* 在加载内建模块时，先创建一个exports空对象
* 然后调用get_builtin_module()方法取 出内建模块对象，通过执行register_func()填充exports对象
* 最后将exports对象按模块名缓存，并返回给调用方完成导出。

#### 核心模块的引入流程

![image-20191210201250065](https://tva1.sinaimg.cn/large/006tNbRwgy1g9rvzu27c2j30m40qa75t.jpg)

#### 编写核心模块

* 编写头文件
* 编写C/C++文件

### C/C++扩展模块

C/C++扩展模块属于文件模块中的一类。

为了实现跨平台，dlopen()方法在内部实现时区 分了平台，分别用的是加载.so和.dll的方式。（一个平台下的.node文件在另一个平台下是无法加载执行的，必须重新用各 自平台下的编译器编译为正确的.node文件。）

![image-20191210202239589](https://tva1.sinaimg.cn/large/006tNbRwgy1g9rwa1zemkj30p60wi0vf.jpg)

require()在引入.node文件的过程中：

* 调用uv_dlopen()方法去打开动态链接库
* 调用uv_dlsym()方法找到动态链接库中通过NODE_MODULE宏定义的方法地址

这 两个过程都是通过libuv库进行封装的:在*nix平台下实际上调用的是dlfcn.h头文件中定义的 dlopen()和dlsym()两个方法;在Windows平台则是通过LoadLibraryExW()和GetProcAddress()这两 个方法实现的，它们分别加载.so和.dll文件(实际为.node文件)。

![image-20191210205752782](https://tva1.sinaimg.cn/large/006tNbRwgy1g9rxappw9mj30pq0lkdhm.jpg)

### 模块调用栈

![image-20191211174120316](https://tva1.sinaimg.cn/large/006tNbRwgy1g9sxcy1mw1j30qm0hy3zw.jpg)

* C/C++内建模块属于最底层的模块，它属于核心模块，主要提供API给JavaScript核心模块和 第三方JavaScript文件模块调用。
* JavaScript核心模块主要扮演的职责有两类:
  * 一类是作为C/C++内建模块的封装层和桥接层， 供文件模块调用;
  * 一类是纯粹的功能模块
* 文件模块通常由第三方编写，包括普通JavaScript模块和C/C++扩展模块，主要调用方向为普通JavaScript模块调用扩展模块。

### 包与NPM

![image-20191211184259742](https://tva1.sinaimg.cn/large/006tNbRwgy1g9sz0nu3v7j30uc0laq4c.jpg)

由包结构和包描述文件两个部分组成，前者用于组织包中的各种文件，后者则用于描述包的相关信息，以供外部读取分析。

#### 包结构

包实际上是一个存档文件，即一个目录直接打包为.zip或tar.gz格式的文件，安装后解压还原为目录。

完全符合CommonJS规范的包目录应该包含如下这些文件：

* package.json:包描述文件。
* bin:用于存放可执行二进制文件的目录。 
* lib:用于存放JavaScript代码的目录。
* doc:用于存放文档的目录。
* test:用于存放单元测试用例的代码。

#### 包描述文件与NPM

包描述文件用于表达非代码相关的信息，它是一个JSON格式的文件——package.json，位于 包的根目录下，是包的重要组成部分。

##### 必需字段：

* name。包名。规范定义它需要由小写的字母和数字组成，可以包含.、_和-，但不允许出现空格。包名必须是唯一的，以免对外公布时产生重名冲突的误解。除此之外，NPM还建议不要在包名中附带上node或js来重复标识它是JavaScript或Node模块。
* description。包简介。
* version。版本号。一个语义化的版本号，该版本号十分重要，常常用于一些版本控制的场合。
* keywords。关键词数组，NPM中主要用来做分类搜索。一个好的关键词数组有利于用户快速找到你编写的包。
* maintainers。包维护者列表。每个维护者由name、email和web这3个属性组成。NPM通过该属性进行权限认证。
* contributors。贡献者列表。列表中的第一个贡献应当是包的作者本人。它的格式与维护者列表相同。
* bugs。一个可以反馈bug的网页地址或邮件地址。
* licenses。当前包所使用的许可证列表，表示这个包可以在哪些许可证下使用。
* repositories。托管源代码的位置列表，表明可以通过哪些方式和地址访问包的源代码。
* dependencies。使用当前包所需要依赖的包列表。这个属性十分重要，NPM会通过这个属性帮助自动加载依赖的包。

##### 可选字段：

* homepage。当前包的网站地址。
* os。操作系统支持列表。这些操作系统的取值包括aix、freebsd、linux、macos、solaris、vxworks、windows。如果设置了列表为空，则不对操作系统做任何假设。
* cpu。CPU架构的支持列表，有效的架构名称有arm、mips、ppc、sparc、x86和x86_64。同os一样，如果列表为空，则不对CPU架构做任何假设。
* engine。支持的JavaScript引擎列表，有效的引擎取值包括ejs、flusspferd、gpsee、jsc、spidermonkey、narwhal、node和v8。
* builtin。标志当前包是否是内建在底层系统的标准组件。 
* directories。包目录说明。
* implements。实现规范的列表。标志当前包实现了CommonJS的哪些规范。
* scripts。脚本说明对象。它主要被包管理器用来安装、编译、测试和卸载包。

**在包描述文件的规范中，NPM实际需要的字段主要有name、version、description、keywords、 repositories、author、bin、main、scripts、engines、dependencies、devDependencies。**

* author。包作者。 
* bin。一些包作者希望包可以作为命令行工具使用。配置好bin字段后，通过npm install package_name -g命令可以将脚本添加到执行路径中，之后可以在命令行中直接执行。前面的node-gyp即是这样安装的。通过-g命令安装的模块包称为全局模式。
* main。模块引入方法require()在引入包时，会优先检查这个字段，并将其作为包中其余 模块的入口。如果不存在这个字段，require()方法会查找包目录下的index.js、index.node、index.json文件作为默认入口。
* devDependencies。一些模块只在开发时需要依赖。配置这个属性，可以提示包的后续开发者安装依赖包。

### 前后端共用模块

#### AMD 规范

AMD规范是CommonJS模块规范的一个延伸

```javascript
define(id?, dependencies?, factory);
```

它的模块id和依赖是可选的，与Node模块相似的地方在于factory的内容就是实际代码的内容。

```javascript
define(function() {
  var exports = {}; 
  exports.sayHello = function() {
    alert('Hello from module: ' + module.id); };
  return exports; 
});
```

不同之处在于AMD模块需要用define来明确定义一个模块，而在Node实现中是隐式包装的， 它们的目的是进行作用域隔离，仅在需要的时候被引入，避免掉过去那种通过全局变量或者全局 命名空间的方式，以免变量污染和不小心被修改。另一个区别则是内容需要通过返回的方式实现 导出。

#### CMD 规范

与AMD规范的主要区别在于定义模块和依赖引入的部分。

AMD需要在声明模块的时候指定所有的依赖，通过形参传递依赖到模块内容中。

在依赖部分，CMD支持动态引入。

## 第三章 异步I/O

### 为什么要异步I/O

#### 用户体验

* 前端通过异步可以消除掉UI阻塞的现象。但是前端获取资源的速度也取决于后端的响应速度。采用异步方式，第一个资源的获取并不会阻塞第二个资源。

* 随着网站或应用不断膨胀，数据将会分布到多台服务器上，分布式将会是常态。分布也意味着M与N的值（M/N分别为两个请求消耗的时间）会线性增长，这也会放大异步和同步在性能方面的差异。

  ![image-20191212150038480](https://tva1.sinaimg.cn/large/006tNbRwgy1g9ty7mwa38j317s0ce400.jpg)

只有后端能够快速响应资源，才能让前端的体验变好。

#### 资源分配

利用单线程，远离多线程死锁、状态同步等问题;利用异 步I/O，让单线程远离阻塞，以更好地使用CPU。

Node提供了类似前端浏览器中Web Workers的子 进程，该子进程可以通过工作进程高效地利用CPU和I/O

![image-20191212171933363](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u2861hzfj30lu0lit9x.jpg)

### 异步I/O实现现状

#### 异步I/O与非阻塞I/O

异步/同步和阻塞/非阻塞实际上是两回事

操作系统内核对于I/O只有两种方式:阻塞与非阻塞。

##### 阻塞：

在调用阻塞I/O时，应用程序需要等待 I/O完成才返回结果

![image-20191212172518477](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u2e5a4fnj30j80k6t9l.jpg)

* 特点：调用之后一定要等到系统内核层面完成所有操作后，调用才结束。

* 阻塞I/O造成CPU等待I/O，浪费等待时间，CPU的处理能力不能得到充分利用

##### 非阻塞：

非阻塞I/O跟阻塞I/O的差别为调用之后会立即返回

![image-20191212172706537](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u2g0l7cmj30jm0kkq3q.jpg)

* 非阻塞I/O返回之后，CPU的时间片可以用来处理其他事务，此时的性能提升是明显的。

* 问题：由于完整的I/O并没有完成，立即返回的并不是业务层期望的 数据，而仅仅是当前调用的状态。为了获取完整的数据，应用程序需要重复调用I/O操作来确认 是否完成。这种重复调用判断操作是否完成的技术叫做轮询。

* 轮询技术：

  * read。它是最原始、性能最低的一种，通过重复调用来检查I/O的状态来完成完整数据的读取。在得到最终数据前，CPU一直耗用在等待上。

    ![image-20191212173038848](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u2jpnno5j30ku0l8gmy.jpg)

  * select。它是在read的基础上改进的一种方案，通过对文件描述符上的事件状态来进行判断。

    ![image-20191212173129915](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u2klirw7j30k20l6wfu.jpg)

    select轮询具有一个较弱的限制，那就是由于它采用一个1024长度的数组来存储状态， 所以它最多可以同时检查1024个文件描述符。

  * poll。该方案较select有所改进，采用链表的方式避免数组长度的限制，其次它能避免不需要的检查。但是当文件描述符较多的时候，它的性能还是十分低下的。

    ![image-20191212173248351](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u2ly78zij30j40km75l.jpg)

  * epoll。该方案是Linux下效率最高的I/O事件通知机制，在进入轮询的时候如果没有检查到 I/O事件，将会进行休眠，直到事件发生将它唤醒。它是真实利用了事件通知、执行回调 的方式，而不是遍历查询，所以不会浪费CPU，执行效率较高

    ![image-20191212173403901](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u2na09ynj30jw0leta2.jpg)

  * kqueue。该方案的实现方式与epoll类似，不过它仅在FreeBSD系统下存在。

#### 现实的异步I/O

通过让部分线程进行阻塞I/O或者非阻塞I/O加 轮询技术来完成数据获取，让一个线程进行计算处理，通过线程之间的通信将I/O得到的数据进 行传递，这就轻松实现了异步I/O

![image-20191212173804605](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u2rft4wij30pq0iwgmo.jpg)

* ibeio 实质上依然是采用线程池与阻塞I/O模拟异步I/O
* IOCP：调用异步方法，等待I/O完成之后的通知，执行回调，用户无须考虑轮询。但是它的 内部其实仍然是线程池原理，不同之处在于这些线程池由系统内核接手管理。

![image-20191212174138825](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u339w1rrj30g80ekq3m.jpg)

Node是单线程的，这里的单线程仅仅只是 JavaScript执行在单线程中罢了。在Node中，无论是*nix还是Windows平台，内部完成I/O任务的 另有线程池。

### Node 的异步 I/O

事件循环、观察者、请求对象、I/O线程池这四者共同构成了Node异步I/O模型的基本要素。

Windows下主要通过IOCP来向系统内核发送I/O调用和从内核获取已完成的I/O操作，配以事 件循环，以此完成异步I/O的过程。在Linux下通过epoll实现这个过程，FreeBSD下通过kqueue实 现，Solaris下通过Event ports实现。不同的是线程池在Windows下由内核(IOCP)直接提供，*nix 系列下由libuv自行实现。

#### 事件循环

**Node自身的执行模型——事件循环**

每执行一次循环体的过程我 们称为Tick。每个Tick的过程就是查看是否有事件待处理，如果有，就取出事件及其相关的回调 函数。如果存在关联的回调函数，就执行它们。然后进入下个循环，如果不再有事件处理，就退出进程。

![image-20191212174548717](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u2zihuruj30km0o6dhb.jpg)

#### 观察者

每个事件循环中有一个或者多个观察者，而判断是否有事件要处理的过程就是向这些观察者询问 是否有要处理的事件。

* 观察者将事件进行分类。在Node中，事件主要来源于网络请求、文件I/O等，这些事件对应的 观察者有文件I/O观察者、网络I/O观察者等。
* 在Windows下，这个循环基于IOCP创建，而在*nix下则基于多线程创建。

#### 请求对象

从JavaScript发起调用到内核执行完I/O操作的 过渡过程中，存在一种中间产物，它叫做请求对象。

```javascript
fs.open = function(path, flags, mode, callback) { 
  // ...
  binding.open(pathModule._makeLong(path), stringToFlags(flags),mode,callback); 
};
```

![image-20191212175431157](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u38jib15j30ss0q40uj.jpg)

* 从JavaScript调用Node的核心模块，核心模块调用C++内建模块，内建模块通过libuv进行系统调用

* 这里libuv作为封装层，有两个平台的实现，实质上是调 用了uv_fs_open()方法。在uv_fs_open()的调用过程中，我们创建了一个FSReqWrap请求对象

* 回调函数则 被设置在这个对象的oncomplete_sym属性上

  ```javascript
  req_wrap->object_->Set(oncomplete_sym, callback);
  ```

* 对象包装完毕后，在Windows下，则调用QueueUserWorkItem()方法将这个FSReqWrap对象推入线程池中等待执行

  * QueueUserWorkItem()方法接受3个参数:第一个参数是将要执行的方法的引用，这里引用的是uv_fs_thread_proc，第二个参数是uv_fs_thread_proc方法运行时所需要的参数;第三个参数是 执行的标志。

* 当线程池中有可用线程时，我们会调用uv_fs_thread_proc()方法。uv_fs_thread_ proc()方法会根据传入参数的类型调用相应的底层函数

* 至此，JavaScript调用立即返回，由JavaScript层面发起的异步调用的第一阶段就此结束。

JavaScript线程可以继续执行当前任务的后续操作。当前的I/O操作在线程池中等待执行，不管它 是否阻塞I/O，都不会影响到JavaScript线程的后续执行，如此就达到了异步的目的。

**请求对象是异步I/O过程中的重要中间产物，所有的状态都保存在这个对象中**，包括送入线程池等待执行以及I/O操作完毕后的回调处理

#### 执行回调

* 线程池中的I/O操作调用完毕之后，会将获取的结果储存在req->result属性上，然后调用 PostQueuedCompletionStatus()通知IOCP，告知当前对象操作已经完成

```javascript
PostQueuedCompletionStatus((loop)->iocp, 0, 0, &((req)->overlapped))
```

* PostQueuedCompletionStatus()方法的作用是向IOCP提交执行状态，并将线程归还线程池。通过PostQueuedCompletionStatus()方法提交的状态，可以通过GetQueuedCompletionStatus()提取。

* 在每次Tick的执行中，它会调用 IOCP相关的GetQueuedCompletionStatus()方法检查线程池中是否有执行完的请求，如果存在，会将请求对象加入到I/O观察者的队列中，然后将其当做事件处理。
* I/O观察者回调函数的行为就是取出请求对象的result属性作为参数，取出oncomplete_sym属性作为方法，然后调用执行，以此达到调用JavaScript中传入的回调函数的目的

 ![image-20191212192431768](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u5u87afbj30uu0u0q6o.jpg)

### 非I/O的异步API

I/O无关的异步API:setTimeout()、setInterval()、 setImmediate()和process.nextTick()

#### 定时器

setTimeout()和setInterval()

它们的实现原理与异步I/O比较类似，只是不需要I/O线程池的参与。

调用setTimeout()或者 setInterval()创建的定时器会被插入到定时器观察者内部的一个红黑树中。每次Tick执行时，会 从该红黑树中迭代取出定时器对象，检查是否超过定时时间，如果超过，就形成一个事件，它的 回调函数将立即执行。

![image-20191212194435097](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u6f2b0goj30u60nu76w.jpg)

#### process.nextTick()

```javascript
process.nextTick = function(callback) { 
  // on the way out, don't bother.
  // it won't get fired anyway
  if (process._exiting) return;
  
  if (tickDepth >= process.maxTickDepth) 
    maxTickWarn();
  
  var tock = { callback: callback };
  if (process.domain) tock.domain = process.domain;
  nextTickQueue.push(tock);
  if (nextTickQueue.length) {
    process._needTickCallback(); 
  }
};
```

每次调用process.nextTick()方法，只会将回调函数放入队列中，在下一轮Tick时取出执行。 定时器中采用红黑树的操作时间复杂度为O(lg(n))，nextTick()的时间复杂度为O(1)。相较之下，process.nextTick()更高效。

#### setImmediate()

```javascript
process.nextTick(function () { 
  console.log('nextTick延迟执行');
});
setImmediate(function () {
  console.log('setImmediate延迟执行'); 
});
console.log('正常执行'); 
```

其执行结果如下:

```json
正常执行 
nextTick延迟执行 
setImmediate延迟执行
```

* process.nextTick()中的回调函数执行的优先级要高于setImmediate()。

* 原因在于事件循环对观察者的检查是有先后顺序的，process.nextTick()属于idle观察者， setImmediate()属于check观察者。在每一个轮循环检查中，idle观察者先于I/O观察者，I/O观察者先于check观察者。
* process.nextTick()的回调函数保存在一个数组中，setImmediate()的结果 则是保存在链表中
* 在行为上，process.nextTick()在每轮循环中会将数组中的回调函数全部执行完，而setImmediate()在每轮循环中执行链表中的一个回调函数。
* 之所以这样设计，是为了保证每轮循环能够较快地执行结束，防止CPU占用过多而阻塞后续I/O 调用的情况。

### 事件驱动与高性能服务器

事件驱动的实质：

通过主循环加事件触发的方式来运行程序。

![image-20191212201217702](https://tva1.sinaimg.cn/large/006tNbRwgy1g9u77vrg44j30vo0nm77g.jpg)

#### 几种经典的服务器模型，对比优缺点

* 同步式。对于同步式的服务，一次只能处理一个请求，并且其余请求都处于等待状态。 
* 每进程/每请求。为每个请求启动一个进程，这样可以处理多个请求，但是它不具备扩展性，因为系统资源只有那么多。
* 每线程/每请求。为每个请求启动一个线程来处理。尽管线程比进程要轻量，但是由于每个线程都占用一定内存，当大并发请求到来时，内存将会很快用光，导致服务器缓慢。

#### 高性能：

Node通过事件驱动的方式处理请求，无须为 每一个请求创建额外的对应线程，可以省掉创建线程和销毁线程的开销，同时操作系统在调度任 务时因为线程较少，上下文切换的代价很低。这使得服务器能够有条不紊地处理请求，即使在大量连接的情况下，也不受线程上下文切换开销的影响，这是Node高性能的一个原因。