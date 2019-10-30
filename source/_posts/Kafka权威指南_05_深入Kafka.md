---
title: 05 | 深入Kafka
date: 2019-10-22 19:42:45
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- Kafka权威指南
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">集群成员关系</font></center>
broker启动的时候会在创建一个**临时节点**，把自己的id注册到zk上（`/brokers/ids/#`这样的形式）。Kafka组件会监听`/brokers/ids`这个路径，当集群中有broker退出或新增时，组件都会知悉。
将一个broker完全关闭之后，启动另外一个拥有相同id的broker，那么它会拥有与旧broker相同的分区与主题。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">控制器</font></center>
集群中的broker在加入集群时，都会尝试创建临时节点`/controller`，但是只有一个可以创建成功，那么它将成为控制器。其他竞争失败的broker将会监听这个临时节点，以便当节点变更时（比如说控制器挂了），它们可以知悉。当控制器挂了，剩余的broker又会开始竞争流程。
新的broker在成为控制器后，通过zk的递增操作获得数值更大的epoch。其他broker会忽略掉比当前epoch小的消息（这些消息也就是从旧的控制器发出的），从而避免了“脑裂”。
> 如果控制器在断开后重连，当发现创建`/controller`失败后，它应该就知道自己不是控制器了，那么之前比较小的epoch又是怎么发出的呢？

当控制器发现若一个broker离开后(通过监听`/brokers/ids`这个节点可知)，若这个broker又是分区leader，那么控制器就会在剩余的分区副本列表里面选出一个leader，并通知副本列表。


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">复制</font></center>
如果Follower在一定时间内（这个时间由`replica.Lag.time.max.ms`配置，默认为10S）没有向Leader请求同步数据，那么这个Follower就被认为是**不同步**的，当分区Leader失去连接触发重新选举时，**未同步的Follower将不会被选举**。

每一个分区都有一个**首选Leader**，它由以下两种方式指定：
- **创建主题时选定的Leader分区**就是首选Leader。
- 手动进行副本分配，**第一个指定的副本**就是首选Leader。

当`auto.leader.rebalance.enable`设为true时，它会去检查首选Leader是不是当前Leader，如果不是，且首选Leader是同步的那么就会触发选举，**让首选Leader成为当前Leader**。

在复制还未完成的时候，如果此时leader崩溃，那么就会导致数据丢失。

#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">请求</font></center>
**1、Kafka处理请求的内部流程**
![](Kafka权威指南_05_深入Kafka\kafka处理请求的内部流程.png)
broker会在它所监听的每一个端口上运行一个**Acceptor线程**，这个线程会创建一个连接，并把它交给**Processor线程**去处理。Processor线程的数量是可配置的。Processor线程把客户端请求的消息放进请求队列，然后从响应队列获取响应消息，把它们发送给客户端。

**2、客户端获取分区leader所在的broker的过程**
![](Kafka权威指南_05_深入Kafka\客户端获取分区leader所在的broker的过程.png)
客户端对**任意broker**（因为每个broker都缓存了这部分数据）发送**元数据请求**，然后将返回的数据（包括表明分区leader在哪个broker的信息）缓存起来，之后需要间隔一定的时间去重新缓存这部分数据（时间间隔可以由`metadata.max.age.ms`来配置）。

当客户端将数据发送到一个不是分区leader的broker的时候，会接收到一个“**非分区首领**”的错误，然后客户端会重新缓存元数据，再重新发送数据到相应的Borker。


**3、生产请求**
如果`acks=0`，那么生产者在把消息发出去之后，完全**不需要等待broker的响应**。
在消息被写入分区的首领之后，broker开始检查acks配置参数
- 如果acks被设为0或1，那么broker立即返回响应。
- 如果acks被设为all，那么请求会被保存在一个叫作**炼狱**的缓冲区里，直到首领发现所有跟随者副本都复制了消息，响应才会被返回给客户端。


**4、获取请求**
Kafka使用**零复制技术**向客户端发送消息。
**零复制技术**的意思就是直接把消息从文件（或者更确切地说是Linux文件系统缓存）里发送到网络通道，而不需要经过任何中间缓冲区。这项技术避免了字节复制，也不需要管理内存缓冲区，从而获得更好的性能。


#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">物理存储</font></center>
