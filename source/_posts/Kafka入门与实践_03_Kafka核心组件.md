---
title: 03 | Kafka核心组件
date: 2019-09-17 11:14:12
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- Kafka入门与实践
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">延迟操作组件</font></center>
**1、DelayedOperation**
**DelayedOperation框架**
![](Kafka入门与实践_03_Kafka核心组件\源码.jpg)
**DelayedOperation调用流程**
![](Kafka入门与实践_03_Kafka核心组件\DelayedOperation调用流程.jpg)
- `TimerTask.cancel()`：将该延迟操作从TimerTaskList链表中移除。
- `forceCompete()`：调用TimerTask.cancel()，并且当completed为true时，调用onComplete()方法，确保onComplete()只被调用一次。
- `safeTryComplete()`：以synchronized同步锁调用onComplete()方法，供外部调用。

