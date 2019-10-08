---
title: 01 | 初识Kafka
date: 2019-10-08 13:59:29
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- Kafka权威指南
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">消息模式</font></center>



|            模式             | 强类型转换 |                    消息模式升级                     |
| :-------------------------: | :--------: | :-------------------------------------------------: |
|       传统的json、xml       |   不支持   |                       不支持                        |
| 序列化框架（如Apache Avro） |    支持    | 支持，可向前/向后兼容，消除消息读写操作之间的耦合性 |


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">集群之间的消息复制</font></center>
Kafka的消息复制机制只能在单个集群里进行，不能在多个集群之间进行。可使用**MirrorMaker**实现集群间的消息复制。

