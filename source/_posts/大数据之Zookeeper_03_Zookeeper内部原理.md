---
title: 03 | Zookeeper内部原理
date: 2019-07-15 18:43:28
tags: [Zookeeper]
categories :
- 学习笔记
- Zookeeper
- 大数据之Zookeeper
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">选举机制</font></center>
- 半数机制：集群中**半数以上**机器存活，集群可用。所以Zookeeper适合安装**奇数**台服务器。
- Zookeeper虽然在配置文件中并没有指定Master和Slave。但是，Zookeeper工作时，如果有一个节点为Leader，其他则为Follower，Leader是通过内部的选举机制临时产生的。
- 选举（启动时）的过程：
  一个节点进来时，先投自己，如果没有选出leader，则把票给id最大的server。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">节点类型</font></center>
**1、类型**
- **持久（Persistent）**：客户端和服务器端断开连接后，创建的节点不删除。
  持久分为两种类型：
  - **持久化目录节点**：客户端与Zookeeper断开连接后，该节点依旧存在。
  - **持久化顺序编号目录节点**：客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号。
  
  
- **短暂（Ephemeral）**：客户端和服务器端断开连接后，创建的节点自己删除。
  短暂节点可以用作探知client是否在线。（因为如果客户端没连的话，这个节点就被删除了）
  短暂节点分为两种类型：
  - **临时目录节点**：客户端与Zookeeper断开连接后，该节点被删除。
  - **临时顺序编号目录节点**：客户端与zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号。

**2、顺序标识**
创建znode时设置顺序标识，znode名称后会附加一个值，顺序号是一个单调递增的计数器，由**父节点**维护。
在分布式系统中，顺序号可以被用于为所有的事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序。
> 这个顺序标识可以知道全局中哪个节点先上线。


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">Stat结构体</font></center>
- czxid：创建节点的事务zid
  - 每次修改ZooKeeper状态都会收到一个zxid形式的时间戳，也就是**ZooKeeper事务ID**。
  - 事务ID是ZooKeeper中所有修改总的次序。每个修改都有**唯一**的zxid，如果zxid1小于zxid2，那么zxid1在zxid2之前发生。
- ctime-znode：被创建的毫秒数（从1970年开始）。
- mzxid-znode：最后更新的事务zxid。
- mtime-znode：最后修改的毫秒数（从1970年开始）。
- pZxid-znode：最后更新的子节点zxid。
- cversion-znode：子节点变化号，zmode子节点修改次数。
- dataversion-znode：数据变化号。
- aclVersion-znode：访问控制列表的变化号。
- ephemeralOwner：如果是临时节点，这个是znode拥有者的session id。如果不是临时节点则是0。
- dataLength：znode的数据长度。
- numChildren：znode子节点数量。


#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">监听器原理</font></center>
**1、监听过程**
- 首先要有一个main()线程。
- 在main线程中创建Zookeeper客户端，这时就会创建两个线程，一个负责网络连接通信（connet），一个负责监听（listener）。
- 通过connect线程将注册的监听事件发送给Zookeeper。
- 在Zookeeper的注册监听器列表中将注册的监听事件添加到列表中。
- Zookeeper监听到有数据或路径变化，就会将这个消息发送给listener线程。
- listener线程内部调用了process()方法。(回调方法)


#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">写数据流程</font></center>
- Client向ZooKeeper的Server1上写数据，发送一个写请求。
- 如果Server1不是Leader，那么Server1会把接受到的请求进一步转发给Leader，因为每个ZooKeeper的Server里面有一个是Leader。这个Leader会将写请求广播给各个Server，比如Server1和Server2，各个Server写成功后就会通知Leader。
- 当Leader收到大多数(半数以上)Server数据写成功了，那么就说明数据写成功了。
> 如果这里三个节点的话，只要有两个节点数据写成功了，那么就认为数据写成功了。写成功之后，Leader会告诉Server1数据写成功了。
- Server1会进一步通知Client数据写成功了，这时就认为整个写操作成功。

![](大数据之Zookeeper_03_Zookeeper内部原理\写数据过程.jpg)


