---
title: 01 | Kafka概述
date: 2019-08-01 15:36:05
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- 大数据之Kafka
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">消息模式</font></center>
- **点对点模式**（一对一，消费者主动拉取数据，消息收到后消息清除）点对点模型通常是一个基于拉取或者轮询的消息传送模型，这种模型从队列中请求信息，而不是将消息推送到客户端。这个模型的特点是发送到队列的消息被一个且只有一个接收者接收处理，即使有多个消息监听者也是如此。
- **发布/订阅模式**（一对多，数据生产后，推送给所有订阅者）发布订阅模型则是一个基于推送的消息传送模型。发布订阅模型可以有多种不同的订阅者，临时订阅者只在主动监听主题时才接收消息，而持久订阅者则监听主题的所有消息，即使当前订阅者不可用，处于离线状态。

Kafka是基于点对点模式实现的，同一时刻，同一个组只有一个消费者可以消费到同一分区的消息。但是消息被消费后不会被删除。
- 一个topic可以分为多个分区。
- 一个组下有多个消费者。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">Kafka架构</font></center>
![](大数据之Kafka_01_Kafka概述\kafka架构.jpg)
- Broker相当于一个kafka实例。
- Producer不与zk打交道。
- Follower只能作备份，不能作读写。可以在leader宕机的时候升级为leader。
> **问题：**
LF选举不是基于zk之间的吗？zk下的节点也可以进行选举？