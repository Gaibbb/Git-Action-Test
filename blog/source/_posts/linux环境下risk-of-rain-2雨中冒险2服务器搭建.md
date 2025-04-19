---
title: Linux环境下Risk of rain 2(雨中冒险2)服务器搭建
tags: []
id: '7'
categories:
  - - 'Linux'
date: 2022-03-10 00:30:10
---

最近雨中冒险2出了新虚空DLC，有个朋友的轻薄本刚好能跑，于是乎就入手了一个开玩，但是他发现以他校园网那种渣渣网络根本不可能和好友愉快的联机，第一天还笑嘻嘻的拉人开房爽玩，第二天就房间都进不去了hhh。突然想到以前还有一个潜渊症的群服务器闲置着，就想用来建一个雨中冒险2的服务器，不浪费服务器资源。过了一会会后发现，这游戏的官方服务端没有Linux版本，只有一个Windows版本的，这就把哥们难住了。接下来就仔细地讲讲完整的搭建过程，顺便让那位摸鱼了一个寒假的哥们学习一下怎么更好的使用Linux。

我采用的是Centos8版本的Linux，服务器使用Wine和X virtual framebuffer在Linux环境下使用Windows的软件。

首先是安装Wine，我这边是腾讯云的服务器，使用的是阿里云的Yum源，wine这玩意在源里面没有，我们得到官方网站上面下一个包来安装。[官网链接](https://www.winehq.org/announce/7.0)

![](https://www.kirisamekano.com/image/ror2/1.png)

两个下载链接哪个快用哪个

使用wget下载速度实在是太慢，我直接下载下来以后用ftp工具丢上服务器的hh

下载下来后是一个后缀为tar.xz的压缩文件，直接解压出来以后进入到wine7.0文件夹中，可以看到文件夹下有一个configure文件，直接运行就好了，等会报错了再依次安装依赖库。想要指定安装文件夹的也可以加入“--prefix=安装路径”

```
tar -xvf wine-7.0.tar.xz
cd wine-7.0
./configure --prefix=/usr/local
```

![](https://www.kirisamekano.com/wp-content/uploads/2022/03/YDM5E958PWE0KESP63YC.png)

这里显示缺少了32位的安装依赖库，那我们首先来安装这玩意。（测试用的阿里云源可以完美运行）

```
yum -y groupinstall 'Development Tools'
yum -y install libX11-devel libxml2-devel libxslt-devel freetype-devel flex bison
```

等待完成后再次运行./configure指令，发现还有问题：

![](https://www.kirisamekano.com/wp-content/uploads/2022/03/U_6@VW@EMI2H9E.png)

tnnd，没完没了了是吧

这个是缺少了libX里面的东西，只需要安装一个就好了，如果安装了还提示的话就搜索一下有没有32位的安装包（像是后缀是i686的），以及是否安装了相对的"-devel"。

```
yum install libX11.i686 libX11-devel.i686
```

![](https://www.kirisamekano.com/wp-content/uploads/2022/03/H5LR_KMT5F_HK1PUWBS8.png)

哼哼哼，啊啊啊啊啊啊啊啊啊啊

```
yum install freetype.i686 freetype-devel.i686
```

![](https://www.kirisamekano.com/wp-content/uploads/2022/03/image-1.png)

接下来就出现了最喜闻乐见的地方了，这里还缺少的几个包我个人觉得其实并不是很需要，如果需要的话也可以后期安装完wine以后再进行安装。我这边直接进行了一个make && make install，接下来要等待非常非常非常长的时间，我第一次运行是半个多小时，建议是挂着后台等他慢慢搞就好了。

![](https://www.kirisamekano.com/wp-content/uploads/2022/03/image-2.png)

走到这一步也是搞定了，接下来就到游戏服务端啦！

老套路先搞定steamcmd，使用steamcmd切换到windows平台下载ror2的服务端。

```
如果没有steamcmd的话先执行下面的步骤！
wget http://media.steampowered.com/installer/steamcmd_linux.tar.gz
tar -zxvf steamcmd_linux.tar.gz
./steamcmd.sh
然后等待更新完成后匿名登录steam
force_install_dir /home/steam/game ///设置安装路径(可选)
login anonymous
@sSteamCmdForcePlatformType windows
app_update 1180760 validate
exit ///完成后退出steam
```

接下来进入安装路径启动Risk of Rain 2.exe

```
cd /home/steam/steamapps/common/Risk\ of\ Rain\ 2
./Risk\ of\ Rain\ 2.exe
```

![](https://www.kirisamekano.com/wp-content/uploads/2022/03/image-3.png)

但是出现了错误的输出，仔细看一下报错，说的是缺少了X server这个东西，他指的其实是xvfb，全称X virtual framebuffer，是一个虚拟图形界面的玩意，我还没仔细研究过。接下来只需要安装这个东西就好了，在yum下面可以直接安装xorg以及xorg-x11-server-Xvfb，指令如下：

```
yum -y install xorg xorg-x11-server-Xvfb
安装完成后再次进入文件夹启动服务端
xvfb-run wine ./Risk\ of\ Rain\ 2.exe
```

到此为止所有工作都已经完成了，还有一个server.cfg文件在"Risk of Rain 2\_Data/Config"里面，第一次开服成功以后应该就会自动出现在里面了，这里列出几个比较常用的。

```
sv_maxplayers 4 //最大玩家数
sv_hostname "" //服务器名字
sv_port 27015 //服务器端口
sv_password "" //服务器密码
```

游戏里面进入服务器需要开启控制台，Ctrl+Alt+\`打开控制台，输入IP+Port进入服务器，接下来就是快乐的游戏时间啦！恭喜完成服务器的所有部署，这次这个还是蛮折腾的，如果用docker啥的应该会简单好多。这个wine安装是真的很久，我开了make && make install以后去散步加跑步一个小时左右回来才刚搞定。但是雨中冒险真的很好玩，我在第一个服务器上面安装其实比这次顺利很多，很多环境都已经早就折腾过了hh

2022.3.10早上更新：突然想起来还有screen没有讲，感觉还是有必要拉出来说一下。

```
yum install screen
screen -S ror2
cd /home/steam/Steam/steamapps/common/Risk\ of\ Rain\ 2
xvfb-run wine ./Risk\ of\ Rain\ 2.exe
//服务器运行成功后就可以退出screen了
ctrl+A+D
//再次进入该screen窗口
screen -ls
//找到相对应窗口前面的那串数字
screen -r 数字
```

最后附上一张虚空boss通关图，大家晚安！

![](https://www.kirisamekano.com/wp-content/uploads/2022/03/20220309181454_1-1024x576.jpg)

参考：

[搭建Risk of Rain 2 Dedicated Server (tand.me)](https://tand.me/36770)

[搭建Risk](https://tand.me/36770) [在RHEL，CentOS和Fedora上安装 Wine 3.0稳定版\_Linux教程\_Linux公社-Linux系统门户网站 (linuxidc.com)](https://www.linuxidc.com/Linux/2018-02/150753.htm)
