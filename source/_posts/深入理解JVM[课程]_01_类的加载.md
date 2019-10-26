---
title: 01 | 类的加载
date: 2019-10-20 13:19:55
tags: [JVM]
categories :
- 学习笔记
- JVM
- 深入理解JVM[课程]
---

在Java代码中，类型的加载、连接与初始化过程都是在**程序运行期间**完成的

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">相关定义</font></center>
类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在内存中创建一个**java.lang.Class**对象用来封装类在方法区内的数据结构。
> 规范并未说明Class对象位于哪里，HotSpot虚拟机将其放在了方法区中。

JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了.class文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误（LinkageError错误）如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误。
> 但是在编译完之后，删除某个类的.class文件，如果需要加载它，会直接报错，而不是等到初始化才报错。

若有一个类加载器能够成功加载Test类，那么这个类加载器被称为**定义类加载器**，所有能成功返回Class对象引用的类加载器（包括定义类加载器）都被称为**初始类加载器**。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">类加载器的类型</font></center>
- Java虚拟机自带的加载器
  - **根类加载器（Bootstrap）**：该加载器没有父加载器，它负责加载虚拟机的核心类库，如java.lang.*等。根类加载器从系统属性`sun.boot.class.path`所指定的目录中加载类库。根类加载器的实现依赖于底层操作系统，属于虚拟机的实现的一部分，它并**没有继承**`java.lang.ClassLoader`类
  - **扩展类加载器（Extension）**:它的父加载器为**根类加载器**。它从`java.ext.dirs`系统属性所指定的目录中加载类库，或者从JDK的安装目录的`jre\lib\ext`子目录（扩展目录）下加载类库，如果把用户创建的JAR文件放在这个目录下，也会自动由扩展类加载器加载。扩展类加载器是纯Java类，**是java.lang.ClassLoader类的子类**。
  - **系统/应用类加载器（System）**（AppClassLoader）:也称为应用类加载器，它的父加载器为**扩展类加载器**。它从环境变量`classpath`或者系统属性`java.class.path`所指定的目录中加载类，它是**用户自定义的类加载器的默认父加载器**。系统类加载器是纯Java类，**是java.lang.ClassLoader类的子类**。
- 用户自定义的类加载器
  - **用户自己定制的java.lang.ClassLoader的子类。**

> 类加载器并不需要等到某个类被“首次主动使用”时再加载它。
> **根类加载器 > 扩展类加载器 > 系统/应用类加载器 > 自定义的类加载器**(箭头左边为箭头右边的父加载器，但是它们之间的关系并不是继承)


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">加载信息的打印</font></center>
可用`-XX:+TraceClassLoading`这个vm参数打印vm加载类的信息。

#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">类加载的机制</font></center>
类加载器用来把类加载到Java虚拟机中。从JDK1.2版本开始，类的加载过程采用**双亲委托机制**，这种机制能更好地保证Java平台的安全。在此委托机制中，**除了Java虚拟机自带的根类加载器**以外，其余的类加载器都有且只有一个父加载器。当Java程序请求加载器loader1加载Sample类时，loader1首先委托自己的父加载器去加载Sample类，若父加载器能加载，则由父加载器完成加载任务，否则才由加载器loader1本身加载Sample类。


#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">代码示例</font></center>

**1、输出ClassLoader**
```java
Class<?> clazz = Class.forName("java.lang.String"); 
System.out.println(clazz. getclassLoader());
```
加载java.lang.String的类加载器是根类加载器，所以HotSpot虚拟机输出为null。（但是有些虚拟机不是输出为Null，并没有严格的规定）
