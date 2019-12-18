---
title: 07 | 虚拟机类加载机制
date: 2019-12-15 23:16:45
tags: [JVM]
categories :
- 学习笔记
- JVM
- 深入理解Java虚拟机
---


<center> <h4><font color = "#36648B">✎</br>类的生命周期</center>
- 加载。
- 连接。
  - 验证。
  - 准备。
  - 解析。
- 初始化。
- 使用。
- 卸载。
> 解析阶段有时会在初始化后面。
> 加载阶段与连接阶段是**交叉进行**的。


对`static int a = 1;`这样的语句，在准备阶段为把a赋值为0，而在初始化阶段才会赋值为1。但是如果是这样的语句`final static int a = 1;`，在准备阶段就会直接赋值为1。

<center> <h4><font color = "#36648B">✎✎</br>解析</center>

**1、符号引用的解析**
对于符号引用的解析，会对第一次解析的结果进行缓存。（除了`invokedynamic`这条指令）

**2、字段的解析**
假设解析成功后，字段所属的类是C。
- 先看C是否含有与目标相匹配的字段，有则返回此字段引用。
- 否则，递归往上查找C实现的接口，有则返回此字段引用。
- 否则，递归往上查找C的父类，有则返回此字段引用。
- 否则，报错。

实际上虚拟机限制的会比较严格，如果继承的类与实现的接口，都有同样的字段，会编译不通过，如下：
```java
public class Test {

    public static void main(String[] args) throws InterruptedException {

        Hehe hehe = new Hehe();
        System.out.println(hehe.a); //Reference to 'a' is ambiguous, both 'TestClass.a' and "TestInterface.a' match

    }

    public static class Hehe extends  TestClass implements TestInterface{

    }

    public static class TestClass{
        int a = 2;
    }
    public interface  TestInterface{
        int a = 1;
    }

}
```

**3、类方法解析**
假设解析成功后，方法所属的类是C。
- 如果在C中有匹配的方法，则直接返回。
- 否则递归在父类中查找相匹配的方法。
- 否则就会报错。

**4、接口方法解析**
假设解析成功后，方法所属的接口是C。
- 如果在C中有匹配的方法，则直接返回。
- 否则递归在父接口中查找相匹配的方法。
- 否则就会报错。


<center> <h4><font color = "#36648B">✎✎✎</br>初始化</center>
编译器可以为接口生成`<clinit>()`类构造器。其实初始化阶段就是执行`<clinit>()`方法的过程。
> `<init>()`是实例构造器。

