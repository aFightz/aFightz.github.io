---
title: 06 | Docker数据卷容器
date: 2019-07-04 11:27:58
tags: [Docker]
categories :
- 学习笔记
- Docker
- Docker核心技术
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
- 数据卷中的更改<font color = "#CD5555">不会包含在镜像的更新</font>中。
- 数据卷的生命周期一直持续到<font color = "#CD5555">没有容器使用它为止</font>。


#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">容器内添加数据卷</font></center>
**1、直接命令添加**
```
docker run -it -v /宿主机绝对路径目录:/容器内目录 IMAGE
```
可以通过`inspect`命令查看数据卷是否挂载成功。
容器关了后，在主机的挂载目录下添加文件A，等容器重启后，容器相应的挂载目录也能看到这个文件A。

```
docker run-it-v/宿主机绝对路径目录:/容器内目录:ro IMAGE
```
上面的命令中(ro = read only,默认是rw)，容器只能对挂载目录进行读，但是主机可以对挂载目录进行读写。
> **问题**：如果主机不存在此目录，会自动创建吗？

**2、DockerFile添加**
可在Dockerfile中使用VOLUME指令来给镜像添加一个或多个数据卷:
```
VOLUME["/dataVolumeContainer"，"/dataVolumeContainer2"，"/dataVolumeContainer3]
```
该命令创建指定的主机目录是随机目录，无法指定特定目录，具体目录路径可以用`inspect`命令查看。

**3、DockerFile Demo**
```
FROM centos
VOLUME["/dataVolumeContainer1","/dataVolumeContainer2"]（这个会在什么时候创建？）
CMD echo "finished,--------success1"（这个会在什么时候打印？）
CMD /bin/bash  （这个有什么用？）
```

**4、build构建镜像**
```
docker build -f /mydocker/Dockerfile -t  IMAGE .
```

> **问题**：Docker挂载主机目录Docker访问出现<font color = "#CD5555">cannot open directory:Permission denied</font>。
</br>解决办法：在挂载目录后多加一个`-privileged=true`参数即可。



#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">数据卷容器</font></center> 
命名的容器挂载数据卷，其它容器通过挂载这个（父容器）实现数据共享，挂载数据卷的容器，称之为数据卷容器。

```
docker run -it --name dc02 --volumes-from dc01 IMAGE
```
dc02容器继承dc01容器，同时子容器02的数据也能被01看到。使父容器与子容器的数据共享。

`--volumes-from`不要理解为继承，理解为“联系”会更好一点，而且这种联系是“多向”的。在任何时候，数据卷都是同步的。
假设A与B联系，然后C与A联系，那么C与B也会联系。















