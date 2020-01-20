---
title: 13 | 显示锁
date: 2020-01-20 00:59:58
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

<center> <h4><font color = "#36648B">✎</br>公平锁与非公平锁</center>

在ReentrantLock、Semaphore的构造函数中提供了两种公平性选择：创建一个**非公平的锁（默认）**或者一个**公平的锁**。

**1、定义**

- **公平锁**：线程将按照它们发出请求的顺序来获得锁。如果有另一个线程持有这个锁或者有其他线程在队列中等待这个锁，那么新发出请求的线程将被放入队列中。
- **非公平的锁**：允许“插队”。当一个线程请求非公平的锁时，如果锁被某个线程持有时，新发出请求的线程才会被放入队列中。但如果在发出请求的同时该锁的状态变为可用，那么这个线程将跳过队列中所有的等待线程并获得这个锁。
> 是不是意味在在队列中的锁依旧会排队？

> 即使对于公平锁而言，可轮询的tryLock仍然会“插队”。（什么意思）。

**2、性能**
非公平锁的性能会比公平锁的性能高，尤其是在竞争激烈的时候。


**3、Lock与Synchronized**
ReentrantLock的非块结构特性仍然意味着，获取锁的操作不能与特定的**栈帧**关联起来，而内置锁却可以。


<center> <h4><font color = "#36648B">✎</br>读写锁</center>
一个资源可以被多个读操作访问，或者被一个写操作访问，但两者不能同时进行。


<center> <h4><font color = "#36648B">✎</br>阻塞队列的实现</center>

用wait()、notifyAll()实现阻塞put、take方法的标准格式
```java
//阻塞并直到：not-full 
public synchronized void put(V v) throws InterruptedException{
   while(isFull())
      wait();
   doPut(v);
   notifyA11();
}
//阻塞并直到：not-empty 
public synchronized V take() throws InterruptedException{
   while(isEmpty())
      wait();
   V v = doTake();
   notifyAll();
   return v;
}
```

可对上面的进行优化：仅当缓存从空变为非空，或者从满转为非满时，才需要去通知（其他状态不会有线程阻塞）
```java
public synchronized void put(V v) throws InterruptedException{
   while(isFull())
      wait(); 
   boolean wasEmpty = isEmpty(); 
   doPut(v); 
   if(wasEmpty) 
      notifyAll();
}
```

                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           