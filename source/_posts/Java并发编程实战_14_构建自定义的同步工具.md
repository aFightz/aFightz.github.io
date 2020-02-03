---
title: 14 | 构建自定义的同步工具
date: 2020-02-01 20:06:44
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

<center> <h4><font color = "#36648B">✎</br>阻塞队列的实现</center>

用wait()、notifyAll()实现阻塞put、take方法的标准格式
```java
//阻塞并直到：not-full 
public synchronized void put(V v) throws InterruptedException{
   while(isFull())
      wait();
   doPut(v);
   notifyAll();
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



<center> <h4><font color = "#36648B">✎✎</br>Lock的Condition对象</center>
每个Lock，可以有任意数量的Condition对象。Condition对象继承了相关的Lock对象的公平性。

**1、对应关系**
- Object.wait：Lock.await
- Object.notify：Lock.signal
- Object.notifyAll：Lock.signalAll

**2、唤醒特定对象Demo**
```java
@ThreadSafe
public class ConditionBoundedBuffer <T> {
    protected final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public void put(T x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length)
                notFull.await();
            items[tail] = x;
            if (++tail == items.length)
                tail = 0;
            ++count;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }
    
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0)
                notEmpty.await();
            T x = items[head];
            items[head] = null;
            if (++head == items.length)
                head = 0;
            --count;
            notFull.signal();
            return x;
        } finally {
            lock.unlock();
        }
    }
}
```

UnSafe类中的park()与unPark()实际上就是相当于wait与notify。
> 这两者有什么不同？

Java的多数安全的方法类都有一个共同的基类：AbstractQueueSynchronizer(AQS)，它实际上上用UnSafe类中的park()与unPark()来实现等待与唤醒。涉及到变量的累加修改则用CAS实现。

                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           