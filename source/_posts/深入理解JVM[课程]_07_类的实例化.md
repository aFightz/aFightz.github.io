---
title: 07 | 类的实例化
date: 2019-10-26 13:19:55
tags: [JVM]
categories :
- 学习笔记
- JVM
- 深入理解JVM[课程]
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">实例化的步骤</font></center>
- 为新的对象分配内存。
- 为实例变量赋默认值。
- 为实例变量赋正确的初始值。
> 通过代码验证得出，构造方法的调用会在实例变量赋初始值的后面。