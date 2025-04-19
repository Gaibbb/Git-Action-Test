---
title: c++ winsock实现sniffer
tags: []
id: '13'
categories:
  - - 'Code'
date: 2023-07-10 18:14:17
---

本文一些基础的winsock函数的使用已经在上一篇文章：[c++ winsock实现Ping功能](https://www.kirisamekano.com/?p=238) 出现过，本篇文章只记录新出现的函数以及新的结构体等。

## 整体结构

1、选定监听地址——本地IPv4地址（该部分可以优化）;

2、设置套接字接收完全IP头部信息;

3、绑定套接字与地址并设置套接字接收所有网络流量;

4、接收并分析数据;

### 选定监听地址——本地IPv4地址

可以在CMD中使用“ipconfig"指令查找以太网IPv4地址：

![](https://www.kirisamekano.com/image/c-winsock-sniffer/1.png)

然后在创建SOCKADDR\_IN时将IP地址填充进结构中：

```c++
sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_port = htons(0);
addr.sin_addr.S_un.S_addr = inet_addr("192.168.1.46");
```

### 设置套接字接收完全IP头部信息

```c++
SOCKET sockfd = socket(AF_INET, SOCK_RAW, IPPROTO_IP);
int optionValue = 1;
if (setsockopt(sockfd, IPPROTO_IP, IP_HDRINCL, (char*)optionValue), sizeof(optionValue)) {
    cout << "setsockopt failed, error code: " << WSAGetLastError() << endl;
    return -1;
}
```

### 绑定套接字与地址并设置套接字接收所有网络流量

```c++
bind(sockfd, (sockaddr*)&addr, sizeof(addr), 0);
ioctlsocket(sockfd, SIO_RCVALL, (u_long*)&optionValue);
```

### 接收并分析数据

```c++
void decode(char *buf) {
    iphdr * ipheader = (iphdr*)buf;
    cout << "Source IP: " << inet_ntoa(ipheader->ip_srcip) << endl;
    cout << "Destination IP: " <<  inet_ntoa(ipheader->ip_dstip) << endl;

    if (ipheader->ip_protocol == IPPROTO_TCP) {
        tcphdr* tcpheader = (tcphdr*)(buf + IPLen);
        cout << "TCP Protocol" << endl;
        cout << "Source Port: " << ntohs(tcpheader->tcp_srcp) << endl;
        cout << "Destination Port: " << ntohs(tcpheader->tcp_dstp) << "\n" << endl;
    } else if (ipheader->ip_protocol == IPPROTO_UDP) {
        udphdr* udpheader = (udphdr*)(buf + IPLen);
        cout << "UDP Protocol" << endl;
        cout << "Source Port: " << ntohs(udpheader->udp_srcp) << endl;
        cout << "Destination Port: " << ntohs(udpheader->udp_dstp) << "\n" << endl;
    } else if (ipheader->ip_protocol == IPPROTO_ICMP){
        cout << "ICMP Protocol" << endl;
    } else {
        cout << "Not support this protocol yet, Protocol in INT is: " << int(ipheader->ip_protocol) << endl;
    }
}
```

由于本程序只分析了两种类型的协议，加上IP首部共三个，他们的结构如下：
## 首部结构

### IP

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
Version  IHL  Type of Service          Total Length         
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
         Identification        Flags      Fragment Offset    
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  Time to Live     Protocol            Header Checksum       
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                       Source IP Address                      
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                    Destination IP Address                    
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                    Options                        Padding    
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

In C++
struct iphdr {
    u_char ip_ver:4 , ip_hlen:4; //ip版本与头部长度
    u_char ip_tos; //服务类型
    u_short ip_tlen; //ip包总长度
    u_short ip_id; //数据包标识
    u_short ip_flag_offset; //分段标志位与偏移位
    u_char ip_ttl; //生存时间
    u_char ip_protocol; //上层协议
    u_short ip_checksum; //校验和
    in_addr ip_srcip; //来源IP地址
    in_addr ip_dstip;  //目标IP地址
};
```

### TCP

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           Source Port                 Destination Port        
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                        Sequence Number                        
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                    Acknowledgment Number                      
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  Data  ReservedUAPRSF                                  
 Offset        RCSSYI            Window                 
               GKHTNN                                  
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           Checksum                     Urgent Pointer         
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                    Options                        Padding    
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

In C++
struct tcphdr {
    u_short tcp_srcp; //来源端口
    u_short tcp_dstp; //目标端口
    u_long tcp_seq; //序列号
    u_long tcp_ack; //确认号
    u_char tcp_offset; //偏移位
    u_char tcp_flags; //标志位
    u_short tcp_window; //窗口大小
    u_short tcp_checksum; //校验和
    u_short tcp_urp; //紧急指针
};
```

### UDP

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           Source Port                 Destination Port        
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            Length                        Checksum            
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

In C++
struct udphdr {
    u_short udp_srcp; //来源端口
    u_short udp_dstp; //目标端口
    u_short udp_len; //包长度
    u_short udp_checksum; //校验和
};
```

这里重点讲一下新的"ioctsocket"函数，它的完整结构如下：

```c++
int ioctlsocket(SOCKET s, long cmd, u_long* argp);
```

他和setsockopt看起来有点相似，都是定义套接字的行为，但是ioctsocket函数能控制的行为更多，例如将套接字设置为非阻塞模式等：

```c++
u_long optionValue = 1;
ioctlsocket(socket, FIONBIO, &optionValue);
```

本程序便是使用了该函数来设置socket接收所有的网络流量，包括发送出去的：

```c++
ioctlsocket(sockfd, SIO_RCVALL, (u_long*)&optionValue);
```

还有一个要讲的地方是setsockopt中的IP\_HDRINCL，启用该选项要在程序头部加入"ws2tcpip.h"头部文件的链接。该选项的作用是使套接字使用完整的IP头部而不是系统自动生成，这意味着在发送数据的时候我们要自行创建一个完整的IP头部。相反在接收数据的时候，我们可以接收到一个拥有完整IP头部的数据包。

```c++
if(setsockopt(sockfd, IPPROTO_IP, IP_HDRINCL, (char*)&optionValue, sizeof(optionValue)) == SOCKET_ERROR) {
    cout << "setsockopt failed, error code: " << WSAGetLastError() << endl;
    return -1;
}
```

## 完整代码

```c++
#include<iostream>
#include<winsock2.h>
#include<ws2tcpip.h>
#pragma comment(lib, "lws2_32.lib")
#define IPLen 20
using std::cout;
using std::endl;

struct iphdr {
    u_char ip_ver:4 , ip_hlen:4; //ip版本与头部长度
    u_char ip_tos; //服务类型
    u_short ip_tlen; //ip包总长度
    u_short ip_id; //数据包标识
    u_short ip_flag_offset; //分段标志位与偏移位
    u_char ip_ttl; //生存时间
    u_char ip_protocol; //上层协议
    u_short ip_checksum; //校验和
    in_addr ip_srcip; //来源IP地址
    in_addr ip_dstip;  //目标IP地址
};

struct tcphdr {
    u_short tcp_srcp; //来源端口
    u_short tcp_dstp; //目标端口
    u_long tcp_seq; //序列号
    u_long tcp_ack; //确认号
    u_char tcp_offset; //偏移位
    u_char tcp_flags; //标志位
    u_short tcp_window; //窗口大小
    u_short tcp_checksum; //校验和
    u_short tcp_urp; //紧急指针
};

struct udphdr {
    u_short udp_srcp; //来源端口
    u_short udp_dstp; //目标端口
    u_short udp_len; //包长度
    u_short udp_checksum; //校验和
};

void decode(char *buf) {
    iphdr * ipheader = (iphdr*)buf;
    cout << "Source IP: " << inet_ntoa(ipheader->ip_srcip) << endl;
    cout << "Destination IP: " <<  inet_ntoa(ipheader->ip_dstip) << endl;

    if (ipheader->ip_protocol == IPPROTO_TCP) {
        tcphdr* tcpheader = (tcphdr*)(buf + IPLen);
        cout << "TCP Protocol" << endl;
        cout << "Source Port: " << ntohs(tcpheader->tcp_srcp) << endl;
        cout << "Destination Port: " << ntohs(tcpheader->tcp_dstp) << "\n" << endl;
    } else if (ipheader->ip_protocol == IPPROTO_UDP) {
        udphdr* udpheader = (udphdr*)(buf + IPLen);
        cout << "UDP Protocol" << endl;
        cout << "Source Port: " << ntohs(udpheader->udp_srcp) << endl;
        cout << "Destination Port: " << ntohs(udpheader->udp_dstp) << "\n" << endl;
    } else if (ipheader->ip_protocol == IPPROTO_ICMP){
        cout << "ICMP Protocol" << endl;
    } else {
        cout << "Not support this protocol yet, Protocol in INT is: " << int(ipheader->ip_protocol) << endl;
    }
}

int main() {
    WSADATA data;
    WSAStartup(MAKEWORD(2,2), &data);

    SOCKET sockfd = socket(AF_INET, SOCK_RAW, IPPROTO_IP);
    int optionValue = 1;
    //设置接收完整的IP首部
    if(setsockopt(sockfd, IPPROTO_IP, IP_HDRINCL, (char*)&optionValue, sizeof(optionValue)) == SOCKET_ERROR) {
        cout << "setsockopt failed, error code: " << WSAGetLastError() << endl;
        return -1;
    }

    SOCKADDR_IN addr;
    addr.sin_addr.S_un.S_addr = inet_addr("192.168.1.46");
    addr.sin_family = AF_INET;
    addr.sin_port = htons(0);

    if(bind(sockfd, (sockaddr*)&addr, sizeof(addr)) == SOCKET_ERROR){
        cout << "bind failed, error code: " << WSAGetLastError() << endl;
        return -1;
    }
    ioctlsocket(sockfd, SIO_RCVALL, (u_long*)&optionValue); //设置接收所有网络流量

    char recvbuf[65536];
    int buflen = sizeof(recvbuf);
    while(TRUE){
        memset(recvbuf, 0, buflen);
        if (recv(sockfd, recvbuf, buflen, 0) > 0) {
            decode(recvbuf);
        } else {
            cout << "recv failed, error code: " << WSAGetLastError() << endl;
            return -1;
        }
    }

    closesocket(sockfd);
    WSACleanup();
    system("pause");
    return 0;
}
```

文章一开始提到了获取本地IP地址的部分可以进行优化，我会在完成下一个winsock聊天室后进行研究，如果大佬有想法的话也可以留下评论，我好好学习一下！
