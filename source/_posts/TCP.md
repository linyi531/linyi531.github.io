---
title: TCP
date: 2019-08-11 21:15:33
tags:
  - HTTP
  - 浏览器
categories: 浏览器
cover_img: https://i.screenshot.net/o2w7gi5
feature_img: https://i.screenshot.net/o2w7gi5
---

# TCP

- TCP 在网络 OSI 的七层模型中的第四层——Transport 层（第四层的数据叫 Segment）
- IP 在第三层——Network 层（在第三层上的数据叫 Packet）
- ARP 在第二层——Data Link 层（在第二层上的数据叫 Frame）

我们程序的数据首先会打到 TCP 的 Segment 中，然后 TCP 的 Segment 会打到 IP 的 Packet 中，然后再打到以太网 Ethernet 的 Frame 中，传到对端后，各个层解析自己的协议，然后把数据交给更高层的协议处理。

<!-- more -->

## TCP 头格式

![img](https://coolshell.cn/wp-content/uploads/2014/05/TCP-Header-01.jpg)

你需要注意这么几点：

- TCP 的包是没有 IP 地址的，那是 IP 层上的事。但是有源端口和目标端口。
- 一个 TCP 连接需要四个元组来表示是同一个连接（src_ip, src_port, dst_ip, dst_port）准确说是五元组，还有一个是协议。但因为这里只是说 TCP 协议，所以，这里我只说四元组。
- 注意上图中的四个非常重要的东西：
  - **Sequence Number**是包的序号，**用来解决网络包乱序（reordering）问题。**
  - **Acknowledgement Number**就是 ACK——用于确认收到，**用来解决不丢包的问题**。
  - **Window 又叫 Advertised-Window**，也就是著名的滑动窗口（Sliding Window），**用于解决流控的**。
  - **TCP Flag** ，也就是包的类型，**主要是用于操控 TCP 的状态机的**。

## TCP 状态机

其实，**网络上的传输是没有连接的，包括 TCP 也是一样的**。而 TCP 所谓的“连接”，其实只不过是在通讯的双方维护一个“连接状态”，让它看上去好像有连接一样。所以，TCP 的状态变换是非常重要的。

“**TCP 协议的状态机**”和 “**TCP 建链接**”、“**TCP 断链接**”、“**传数据**” 的对照图

<img src="https://coolshell.cn/wp-content/uploads/2014/05/tcpfsm.png" style="zoom:70%;display:inline" /><img src="https://coolshell.cn/wp-content/uploads/2014/05/tcp_open_close.jpg" style="zoom:40%;display:inline" />

很多人会问，为什么建链接要 3 次握手，断链接需要 4 次挥手？

- **对于建链接的 3 次握手，**主要是要初始化 Sequence Number 的初始值。通信的双方要互相通知对方自己的初始化的 Sequence Number（缩写为 ISN：Inital Sequence Number）——所以叫 SYN，全称 Synchronize Sequence Numbers。也就上图中的 x 和 y。这个号要作为以后的数据通信的序号，以保证应用层接收到的数据不会因为网络上的传输的问题而乱序（TCP 会用这个序号来拼接数据）。
- **对于 4 次挥手，**其实你仔细看是 2 次，因为 TCP 是全双工的，所以，发送方和接收方都需要 Fin 和 Ack。只不过，有一方是被动的，所以看上去就成了所谓的 4 次挥手。如果两边同时断连接，那就会就进入到 CLOSING 状态，然后到达 TIME_WAIT 状态。下图是双方同时断连接的示意图（你同样可以对照着 TCP 状态机看）：

## 数据传输中的 Sequence Number

下图是从 Wireshark 中截了个我在访问 coolshell.cn 时的有数据传输的图给你看一下，SeqNum 是怎么变的。（使用 Wireshark 菜单中的 Statistics ->Flow Graph… ）

![img](https://coolshell.cn/wp-content/uploads/2014/05/tcp_data_seq_num.jpg)

你可以看到，**SeqNum 的增加是和传输的字节数相关的**。上图中，三次握手后，来了两个 Len:1440 的包，而第二个包的 SeqNum 就成了 1441。然后第一个 ACK 回的是 1441，表示第一个 1440 收到了。

**注意**：如果你用 Wireshark 抓包程序看 3 次握手，你会发现 SeqNum 总是为 0，不是这样的，Wireshark 为了显示更友好，使用了 Relative SeqNum——相对序号，你只要在右键菜单中的 protocol preference 中取消掉就可以看到“Absolute SeqNum”了

## TCP 重传机制

TCP 要保证所有的数据包都可以到达，所以，必需要有重传机制。

注意，接收端给发送端的 Ack 确认只会确认最后一个连续的包，比如，发送端发了 1,2,3,4,5 一共五份数据，接收端收到了 1，2，于是回 ack 3，然后收到了 4（注意此时 3 没收到），此时的 TCP 会怎么办？我们要知道，因为正如前面所说的，**SeqNum 和 Ack 是以字节数为单位，所以 ack 的时候，不能跳着确认，只能确认最大的连续收到的包**，不然，发送端就以为之前的都收到了。

### 超时重传机制

一种是不回 ack，死等 3，当发送方发现收不到 3 的 ack 超时后，会重传 3。一旦接收方收到 3 后，会 ack 回 4——意味着 3 和 4 都收到了。

但是，这种方式会有比较严重的问题，那就是因为要死等 3，所以会导致 4 和 5 即便已经收到了，而发送方也完全不知道发生了什么事，因为没有收到 Ack，所以，发送方可能会悲观地认为也丢了，所以有可能也会导致 4 和 5 的重传。

对此有两种选择：

- 一种是仅重传 timeout 的包。也就是第 3 份数据。
- 另一种是重传 timeout 后所有的数据，也就是第 3，4，5 这三份数据。

这两种方式有好也有不好。第一种会节省带宽，但是慢，第二种会快一点，但是会浪费带宽，也可能会有无用功。但总体来说都不好。因为都在等 timeout，timeout 可能会很长（在下篇会说 TCP 是怎么动态地计算出 timeout 的）

### 快速重传机制

于是，TCP 引入了一种叫**Fast Retransmit** 的算法，**不以时间驱动，而以数据驱动重传**。也就是说，如果，包没有连续到达，就 ack 最后那个可能被丢了的包，如果发送方连续收到 3 次相同的 ack，就重传。Fast Retransmit 的好处是不用等 timeout 了再重传。

比如：如果发送方发出了 1，2，3，4，5 份数据，第一份先到送了，于是就 ack 回 2，结果 2 因为某些原因没收到，3 到达了，于是还是 ack 回 2，后面的 4 和 5 都到了，但是还是 ack 回 2，因为 2 还是没有收到，于是发送端收到了三个 ack=2 的确认，知道了 2 还没有到，于是就马上重转 2。然后，接收端收到了 2，此时因为 3，4，5 都收到了，于是 ack 回 6。示意图如下：

![img](https://coolshell.cn/wp-content/uploads/2014/05/FASTIncast021.png)

Fast Retransmit 只解决了一个问题，就是 timeout 的问题，它依然面临一个艰难的选择，就是，是重传之前的一个还是重传所有的问题。对于上面的示例来说，是重传#2 呢还是重传#2，#3，#4，#5 呢？因为发送端并不清楚这连续的 3 个 ack(2)是谁传回来的？也许发送端发了 20 份数据，是#6，#10，#20 传来的呢。这样，发送端很有可能要重传从 2 到 20 的这堆数据（这就是某些 TCP 的实际的实现）。可见，这是一把双刃剑。

### SACK 方法

另外一种更好的方式叫：**Selective Acknowledgment (SACK)**（参看[RFC 2018](http://tools.ietf.org/html/rfc2018)），这种方式需要在 TCP 头里加一个 SACK 的东西，ACK 还是 Fast Retransmit 的 ACK，SACK 则是汇报收到的数据碎版。参看下图：

![img](https://coolshell.cn/wp-content/uploads/2014/05/tcp_sack_example-1024x577.jpg)

这样，在发送端就可以根据回传的 SACK 来知道哪些数据到了，哪些没有到。于是就优化了 Fast Retransmit 的算法。当然，这个协议需要两边都支持。在 Linux 下，可以通过**tcp_sack**参数打开这个功能（Linux 2.4 后默认打开）。

这里还需要注意一个问题——**接收方 Reneging，所谓 Reneging 的意思就是接收方有权把已经报给发送端 SACK 里的数据给丢了**。这样干是不被鼓励的，因为这个事会把问题复杂化了，但是，接收方这么做可能会有些极端情况，比如要把内存给别的更重要的东西。**所以，发送方也不能完全依赖 SACK，还是要依赖 ACK，并维护 Time-Out，如果后续的 ACK 没有增长，那么还是要把 SACK 的东西重传，另外，接收端这边永远不能把 SACK 的包标记为 Ack。**

注意：SACK 会消费发送方的资源，试想，如果一个攻击者给数据发送方发一堆 SACK 的选项，这会导致发送方开始要重传甚至遍历已经发出的数据，这会消耗很多发送端的资源。

### Duplicate SACK – 重复收到数据的问题

Duplicate SACK 又称 D-SACK，**其主要使用了 SACK 来告诉发送方有哪些数据被重复接收了**。[RFC-2883 ](http://www.ietf.org/rfc/rfc2883.txt)里有详细描述和示例。下面举几个例子（来源于[RFC-2883](http://www.ietf.org/rfc/rfc2883.txt)）

D-SACK 使用了 SACK 的第一个段来做标志，

- 如果 SACK 的第一个段的范围被 ACK 所覆盖，那么就是 D-SACK
- 如果 SACK 的第一个段的范围被 SACK 的第二个段覆盖，那么就是 D-SACK

#### 示例一：ACK 丢包

下面的示例中，丢了两个 ACK，所以，发送端重传了第一个数据包（3000-3499），于是接收端发现重复收到，于是回了一个 SACK=3000-3500，因为 ACK 都到了 4000 意味着收到了 4000 之前的所有数据，所以这个 SACK 就是 D-SACK——旨在告诉发送端我收到了重复的数据，而且我们的发送端还知道，数据包没有丢，丢的是 ACK 包。

```json
Transmitted  Received    ACK Sent
Segment      Segment     (Including SACK Blocks)
3000-3499    3000-3499   3500 (ACK dropped)
3500-3999    3500-3999   4000 (ACK dropped)
3000-3499    3000-3499   4000, SACK=3000-3500
                                    ---------
```

#### 示例二，网络延误

下面的示例中，网络包（1000-1499）被网络给延误了，导致发送方没有收到 ACK，而后面到达的三个包触发了“Fast Retransmit 算法”，所以重传，但重传时，被延误的包又到了，所以，回了一个 SACK=1000-1500，因为 ACK 已到了 3000，所以，这个 SACK 是 D-SACK——标识收到了重复的包。

这个案例下，发送端知道之前因为“Fast Retransmit 算法”触发的重传不是因为发出去的包丢了，也不是因为回应的 ACK 包丢了，而是因为网络延时了。

```json
Transmitted    Received    ACK Sent
Segment        Segment     (Including SACK Blocks)
500-999        500-999     1000
1000-1499      (delayed)
1500-1999      1500-1999   1000, SACK=1500-2000
2000-2499      2000-2499   1000, SACK=1500-2500
2500-2999      2500-2999   1000, SACK=1500-3000
1000-1499      1000-1499   3000
               1000-1499   3000, SACK=1000-1500
                                      ---------
```

可见，引入了 D-SACK，有这么几个好处：

1）可以让发送方知道，是发出去的包丢了，还是回来的 ACK 包丢了。

2）是不是自己的 timeout 太小了，导致重传。

3）网络上出现了先发的包后到的情况（又称 reordering）

4）网络上是不是把我的数据包给复制了。

**知道这些东西可以很好得帮助 TCP 了解网络情况，从而可以更好的做网络上的流控**。

Linux 下的 tcp_dsack 参数用于开启这个功能（Linux 2.4 后默认打开）

## TCP 的 RTT 算法

从前面的 TCP 重传机制我们知道 Timeout 的设置对于重传非常重要。

- 设长了，重发就慢，丢了老半天才重发，没有效率，性能差；
- 设短了，会导致可能并没有丢就重发。于是重发的就快，会增加网络拥塞，导致更多的超时，更多的超时导致更多的重发。

而且，这个超时时间在不同的网络的情况下，根本没有办法设置一个死的值。只能动态地设置。 为了动态地设置，TCP 引入了 RTT——Round Trip Time，也就是一个数据包从发出去到回来的时间。这样发送端就大约知道需要多少的时间，从而可以方便地设置 Timeout——RTO（Retransmission TimeOut），以让我们的重传机制更高效。 听起来似乎很简单，好像就是在发送端发包时记下 t0，然后接收端再把这个 ack 回来时再记一个 t1，于是 RTT = t1 – t0。没那么简单，这只是一个采样，不能代表普遍情况。

### 经典算法

[RFC793](http://tools.ietf.org/html/rfc793) 中定义的经典算法是这样的：

1）首先，先采样 RTT，记下最近好几次的 RTT 值。

2）然后做平滑计算 SRTT（ Smoothed RTT）。公式为：（其中的 α 取值在 0.8 到 0.9 之间，这个算法英文叫 Exponential weighted moving average，中文叫：加权移动平均）

**SRTT = ( α \* SRTT ) + ((1- α) \* RTT)**

3）开始计算 RTO。公式如下：

**RTO = min [ UBOUND, max [ LBOUND, (β \* SRTT) ] ]**

其中：

- UBOUND 是最大的 timeout 时间，上限值
- LBOUND 是最小的 timeout 时间，下限值
- β 值一般在 1.3 到 2.0 之间。

### Karn / Partridge 算法

但是上面的这个算法在重传的时候会出有一个终极问题——你是用第一次发数据的时间和 ack 回来的时间做 RTT 样本值，还是用重传的时间和 ACK 回来的时间做 RTT 样本值？

这个问题无论你选那头都是按下葫芦起了瓢。 如下图所示：

- 情况（a）是 ack 没回来，所以重传。如果你计算第一次发送和 ACK 的时间，那么，明显算大了。
- 情况（b）是 ack 回来慢了，但是导致了重传，但刚重传不一会儿，之前 ACK 就回来了。如果你是算重传的时间和 ACK 回来的时间的差，就会算短了。

![img](https://coolshell.cn/wp-content/uploads/2014/05/Karn-Partridge-Algorithm.jpg)

所以 1987 年的时候，搞了一个叫[Karn / Partridge Algorithm](http://en.wikipedia.org/wiki/Karn's_Algorithm)，这个算法的最大特点是——**忽略重传，不把重传的 RTT 做采样**（你看，你不需要去解决不存在的问题）。

但是，这样一来，又会引发一个大 BUG——**如果在某一时间，网络闪动，突然变慢了，产生了比较大的延时，这个延时导致要重转所有的包（因为之前的 RTO 很小），于是，因为重转的不算，所以，RTO 就不会被更新，这是一个灾难**。 于是 Karn 算法用了一个取巧的方式——只要一发生重传，就对现有的 RTO 值翻倍（这就是所谓的 Exponential backoff），很明显，这种死规矩对于一个需要估计比较准确的 RTT 也不靠谱。

### Jacobson / Karels 算法

前面两种算法用的都是“加权移动平均”，这种方法最大的毛病就是如果 RTT 有一个大的波动的话，很难被发现，因为被平滑掉了。所以，1988 年，又有人推出来了一个新的算法，这个算法叫 Jacobson / Karels Algorithm（参看[RFC6289](http://tools.ietf.org/html/rfc6298)）。这个算法引入了最新的 RTT 的采样和平滑过的 SRTT 的差距做因子来计算。 公式如下：（其中的 DevRTT 是 Deviation RTT 的意思）

**SRTT** **= S\*\***RTT\*\* **+ α** **(\*\***RTT\*\* **– S\*\***RTT\***\*)** —— 计算平滑 RTT

**DevRTT** **= (1-β\*\***)\*\***\*DevRTT** **+ β\*\*\***(|\***\*RTT-SRTT\*\***|)\*\* ——计算平滑 RTT 和真实的差距（加权移动平均）

**RTO= µ \* SRTT + ∂ \*DevRTT** —— 神一样的公式

（其中：在 Linux 下，α = 0.125，β = 0.25， μ = 1，∂ = 4 ——这就是算法中的“调得一手好参数”，nobody knows why, it just works…） 最后的这个算法在被用在今天的 TCP 协议中（Linux 的源代码在：[tcp_rtt_estimator](http://lxr.free-electrons.com/source/net/ipv4/tcp_input.c?v=2.6.32#L609)）。

## TCP 滑动窗口

需要说明一下，如果你不了解 TCP 的滑动窗口这个事，你等于不了解 TCP 协议。我们都知道，**TCP 必需要解决的可靠传输以及包乱序（reordering）的问题**，所以，TCP 必需要知道网络实际的数据处理带宽或是数据处理速度，这样才不会引起网络拥塞，导致丢包。

所以，TCP 引入了一些技术和设计来做网络流控，Sliding Window 是其中一个技术。 前面我们说过，**TCP 头里有一个字段叫 Window，又叫 Advertised-Window，这个字段是接收端告诉发送端自己还有多少缓冲区可以接收数据**。**于是发送端就可以根据这个接收端的处理能力来发送数据，而不会导致接收端处理不过来**。 为了说明滑动窗口，我们需要先看一下 TCP 缓冲区的一些数据结构：

![img](https://coolshell.cn/wp-content/uploads/2014/05/sliding_window.jpg)

上图中，我们可以看到：

- 接收端 LastByteRead 指向了 TCP 缓冲区中读到的位置，NextByteExpected 指向的地方是收到的连续包的最后一个位置，LastByteRcved 指向的是收到的包的最后一个位置，我们可以看到中间有些数据还没有到达，所以有数据空白区。
- 发送端的 LastByteAcked 指向了被接收端 Ack 过的位置（表示成功发送确认），LastByteSent 表示发出去了，但还没有收到成功确认的 Ack，LastByteWritten 指向的是上层应用正在写的地方。

于是：

- 接收端在给发送端回 ACK 中会汇报自己的 AdvertisedWindow = MaxRcvBuffer – LastByteRcvd – 1;
- 而发送方会根据这个窗口来控制发送数据的大小，以保证接收方可以处理。

下面我们来看一下发送方的滑动窗口示意图：

![img](https://coolshell.cn/wp-content/uploads/2014/05/tcpswwindows.png)

上图中分成了四个部分，分别是：（其中那个黑模型就是滑动窗口）

- \#1 已收到 ack 确认的数据。
- \#2 发还没收到 ack 的。
- \#3 在窗口中还没有发出的（接收方还有空间）。
- \#4 窗口以外的数据（接收方没空间）

下面是个滑动后的示意图（收到 36 的 ack，并发出了 46-51 的字节）：

![img](https://coolshell.cn/wp-content/uploads/2014/05/tcpswslide.png)

下面我们来看一个接受端控制发送端的图示：

![img](https://coolshell.cn/wp-content/uploads/2014/05/tcpswflow.png)

### Zero Window

上图，我们可以看到一个处理缓慢的 Server（接收端）是怎么把 Client（发送端）的 TCP Sliding Window 给降成 0 的。此时，你一定会问，如果 Window 变成 0 了，TCP 会怎么样？是不是发送端就不发数据了？是的，发送端就不发数据了，你可以想像成“Window Closed”，那你一定还会问，如果发送端不发数据了，接收方一会儿 Window size 可用了，怎么通知发送端呢？

解决这个问题，TCP 使用了 Zero Window Probe 技术，缩写为 ZWP，也就是说，发送端在窗口变成 0 后，会发 ZWP 的包给接收方，让接收方来 ack 他的 Window 尺寸，一般这个值会设置成 3 次，第次大约 30-60 秒（不同的实现可能会不一样）。如果 3 次过后还是 0 的话，有的 TCP 实现就会发 RST 把链接断了。

**注意**：只要有等待的地方都可能出现 DDoS 攻击，Zero Window 也不例外，一些攻击者会在和 HTTP 建好链发完 GET 请求后，就把 Window 设置为 0，然后服务端就只能等待进行 ZWP，于是攻击者会并发大量的这样的请求，把服务器端的资源耗尽。（关于这方面的攻击，大家可以移步看一下[Wikipedia 的 SockStress 词条](http://en.wikipedia.org/wiki/Sockstress)）

另外，Wireshark 中，你可以使用 tcp.analysis.zero_window 来过滤包，然后使用右键菜单里的 follow TCP stream，你可以看到 ZeroWindowProbe 及 ZeroWindowProbeAck 的包。

### Silly Window Syndrome

Silly Window Syndrome 翻译成中文就是“糊涂窗口综合症”。正如你上面看到的一样，如果我们的接收方太忙了，来不及取走 Receive Windows 里的数据，那么，就会导致发送方越来越小。到最后，如果接收方腾出几个字节并告诉发送方现在有几个字节的 window，而我们的发送方会义无反顾地发送这几个字节。

要知道，我们的 TCP+IP 头有 40 个字节，为了几个字节，要达上这么大的开销，这太不经济了。

另外，你需要知道网络上有个 MTU，对于以太网来说，MTU 是 1500 字节，除去 TCP+IP 头的 40 个字节，真正的数据传输可以有 1460，这就是所谓的 MSS（Max Segment Size）注意，TCP 的 RFC 定义这个 MSS 的默认值是 536，这是因为 [RFC 791](http://tools.ietf.org/html/rfc791)里说了任何一个 IP 设备都得最少接收 576 尺寸的大小（实际上来说 576 是拨号的网络的 MTU，而 576 减去 IP 头的 20 个字节就是 536）。

**如果你的网络包可以塞满 MTU，那么你可以用满整个带宽，如果不能，那么你就会浪费带宽**。（大于 MTU 的包有两种结局，一种是直接被丢了，另一种是会被重新分块打包发送） 你可以想像成一个 MTU 就相当于一个飞机的最多可以装的人，如果这飞机里满载的话，带宽最高，如果一个飞机只运一个人的话，无疑成本增加了，也而相当二。

所以，**Silly Windows Syndrome 这个现像就像是你本来可以坐 200 人的飞机里只做了一两个人**。 要解决这个问题也不难，就是避免对小的 window size 做出响应，直到有足够大的 window size 再响应，这个思路可以同时实现在 sender 和 receiver 两端。

- 如果这个问题是由 Receiver 端引起的，那么就会使用 David D Clark’s 方案。在 receiver 端，如果收到的数据导致 window size 小于某个值，可以直接 ack(0)回 sender，这样就把 window 给关闭了，也阻止了 sender 再发数据过来，等到 receiver 端处理了一些数据后 windows size 大于等于了 MSS，或者，receiver buffer 有一半为空，就可以把 window 打开让 send 发送数据过来。
- 如果这个问题是由 Sender 端引起的，那么就会使用著名的 [Nagle’s algorithm](http://en.wikipedia.org/wiki/Nagle's_algorithm)。这个算法的思路也是延时处理，他有两个主要的条件：1）要等到 Window Size>=MSS 或是 Data Size >=MSS，2）收到之前发送数据的 ack 回包，他才会发数据，否则就是在攒数据。

另外，Nagle 算法默认是打开的，所以，对于一些需要小包场景的程序——**比如像 telnet 或 ssh 这样的交互性比较强的程序，你需要关闭这个算法**。你可以在 Socket 设置 TCP_NODELAY 选项来关闭这个算法（关闭 Nagle 算法没有全局参数，需要根据每个应用自己的特点来关闭）

```
setsockopt(sock_fd, IPPROTO_TCP, TCP_NODELAY, (char*)&value,sizeof(int));
```

另外，网上有些文章说 TCP_CORK 的 socket option 是也关闭 Nagle 算法，这不对。**TCP_CORK 其实是更新激进的 Nagle 算汉，完全禁止小包发送，而 Nagle 算法没有禁止小包发送，只是禁止了大量的小包发送**。最好不要两个选项都设置。

## TCP 的拥塞处理 – Congestion Handling

上面我们知道了，TCP 通过 Sliding Window 来做流控（Flow Control），但是 TCP 觉得这还不够，因为 Sliding Window 需要依赖于连接的发送端和接收端，其并不知道网络中间发生了什么。TCP 的设计者觉得，一个伟大而牛逼的协议仅仅做到流控并不够，因为流控只是网络模型 4 层以上的事，TCP 的还应该更聪明地知道整个网络上的事。

具体一点，我们知道 TCP 通过一个 timer 采样了 RTT 并计算 RTO，但是，**如果网络上的延时突然增加，那么，TCP 对这个事做出的应对只有重传数据，但是，重传会导致网络的负担更重，于是会导致更大的延迟以及更多的丢包，于是，这个情况就会进入恶性循环被不断地放大。试想一下，如果一个网络内有成千上万的 TCP 连接都这么行事，那么马上就会形成“网络风暴”，TCP 这个协议就会拖垮整个网络。**这是一个灾难。

所以，TCP 不能忽略网络上发生的事情，而无脑地一个劲地重发数据，对网络造成更大的伤害。对此 TCP 的设计理念是：**TCP 不是一个自私的协议，当拥塞发生的时候，要做自我牺牲。就像交通阻塞一样，每个车都应该把路让出来，而不要再去抢路了。**

拥塞控制主要是四个算法：**1）慢启动**，**2）拥塞避免**，**3）拥塞发生**，**4）快速恢复**。这四个算法不是一天都搞出来的，这个四算法的发展经历了很多时间，到今天都还在优化中。 备注:

- 1988 年，TCP-Tahoe 提出了 1）慢启动，2）拥塞避免，3）拥塞发生时的快速重传
- 1990 年，TCP Reno 在 Tahoe 的基础上增加了 4）快速恢复

### 慢热启动算法 – Slow Start

首先，我们来看一下 TCP 的慢热启动。慢启动的意思是，刚刚加入网络的连接，一点一点地提速，不要一上来就像那些特权车一样霸道地把路占满。新同学上高速还是要慢一点，不要把已经在高速上的秩序给搞乱了。

慢启动的算法如下(cwnd 全称 Congestion Window)：

1）连接建好的开始先初始化 cwnd = 1，表明可以传一个 MSS 大小的数据。

2）每当收到一个 ACK，cwnd++; 呈线性上升

3）每当过了一个 RTT，cwnd = cwnd\*2; 呈指数让升

4）还有一个 ssthresh（slow start threshold），是一个上限，当 cwnd >= ssthresh 时，就会进入“拥塞避免算法”（后面会说这个算法）

所以，我们可以看到，如果网速很快的话，ACK 也会返回得快，RTT 也会短，那么，这个慢启动就一点也不慢。下图说明了这个过程。

![img](https://coolshell.cn/wp-content/uploads/2014/05/tcp.slow_.start_.jpg)

一篇 Google 的论文《[An Argument for Increasing TCP’s Initial Congestion Window](http://static.googleusercontent.com/media/research.google.com/zh-CN//pubs/archive/36640.pdf)》Linux 3.0 后采用了这篇论文的建议——把 cwnd 初始化成了 10 个 MSS。而 Linux 3.0 以前，比如 2.6，Linux 采用了[RFC3390](http://www.rfc-editor.org/rfc/rfc3390.txt)，cwnd 是跟 MSS 的值来变的，如果 MSS< 1095，则 cwnd = 4；如果 MSS>2190，则 cwnd=2；其它情况下，则是 3。

### 拥塞避免算法 – Congestion Avoidance

前面说过，还有一个 ssthresh（slow start threshold），是一个上限，当 cwnd >= ssthresh 时，就会进入“拥塞避免算法”。一般来说 ssthresh 的值是 65535，单位是字节，当 cwnd 达到这个值时后，算法如下：

1）收到一个 ACK 时，cwnd = cwnd + 1/cwnd

2）当每过一个 RTT 时，cwnd = cwnd + 1

这样就可以避免增长过快导致网络拥塞，慢慢的增加调整到网络的最佳值。很明显，是一个线性上升的算法。

### 拥塞状态时的算法

前面我们说过，当丢包的时候，会有两种情况：

1）等到 RTO 超时，重传数据包。TCP 认为这种情况太糟糕，反应也很强烈。

- - sshthresh = cwnd /2
  - cwnd 重置为 1
  - 进入慢启动过程

2）Fast Retransmit 算法，也就是在收到 3 个 duplicate ACK 时就开启重传，而不用等到 RTO 超时。

- - TCP Tahoe 的实现和 RTO 超时一样。
- - TCP Reno 的实现是：
    - cwnd = cwnd /2
    - sshthresh = cwnd
    - 进入快速恢复算法——Fast Recovery

上面我们可以看到 RTO 超时后，sshthresh 会变成 cwnd 的一半，这意味着，如果 cwnd<=sshthresh 时出现的丢包，那么 TCP 的 sshthresh 就会减了一半，然后等 cwnd 又很快地以指数级增涨爬到这个地方时，就会成慢慢的线性增涨。我们可以看到，TCP 是怎么通过这种强烈地震荡快速而小心得找到网站流量的平衡点的。

### 快速恢复算法 – Fast Recovery

#### TCP Reno

这个算法定义在[RFC5681](http://tools.ietf.org/html/rfc5681)。快速重传和快速恢复算法一般同时使用。快速恢复算法是认为，你还有 3 个 Duplicated Acks 说明网络也不那么糟糕，所以没有必要像 RTO 超时那么强烈。 注意，正如前面所说，进入 Fast Recovery 之前，cwnd 和 sshthresh 已被更新：

- cwnd = cwnd /2
- sshthresh = cwnd

然后，真正的 Fast Recovery 算法如下：

- cwnd = sshthresh + 3 \* MSS （3 的意思是确认有 3 个数据包被收到了）
- 重传 Duplicated ACKs 指定的数据包
- 如果再收到 duplicated Acks，那么 cwnd = cwnd +1
- 如果收到了新的 Ack，那么，cwnd = sshthresh ，然后就进入了拥塞避免的算法了。

如果你仔细思考一下上面的这个算法，你就会知道，**上面这个算法也有问题，那就是——它依赖于 3 个重复的 Acks**。注意，3 个重复的 Acks 并不代表只丢了一个数据包，很有可能是丢了好多包。但这个算法只会重传一个，而剩下的那些包只能等到 RTO 超时，于是，进入了恶梦模式——超时一个窗口就减半一下，多个超时会超成 TCP 的传输速度呈级数下降，而且也不会触发 Fast Recovery 算法了。

通常来说，正如我们前面所说的，SACK 或 D-SACK 的方法可以让 Fast Recovery 或 Sender 在做决定时更聪明一些，但是并不是所有的 TCP 的实现都支持 SACK（SACK 需要两端都支持），所以，需要一个没有 SACK 的解决方案。而通过 SACK 进行拥塞控制的算法是 FACK（后面会讲）

#### TCP New Reno

于是，1995 年，TCP New Reno（参见 [RFC 6582](http://tools.ietf.org/html/rfc6582) ）算法提出来，主要就是在没有 SACK 的支持下改进 Fast Recovery 算法的——

- 当 sender 这边收到了 3 个 Duplicated Acks，进入 Fast Retransimit 模式，开发重传重复 Acks 指示的那个包。如果只有这一个包丢了，那么，重传这个包后回来的 Ack 会把整个已经被 sender 传输出去的数据 ack 回来。如果没有的话，说明有多个包丢了。我们叫这个 ACK 为 Partial ACK。
- 一旦 Sender 这边发现了 Partial ACK 出现，那么，sender 就可以推理出来有多个包被丢了，于是乎继续重传 sliding window 里未被 ack 的第一个包。直到再也收不到了 Partial Ack，才真正结束 Fast Recovery 这个过程

我们可以看到，这个“Fast Recovery 的变更”是一个非常激进的玩法，他同时延长了 Fast Retransmit 和 Fast Recovery 的过程。

##### 算法示意图

下面我们来看一个简单的图示以同时看一下上面的各种算法的样子：

![img](https://coolshell.cn/wp-content/uploads/2014/05/tcp.fr_-1024x359.jpg)

#### FACK 算法

FACK 全称 Forward Acknowledgment 算法，论文地址在这里（PDF）[Forward Acknowledgement: Refining TCP Congestion Control](http://conferences.sigcomm.org/sigcomm/1996/papers/mathis.pdf) 这个算法是其于 SACK 的，前面我们说过 SACK 是使用了 TCP 扩展字段 Ack 了有哪些数据收到，哪些数据没有收到，他比 Fast Retransmit 的 3 个 duplicated acks 好处在于，前者只知道有包丢了，不知道是一个还是多个，而 SACK 可以准确的知道有哪些包丢了。 所以，SACK 可以让发送端这边在重传过程中，把那些丢掉的包重传，而不是一个一个的传，但这样的一来，如果重传的包数据比较多的话，又会导致本来就很忙的网络就更忙了。所以，FACK 用来做重传过程中的拥塞流控。

- 这个算法会把 SACK 中最大的 Sequence Number 保存在**snd.fack**这个变量中，snd.fack 的更新由 ack 带秋，如果网络一切安好则和 snd.una 一样（snd.una 就是还没有收到 ack 的地方，也就是前面 sliding window 里的 category #2 的第一个地方）
- 然后定义一个**awnd = snd.nxt – snd.fack**（snd.nxt 指向发送端 sliding window 中正在要被发送的地方——前面 sliding windows 图示的 category#3 第一个位置），这样 awnd 的意思就是在网络上的数据。（所谓 awnd 意为：actual quantity of data outstanding in the network）
- 如果需要重传数据，那么，**awnd = snd.nxt – snd.fack + retran_data**，也就是说，awnd 是传出去的数据 + 重传的数据。
- 然后触发 Fast Recovery 的条件是： ( **( snd.fack – snd.una ) > (3\*MSS)** ) || (dupacks == 3) ) 。这样一来，就不需要等到 3 个 duplicated acks 才重传，而是只要 sack 中的最大的一个数据和 ack 的数据比较长了（3 个 MSS），那就触发重传。在整个重传过程中 cwnd 不变。直到当第一次丢包的 snd.nxt<=snd.una（也就是重传的数据都被确认了），然后进来拥塞避免机制——cwnd 线性上涨。

我们可以看到如果没有 FACK 在，那么在丢包比较多的情况下，原来保守的算法会低估了需要使用的 window 的大小，而需要几个 RTT 的时间才会完成恢复，而 FACK 会比较激进地来干这事。 但是，FACK 如果在一个网络包会被 reordering 的网络里会有很大的问题。

## 其它拥塞控制算法简介

### **TCP Vegas 拥塞控制算法**

这个算法 1994 年被提出，它主要对 TCP Reno 做了些修改。这个算法通过对 RTT 的非常重的监控来计算一个基准 RTT。然后通过这个基准 RTT 来估计当前的网络实际带宽，如果实际带宽比我们的期望的带宽要小或是要多的活，那么就开始线性地减少或增加 cwnd 的大小。如果这个计算出来的 RTT 大于了 Timeout 后，那么，不等 ack 超时就直接重传。（Vegas 的核心思想是用 RTT 的值来影响拥塞窗口，而不是通过丢包） 这个算法的论文是《[TCP Vegas: End to End Congestion Avoidance on a Global Internet](http://www.cs.cmu.edu/~srini/15-744/F02/readings/BP95.pdf)》这篇论文给了 Vegas 和 New Reno 的对比：

![img](https://coolshell.cn/wp-content/uploads/2014/05/tcp_vegas_newreno-1024x555.jpg)

关于这个算法实现，你可以参看 Linux 源码：[/net/ipv4/tcp_vegas.h](http://lxr.free-electrons.com/source/net/ipv4/tcp_vegas.h)， [/net/ipv4/tcp_vegas.c](http://lxr.free-electrons.com/source/net/ipv4/tcp_vegas.c)

### HSTCP(High Speed TCP) 算法

这个算法来自[RFC 3649](http://tools.ietf.org/html/rfc3649)（[Wikipedia 词条](http://en.wikipedia.org/wiki/HSTCP)）。其对最基础的算法进行了更改，他使得 Congestion Window 涨得快，减得慢。其中：

- 拥塞避免时的窗口增长方式： cwnd = cwnd + α(cwnd) / cwnd
- 丢包后窗口下降方式：cwnd = (1- β(cwnd))\*cwnd

注：α(cwnd)和 β(cwnd)都是函数，如果你要让他们和标准的 TCP 一样，那么让 α(cwnd)=1，β(cwnd)=0.5 就可以了。 对于 α(cwnd)和 β(cwnd)的值是个动态的变换的东西。 关于这个算法的实现，你可以参看 Linux 源码：[/net/ipv4/tcp_highspeed.c](http://lxr.free-electrons.com/source/net/ipv4/tcp_highspeed.c)

### TCP BIC 算法

2004 年，产内出 BIC 算法。现在你还可以查得到相关的新闻《Google：[美科学家研发 BIC-TCP 协议 速度是 DSL 六千倍](https://www.google.com/search?lr=lang_zh-CN|lang_zh-TW&newwindow=1&biw=1366&bih=597&tbs=lr%3Alang_1zh-CN|lang_1zh-TW&q=美科学家研发BIC-TCP协议+速度是DSL六千倍&oq=美科学家研发BIC-TCP协议+速度是DSL六千倍)》 BIC 全称[Binary Increase Congestion control](http://research.csc.ncsu.edu/netsrv/?q=content/bic-and-cubic)，在 Linux 2.6.8 中是默认拥塞控制算法。BIC 的发明者发这么多的拥塞控制算法都在努力找一个合适的 cwnd – Congestion Window，而且 BIC-TCP 的提出者们看穿了事情的本质，其实这就是一个搜索的过程，所以 BIC 这个算法主要用的是 Binary Search——二分查找来干这个事。 关于这个算法实现，你可以参看 Linux 源码：[/net/ipv4/tcp_bic.c](http://lxr.free-electrons.com/source/net/ipv4/tcp_bic.c)

### TCP WestWood 算法

westwood 采用和 Reno 相同的慢启动算法、拥塞避免算法。westwood 的主要改进方面：在发送端做带宽估计，当探测到丢包时，根据带宽值来设置拥塞窗口、慢启动阈值。 那么，这个算法是怎么测量带宽的？每个 RTT 时间，会测量一次带宽，测量带宽的公式很简单，就是这段 RTT 内成功被 ack 了多少字节。因为，这个带宽和用 RTT 计算 RTO 一样，也是需要从每个样本来平滑到一个值的——也是用一个加权移平均的公式。 另外，我们知道，如果一个网络的带宽是每秒可以发送 X 个字节，而 RTT 是一个数据发出去后确认需要的时候，所以，X _ RTT 应该是我们缓冲区大小。所以，在这个算法中，ssthresh 的值就是 est_BD _ min-RTT(最小的 RTT 值)，如果丢包是 Duplicated ACKs 引起的，那么如果 cwnd > ssthresh，则 cwin = ssthresh。如果是 RTO 引起的，cwnd = 1，进入慢启动。 关于这个算法实现，你可以参看 Linux 源码： [/net/ipv4/tcp_westwood.c](http://lxr.free-electrons.com/source/net/ipv4/tcp_westwood.c)

## 四次挥手优化——三次挥手

```json
#三次挥手 -- 客户端发起断开连接请求    客户端序列号为 3495051432
#fin  seq = 3495051432
00:05:30.157325 IP 221.122.42.100.58706 > VM_0_6_centos.webcache: Flags [F.], seq 3495051432, ack 485492629, win 115, options [nop,nop,TS val 1695781058 ecr 629900122], length 0
#三次挥手 -- 服务器端发起断开连接请求
#ack = 3495051432 + 1 = 3495051433  注意 因为服务器端也没有东西要发送了，所以也要关闭连接，因此同时发送了fin信号，seq = 485492629
00:05:30.157562 IP VM_0_6_centos.webcache > 221.122.42.100.58706: Flags [F.], seq 485492629, ack 3495051433, win 235, options [nop,nop,TS val 629900162 ecr 1695781058], length 0
#三次挥手 -- 客户端应答
#ack = 485492629 + 1 = 485492630
00:05:30.196710 IP 221.122.42.100.58706 > VM_0_6_centos.webcache: Flags [.], ack 485492630, win 115, options [nop,nop,TS val 1695781097 ecr 629900162], length 0
```

这是因为关闭连接有两种方式，当一方关闭连接，另外一方没有数据发送时，马上关闭连接，也就将第二步的 ack 与第三步的 fin 合并为一步了
