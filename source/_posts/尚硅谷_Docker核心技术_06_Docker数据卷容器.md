---
title: 06 | Docker数据卷容器
date: 2019-07-04 11:27:58
tags: [Docker]
categories :
- 学习笔记
- Docker
- 尚硅谷_Docker核心技术
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">卷的概念</font></center>
卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过UnionFS提供一些用于持续存储或共享数据的特性。
卷的设计目的就是<font color = "#CD5555">数据的持久化</font>，完全<font color = "#CD5555">独立于容器的生存周期</font>，因此Docker不会在容器删除时删除其挂载的数据卷。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">卷的作用</font></center>
- 容器的持久化。
- 容器间继承+共享数据。
- 容器与主机的数据共享。

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">卷的特点</font></center>
- 数据卷可在容器之间共享或重用数据。
- 卷中的更改可以直接生效。
- 数据卷中的更改不会包含在镜像的更新中。
- 数据卷的生命周期一直持续到没有容器使用它为止。


#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">容器内添加数据卷</font></center>
**1、直接命令添加**
```
docker run -it -v /宿主机绝对路径目录:/容器内目录镜像名 镜像名
```
可以通过`inspect`命令查看数据卷是否挂载成功。
容器关了后，在主机的挂载目录下添加文件A，等容器重启后，容器相应的挂载目录也能看到这个文件A。

```
docker run-it-v/宿主机绝对路径目录:/容器内目录:ro 镜像名
```
上面的命令中(ro = read only)，容器只能对挂载目录进行读，但是主机可以对挂载目录进行读写。

**2、DockerFile添加**

















