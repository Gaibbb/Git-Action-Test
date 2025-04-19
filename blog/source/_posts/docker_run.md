---
title: Docker Run
tags: []
id: '15'
categories:
    - Linux
date: 2023-7-231 11:40:00
---

## docker run
重点只记一下挂载部分，其他的都比较好理解，就不展开说了。( -e 环境配置放到下次再说)    

    docker run
    -d 后台运行
    -p 端口映射——本地端口:容器端口
    -v 挂载——本地地址:容器地址
    -e 环境配置
    --name 容器名字

### 普通挂载
```shell
    # 例子
    root@EzrCym397042:~# docker run --name nginx01 -v /root/blog:/home/nginx -it nginx /bin/bash
    root@6c49aa48e425:~# ls /home/nginx
    _config.landscape.yml  db.json       package-lock.json  scaffolds  themes
    _config.yml            node_modules  package.json       source

    # 打开另外一个ssh连接
    root@EzrCym397042:~# ls /root/blog
    _config.landscape.yml  db.json       package.json       scaffolds  themes
    _config.yml            node_modules  package-lock.json  source

    # 在一个ssh连接下在本地主机创建一个文本文件abc.txt
    root@EzrCym397042:~# echo "Hello, world" >> /root/blog/abc.txt

    # 查看docker容器内的/home/blog目录是否出现abc.txt
    root@6c49aa48e425:/home/nginx# ls
    _config.landscape.yml  abc.txt  node_modules       package.json  source
    _config.yml            db.json  package-lock.json  scaffolds     themes
    root@6c49aa48e425:/home/nginx# cat abc.txt
    Hello, world

    # 然后在docker容器内创建一个123.txt
    root@6c49aa48e425:/home/nginx# echo "Hello" >> 123.txt

    # 在本地主机查看是否存在123.txt
    root@EzrCym397042:~/blog# ls
    123.txt  _config.landscape.yml  db.json       package.json       scaffolds  themes
    abc.txt  _config.yml            node_modules  package-lock.json  source
    root@EzrCym397042:~/blog# cat 123.txt
    Hello

    # 在这种挂载方式下，本地主机挂载点和容器内的挂载会相互影响。
    # 挂载指令很清晰，将容器内的哪个点和主机的哪个点做连接。
```

### 只读挂载
```shell
    # 在上面挂载指令的基础上添加ro或rw（默认挂载模式）
    root@EzrCym397042:~# docker run --name nginx02 -v /root/blog/:/home/nginx:ro -it nginx /bin/bash
    root@b8c887afb574:/home/nginx# echo "Hello" >> /home/nginx/abc.txt
    bash: abc.txt: Read-only file system
```
显示无法在挂载的目录下创建abc.txt，是一个只读文件系统。     
在该模式下只有主机能在挂载点创建文件给容器内使用，容器内无法在挂载目录创建文件。


### 具名挂载和匿名挂载
```shell
    # 具名挂载——在挂载目录的时候指定挂载名  -v 卷名:容器路径
    root@EzrCym397042:~# docker run -v nginx_blog:/home/nginx --name nginx03 -it nginx /bin/bash
    root@EzrCym397042:~# docker volume ls
    DRIVER    VOLUME NAME
    local     ab49bb971b4c9b4b37fb554fdc0dd1a7a668cb314a692eb740d34447bbc9cd05
    local     mount_blog
    local     nginx_blog
    root@EzrCym397042:~# docker volume inspect nginx_blog
    [
        {
            "CreatedAt": "2023-07-31T11:24:56+08:00",
            "Driver": "local",
            "Labels": null,
            "Mountpoint": "/var/lib/docker/volumes/nginx_blog/_data",
            "Name": "nginx_blog",
            "Options": null,
            "Scope": "local"
        }
    ]

    # 然后我们进入Mountpoint这个路径查看并创建一个123.txt文本
    root@EzrCym397042:/var/lib/docker/volumes/nginx_blog/_data# echo "Hello, world" >> 123.txt
    root@EzrCym397042:/var/lib/docker/volumes/nginx_blog/_data# ls
    123.txt

    # 进入容器中查看是否存在
    root@754ad652da13:~# ls /home/nginx/
    123.txt



    # 匿名挂载——不指定挂载名    -v 容器路径
    root@EzrCym397042:~# docker run --name nginx04 -v /home/nginx -it nginx /bin/bash
    root@EzrCym397042:~# docker volume ls
    DRIVER    VOLUME NAME
    local     ab49bb971b4c9b4b37fb554fdc0dd1a7a668cb314a692eb740d34447bbc9cd05
    local     c40fc317631c6e478b8d7f92bb0b5a2a2ffa3184ee30de30326af0346bf4f04f
    local     mount_blog
    local     nginx_blog
    root@EzrCym397042:~# docker volume inspect c40fc317631c6e478b8d7f92bb0b5a2a2ffa3184ee30de30326af0346bf4f04f
    [
        {
            "CreatedAt": "2023-07-31T11:29:52+08:00",
            "Driver": "local",
            "Labels": null,
            "Mountpoint": "/var/lib/docker/volumes/c40fc317631c6e478b8d7f92bb0b5a2a2ffa3184ee30de30326af0346bf4f04f/_data",
            "Name": "c40fc317631c6e478b8d7f92bb0b5a2a2ffa3184ee30de30326af0346bf4f04f",
            "Options": null,
            "Scope": "local"
        }
    ]
```
用docker volume ls可以看到这次VOLUME NAME自动创建了一个很长的唯一ID给我们刚刚挂载的容器内目录。     
这种方法一般只用于短时间的数据共享，一般建议使用具名挂载进行目录的挂载。
