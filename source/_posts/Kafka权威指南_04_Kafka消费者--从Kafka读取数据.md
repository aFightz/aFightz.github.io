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
两者哪个先被满足就会接收消息。

**3、max.partition.fetch.bytes**
该属性指定了服务器从每个分区里返回给消费者的最大字节数。它的默认值是1MB。

**4、session.timeout.ms与heartbeat.interval.ms**
session.timeout.ms指定了消费者在被认为死亡之前可以与服务器断开连接的时间，默认是3s。
heartbeat.interval.ms指定了poll()方法向协调器发送心跳的频率。一般是session.timeout.ms的三分之一。

**5、auto.offset.reset**
该属性指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下（因消费者长时间失效，包含偏移量的记录已经过时并被删除）该作何处理。
- `latest`。默认值，从最新的记录开始读。
- `earliest`。从起始的记录开始读。

**6、enable.auto.commit**
是否自动提交偏移量。默认值为true。可以通过`auto.commit.interval.ms`（默认为5S）来控制自动提交的频率。 

**7、partition.assignment.strategy**
分区分配的策略：
- `Range`：把主题的若干个连续的分区分配给消费者。
- `RoundRobin`：把主题的所有分区逐个分配给消费者。

**8、client.id**
客户端Id。

**9.max.poll.records**
该属性用于控制单次调用call()方法能够返回的记录数量。

**10.receive.buffer.bytes 和send.buffer.bytes**
TCP的缓冲区配置。默认为-1（使用操作系统默认值）。



#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">提交</font></center>
**1、手动提交偏移量**
`enable.auto.commit`设置为false。并手动调用`commitSync()`或`commitASync()`。

**2、同步提交与异步提交**
- `commitSync()`：同步提交。如果出错会重试。
- `commitASync()`：异步提交。如果出错不会重试，因为如果重试的话有可能会让偏移量小的提交覆盖偏移量大的提交。
> 解决方法：因为异步提交支持回调，所以可以在本地记录上一次已成功提交的偏移量，在回调方法中，若提交失败则根据当前提交偏移量与本地保存的偏移量决定是否要重试。

默认情况下，手动提交默认都是提交最后一个偏移量，也可以指定提交特定的偏移量。

**3、同步提交与异步提交结合**
伪代码实现如下：
```java
try{
    poll();
    commitASync();
}finally{
    commitSync();
}
```



#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">再均衡监听器</font></center>
**1、步骤**
- 实现`ConsumerRebalanceListener`这个类。
- 在subscribe时，传入ConsumerRebalanceListener实现类。

**2、作用**
当发生`分区再均衡/分区被分配`时，可以通知到消费者，这时我们可以做一些相应的操作（如强制提交偏移量等）。
还可以结合`seek()`方法，在本地维护偏移量（将处理消息与保存偏移量放在同一个事务中）。可以实现消息的“零重复/零丢失”。


#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">安全的退出</font></center>
可以通过`consumer.wakeup()`方法退出`poll()`，并且`poll()`能抛出`WakeupException`，在退出时最好要调用`consumer.close()`,因为它会主动地像群组协调器发送自己要离开的消息，而不用等到超时触发再均衡。并且还能主动提交任何还未提交的东西。


#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">读取特定的分区</font></center>
可以通过`comsumer.assign()`方法去读取特定的分区，此时就不需要订阅主题，也不需要群组了。



