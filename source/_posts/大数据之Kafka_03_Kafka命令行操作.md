---
title: 03 | Kafka命令行操作
date: 2019-08-02 00:51:03
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- 大数据之Kafka
---

**1、查看当前服务器中的所有topic**
```
bin/kafka-topics.sh --zookeeper hadoop102:2181 --list
```

**2、创建 topic**
```
bin/kafka-topics.sh --zookeeper hadoop102:2181 --create --replication-factor 2 --partitions 2 --topic first
```
选项说明： 
- **topic**。定义topic名。
- **replication-factor**。定义副本数（副本数不能大于broker数）。
- **partitions**。定义分区数。

logs文件夹不仅仅存日志，还存topic的分区。
上面的命令会在102的logs/目录下新建first-0和first-1的分区文件目录，会在103上新建first-0，在104上新建first-1。（因为有2个副本）

**3、删除topic**
```
bin/kafka-topics.sh--zookeeper hadoopl02：2181 --delete --topic first
```
需要`server.properties`中设置`delete.topic.enable=true`否则只是标记删除或者直接重启。

**4、发送消息**
```
bin/kafka-console-producer.sh --broker-list hadoop102:9092 --topic first
>hello world
>atguigu atguigu
```
如果往不存在的主题发送数据，那么这个主题会被创建。


**5、消费消息**
```
bin/kafka-console-consumer.sh --zookeeper hadoop102:2181 --from-beginning --topic first
```
- `from-beginning`会把first主题中以往所有的数据都读取出来。
- 新版本zk直接用`--bootstrap-server`去连kafka实例。
新版本的kafka会把offset**维护在本地**（`consumer_offsets`这个topic）而不是zk。
consumer在消费时如果没指定组id，则随机分配一个组Id。

> 为什么发送消息与消费消息不用指定分区？是有默认的吗？

**6、查看某个Topic的详情**
```
bin/kafka-topics.sh --zookeeper hadoop102:2181 --describe --topic first
```




