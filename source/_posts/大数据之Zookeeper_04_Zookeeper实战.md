---
title: 04 | Zookeeper实战
date: 2019-07-15 20:02:50
tags: [Zookeeper]
categories :
- 学习笔记
- Zookeeper
- 大数据之Zookeeper
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">分布式安装部署</font></center>
**1.集群规划**
A、B和C三个节点上部署Zookeeper。

**2.解压安装。**
- 在A的服务器解压安装zk。
- 同步zk到B、C。
```
xsync [ZK_PATH]/
```
> xsync是怎么同步A到B和C上的，在其他地方是否有相关设置？

**3.配置服务器编号。**
- 创建目录`[dataDir_path]`。
- 在`[dataDir_path]`目录下创建一个`myid`的文件。
- 编辑`myid文件`，在文件中添加与server对应的唯一编号`[SERVER_ID]`。
- 拷贝配置好的zookeeper到其他机器上。
```
xsync myid
```
并分别在B、C上修改myid文件中内容为自己的`[SERVER_ID]`。

**4.配置 zoo.cfg文件**
- 修改配置文件为zoo.cfg。
- 修改数据存储路径配置dataDir为`[dataDir_path]`。
- 增加如下配置。
```
server.[A_ID]=A:2888:3888
server.[B_ID]=B:2888:3888
server.[C_ID]=C:2888:3888
```
- 同步zoo.cfg配置文件。
```
xsync zoo.cfg
```
> 这里的A、B、C是怎么映射到IP的？

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">配置参数解读</font></center>
`server.A=B:C:D`
- A是一个数字，表示这个是第几号服务器。集群模式下在dataDir目录下配置一个文件myid，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。
- B是这个服务器的ip地址。
- C是这个服务器与集群中的Leader服务器交换信息的端口。
- D是用来执行**重新选举**时服务器相互通信的端口。

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">集群操作</font></center>
- 分别启动Zookeeper。
- 分别查看状态。
  可从结果中知道这个节点是follower还是leader。









