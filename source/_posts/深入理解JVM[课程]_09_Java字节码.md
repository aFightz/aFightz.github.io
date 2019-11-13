---
title: 09 | Java字节码
date: 2019-11-12 12:22:00
tags: [JVM]
categories :
- 学习笔记
- JVM
- 深入理解JVM[课程]
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">字节码文件结构</font></center>

字节码文件由以下部分组成：
- 魔数。所有的.class字节码文件的前4个字节都是魔数，魔数值为固定值：**0xCAFEBABE**。
- 版本号。
- 常量池。
- 类信息。
- 类的构造方法。
- 类中的方法信息。
- 类变量。
- 成员变量。