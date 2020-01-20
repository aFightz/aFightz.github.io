---
title: 10 | 活跃性、性能与测试
date: 2020-01-19 17:03:37
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

<center> <h4><font color = "#36648B">✎</br>死锁</center>

数据库在检测到事务发生死锁时，将选择一个牺牲者并放弃这个事务。

若所有线程以固定的顺序来获得锁，那么在程序中就不会出现顺序死锁问题。
 
解决顺序死锁的加锁方式
```java
synchronized(锁A){
   synchronized(锁B){
   }
}                    
```

> 尽量别再持有锁的时候调用加锁方法，容易造成死锁。

<center> <h4><font color = "#36648B">✎✎</br>Thread.yield与Thread.sleep(0)</center>

在JVM中Thread.yield（以及Thread.sleep(0)）的语义都是未定义的。JVM既可以将它们实现为空操作，也可以将它们视为线程调度的参考。
> 有些JVM实现yield方法的方式就是sleep(0)。     

UNIX系统中sleep(0)的语义：将当前线程放在与该优先级对应的运行队列末尾，并将执行权交给拥有相同优先级的其他线程。   




                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           