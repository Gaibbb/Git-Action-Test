---
title: c++ winsock实现Ping功能
tags: []
id: '10'
categories:
  - - 'Code'
date: 2023-07-08 21:43:21
---

## 整体流程

1、创建套接字（SOCKET），为套接字设定超时选项（SETSOCKOPT）；

2、创建目标地址ADDR（SOCKADDR\_IN）；

3、创建并填充ICMP包（计算校验和）；

4、连接至目标主机并发送ICMP包（connect、send）；

5、接收并解析回来的ICMP包（recvfrom）；

### 创建套接字（SOCKET），为套接字设定超时选项（SETSOCKOPT）

```c++
int nTime = 1000; //设定超时时间为1s
SOCKET sockfd = socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);
setsockopt(sockfd, SOL_SOCKET, SO_RCVTIMEO, (char*)&nTime, sizeof(nTime));
```

### 创建目标地址ADDR（SOCKADDR\_IN）

```c++
SOCKADDR_IN addr;
addr.sin_family = AF_INET;
addr.sin_port = htons(0); //端口0指让系统动态选择一个可用的端口
addr.sin_addr.S_un.S_addr = inet_addr("Your destIP");
```

### 创建并填充ICMP包（计算校验和）

```
+---------------------------------------------------+
              ICMP Header (8 bytes)                
+---------------------------------------------------+
   Type (1 byte)     Code (1 byte)    Checksum    
+---------------------------------------------------+
              Identifier (2 bytes)                 
+---------------------------------------------------+
              Sequence Number (2 bytes)             
+---------------------------------------------------+
                    Data (variable)                 
+---------------------------------------------------+
```

上图是由GPT生成的一个ICMP包的基本结构，具体代码如下：

```c++
struct icmp_header{
    char type;  //类型
    char code;  //进一步细化ICMP类型
    unsigned short checksum;  //校验和
    unsigned short id;  //匹配进程ID
    unsigned short seq;  //序列号
    unsigned long timestamp; //时间戳 用以计算延迟
}

char buf[sizeof(icmp_hdr)];
icmp_header* pIcmp = (imcp_headerr*)buf;  //创建icmp_hdr的指针pIcmp并指向强制类型转换后的buff
pIcmp->type = 8; //回显
pIcmp->code = 0;
pIcmp->checksum = 0; //发送前再填充
pIcmp->id = GetCurrentProcessId(); //获取当前程序ID用以后面匹配包
pIcmp->seq = 0; //发送前再填充
pIcmp->timestamp; //发送前再填充

//计算校验和，直接借用于
unsigned short checksum(unsigned short* buf, int size)
{
unsigned long sum = 0;
while(size>1){
sum += *buf++;
size -= sizeof(unsigned short);
}
if(size){
sum += *(char*)buf;
    }
sum = (sum >> 16) + (sum & 0xffff);
sum += (sum >> 16);
return (unsigned short)(~sum);
}
```

### 连接至目标主机并发送ICMP包（connect、send）

```c++
connect(sockfd, (sockaddr*)&addr, sizeof(addr);
int ret = send(sockfd, buf, sizeof(icmp_hdr), 0);
if(ret <= 0){
cout << "Send failed, error code: " << WSAGetLastError() << endl;
 return -1;
}
```

### 接收并解析回来的ICMP包（recvfrom）

```c++
unsigned short nSeq;
char recvbuf[32];
SOCKADDR_IN from; //保存对方地址信息
int nLen = sizeof(from);
icmp_header* precvimcp;

memset((void *)recvbuf, 0, sizeof(recvbuf));
int ret = recvfrom(sockfd, recvbuf, ,sizeof(recvbuf), 0, (sockaddr*)&from, &nLen);

if(ret == SOCKET_ERROR){
if(WSAGetLastError() == WSATIMEDOUT){
cout << "Timed out!" << endl;
continue; //完全代码请看下面
}else{
cout << "Recv failed, error code: " << WSAGetLastError() << endl;
return -1;
}
}

precvimp = (icmp_header*)(recvbuf + 20);
cout << ret << " bytes from: " << inet_ntoa(from.sin_addr);
cout << " icmp_seq = " << precvicmp->seq;
cout << " time: " << GetTickCount() - precvicmp->timestamp << " ms" << endl;
Sleep(1000);
```

## 完整程序代码

```c++
#include<winsock2.h>
#include<iostream>
#include<windows.h>
#pragma comment(lib,"ws2_32.lib")
using std::cout;
using std::endl;
#define DATALEN 1024

struct icmp_header{
    unsigned char type; //报文类型
    unsigned char code; //详细报文类型
    unsigned short checksum; //校验和
    unsigned short id; //进程ID
    unsigned short seq; //序列号
    unsigned long timestamp; //时间戳，非标准ICMP头部，用于计算延迟
};

//计算校验和
unsigned short checksum(unsigned short* buf, int size)  
{
unsigned long sum = 0;
while(size>1){
sum += *buf++; //每16位依次相加
size -= sizeof(unsigned short);
}
if(size){
sum += *(char*)buf; //若为奇数，最后一个字节将被视为低8位，高8位为0
    }
sum = (sum >> 16) + (sum & 0xffff); //高16位与低16位相加
sum += (sum >> 16); //将所有相加得到的16位值相加
return (unsigned short)(~sum); //取结果的补码
}

int main(){
    WSADATA data;
    WSAStartup(MAKEWORD(2,2), &data);

    SOCKET sockfd = socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);
    int ntime = 1000;
    setsockopt(sockfd, SOL_SOCKET, SO_RCVTIMEO, (char*)&ntime, sizeof(ntime));

    SOCKADDR_IN addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(0);
    addr.sin_addr.S_un.S_addr = inet_addr("Your_IP");

    char buf[sizeof(icmp_header)];
    icmp_header* picmp = (icmp_header*)buf;
    picmp->checksum = 0; //发送前再填充
    picmp->code = 0;
    picmp->id = (unsigned short)GetCurrentProcessId(); //获取当前程序ID
    picmp->seq = 0; //发送前再填充
    picmp->timestamp = 0; //发送前再填充
    picmp->type = 8; //h回显
    connect(sockfd, (sockaddr*)&addr, sizeof(addr)); //连接目标服务器

    unsigned short nseq = 0; //序列号
    char recvbuf[32];
    SOCKADDR_IN from; //保存对方地址信息
    int nLen = sizeof(from);
    icmp_header* precvicmp; //保存RECV的数据
    int ret;

    while(nseq < 10){
        picmp->checksum = 0; //重置校验和
        picmp->seq = nseq++; //完成本次循环后序列号++
        picmp->timestamp = GetTickCount(); //获取当前时间戳
        //这里一定要注意，校验和需要在正确填充完ICMP包后才进行计算，否则会出现TIMEOUT 10060错误
        picmp->checksum = checksum((unsigned short*)&buf, sizeof(icmp_header));

        ret = send(sockfd, buf, sizeof(icmp_header), 0);

        memset((void *)recvbuf, 0, sizeof(recvbuf)); //重置recv缓冲区
        ret = recvfrom(sockfd, recvbuf, sizeof(recvbuf), 0, (sockaddr*)&from, &nLen);
        if(ret == SOCKET_ERROR){
            if(WSAGetLastError() == WSAETIMEDOUT){
                cout << "Timed out! " << endl;
                continue;
            }else{
                cout << "recv failed, error code: " << WSAGetLastError() << endl;
                return -1;
            }
        }

        precvicmp = (icmp_header*)(recvbuf + 20); //20后的才是ICMP头部
        cout << ret << " bytes from: " << inet_ntoa(from.sin_addr);
        cout << " icmp_seq = " << precvicmp->seq;
        cout << " time: " << GetTickCount() - precvicmp->timestamp << " ms" << endl;
        Sleep(1000);
    }

    closesocket(sockfd);
    WSACleanup();
    return 0;
}
```

## 效果图

![](https://www.kirisamekano.com/image/c-winsock-ping/1.jpg)

若在编译时出现下图错误：

![](https://www.kirisamekano.com/image/c-winsock-ping/2.png)

可以尝试在编译时加入-lws2\_32，完整编译指令：

```
g++ winsock_Ping.cpp -lws2_32 -o winsock_Ping.exe
```
