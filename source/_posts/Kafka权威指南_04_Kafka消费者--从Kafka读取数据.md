---
title: 04 | Kafka消费者--向Kafka读取数据
date: 2019-10-09 20:56:59
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- Kafka权威指南
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">消费者组</font></center>

**1、分配分区**
每个消费者组有一个自己的**群组协调器（broker）**，当消费者要加入群组时，它会向**协调器**发送一个JoinGroup请求。第一个加入群组的消费者将成为“**群主**”。
群主可以从协调器那里获得组员的所有信息，并为组员分配分区（通过实现`PartitionAssignor`接口的类）。群主将分配情况发送给群组协调器，再由群组协调器发送给所有组员。
每当消费者离开（主动断开或心跳失败）时会触发再均衡，此时会再重新分区。
> 新组员加进来时，是否也会触发再均衡？

消费者会在**轮询消息**或**提交偏移量**时，发送心跳消息。
如果消费组里的消费者，超过主题的分区数量，那么有一部分消费者就会被闲置，不会接收到任何消息。

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">消费者的配置</font></center>
**1、group.id**
指定组id。

**2、fetch.min.bytes与fetch.max.wait.ms**
fetch.min.bytes表示消费者从服务器获取记录的最小字节数。
fetch.max.wait.ms表示获取数据时的等待时间，默认为500ms。
两者取时间






