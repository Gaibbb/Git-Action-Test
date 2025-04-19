---
title: Linux云服务器——Bash Shell学习笔记 (1)
tags: []
id: '3'
categories:
  - - 'Linux'
date: 2021-12-04 11:12:23
---

这第一篇的Bash Shell学习笔记呢，是我在研究[L4D2-Competitive-Rework](https://github.com/SirPlease/L4D2-Competitive-Rework)他的服务器一键脚本中得出来的，主要分为对指令后添加的像是-l -s之类的作用是什么，还有就是Bash Shell的一些基本的指令和语法。

首先先来看srcds的第一段

```bash
case "$1" in
start)
echo "Starting $DESC: $NAME"
if [ -e $DIR ]; then
cd $DIR
su $SRCDS_USER -l -c "screen -d -m -S $NAME $DAEMON $PARAMS"
else
echo "No such directory: $DIR!"
fi
;;
```

”Case“ 类似于C++中的switch语句，具体就不多在这里阐述。

```bash
echo "Starting $DESC: $NAME"
```

“echo” 指的是字符串的输出，也不用多说。

“$” 是用在变量前面，作用是引用变量。

```bash
if [ -e $DIR ]; then
cd $DIR
su $SRCDS_USER -l -c "screen -d -m -S $NAME $DAEMON $PARAMS"
else
echo "No such directory: $DIR!"
fi
```

然后就是常见的If Else语句，这个要注意一下包含条件语句要用\[ \]圈起来，else不用esle结束，只有if要用fi结束。

```bash
if [ -e $DIR ]; then
```

在If里面的条件是判断前面用户所设的路径是否正确，“-e”的作用是判断文件是否存在，存在的话则为真。

```bash
su $SRCDS_USER -l -c "screen -d -m -S $NAME $DAEMON $PARAMS"
```

切换用户语句里面的附加条件"-l"与"-c"分别代表login(登录)与command(指令)，代表了登录进用户以后使用后面的screen指令

Screen后面的附加条件"-d" "-m" "-S"分别代表Detached(离线)、即使已经有了一个对应的Screen依旧强制创建一个新的，并且对创建的Screen窗口命名。

* * *

接下来看第二段stop代码

```bash
stop)
if su $SRCDS_USER -l -c "screen -ls" grep $NAME; then
echo -n "Stopping $DESC: $NAME"
kill `su $SRCDS_USER -l -c "screen -ls" grep $NAME awk -F . '{print $1}'awk '{print $1}'`
echo " ... done."
else
echo "Couldn't find a running $DESC"
fi
;;
```

在第二行的If语句中出现了新的" grep "代码，这是linux的基础功能，“ ”代表管道符，对前面已经执行出的结果执行新的命令，"grep"是字符串查询命令。

```bash
echo -n "Stopping $DESC: $NAME"
```

echo后面出现了新的"-n"指令，这是换行符的意思。

```bash
kill `su $SRCDS_USER -l -c "screen -ls" grep $NAME awk -F . '{print $1}'awk '{print $1}'`
```

这里就好玩了，用"kill"杀死查询出来的screen的进程，注意su前面的不是单引号而是键盘Esc下面的那个波浪号的小点，具体什么作用我暂时也不知道xD

回到代码，先用"screen -ls"调出所有screen窗口，然后用 " grep"查询相对应的窗口，awk是文本处理语言，"-F ."代表以“ . ”作为分隔符输出第一项(也就是$1)。举个例子，当语句中有3个单词分别是Apple Bat Cat时，不输入-F代表默认以空格作为分隔符，此时print $1时只会出现Apple，要是用" . "作为分隔符的话则会输出Apple Bat Cat，因为他查询不到" . "就会继续向下输出。

* * *

下面来看我也是第一次才学到的知识，

```bash
status)
# Check whether there's a "srcds" process
ps aux  grep -v grep  grep srcds_r > /dev/null
CHECK=$?
[ $CHECK -eq 0 ] && echo "SRCDS is UP"  echo "SRCDS is DOWN"
;;
```

这一段的关键在于第三行代码，实在是太牛逼了。

```bash
ps aux  grep -v grep  grep srcds_r > /dev/null
```

ps aux以BSD格式来显示进程，接下来用grep -v去除grep查询的进程得出一个更为正确的进程表，再次嵌套一个grep查询是否存在srcds\_r，接下来就是最牛逼的地方，grep srcds\_r之后的" > /dev/null"将查询结果重定向到/dev/null这个文件，这个文件是一个特殊的文件，任何写入的东西都会被删除，它的作用就在于，清楚查询的结果，只返回一个程序是否正确执行的数字。后面一段的CHECK=$?就用到了这个返回值，"$?"代表读取上一条程序的返回结果，如果为0则程序正确运行，如果为其他数字则是程序没有正常执行。最后通过一个判断条件\[ $CHECK -eq 0 \]判断$CHECK是否为0，是的话输出"SRCDS is UP"，为其他数字的话就输出"SRCDS is DOWN"。总的来说就是以查询程序是否正确执行来判断服务器的运行情况，我这是第一次接触这种方法，让我大受震撼。

感谢你看到这里，希望我的总结对你有用，一起学习，一起进步！

下面是一些我用到的参考文档，想要深入理解文中出现的一些内容可以看看，也是非常有用的东西捏！

参考：

[Linux 下的两个特殊的文件 -- /dev/null 和 /dev/zero 简介及对比\_longerzone的专栏-CSDN博客](https://blog.csdn.net/longerzone/article/details/12948925)

[Linux下ps -ef和ps aux的区别及格式详解 - 旅途 - 博客园 (cnblogs.com)](https://www.cnblogs.com/mydriverc/p/8303242.html)

[L4D2-Competitive-Rework/srcds1 at master · SirPlease/L4D2-Competitive-Rework · GitHub](https://github.com/SirPlease/L4D2-Competitive-Rework/blob/master/Dedicated%20Server%20Install%20Guide/srcds1)
