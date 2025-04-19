---
title: WireShark使用（二）
tags: []
id: '12'
categories:
  - - 'Software'
date: 2023-07-09 16:53:50
---

实验二：利用WireShark抓包并分析三次握手

步骤：

打开WireShark监听后，随便打开一个网页即可。

由于上一篇文章已经讲述过了数据链路层和传输层，本实验和上一个实验的这两部分除了包长度以外并无太大差异，因此直接讲解TCP包。

![](https://www.kirisamekano.com/image/wireshark2/1.png)

在最右侧的一列可以看到很完美的一个三次握手流程，从本地端到服务端的SYN包，然后是服务端到本地段的SYN-ACK包，最后是本地端到服务端的ACK包。

![](https://www.kirisamekano.com/image/wireshark2/2.png)

首先是第一个SYN包，可以看到发送的端口与目标端口，以及该SYN包的序列号，ACK号。

接下来的Flags中SYN的字段被设置为了1，表示这是一个SYN包。然后是窗口大小与校验和。

紧急数据指针被设置为0。

接下来的选项中可以看到Maximum segment size（MSS）的大小，以及Window scale窗口比例为8，并且通过乘256来确定窗口大小。这一点在计算机网络里面有详细的解释。

最后是选择性重传选项，涉及不多就不展开。

![](https://www.kirisamekano.com/image/wireshark2/3.png)

第二个SYN-ACK包，可以看到发送端口和接收端口和上一个刚好反过来，这次序列号发生了改变，并且ACK（RAW）号在收到的SYN包的序列号上加了1，变为1452813178。

接下来的Flags中除了SYN位被设置为1，ACK位也被设置为1。

在选项中的MSS也发生了改变，变为服务端能接受的最大MSS。

以及窗口比例也发生了改变，都是同MSS一样，是服务端能接受的大小。

![](https://www.kirisamekano.com/image/wireshark2/4.png)

最后的ACK应答中，端口号和第一个SYN包一致。

可以看到偏移序列号变为了1，并且raw SEQ变为了上一个接收到的SYN-ACK包中的raw ACK号，ACK号变为了它的raw SEQ + 1。

Flags位只有ACK被设置为1。

确定了最后的窗口大小与窗口比例。

至此，三次握手完成，双方建立了可靠的TCP连接。这里需要重点关注的是SEQ与ACK号之间的变化，很好的说明了二者是如何完成三次握手建立信任关系的。
