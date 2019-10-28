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

