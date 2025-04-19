---
title: Wordpress更新你害人不浅啊！
tags: []
id: '9'
categories:
  - - 'Linux'
date: 2023-02-08 20:05:32
---

今天上来才发现网站的数据库连接boom掉了，也不怕说重置系统后我的数据库就没有设定密码，因为mysql为root用户设定密码太乱了，网上的教程有说有初始密码的，但是我的初始就是没有密码，太懒了就干脆不搞。前几天apt-get update了一下，估计mysql给升级了，加上Wordpress又自动更新了一次，我就感觉是空密码不让用，所以才有了这篇文章。

网上搜索的改密码方法很多，但是我全试了一遍都没用，找官方文档的也不行，唯一有用的就是下面的这条指令：

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PASSWORD';
```
