---
title: 03 | Docker的helloworld
date: 2019-07-03 17:19:48
tags: [Docker]
categories :
- 学习笔记
- Docker
- 尚硅谷_Docker核心技术
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">镜像加速</font></center>
将配置文件的仓库地址修改为阿里云的镜像加速地址。（具体步骤可以去阿里云仓库看）
可用`ps -ef`命令查看docker的启动配置是否是修改的阿里云镜像加速地址。

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">run hello world镜像</font></center>
```
docker run hello-world（不添加版本号，则默认是 hello-world:latest）
```
**run的过程**
![](尚硅谷_Docker核心技术_03_Docker的helloworld\run命令的过程.jpg)



#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">运行原理</font></center>

Docker是一个<font color = "#CD5555">Client-Server</font>结构的系统，<font color = "#CD5555">Docker守护进程</font>运行在主机上，然后通过<font color = "#CD5555">Socket连接</font>从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。

















