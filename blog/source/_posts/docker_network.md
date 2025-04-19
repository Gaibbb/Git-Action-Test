---
title: Docker Network
tags: []
id: '17'
categories:
    - Linux
date: 2023-8-1 18:00:00
---

## 查看本地网络
```shell
    root@EzrCym397042:~/dockerfile# ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 00:0c:29:42:87:7d brd ff:ff:ff:ff:ff:ff
        inet 43.226.41.70/26 brd 43.226.41.127 scope global eth0
        valid_lft forever preferred_lft forever
        inet6 fe80::20c:29ff:fe42:877d/64 scope link
        valid_lft forever preferred_lft forever
    3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 00:0c:29:42:87:87 brd ff:ff:ff:ff:ff:ff
        inet 10.226.41.70/26 brd 10.226.41.127 scope global eth1
        valid_lft forever preferred_lft forever
        inet6 fe80::20c:29ff:fe42:8787/64 scope link
        valid_lft forever preferred_lft forever
    4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
        link/ether 02:42:ab:d4:1e:c9 brd ff:ff:ff:ff:ff:ff
        inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
        valid_lft forever preferred_lft forever
    204: vethfb58142@if203: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
        link/ether b6:24:52:29:5f:bb brd ff:ff:ff:ff:ff:ff link-netnsid 0

    # 可以看到一个Docker0的网卡，这是Docker的默认网络
```
## Docker网络类型

### bridge网络(默认)
Bridge——Docker默认的网络驱动，每当创建一个容器的时候就会为容器创建一个隔离的网络命名空间，并为容器创建一个名为“bridge”的虚拟网络接口。容器之间通过这个接口进行连接，通过Docker0进行相互连接（--link）。
```shell
    root@EzrCym397042:~/dockerfile# docker ps
    CONTAINER ID   IMAGE        COMMAND       CREATED       STATUS       PORTS                  NAMES
    a7f2c99c8bc5   nginx_test   "/bin/bash"   5 hours ago   Up 5 hours   0.0.0.0:4000->80/tcp   nginxtest01
    root@EzrCym397042:~/dockerfile# docker network ls
    NETWORK ID     NAME      DRIVER    SCOPE
    0e6ec526534f   bridge    bridge    local
    597cd13e2c3f   host      host      local
    0481f5ec9480   none      null      local
    root@EzrCym397042:~/dockerfile# docker network inspect 0e6ec
    [
        {
            "Name": "bridge",
            "Id": "0e6ec526534ff06868f74119f69126034a9669ff6e4654af3a1e30397ff59589",
            "Created": "2023-07-31T17:16:25.06320541+08:00",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": null,
                "Config": [
                    {
                        "Subnet": "172.17.0.0/16",
                        "Gateway": "172.17.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {
                "a7f2c99c8bc519b04ea9e41983c5b454de7024062b060ea0f2bce5714a10bfd8": {
                    "Name": "nginxtest01",
                    "EndpointID": "31770cca6186dfef018cb7cecc9bb7d7d1760985546daf5790e320a7e0d38749",
                    "MacAddress": "02:42:ac:11:00:02",
                    "IPv4Address": "172.17.0.2/16",
                    "IPv6Address": ""
                }
            },
            "Options": {
                "com.docker.network.bridge.default_bridge": "true",
                "com.docker.network.bridge.enable_icc": "true",
                "com.docker.network.bridge.enable_ip_masquerade": "true",
                "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
                "com.docker.network.bridge.name": "docker0",
                "com.docker.network.driver.mtu": "1500"
            },
            "Labels": {}
        }
    ]

    root@EzrCym397042:~/dockerfile# docker exec a7f2c99c8bc5 ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
    203: eth0@if204: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
        link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
        inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
        valid_lft forever preferred_lft forever

    # 创建容器时，使用默认的设置会出现"203:eth0@if204"，通过veth-pair虚拟网卡对进行容器内和外部网络网络连接。
    # 会有一个Docker0作为路由器一样的东西对容器的网络进行管理，Docker0使用NAT让容器与主机进行网络连接。
```

### host网络
容器和主机共享一个网络命名空间，这意味着容器的端口会占用实际主机的端口。
```shell
    root@EzrCym397042:~/dockerfile# docker network inspect 597cd
    [
        {
            "Name": "host",
            "Id": "597cd13e2c3f1d82873468baa4d5c870a7cc9d52ab83da0c71d32c9fc0bb6c3b",
            "Created": "2023-06-27T18:11:33.435536811+08:00",
            "Scope": "local",
            "Driver": "host",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": null,
                "Config": []
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {},
            "Options": {},
            "Labels": {}
        }
    ]
```

### none网络
这会使容器无法与外部网络进行通信，可以以此创建一个与网络完全隔离的容器。
```shell
    root@EzrCym397042:~/dockerfile# docker network inspect 0481f5
    [
        {
            "Name": "none",
            "Id": "0481f5ec9480d8fe0ab94f9a6ca78582b699a50eb4dc214d3ae4a4767a6aed0b",
            "Created": "2023-06-27T18:11:33.410690195+08:00",
            "Scope": "local",
            "Driver": "null",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": null,
                "Config": []
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {},
            "Options": {},
            "Labels": {}
        }
    ]
```

### overlay网络与macvlan网络
没有实际用到，这里暂时先不做记录。  
overlay网络用于多个Docker主机间的跨主机容器网络，适用于容器分布式应用。     
macvlan网络允许为容器配置虚拟网卡的mac地址，这对需要与外部网络进行直接通信的容器来说很有用。


## docker network指令

### 指令介绍
```shell
    root@EzrCym397042:~/dockerfile# docker network -help
    unknown shorthand flag: 'e' in -elp
    See 'docker network --help'.

    Usage:  docker network COMMAND

    Manage networks

    Commands:
    connect     Connect a container to a network    连接一个容器到一个网络
    create      Create a network    创建一个网络
    disconnect  Disconnect a container from a network   断开一个容器和一个网络
    inspect     Display detailed information on one or more networks    显示网络的详细信息
    ls          List networks   列出已有网络
    prune       Remove all unused networks  移除所有没被使用的网络
    rm          Remove one or more networks 删除一个或多个网络

    Run 'docker network COMMAND --help' for more information on a command.
```

### 示例——创建一个自己的容器网络并连接多个容器

#### 创建网络
```shell
    root@EzrCym397042:~/dockerfile# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 test_network
    cde3254e2c84910cad938a4d5dd786519c30f507cd6b8cc81832ee1fd01890a6
    # --driver  网络类型
    # --subnet  子网
    # --gateway 网关

    root@EzrCym397042:~/dockerfile# docker network ls
    NETWORK ID     NAME           DRIVER    SCOPE
    0e6ec526534f   bridge         bridge    local
    597cd13e2c3f   host           host      local
    0481f5ec9480   none           null      local
    cde3254e2c84   test_network   bridge    local
    root@EzrCym397042:~/dockerfile# docker inspect cde3254
    [
        {
            "Name": "test_network",
            "Id": "cde3254e2c84910cad938a4d5dd786519c30f507cd6b8cc81832ee1fd01890a6",
            "Created": "2023-08-01T17:11:38.541215579+08:00",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "192.168.0.0/16",
                        "Gateway": "192.168.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {},
            "Options": {},
            "Labels": {}
        }
    ]
```

#### 创建容器并连接网络
```shell
    # 使用bash shell脚本创建docker
    root@EzrCym397042:~/dockerfile# cat docker_create.sh
    #~/bin/bash

    for((i=0;i<4;i++)); do
            docker run --name network_test$i -p `expr $i + 5000`:80 --network test_network -d nginx
    done


    root@EzrCym397042:~/dockerfile# ./docker_create.sh
    1c2c0b07761008a005ec3b83097aef448fc26b7ae965c4f26fd88bc4d7e2f935
    7b00e538689ebecf57eee9f327f6752b9f93f33e7c06d2fbd1b3d43dac7ea5f5
    0170b67f68615d2ac0e92014c83f5c0dc6a3c996a4473e9d3b3514891a1fe017
    44df6ce789acd73b89066d04ae574ef3755c66b742bebc6955edbdaf5e9e2ea1
    root@EzrCym397042:~/dockerfile# docker ps
    CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                  NAMES
    44df6ce789ac   nginx     "/docker-entrypoint.…"   9 seconds ago    Up 8 seconds    0.0.0.0:5003->80/tcp   network_test3
    0170b67f6861   nginx     "/docker-entrypoint.…"   10 seconds ago   Up 9 seconds    0.0.0.0:5002->80/tcp   network_test2
    7b00e538689e   nginx     "/docker-entrypoint.…"   10 seconds ago   Up 9 seconds    0.0.0.0:5001->80/tcp   network_test1
    1c2c0b077610   nginx     "/docker-entrypoint.…"   11 seconds ago   Up 10 seconds   0.0.0.0:5000->80/tcp   network_test0
    root@EzrCym397042:~/dockerfile# docker network inspect cde3254
    [
        {
            "Name": "test_network",
            "Id": "cde3254e2c84910cad938a4d5dd786519c30f507cd6b8cc81832ee1fd01890a6",
            "Created": "2023-08-01T17:11:38.541215579+08:00",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "192.168.0.0/16",
                        "Gateway": "192.168.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Ingress": false,
            "ConfigFrom": {
                "Network": ""
            },
            "ConfigOnly": false,
            "Containers": {
                "0170b67f68615d2ac0e92014c83f5c0dc6a3c996a4473e9d3b3514891a1fe017": {
                    "Name": "network_test2",
                    "EndpointID": "b4111842dce1ee5b88d9b49eca09783156a836b31829afd9fd4a334f3e9ec7b3",
                    "MacAddress": "02:42:c0:a8:00:04",
                    "IPv4Address": "192.168.0.4/16",
                    "IPv6Address": ""
                },
                "1c2c0b07761008a005ec3b83097aef448fc26b7ae965c4f26fd88bc4d7e2f935": {
                    "Name": "network_test0",
                    "EndpointID": "1bd43ba7f2b5c89b47535f7b58ba7ed419f598289d10c020d880f89e2313ebd8",
                    "MacAddress": "02:42:c0:a8:00:02",
                    "IPv4Address": "192.168.0.2/16",
                    "IPv6Address": ""
                },
                "44df6ce789acd73b89066d04ae574ef3755c66b742bebc6955edbdaf5e9e2ea1": {
                    "Name": "network_test3",
                    "EndpointID": "1f0d76c2ea1214fb3d81f62cb6afe935571d229cb9b7629877a6ba89acf4dab2",
                    "MacAddress": "02:42:c0:a8:00:05",
                    "IPv4Address": "192.168.0.5/16",
                    "IPv6Address": ""
                },
                "7b00e538689ebecf57eee9f327f6752b9f93f33e7c06d2fbd1b3d43dac7ea5f5": {
                    "Name": "network_test1",
                    "EndpointID": "8a439b3bc9766e94d38d559a3ea356d45ba8c41c0e279b0c87d19538f3319dae",
                    "MacAddress": "02:42:c0:a8:00:03",
                    "IPv4Address": "192.168.0.3/16",
                    "IPv6Address": ""
                }
            },
            "Options": {},
            "Labels": {}
        }
    ]
```

可以看到刚刚创建的每一个容器都在新创建的test_network网络中了，他们的IPv4地址也是在刚刚创建的网段中。    
使用这种方法可以为每一种容器创建自己的网络，方便管理。

### docker network connect
先创建一个使用默认网络驱动的新容器，然后将其连接到我们刚刚创建的网络中。
```shell
    root@EzrCym397042:~/dockerfile# docker run --name network_test5 -p 5005:80 -d nginx
    #然后docker inspect一下看一下它的网络
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "0b5ee899dbe20782242c562501fe1a71c8cec65b0bc0e0f1f38de030c3849801",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "5005"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/0b5ee899dbe2",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "4e4d69530241c8d53108b74660b7ee88efb111173fc1a4c1cb6e07c59be8696f",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "0e6ec526534ff06868f74119f69126034a9669ff6e4654af3a1e30397ff59589",
                    "EndpointID": "4e4d69530241c8d53108b74660b7ee88efb111173fc1a4c1cb6e07c59be8696f",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }


    # 还是在使用默认的"172.17.0.2"，然后把它添加到test_network中
    root@EzrCym397042:~/dockerfile# docker network connect test_network 76895cc01

    # inspect一下容器
    "Networks": {
        "bridge": {
            "IPAMConfig": null,
            "Links": null,
            "Aliases": null,
            "NetworkID": "0e6ec526534ff06868f74119f69126034a9669ff6e4654af3a1e30397ff59589",
            "EndpointID": "4e4d69530241c8d53108b74660b7ee88efb111173fc1a4c1cb6e07c59be8696f",
            "Gateway": "172.17.0.1",
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "MacAddress": "02:42:ac:11:00:02",
            "DriverOpts": null
        },
        "test_network": {
            "IPAMConfig": {},
            "Links": null,
            "Aliases": [
                "76895cc0185c"
            ],
            "NetworkID": "cde3254e2c84910cad938a4d5dd786519c30f507cd6b8cc81832ee1fd01890a6",
            "EndpointID": "5992c602026c7cae939c059a2ecf220469130d83abd208700dbd9a325fc37cc4",
            "Gateway": "192.168.0.1",
            "IPAddress": "192.168.0.6",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "MacAddress": "02:42:c0:a8:00:06",
            "DriverOpts": {}
        }
    }
    # 可以看到网络里面已经添加了一个"test_network"，此时它可以同时访问默认网络以及"test_network"中的主机。
```

### --link
--link是一个在创建容器时可以将该容器连接到另外一个容器的附加指令。    
```shell
    root@EzrCym397042:~/dockerfile# docker run --name network_test6 -p 5006:80 --link network_test5 -d nginx
    b867888bbd4ec9e66e1777d337b408c1edc3f9b4874fb9bd8b1368619978b071

    root@EzrCym397042:~/dockerfile# docker exec network_test6 cat /etc/hosts
    127.0.0.1       localhost
    ::1     localhost ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    172.17.0.2      network_test5 76895cc0185c
    172.17.0.3      b867888bbd4e
```
从中可以看到它采用了静态连接的方式在hosts文件中添加了对network_test5的解析。这种方式已经落后，一般不采用该方式进行容器间的连接。