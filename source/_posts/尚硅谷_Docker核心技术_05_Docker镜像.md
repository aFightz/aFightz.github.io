---
title: 05 | Docker镜像
date: 2019-07-04 10:52:23
tags: [Docker]
categories :
- 学习笔记
- Docker
- 尚硅谷_Docker核心技术
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">镜像是什么</font></center>
镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">UnionFS（联合文件系统）</font></center>
Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下（unite several directories into a single virtual filesystem）。Union文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。
<font color = "#CD5555">特性：</font>一次同时加载多个文件系统，从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">Docker镜像加载原理</font></center>
bootfs（boot file system）主要包含bootloader和kernel，bootloader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs，这一层与我们典型的Linux/Unix系统是一样的：包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。
rootfs（root file system），在bootfs之上。包含的就是典型Linux系统中的/dev，/proc，/bin，/etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。
重点说是就是<font color = "#CD5555">内核共用</font>。

#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">镜像的分层</font></center>

分层的优点：
最大的一个好处就是<font color = "#CD5555">共享资源</font>。
比如：有多个镜像都从相同的base镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像，同时内存中也只需加载一份base镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">特点</font></center>
- Docker镜像都是<font color = "#CD5555">只读的</font>。
- 当容器启动时，一个新的<font color = "#CD5555">可写层</font>被加载到镜像的顶部，这一层通常被称作<font color = "#CD5555">“容器层”</font>，容器层”之下的都叫<font color = "#CD5555">“镜像层”</font>。

#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">镜像Commit</font></center>
提交容器副本使之成为一个<font color = "#CD5555">新的镜像</font>。
```
docker commit -m="提交的描述信息" -a="作者" 容器lD 要创建的目标镜像名:[标签名]
```

















