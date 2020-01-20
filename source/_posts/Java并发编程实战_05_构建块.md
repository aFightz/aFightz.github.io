---
title: 05 | 构建块
date: 2019-04-15 19:26:35
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

<center> <h4><font color = "#36648B">✎</br>在并发中迭代</center>
`Iterator`与`for-each`在循环的过程中，会监听一个计数器，如果计数器被修改（集合被修改），则会抛出`ConcurrentModificationException`。
值得一提的是修改计数器并不是一个线程安全的操作，所以有可能线程A在迭代，线程B修改了集合，但是线程A没抛出则会抛出`ConcurrentModificationException`。

```java
public class HiddenIterator {
    private final Set<Integer> set = new HashSet<>();

    public synchronized void add(Integer i){
        set.add(i);
    }
    
    public synchronized void remove(Integer i){
        set.remove(i);
    }
    
    public void addThenThings(){
        Random random = new Random();
        for(int i = 0 ; i < 10 ; i ++){
            add(random.nextInt());
        }
        System.out.println(set);
    }
}
```
上面的代码中，由于迭代（println会打印toString的内容，而hashSet的toString方法就是使用Iterator拼接Set的所有元素）与容器内元素的改变（HiddenIterator#get/set）没有使用同一个锁，所以在迭代时候有可能会出现ConcurrentModificationException。
解决的方法很简单，保持这两者的锁一致即可。

<center> <h4><font color = "#36648B">✎✎</br>常用的并发类</center>

**1、Map**
- **ConcurrentHashMap**：非独占锁，在并发情况下用来代替`HashMap`。提供不抛出ConcurrentModificationException的迭代器，这个迭代器拥有“弱一致性”而不是“及时失败”，允许并发修改。它的`size()`方法与`empty()`方法是估算值，并不完全准确。
> 这个迭代器是怎么实现的？
- **ConcurrentSkipListMap**：在并发情况下用来代替`SortedMap`。
> SortedMap是什么形式的Map？
- **Hashtable**：独占锁。
- **Collections.synchronizedMap**：独占锁。


**2、List**
- **CopyOnWriteList**：在并发情况下用来代替List。
容器的迭代器保留一个底层基础数组，这个数组的元素永远不会被修改，所以在迭代进行读取时，不需要加锁。迭代时，也不会抛出`ConcurrentModificationException`。
当需要修改元素时，会复制一个数组并对这个数组进行修改，然后直接替换基础数组。

**3、Set**
- **ConcurrentSkipListSet**：在并发情况下用来代替SortedSet。
- **CopyOnWriteArraySet**：在并发情况下用来代替Set。


<center> <h4><font color = "#36648B">✎✎✎</br>Queue</center>

**1、Queue的框架图**

![](Java并发编程实战_05_构建块\Queue框架图.png)

实现了BlockingQueue的都是阻塞队列（支持并发）。阻塞队列需要`catch InterruptionException`。
带有Priority的都是优先级队列。
`ConcurrentLinkedQueue`、`LinkedBlockingQueue`、`ArrayListBlockingQueue`是**FIFO队列**。

**2、BlockingQueue**
可以是有界的也可以是无界的。
非定时阻塞的方法：`put`、`take`。
可定时阻塞的方法：`offer`、`poll`。

**3、SynchronousQueue**
SynchronousQueue没有存储功能，因此put和take会一直阻塞，直到有另一个线程已经准备好参与到交付过程中。仅当有足够多的消费者，并且总是有一个消费者准备好获取交付的工作时，才适合使用同步队列。


生产者-消费者模式中，如果一个是I/O密集型，另一个是CPU密集型，那么并发执行的吞吐率要高于串行执行的吞吐率。如果生产者和消费者的并行度不同，那么将它们紧密耦合在一起会把整体并行度降低为二者中更小的并行度。


**4、用双端队列实现工作密取。**
如果一个消费者完成了自己双端队列中的全部工作，那么它可以从其他消费者双端队列末尾秘密地获取工作。

<center> <h4><font color = "#36648B">✎✎✎✎</br>中断</center>
中断并不是强制另外一个线程立刻停下来，只能要求另外一个线程在可以暂停的地方停止正在执行的工作。（当然被中断的线程也可以恢复中断，继续执行）
> 其实就是try catch Interruption的应用。

<center> <h4><font color = "#36648B">✎✎✎✎✎</br>缓存方案</center>

在做缓存时，计算value可能会需要很长的时间。通常的逻辑是：
```java
public String getValue(int key){
    同步块保护{
        //根据key get value
       if(value 不存在){
          //根据key 计算value，
          //push 进map
       }
    }
    //返回value 
}
```

> 当然，还可能再优化成分段锁(根据key分段)，或者优化成DCL双锁去保护value的计算，避免在读已存在缓存时进入同步块。
但是终究会有弊端：
- 同步开销大。
- 不同的key可能会使用同一个锁。（不可能为每一个key都生成一个锁，一般都会采用对key进行hash后取得相应的锁这种策略）

可用futureTask来代替上述逻辑，将计算放到一个线程任务中：
```java
public String getValue(int key){
    /**新建计算value的futureTask A**/
    
    //如果map已存在此future，则返回此future，此时抛弃futureTask A。否则将futureTask A push进去
    currentFutureTask = map.putIfAbsent(key , future);
    if(currentFutureTask == null){
        //如果purIfAbsent为null，则表示push进去的是futureTask A，需要submit futureTask A进行计算
        /**submit future任务**/
        /**currentFutureTask = futureTask A**/
    }
    return currentFutureTask.get();
}
```
使用这种方式可以避免上述过程中所论述的弊端。

> 需要整理一下，JDK提供的各种线程安全类。