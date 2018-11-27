---
title: 鸟哥的Linux私房菜基础学习篇（六）学习笔记
date: 2018-11-17 21:13:46
tags: [Linux]
categories :
- 学习笔记
- Linux
- 鸟哥的Linux私房菜基础学习篇
---

> **概要：**
>
> - 文件所有者、用户组、其他人的相关概念
> - 权限的相关操作
> - 权限对于目录与文件的意义

### 文件所有者（User）、用户组（Group）、其他人（Others）

- 账号记录在/etc/passwd

- 组名记录在/etc/group
- 密码记录在/ect/shadow

`ls -al`  可以查看所有文件的<font color = "red">权限与属性</font>（包括隐藏文件）
执行命令后的结果如下(部分)

```
huangjunlong@ubuntu:~$ ls -al
total 84
drwxr-xr-x 14 huangjunlong huangjunlong 4096 Nov 18 05:03 .
drwxr-xr-x  3 root         root         4096 Nov 15 08:54 ..
-rw-r--r--  1 huangjunlong huangjunlong  220 Nov 15 08:54 .bash_logout
-rw-r--r--  1 huangjunlong huangjunlong 3771 Nov 15 08:54 .bashrc
drwx------ 13 huangjunlong huangjunlong 4096 Nov 17 06:27 .cache
```

我们以`.cache`为例对属性的结构进行分析：

|     属性     |                         含义                          |
| :----------: | :---------------------------------------------------: |
|  drwx------  |                    文件类型与权限                     |
|      13      |                        连接数                         |
| huangjunlong |                      文件所有者                       |
| huangjunlong |                      文件所属组                       |
|     4096     |                文件大小（默认单位为B）                |
| Nov 17 06:27 |  文件最后被修改的时间（如果距离太久，就仅显示年份）   |
|    .cache    | 文件名（文件名前面多一个“.”则代表这个文件是隐藏文件） |

我们需要重点关注的是文件类型与权限这一块，它由<font color = "red">10</font>个字符组成，其中：

**第1个字符**表示<font color = "red">文件类型</font>，具体的代号表示如下

| 代号 |               含义               |
| :--: | :------------------------------: |
|  d   |               目录               |
|  -   |               文件               |
|  l   |       连接文件（linkfile）       |
|  b   | 设备文件里面的可供存储的接口设备 |
|  c   |    设备文件里面的串行端口设备    |

接下来的**9个字符**表示<font color = "red">文件权限</font>，可以分为<font color = "red">3组</font>，依次是

- 文件所有者的权限
- 同用户组的权限
- 其他非本用户组的权限

权限由<font color = "red">rwx</font>的组合构成，分别表示

- 可读
- 可写
- 可执行

浏览时如果想要显示完整的时间格式可以加上`full-time`参数，如下

```
huangjunlong@ubuntu:~/Desktop$ ls -l --full-time
total 4
drwxr-xr-- 2 root root 4096 2018-11-18 06:08:22.190687227 -0800 hello
```



### 改变文件属性与权限

**`chgrp`：改变文件所属用户组**

切换到root用户，将test.txt这个文件的用户组改为root，如下

```
root@ubuntu:/home/huangjunlong/Desktop# chgrp root test.txt
```

**`chown`：改变文件所有者，也可以改变文件所属用户组。**

将test.txt这个文件的用户组与文件所有者都改为huangjunlong，如下

```
root@ubuntu:/home/huangjunlong/Desktop# chown huangjunlong:huangjunlong test.txt 
```

单纯的将test.txt这个文件的用户组改为root，如下

```
root@ubuntu:/home/huangjunlong/Desktop# chown :root test.txt
```

**`chmod`：改变文件权限。**

> 注意：复制文件时，会把文件的属性也一起复制，所以要关注权限是否需要更改的问题。

如果改变的是目录，加上`-R`选项可以递归修改子目录下的所有文件、目录。

设置权限的方法有两种：

- **数字类型**
  其中<font color = "red">4</font>代表r，<font color = "red">2</font>代表w，<font color = "red">1</font>代表x。

  将hello.txt文件的权限全部开启，也就是设置为rwxrwxrwx：

  ```
  huangjunlong@ubuntu:~$ chmod 777 hello.txt
  ```

  

- **符号类型**
  <font color = "red">u</font>代表user，<font color = "red">g</font>代表group，<font color = "red">o</font>代表others，<font color = "red">a</font>代表全部。可以用<font color = "red">+、-、=</font>这三个符号来赋值：

  利用“=”将hello.text的文件设置为rwxr-xr-x：

  ```
  huangjunlong@ubuntu:~$ chmod u=rwx,go=rx hello.txt
  ```

  利用“+”给hello.text加上w权限，对所有人生效：

  ```
  huangjunlong@ubuntu:~$ chmod a+x hello.txt
  ```

  利用“-”给hello.text减去x权限，对所有人生效：

  ```
  huangjunlong@ubuntu:~$ chmod a-x hello.txt
  ```

  

### 权限对目录与文件的意义

从文件的角度上来说，理解rwx不难。

但从目录的角度来说还是有点区别的：

- r代表可以用ls浏览目录下的内容。
- w代表对该目录下的文件/目录有增删改的权限。
- x代表可以进入该目录。

***其中x是执行rw的前提。***
上面的知识可以得到一个很重要的知识点，<font color = "red">如果要删除一个文件，设置文件本身的权限是没有用的，需要设置它所在目录的权限为wx才行。<font/>

### 文件长度限制

单一文件或者目录的最长文件名为<font color = "red">255个字符。<font/>

包含完整路径名的文件或目录的最长文件名为<font color = "red">4096个字符。<font/>

### 目录的表示

- .代表当前目录，也可以用./来表示。
- ..代表上一层目录，也可以用../来表示。