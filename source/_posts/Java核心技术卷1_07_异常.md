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