---
title: 06 | 任务执行
date: 2020-01-17 12:45:58
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

ExecutorService有三种状态：
- 运行。
- 关闭。
- 终止。
shutdown方法是平缓的关闭。
shutdownNow方法是暴力关闭。
可用awaitTermination来等待ExecutorService到达终止状态，或者调用isTerminated来轮询ExecutorService是否终止。


Timer的缺陷：
例如某个周期TimerTask需要每l0ms执行一次，而另一个TimerTask需要执行40ms，那么这个周期任务或者在40ms任务执行完成后快速连续地调用4次，或者彻底“丢失”4次调用（取决于它是基于固定速率来调度还是基于固定延时来调度）。
Timer不会捕获异常，如果任务抛出异常时，Timer不会恢复线程的运行，此时将导致整个定时任务不可用。
可用ScheduledThreadPoolExecutor代替Timer。
> DelayQueue实现BlockingQueue，并为ScheduledThreadPoolExecutor提供了调度功能。


ExecutorService提交任务
先完成的先返回
executorService.submit(一组任务)
循环{
   executorService.take()//哪个任务优先完成则先返回。
}

等待任务全部结束或中断、超时
```java
List<Future<TravelQuote>> futures = exec.invokeAll(tasks, time, unit); 
List<TravelQuote> quotes = new ArrayList<TravelQuote>(tasks.size()); 
Iterator<QuoteTask> taskIter = tasks.iterator(); 
for(Future<TravelQuote> f : futures){
    QuoteTask task = taskIter.next(); 
    try{
        quotes.add(f.get());
    } catch(ExecutionException e){
        quotes.add(task.getFailureQuote(e, getCause()));
    } catch(CancellationException e){
        quotes.add(task.getTimeoutQuote(e));
    }
}
```        