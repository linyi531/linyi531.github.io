---
title: 深入浅出Node.js学习笔记（三）
date: 2020-07-02 00:35:24
tags:
  - Node.js
categories: Node.js
cover_img: https://tva1.sinaimg.cn/large/007S8ZIlgy1ggby0l39a6j31c20u0kjt.jpg
feature_img: https://tva1.sinaimg.cn/large/007S8ZIlgy1ggby0l39a6j31c20u0kjt.jpg
---

# 深入浅出Node.js学习笔记（三）

## 第七章 网络编程

### 构建TCP服务

#### TCP

TCP全名为传输控制协议，在OSI模型(由七层组成，分别为物理层、数据链结层、网络层、 传输层、会话层、表示层、应用层)中属于传输层协议。许多应用层协议基于TCP构建，典型的 是HTTP、SMTP、IMAP等协议。

![image-20191223151304958](https://tva1.sinaimg.cn/large/006tNbRwgy1ga6odzsgm4j30f00ecgmy.jpg)

TCP是面向连接的协议，其显著的特征是在传输之前需要3次握手形成会话

![image-20191223151332410](https://tva1.sinaimg.cn/large/006tNbRwgy1ga6oefcwg4j30hm0hwmy7.jpg)

只有会话形成之后，服务器端和客户端之间才能互相发送数据。在创建会话的过程中，服务 器端和客户端分别提供一个套接字，这两个套接字共同形成一个连接。服务器端与客户端则通过 套接字实现两者之间连接的操作。

#### 创建TCP服务器端

创建一个TCP服务器端来接受网络请求

```javascript
var net = require('net');
var server = net.createServer(function (socket) { 
  // 新的连接
  socket.on('data', function (data) {
    socket.write("你好"); 
  });
  socket.on('end', function () { 
    console.log('连接断开');
  });
  socket.write("欢迎光临《深入浅出Node.js》示例:\n"); 
});
server.listen(8124, function () { 
  console.log('server bound');
});
```

我们通过net.createServer(listener)即可创建一个TCP服务器，listener是连接事件connection的侦听器

也可以采用如下的方式进行侦听:

```javascript
var server = net.createServer();
server.on('connection', function (socket) { 
  // 新的连接
}); server.listen(8124);
```

我们可以利用Telnet工具作为客户端对刚才创建的简单服务器进行会话交流

```javascript
$ telnet 127.0.0.1 8124
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'. 欢迎光临《深入浅出Node.js》示例: 
hi
你好
```

除了端口外，同样我们也可以对Domain Socket进行监听

```javascript
server.listen('/tmp/echo.sock');
```

通过net模块自行构造客户端进行会话

```javascript
var net = require('net');
var client = net.connect({port: 8124}, function () {
  //'connect' listener
  console.log('client connected');
  client.write('world!\r\n'); 
});
client.on('data', function (data) {
  console.log(data.toString());
  client.end();
});
client.on('end', function () { 
  console.log('client disconnected');
});
```

将以上客户端代码存为client.js并执行

```javascript
$ node client.js
client connected 欢迎光临《深入浅出Node.js》示例:

你好
client disconnected
```

#### TCP 服务的事件

##### 1. 服务器事件

对于通过net.createServer()创建的服务器而言，它是一个EventEmitter实例，它的自定义 事件有如下几种。

* listening:在调用server.listen()绑定端口或者Domain Socket后触发，简洁写法为 server.listen(port,listeningListener)，通过listen()方法的第二个参数传入。
* connection:每个客户端套接字连接到服务器端时触发，简洁写法为通过net.createServer()，最后一个参数传递。

- close:当服务器关闭时触发，在调用server.close()后，服务器将停止接受新的套接字连接，但保持当前存在的连接，等待所有连接都断开后，会触发该事件。
- error:当服务器发生异常时，将会触发该事件。比如侦听一个使用中的端口，将会触发 一个异常，如果不侦听error事件，服务器将会抛出异常。

##### 2. 连接事件

服务器可以同时与多个客户端保持连接，对于每个连接而言是典型的可写可读Stream对象。 Stream对象可以用于服务器端和客户端之间的通信，既可以通过data事件从一端读取另一端发来 的数据，也可以通过write()方法从一端向另一端发送数据。它具有如下自定义事件。

- data:当一端调用write()发送数据时，另一端会触发data事件，事件传递的数据即是 write()发送的数据。
- end:当连接中的任意一端发送了FIN数据时，将会触发该事件。
- connect:该事件用于客户端，当套接字与服务器端连接成功时会被触发。
- drain:当任意一端调用write()发送数据时，当前这端会触发该事件。
- error:当异常发生时，触发该事件。
- close:当套接字完全关闭时，触发该事件。
- timeout:当一定时间后连接不再活跃时，该事件将会被触发，通知用户当前该连接已经被闲置了。

TCP针对网络中的小数据包有一定的优化策略:Nagle算法。如果每次只发送一个字节的内容而不优化，网络中将充满只有极少数有效数据的数据包，将十分浪费网络资源。 Nagle算法针对这种情况，要求缓冲区的数据达到一定数量或者一定时间后才将其发出，所以小 数据包将会被Nagle算法合并，以此来优化网络。这种优化虽然使网络带宽被有效地使用，但是 数据有可能被延迟发送。

在Node中，由于TCP默认启用了Nagle算法，可以调用socket.setNoDelay(true)去掉Nagle算 法，使得write()可以立即发送数据到网络中。

尽管在网络的一端调用write()会触发另一端的data事件，但是并不 意味着每次write()都会触发一次data事件，在关闭掉Nagle算法后，另一端可能会将接收到的多 个小数据包合并，然后只触发一次data事件。

### 构建UDP服务

UDP又称用户数据包协议，与TCP一样同属于网络传输层。UDP与TCP最大的不同是UDP不是 面向连接的。在UDP中，一个套接字可以与多个UDP服务通信，它 虽然提供面向事务的简单不可靠信息传输服务，在网络差的情况下存在丢包严重的问题，但是由于 它无须连接，资源消耗低，处理快速且灵活。

#### 创建UDP套接字

创建UDP套接字十分简单，UDP套接字一旦创建，既可以作为客户端发送数据，也可以作为服务器端接收数据

```javascript
var dgram = require('dgram');
var socket = dgram.createSocket("udp4");
```

#### 创建UDP服务器端

若想让UDP套接字接收网络消息，只要调用dgram.bind(port, [address])方法对网卡和端口进行绑定即可

```javascript
var dgram = require("dgram");
var server = dgram.createSocket("udp4");
server.on("message", function (msg, rinfo) {
  console.log("server got: " + msg + " from " +
rinfo.address + ":" + rinfo.port); 
});
server.on("listening", function () { 
  var address = server.address();
  console.log("server listening " + address.address + ":" + address.port);
}); 
server.bind(41234);
```

该套接字将接收所有网卡上41234端口上的消息。在绑定完成后，将触发listening事件。

#### 创建UDP客户端

我们创建一个客户端与服务器端进行对话

```javascript
var dgram = require('dgram');
var message = new Buffer("深入浅出Node.js");
var client = dgram.createSocket("udp4");
client.send(message, 0, message.length, 41234, "localhost", function(err, bytes) {
  client.close(); 
});
```

保存为client.js并执行

```javascript
$ node server.js
server listening 0.0.0.0:41234
server got: 深入浅出Node.js from 127.0.0.1:58682
```

当套接字对象用在客户端时，可以调用send()方法发送消息到网络中。send()方法的参数如下:

```javascript
socket.send(buf, offset, length, port, address, [callback])
```

这些参数分别为要发送的Buffer、Buffer的偏移、Buffer的长度、目标端口、目标地址、发送 完成后的回调。与TCP套接字的write()相比，send()方法的参数列表相对复杂，但是它更灵活的 地方在于可以随意发送数据到网络中的服务器端，而TCP如果要发送数据给另一个服务器端，则 需要重新通过套接字构造新的连接。

#### UDP 套接字事件

UDP套接字相对TCP套接字使用起来更简单，它只是一个EventEmitter的实例，而非Stream的实例。它具备如下自定义事件。

* message:当UDP套接字侦听网卡端口后，接收到消息时触发该事件，触发携带的数据为 消息Buffer对象和一个远程地址信息。
* listening:当UDP套接字开始侦听时触发该事件。
* close:调用close()方法时触发该事件，并不再触发message事件。如需再次触发message事件，重新绑定即可。
* error:当异常发生时触发该事件，如果不侦听，异常将直接抛出，使进程退出。

### 构建HTTP服务

构建HTTP服务

```javascript
var http = require('http'); http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
}).listen(1337, '127.0.0.1');
console.log('Server running at http://127.0.0.1:1337/');
```

#### HTTP

HTTP的全称是超文本传输协议，英文写作HyperText Transfer Protocol。

HTTP构建在TCP之上，属于应用层协议。在HTTP的两 端是服务器和浏览器，即著名的B/S模式。

HTTP是基于请求响应式的，以一问一答的方式实 现服务，虽然基于TCP会话，但是本身却并无会话的特点。

HTTP服务只做两件事情:处理HTTP请求和发送HTTP响应

#### http 模块

Node的http模块包含对HTTP处理的封装。在Node中，HTTP服务继承自TCP服务器(net模块)，它能够与多个客户端保持连接，由于其采用事件驱动的形式，并不为每一个连接创建额外的线程或进程，保持很低的内存占用，所以能实现高并发。HTTP服务与TCP服务模型有区别的 地方在于，在开启keepalive后，一个TCP会话可以用于多次请求和响应。TCP服务以connection 为单位进行服务，HTTP服务以request为单位进行服务。http模块即是将connection到request的 过程进行了封装

![image-20191223162355440](https://tva1.sinaimg.cn/large/006tNbRwgy1ga6qfnzupdj310q0d2wfl.jpg)

http模块将连接所用套接字的读写抽象为ServerRequest和ServerResponse对象， 它们分别对应请求和响应操作。在请求产生的过程中，http模块拿到连接中传来的数据，调用二 进制模块http_parser进行解析，在解析完请求报文的报头后，触发request事件，调用用户的业 务逻辑。

![image-20191223162545615](https://tva1.sinaimg.cn/large/006tNbRwgy1ga6qhkd2djj30og0m2jst.jpg)

##### 1. HTTP请求

对于TCP连接的读操作，http模块将其封装为ServerRequest对象。报文头部将会通过http_parser进行解析。

```http
> GET / HTTP/1.1
> User-Agent: curl/7.24.0 (x86_64-apple-darwin12.0) libcurl/7.24.0 OpenSSL/0.9.8r zlib/1.2.5 2
> Host: 127.0.0.1:1337 > Accept: */*
>
```

报文头第一行GET / HTTP/1.1被解析之后分解为如下属性。

* req.method属性:值为GET，是为请求方法，常见的请求方法有GET、POST、DELETE、PUT、CONNECT等几种。
* req.url属性:值为/。
* req.httpVersion属性:值为1.1。

其余报头是很规律的Key: Value格式，被解析后放置在req.headers属性上传递给业务逻辑以供调用

```javascript
headers: { 
  'user-agent': 'curl/7.24.0 (x86_64-apple-darwin12.0) libcurl/7.24.0 OpenSSL/0.9.8r zlib/1.2.5',
    host: '127.0.0.1:1337', 
    accept: '*/*' 
},
```

报文体部分则抽象为一个只读流对象，如果业务逻辑需要读取报文体中的数据，则要在这个 数据流结束后才能进行操作

##### 2. HTTP响应

它封装了对底层连接的写操作，可以将 其看成一个可写的流对象。它影响响应报文头部信息的API为res.setHeader()和res. writeHead()。

```javascript
res.writeHead(200, {'Content-Type': 'text/plain'});
```

可以调用setHeader进行多次设置，但只有调用writeHead后，报头才会写入到连接中。除此之外，http模块会自动帮你设置一些头信息

```javascript
< Date: Sat, 06 Apr 2013 08:01:44 GMT
< Connection: keep-alive
< Transfer-Encoding: chunked 
<
```

报文体部分则是调用res.write()和res.end()方法实现，后者与前者的差别在于res.end()会 先调用write()发送数据，然后发送信号告知服务器这次响应结束

响应结束后，HTTP服务器可能会将当前的连接用于下一个请求，或者关闭连接。

报头是在报文体发送前发送的，一旦开始了数据的发送，writeHead()和setHeader()将不 再生效。这由协议的特性决定。

无论服务器端在处理业务逻辑时是否发生异常，务必在结束时调用res.end()结束请 求，否则客户端将一直处于等待的状态。当然，也可以通过延迟res.end()的方式实现客户端与 服务器端之间的长连接，但结束时务必关闭连接。

##### 3. HTTP服务的事件

- connection事件:在开始HTTP请求和响应前，客户端与服务器端需要建立底层的TCP连接，这个连接可能因为开启了keep-alive，可以在多次请求响应之间使用;当这个连接建立时，服务器触发一次connection事件。
- request事件:建立TCP连接后，http模块底层将在数据流中抽象出HTTP请求和HTTP响应，当请求数据发送到服务器端，在解析出HTTP请求头后，将会触发该事件;在res.end() 后，TCP连接可能将用于下一次请求响应。
- close事件:与TCP服务器的行为一致，调用server.close()方法停止接受新的连接，当已有的连接都断开时，触发该事件;可以给server.close()传递一个回调函数来快速注册该事件。
- checkContinue事件:某些客户端在发送较大的数据时，并不会将数据直接发送，而是先发送一个头部带Expect: 100-continue的请求到服务器，服务器将会触发checkContinue 事件;如果没有为服务器监听这个事件，服务器将会自动响应客户端100 Continue的状态码，表示接受数据上传;如果不接受数据的较多时，响应客户端400 Bad Request拒绝客 户端继续发送数据即可。需要注意的是，当该事件发生时不会触发request事件，两个事件之间互斥。当客户端收到100 Continue后重新发起请求时，才会触发request事件。
- connect事件:当客户端发起CONNECT请求时触发，而发起CONNECT请求通常在HTTP代理时出现;如果不监听该事件，发起该请求的连接将会关闭。
- upgrade事件:当客户端要求升级连接的协议时，需要和服务器端协商，客户端会在请求 头中带上Upgrade字段，服务器端会在接收到这样的请求时触发该事件。这在后文的 WebSocket部分有详细流程的介绍。如果不监听该事件，发起该请求的连接将会关闭。
- clientError事件:连接的客户端触发error事件时，这个错误会传递到服务器端，此时触发该事件。

#### HTTP 客户端

http模块提供了一个底层API:http.request(options, connect)，用于构造HTTP客户端

```javascript
var options = { 
  hostname: '127.0.0.1', 
  port: 1334,
  path: '/',
  method: 'GET'
 };
var req = http.request(options, function(res) {
  console.log('STATUS: ' + res.statusCode);
  console.log('HEADERS: ' + JSON.stringify(res.headers));
  res.setEncoding('utf8');
  res.on('data', function (chunk) {
    console.log(chunk);
  }); 
});
req.end();
```

其中options参数决定了这个HTTP请求头中的内容

* host:服务器的域名或IP地址，默认为localhost。
* hostname:服务器名称。
* port:服务器端口，默认为80。
* localAddress:建立网络连接的本地网卡。 
* socketPath:Domain套接字路径。
* method:HTTP请求方法，默认为GET。
* path:请求路径，默认为/。
* headers:请求头对象。
* auth:Basic认证，这个值将被计算成请求头中的Authorization部分。

报文体的内容由请求对象的write()和end()方法实现:通过write()方法向连接中写入数据， 通过end()方法告知报文结束。它与浏览器中的Ajax调用几近相同，Ajax的实质就是一个异步的 网络HTTP请求。

##### 1. HTTP响应

在ClientRequest对象中，它的事件叫做 response。ClientRequest在解析响应报文时，一解析完响应头就触发response事件，同时传递一 个响应对象以供操作ClientResponse。后续响应报文体以只读流的方式提供

```javascript
function(res) {
  console.log('STATUS: ' + res.statusCode);
  console.log('HEADERS: ' + JSON.stringify(res.headers));
  res.setEncoding('utf8');
  res.on('data', function (chunk) {
    console.log(chunk); 
  });
}
```

##### 2. HTTP 代理

http提供的ClientRequest对象也是基于TCP层实现的，在 keepalive的情况下，一个底层会话连接可以多次用于请求。为了重用TCP连接，http模块包含一 个默认的客户端代理对象http.globalAgent。它对每个服务器端(host + port)创建的连接进行了 管理，默认情况下，通过ClientRequest对象对同一个服务器端发起的HTTP请求最多可以创建5 个连接。它的实质是一个连接池

![image-20191223173929894](https://tva1.sinaimg.cn/large/006tNbRwgy1ga6smaeshbj30lc0l8jtb.jpg)

调用HTTP客户端同时对一个服务器发起10次HTTP请求时，其实质只有5个请求处于并发状态，后续的请求需要等待某个请求完成服务后才真正发出。这与浏览器对同一个域名有下载连接 数的限制是相同的行为。

设置agent选项为false值，以脱离连接池的管理，使得请求不受并发的限制。Agent对象的sockets和requests属性分别表示当前连接池中使用中的连接数和处于等待状态 的请求数，在业务中监视这两个值有助于发现业务状态的繁忙程度。

##### 3. HTTP客户端事件

* response:与服务器端的request事件对应的客户端在请求发出后得到服务器端响应时， 会触发该事件。

- socket:当底层连接池中建立的连接分配给当前请求对象时，触发该事件。
- connect:当客户端向服务器端发起CONNECT请求时，如果服务器端响应了200状态码，客户端将会触发该事件。 
- upgrade:客户端向服务器端发起Upgrade请求时，如果服务器端响应了101 Switching Protocols状态，客户端将会触发该事件。
- continue:客户端向服务器端发起Expect: 100-continue头信息，以试图发送较大数据量，如果服务器端响应100 Continue状态，客户端将触发该事件。

### 构建WebSocket服务

- WebSocket客户端基于事件的编程模型与Node中自定义事件相差无几。
- WebSocket实现了客户端与服务器端之间的长连接，而Node事件驱动的方式十分擅长与大量的客户端保持高并发连接。

WebSocket与传统HTTP有如下好处：

* 客户端与服务器端只建立一个TCP连接，可以使用更少的连接。
* WebSocket服务器端可以推送数据到客户端，这远比HTTP请求响应模式更灵活、更高效。 
* 有更轻量级的协议头，减少数据传送量。

浏览器与服务器端创建WebSocket协议请求，在请求完成后连接打开，每50毫 秒向服务器端发送一次数据，同时可以通过onmessage()方法接收服务器端传来的数据。能够双向通信。

长轮询的原理是客户端向服务器端发起请求， 服务器端只在超时或有数据响应时断开连接(res.end());客户端在收到数据或者超时后重新发起请求。

WebSocket更接近于传输层协 议，它并没有在HTTP的基础上模拟服务器端的推送，而是在TCP上定义独立的协议。

WebSocket协议主要分为两个部分:**握手和数据传输。**

#### WebSocket 握手

客户端建立连接时，通过HTTP发起请求报文

```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ== 
Sec-WebSocket-Protocol: chat, superchat 
Sec-WebSocket-Version: 13
```

与普通的HTTP请求协议略有区别的部分在于如下这些协议头

```http
Upgrade: websocket 
Connection: Upgrade
```

上述两个字段表示请求服务器端升级协议为WebSocket。

其中Sec-WebSocket-Key用于安全校验：

```http
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
```

Sec-WebSocket-Key的值是随机生成的Base64编码的字符串。服务器端接收到之后将其与字符串258EAFA5-E914-47DA-95CA-C5AB0DC85B11相连，形成字符串dGhlIHNhbXBsZSBub25jZQ==258EAFA5- 3 E914-47DA-95CA-C5AB0DC85B11，然后通过sha1安全散列算法计算出结果后，再进行Base64编码， 最后返回给客户端。

算法如下:

```javascript
var crypto = require('crypto');
var val = crypto.createHash('sha1').update(key).digest('base64');
```

下面两个字段指定子协议和版本号:

```http
Sec-WebSocket-Protocol: chat, superchat 
Sec-WebSocket-Version: 13
```

服务器端在处理完请求后，响应如下报文

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade 6 Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

上面的报文告之客户端正在更换协议，更新应用层协议为WebSocket协议，并在当前的套接 字连接上应用新协议。剩余的字段分别表示服务器端基于Sec-WebSocket-Key生成的字符串和选中的子协议。客户端将会校验Sec-WebSocket-Accept的值，如果成功，将开始接下来的数据传输。

一旦WebSocket握手成功，服务器端与客户端将会呈现对等的效果，都能接收和发送消息。

#### WebSocket 数据传输

在握手顺利完成后，当前连接将不再进行HTTP的交互，而是开始WebSocket的数据帧协议，实现客户端与服务器端的数据交换

![image-20191224155417997](https://tva1.sinaimg.cn/large/006tNbRwgy1ga7v775i6uj30lg0k6jsy.jpg)

握手完成后，客户端的onopen()将会被触发执行

```javascript
socket.onopen = function () { 
  // TODO: opened()
};
```

服务器端则没有onopen()方法可言

为了完成TCP套接字事件到WebSocket事件的封装，需要在接收数据时进行处理，WebSocket的数据帧协议即是在底层data事件上封装完成的

```javascript
WebSocket.prototype.setSocket = function (socket) {
  this.socket = socket;
  this.socket.on('data', this.receiver);
};
```

同样的数据发送时，也需要做封装操作

```javascript
WebSocket.prototype.send = function (data) {
  this._send(data);
};
```

当客户端调用send()发送数据时，服务器端触发onmessage();当服务器端调用send()发送数据时，客户端的onmessage()触发。当我们调用send()发送一条数据时，协议可能将这个数据封装 为一帧或多帧数据，然后逐帧发送。

为了安全考虑，客户端需要对发送的数据帧进行掩码处理，服务器一旦收到无掩码帧(比如 中间拦截破坏)，连接将关闭。而服务器发送到客户端的数据帧则无须做掩码处理，同样，如果客户端收到带掩码的数据帧，连接也将关闭。

![image-20191224164858601](https://tva1.sinaimg.cn/large/006tNbRwgy1ga7ws1ac04j311m0f0wg0.jpg)

每8位为一列，也即1个字节

* fin:如果这个数据帧是最后一帧，这个fin位为1，其余情况为0。当一个数据没有被分为 多帧时，它既是第一帧也是最后一帧。
* rsv1、rsv2、rsv3:各为1位长，3个标识用于扩展，当有已协商的扩展时，这些值可能为 1，其余情况为0。
* opcode:长为4位的操作码，可以用来表示0到15的值，用于解释当前数据帧。
  * 0表示附加数据帧
  * 1表示文本数据帧
  * 2表示二进制数据帧
  * 8表示发送一个连接关闭的数据帧
  * 9 表示ping数据帧
  * 10表示pong数据帧（ping数据帧和pong数据帧用于心跳检测，当一端发送ping数据帧时，另一端必须发送pong数据帧作为响应，告知对方这一端仍然处于响应状态。）
  * 其余值暂时没有定义。
* masked:表示是否进行掩码处理，长度为1。客户端发送给服务器端时为1，服务器端发送给客户端时为0。
* payload length:一个7、7+16或7+64位长的数据位，标识数据的长度
  * 如果值在0~125 之间，那么该值就是数据的真实长度;
  * 如果值是126，则后面16位的值是数据的真实长度; 
  * 如果值是127，则后面64位的值是数据的真实长度
* masking key:当masked为1时存在，是一个32位长的数据位，用于解密数据。 
* payload data:我们的目标数据，位数为8的倍数。

客户端发送消息时，需要构造一个或多个数据帧协议报文。

### 网络服务与安全

SSL作为一种安全协议，它在传输层提供对网络连接加密的功能。对于应用层而 言，它是透明的，数据在传递到应用层之前就已经完成了加密和解密的过程。

最初的SSL应用在 Web上，被服务器端和浏览器端同时支持，随后IETF将其标准化，称为TLS(Transport Layer Security，安全传输层协议)。

Node在网络安全上提供了3个模块，分别为crypto、tls、https。其中crypto主要用于加密解密。tls模块提供了与net模块类似的功能，区别在于它建立在TLS/SSL加密的TCP 连接上。对于https而言，它完全与http模块接口一致，区别也仅在于它建立于安全的连接 之上。

#### TLS/SSL

##### 1. 密钥

TLS/SSL是一个公钥/私钥的结构，它是一个非对称的结构，每个服务器端和客户端都有自己 的公私钥。公钥用来加密要传输的数据，私钥用来解密接收到的数据。公钥和私钥是配对的，通 过公钥加密的数据，只有通过私钥才能解密，所以在建立安全传输之前，客户端和服务器端之间 需要互换公钥。客户端发送数据时要通过服务器端的公钥进行加密，服务器端发送数据时则需要 客户端的公钥进行加密，如此才能完成加密解密的过程

![image-20191224170115187](https://tva1.sinaimg.cn/large/006tNbRwgy1ga7x4t082vj311e0fm40g.jpg)

Node在底层采用的是openssl实现TLS/SSL的，为此要生成公钥和私钥可以通过openssl完成。

我们分别为服务器端和客户端生成私钥

```javascript
// 生成服务器端私钥
$ openssl genrsa -out server.key 1024 
// 生成客户端私钥
$ openssl genrsa -out client.key 1024
```

上述命令生成了两个1024位长的RSA私钥文件，我们可以通过它继续生成公钥

```javascript
$ openssl rsa -in server.key -pubout -out server.pem
$ openssl rsa -in client.key -pubout -out client.pem
```

中间人攻击：为了解决这种问题，数据传 输过程中还需要对得到的公钥进行认证，以确认得到的公钥是出自目标服务器。如果不能保证这 种认证，中间人可能会将伪造的站点响应给用户，从而造成经济损失。

![image-20191224191346081](https://tva1.sinaimg.cn/large/006tNbRwgy1ga80yq45adj30oa0bujs6.jpg)

TLS/SSL引入了数字证书来进行认证。与直接用公钥不同，数字证书中 包含了服务器的名称和主机名、服务器的公钥、签名颁发机构的名称、来自签名颁发机构的签名。 在连接建立前，会通过证书中的签名确认收到的公钥是来自目标服务器的，从而产生信任关系。

##### 2. 数字证书

CA(Certificate Authority，数字证 书认证中心)。CA的作用是为站点颁发证书，且这个证书中具有CA通过自己的公钥和私钥实现 的签名。

为了得到签名证书，服务器端需要通过自己的私钥生成CSR(Certificate Signing Request，证 书签名请求)文件。CA机构将通过这个文件颁发属于该服务器端的签名证书，只要通过CA机构 就能验证证书是否合法。

自签名证书：自己扮演CA机构，给自 己的服务器端颁发签名证书。

```javascript
$ openssl genrsa -out ca.key 1024
$ openssl req -new -key ca.key -out ca.csr
$ openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

![image-20191224191611387](https://tva1.sinaimg.cn/large/006tNbRwgy1ga8117jmefj30hu0a8q3i.jpg)

服务器端需要向CA机构申请签名证书。这个过程中的 Common Name要匹配服务器域名，否则在后续的认证过程中会出错。

```javascript
$ openssl req -new -key server.key -out server.csr
```

签名过程需要CA的证书和私钥参与， 最终颁发一个带有CA签名的证书

```javascript
$ openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -out server.crt
```

客户端在发起安全连接前会去获取服务器端的证书，并通过CA的证书验证服务器端证书的 真伪。除了验证真伪外，通常还含有对服务器名称、IP地址等进行验证的过程。

![image-20191224191819509](https://tva1.sinaimg.cn/large/006tNbRwgy1ga813fr14qj30su09w0tp.jpg)

CA机构将证书颁发给服务器端后，证书在请求的过程中会被发送给客户端，客户端需要通 过CA的证书验证真伪。如果是知名的CA机构，它们的证书一般预装在浏览器中。如果是自己扮 演CA机构，颁发自有签名证书则不能享受这个福利，客户端需要获取到CA的证书才能进行验证。

上述的过程中可以看出，签名证书是一环一环地颁发的，但是在CA那里的证书是不需要上 级证书参与签名的，这个证书我们通常称为根证书。

#### TLS 服务

##### 1. 创建服务器端

创建一个安全的TCP服务

```javascript
var tls = require('tls'); 
var fs = require('fs');
var options = {
  key: fs.readFileSync('./keys/server.key'), 
  cert: fs.readFileSync('./keys/server.crt'),
  requestCert: true,
  ca: [ fs.readFileSync('./keys/ca.crt') ]
};
var server = tls.createServer(options, function (stream) {
  console.log('server connected', stream.authorized ? 'authorized' : 'unauthorized');
  stream.write("welcome!\n");
  stream.setEncoding('utf8');
  stream.pipe(stream);
});
server.listen(8000, function() {
  console.log('server bound'); 
});
```

启动上述服务后，通过下面的命令可以测试证书是否正常

```javascript
$ openssl s_client -connect 127.0.0.1:8000
```

##### 2. TLS客户端

模拟客户端，要为客户端生成属于自己的私钥和 签名

```javascript
// 创建私钥
$ openssl genrsa -out client.key 1024
// 生成CSR
$ openssl req -new -key client.key -out client.csr
// 生成签名证书
$ openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in client.csr -out client.crt
```

```javascript
//并创建客户端，代码如下:
var fs = require('fs'); 
var tls = require('tls');
var options = {
  key: fs.readFileSync('./keys/client.key'), 
  cert: fs.readFileSync('./keys/client.crt'), 
  ca: [ fs.readFileSync('./keys/ca.crt') ]
};

var stream = tls.connect(8000, options, function () {
  console.log('client connected', stream.authorized ? 'authorized' : 'unauthorized');
  process.stdin.pipe(stream);
});
stream.setEncoding('utf8'); 
stream.on('data', function(data) {
  console.log(data); 
});
stream.on('end', function() { 
  server.close();
});
```

启动客户端的过程中，用到了为客户端生成的私钥、证书、CA证书。客户端启动之后可以 9 在输入流中输入数据，服务器端将会回应相同的数据。

至此我们完成了TLS的服务器端和客户端的创建。与普通的TCP服务器端和客户端相比，TLS 的服务器端和客户端仅仅只在证书的配置上有差别，其余部分基本相同。

#### HTTPS 服务

##### 1. 准备证书

HTTPS服务需要用到私钥和签名证书，我们可以直接用上文生成的私钥和证书。

##### 2. 创建HTTPS服务

创建HTTPS服务只比HTTP服务多一个选项配置

```javascript
var https = require('https'); 
var fs = require('fs');
var options = {
  key: fs.readFileSync('./keys/server.key'), 
  cert: fs.readFileSync('./keys/server.crt')
};
https.createServer(options, function (req, res) {
  res.writeHead(200);
  res.end("hello world\n");
}).listen(8000);
```

##### 3. HTTPS客户端

```javascript
var https = require('https');
var fs = require('fs'); 
var options = {
  hostname: 'localhost',
  port: 8000,
  path: '/', 
  method: 'GET',
  key: fs.readFileSync('./keys/client.key'),
  cert: fs.readFileSync('./keys/client.crt'),
  ca: [fs.readFileSync('./keys/ca.crt')]
};
options.agent = new https.Agent(options);
var req = https.request(options, function(res) {
  res.setEncoding('utf-8');
  res.on('data', function(d) {
    console.log(d); 
  });
}); 
req.end();
req.on('error', function(e) { 
  console.log(e);
});
```

## 第八章 构建Web应用

### 基础功能

#### 请求方法

最常见的请求方法是GET和POST，除此之外，还有HEAD、DELETE、PUT、CONNECT等方法。

HTTP_Parser在解析请求报文的时候，将报文头抽取出来，设置为req.method。

通过请求方法来决定响应行为：PUT代表新建一个资源，POST表示要更新一个资源，GET表示查看一个资源， 而DELETE表示删除一个资源。

```javascript
function (req, res) { 
  switch (req.method) {
    case 'POST':
      update(req, res);
      break;
    case 'DELETE':
      remove(req, res);
      break; 
    case 'PUT':
      create(req, res);
      break; 
    case 'GET': 
    default:
      get(req, res); 
  }
}
```

#### 路径解析

路径部分存在于报文的第一行的第二部分,HTTP_Parser将其解析为req.url

客户端代理(浏览器)会将这个地址解析成报文，将路径和查询部分放在报文第一行。需要 注意的是，hash部分会被丢弃，不会存在于报文的任何地方。

#### 查询字符串

查询字符串位于路径之后，在地址栏中路径后的?foo=bar&baz=val字符串就是查询字符串。 这个字符串会跟随在路径后，形成请求报文首行的第二部分。这部分内容经常需要为业务逻辑所 用，Node提供了querystring模块用于处理这部分数据

```javascript
var url = require('url');
var querystring = require('querystring');
var query = querystring.parse(url.parse(req.url).query);
```

更简洁的方法是给url.parse()传递第二个参数

```javascript
var query = url.parse(req.url, true).query;
```

它会将foo=bar&baz=val解析为一个JSON对象

```javascript
{
  foo: 'bar',
  baz: 'val' 
}
```

要注意的点是，如果查询字符串中的键出现多次，那么它的值会是一个数组

```javascript
// foo=bar&foo=baz
var query = url.parse(req.url, true).query; 
// {
// 		foo: ['bar', 'baz']
// }
```

业务的判断一定要检查值是数组还是字符串，否则可能出现TypeError异常的情况。

#### Cookie

Cookie的处理分为如下几步。

* 服务器向客户端发送Cookie。
* 浏览器将Cookie保存。
* 之后每次浏览器都会将Cookie发向服务器端。

客户端发送的Cookie在请求报文的Cookie字段中，我们可以通过curl工具构造这个字段

```javascript
curl -v -H "Cookie: foo=bar; baz=val" "http://127.0.0.1:1337/path?foo=bar&foo=baz"
```

HTTP_Parser会将所有的报文字段解析到req.headers上，那么Cookie就是req.headers. cookie。

```javascript
Set-Cookie: name=value; Path=/; Expires=Sun, 23-Apr-23 09:01:35 GMT; Domain=.domain.com;
```

name=value是必须包含的部分，其余部分皆是可选参数。

* path表示这个Cookie影响到的路径，当前访问的路径不满足该匹配时，浏览器则不发送这 个Cookie。
* Expires和Max-Age是用来告知浏览器这个Cookie何时过期的，如果不设置该选项，在关闭浏览器时会丢失掉这个Cookie。如果设置了过期时间，浏览器将会把Cookie内容写入到磁盘中并保存，下次打开浏览器依旧有效。Expires的值是一个UTC格式的时间字符串，告知浏览器此Cookie何时将过期，Max-Age则告知浏览器此Cookie多久后过期。前者一般而言不存在问题，但是如果服务器端的时间和客户端的时间不能匹配，这种时间设置就会存在偏差。为此，Max-Age告知浏览器这条Cookie多久之后过期，而不是一个具体的时间点。
* HttpOnly告知浏览器不允许通过脚本document.cookie去更改这个Cookie值，事实上，设置 HttpOnly之后，这个值在document.cookie中不可见。但是在HTTP请求的过程中，依然会 发送这个Cookie到服务器端。
* Secure。当Secure值为true时，在HTTP中是无效的，在HTTPS中才有效，表示创建的Cookie只能在HTTPS连接中被浏览器传递到服务器端进行会话验证，如果是HTTP连接则不会传递该信息，所以很难被窃听到。

##### Cookie的性能影响

由于Cookie的实现机制，一旦服务器端向客户端发送了设置Cookie的意图，除非Cookie过期， 否则客户端每次请求都会发送这些Cookie到服务器端，一旦设置的Cookie过多，将会导致报头较 大。大多数的Cookie并不需要每次都用上，因为这会造成带宽的部分浪费。在YSlow的性能优化 规则中有这么一条:

* 减小Cookie的大小

  如果在域名的根节点设置Cookie，几乎所有子路径下的请求都会带上这些Cookie，这些Cookie在某些情况下是有用的，但是在有些情况下是完全无用的。其中以静态文件 最为典型，静态文件的业务定位几乎不关心状态，Cookie对它而言几乎是无用的，但是一旦有 Cookie设置到相同域下，它的请求中就会带上Cookie。

* 为静态组件使用不同的域名

  * 为不需要Cookie的组件换个域名可以实现减少无效Cookie的传输。
  * 额外的域名还可以突破浏览器下载线程数量的限制，因为域名不同，可以将下载线程数翻倍
  * 缺点：多一个域名就多一次DNS查询

* 减少DNS查询

  浏览器都会进行DNS缓存，以削弱这个副作用的影响

**Cookie对于敏感数据的保护是无效的。**

#### Session

Session的数据只保留在服务器端，客户 端无法修改，这样数据的安全性得到一定的保障，数据也无须在协议中每次都被传递。

##### 将每个客户和服务器中的数据一一对应的方法

* 第一种:基于Cookie来实现用户和数据的映射

  将口令放在Cookie中。口令一旦被篹改，就丢失了映射关系，也无法修改服务器端存在的数据了。并且Session的有效期通常较短， 普遍的设置是20分钟，如果在20分钟内客户端和服务器端没有交互产生，服务器端就将数据删除。由于数据过期时间较短，且在服务器端存储数据，因此安全性相对较高。

  一旦服务器端启用了Session，它将约定一个键值作为Session的口令，这个值可以随意约定一旦服务器检查到用户请求 Cookie中没有携带该值，它就会为之生成一个值，这个值是唯一且不重复的值，并设定超时时间。

  ```javascript
  var sessions = {};
  var key = 'session_id';
  var EXPIRES = 20 * 60 * 1000;
  var generate = function () {
    var session = {};
    session.id = (new Date()).getTime() +Math.random(); 
    session.cookie = {
      expire: (new Date()).getTime() + EXPIRES 
    };
    sessions[session.id] = session;
    return session; 
  };
  ```

  每个请求到来时，检查Cookie中的口令与服务器端的数据，如果过期，就重新生成

  ```javascript
  function (req, res) {
    var id = req.cookies[key]; 
    if (!id) {
      req.session = generate(); 
    } else {
      var session = sessions[id]; 
      if (session) {
        if (session.cookie.expire > (new Date()).getTime()) {
          // 更新超时时间
          session.cookie.expire = (new Date()).getTime() + EXPIRES; 
          req.session = session;
        } else {
          // 超时了，删除旧的数据，并重新生成 
          delete sessions[id];
          req.session = generate();
        }
      } else {
        // 如果session过期或口令不对，重新生成session
        req.session = generate(); 
      }
    }
    handle(req, res); 
  }
  ```

  仅仅重新生成Session还不足以完成整个流程，还需要在响应给客户端时设置新的值，以便下次请求时能够对应服务器端的数据。

  hack响应对象的writeHead()方法，在它的内部 注入设置Cookie的逻辑：

  ```javascript
  var writeHead = res.writeHead; 
  res.writeHead = function () {
    var cookies = res.getHeader('Set-Cookie');
    var session = serialize(key, req.session.id);
    cookies = Array.isArray(cookies) ?cookies.concat(session) : [cookies, session];
    res.setHeader('Set-Cookie', cookies);
    return writeHead.apply(this, arguments);
  };
  ```

* 第二种:通过查询字符串来实现浏览器端和服务器端数据的对应

  它的原理是检查请求的查询字符串，如果没有值，会先生成新的带值的URL

  ```javascript
  var getURL = function (_url, key, value) { 
    var obj = url.parse(_url, true); 
    obj.query[key] = value;
    return url.format(obj);
  };
  ```

  然后形成跳转，让客户端重新发起请求

  ```javascript
  function (req, res) {
    var redirect = function (url) {
      res.setHeader('Location', url);
      res.writeHead(302);
      res.end();
    };
    var id = req.query[key]; 
    if (!id) {
      var session = generate();
      redirect(getURL(req.url, key, session.id)); 
    } else {
      var session = sessions[id]; 
      if (session) {
        if (session.cookie.expire > (new Date()).getTime()) {
          // 更新超时时间
          session.cookie.expire = (new Date()).getTime() + EXPIRES; 
          req.session = session;
          handle(req, res); 
        } else {
          // 超时了，删除旧的数据，并重新生成
          delete sessions[id];
          var session = generate();
          redirect(getURL(req.url, key, session.id));
        }
      } else {
        // 如果session过期或口令不对，重新生成session 
        var session = generate();
        redirect(getURL(req.url, key, session.id));
      } 
    }
  }
  ```

  用户访问http://localhost/pathname时，如果服务器端发现查询字符串中不带session_id参数， 就会将用户跳转到http://localhost/pathname?session_id=12344567这样一个类似的地址。如果浏览 器收到302状态码和Location报头，就会重新发起新的请求

  ```javascript
  < HTTP/1.1 302 Moved Temporarily
  < Location: /pathname?session_id=12344567
  ```

  这样，新的请求到来时就能通过Session的检查，除非内存中的数据过期。

  有的服务器在客户端禁用Cookie时，会采用这种方案实现退化。通过这种方案，无须在响应 时设置Cookie。但是这种方案带来的风险远大于基于Cookie实现的风险，因为只要将地址栏中的 地址发给另外一个人，那么他就拥有跟你相同的身份。Cookie的方案在换了浏览器或者换了电脑 之后无法生效，相对较为安全

##### Session与内存

用户请求的连接将可能随意分配到各个进程中，Node的进程与进程之间是不能直接共享内存 的，用户的Session可能会引起错乱。

为了解决性能问题和Session数据无法跨进程共享的问题，常用的方案是将Session集中化，将 原本可能分散在多个进程里的数据，统一转移到集中的数据存储中。

目前常用的工具是Redis、 Memcached等，通过这些高效的缓存，Node进程无须在内部维护数据对象，垃圾回收问题和内存 限制问题都可以迎刃而解，并且这些高速缓存设计的缓存过期策略更合理更高效，比在Node中自 行设计缓存策略更好。

采用第三方缓存来存储Session引起的一个问题是会引起网络访问，尽管如此但依然会采用这些高速缓存的理由有以下几条:

* Node与缓存服务保持长连接，而非频繁的短连接，握手导致的延迟只影响初始化。
* 高速缓存直接在内存中进行数据存储和访问。
* 缓存服务通常与Node进程运行在相同的机器上或者相同的机房里，网络速度受到的影响较小。

Session异步的方式获取：

```javascript
function (req, res) {
  var id = req.cookies[key]; 
  if (!id) {
    req.session = generate();
    handle(req, res); 
  } else {
    store.get(id, function (err, session) { 
      if (session) {
        if (session.cookie.expire > (new Date()).getTime()) {
          // 更新超时时间
          session.cookie.expire = (new Date()).getTime() + EXPIRES; 
          req.session = session;
        } else {
          // 超时了，删除旧的数据，并重新生成 
          delete sessions[id];
          req.session = generate();
        }
      } else {
        // 如果session过期或口令不对，重新生成session
        req.session = generate(); 
      }
      handle(req, res); 
    });
  } 
}
```

在响应时，将新的session保存回缓存中

```javascript
var writeHead = res.writeHead; 
res.writeHead = function () {
  var cookies = res.getHeader('Set-Cookie');
  var session = serialize('Set-Cookie', req.session.id);
  cookies = Array.isArray(cookies) ? cookies.concat(session) : [cookies, session];
  res.setHeader('Set-Cookie', cookies);
  // 保存回缓存
  store.save(req.session);
  return writeHead.apply(this, arguments);
};
```

##### Session与安全

Session的安全，就 主要指如何让这个口令更加安全

有一种做法是将这个口令通过私钥加密进行签名，使得伪造的成本较高。客户端尽管可以伪 造口令值，但是由于不知道私钥值，签名信息很难伪造。如此，我们只要在响应时将口令和签名 进行对比，如果签名非法，我们将服务器端的数据立即过期即可

```javascript
// 将值通过私钥签名，由.分割原值和签名 
var sign = function (val, secret) {
  return val + '.' + crypto 
    .createHmac('sha256', secret) 
    .update(val) 
    .digest('base64') 
    .replace(/\=+$/, '');
};
```

在响应时，设置session值到Cookie中或者跳转URL中

```javascript
var val = sign(req.sessionID, secret); res.setHeader('Set-Cookie', cookie.serialize(key, val));
```

接收请求时，检查签名

```javascript
// 取出口令部分进行签名，对比用户提交的值 
var unsign = function (val, secret) {
  var str = val.slice(0,val.lastIndexOf('.'));
  return sign(str, secret) == val ? str : false; 
};
```

这样一来，即使攻击者知道口令中.号前的值是服务器端Session的ID值，只要不知道secret 私钥的值，就无法伪造签名信息，以此实现对Session的保护。该方法被Connect中间件框架所使 用，保护好私钥，就是在保障自己Web应用的安全。

将客户端的某些独有信息与口令作为原值， 然后签名，这样攻击者一旦不在原始的客户端上进行访问，就会导致签名失败。这些独有信息包 9 括用户IP和用户代理(User Agent)。

* XSS漏洞

  XSS的全称是跨站脚本攻击(Cross Site Scripting，通常简称为XSS)。XSS漏洞会让别的脚本执行。它的主要形成原因多数是 用户的输入没有被转义，而被直接执行

#### 缓存

*  添加Expires 或Cache-Control 到报文头中。
*  配置 ETags。
*  让Ajax 可缓存。

POST、DELETE、PUT这类带行为性的请求操作一般不做任何缓存，大多数缓存只应用在GET请求中。

![image-20200103155111525](https://tva1.sinaimg.cn/large/006tNbRwgy1gajfb4b6e0j30i80pi0uq.jpg)

本地没有文件时，浏览器必然会请求服务器端的内容，并将这部分内容放置在本 地的某个缓存目录中。在第二次请求时，它将对本地文件进行检查，如果不能确定这份本地文件 是否可以直接使用，它将会发起一次条件请求。所谓条件请求，就是在普通的GET请求报文中， 附带If-Modified-Since字段

```
If-Modified-Since: Sun, 03 Feb 2013 06:01:12 GMT
```

它将询问服务器端是否有更新的版本，本地文件的最后修改时间。如果服务器端没有新的版 本，只需响应一个304状态码，客户端就使用本地版本。如果服务器端有新的版本，就将新的内 容发送给客户端，客户端放弃本地版本。

```javascript
var handle = function (req, res) { 
  fs.stat(filename, function (err, stat) {
    var lastModified = stat.mtime.toUTCString();
    if (lastModified === req.headers['if-modified-since']) {
      res.writeHead(304, "Not Modified");
      res.end(); 
    } else {
      fs.readFile(filename, function(err, file) {
        var lastModified = stat.mtime.toUTCString();
        res.setHeader("Last-Modified", lastModified);
        res.writeHead(200, "Ok");
        res.end(file);
      }); 
    }
  }); 
};
```

这里的条件请求采用时间戳的方式实现，但是时间戳有一些缺陷存在。

* 文件的时间戳改动但内容并不一定改动。
* 时间戳只能精确到秒级别，更新频繁的内容将无法生效。

为此HTTP1.1中引入了ETag来解决这个问题。ETag的全称是Entity Tag，由服务器端生成，服 务器端可以决定它的生成规则。如果根据文件内容生成散列值，那么条件请求将不会受到时间戳 改动造成的带宽浪费。

根据内容生成散列值的方法:

```javascript
var getHash = function (str) {
  var shasum = crypto.createHash('sha1'); 
  return shasum.update(str).digest('base64');
};
```

与If-Modified-Since/Last-Modified不同的是，ETag的请求和响应是If-None-Match/ETag

```javascript
var handle = function (req, res) {
  fs.readFile(filename, function(err, file) {
    var hash = getHash(file);
    var noneMatch = req.headers['if-none-match'];
    if (hash === noneMatch) {
      res.writeHead(304, "Not Modified");
      res.end(); 
    } else {
      res.setHeader("ETag", hash); res.writeHead(200, "Ok"); 
      res.end(file);
    } 
  });
};
```

HTTP1.0时，在服务器端设置Expires可以告知浏览器要缓存文件内容

```javascript
var handle = function (req, res) {
  fs.readFile(filename, function(err, file) {
    var expires = new Date();
    expires.setTime(expires.getTime() + 10 * 365 * 24 * 60 * 60 * 1000); 
    res.setHeader("Expires", expires.toUTCString());
    res.writeHead(200, "Ok");
    res.end(file);
  }); 
};
```

Expires是一个GMT格式的时间字符串。浏览器在接到这个过期值后，只要本地还存在这个 缓存文件，在到期时间之前它都不会再发起请求

Cache-Control以更丰富的形式

```javascript
var handle = function (req, res) {
  fs.readFile(filename, function(err, file) {
    res.setHeader("Cache-Control", "max-age=" + 10 * 365 * 24 * 60 * 60 * 1000); 
    res.writeHead(200, "Ok");
    res.end(file);
  }); 
};
```

Cache-Control 能够避免浏览器端与服务器端时间不同步带来的不一致性问题，只要进行类似倒计时的方式计算 过期时间即可。除此之外，Cache-Control的值还能设置public、private、no-cache、no-store 等能够更精细地控制缓存的选项。

由于在HTTP1.0时还不支持max-age，如今的服务器端在模块的支持下多半同时对Expires和 Cache-Control进行支持。在浏览器中如果两个值同时存在，且被同时支持时，max-age会覆盖 Expires。

##### 清除缓存

缓存一旦设定，当服务器端意外更新内容时，却无法通知客户端更新。这使得我们在使用缓存时也要为其设定版本号，所幸浏览器是根据URL进行缓存，那么一旦内容有所更新时，我们就让浏览器发起新的URL请求， 使得新内容能够被客户端更新。

更新机制:

* 每次发布，路径中跟随Web应用的版本号:http://url.com/?v=20130501。
* 每次发布，路径中跟随该文件内容的hash值:http://url.com/?hash=afadfadwe。

根据文件内容的hash值进行缓存淘汰会更加高效，因为文件内容不一定随着Web 应用的版本而更新，而内容没有更新时，版本号的改动导致的更新毫无意义，因此以文件内容形 成的hash值更精准。

#### Basic 认证

Basic认证是当客户端与服务器端进行请求时，允许通过用户名和密码实现的一种身份认证方式。

* 如果一个页面需要Basic认证，它会检查请求报文头中的Authorization字段的内容，该字段 的值由认证方式和加密值构成

```javascript
$ curl -v "http://user:pass@www.baidu.com/"
> GET / HTTP/1.1
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/7.24.0 (x86_64-apple-darwin12.0) libcurl/7.24.0 OpenSSL/0.9.8r zlib/1.2.5
> Host: www.baidu.com
> Accept: */*
```

* 在Basic认证中，它会将用户和密码部分组合:username + ":" + password。然后进行Base64编码

```javascript
var encode = function (username, password) {
  return new Buffer(username + ':' + password).toString('base64');
};
```

* 如果用户首次访问该网页，URL地址中也没携带认证内容，那么浏览器会响应一个401未授 权的状态码

```javascript
function (req, res) {
  var auth = req.headers['authorization'] || '';
  var parts = auth.split(' ');
  var method = parts[0] || ''; // Basic
  var encoded = parts[1] || ''; // dXNlcjpwYXNz
  var decoded = new Buffer(encoded, 'base64').toString('utf-8').split(":"); var user = decoded[0]; // user
  var pass = decoded[1]; // pass
  if (!checkUser(user, pass)) {
    res.setHeader('WWW-Authenticate', 'Basic realm="Secure Area"');//响应头中的WWW-Authenticate字段告知浏览器采用什么样的认证和加密方式
    res.writeHead(401);
    res.end();
  } else { 
    handle(req, res);
  } 
}
```

* 当认证通过，服务器端响应200状态码之后，浏览器会保存用户名和密码口令，在后续的请 求中都携带上Authorization信息。

Basic认证有太多的缺点，它虽然经过Base64加密后在网络中传送，但是这近乎于明文，十分 危险，一般只有在HTTPS的情况下才会使用。不过Basic认证的支持范围十分广泛，几乎所有的 浏览器都支持它。

### 数据上传

如果请求中还带 4 有内容部分(如POST请求，它具有报头和内容)，内容部分需要用户自行接收和解析。通过报头 的Transfer-Encoding或Content-Length即可判断请求中是否带有内容

```javascript
var hasBody = function(req) {
  return 'transfer-encoding' in req.headers || 'content-length' in req.headers;
};
```

在HTTP_Parser解析报头结束后，报文内容部分会通过data事件触发，我们只需以流的方式处理即可

```javascript
function (req, res) { 
  if (hasBody(req)) { 
    var buffers = [];
    req.on('data', function (chunk) { 
      buffers.push(chunk);
    });
    req.on('end', function () {
      req.rawBody = Buffer.concat(buffers).toString();
      handle(req, res);
    }); 
  } else { 
    handle(req, res);
  } 
}
```

将接收到的Buffer列表转化为一个Buffer对象后，再转换为没有乱码的字符串，暂时挂置在 req.rawBody处。

#### 表单数据

默认的表单提交，请求头中的Content-Type字段值为application/x-www-form-urlencoded

```http
Content-Type: application/x-www-form-urlencoded
```

由于它的报文体内容跟查询字符串相同:`foo=bar&baz=val`

解析它十分容易

```javascript
var handle = function (req, res) {
  if (req.headers['content-type'] === 'application/x-www-form-urlencoded') {
    req.body = querystring.parse(req.rawBody); 
  }
  todo(req, res); 
};
```

后续业务中直接访问req.body就可以得到表单中提交的数据

#### 其他格式

常见的提交还有JSON和XML文件,依据Content-Type中的值决定，其中JSON类型的值为application/json，XML的值为 application/xml

需要注意的是，在Content-Type中可能还附带如下所示的编码信息`Content-Type: application/json; charset=utf-8`所以在做判断时，需要注意区分

```javascript
var mime = function (req) {
  var str = req.headers['content-type'] || ''; 
  return str.split(';')[0];
};
```



##### 1. JSON文件

如果从客户端提交JSON内容，这对于Node来说，要处理它都不需要额外的任何库

```javascript
var handle = function (req, res) {
  if (mime(req) === 'application/json') {
    try {
      req.body = JSON.parse(req.rawBody);
    } catch (e) {
      // 异常内容，响应Bad request 
      res.writeHead(400); 
      res.end('Invalid JSON'); 
      return;
    } 
  }
  todo(req, res); 
};
```

##### 2. XML文件

解析XML文件稍微复杂一点，但是社区有支持XML文件到JSON对象转换的库

```javascript
var xml2js = require('xml2js');
var handle = function (req, res) {
  if (mime(req) === 'application/xml') {
    xml2js.parseString(req.rawBody, function (err, xml) { 
      if (err) {
        // 异常内容，响应Bad request 
        res.writeHead(400); 
        res.end('Invalid XML'); 
        return;
      }
      req.body = xml; todo(req, res);
    }); 
  }
};
```

#### 附件上传

通常的表单，其内容 可以通过urlencoded的方式编码内容形成报文体，再发送给服务器端，但是业务场景往往需要用 户直接提交文件。在前端HTML代码中，特殊表单与普通表单的差异在于该表单中可以含有file 类型的控件，以及需要指定表单属性enctype为multipart/form-data

```html
<form action="/upload" method="post" enctype="multipart/form-data">
  <label for="username">Username:</label> 
  <input type="text" name="username" id="username" /> 
  <label for="file">Filename:</label> 
  <input type="file" name="file" id="file" />
  <br />
  <input type="submit" name="submit" value="Submit" />
</form>
```

浏览器在遇到multipart/form-data表单提交时，构造的请求报文与普通表单完全不同。首先它的报头中最为特殊的

```http
Content-Type: multipart/form-data; boundary=AaB03x 
Content-Length: 18231
```

它代表本次提交的内容是由多部分构成的，其中boundary=AaB03x指定的是每部分内容的分界 符，AaB03x是随机生成的一段字符串，报文体的内容将通过在它前面添加--进行分割，报文结束 时在它前后都加上--表示结束。另外，Content-Length的值必须确保是报文体的长度。

假设上面的表单选择了一个名为diveintonode.js的文件，并进行提交上传，那么生成的报文如 下所示:

```http
--AaB03x\r\n
Content-Disposition: form-data; name="username"\r\n
\r\n
Jackson Tian\r\n
--AaB03x\r\n
Content-Disposition: form-data; name="file"; filename="diveintonode.js"\r\n Content-Type: application/javascript\r\n
\r\n
... contents of diveintonode.js ... 
--AaB03x--
```

接收 大小未知的数据量时，我们需要十分谨慎

```javascript
function (req, res) { 
  if (hasBody(req)) {
    var done = function () { 
      handle(req, res);
    };
    if (mime(req) === 'application/json') {
      parseJSON(req, done);
    } else if (mime(req) === 'application/xml') {
      parseXML(req, done);
    } else if (mime(req) === 'multipart/form-data') {
      parseMultipart(req, done); }
  } else { 
    handle(req, res);
  } 
}
```

这里我们将req这个流对象直接交给对应的解析方法，由解析方法自行处理上传的内容，或接收内容并保存在内存中，或流式处理掉。

formidable模块，基于流式处理解析报文，将接收到的文件写入到系统 的临时文件夹中，并返回对应的路径。

```javascript
var formidable = require('formidable');
function (req, res) { 
  if (hasBody(req)) {
    if (mime(req) === 'multipart/form-data') {
      var form = new formidable.IncomingForm(); 
      form.parse(req, function(err, fields, files) {
        req.body = fields; 
        req.files = files; 
        handle(req, res);
      }); 
    }
  } else { 
    handle(req, res);
  } 
}
```

因此在业务逻辑中只要检查req.body和req.files中的内容即可

#### 数据上传与安全

##### 1. 内存限制

在解析表单、JSON和XML部分，我们采取的策略是先保存用户提交的所有数据，然后再解 析处理，最后才传递给业务逻辑。这种策略存在潜在的问题是，它仅仅适合数据量小的提交请求， 一旦数据量过大，将发生内存被占光的情况。攻击者通过客户端能够十分容易地模拟伪造大量数 据，如果攻击者每次提交1 MB的内容，那么只要并发请求数量一大，内存就会很快地被吃光。

要解决这个问题主要有两个方案。

* 限制上传内容的大小，一旦超过限制，停止接收数据，并响应400状态码。 
* 通过流式解析，将数据流导向到磁盘中，Node只保留文件路径等小数据。

```javascript
var bytes = 1024;
function (req, res) {
  var received = 0,
  var len = req.headers['content-length'] ? parseInt(req.headers['content-length'], 10) : null;
  // 如果内容超过长度限制，返回请求实体过长的状态码 
  if (len && len > bytes) {
    res.writeHead(413); 
    res.end();
    return;
  }
  // limit
  req.on('data', function (chunk) {
    received += chunk.length; 
    if (received > bytes) {
      // 停止接收数据，触发end()
      req.destroy(); 
    }
  });
  handle(req, res); 
};
```

数据是由包含Content-Length的请求报文判断是否长度超过 限制的，超过则直接响应413状态码。对于没有Content-Length的请求报文，略微简略一点，在每 个data事件中判定即可。一旦超过限制值，服务器停止接收新的数据片段。如果是JSON文件或 XML文件，极有可能无法完成解析。对于上线的Web应用，添加一个上传大小限制十分有利于保 护服务器，在遭遇攻击时，能镇定从容应对。

##### 2. CSRF

用户通过POST提交content字段就能成功留言。服务器端会自动从Session数据中判断是谁提交的数据，补足username和updatedAt两个字段后向数据库中写入数据。攻击者只要引诱某个domain_a的登录用户访问这个domain_b的网站，就会自动 提交一个留言。由于在提交到domain_a的过程中，浏览器会将domain_a的Cookie发送到服务器， 尽管这个请求是来自domain_b的，但是服务器并不知情，用户也不知情。

### 路由解析

#### 文件路径型

##### 1. 静态文件

URL的路径与网站 目录的路径一致，无须转换，非常直观。将请求路径对应的文 件发送给客户端即可

##### 2. 动态文件

根据文件路径执行动态脚本也是基本的路由方式，它的处理原 理是Web服务器根据URL路径找到对应的文件，Web服务器根据文件名 后缀去寻找脚本的解析器，并传入HTTP请求的上下文。
（现今大多数的服务器都能很智能 地根据后缀同时服务动态和静态文件。这种方式在Node中不太常见，主要原因是文件的后缀都 是.js，分不清是后端脚本，还是前端脚本，而且Node中Web服务器与应 用业务脚本是一体的，无须按这种方式实现。）

#### MVC

MVC模型的主要思想是**将业务逻辑按职责分离**，主要分为以下几种。 

*  控制器(Controller)，一组行为的集合。
*  模型(Model)，数据相关的操作和封装。
*  视图(View)，视图的渲染。

工作模式：

* 路由解析，根据URL寻找到对应的控制器和行为。
  * 根据URL做路由映射
    * 手工关联映射：有一个对应的路由文件来将URL映射到对应的控制器
    * 自然关联映射：没有映射的文件
* 行为调用相关的模型，进行数据操作。
* 数据操作结束后，调用视图和相关数据进行页面渲染，输出到客户端。

![image-20200701213512319](https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbsshjjicj30m40cmjs8.jpg)

##### 1. 手工映射

手工映射除了需要手工配置路由外较为原始外，它对URL的要求十分灵活，几乎没有格式上 的限制。

```tex
/user/setting
/setting/user
```

这里假设已经拥有了一个处理设置用户信息的控制器

```javascript
exports.setting = function (req, res) { 
  // TODO
};
```

再添加一个映射的方法就行

```javascript
var routes = [];
var use = function (path, action) { 
  routes.push([path, action]);
};
```

我们在入口程序中判断URL，然后执行对应的逻辑，于是就完成了基本的路由映射过程

```javascript
function (req, res) {
  var pathname = url.parse(req.url).pathname; 
  for (var i = 0; i < routes.length; i++) {
    var route = routes[i];
    if (pathname === route[0]) {
      var action = route[1]; 
      action(req, res); 
      return;
    }
  }
  // 处理404请求
  handle404(req, res); 
}
```

* 正则匹配

根据不同的用户显示不同的内容，我们就不太可能去手工维护所有用户的路由请求，因此正则匹配应运而生

```javascript
use('/profile/:username', function (req, res) { 
  // TODO
});
```

在通过use注册路由时需要将路径转换为一个正则表达式，然后通过它来进行匹配

```javascript
var pathRegexp = function(path) { 
  path = path
    .concat(strict ? '' : '/?')
    .replace(/\/\(/g, '(?:/')
    .replace(/(\/)?(\.)?:(\w+)(?:(\(.*?\)))?(\?)?(\*)?/g,function(_, slash, format, key, capture,
optional, star){
    slash = slash || '';
    return ''
      + (optional ? '' : slash)
      + '(?:'
      + (optional ? slash : '')
      + (format || '') + (capture || (format && '([^/.]+?)' || '([^/]+?)')) + ')' 
      + (optional || '')
      + (star ? '(/*)?' : '');
  })
    .replace(/([\/.])/g, '\\$1') 
    .replace(/\*/g, '(.*)');
  return new RegExp('^' + path + '$'); 
}
```

```javascript
var use = function (path, action) {
  routes.push([pathRegexp(path), action]);
};
```

```javascript
function (req, res) {
  var pathname = url.parse(req.url).pathname; 
  for (var i = 0; i < routes.length; i++) {
    var route = routes[i];
    // 正则匹配
    if (route[0].exec(pathname)) {
      var action = route[1]; 
      action(req, res);
      return;
    }
  }
  // 处理404请求
  handle404(req, res);
}
```

* 参数解析

:username到底匹配了啥，还没有解决。为此我们还需要进一步将匹配到的内容抽取出来，希望在业务中能如下这样调用:

```javascript
use('/profile/:username', function (req, res) { 
  var username = req.params.username;
  // TODO
});
```

这里的目标是将抽取的内容设置到req.params处。那么第一步就是将键值抽取出来

```javascript
var pathRegexp = function(path) { 
  var keys = [];
  path = path
    .concat(strict ? '' : '/?')
    .replace(/\/\(/g, '(?:/')
    .replace(/(\/)?(\.)?:(\w+)(?:(\(.*?\)))?(\?)?(\*)?/g,function(_, slash, format, key, capture,
 optional, star){
    // 将匹配到的键值保存起来 
    keys.push(key);
    slash = slash || ''; 
    return ''
      + (optional ? '' : slash)
      + '(?:' 8 + (optional ? slash : '')
        + (format || '') + (capture || (format && '([^/.]+?)' || '([^/]+?)')) + ')'
        + (optional || '')
        + (star ? '(/*)?' : '');
  })
    .replace(/([\/.])/g, '\\$1') 
    .replace(/\*/g, '(.*)');
  return {
    keys: keys,
    regexp: new RegExp('^' + path + '$')
  }; 
}
```

我们将根据抽取的键值和实际的URL得到键值匹配到的实际值，并设置到req.params处

```javascript
function (req, res){
  var pathname = url.parse(req.url).pathname;
  for (var i =0; i < routes.length; i++) {
    var route = routes[i];
    // 正则匹配
    var reg = route[0].regexp;
    var keys = route[0].keys;
    var matched = reg.exec(pathname); 
    if (matched) {
      // 抽取具体值
      var params = {};
      for (var i = 0, l = keys.length; i < l; i++) {
        var value = matched[i + 1]; 
        if (value) {
          params[keys[i]] = value; 
        }
      }
      req.params = params;
      
      var action = route[1]; 
      action(req, res); 
      return;
    } 
  }
  // 处理404请求
  handle404(req, res); 
}
```

至此，我们除了从查询字符串(req.query)或提交数据(req.body)中取到值外，还能从路 径的映射里取到值。

##### 2. 自然映射

实际上并非没有路由，而是路由按一种约定的方式自然而然地实现了路由，而无 须去维护路由映射。

它将如下路径进行了划 分处理:`/controller/action/param1/param2/param3`

以/user/setting/12/1987为例，它会按约定去找controllers目录下的user文件，将其require出来后，调用这个文件模块的setting()方法，而其余的值作为参数直接传递给这个方法。

由于这种自然映射的方式没有指明参数的名称，所以无法采用req.params的方式提取，但是直接通过参数获取更简洁

#### RESTful

REST的全称是Representational State Transfer，中文含义为表现层状态转化。符合REST规范 的设计，我们称为RESTful设计。它的设计哲学主要将服务器端提供的内容实体看作一个资源， 并表现在URL上。

地址代表了一个资源，对这个资源的操作，主要体现在HTTP请求方法上，不是体现在URL上。

过去我们对用户的增删改查或许是如下这样设计URL的:

```tex
POST /user/add?username=jacksontian 
GET /user/remove?username=jacksontian 
POST /user/update?username=jacksontian 
GET /user/get?username=jacksontian
```

在RESTful设计中，它是如 下这样的:

```tex
POST /user/jacksontian 
DELETE /user/jacksontian 
PUT /user/jacksontian 
GET /user/jacksontian
```

**它将DELETE和PUT请求方法引入设计中，参与资源的操作和更改资源的状态。**

在RESTful设计中，**资源的具体格式由请求报头中的Accept字段和服务器端的支持情况来决定**。如果客户端同时接受JSON和XML格式的响应，那么它的Accept字段值是如下这样的:

```tex
Accept: application/json,application/xml
```

靠谱的服务器端应该要顾及这个字段，然后根据自己能响应的格式做出响应。在响应报文中， 通过Content-Type字段告知客户端是什么格式

```tex
Content-Type: application/json
```

**通过URL设计资源、请求方法 定义资源的操作，通过Accept决定资源的表现形式**

##### 1. 请求方法

为了让Node能够支持RESTful需求，我们改进了我们的设计。在RESTful的场景下，我们需要区分请求方法设计。

```javascript
var routes = {'all': []};
var app = {};
app.use = function (path, action) {
  routes.all.push([pathRegexp(path), action]); 
};

['get', 'put', 'delete', 'post'].forEach(function (method) {
  routes[method] = [];
  app[method] = function (path, action) {
    routes[method].push([pathRegexp(path), action]); 
  };
});
```

上面的代码添加了get()、put()、delete()、post()4个方法后，我们希望通过如下的方式完成路由映射:

```javascript
// 增加用户
app.post('/user/:username', addUser);
// 删除用户
app.delete('/user/:username', removeUser); 
// 修改用户
app.put('/user/:username', updateUser); 
// 查询用户
app.get('/user/:username', getUser);
```

将匹配 的部分抽取为match()方法

```javascript
var match = function (pathname, routes) { 
  for (var i = 0; i < routes.length; i++) {
    var route = routes[i];
    // 正则匹配
    var reg = route[0].regexp;
    var keys = route[0].keys;
    var matched = reg.exec(pathname); 
    if (matched) {
      // 抽取具体值
      var params = {};
      for (var i = 0, l = keys.length; i < l; i++) {
        var value = matched[i + 1]; 
        if (value) {
          params[keys[i]] = value; 
        }
      }
      req.params = params;
      var action = route[1]; 
      action(req, res); 
      return true;
    }
  }
  return false; 
};
```

改进分发部分

```javascript
function (req, res) {
  var pathname = url.parse(req.url).pathname; // 将请求方法变为小写
  var method = req.method.toLowerCase();
  if (routes.hasOwnPerperty(method)) {
    // 根据请求方法分发
    if (match(pathname, routes[method])) {
      return;
    } else {
      // 如果路径没有匹配成功，尝试让all()来处理 
      if (match(pathname, routes.all)) {
        return;
      }
    }
  } else {
    // 直接让all()来处理
    if (match(pathname, routes.all)) {
      return;
    }
  }
  // 处理404请求 
  handle404(req, res);
}
```

### 中间件

引入中间件(middleware)来简 化和隔离这些基础设施与业务逻辑之间的细节，让开发者能够关注在业务的开发上，以达到提升 开发效率的目的。

* 在最早的中间件的定义中，它是一种在操作系统上为应用软件提供服务的计算机软件。它既 不是操作系统的一部分，也不是应用软件的一部分，它处于操作系统与应用软件之间，让应用软 件更好、更方便地使用底层服务。
* 如今中间件的含义借指了这种封装底层细节，为上层提供更方 便服务的意义，并非限定在操作系统层面。

中间件的行为比较类似Java中过滤器(filter)的工作原理，就是在进入具体的业务处理之前， 先让过滤器处理。

![image-20200701221214769](https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbtuybqsbj30zk0n0whd.jpg)

#### 异常处理

next()方法添加err参数，并捕获中间件直接抛出的同步异常

```javascript
var handle = function (req, res, stack) { 
  var next = function (err) {
    if (err) {
      return handle500(err, req, res, stack);
    }
    // 从stack数组中取出中间件并执行 
    var middleware = stack.shift(); 
    if (middleware) {
      // 传入next()函数自身，使中间件能够执行结束后递归
      try {
        middleware(req, res, next);
      } catch (ex) {
        next(err);
      }
    }
  };
  // 启动执行
  next();
};
```

异步方法的异常不能直接捕获，中间件异步产生的异常需要自己 传递出来

```javascript
var session = function (req, res, next) {
  var id = req.cookies.sessionid; 
  store.get(id, function (err, session) {
    if (err) {
      // 将异常通过next()传递
      return next(err);
    }
    req.session = session;
    next(); 
  });
 };
```

Next()方法接到异常对象后，会将其交给handle500()进行处理。为了将中间件的思想延续下去，我们认为进行异常处理的中间件也是能进行数组式处理的。由于要同时传递异常，所以用于 处理异常的中间件的设计与普通中间件略有差别，它的参数有4个

```javascript
var middleware = function (err, req, res, next) { 
  // TODO
  next();
};
```

为了区分普通中间件和异常处理中间件，handle500()方法将会对中间件按参数进行选取，然后递归执行

```javascript
var handle500 = function (err, req, res, stack) { 
  // 选取异常处理中间件
  stack = stack.filter(function (middleware) {
    return middleware.length === 4; 
  });
  var next = function () {
    // 从stack数组中取出中间件并执行
    var middleware = stack.shift();
    if (middleware) {
      // 传递异常对象
      middleware(err, req, res, next);
    }
  };
  // 启动执行
  next();
};
```

#### 中间件与性能

为了让业务逻辑提早执行，尽早响应给终端用户，中间件的编写和使用是需要一番考究的。两个主要的能提升的点。

* 编写高效的中间件。
* 合理利用路由，避免不必要的中间件执行。

##### 1. 编写高效的中间件

编写高效的中间件其实就是提升单个处理单元的处理速度，以尽早调用next()执行后续逻 辑。需要知道的事情是，一旦中间件被匹配，那么每个请求都会使该中间件执行一次，哪怕它只 浪费1毫秒的执行时间，都会让我们的QPS显著下降。常见的优化方法有几种。

* 使用高效的方法。必要时通过jsperf.com测试基准性能。
* 缓存需要重复计算的结果(需要控制缓存用量，原因在第5章阐述过)。
* 避免不必要的计算。比如HTTP报文体的解析，对于GET方法完全不需要。

##### 2. 合理使用路由

合理的路由使得不必要的 中间件不参与请求处理的过程。

### 页面渲染

#### 内容响应

##### 1. MIME

浏览器正是通过不同的 Content-Type的值来决定采用不同的渲染方式，这个值我们简称为MIME值。

MIME的全称是Multipurpose Internet Mail Extensions。

不同的文件类型具有不同的MIME值，如JSON文件的值为 application/json、XML文件的值为application/xml、PDF文件的值为application/pdf。

##### 2. 附件下载

Content- Disposition字段影响的行为是客户端会根据它的值判断是应该将报文数据当做即时浏览的内 容，还是可下载的附件。当内容只需即时查看时，它的值为inline，当数据可以存为附件时，它 的值为attachment。另外，Content-Disposition字段还能通过参数指定保存时应该使用的文件名。

```http
Content-Disposition: attachment; filename="filename.ext"
```

##### 3. 响应JSON

为了快捷地响应JSON数据，我们也可以如下这样进行封装:

```javascript
res.json = function (json) { 
  res.setHeader('Content-Type', 'application/json');
  res.writeHead(200);
  res.end(JSON.stringify(json));
};
```

##### 4. 响应跳转

当我们的URL因为某些问题(譬如权限限制)不能处理当前请求，需要将用户跳转到别的 URL时，我们也可以封装出一个快捷的方法实现跳转

```javascript
res.redirect = function (url) {
  res.setHeader('Location', url);
  res.writeHead(302);
  res.end('Redirect to ' + url);
};
```

#### 视图渲染

Web应用最终呈现 在界面上的内容，都是通过一系列的视图渲染呈现出来的。在动态页面技术中，最终的视图是由 模板和数据共同生成出来的。

模板是带有特殊标签的HTML片段，通过与数据的渲染，将数据填充到这些特殊标签中，最 后生成普通的带数据的HTML片段。通常我们将渲染方法设计为render()，参数就是模板路径和 数据

```javascript
res.render = function (view, data) {
  res.setHeader('Content-Type', 'text/html');
  res.writeHead(200);
  // 实际渲染 
  var html = render(view, data);
  res.end(html);
};
```

#### 模板

形成模板技术的4个要素：

* 模板语言。
* 包含模板语言的模板文件。 
* 拥有动态数据的数据对象。 
* 模板引擎。

模板技术并不是什么神秘的技术，它干的实际上是拼接字符串这样很底层的活，只是各种 模板有着各自的优缺点和技巧。

例如，<%=%>就是我们制定的模板标签，`Hello <%= username%>`如果我们的数据是{username: "JacksonTian"}，那么我们期望的结果就是Hello JacksonTian。**具体实现的过程是模板分为Hello和<%= username%>两个部分，前者为普通字符串，后者是表达式。 表达式需要继续处理，与数据关联后成为一个变量值，最终将字符串与变量值连成最终的字符串。**

![image-20200701230518087](https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbve5ww57j30gy0eagmo.jpg)

##### 1. 模版引擎

这个模板引擎会将Hello <%= username%>转换为"Hello " + obj.username。该过程进行以下几个步骤。

* 语法分解。提取出普通字符串和表达式，这个过程通常用正则表达式匹配出来，<%=%>的正则表达式为/<%=([\s\S]+?)%>/g。
* 处理表达式。将标签表达式转换成普通的语言表达式。
* 生成待执行的语句。
* 与数据一起执行，生成最终字符串。

```javascript
var render = function (str, data) {
  // 模板技术呢，就是替换特殊标签的技术
  var tpl = str.replace(/<%=([\s\S]+?)%>/g, function(match, code) {
    return "' + obj." + code + "+ '"; 
  });
  tpl = "var tpl = '" + tpl + "'\nreturn tpl;";
  var complied = new Function('obj', tpl);
  return complied(data);
};
```

###### **模板编译**

为了能够最终与数据一起执行生成字符串，我们需要将原始的模板字符串转换成一个函数对象。这个过程称为模板编译，生成的中间函数只与模板字符串相关，与具体的数据无关。如果每 次都生成这个中间函数，就会浪费CPU。为了提升模板渲染的性能速度，我们通常会采用模板预编译的方式。

```javascript
var complie = function (str) {
  var tpl = str.replace(/<%=([\s\S]+?)%>/g, function(match, code) {
    return "' + obj." + code + "+ '";
  });
  tpl = "var tpl = '" + tpl + "'\nreturn tpl;";
  return new Function('obj, escape', tpl);
};

var render = function (complied, data) { 
  return complied(data);
};
```

通过预编译缓存模板编译后的结果，实际应用中就可以实现一次编译，多次执行。

##### 2. with的应用

上面实现的模板引擎非常弱，只能替换变量，<%="Jackson Tian"%>就无法支持了。为了让它 更灵活，我们需要改进它的实现，使字符串能继续表达为字符串，变量能够自动寻找属于它的对 象。于是with关键字引入到我们的实现中。

```javascript
var complie = function (str, data) {
  // 模板技术呢，就是替换特殊标签的技术
  var tpl = str.replace(/<%=([\s\S]+?)%>/g, function (match, code) {
    return "' + " + code + "+ '"; 
  });
  
  tpl = "tpl = '" + tpl + "'";
  tpl = 'var tpl = "";\nwith (obj) {' + tpl + '}\nreturn tpl;'; 
  return new Function('obj', tpl);
};
```

###### 模板安全

为了提高安全性，大多数模板都提供了转义的功能。转义就是将能形成HTML 标签的字符转换成安全的字符，这些字符主要有&、<、>、"、'。

转义函数如下

```javascript
var escape = function (html) { 
  return String(html)
    .replace(/&(?!\w+;)/g, '&amp;')
    .replace(/</g, '&lt;') 
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;'); // IE下不支持&apos;(单引号)转义
};
```

不确定要输出HTML标签的字符最好都转义

```javascript
var render = function (str, data) {
  var tpl = str.replace(/\n/g, '\\n') // 将换行符替换
  .replace(/<%=([\s\S]+?)%>/g, function (match, code) {
    // 转义
    return "' + escape(" + code + ") + '";
  }).replace(/<%-([\s\S]+?)%>/g, function (match, code) {
    // 正常输出
    return "' + " + code + "+ '";
  });
  tpl = "tpl = '" + tpl + "'";
  tpl = 'var tpl = "";\nwith (obj) {' + tpl + '}\nreturn tpl;';
  // 加上escape()函数
  return new Function('obj', 'escape', tpl);
};
```

模板引擎通过正则分别匹配-和=并区别对待，最后不要忘记传入escape()函数。最终上面的危险代码会转换为安全的输出

##### 3. 模板逻辑

尽管模板技术已经将业务逻辑与视图部分分离开来，但是视图上还是会存在一些逻辑来控制 页面的最终渲染。为了让上述模板变得强大一点，我们为它添加逻辑代码，使得模板可以像ASP、 PHP那样控制页面渲染。

##### 4. 集成文件系统

前文我们实现的complie()和render()函数已经能够实现将输入的模板字符串进行编译和替 换的功能。如果与前文的HTTP响应对象组合起来处理的话，其缺点有如下几点

* 每次请求需要反复读磁盘上的模板文件。
* 每次请求需要编译。
* 调用烦琐。

我们需要一个更简洁、性能更好的render()函数

```javascript
var cache = {};
var VIEW_FOLDER = '/path/to/wwwroot/views';
res.render = function (viewname, data) {
  if (!cache[viewname]) {  
    var text;
    try {
      text = fs.readFileSync(path.join(VIEW_FOLDER, viewname), 'utf8');
    } catch (e) {
 
      res.writeHead(500, {'Content-Type': 'text/html'});
      res.end('模板文件错误');
      return;
    }
    cache[viewname] = complie(text);
  }
  var complied = cache[viewname];
  res.writeHead(200, {'Content-Type': 'text/html'});
  var html = complied(data);
  res.end(html);
};
```

这个res.render()实现中，虽然有同步读取文件的情况，但是由于采用了缓存，只会在第一次读取的时候造成整个进程的阻塞，一旦缓存生效，将不会反复读取模板文件。其次，缓存之前 已经进行了编译，也不会每次读取都编译

与文件系统集成之后，再引入缓存，可以很好地解决性能问题，接口也大大得到简化。由于 模板文件内容都不太大，也不属于动态改动的，所以使用进程的内存来缓存编译结果，并不会引 起太大的垃圾回收问题。

##### 5. 子模板

有时候模板文件太大，太过复杂，会增加维护上的难度，而且有些模板是可以重用的，这催 生了子模板(Partial View)的产生。子模板可以嵌套在别的模板中，多个模板可以嵌入同一个子 模板中。维护多个子模板比维护完整而复杂的大模板的成本要低很多，很多复杂问题可以降解为 多个小而简单的问题。

这里我们采用include关键字来实现模板的嵌套。

母模板如下:

```html
<ul>
  <% users.forEach(function(user){ %>
    <% include user/show %> 
  <% }) %>
</ul>
```

子模板user/show内容如下:

```html
<li><%=user.name%></li>
```

渲染出来的效果应当跟以下代码渲染出来的效果别无二致:

```html
<ul>
  <% users.forEach(function(user){ %> 
  	<li><%=user.name%></li> 
  <% }) %>
</ul>
```

所以实现子模板的诀窍就是先将include语句进行替换，再进行整体性编译

##### 6. 布局视图

子模板的另一种使用方式就是布局视图 (layout)，布局视图又称母版页，它与子模板的原理相同，但是场景稍有区别。一般而言模板指 定了子模板，那它的子模板就无法进行替换了，子模板被嵌入到多个父模板中属于正常需求，但 是如果在多个父模板中只是嵌入的子视图不同，模板内容却完全一样，也会出现重复。

比如下面两个简单的父模板:

```html
// 模板1 
<ul>
  <% users.forEach(function(user){ %> 
    <% include user/show %>
  <% }) %> 
</ul>
// 模板2 
<ul>
  <% users.forEach(function(user){ %> 
    <% include profile %>
  <% }) %> 
</ul>
```

这些重复的内容主要用来布局，为了能将这些布局模板重用起来，模板技术必须支持布局视 图。支持布局视图之后，布局模板就只有一份，渲染视图时，指定好布局视图就可以了

```javascript
res.render('viewname', { 
  layout: 'layout.html', 
  users: []
});
```

对于布局模板文件，我们设计为将<%- body %>部分替换为我们的子模板

```html
<ul>
	<% users.forEach(function(user){ %> 
 		<%- body %> 
  <% }) %>
</ul>
```

##### 7. 模板性能

模板引擎的优化步骤:

* 缓存模板文件。
* 缓存模板文件编译后的函数。
* 优化模板中的执行表达式

#### Bigpipe

它的提出主要是为了解决重数据页面 的加载速度问题

Bigpipe的解决思路则是将页面分割成多个部分(pagelet)，先向用户输出没有数据的布局(框 架)，将每个部分逐步输出到前端，再最终渲染填充框架，完成整个网页的渲染。这个过程中需 要前端JavaScript的参与，它负责将后续输出的数据渲染到页面上。

Bigpipe是一个需要前后端配合实现的优化技术，这个技术有几个重要的点。 

* 页面布局框架(无数据的)。

* 后端持续性的数据输出。
* 前端渲染

![image-20200702001224073](https://tva1.sinaimg.cn/large/007S8ZIlgy1ggbxbyx0e4j31500f4abp.jpg)

##### 1. 页面布局框架

页面布局框架依然由后端渲染而出

```javascript
var cache = {};
var layout = 'layout.html';
app.get('/profile', function (req, res) { 
  if (!cache[layout]) {
    cache[layout] = fs.readFileSync(path.join(VIEW_FOLDER, layout), 'utf8');
  }
  res.writeHead(200, {'Content-Type': 'text/html'});
  res.write(render(complie(cache[layout])));
  // TODO
});
```

这个布局文件中要引入必要的前端脚本，如jQuery、Underscore等常用库，其次要引入我们 重要的前端脚本，这里的文件名为bigpipe.js。

整体模板文件如下

```html
// layout.html 
<!DOCTYPE html> 
<html>
<head>
  <title>Bigpipe示例</title>
  <script src="jquery.js"></script> 
  <script src="underscore.js"></script> 
  <script src="bigpipe.js"></script>
</head> 
<body>
  <div id="body"></div>
  <script type="text/template" id="tpl_body">
  	<div><%=articles%></div>
  </script>
  <div id="footer"></div>
  <script type="text/template" id="tpl_footer">
  	<div><%=users%></div> 
  </script>
</body> 
</html> 
<script>
  var bigpipe = new Bigpipe(); 
  bigpipe.ready('articles', function (data) {
    $('#body').html(_.render($('#tpl_body').html(), {articles: data})); 
  });
  bigpipe.ready('copyright', function (data) {
    $('#footer').html(_.render($('#tpl_footer').html(), {users: data}));
  }); 
</script>
```

##### 2. 持续数据输出

模板输出后，整个网页的渲染并没有结束，但用户已经可以看到整个页面的大体样子。接下 来我们继续数据输出，与普通的数据输出不同，这里的数据输出之后需要被前端脚本处理，是故 需要对它进行封装处理

##### 3. 前端渲染

前文的bigpipe.ready()和bigpipe.set()是整个前端的渲染机制，前者以一个key注册一个事 件，后者则触发一个事件，以此完成页面的渲染机制。

```javascript
var Bigpipe = function () { 
  this.callbacks = {};
};

Bigpipe.prototype.ready = function (key, callback) { 
  if (!this.callbacks[key]) {
    this.callbacks[key] = [];
  }
  this.callbacks[key].push(callback);
};

Bigpipe.prototype.set = function (key, data) {
  var callbacks = this.callbacks[key] || [];
  for (var i = 0; i < callbacks.length; i++) {
    callbacks[i].call(this, data); 
  }
};
```

Bigpipe将网页布局和数据渲染分离，使得用户在视觉上觉得网页提前渲染好了，其随着数据 输出的过程逐步渲染页面，使得用户能够感知到页面是活的。这远比一开始给出空白页面，然后 在某个时候突然渲染好带给用户的体验更好。Node在这个过程中，其异步特性使得数据的输出能 够并行，数据的输出与数据调用的顺序无关，越早调用完的数据可以越早渲染到页面中，这个特 性使得Bigpipe更趋完美。