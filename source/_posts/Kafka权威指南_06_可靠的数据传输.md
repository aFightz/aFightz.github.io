---
title: 06 | 可靠的数据传输
date: 2019-11-05 19:56:24
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- Kafka权威指南
---


#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">broker层面的保证</font></center>
- kafka可以保证分区内消息的一致性。
- 消费者接收到的消息，一定是已经被写入分区的所有同步副本的。
- 设置副本进行数据备份，防止数据丢失。


#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">生产者层面的保证</font></center>




