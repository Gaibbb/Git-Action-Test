---
title: Dockerfile
tags: []
id: '16'
categories:
    - Linux
date: 2023-8-1 11:20:00
---

## Dockerfile
    FROM        # 基础镜像
    MAINTAINER  # 镜像创建者信息
    RUN         # 镜像构建时的运行命令
    ADD         # 添加构建所需文件
    WORKDIR     # 镜像工作目录
    VOLUME      # 容器卷——挂载
    EXPOSE      # 指定暴露端口
    CMD         # 指定容器启动后执行的命令
    # CMD指令可以有多条，但是只有最后一条会生效
    ENTRYPOINT  # 容器启动后用于执行默认应用程序
    ENV         # 用于设置环境变量
    COPY        # 将本地文件或目录复制到容器中，不支持自解压
    USER        # 设置运行后续指令的用户和用户组


## 示例Dockerfile
```dockerfile
    FROM ubuntu:20.04
    MAINTAINER AME<ame@kirisamekano.com>

    RUN     apt-get update && \
            apt-get install -y net-tools && \
            apt-get install -y vim && \
            apt-get install -y nginx

    WORKDIR /app

    ADD     blog.tar /var/www/
    COPY    default /etc/nginx/sites-available/
    COPY    nginx.conf /etc/nginx
    VOLUME  /root/nginx_mount/nginx:/etc/nginx

    EXPOSE  80
    CMD     tail -F /var/log/nginx/access.log
```


## 开始构建镜像
```shell
    root@EzrCym397042:~/dockerfile# docker build -t nginx_test .                                                         [+] Building 0.5s (11/11) FINISHED                                                                    docker:default
    => [internal] load build definition from Dockerfile                                                            0.0s
    => => transferring dockerfile: 407B                                                                            0.0s
    => [internal] load .dockerignore                                                                               0.0s
    => => transferring context: 2B                                                                                 0.0s
    => [internal] load metadata for docker.io/library/ubuntu:20.04                                                 0.4s
    => [1/6] FROM docker.io/library/ubuntu:20.04@sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae2  0.0s
    => [internal] load build context                                                                               0.0s
    => => transferring context: 89B                                                                                0.0s
    => CACHED [2/6] RUN APT-GET update &&  apt-get install -y net-tools &&  apt-get install -y vim &&  apt-get in  0.0s
    => CACHED [3/6] WORKDIR /APP                                                                                   0.0s
    => CACHED [4/6] ADD BLOG.TAR /var/www/                                                                         0.0s
    => CACHED [5/6] COPY DEFAULT /etc/nginx/sites-available/                                                       0.0s
    => CACHED [6/6] COPY NGINX.CONF /etc/nginx                                                                     0.0s
    => exporting to image                                                                                          0.0s
    => => exporting layers                                                                                         0.0s
    => => writing image sha256:376fb40f5780103fe72ac9802295d51ad907b1e72ec0733ffd3a298c99bb2462                    0.0s
    => => naming to docker.io/library/nginx_test                                                                   0.0s



## 容器内部
    root@EzrCym397042:~# docker ps
    CONTAINER ID   IMAGE        COMMAND       CREATED          STATUS          PORTS                  NAMES
    a7f2c99c8bc5   nginx_test   "/bin/bash"   23 seconds ago   Up 22 seconds   0.0.0.0:4000->80/tcp   nginxtest01

    # 可以看见一进来就是在Dockerfile中定义的工作目录  
    root@EzrCym397042:~/dockerfile# docker run --name nginxtest01 -p 4000:80 -it nginx_test /bin/bash
    root@a7f2c99c8bc5:/app#


    # 开启一下nginx服务
    root@1d98584b4c4c:/app# service nginx start
    * Starting nginx nginx
```

![](https:/www.kirisamekano.com/image/docker_file/1.png)

## 镜像信息
```shell
    root@EzrCym397042:~/dockerfile# docker inspect nginx_test
    [
        {
            "Id": "sha256:376fb40f5780103fe72ac9802295d51ad907b1e72ec0733ffd3a298c99bb2462",
            "RepoTags": [
                "nginx_test:latest"
            ],
            "RepoDigests": [],
            "Parent": "",
            "Comment": "buildkit.dockerfile.v0",
            "Created": "2023-08-01T10:58:48.385977106+08:00",
            "Container": "",
            "ContainerConfig": {
                "Hostname": "",
                "Domainname": "",
                "User": "",
                "AttachStdin": false,
                "AttachStdout": false,
                "AttachStderr": false,
                "Tty": false,
                "OpenStdin": false,
                "StdinOnce": false,
                "Env": null,
                "Cmd": null,
                "Image": "",
                "Volumes": null,
                "WorkingDir": "",
                "Entrypoint": null,
                "OnBuild": null,
                "Labels": null
            },
            "DockerVersion": "",
            "Author": "AME<ame@kirisamekano.com>",
            "Config": {
                "Hostname": "",
                "Domainname": "",
                "User": "",
                "AttachStdin": false,
                "AttachStdout": false,
                "AttachStderr": false,
                "ExposedPorts": {
                    "80/tcp": {}
                },
                "Tty": false,
                "OpenStdin": false,
                "StdinOnce": false,
                "Env": [
                    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
                ],
                "Cmd": [
                    "/bin/sh",
                    "-c",
                    "tail -F /var/log/nginx/access.log"
                ],
                "ArgsEscaped": true,
                "Image": "",
                "Volumes": {
                    "/root/nginx_mount/nginx:/etc/nginx": {}
                },
                "WorkingDir": "/app",
                "Entrypoint": null,
                "OnBuild": null,
                "Labels": null
            },
            "Architecture": "amd64",
            "Os": "linux",
            "Size": 310029045,
            "VirtualSize": 310029045,
            "GraphDriver": {
                "Data": {
                    "LowerDir": "/var/lib/docker/overlay2/kpy964hnf41ircnk7h1qxt6sy/diff:/var/lib/docker/overlay2/ylisxlzczml24xjapaox80vby/diff:/var/lib/docker/overlay2/wd36wmoug36c3iyi096scw9dj/diff:/var/lib/docker/overlay2/3h0hpqk5auzmqvpvqg0kf9xw2/diff:/var/lib/docker/overlay2/37ad11f9914e1cee8529967eee3f2f67413e121da49225c6e478293563c749ac/diff",
                    "MergedDir": "/var/lib/docker/overlay2/semacn6d54a1e4m8w7kunvoyg/merged",
                    "UpperDir": "/var/lib/docker/overlay2/semacn6d54a1e4m8w7kunvoyg/diff",
                    "WorkDir": "/var/lib/docker/overlay2/semacn6d54a1e4m8w7kunvoyg/work"
                },
                "Name": "overlay2"
            },
            "RootFS": {
                "Type": "layers",
                "Layers": [
                    "sha256:9f54eef412758095c8079ac465d494a2872e02e90bf1fb5f12a1641c0d1bb78b",
                    "sha256:24a3ff30ae20b49ec044324ab35e2a2e4c3b7c37c1b5cd5ebe1e6131812f3f24",
                    "sha256:04da9cd845e964bd934e78d4f8cb8988c4dded74fcf073325927d45f4a54b21e",
                    "sha256:0f221ca5b6b0304cbbf899a09e9eea21669e7f31d140b879a2fe6779af5dfd64",
                    "sha256:2b573f2aa5c73d920732501fffef5cdda79073d352d5bb64facba58c84108055",
                    "sha256:e987be2c87038b0d72850c7c67578835ed425f12c0a4c5bd0a949e359638e0f4"
                ]
            },
            "Metadata": {
                "LastTagTime": "2023-08-01T11:21:20.060563076+08:00"
            }
        }
    ]
```