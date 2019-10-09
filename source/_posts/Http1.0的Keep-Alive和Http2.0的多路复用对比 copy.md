---
title: Http/1.0的Keep-Alive和Http/2.0的多路复用对比
date: 2019-10-04 16:14:38
tags:
  - HTTP
  - 浏览器
categories: 浏览器
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g7s0p7lyp1j318g0u0qv8.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g7s0p7lyp1j318g0u0qv8.jpg
---

# Http/1.0 的 Keep-Alive 和 Http/2.0 的多路复用对比

## Http/1.0 的 Keep-Alive

在没有`Keep-Alive`前，我们与服务器请求数据的流程是这样：

![clipboard.png](https://tva1.sinaimg.cn/large/006y8mN6ly1g6nsz9l0hxj30ac0caq3x.jpg)

- 浏览器请求`//static.mtime.cn/a.js`-->解析域名-->HTTP 连接-->服务器处理文件-->返回数据-->浏览器解析、渲染文件
- 浏览器请求`//static.mtime.cn/b.js`-->解析域名-->HTTP 连接-->服务器处理文件-->返回数据-->浏览器解析、渲染文件
- ...
- 这样循环下去，直至全部文件下载完成。

这个流程最大的问题就是：**每次请求都会建立一次 HTTP 连接**，也就是我们常说的 3 次握手 4 次挥手，这个过程在一次请求过程中占用了相当长的时间，而且逻辑上是非必需的，因为不间断的请求数据，第一次建立连接是正常的，以后就占用这个通道，下载其他文件，这样效率多高啊！你猜对了，这就是`Keep-Alive`。

### `Keep-Alive`解决的问题

`Keep-Alive`解决的核心问题：一定时间内，同一域名多次请求数据，只建立一次 HTTP 请求，其他请求可复用每一次建立的连接通道，以达到提高请求效率的问题。这里面所说的**一定时间**是可以配置的，不管你用的是`Apache`还是`nginx`。

### `HTTP1.1`还是存在效率问题

如上面所说，在`HTTP1.1`中是默认开启了`Keep-Alive`，他解决了多次连接的问题，但是依然有两个效率上的问题：

- 第一个：**串行的文件传输**。当请求 a 文件时，b 文件只能等待，等待 a 连接到服务器、服务器处理文件、服务器返回文件，这三个步骤。我们假设这三步用时都是 1 秒，那么 a 文件用时为 3 秒，b 文件传输完成用时为 6 秒，依此类推。（注：此项计算有一个前提条件，就是浏览器和服务器是单通道传输）
- 第二个：**连接数过多**。我们假设`Apache`设置了最大并发数为 300，因为浏览器限制，浏览器发起的最大请求数为 6，也就是服务器能承载的最高并发为 50，当第 51 个人访问时，就需要等待前面某个请求处理完成。

## HTTP/2 的多路复用

HTTP/2 的多路复用就是为了解决上述的两个性能问题，我们来看一下，他是如何解决的。

- 解决第一个：在`HTTP1.1`的协议中，我们传输的`request`和`response`都是基本于文本的，这样就会引发一个问题：所有的数据必须按顺序传输，比如需要传输：`hello world`，只能从`h`到`d`一个一个的传输，不能并行传输，因为接收端并不知道这些字符的顺序，所以并行传输在`HTTP1.1`是不能实现的。

![clipboard.png](https://tva1.sinaimg.cn/large/006y8mN6ly1g6nszfhktpj30kp05lmxt.jpg)

`HTTP/2`引入`二进制数据帧`和`流`的概念，其中帧对数据进行顺序标识，如下图所示，这样浏览器收到数据之后，就可以按照序列对数据进行合并，而不会出现合并后数据错乱的情况。同样是因为有了序列，服务器就可以并行的传输数据，这就是`流`所做的事情。

![clipboard.png](https://tva1.sinaimg.cn/large/006y8mN6ly1g6nszichcoj30fm0aft9y.jpg)

- 解决第二个问题：`HTTP/2`对同一域名下所有请求都是基于`流`，也就是说同一域名不管访问多少文件，也只**建立一路连接**。同样`Apache`的最大连接数为 300，因为有了这个新特性，最大的并发就可以提升到 300，比原来提升了 6 倍！

## **多路复用和 keep alive 区别？**

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6nt1pb7xkj30ln0l940f.jpg)

1）线头阻塞（Head-of-Line Blocking），HTTP1.X 虽然可以采用 keep alive 来解决复用 TCP 的问题，但是还是无法解决请求阻塞问题。

2）所谓请求阻塞意思就是一条 TCP 的 connection 在同一时间只能允许一个请求经过，这样假如后续请求想要复用这个链接就必须等到前一个完成才行，正如上图左边表示的。

3）之所以有这个问题就是因为 HTTP1.x 需要每条请求都是可是识别，按顺序发送，否则 server 就无法判断该相应哪个具体的请求。

4）HTTP2 采用多路复用是指，在同一个域名下，开启一个 TCP 的 connection，每个请求以 stream 的方式传输，每个 stream 有唯一标识，connection 一旦建立，后续的请求都可以复用这个 connection 并且可以同时发送，server 端可以根据 stream 的唯一标识来相应对应的请求。
