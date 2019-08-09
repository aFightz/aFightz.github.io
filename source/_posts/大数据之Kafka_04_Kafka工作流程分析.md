---
title: 04 | Kafka工作流程分析
date: 2019-08-05 10:58:46
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- 大数据之Kafka
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">生产过程分析</font></center>
**1、写入方式**
producer采用push模式将消息发布到broker，每条消息都被append到patition中，属于**顺序写磁盘**（**顺序写磁盘效率比随机写内存要高**，保障kafka吞吐率）。

**2、分区**
要理解topic本质上就是一个**文件目录**，而topic又由分区组成。分区也是文件目录，文件目录名会类似topic-0、topic-1这样的命名。
![](大数据之Kafka_04_Kafka工作流程分析\分区结构.png)

每个Partition中的消息都是有序的，**分区内的每一个消息**都被赋予了一个**唯一的offset值**。
消息可以**重复被消费**，经常所说的“offset消费到哪里了”是针对消费者而不是生产者。
![](大数据之Kafka_04_Kafka工作流程分析\offset.png)


- 分区的原因
  - 方便在集群中扩展，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了。
  - 可以提高并发，因为可以以Partition为单位读写。

- 分区的原则
  - 指定了patition，则直接使用。
  - 未指定patition但指定key，通过对key的value进行hash出一个patition。
  - patition和key都未指定，使用轮询选出一个patition。


**3、副本**
同一个partition可能会有多个replication（对应`server.properties`配置中的`default.replication.factor=N`）。没有replication的情况下，一旦broker 宕机，其上所有 patition 的数据都不可被消费，同时producer也不能再将数据存于其上的patition。引入replication之后，同一个partition可能会有多个replication，而这时需要在这些replication之间选出一个leader，producer和consumer只与这个leader交互，其它replication作为follower从leader 中复制数据。

**4、写入流程**
ack为on可以保证生产者不会丢数据。ack为on时的写入流程如下：
![](大数据之Kafka_04_Kafka工作流程分析\写入过程.png)


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">Broker保存消息</font></center>
**1、存储方式**
消息实际上是保存在分区文件夹下的.log文件。

**2、存储策略**
无论消息是否被消费，kafka都会保留所有消息。有两种策略可以删除旧数据：
- 基于时间：`log.retention.hours=168`
- 基于大小：`log.retention.bytes=1073741824`

**3、zk存储结构**
![](大数据之Kafka_04_Kafka工作流程分析\kafka存储结构.jpg)


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">Kafka消费过程分析</font></center>
**1、消费者组**
每个分区在同一时间只能由group中的一个消费者读取，但是多个group可以同时消费这个partition。如果一个消费者失败了，那么其他的group成员会自动负载均衡读取之前失败的消费者读取的分区。


**2、消费者组Demo**

- 指定groupId。修改/config/consumer.properties配置文件中的group.id属性。
```
vi consumer.properties
group.id=atguigu
```
- 启动消费者
```
bin/kafka-console-consumer.sh --zookeeper hadoop102:2181 --topic first --consumer.config config/consumer.properties
```
