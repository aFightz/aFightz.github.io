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

SynchronousQueue
SynchronousQueue没有存储功能，因此put和take会一直阻塞，直到有另一个线程已经准备好参与到交付过程中。仅当有足够多的消费者，并且总是有一个消费者准备好获取交付的工作时，才适合使用同步队列。





生产者-消费者模式同样能带来许多性能优势。生产者和消费者可以并发地执行。如果一个是I/O密集型，另一个是CPU密集型，那么并发执行的吞吐率要高于串行执行的吞吐率。如果生产者和消费者的并行度不同，那么将它们紧密耦合在一起会把整体并行度降低为二者中更小的并行度。