---
title: c++ winsock + thread实现聊天室
tags: []
id: '14'
categories:
  - - 'Code'
date: 2023-07-16 12:04:08
---

狠狠的学了一天线程写了一天代码，终于搞完只因网实验的最后一个了，大致流程如下：

## 流程
  
### 服务端

1、初始化并创建套接字，监听端口。

2、将到来的连接用accept接收，并为客户端创建一个线程。

3、客户端输入信息，将到来的客户端信息转发到其他套接字。

4、客户端输入quit离开聊天室，服务器关闭该线程并删除其套接字信息。

### 服务端

1、初始化并创建套接字，连接服务器。

2、使用一个while循环输入信息并发送。

3、使用一个子线程为接收数据做处理。

4、输入quit离开聊天室，关闭接收线程。

没有什么难点，花的时间主要是在thread和mutex的学习上面。一开始想着线程池解决，但是发现看不懂，得先从普通线程做起，下面直接上代码。   


## 代码文件

### 服务端头文件

```c++
#include<iostream>
#include<winsock2.h>
#include<thread>
#include<mutex>
#include<vector>
#include<string.h>
#include<queue>
#pragma comment(lib, "ws2_32.lib")
using std::cout;
using std::endl;

struct socketforward {
    SOCKET socfd; //信息发送者（套接字）
    std::string msg; //信息
};

struct iphdr {
    u_short ihlver;
    u_short tos;
    u_long tlen;
    u_long id;
    u_long flag_offset;
    u_short ttl;
    u_short protocol;
    u_long sum;
    in_addr srcIP;
    in_addr desIP;
};

extern std::mutex mtx; //创建互斥量
extern std::vector<std::thread> threads; //创建线程数组
extern std::vector<SOCKET> socketArray; //创建套接字数组
extern std::queue<socketforward> MessageQueue; //创建信息队列
extern int clinum; //客户端数量

void forwardMessages();  //转发函数
void sockethandler(SOCKET socketcli); //处理函数

std::string getcliIP(std::string buf); //获取客户端ip地址
```

### 服务端函数实现

```c++
#include<iostream>
#include<winsock2.h>
#include<thread>
#include<mutex>
#include<vector>
#include<string.h>
#include<queue>
#include"threadsocketserver.h"
#pragma comment(lib, "ws2_32.lib")
using std::cout;
using std::endl;

std::mutex mtx; //创建互斥量
std::vector<std::thread> threads; //创建线程数组
std::vector<SOCKET> socketArray; //创建套接字数组
std::queue<socketforward> MessageQueue; //创建信息队列
int clinum;

void forwardMessages() {
    while (true) {
        std::string message; //转发信息处理
        std::lock_guard<std::mutex> lock(mtx); //使用互斥锁保证共享资源的读写
        if (MessageQueue.empty()) {
            continue;  //信息队列为空的话就重新检测信息队列
        }

        std::string user = getcliIP(MessageQueue.front().msg); //获取客户端IP地址

        message = user + ": " + MessageQueue.front().msg; //拼接转发信息
        int msglen = message.size();
        const char* msgsend = message.c_str();
        SOCKET sockfd = MessageQueue.front().socfd; //获取发送该信息的套接字
        MessageQueue.pop(); //弹出该信息

        //下面这个for循环用了在线套接字的数量作为要转发的次数
        //if语句判断是否为信息的发送者，如果是的话就不用转发给他了
        //挨个用户转发信息
        for (int i = 0; i < socketArray.size(); i++) {
            if(socketArray[i] != sockfd) {
                if (send(socketArray[i], msgsend, msglen, 0) == SOCKET_ERROR) {
                    cout << "send failed, socket code: " << socketArray[i] << endl;
                }
            }
        }
    }
}

void sockethandler(SOCKET socketcli) {
    char buf[256];
    int buflen = sizeof(buf);
    while (true) {
        memset(buf, 0, buflen);
        int ret = recv(socketcli, buf, buflen, 0);
        if (ret < 0) {
            cout << "recv failed, error code: " << WSAGetLastError() << endl;
            break;
        } else if(ret > 0) {
            std::lock_guard<std::mutex> lock(mtx);
            cout << "recv from thread " << std::this_thread::get_id() << " msg: " << buf << endl;
            socketforward sf;  //创建socketforward结构对象 sf
            sf.msg = buf; //填充sf对象
            sf.socfd = socketcli; //填充sf对象
            MessageQueue.push(sf); //将sf对象推入消息队列
        } else {
            std::lock_guard<std::mutex> lock(mtx);
            std::vector<SOCKET>::iterator itr = socketArray.begin(); //创建迭代器指向套接字数组头
            while(itr != socketArray.end()) {
                if(*itr == socketcli) {
                    itr = socketArray.erase(itr); //判断该socket是否为退出聊天室的用户，是的话就使用erase删除
                    break; //跳出该while循环，意味着线程结束
                } else {
                    ++itr;
                }
            }
            cout << "Client close socket" << endl;
            clinum--; //客户端数量-1
            return; //退出线程
        }
    }
    closesocket(socketcli);
}

//前面文章讲sniffer的地方有这个
std::string getcliIP(std::string buf) {
    iphdr* header = (iphdr*)buf.c_str();
    return inet_ntoa(header->srcIP);
}
```

### 服务端主函数

```c++
#include"threadsocketserver.h"

#include<iostream>

int main() {

    WSADATA data;

    WSAStartup(MAKEWORD(2, 2), &data);

    SOCKET socketser, socketcli;

    socketser = socket(AF_INET, SOCK_STREAM, IPPROTO_IP);

    SOCKADDR_IN addr;

    addr.sin_family = AF_INET;

    addr.sin_port = htons(27015);

    addr.sin_addr.s_addr = inet_addr("127.0.0.1");

    int addrlen = sizeof(addr);

    bind(socketser, (sockaddr*)&addr, addrlen);

    listen(socketser, 5);

    std::thread forwardthread(forwardMessages);

    //最多只能支持5个用户进入聊天室，具体数量可以通过改变clinum实现

    while (clinum < 5) {

        socketcli = accept(socketser, (sockaddr*)&addr, &addrlen); //接收连接请求并保存客户端套接字

        cout << "current client socket = " << socketcli << endl;

        if (socketcli == SOCKET_ERROR) {

            cout << "accept failed, error code: " << WSAGetLastError() << endl;

            continue;

        } else {

            socketArray.push_back(socketcli); //将客户端套接字保存到套接字数组

            cout << "Now have: " << socketArray.size() << " sockets" << endl;

            for(int i = 0; i < socketArray.size(); i++) {

                cout << "socket: " << socketArray[i] << endl;

            }

            threads.emplace_back([socketcli]() {sockethandler(socketcli);}); //为客户端创建线程

        }

        ++clinum; //客户端数量+1

    }

    //等待所有客户端线程完成

    for (auto& thread : threads)

        thread.join();

    forwardthread.join(); //等待转发线程完成

    closesocket(socketser);

    WSACleanup();

    system("pause");

    return 0;

}
```

正如代码里面的注释讲解的一样，其实都非常简单，能搞我一天的原因主要也是因为有些逻辑上面没搞通。要单独拎出来说的只有下面这几个点：

阻塞的问题，一定要搞清楚整体的框架，才能保证整个流程的合理运行，使用线程就是为了解决阻塞的问题。send和recv都会阻塞进程，直到收到相应的指令，分开二者就能保证收发的同时运行，也能实现多个客户端的同时服务。

互斥锁的问题，在线程调用到共享资源的时候一定要上锁，上完锁一定要记得解锁，不然容易造成死锁等问题的出现。因为我的程序比较简单，所以就使用了lock\_guard来自动解锁互斥锁。在复杂的程序中可以使用其他的函数来进行上锁和解锁。

还有就是vector函数，STL没学好导致搞个迭代器都搞了半天，在想实现检测信息发送者的时候卡了挺久。

### 客户端

```c++
#include<iostream>
#include<winsock2.h>
#include<string>
#include<thread>
#include<mutex>
#pragma comment (lib, "lws2_32.lib")
using std::cout;
using std::endl;

bool running = TRUE;
std::mutex mtx;

void recvt(SOCKET sockfd) {
    char buf[256];
    int buflen = sizeof(buf);
    int ret = 0;
    while(running) {
        std::lock_guard<std::mutex> lock(mtx);
        memset(buf, 0, buflen);
        ret = recv(sockfd, buf, buflen, 0);
        if(ret < 0) {
            cout << "recv failed, error code: " << WSAGetLastError() << endl;
            continue;
        } else if(ret == 0){
            cout << "server shutdown" << endl;
            break;
        } else {
            cout << "recv: " << buf << endl;
        }
    }
}

int main() {
    WSADATA data;
    WSAStartup(MAKEWORD(2,2), &data);

    SOCKET sockfd = socket(AF_INET, SOCK_STREAM, IPPROTO_IP);
    sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(27015);
    addr.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");

    int ret = connect(sockfd, (sockaddr*)&addr, sizeof(addr));
    if(ret == SOCKET_ERROR) {
        cout << "connect failed, error code: " << WSAGetLastError() << endl;
        return -1;
    }

    std::thread recvthread(recvt, sockfd);

    while(running) {
        std::string msg;
        std::getline(std::cin, msg);

        if(msg == "quit") {
            cout << "exit..." << endl;
            running = FALSE;
            shutdown(sockfd, SD_BOTH);
        } else {
            send(sockfd, msg.c_str(), sizeof(msg), 0);
        }
    }

    recvthread.join();

    closesocket(sockfd);
    WSACleanup();
    system("pause");
    return 0;
}
```

客户端太简单了就不写注释了，有些出现的东西和服务端的也大差不差。服务端倒是挺有意思，阻塞这个问题在这里卡了我巨久。我一开始打算使用线程分开recv和send，但是经过测试发现send放在子线程中，在调用std::getline(std::cin, msg); 时会阻塞recv线程，导致无法接收到服务器转发的信息，解决办法就是将它放到主线程中进行。

## 成果图

![](https://www.kirisamekano.com/image/c-winsock-thread-chat/1.png)

本次实验收获挺多的，最大的感受就是多使用结构体和模板等工具，可以很方便的实现自己想要的功能，像我服务端就用了一个socketforward作为MessageQueue的数据类型，完成了保存消息与消息发送者的功能，在后面转发的时候就能利用到。

计算机网络学习到此为止完美结束，下一步就是CSAPP咯，希望能继续保持学习的热情给它拿下。同时这几个实验的代码其实都能继续优化，像是这一个的代码还能加很多好玩的东西，在等我学完线程池以后再试试重置一下这段代码，将它变成一个高性能的并发服务。
