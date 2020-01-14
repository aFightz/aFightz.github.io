---
title: 03 | 对象的共享
date: 2019-03-22 10:49:43
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

<center> <h4><font color = "#36648B">✎</br>未同步的情况下使用共享变量带来的问题</center>

```java
public class NoVisibility {
    private static boolean ready;
    private static int value;

    public static class ReaderThread extends Thread{
    
        public void run() {
            while(!ready){
                Thread.yield();
            }
            System.out.println(value);
        }
    }
    public static void main(String[] args){
        new ReaderThread().start();
        value = 10;
        ready = true;
    }
}
```
可能会有三种情况出现
- 程序正常结束，输出10
- 程序正常结束，输出0
- 正确无法结束，一直在无限循环。

> 涉及到的知识点:
- volatile与Java线程模型
- 重排序的概念
- 多线程的环境下重排序导致的问题(可以将两个线程变成一个流程，串行去理解)
- volatile防止重排序


<center> <h4><font color = "#36648B">✎✎</br>volatile与synchronized</center>

```java
public class Score {
    public int value;
    
    //强制从主存拿value
    public synchronized int getValue() {
        return value;
    }
    
    //强制把value刷到主存
    public synchronized void setValue(int value) {
        this.value = value;
    }
}
```

```java
public class Score {
    public volatile int value;

    //getter and setter
}
```
上面的两种方式的效果大体相同，但是前者可见性更强（16章会讲）。

volatile不会执行加锁操作，所以开销会比synchronized更小。

<center> <h4><font color = "#36648B">✎✎</br>内部类导致的线程不安全（this引用逸出）</center>

```java
    public class ThisEscape{
        public ThisEscape(EventSource source){
            source.registerListener(
                new EventListener(){
                    public void onEvent(Event e){
                        //doSomething属于ThisEscape
                        doSomething(e);
                    }
                }
            );
        }
    }
```
上述例子中，ThisEscape有可能还没被创建完成，doSomething方法就被调用了。
可以使用工厂方法避免这种的错误，如下：

```java
public class SafeListener{
    private final EventListener listener;
    private SafeListener(){
        listener = new EventListener(){
            public void onEvent(Event e){
                //doSomething属于ThisEscape
                doSomething(e);
            }
        };
    }
    
    public static SafeListener newInstance(EventSource source){
        SafeListener safe = new SafeListener();
        source.registerListener(safe.listener);
        return safe; 
    }
}
```
总的思路就是，在构造完对象之前,不能让this逸出。
> 如果listener对象不是final类型或者volatile类型的，也是属于不安全发布的。


<center> <h4><font color = "#36648B">✎✎✎</br>使用ThreadLocal保证线程安全</center>

```java
    public static ThreadLocal<Integer> value = new ThreadLocal<Integer>(){
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };
    
    public static Integer getValue(){
        return value.get();
    }
```

ThreadLocal内部用Map结构保存了当前线程（Thread.currentThread()）对应的值。所以每一个线程都有属于自己的value。
ThreadLocal变量类似于全局变量，它能降低代码的可重用性。例如将某个全局变量作为ThreadLocal对象。

<center> <h4><font color = "#36648B">✎✎✎✎</br>final</center>
final能保证初始化过程中的安全性（如果this引用逸出还能保证安全性吗）。
volatile不仅能保证初始化过程中的安全性，还能保证可见性。

**final的重排序规则**
- 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
- 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。

> <font color = "#7EC0EE">总的来说就是final在写之前不会被读。</font>


<center> <h4><font color = "#36648B">✎✎✎✎✎</br>对“使用不可变类来实现线程安全”的分析</center>
```java
public static class Cache{
    private final Integer value;
    private final Integer[] factors;

    public Cache(Integer value , Integer[] factors){
        this.value = value;
        this.factors = factors;
    }
    
    public Integer getValue() {
        return value;
    }
    
    public Integer[] getFactors() {
        return Arrays.copyOf(factors , factors.length);
    }
}
```
```java
public static class CacheUtil{
    private volatile Cache cache = new Cache(null , null);

    public Integer[] getFactors(Integer i){
        Integer[] factors = cache.getFactors(i);
        if(factors == null){
            //因式分解得到factors
            
            cache = new Cache(i , factors);//volatile类型的初始化是线程安全的    
        }
        return factors;
    }
}
```

在这个Demo里，线程安全是指Cache类里的value变量与factors变量要保持同步一致，也就是说这两者要能同步初始化，同步更新。由于没有使用DCL，所以可能会出现初始化两次的情况，但是这没关系，因为逻辑就是只保存最新的因式分解结果。
CacheUtil的cache变量使用了volatile。volatile保证cache不会失效，以及保证cache的初始化是线程安全的。


<center> <h4><font color = "#36648B">✎✎✎✎✎</br>安全发布一个对象的四种模式</center>
- 在静态初始化函数中初始化一个对象引用。
- 将对象的引用保存到volatile或AtomicReferance对象中。
- 用final修饰对象的引用。
- 将对象的引用保存到一个由锁保护的域中。
> 即使this引用逸出，volatile、Atomic、final也会是线程安全的吗？
> 锁与AtomicReferance是如何保证安全发布的？
> 线程安全的容器类也属于锁保护的域。



#### 疑惑
- NoVisibilityDemo实际运行的时候，如果加上Thread.yield()，其他线程ready会拿到最新值，而如果去掉Thread.yield()让while无限循环，那么ready则不会取到最新值，这是为什么？
- volatile类型能保证long、double的get/set是原子的吗？
- 进制volatile的重排序意义是什么？因为其他类型应该都不是共享的。


锁
假设有锁M，那么在M上调用unlock之前的所有操作结果，对于在M上调用lock之后的线程都是可见的。





