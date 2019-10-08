---
title: 02 | 安装Kafka
date: 2019-10-08 14:36:33
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- Kafka权威指南
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">常规配置</font></center>
**1、zookeeper.connect**
该配置参数是用冒号分隔的一组`hostname:port/path`列表。
`/path`是可选的Zookeeper路径，作为Kafka集群的chroot环境。如果不指定，**默认使用根路径**。如果指定的chroot路径不存在，broker会在启动的时候创建它。
最好是配置`/path`，那么一个Zookeeper下就可以存在多个kafka集群了而**不会发生冲突**了。

**2、log.dirs**
如果指定了多个路径，那么broker会根据“**最少使用**”原则，把**同一个分区的日志片段保存到同一个路径下**。要注意，broker会往拥有**最少数目分区的路径新增分区**，而不是往拥有最小磁盘空间的路径新增分区。

**3、num.recovery.threads.per.data.dir**
对于如下3种情况，Kafka会使用可配置的线程池来处理日志片段：
- 服务器正常启动，用于打开每个分区的日志片段。
- 服务器崩溃后重启，用于检查和截短每个分区的日志片段。
- 服务器正常关闭，用于关闭日志片段。

默认情况下，**每个日志目录只使用一个线程**。
所配置的数字对应的是`log.dirs`指定的单个日志目录，如果`num.recovery.threads.per.data.dir`被设为8，并且`log.dirs`指定了3个路径，那么总共需要24个线程。
配置多个线程，对于包含大量分区的服务器来说，一旦发生崩溃，在进行恢复时使用并行操作可能会省下数小时的时间。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">主题配置</font></center>

**1、数据的保留参数配置**
- **log.retention.ms/minutes/hours**
- **log.retention.bytes**
- **log.segment.ms**
- **log.segment.bytes**

上面的参数都是作用在**分区**上的，当达到`segment`的限制时，日志片段就会被关闭。此时这个片段就进入**等待删除**阶段。当**等待删除**阶段达到了`retention`限制，这个日志片段就会被删除。
`retention`是通过检查磁盘上日志片段文件的**最后修改时间**来实现的。所以如果移动了**处于等待删除阶段的日志片段文件**导致最后修改时间改变，那么删除的数据就会不准确。

**2、message.max.bytes**
这个参数用来限制单个消息的大小，默认是1000000（1MB）。若超过这个大小，broker会返回错误。
这个参数必须要与消费者的`fetch.message.max.bytes`与集群里broker的`replica.fetch.max.bytes`进行协调，否则若`fetch.message.max.bytes`与`replica.fetch.max.bytes`小于这个参数，那么会出现**堵塞**的情况。



#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">操作系统调优</font></center>
**1、vm.swappiness**
默认为60，表示剩余内存低于40%时，使用交换空间。
0意味着“在任何情况下都不要发生交换”。所以现在建议把这个值设为1。

**2、vm.dirty_background_ratio与vm.dirty_ratio**
当脏页数量到达`vm.dirty_background_ratio`时，会触发刷新进程异步地将脏页写入磁盘。
当脏页数量到达`vm.dirty_ratio`时，会触发刷新进程同步地将脏页写入磁盘，此时写操作**阻塞**。
