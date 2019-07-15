---
title: 02 | Zookeeper安装
date: 2019-07-15 17:10:02
tags: [Zookeeper]
categories :
- 学习笔记
- Zookeeper
- 大数据之Zookeeper
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">安装</font></center>
**1、解压**
- 安装jdk。
- 拷贝Zookeeper安装包到Linux系统下。
- 解压到指定目录
 ```
 tar -zxvf [zookeeper.tar.gz] -C [PATH]
 ```
 
**2、配置修改**
- 将`[zk解压路径]/conf`这个路径下的`zoo_sample.cfg`修改为`zoo.cfg`；
```
mv zoo_sample.cfg zoo.cfg
```

- 打开zoo.cfg文件，修改dataDr路径：
```
dataDir=[自定义的zkData路径]
```
- 根据dataDir配置在相应的目录上创建相应文件夹。

**3、操作Zookeeper**
- 启动Zookeeper
```
[ZK_PATH]/bin/zkServer.sh start
```

- 查看进程是否启动
```
[ZK_PATH]/jps
```

- 查看状态
```
[ZK_PATH]/bin/zkServer.sh status
```
可以知道这个server是leader还是follower。

- 启动客户端
```
[ZK_PATH]/bin/zkCli.sh
```

- 退出客户端：
```
quit
```

- 停止Zookeeper
```
[ZK_PATH]/bin/zkServer.sh stop
```

- 客户端查看根节点
```
ls /
```

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">配置参数解读</font></center>
- **tickTime=2000**
每隔一个tickTime就会发送一个心跳，时间单位为毫秒。它用于心跳机制，并且设置session的最小超时时间是`2*tickTime`。

- **initLimit=10**
LF初始通信时限。集群中的Follower与Leader之间初始连接时最长等待时间为`initLimit*tickTime`。

- **syncLimit=5**
LF同步通信时限。假如响应超过`syncLimit*tickTime`，Leader认为Follwer死掉，从服务器列表中删除Follwer。

- **dataDir**
数据文件目录+数据持久化路径。主要用于保存Zookeeper中的数据。

- **clientPort=2181**
客户端连接端口监听客户端连接的端口。



