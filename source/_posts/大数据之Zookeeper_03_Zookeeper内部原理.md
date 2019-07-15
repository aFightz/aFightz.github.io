---
title: 02 | Zookeeper内部原理
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
**1、类型分类**
- **持久（Persistent）**：客户端和服务器端断开连接后，创建的节点不删除。
  持久分为两种类型：
  - **持久化目录节点**：客户端与Zookeeper断开连接后，该节点依旧存在。
  - **持久化顺序编号目录节点**：客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号。
  
  
- **短暂（Ephemeral）**：客户端和服务器端断开连接后，创建的节点自己删除。
  短暂节点可以用作探知client是否在线。（因为如果客户端没连的话，这个节点就被删除了）
  短暂节点分为两种类型：
  - **临时目录节点**：客户端与Zookeeper断开连接后，该节点被删除。
  - **临时顺序编号目录节点**：客户端与ookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号。

**2、顺序标识**
创建znode时设置顺序标识，znode名称后会附加一个值，顺序号是一个单调递增的计数器，由**父节点**维护。
在分布式系统中，顺序号可以被用于为所有的事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序。
> 这个顺序标识可以知道全局中哪个节点先上线。







