---
title: 07 | 异常
date: 2019-09-09 19:53:31
tags: [Java基础]
categories :
- 学习笔记
- Java
- Java核心技术卷1
---
#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">Java异常层次结构</font></center>
![](Java核心技术卷1_07_异常\异常结构.jpg)

Error类层次结构描述了Java运行时系统的内部错误和资源耗尽错误。应用程序不应该抛出这种类型的对象。

**Exception**这个层次结构又分解为两个分支：
- **RuntimeException**：由程序错误导致的异常属于**RuntimeException**。其异常包含下面几种情况：
  - 错误的类型转换。
  - 数组访问越界。
  - 访问null指针。
- **其他异常**：由于像**I/O错误**这类问题导致的异常属于其他异常。其异常包含以下几种情况：
  - 试图在文件尾部后面读取数据。
  - 试图打开一个不存在的文件。
  - 试图根据给定的字符串查找Class对象，而这个字符串表示的类并不存在。

如果出现RuntimeException异常，那么就一定是程序的问题。

派生于Error类或RuntimeException类的所有异常称为**非受查（unchecked）异常**，所有其他的异常称为**受查（checked）异常**。
受查异常一定要try-catch捕获或者throw出去。

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">异常相关代码编写</font></center>
**1、合并 catch 子句**
```java
try{
    //code that might throw exceptions
}catch (IOException | FileNotFoundException e){
    e.printStackTrace();
}
```
只有当捕获的异常类型彼此之间不存在子类关系时才需要这个特性。
捕获多个异常时， 异常变量隐含为**final变量**。


**2、再次抛出异常与异常链**
在catch子句中可以抛出一个异常，这样做的目的是改变异常的类型，
```java
try{
    ...
}catch (SQLException e){
    throw new IOException(e.getMessage());
}
```
可以有一种更好的处理方法，并且将原始异常设置为新异常的“原因”：
```java
try{
    ...
}catch (SQLException e){
    Throwable se = new IOException("Exception 转换");
    se.initCause(e);
    throw se;
}
```
当捕获到异常时， 就可以使用下面这条语句重新得到原始异常：
```java
Throwable se = se.getCause();
```
如果在一个方法中发生了一个受查异常，而不允许抛出它，那么包装技术就十分有用。我们可以捕获这个受查异常，并将它包装成一个运行时异常。

**3、finally块抛异常**
假设在try语句块中的代码抛出了一些非IOException的异常，并throw了出去。最后执行finally语句块，并调用close方法。而close方法本身也有可能抛出IOException异常。当出现这种情况时，原始的异常将会丢失，转而抛出close方法的异常。

**4、带资源的 try语句**
资源如果实现了`AutoCloseable`接口或者`Closeable`接口（Closeable是AutoCloseable的子接口），可以用如下方法关闭资源：
```java
try(PrintWriter writer = new PrintWriter("/");PrintWriter writer1 = new PrintWriter("/")){
    writer...
    writer1...
} catch (FileNotFoundException e) {
    e.printStackTrace();
}
```
无论这个块是否正常退出，writer都会自动关闭。
如果关闭的时候抛出异常，这些异常将自动捕获，并由`addSuppressed`方法增加到原来的异常。可以调用`getSuppressed`方法，它会得到从close方法抛出的异常列表。