---
title: 01 | Zookeeper入门
date: 2019-07-15 12:56:07
tags: [Zookeeper]
categories :
- 学习笔记
- Zookeeper
- 大数据之Zookeeper
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">概述</font></center>
Zookeeper是一个开源的分布式的，为分布式应用提供协调服务的Apache项目。

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">工作机制</font></center>
Zookeeper从设计模式角度来理解：是一个基于**观察者模式**设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就通知已经在Zookeeper上注册的其他观察者。

> **联想：**
Eureka客户端是定时去注册中心获取服务。并把获取到的服务列表缓存起来。
而zk在服务器列表有变化时，会发送消息给客户端，客户端就会重新向zk缓存服务列表。
也就是说，对服务列表变化非常敏感的场景，zk应该是更好的选择。


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">工作机制</font></center>
![](大数据之Zookeeper_01_Zookeeper入门\特点.jpg)
- 一个领导者（Leader），多个跟随者（Follower）组成的集群。
- 集群中只要有**半数以上**节点存活，zk集群就能正常服务。
- 全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。
- 更新请求顺序进行，来自同一个Client的更新请求按其发送顺序依次执行。
- 数据更新原子性，一次数据更新要么成功，要么失败。
- 实时性，在一定时间范围内，Client能读到最新数据。



#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">数据结构</font></center>
![](大数据之Zookeeper_01_Zookeeper入门\节点.jpg)
ZooKeeper数据模型的结构与Unix文件系统很类似，整体上可以看作是一棵树，每个节点称做一个ZNode。每一个ZNode默认能够存储**1MB**的数据，每个ZNode都可以通过其路径唯一标识。


#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">应用场景</font></center>
提供的服务包括：**统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡**等。

**1、统一命名服务**
在分布式环境下，经常需要对应用/服务进行统一命名，便于识别。（类似Eureka Server）


**2、统一配置管理**
![](大数据之Zookeeper_01_Zookeeper入门\统一配置.jpg)
- 将配置信息写入ZooKeeper上的一个Znode。
- 各个客户端服务器监听这个Znode。
- 一旦Znode中的数据被修改，zk将通知各个客户端服务器。

**3、统一集群管理**
![](大数据之Zookeeper_01_Zookeeper入门\统一集群管理.jpg)
- 将节点信息写入ZooKeeper上的一个ZNode。
- 监听这个ZNode可获取它的实时状态变化。

**4、服务器节点动态上下线**
客户端能**实时洞察**到服务器上下线的变化，过程如下：
- 服务端启动时去注册信息（创建都是临时节点）。
- 客户端获取到当前在线服务器列表，并且**注册监听**。
- 服务器节点下线，将通知客户端。

**5、软负载均衡**
在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求。
