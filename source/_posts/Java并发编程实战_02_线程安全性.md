---
title: 02 | 线程安全性
date: 2019-03-22 10:49:43
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

##### 什么时候需要同步？

该变量是<font color = "red">共享</font>且<font color = "red">可变</font>的。

##### 实现同步的4种方法

- volatile
- synchronized
- 显示锁Lock
- 原子类Atomic
快速记忆：<font color = "blue">阿sa背着lv（salv）</font>

##### 不变性的定义

无论在单线程还是多线程，结果永远不变。



##### 其他知识点

Servlet是<font color = "red">单例</font>的，如果在某个Servlet类添加可变的局部变量，则要考虑此变量是否需要同步访问。

加锁与解锁都需要消耗资源，对于同一块代码，应该尽可能地少用同步块（能用一个同步块解决的，就不要用两个同步块）。

可以用FindBugs插件检查代码是否会有bug。
