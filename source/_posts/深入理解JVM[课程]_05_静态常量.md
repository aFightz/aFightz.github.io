---
title: 05 | 静态常量
date: 2019-10-22 12:33:17
tags: [JVM]
categories :
- 学习笔记
- JVM
- 深入理解JVM[课程]
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">静态常量的本质含义</font></center>
静态常量在**编译阶段**会存入到**调用这个静态常量的方法所在的类的静态常量池**中，本质上，调用类并没有直接引用到定义静态常量的类，因此并**不会触发定义静态常量的类的初始化**。如下：


**Demo**
```java
class Parent{
    public static final int parent = 1;

    static {
        System.out.println("parent");
    }
}
```

```java
public class Test{
    public static void main(String[] args) {
        /**由于将静态常量存放到了Test的静态常量池中，不触发Test的初始化。所以输出：
         * 1
         * 
         * 我们甚至可以在编译完之后将Parent.class文件删除也不会影响程序运行结果(说明不会去加载Parent，更别说去初始化了)。
         */
        System.out.println(Parent.parent);
    }
}
```

但是，当一个静态常量的值**并非编译期间可以确定的**，那么其值就不会被放到调用类的静态常量池中，这时在程序运行时，会主动使用这个静态常量所在的类，显然会导致这个类被初始化。将上述的Parent类改一下，如下：

**Demo**
```java
class Parent{
    public static final int parent = UUID.randomUUID().toString();
    
    static {
        System.out.println("parent");
    }
}
```
Paren类将会被初始化，输出“parent”。

> 同理可得，非静态常量也不是非编译期间可以确定的，所以也不会被放到调用类的常量池中。

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">反编译与助记符</font></center>

**1、反编译命令**
```javap -c [.class path]```
> 反编译上面的Test.class文件，会发现Test的字节码文件中确实没有引用到到Parent，而是使用下述的助记符直接推静态常量。

**2、助记符**
- **ldc**：表示将int，float或是string类型的静态常量值从静态常量池中推送至栈顶。
- **sipush**：表示格一个短整型静态常量值（-32768~32767）推送至栈顶。
- **bipush**：表示将单字节（-128~127）的静态常量值推送至栈顶。
- **iconst_1**：表示将int类型1推送至栈顶（iconst_1~iconst_5）。

> double、char、byte类型呢？
是否是优先使用表示“小区间”的助记符，如果不符合再使用“大区间”的？