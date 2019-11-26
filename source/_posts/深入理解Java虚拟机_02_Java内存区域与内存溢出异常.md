---
title: 02 | Java内存区域与内存溢出异常
date: 2019-11-24 21:58:59
tags: [JVM]
categories :
- 学习笔记
- JVM
- 深入理解Java虚拟机
---

<center> <h4><font color = "#36648B">✎</br>程序计数器</center>
程序计数器是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。

每一个线程都会有一个独立的程序计数器。

<center> <h4><font color = "#36648B">✎✎</br>栈</center>
每个方法在执行的同时都会创立一个**栈帧**。

在oom时存储快照的参数：`-XX:+HeapDumpOnOutOfMemoryError`。

指定栈内存大小参数：`-Xss`(默认为512k)

抛出**StackOverflowError** 是因为栈内存不够，而不是栈深度到了指定的深度。

同一个类，如果被不同的类加载器加载两次，那么在栈中也就会存在两份Class对象。


<center> <h4><font color = "#36648B">✎✎✎</br>常量池</center>
jdk1.6中，String的`intern()`方法会把“**首次遇到**”的字符串实例复制到常量池中，并返回这个实例引用。
而在jdk1.7中，不再复制实例，而是记录这个实例的引用。(也就是说把实例放到了堆中)

demo
```java
public class Test {

    public static void main(String[] args) throws InterruptedException {

        String str =  new StringBuilder("A").toString();
        System.out.println(str.intern() == str);  //false 非常量池的对象

        String str2 =  new StringBuilder("B").append("C").toString();
        System.out.println(str4.intern() == str4);//true 为常量池对象
    }
}
```
上面的例子有点意思，在编译期间A、B、C就已经放入常量池了，所以新建值为A的实例与常量池值为A的实例肯定不同。而常量池没有值为BC的实例，所以当执行itern()方法时，自然就会把新建的“BC”实例给“放入”常量池，所以两只的引用指向的是同一个实例。

> str.itern()与str = ""这种方式赋值的区别是？
> 如果大量使用intern或者 str = ""这种写法，是不是会容易造成OOM？
> 常量是否也会被回收？


<center> <h4><font color = "#36648B">✎✎✎✎</br>堆</center>
**1、创建对象时，分配内存的方式有如下两种：**

- 如果内存是绝对规整的，就直接使用“**指针碰撞**”。（如果使用复制或带有压缩功能的算法，那么内存就是绝对规整的）
- 否则就使用“**空闲列表**”的方式，在一个列表中维护各个可用的内存块。需要时则直接在列表找到符合条件的内存块。

**2、在并发的情况下对分配内存的正确结果的保证有如下两种方式：**

-  采用CAS配上失败重试的方式。
- 给每个线程分配一小块内存（**TLAB**），若这小块内存不够用时，再**同步申请**新的TLAB。（是否使用这种方式可以由 -XX:+/-UseTLAB）

**3、内存中的实例对象包含以下3种数据**

- 对象头。
- 实例数据。
- 对齐填充。


<center> <h4><font color = "#36648B">✎✎✎✎✎</br>对象的访问定位</center>
对象的访问定位有以下两种方式
- **直接访问**。优点是访问速度快。
- **使用句柄间接访问**。优点是对象被移动时，只需要移动句柄，不需要动原来对象。


<center> <h4><font color = "#36648B">✎✎✎✎✎✎</br>直接内存</center>
Nio直接请求分配了**堆外内存**（也就是直接内存），**避免了在Java堆和Native堆中来回复制数据**。
直接内存容量可通过`-XX:MaxDirectMemorySize`指定，如果不指定，则**默认与Java堆最大值**（-Xmx指定）一样。
可通过反射获取`UnSafe`实例分配直接内存。

> Unsafe类的`getUnsafe()`方法限制了只有引导类加载器才会返回实例。所以只能用反射的方式去使用Unsafe。