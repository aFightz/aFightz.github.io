---
title: 01 | Java代码是怎么运行的？
date: 2019-01-09 16:43:43
tags: [MySQL]
categories :
- 学习笔记
- Java虚拟机
- 深入拆解Java虚拟机
---

JRE就是Java运行时环境。JRE包括了Java虚拟机以及Java核心类库。

Java虚拟机内存划分
![](深入拆解Java虚拟机_01_Java代码是怎么运行的？）\Java虚拟机的内存划分.png)

Java 虚拟机具体是怎样运行 Java 字节码的？
执行Java代码首先需要将它编译而成的class文件加载到Java虚拟机中，加载后的Java类会被存放于方法区（Method Area）中。




