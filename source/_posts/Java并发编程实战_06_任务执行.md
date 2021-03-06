---
title: 06 | 任务执行
date: 2020-01-17 12:45:58
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

<center> <h4><font color = "#36648B">✎</br>ExecutorService</center>

**1、ExecutorService有三种状态：**
- 运行。
- 关闭。
- 终止。

**2、ExecutorService的关闭方式：**
- shutdown：平缓的关闭。
- shutdownNow：暴力关闭。
> 可用awaitTermination来等待ExecutorService到达终止状态，或者调用isTerminated来轮询ExecutorService是否终止。

**3、ExecutorService等待一组提交任务**

**先完成的先返回**
```java
executorService.submit(一组任务)
循环{
   executorService.take()//哪个任务优先完成则先返回。
}
```

**等待任务全部结束或中断、超时才返回**
```java
List<Future<TravelQuote>> futures = executorService.invokeAll(tasks, time, unit); 
``` 



<center> <h4><font color = "#36648B">✎</br>Timer</center>
**Timer的缺陷：**
例如某个周期TimerTask需要每l0ms执行一次，而另一个TimerTask需要执行40ms，那么这个周期任务或者在40ms任务执行完成后快速连续地调用4次，或者彻底“丢失”4次调用（取决于它是基于固定速率来调度还是基于固定延时来调度）。
Timer不会捕获异常，如果任务抛出异常时，Timer不会恢复线程的运行，此时将导致整个定时任务不可用。
可用`ScheduledThreadPoolExecutor`代替Timer。
> DelayQueue实现了BlockingQueue，并为ScheduledThreadPoolExecutor提供了调度功能。       