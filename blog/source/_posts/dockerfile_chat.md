---
title: Dockerfile部署聊天室实战
tags: []
id: '15'
categories:
    - Linux
date: 2023-8-5 11:40:00
---

## 聊天室代码
这里的聊天室代码是使用了linux下的socket进行重新编程，同时也加了一些新的功能，下面直接把代码放出来。

### 服务端
```cpp
#include<iostream>
#include<mutex>
#include<thread>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<netinet/in.h>
#include<vector>
#include<queue>
#include<errno.h>
#include<cstring>
#include<unistd.h>
using std::vector;
using std::queue;
using std::thread;

struct msg_struct {
    int sockfd;
    std::string msg;
};

struct socket_struct {
    int socketcli;
    std::string cliip;
};

int client_count = 0;
std::mutex mtx;
vector<thread> threads;
vector<socket_struct> socket_vector;
queue<msg_struct> msg_queue;

struct ip_hdr {
    u_char ip_ver:4, ip_hlen:4;
    u_char tos;
    u_short total_len;
    u_short id;
    u_short flags_offset;
    u_char ttl;
    u_char protocol;
    u_short checksum;
    in_addr srcip;
    in_addr desip;
};

std::string get_ip(int sock) {
    for (int i = 0; i < client_count; i++) {
        if (socket_vector[i].socketcli == sock) {
            return socket_vector[i].cliip;
        }
    }
    return "";
}

void forwardmsg() {
    int ret = 0;
    while (true) {
        std::lock_guard<std::mutex> lock(mtx);
        if (msg_queue.empty()) {
            continue;
        } else if (msg_queue.front().msg == "0x02") {
            msg_queue.pop();
            continue;
        }

        int sendsock = msg_queue.front().sockfd;
        std::string send_msg = get_ip(sendsock) + " send: " + msg_queue.front().msg;
        msg_queue.pop();

        const char* msgchar = send_msg.c_str();
        int msglen = send_msg.length();
        //printf("%s", send_msg);
        std::cout << send_msg << "\n";
        for (int i = 0; i < client_count; i++) {
            if(socket_vector[i].socketcli != sendsock) {
                ret = send(socket_vector[i].socketcli, msgchar, msglen, 0);
                if (ret < 0) {
                    printf("send failed, error code: %d", errno);
                } else {
                    printf("%d socket sent success.\n", socket_vector[i].socketcli);
                }
            }
        }
    }
}

void welcome_msg(std::string buf, int clisock) {
    for (int i = 0; i < client_count; i++) {
        if(socket_vector[i].socketcli != clisock) {
            send(socket_vector[i].socketcli, buf.c_str(), sizeof(buf), 0);
        }
    }
}

void bye_msg(std::string byemsg, int clisock) {
    for (int i = 0; i < client_count; i++) {
        if(socket_vector[i].socketcli != clisock) {
            send(socket_vector[i].socketcli, byemsg.c_str(), sizeof(byemsg), 0);
        }
    }
}

void sockethandler(int socketcli) {
    int ret;
    char buf[256];
    int buflen = sizeof(buf);

    while (true) {
        memset(buf, 0, buflen);
        ret = recv(socketcli, &buf, buflen, 0);
        if (ret < 0) {
            printf("recv failed, error code: %d", errno);
            continue;
        } else if (ret > 0) {
            std::lock_guard<std::mutex> lock(mtx);
            printf("recv from %d, message: %s \n", socketcli, buf);
            msg_struct recvmsg;
            recvmsg.msg = buf;
            recvmsg.sockfd = socketcli;
            msg_queue.push(recvmsg);
        } else {
            std::lock_guard<std::mutex> lock(mtx);
            if (msg_queue.front().msg == "0x02" || ret == 0) {
                std::string byemsg = get_ip(socketcli) + " quit...";
                bye_msg(byemsg, socketcli);
            }
            std::vector<socket_struct>::iterator itr = socket_vector.begin();
            while (itr != socket_vector.end()) {
                if ((*itr).socketcli == socketcli) {
                    itr = socket_vector.erase(itr);
                    break;
                } else {
                    ++itr;
                }
            }
            client_count--;
        }
    }
}

int main() {
    int socket_server = socket(AF_INET, SOCK_STREAM, IPPROTO_IP);
    if (socket_server <=0) {
        printf("socket_server created failed, error code: %d", errno);
    }

    sockaddr_in addr_server;
    addr_server.sin_addr.s_addr = INADDR_ANY;
    addr_server.sin_family = AF_INET;
    addr_server.sin_port = htons(11451);
    int addr_server_len = sizeof(addr_server);

    int ref;
    ref = bind(socket_server, (sockaddr*)&addr_server, addr_server_len);
    if (ref < 0)
        printf("socket_server bind faild, error code: %d \n", errno);

    ref = listen(socket_server, 5);
    if (ref < 0)
        printf("socket_server listen failed, error code: %d \n", errno);

    thread forwardthread(forwardmsg);

    int socket_client;
    while (client_count < 5) {
        socket_struct sockfd;
        ref = socket_client = accept(socket_server, (sockaddr*)&addr_server, (socklen_t*)&addr_server_len);
        if (ref <= 0) {
            printf("accept failed, error code: %d \n", errno);
            continue;
        }

        char cliip[INET_ADDRSTRLEN];
        inet_ntop(AF_INET, &addr_server.sin_addr, cliip, INET_ADDRSTRLEN);
        sockfd.socketcli = socket_client;
        sockfd.cliip = cliip;
        socket_vector.push_back(sockfd);
        //printf("connect from: %s, socket num: %d \n", sockfd.cliip, sockfd.socketcli);
        std:: cout << "connect from: " << sockfd.cliip << " , socket num: " << sockfd.socketcli << std::endl;
        std::string welcome = sockfd.cliip + " connect to the server.";
        welcome_msg(welcome, sockfd.socketcli);
        
        threads.emplace_back([socket_client]() {sockethandler(socket_client);});
        client_count++;
    }

    for (auto& thread : threads)
        thread.join();

    close(socket_server);
    return 0;
}
```

### 客户端
```cpp
#include<iostream>
#include<sys/socket.h>
#include<mutex>
#include<netinet/in.h>
#include<thread>
#include<unistd.h>
#include<arpa/inet.h>
#include<error.h>
#include<cstring>

std::mutex mtx;
bool running = 0;
std::string quitmsg = "0x02";
int quitmsglen = sizeof(quitmsg);

void recvmsg(int sockfd) {
    char buf[256];
    int buflen = sizeof(buf);
    int ret = 0;

    while (running) {
        memset(buf, 0, buflen);
        ret = recv(sockfd, buf, buflen, 0);
        if (ret < 0) {
            printf("recv failed, error code: %d", errno);
            continue;
        } else if (ret > 0) {
            std::lock_guard<std::mutex> lock(mtx);
            std::cout << buf << "\n";
        } else {
            printf ("server shutdown");
            break;
        }
    }
}

int main() {
    int sockfd = socket(AF_INET, SOCK_STREAM, IPPROTO_IP);

    sockaddr_in addr;
    addr.sin_addr.s_addr = inet_addr("43.226.41.70");
    addr.sin_family = AF_INET;
    addr.sin_port = htons(11451);

    int ret = connect(sockfd, (sockaddr*)&addr, sizeof(addr));
    if (ret < 0) {
        printf ("connect failed, error code: %d", errno);
        return 0;
    } else {
        running = 1;
    }

    std::thread recvthread([sockfd]() {recvmsg(sockfd);});

    while (running) {
        std::string msg;
        std::getline(std::cin, msg);

        if (msg == "quit") {
            printf("exit... \n");
            running = 0;
            recvthread.detach();
            send(sockfd, quitmsg.c_str(), quitmsglen, 0);
            shutdown(sockfd, SHUT_RDWR);
            break;
        } else {
            send(sockfd, msg.c_str(), sizeof(msg), 0);
        }
    }

    close(sockfd);
    return 0;
}
```

## Dockerfile

也没什么好解释的，直接上代码：
```Dockerfile
FROM ubuntu:20.04
MAINTAINER Ame<ame@kirisamekano.com>

WORKDIR /cpp
RUN     apt-get update && \
        apt-get install -y build-essential

COPY    server.cpp /cpp
EXPOSE  11451

RUN     ["g++", "server.cpp", "-o", "server", "-pthread"]

CMD     ["./server"]
```



```bash shell
root@EzrCym397042:~/cpp/server_docker# ls
Dockerfile  rm.sh  server.cpp
root@EzrCym397042:~/cpp/server_docker# docker build -t server_test .
[+] Building 15.5s (10/10) FINISHED                                                                   docker:default
 => [internal] load build definition from Dockerfile                                                            0.0s
 => => transferring dockerfile: 278B                                                                            0.0s
 => [internal] load .dockerignore                                                                               0.0s
 => => transferring context: 2B                                                                                 0.0s
 => [internal] load metadata for docker.io/library/ubuntu:20.04                                                15.4s
 => [1/5] FROM docker.io/library/ubuntu:20.04@sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae2  0.0s
 => [internal] load build context                                                                               0.0s
 => => transferring context: 32B                                                                                0.0s
 => CACHED [2/5] WORKDIR /cpp                                                                                   0.0s
 => CACHED [3/5] RUN APT-GET update &&  apt-get install -y build-essential                                      0.0s
 => CACHED [4/5] COPY SERVER.CPP /cpp                                                                           0.0s
 => CACHED [5/5] RUN ["G++", "server.cpp", "-o", "server", "-pthread"]                                          0.0s
 => exporting to image                                                                                          0.0s
 => => exporting layers                                                                                         0.0s
 => => writing image sha256:7827735ac95fea67c702a28bc55d92fdfd1a78aaf96a08b676db2405b5cf527a                    0.0s
 => => naming to docker.io/library/server_test                                                                  0.0s
root@EzrCym397042:~/cpp/server_docker# docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
client_test   latest    842ce04e6d29   37 minutes ago   394MB
server_test   latest    7827735ac95f   52 minutes ago   394MB
nginx_test    latest    376fb40f5780   4 days ago       310MB
mysql         latest    7c5ae0d3388c   11 days ago      577MB
nginx         latest    021283c8eb95   4 weeks ago      187MB
ubuntu        latest    5a81c4b8502e   5 weeks ago      77.8MB
root@EzrCym397042:~/cpp/server_docker# docker run --name server -p 11451:11451 -d server_test
6516ce7cc009324a6aad865e351ab106d0a2752824e39e29a9abab90ed925795
root@EzrCym397042:~/cpp/server_docker# docker ps
CONTAINER ID   IMAGE         COMMAND      CREATED         STATUS         PORTS                      NAMES
6516ce7cc009   server_test   "./server"   7 seconds ago   Up 6 seconds   0.0.0.0:11451->11451/tcp   server
```

## 测试
用了一个本地和另外一台服务器作为客户端测试能否正常使用。

![](https://kirisamekano.com/image/dockerfile_chat/1.png)

## 推送至Dockerhub
```bash shell
# 改个tag 将"/"前面的名字改成自己的Dockerhub Account
root@EzrCym397042:~# docker tag server_test:latest gaibbb/chat_server:1.0
root@EzrCym397042:~# docker push gaibbb/chat_server:1.0
The push refers to repository [docker.io/gaibbb/chat_server]
88e2a99b3daf: Pushed
4aa195e0add7: Pushed
4a2f57ece75c: Pushed
2d43dec47e69: Pushed
9f54eef41275: Pushed
1.0: digest: sha256:e6535764cdb1a3358db7dd52959167d417be5db0505dfabe325a5d0fb7172b8a size: 1366
```

![](https://kirisamekano.com/image/dockerfile_chat/2.png)
