---
对象的共享
date: 2019-03-22 10:49:43
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

未同步的情况下使用共享变量带来的问题
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
可能会有三种情况出现
程序正常结束，输出10
程序正常结束，输出0
正确无法结束，一直在无线循环。


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

public class Score {
    public volatile int value;

    //getter and setter
}

上面的两种方式的效果大体相同，但是前者可见性更强（16章会讲）。

volatile不会执行加锁操作，所以开销会比synchronized更小


volatile boolean asleep;

    while(!asleep){
        countSomeSheep();
    }
若asleep不设置为volatile，server模式的jvm运行此段代码会无限循环。
原因是jvm的server模式会比client模式优化的更多，譬如把在循环中未经过修改的变量（asleep）提升到循环外部。

volatile的正确使用方式
确保自身状态的可见性
确保所引用对象的状态的可见性（不是只能保证引用的可见性吗）
标识一些重要的程序生命周期事件的发生（例如：初始化或关闭）（加了volatile是否意味着对象的初始化是原子性的？）


内部类导致的线程不安全
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
上述例子中，ThisEscape有可能还没被创建完成，doSomething方法就被调用了。
可以在ThisEscape构造方法加锁避免这个问题。


不要在构造函数中，启动线程或者创建内部类，若一定要这么做，则可以使用工厂方法避免不必要的错误。
public class SafeListener{
    private final EventListener listener;
    public SafeListener(){
        listener = new EventListener(){
            public void onEvent(Event e){
                //doSomething属于ThisEscape
                doSomething(e);
            }
        }
    }
    
    public static SafeListener newInstance(EventSource source){
        SafeListener safe = new SafeListener();
        safe.registerListener(listener);
        return safe; 
    }
}

使用ThreadLocal保证线程安全

    public static ThreadLocal<Integer> value = new ThreadLocal<Integer>(){
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };

    public static Integer getValue(){
        return value.get();
    }
    
ThreadLocal内部用Map结构保存了当前线程（Thread.currentThread()）对应的值。所以每一个线程都有属于自己的value。


final
final能保证初始化过程中的安全性。
volatile不仅能保证初始化过程中的安全性，还能保证可见性。

final的重排序规则
在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。


对“使用不可变类来实现线程安全”的分析
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

public static class CacheUtil{
    private volatile Cache cache = new Cache(null , null);

    public Integer[] getFactors(Integer i){

        Integer[] factors = cache.getFactors();
        if(factors == null){
            //因式分解得到factors


            cache = new Cache(i , factors);//这一步是原子操作吗

        }
        return cache.getFactors();
    }
}
在这个Demo里，线程安全只指Cache类里的value变量与factors变量要保持同步一致，也就是说这两者要能同步初始化，同步更新。
重点看两个点：
CacheUtil的cache变量使用了volatile。
Cache的value、factors变量使用了final。
volatile保证cache的不会失效，这个显而易见。
value、factors变量如果不使用final，会出现什么情况呢？有可能在构造Cache对象的时候，这两个变量还没被赋值完毕，cache引用就已经被传递出去了。导致其他线程读取的value与factors会有错误。
但是volatile能保证构造对象的安全性，那么是不是意味着这里不用final也行呢？


创建一个可以让其他线程访问的可变对象的四种模式
在静态初始化函数中初始化一个对象引用。
将对象的引用保存到volatile或AtomicReferance对象中。
用final修饰对象的引用
将对象的引用保存到一个由锁保护的域中











