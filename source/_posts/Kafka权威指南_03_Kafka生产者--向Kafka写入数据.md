---
title: 03 | Kafka生产者--向Kafka写入数据
date: 2019-10-08 19:33:31
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- Kafka权威指南
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">生产者发送流程</font></center>

![](Kafka权威指南_03_Kafka生产者--向Kafka写入数据\发送消息流程.png)

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">生产者配置参数</font></center>
**1、bootstrap.servers**
指定broker地址，建议设置两个以上。不必要设置全部broker，因为只要连上一个broker，就能查找到其他broker的信息。

**2、acks**
这个参数指定了**必须要有多少个分区副本收到消息**，生产者才会认为消息写入是成功的。

**3、buffer.memory**
该参数用来设置生产者内存缓冲区的大小，生产者用它缓冲要发送到服务器的消息。如果应用程序发送消息的速度超过发送到服务器的速度，会导致生产者空间不足。这个时候，`send()`方法调用要么被阻塞，要么抛出异常，取决于如何设置`max.block.ms`参数。
    
**4、retries**
这个参数指定了当发送消息时，返回可重试错误时，生产者重试发送的次数。默认重试间隔为100ms，可通过`retry.backoff.ms`改变这个间隔。

**5、batch.size与linger.ms**
`batch.size`指定了一个批次可以使用的内存大小（按照字节数计算）。
`linger.ms`指定了一个批次在等待到达batch.size时的最大等待时间。

**6、max.in.flight.requests.per.connection**
该参数指定了生产者在收到服务器响应之前可以发送多少个消息。
把它设为1可以保证消息是**按照发送的顺序写入服务器**的，即使发生了重试。

**7、timeout.ms、request.timeout.ms和metadata.fetch.timeout.ms**
- `request.timeout.ms`指定了生产者在发送数据时等待服务器返回响应的时间。
- `metadata.fetch.timeout.ms`指定了生产者在获取元数据（比如目标分区的首领是谁）时等待服务器返回响应的时间。如果等待响应超时，那么生产者要么重试发送数据，要么返回一个错误（抛出异常或执行回调）。
- `timeout.ms`指定了broker等待同步副本返回消息确认的时间，与`asks`的配置相匹配。

**8、max.block.ms**
该参数指定了在调用`send()`方法或使用`partitionsFor()`方法获取元数据时生产者的阻塞时间。

**9、max.request.size**
该参数用于控制生产者发送的请求大小。

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">生产者发送消息</font></center>
KafkaProducer在发送消息时一般会发生两类错误：
- 可重试错误。如连接错误、“无主”错误等。可配置成自动重试。
- 不可重试错误。如“消息太大”错误，会直接抛出异常。






