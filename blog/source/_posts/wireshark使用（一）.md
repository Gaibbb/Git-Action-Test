---
title: WireShark使用（一）
tags: []
id: '11'
categories:
  - - 'Software'
date: 2023-07-09 15:55:14
---

实验一：利用WireShark与Ping功能抓取分析ICMP包

步骤：

1、打开WireShark并选择监听的网卡。

2、发送Ping指令，等待Ping指令完成。

3、暂停WireShark抓包，过滤出Ping指令发送的包。

![](https://www.kirisamekano.com/image/wireshark1/1.png)

在这里可以看到Ping向百度的目标IP地址为120.232.145.185，在WireShark的过滤栏中输入ip.addr==120.232.145.185。得到结果如下图：

![](https://www.kirisamekano.com//image/wireshark1/2.png)

选择一个包双击进入查看详情，由上到下分别代表不同的层级，分别是物理层、数据链路层、网络层。

![](https://www.kirisamekano.com/image/wireshark1/3.png)

本文不关注物理层的数据，直接从第二层数据链路层开始分析：

![](https://www.kirisamekano.com//image/wireshark1/4.png)

Destination中的便是下一条的MAC地址，Source的就是自己的MAC地址。

接下来看第三层网络层：

![](https://www.kirisamekano.com/image/wireshark1/5.png)

结合下面的IPv4包结构来看

```
   0                   1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  Version  IHL     DSCP  ECN          Total Length           
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           Identification        Flags      Fragment Offset    
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    TTL      Protocol        Header Checksum                 
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                          Source IP Address                      
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                      Destination IP Address                     
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                      Options (if any)                          
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                               Data                              
                               ...                               
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

使用IP协议，版本为IPv4，头部长度为20bytes。

然后是Differentiated Services Field区分服务字段，填入了0x00，DSCP优先度为0（普通数据），ECN字段被设置为Not-ECT，也就是不支持显示拥塞通知。

Total Lenght，IP包总长度为60。

Flags位都设置为了0，第一个表示DF（Don‘t Flags）不要分片；第二个表示MF（More Flags）当它被设置为1时，表示次IP包是分片的一部分；第三个是保留位。

Fragment Offset偏移位，由于没有进行分片，所以此处为0；

Time to Live（TTL），生存时间。

Protocol，封装在IP包上面的是ICMP协议。

Header Checksum头部校验和，0x0000关闭校验和。

来源地址为192.168.1.46，目标地址为120.232.145.185。

接下来是ICMP协议的部分：

![](https://www.kirisamekano.com/image/wireshark1/6.png)

Type类型8，回显请求，下面的其它选项都在上一篇文章中的构造ICMP包讲过了，这里不多赘述。
