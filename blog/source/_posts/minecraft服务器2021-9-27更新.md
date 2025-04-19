---
title: Minecraft服务器2021.9.27更新
tags: []
id: '2'
categories:
  - - 'Game'
date: 2021-09-27 22:59:31
---

这次更新是在一个很特殊的环境下搞好的，属于是一整天满课，从早八上到晚上九点钟没停过。正好想试试平板加JuiceSSH掉头发效率有多高，就随随便便搞了两个更新，在这里做一下笔记以及说明一下更新内容捏~

首先是更新内容：本次更新修复好了几百年没碰过的皮肤插件（下次不知道什么时候又崩xD）以及将登陆插件数据连接到了Mysql云数据库上面，取代了本地数据存储。

本次更新笔记大多数都是在登陆插件放到Mysql上面：

①Mysql得开放对外连接，把localhost换成%

```
update user set host = '%' where user = 'root' and host = 'localhost';
```

```
grant all on *.* to 'root'@'%';
```

```
flush privileges;
```

②AuthMe设置

```
vim Minecraf server/plugins/AuthMe/config.yml;
```

```
DataSource:
    # What type of database do you want to use?
    # Valid values: SQLITE, MYSQL, POSTGRESQL
    backend: MYSQL  //调成MYSQL
    # Enable the database caching system, should be disabled on bungeecord environments
    # or when a website integration is being used.
    caching: true
    # Database host address
    mySQLHost: 'Mysql服务器地址'
    # Database port
    mySQLPort: '3306' //一般mysqld服务都是开这个端口
    # Connect to MySQL database over SSL
    mySQLUseSSL: false //我这边是要关了这个才能用
    # Verification of server's certificate.
    # We would not recommend to set this option to false.
    # Set this option to false at your own risk if and only if you know what you're doing
    mySQLCheckServerCertificate: false  //我这边是要关了这个才能用
    # Username to connect to the MySQL database
    mySQLUsername: 数据库用户名
    # Password to connect to the MySQL database
    mySQLPassword: '数据库密码'
    # Database Name, use with converters or as SQLITE database name
    mySQLDatabase: authme
    # Table of the database
    mySQLTablename: authme
```

当然了，上面还存在着两个安全选项关闭的问题，接下来我也会继续研究要怎么去搞定这个问题，保证服务器的安全。

谢谢你的观看！
