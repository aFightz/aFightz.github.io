---
title: 03 | 堆
date: 2019-10-15 00:53:25
tags: [JVM]
categories :
- 学习笔记
- JVM
- 网课_JVM
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">堆的结构</font></center>

![](网课_JVM_03_堆\堆结构.png)
如箭头所示，对象从上（新生区）往下（养老区）走。

**1、新生区**
新生区又分为两部分：
- **伊甸区（Eden space）**。
- **幸存者区（Survivor pace）**。

新生区进行的是**Minor GC**。

**2、养老区**
养老区进行的是**Full GC**。
养老区执行了Full GC之后发现依然无法进行对象的保存，就会产生**OOM异常**。
**连接池对象**一般都在这个区活跃。

**3、永久区**
永久区**不会被回收**。用于存放JDK自身所携带的Class，Interface的元数据（例如Object类）。


> 如果出现**java.lang.OutOfMemoryError:Java heap space**异常，原因有二：
- Java虚拟机设置的堆内存不够大，可以通过参数**-Xms**、**-Xmx**来调整。
- 代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（一直被引用）。

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">代码体现</font></center>

**1、查看当前可以使用的最大堆内存**
```
Runtime.getRuntime().maxMemory()/1024/1024)
```