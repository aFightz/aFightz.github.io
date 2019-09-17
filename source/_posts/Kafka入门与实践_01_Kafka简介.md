---
title: 01 | Kafka简介
date: 2019-09-16 19:36:56
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- Kafka入门与实践
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">Leader副本和Follower副本</font></center>
如果采用“**所有副本都同时负责读/写请求处理**”这种方式，那么得保证这些副本之间数据的一致性，假设有n个副本则需要有`n x n`条通路来同步数据，这样数据的一致性和有序性就很难保证。
而采用“**Leader副本进行读写，Follower副本从Leader副本同步消息**”这种方式，对于n个副本只需`n-1`条通路即可。
> 上述采用第一种方式，通路数应该是`n x (n - 1)`，原书作者应该是算错了。
其实这种方式就是Eureka注册中心的实现方式。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">消费者组</font></center>
如果不指定消费组，则该消费者属于默认消费组`test-consumer-group`。
每个消费者也有一个全局唯一的id，通过配置项`client.id`指定，如果客户端没有指定消费者的id，Kafka会自动为该消费者生成一个全局唯一的id，格式为`${groupld}-${hostName}-${timestamp}-${UUID前8位字符}`。

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">ISR</font></center>
Kafka在ZooKeeper中动态维护了一个**ISR（In-sync Replica）**，即保存同步的副本列表，该列表中保存的是与Leader副本保持消息同步的所有副本对应的代理节点id。

#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">高吞吐量的实现</font></center>
- Kafka充分利用磁盘的顺序读写，将数据写到磁盘。
- 同时，Kafka在数据写入及数据同步采用了`零拷贝（zero-copy）技术`，采用`sendFile()`函数调用，`sendFile()函数`是在两个文件描述符之间直接传递数据，完全在**内核**中操作。


