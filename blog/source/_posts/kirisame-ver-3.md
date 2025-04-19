---
title: KirisAme Ver.3
tags: []
id: '14'
categories:
    - Linux
date: 2023-7-21 11:00:00
---

# 全新的博客主题！
正如你所见的一样，我的博客发生了天翻地覆的大改变，这不仅是因为更换了一个博客的主题这么简单。在这次的折腾中，我放弃了原来使用的LAMP环境转用了Nginx + Hexo的新搭配，虽然最后还是没能完全放弃掉PHP和Mysql，原因放到后面再讲~   
由于全部东西都是新的，我还搞评论那些东西，也不知道到底行不行，有事的话可以发送邮件给我：**ame@kirisamekano.com**。

## 新的文章推送方式
因为环境的改变，我编写文章的方式也大有不同了，以前是通过进入Wordpress后台，通过在线编辑器进行文章的编写，现在采用了Hexo静态页面的方式以后，我只能通过本地或者是VS CODE连接到SSH服务器进行在线的MD文档编写，然后再通过Hexo将静态页面渲染为Html页面进行文章的推送上传。  

这个编写方式的转变其实有好有坏，坏的一处是我要放弃便捷的可视化文档编写，Wordpress自带的编辑工具其实挺好用的说~但是没有关系，现在我也可以通过一些其他的在线MD文档编辑器进行编写，但是我还是更加偏向于自己本地或者在线编写MD文档，顺便也当MD文档加强训练了hh。本地编写其实挺方便的，搞完了以后就直接推送到服务器上面然后开个渲染就行，可以搞成完全自动化的，唯一麻烦的就是我现在还没有想好图片的问题要怎么解决。我有强迫症，每一篇文章的图片必须严格分开管理，这样以后有要重新编辑或者是拿出来用才方便，因此博客文章编写的完全自动化还得过几天才能推出来，到时候可以单独开一篇文章讲解一下。  

## 新的中间件
虽然我没有用很专业的方法去对比Apache2与Nginx的性能，但是根据我的肉眼多次观察，Nginx打开网页就是会比Apache2 + Hexo的组合快，加载网页的时候我这个主题顶部会有一条蓝色的进度条，Apache2的每次都会卡顿一下才走完，但是Nginx是一加载就直接拉满，观感上面我就觉得比Apache2的快。再次声明：**对比很不严谨！对比很不严谨！对比很不严谨！**

## 为什么不能完全放弃PHP与Mysql
我这个服务器上面还跑了个人邮箱，用的是Postfix和Dovecot的组合，Postfix的后台需要有PHP的支持还能跑起来，而且邮件的存储可以用Mysql进行。其实放不放弃这两个都没什么关系，数据库本来就是必备的软件，好多东西都要依赖数据库进行活动，感觉最多就是省略一个PHP，但是就会放弃一个方便管理的后台。服务器本来就没跑什么东西，因此放不放弃都无所谓啦~

# 配置Nginx的一些笔记
Nginx感觉配置起来比Apache直观不少，下面是我配置新网站时的一些记录:  

    server {
        listen 443 ssl;     #监听443 SSL端口
        server_name kirisamekano.com, www.kirisamekano.com;     #以子域名区分虚拟主机
        root /var/www/html;       #网页文件所在位置
        index index.html, index.htm;    #网页文件
        ssl_certificate /ssl/kirisamekano.com.crt;  #SSL证书
        ssl_certificate_key /ssl/kirisamekano.com.key;  #私钥
    }

    server {
        listen 80;  #监听80端口
        server_name kirisamekano.com, www.kirisamekano.com
        root /var/www/html;
        return 301 https://$host$request_uri;   #强制HTTPS转换，重定向HTTPS访问地址
    }

***
文章到这里就结束啦~     

久闻Nginx大名，今日有幸使用到，虽然还有很多功能像是反向代理这些没有用到，但是我手头上恰好又有一台空闲的服务器，晚点可以试试负载均衡和一些其他功能。虚拟主机的配置也是很有意思，目前我只用到了子域名划分，日后看看有没有机会试试端口划分。   

还有经过了三天的CSAPP学习，我发现了这玩意是真难，看着汇编代码一愣一愣的，草稿纸上面嘎嘎写题目，一天撑死了也就看二三十页的样子，学习路漫漫啊/(ㄒoㄒ)/~~  

最后试试图片的插入，人生第一次去Livehouse居然是在台上表演！
![这是一张普通的图片](https://www.kirisamekano.com/image/kirisame-ver-3/1.jpg)