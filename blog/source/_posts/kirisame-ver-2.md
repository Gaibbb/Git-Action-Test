---
title: KirisAme Ver.2
tags: []
id: '5'
categories:
  - - 'Linux'
date: 2022-02-04 00:30:30
---

相信看完标题的时候你也大概知道这篇文章要讲什么了！

我的网站终于是迎来了二周目版本！

全新的访问域名与访问体验，也是抓着旧服务器准备过期的日子搞出来了这个Ver.2。旧的服务器我想丢很久了，因为以前刚入坑不太认识这里面的门道，买了个上海的服务器，结果就导致了域名搞了大半年都没套上IP，国内的域名审查真的是很严格😭。早就想换一个境外的服务器套上自己的域名了！这次刚好国内手里拿了点红包钱，顺便买了个香港的服务器和域名就开始折腾上了，相较于第一次建站真的是成熟不少进步颇大！

在还没搞出Ver.2之前我一直不知道自己原来学习到了这么多东西，熟悉我的人都知道我是一个非常自卑的人，每次网络上面看到别的大佬的作品都感觉自己什么都不是，拿自己的学识和别人对比一下感觉自己真的很无知hhh。但是今天建站的过程中，我能很快的安装完环境，然后导入数据库，Copy一份网站到www目录下。这里就顺便做个笔记，如何迁移wordpress小站。

我的做法可能比较傻，工程量看上去还是蛮大的，但是并不会花费太长的时间（如果服务器带宽够大的话）。首先是Copy一份网站下来备用，我选择的是先压缩在传输，毕竟旧服务器带宽3M，看着300kb的传输速度我真的难受。

```
tar -zcvf html.tar.gz /var/www/html
```

接下来我是用了FTP工具下载到本地的目录了，其实如果嫌他太慢自己又已经搭建好了rsync的话可以先把文件同步到带宽较好的服务器上面再进行传输。关于rsync的使用我会在下一篇文章（也许）中再做笔记。网站的文件部分已经解决了，下一部分是数据库的备份，我用到的数据库是Mysql8.0，其实通过一次站点迁移就能理解一部分wordpress的工作原理，像是我第一次迁移过去很天真的以为只需要Copy网站文件过去就好了，搞到最后发现只有一个首页的皮，里面文章那些全都是点不开的，而且文章的封面图片是和数据库挂钩的。

```
mysqldump -u root -p wordpree > /root/wordpress.sql
```

接下来只要输入密码就ok了，这一步还是比较简单的。

搞定了旧服务器的部分，接下来就到了新服务器了，首先安装Mysql、PHP和Httpd，由于这次建站我用的是Ubuntu作为新系统，还是有必要做一下相应的笔记。

```
apt-get update
apt -y install mysql mysql-devel mysql-server php php-fpm apache2
systemctl start mysqld
systemctl start apache2
```

我觉得比较不适应的一点是Centos8里面还是httpd的跑到Ubuntu里面变成了apache2，虽然都是一个东西，但是还是httpd比较好找hh。至于为什么没有启动php这一点，我也还是一个很懵逼的状态，他自己安装完以后貌似是自己启动了，我无论是输入php-fpm还是php7.4-fpm都提示我找不到这个服务，就这个比较奇怪一点，还得研究一下到底怎么回事。

接下来就非常轻松了，先进入mysql设置一下密码与创建wordpress数据库，方便等等导入。在mysql8.0中，设置密码有了新的指令。

```
ALTER USER 'root'@'localhost'
IDENTIFIED WITH mysql_native_password
BY 'password';
```

设置完密码后导入数据库文件

```
mysql -u root -p
> create database wordpress;
> use database wordpress;
> source /root/wordpress.sql
```

```
mv html.tar.gz /var/www/
rm -r html #删除原来的html文件夹
tar -zxvf html.tar.gz
```

问题大头已经基本解决了，接下来是一些细节的部分。在刚搞好以后进入网站发现已经可以正常访问了，但是有一个问题，点进去的页面会点不开，这时候再看看url地址发现是以前服务器的url，这个时候怎么办呢，两种解决办法。

第一种是建立在能进入wp-admin页面的基础上的，这个比较简单，进入管理后台以后，点击设置->常规，就可以看到"wordpress地址(URL)"和"站点地址(URL)"，分别将里面的url改成自己当前的网址就好了。

第二种方法比较麻烦，我也是实在进不去后台以后才想到的这招，直接进入到数据库中修改相关文件。

```
mysql-u root -p
> use wordpress;
> select * from wp_options;
```

可以看到在wp\_options里面有这么两个东西

![图片噶了](https://www.kirisamekano.com/wp-content/uploads/2022/02/mysql-name.png)

再往下看就可以看到option\_name和option\_value两个字段名下的values

![图片噶了](https://www.kirisamekano.com/wp-content/uploads/2022/02/mysql-values-1024x164.png)

接下来只需要用到一点点简单的sql语句就能结束了

```
> update wp_options option_value = "https://www.kirisamekano.com" where option_name = "siteurl";
> update wp_options option_value = "https://www.kirisamekano.com" where option_name = "home";
```

接下来应该就能正常访问所有的页面了，但是还有一个很重要的东西，图片的地址还没改呢，这个也比较简单，回到文章编辑页面修改一下图片的url就可以了，这里就不做演示了。

那么站点迁移也是差不多搞定了，由于是一个新的域名，我又重新申请了一个SSL证书，这里也顺便做做笔记吧！这边我用的是腾讯云的证书，先在产品里面搜索SSL证书，然后在“我的证书”中找到申请证书，绑定想要的域名就好了，接着就根据自己是nginx还是apache下载相关的证书，或者是wget直接下载到服务器上面。

我用的是apache2，所以我就只针对这个来讲啦，结合腾讯云自己给的文档的，但是他没给ubuntu的，于是我又看了隔壁阿里云的文档哈哈哈哈。

```
mkdir /etc/apache2/ssl #首先创建一个ssl文件夹用来放证书文件
unzip 证书名.zip #解压到刚刚创建的ssl文件夹里面
sudo a2enmod ssl #启动ssl模块

vim /etc/apache2/sites-available/default-ssl.conf #编辑文件修改以下内容

<IfModules mod_ssl.c>
<VirtualHost *:443>  
ServerName 域名 #如果没有的话要加上这个
SSLCertificateFile /etc/apache2/ssl/YourDomainName_public.crt
SSLCertificateKeyFile /etc/ssl/apache2/YourDomainName.key
SSLCertificateChainFile /etc/apache2/ssl/YourDomainName_chain.crt
#将文件名改成你证书的名字就好，我这边一开始是没有SSLCertificateChainFile这个的，所以还要自己把他加上去

sudo ln -s /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-enabled/001-ssl.conf #建立软连接


systemctl restart apache2 #重启apache2服务
```

到这里应该就差不多了，阿里云的文档还有一个是要重新加载apache2的配置文件，我反正没用到都可以，下面也顺便放出来一下吧，在重启apache2服务之前用。

```
sudo /etc/init.d/apache2 force-reload
```

这个到这里也顺利解决啦！新站的迁移到此结束，但是学习的路还不能停下，最近学习Mirai机器人的部署那些搞的头很大，又顺便想重新拾起C++来，于是选择了MiraiCP这个开发API，但是发现自己连部署开发环境都不是很清楚，就弄得很头大，这之中有用到JDK和Cmake的安装和使用，详细的我也会在以后的文章中发出来，虽然可能只有我这种笨比会遇到这种坑hh。由于MiraiCP的文档真的很少，所以我还是发出来帮助一下有需要的人。部署完了以后又不会用了，发现我对C++的开发确实还没有学到多少，学习可谓是道阻且长啊。突然发现真的有好多东西要学，大学的专业也不是自己喜欢的，所有的这些成果啊，想法之类的都要我自己自学来实现，顿时又觉得很孤独，只能说非常的痛苦了。我也不是什么出身名门贵族啊，我还是很担心自己的未来，所以才猛地在学习这些东西，不想让自己的未来太痛苦。但是又因为无钱钱的原因不想找一些课来上，自己就看CSDN，看B站，买几本书自己看。学习编程这个过程说枯燥和有趣都是真的，对于学习Linux这些东西我就很有专研精神，但是单学C++有的时候会觉得很无聊，我感觉还是得有个项目比较能激发专研精神吧。希望新的一年自己能拿下C++的大部分，也不要求完全学会，毕竟还有数据结构那些东西好多好多要学，希望大家新的一年也能实现自己的愿望吧！

这篇文章就到这里了，新年快乐！多去陪陪家人和朋友，别像我一样除夕夜一个人在敲代码搞些有的没的。加油！！！
