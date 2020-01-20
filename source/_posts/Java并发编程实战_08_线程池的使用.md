---
title: 08 | 线程池的使用
date: 2020-01-19 12:23:59
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

在线程池的线程中不应该使用ThreadLocal在任务之间传递值。

在线程池中，如果任务依赖于其他任务，那么可能产生饥饿死锁。除非线程池足够大。


<center> <h4><font color = "#36648B">✎</br>线程池数量</center>
对于计算密集型的任务，在拥有N个处理器的系统上，当线程池的大小为**N+1**时，通常能实现最优的利用率。（即使当计算密集型的线程偶尔由于页缺失故障或者其他原因而暂停时，这个“额外”的线程也能确保CPU的时钟周期不会被浪费。）

而大多数的任务是计算与IO并存的，所以
```
最佳线程池数量 = CPU数量 * CPU利用率 * (1 + W/C)
```
W为线程等待时间，C为线程计算时间。W+C即是一个线程任务实际运行时间。

Java中获得CPU数量
```java
int N_CPUS=Runtime.getRuntime().availableProcessors();
```


<center> <h4><font color = "#36648B">✎✎</br>线程池原理</center>

**1、线程池构造方法**
```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    ...
}
```

**2、线程池工作原理**
- **keepAliveTime**：线程存活时间。
- **core Pool Size**：线程池的基本大小，在没有任务执行时线程池的大小，并且只有在工作队列满了的情况下才会创建超出这个数量的线程。
- **maximum Pool Size**：最大大小。表示可同时活动的线程数量的上限。如果某个线程的空闲时间超过了存活时间，那么将被标记为可回收的，并且当线程池的当前大小超过了基本大小时，这个线程将被终止。

>**newFixedThreadPool**工厂方法将线程池的基本大小和最大大小设置为**参数中指定的值**，而且创建的线程池**不会超时**。
**newCachedThreadPool**工厂方法将线程池的最大大小设置为**Integer.MAX_VALUE**，而将基本大小设置为**零**，并将超时设置为**1分钟**，这种方法创建出来的线程池可以被无限扩展，并且当需求降低时会自动收缩。

如果线程池中的线程数量等于线程池的基本大小，那么仅当在**工作队列已满**的情况下ThreadPoolExecutor才会创建新的线程。

**3、线程池其他知识点**
默认的线程池饱和策略是“**中止**”：即抛出`RejectedExecutionException`。

可用Executors的`unconfigurableExecutorService`工厂方法包装ExecutorService，避免其被修改。

可继承`ThreadPoolExecutor`的`beforeExecute`、`afterExecute`和`terminated`以实现一些日志统计等功能。

<center> <h4><font color = "#36648B">✎✎</br>等待任务集返回</center>

```java
//提交任务
exec.shutdown();
exec.awaitTermination(Long.MAX_VALUE, TimeUnit.SECONDS);
//返回任务结果
```


<center> <h4><font color = "#36648B">✎✎✎</br>树与并发</center>
可用并发实现树的广度优先搜索。
- 深度优先搜索的搜索过程将受限于栈的大小。
- 广度优先搜索不会受到栈大小的限制（但如果待搜索的或者已搜索的位置集合大小超过了可用的内存总量，那么仍可能耗尽内存）。