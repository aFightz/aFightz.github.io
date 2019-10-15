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
养老区进行的是**Full GC（Major GC）**。
养老区执行了Full GC之后发现依然无法进行对象的保存，就会产生**OOM异常**。
**连接池对象**一般都在这个区活跃。

**3、永久区**
永久区**不会被回收**。用于存放JDK自身所携带的Class，Interface的元数据（例如Object类）。


**4、方法区**
java1.8以前，可以理解为**永久区是方法区的实现**。



#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">代码体现</font></center>

**1、查看当前可以使用的最大堆内存**
```
Runtime.getRuntime().maxMemory()/1024/1024)
```


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">异常分析</font></center>
**1、java.lang.OutOfMemoryError:Java heap space**
- Java虚拟机设置的堆内存不够大。
> 代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（一直被引用）。

**2、java.lang.0utOfMemoryError:PermGen space**
- Java虚拟机对永久代Perm内存设置不够。
> 一般出现这种情况，都是程序启动需要加载大量的第三方jar包。例如：在一个Tomcat下部署了太多的应用。或者大量动态反射生成的类不断被加载，最终导致Perm区被占满。

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">JDK版本</font></center>
- Jdk1.6及之前：有永久代，常量池在方法区。
- Jdk1.7：有永久代，常量池在堆。
- Jdk1.8及之后：无永久代，常量池1.8在元空间。

#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">堆的配置与GC</font></center>
**1、java1.7**
![](网课_JVM_03_堆\堆的配置与GC1.png)

**2、java1.8**
![](网课_JVM_03_堆\堆的配置与GC2.png)

> full gc又是什么？


#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">堆内存调优</font></center>
**1、配置**
- **-Xms**：设置初始分配大小，默认为物理内存的**1/64**。
- **-Xmx**：最大分配内存，默认为物理内存的**1/4**。
- **-XX:+PrintGCDetails**：输出详细的GC处理日志。
> 注意：上述分配的堆空间不包含元数据区/方法区，只有新生区和老年区。

**2、代码**
```java
public static void main(String[] args){
    //返回 Java 虚拟机试图使用的最大内存量。
    long maxMemory = Runtime.getRuntime().maxMemory();
    //返回 Java 虚拟机中的内存总量。
    long totalMemory = Runtime.getRuntime().totalMemory();
    System.out.println("MAX_MEMORY = " + maxMemory + "（字节）、" + (maxMemory / (double)1024 / 1024) + "MB");
    System.out.println("TOTAL_MEMORY = " + totalMemory + "（字节）、" + (totalMemory / (double)1024 / 1024) + "MB");
}
```




