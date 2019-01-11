---
title: 鸟哥的Linux私房菜基础学习篇（五）学习笔记
date: 2018-11-15 00:41:46
tags: [Linux]
categories :
- 学习笔记
- Linux
- 鸟哥的Linux私房菜基础学习篇
---

> **前言**：由于第一章到第四章主要是在介绍Linux以及如何装Linux，所以先暂时略过，直接跳到第五章。
>

> **概要**：本章主要介绍了关于man、info查找命令的技巧。倡导大家不需要背命令，而是利用Linux的man、Info命令去寻找自己所需要的命令。除此之外，还介绍了关于关机与重启的命令以及Linux的几个Run Level。
>

> **特别说明**：本章读书笔记（以及接下来的章节）的Demo运行在<font color=red>Ubuntu18.04.1</font>（截至2018.11.12为止是最新版本）上，username为huangjunlong。
>

------

### ~ 、$、 #的区别

Linux用户可以分为root（超级用户）与一般用户。

当用一般用户登录时，命令行会显示<font color=red>$</font>，如下：

```shell
huangjunlong@ubuntu:~$
```

如果是root登录则会显示<font color=red>#</font>，如下：

```shell
root@ubuntu:/home/huangjunlong#
```

<font color=red>~</font>则表示<font color=red>当前用户的主目录</font>，它是一个“变量”。每个Linux用户都有自己的主文件夹，huangjunlong这个账户的主文件夹就是

```shell
/home/huangjunlong
```


cd到这个路径，当我们用huangjunlong这个账户登录时，它会显示~，此时我们也可以通过以下命令进入主目录。

```shell
cd ~
```



### Run Level

linux一共有<font color=red>7</font>个level，分别是<font color=red>0~6</font>。

在/etc这个目录下，以下目录分别存放了0~6七个运行级别的初始化文件。

```
rc0.d/  rc1.d/  rc2.d/  rc3.d/  rc4.d/  rc5.d/  rc6.d/
```



run level的部分含义如下：

| run level |       含义       |
| :-------: | :--------------: |
|     0     |       关机       |
|     3     |   纯命令行模式   |
|     5     | 含有图形界面模式 |
|     6     |       重启       |

可以使用`runlevel`命令查询当前的runlevel，如下：

```
root@ubuntu:/etc# runlevel
N 5
```


也可以使用`init`命令切换runlevel，如下：

```
root@ubuntu:/etc# init 0   //表示关机
```



### 其他命令

注销命令

```
huangjunlong@ubuntu:~$ exit
```

显示目前的语言

```
root@ubuntu:/etc# echo $LANG
en_US.UTF-8    
```

修改语言

```
huangjunlong@ubuntu:~$ LANG=en_US  //只对这次登录有效
```

显示日期与时间

```
huangjunlong@ubuntu:~$ date
Mon Nov 12 01:54:29 PST 2018
huangjunlong@ubuntu:~$ date +%Y/%m/%d
2018/11/12
huangjunlong@ubuntu:~$ date +%H:%M
01:57
```

显示日历

![](鸟哥的Linux私房菜基础学习篇（五）\cal.png)

![](鸟哥的Linux私房菜基础学习篇（五）\cal2.png)

计算器

```
huangjunlong@ubuntu:~$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'. 
1+2
3
10%3
1
```



### man

`man`的作用是查看某个命令的详细说明，如下：

```
huangjunlong@ubuntu:~$ man ls
LS(1)                           User Commands                           LS(1)
....<内容太长，省略参数的描述，具体可以看下表>
```



|    代号     |                    内容说明                    |
| :---------: | :--------------------------------------------: |
|    NAME     |            简短的命令、数据名称说明            |
|  SYNOPSIS   |        简短的命令执行语法（syntax）简介        |
| DESCRIPTION |                 较为完整的说明                 |
|   OPTIONS   | 针对SYNOPSIS部分中，有列举的所有可用的选项说明 |
|   COMMADS   |   当这个程序在执行时，可以在此程序执行的命令   |
|    FILES    |  这个程序或数据所使用或参考或连接到的某些文件  |
|   SEEALSO   |             这个命令有关的其他说明             |
|   EXAMPLE   |               一些可以参考的例子               |
|    BUGS     |                 是否有相关错误                 |

上面LS(1)中的“1”有特殊的意义。比较重要的几个数字的意义如下：

| 代号 |                  代表内容                   |
| :--: | :-----------------------------------------: |
|  1   | 用户在shell环境中可以操作的命令或可执行文件 |
|  5   |         配置文件或者某些文件的格式          |
|  8   |          系统管理员可用的管理命令           |

进入man界面后，按“空格”可以浏览下一页，也可以按“q”离开man环境。



查询与某个命令相关的信息，如下：

```
huangjunlong@ubuntu:~$ man -f man
man (1)              - an interface to the on-line reference manuals
man (7)              - macros to format man pages
```


此时man有两个命令，如果用

```
man man
```

那么会显示man(1)这个命令的相关信息，因为先查询到的说明文件会被先显示出来。如果用

```
man 7 man
```

则会显示man(7)这个命令的相关信息

如果希望某个关键字存在(无论是命令还是说明)就被查出来，就加上`-k`参数，如下

```
huangjunlong@ubuntu:~$ man -k man
HEAD (1p)            - Simple command line user agent
UPower (7)           - System-wide Power Management
accessdb (8)         - dumps the content of a man-db database in a human rea...
<省略...>
```

另外，`whatis`命令相当于`man -f`，`apropos`命令相当于`man -k`
原书上说使用这两个命令需要通过以下命令创建whatis数据库

```
makewhatis
```


*但是经过试验，在Ubuntu18.04.1版本中<font color=red>不需要创建数据库</font>也可以直接用这两个命令。*



 

### nano

nano命令可以浏览文件，如下

```shell
huangjunlong@ubuntu:~/Desktop$ nano test.txt
```

打开文件后的底部有如下信息

![](鸟哥的Linux私房菜基础学习篇（五）\nano.png)

其中^表示[ctrl]。



### 关机与重启

关机与重启前，最好先查看一下系统的一些状态看下是否需要做一些应对的措施，以免发生一些不必要的错误。

`who`命令可以查看有谁在线：

```
huangjunlong@ubuntu:~/Desktop$ who
huangjunlong :0           2018-11-12 01:27 (:0)
huangjunlong pts/1        2018-11-12 01:46 (192.168.220.1)
```

`nestat -a` 可以查看网络的联机状态：

```
huangjunlong@ubuntu:~/Desktop$ netstat -a | grep 8080   //把与8080有关的端口显示出来
unix  3      [ ]         STREAM     CONNECTED     28080    /var/run/dbus/system_bus_socket
```

`ps -aux` 可以查看后台执行的程序：

```
huangjunlong@ubuntu:~/Desktop$ ps -aux | grep tomcat     //把与tomcat有关的程序显示出来
huangju+   3877  0.0  0.1  21536  1028 pts/0    S+   04:36   0:00 grep --color=auto tomcat
```



`sync`可以强制将数据写回硬盘
如果数据从硬盘加载到内存中，操作完了立刻写回硬盘去，那么在读写很频繁的时候，性能会大大降低。所以加载到内存的数据不会立刻写回硬盘中。但这样也有风险，万一系统意外关机了，那么数据有可能会丢失。而sync命令就是<font color=red>强制将内存中尚未更新的数据写回硬盘</font>。在执行关机操作时，最好先调用此命令。



实现关机或重启的基础命令有<font color=red>3</font>个

|  命令   |         含义         |
| :-----: | :------------------: |
| reboot  |       重新启动       |
|  halt   | 关机，但并不关闭电源 |
| powerff |   关机，并关闭电源   |

通过`--help`参数查看上面三个命令，发现这三个命令通过参数是可以<font color=red>相互转换</font>的。
举个例子，我们看一下halt的帮助文档来证实上面的言论：

![](鸟哥的Linux私房菜基础学习篇（五）\halt.png)

同样的，我们根据`shutdown --help`可知，`shutdown`命令可以通过不同的参数调用上面的3个命令实现关机或重启，那`shutdown`与它们之间有什么区别与联系呢？

- shutdown会<font color=red>关闭系统各项服务后</font>再执行reboot/halt/poweroff，可以理解为“安全关机”。
- reboot/halt/poweroff在没有加参数`--force`的情况下，会调用shutdown关闭各项服务，有点意思的是shutdown又会再调用reboot/halt/poweroff。（形成一个相互调用链）。当然如果加上`--force`，就不调用shutdown直接关闭了，这样是有风险的，和拔电源这种行为没什么区别。
- shutdown还能选择在<font color=red>未来某个时间点</font>执行shutdown，并且还可以给在线的用户发送相应的消息。到了这个时间点的<font color=red>前 5 分钟</font>，会创建<font color=red>/run/nologin</font>文件，确保没有人可以再登录。







