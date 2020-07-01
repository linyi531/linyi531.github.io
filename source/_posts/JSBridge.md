---
title: JSBridge
date: 2020-05-22 12:35:51
tags:
  - JavaScript
categories: JavaScript
cover_img: https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbqfn6h7uj31900u0npf.jpg
feature_img: https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbqfn6h7uj31900u0npf.jpg
---

# JsBridge

[JS bridge原理](https://www.cnblogs.com/dailc/p/5931324.html)

## JSBridge 的用途

JSBridge 简单来讲，主要是 **给 JavaScript 提供调用 Native 功能的接口**，让混合开发中的『前端部分』可以方便地使用地址位置、摄像头甚至支付等 Native 功能。

既然是『简单来讲』，那么 JSBridge 的用途肯定不只『调用 Native 功能』这么简单宽泛。实际上，JSBridge 就像其名称中的『Bridge』的意义一样，是 Native 和非 Native 之间的桥梁，它的核心是 **构建 Native 和非 Native 间消息通信的通道**，而且是 **双向通信的通道**。

所谓 **双向通信的通道**

- JS 向 Native 发送消息 : 调用相关功能、通知 Native 当前 JS 的相关状态等。
- Native 向 JS 发送消息 : 回溯调用结果、消息推送、通知 JS 当前 Native 的状态等。

## JSBridge 的实现原理

JavaScript 是运行在一个单独的 JS Context 中（例如，WebView 的 Webkit 引擎、JSCore）。由于这些 Context 与原生运行环境的天然隔离，我们可以将这种情况与 RPC（Remote Procedure Call，远程过程调用）通信进行类比，将 Native 与 JavaScript 的每次互相调用看做一次 RPC 调用。

在 JSBridge 的设计中，可以把前端看做 RPC 的客户端，把 Native 端看做 RPC 的服务器端，从而 JSBridge 要实现的主要逻辑就出现了：**通信调用（Native 与 JS 通信）** 和 **句柄解析调用**。（如果你是个前端，而且并不熟悉 RPC 的话，你也可以把这个流程类比成 JSONP 的流程）

通过以上的分析，可以清楚地知晓 JSBridge 主要的功能和职责，接下来就以 **Hybrid 方案** 为案例从这几点来剖析 JSBridge 的实现原理。

### JavaScript 调用 Native

JavaScript 调用 Native 的方式，主要有两种：**注入 API** 和 **拦截 URL SCHEME**。

#### 注入API

注入 API 方式的主要原理是，通过 WebView 提供的接口，向 JavaScript 的 Context（window）中注入对象或者方法，让 JavaScript 调用时，直接执行相应的 Native 代码逻辑，达到 JavaScript 调用 Native 的目的。

#### 拦截 URL SCHEME

先解释一下 URL SCHEME：URL SCHEME是一种类似于url的链接，是为了方便app直接互相调用设计的，形式和普通的 url 近似，主要区别是 protocol 和 host 一般是自定义的，例如: qunarhy://hy/url?url=[ymfe.tech](https://link.juejin.im?target=http%3A%2F%2Fymfe.tech)，protocol 是 qunarhy，host 则是 hy。

拦截 URL SCHEME 的主要流程是：Web 端通过某种方式（例如 iframe.src）发送 URL Scheme 请求，之后 Native 拦截到请求并根据 URL SCHEME（包括所带的参数）进行相关操作。

在时间过程中，这种方式有一定的 **缺陷**：

- 使用 iframe.src 发送 URL SCHEME 会有 url 长度的隐患。
- 创建请求，需要一定的耗时，比注入 API 的方式调用同样的功能，耗时会较长。

但是之前为什么很多方案使用这种方式呢？因为它 **支持 iOS6**。而现在的大环境下，iOS6 占比很小，基本上可以忽略，所以并不推荐为了 iOS6 使用这种 **并不优雅** 的方式。

【注】：有些方案为了规避 url 长度隐患的缺陷，在 iOS 上采用了使用 Ajax 发送同域请求的方式，并将参数放到 head 或 body 里。这样，虽然规避了 url 长度的隐患，但是 WKWebView 并不支持这样的方式。

【注2】：为什么选择 iframe.src 不选择 locaiton.href ？因为如果通过 location.href 连续调用 Native，很容易丢失一些调用。

### Native 调用 JavaScript

相比于 JavaScript 调用 Native， Native 调用 JavaScript 较为简单，毕竟不管是 iOS 的 UIWebView 还是 WKWebView，还是 Android 的 WebView 组件，都以子组件的形式存在于 View/Activity 中，直接调用相应的 API 即可。

Native 调用 JavaScript，其实就是**执行拼接 JavaScript 字符串，从外部调用 JavaScript 中的方法**，因此 **JavaScript 的方法必须在全局的 window 上**。（闭包里的方法，JavaScript 自己都调用不了，更不用想让 Native 去调用了）

### 实现

```javascript
(function () {
    var id = 0,
        callbacks = {},
        registerFuncs = {};

    window.JSBridge = {
        // 调用 Native
        invoke: function(bridgeName, callback, data) {
            // 判断环境，获取不同的 nativeBridge
            var thisId = id ++; // 获取唯一 id
            callbacks[thisId] = callback; // 存储 Callback
            nativeBridge.postMessage({
                bridgeName: bridgeName,
                data: data || {},
                callbackId: thisId // 传到 Native 端
            });
        },
        receiveMessage: function(msg) {
            var bridgeName = msg.bridgeName,
                data = msg.data || {},
                callbackId = msg.callbackId, // Native 将 callbackId 原封不动传回
                responstId = msg.responstId;
            // 具体逻辑
            // bridgeName 和 callbackId 不会同时存在
            if (callbackId) {
                if (callbacks[callbackId]) { // 找到相应句柄
                    callbacks[callbackId](msg.data); // 执行调用
                }
            } elseif (bridgeName) {
                if (registerFuncs[bridgeName]) { // 通过 bridgeName 找到句柄
                    var ret = {},
                        flag = false;
                    registerFuncs[bridgeName].forEach(function(callback) => {
                        callback(data, function(r) {
                            flag = true;
                            ret = Object.assign(ret, r);
                        });
                    });
                    if (flag) {
                        nativeBridge.postMessage({ // 回调 Native
                            responstId: responstId,
                            ret: ret
                        });
                    }
                }
            }
        },
        register: function(bridgeName, callback) {
            if (!registerFuncs[bridgeName])  {
                registerFuncs[bridgeName] = [];
            }
            registerFuncs[bridgeName].push(callback); // 存储回调
        }
    };
})();
```

## JsBridge调用过程

1. Native初始化webview，注册Handler；加载页面完成后，将WebViewJavascriptBridge.js文件注入页面。查询消息队列是否有信息需要被接收。
2. H5页面初始化，注册Handler，查询消息队列是否有信息需要别接收。
3. 用户操作，H5调用本地功能：Js将消息内容放在`sendMessageQueue`中，并设置iframe的src为`yy://__QUEUE_MESSAGE__/` 
4. Webview设置的WebViewClient拦截到约定url，调用Webview的刷新消息队列的方法`flushMessageQueue`，此方法就是加载了一个url：`javascript:WebViewJavascriptBridge._fetchQueue();`,这也是Js中定义的方法，另外定义了一个回调；回调方法主要做了两件事：①判断Native是否为此返回数据保有响应回调操作，若有，则执行，若没有，则为判断callId，不为空时为这个callId初始化一个回调。②通过handlername判断是否为默认的Handler还是自定义的Handler，调用相应Handler的handler方法，入参为消息数据内容和第一步中定义的回调。【这段较为难消化，需要阅读代码来理解】
5. Js中`_fetchQueue`设置了iframe的src，内容为：`yy://return/_fetchQueue/`+第二步中放入`sendMessageQueue`中的消息内容。
6. WebViewClient拦截到url为`yy://return/`，调用WebView的`handlerReturnData`方法；通过url中定义的方法名，找到第四个步骤中定义的回调，并调用。回调方法走完后，删除此回调方法。
7. 如果Js在调用Handler的时候设置了回调方法，也就是在第四步骤中的含有callId，就会调用queueMessage的方法，然后往下就是走Native给Js发送消息的步骤。
   **Ps:** Native给Js发送消息的步骤跟上述从第三步骤到第七步骤完全相同，只不过Native和Js对象调换位置即可。