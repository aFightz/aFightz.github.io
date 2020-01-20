---
title: 07 | 取消与关闭
date: 2020-01-19 10:54:46
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

<center> <h4><font color = "#36648B">✎</br>中断与阻塞</center>

**1、Thread中的中断方法**
```java
public class Thread{
    public void interrupt(){...}
    public boolean isInterrupted){...}
    public static boolean interrupted(){...}    
}
```
- interrupt：中断目标线程。
- isInterrupted：能返回目标线程的中断状态。
- 静态的interrupted：清除当前线程的中断状态，并返回它之前的值。
> 这也是清除中断状态的唯一方法。

**2、阻塞库方法**
例如Thread.sleep和Object.wait等，都会检查线程何时中断，并且在发现中断时提前返回。它们在响应中断时执行的操作包括：
- 清除中断状态。
- 抛出InterruptedException。

> 用中断来取消任务并不是可靠的，如果任务内不响应中断，那么中断也没有用。

**3、join的缺陷**
无法知道执行控制是因为线程正常退出而返回还是因为join超时而返回。


**4、通过Future.cancel()取消任务**
>此方法很重要，可避免不必要的资源浪费。

```java
public static void timedRun(Runnable r，long timeout，TimeUnit unit) throws InterruptedException{
    Future<?> task = taskExec.submit(x);
    try{
        task.get(timeout , unit);
    }catch()TimeoutException e){
        //接下来任务将被取消
    }catch(ExecutionException e){
        //如果在任务中抛出了异常，那么重新抛出该异常throw launderThrowable(e.getCause())；
    }finally{
        //如果任务已经结来，那么执行取消操作也不会带来任何影响
        task.cancel(true);//如果任务正在运行，那么将被中断
    }
}
```

<center> <h4><font color = "#36648B">✎</br>线程异常</center>

当一个线程由于未捕获异常而退出时，JVM会把这个事件报告给应用程序提供的`UncaughtExceptionHandler`异常处理器。如果没有提供任何异常处理器，那么默认的行为是将栈追踪信息输出到`System.err`。

只有通过`execute`提交的任务，才能将它抛出的异常交给未捕获异常处理器，而通过`submit`提交的任务，无论是抛出的未检查异常还是已检查异常，都将被认为是任务返回状态的一部分。如果一个由submit提交的任务由于抛出了异常而结束，那么这个异常将被`Future.get`封装在`ExecutionException`中重新抛出。

