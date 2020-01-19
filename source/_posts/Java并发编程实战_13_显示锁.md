---
title: 13 | 显示锁
date: 2020-01-20 00:59:58
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

在ReentrantLock、Semaphore的构造函数中提供了两种公平性选择：创建一个非公平的锁（默认）或者一个公平的锁。

公平锁：线程将按照它们发出请求的顺序来获得锁。
非公平的锁：则允许“插队”。即当一个线程请求非公平的锁时，如果在发出请求的同时该锁的状态变为可用，那么这个线程将跳过队列中所有的等待线程并获得这个锁。

在公平的锁中，如果有另一个线程持有这个锁或者有其他线程在队列中等待这个锁，那么新发出请求的线程将被放入队列中。在非公平的锁中，只有当锁被某个线程持有时，新发出请求的线程才会被放入队列中。

即使对于公平锁而言，可轮询的tryLock仍然会“插队”。（什么意思）。

非公平锁的性能会比公平锁的性能高，尤其是在竞争激烈的时候。

ReentrantLock的非块结构特性仍然意味着，获取锁的操作不能与特定的栈帧关联起来，而内置锁却可以。


读写锁
一个资源可以被多个读操作访问，或者被一个写操作访问，但两者不能同时进行。


用wait()、notifyAll()实现阻塞put、take方法的标准格式
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

可对上面的进行优化：仅当缓存从空变为非空，或者从满转为非满时，才需要去通知（其他状态不会有线程阻塞）
public synchronized void put(V v) throws InterruptedException{
   while(isFull())
      wait(); 
   boolean wasEmpty = isEmpty(); 
   doPut(v); 
   if(wasEmpty) 
      notifyAll();
}


                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           