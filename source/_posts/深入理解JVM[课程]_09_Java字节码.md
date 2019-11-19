---
title: 09 | Java字节码
date: 2019-11-12 12:22:00
tags: [JVM]
categories :
- 学习笔记
- JVM
- 深入理解JVM[课程]
---

<center> <h4><font color = "#36648B">✎</br>字节码文件结构</center>
字节码文件由以下部分组成，如下图：
![](深入理解JVM[课程]_09_Java字节码\java字节码结构.png)





jclasslib

从字节码的角度上来看，每个实例化方法的入参都会有传入一个**this**。
> 入参必须会存放到局部变量表中。

假设方法中有try catch的语法，那么在**运行时**局部变量表会多一个记录异常的变量。
JVM对异常的处理方式
1.采用异常表的方式来对异常进行处理。
2.当异常处理存在finally语句块时，现代化的JVM采取的处理方式是将fina1ly语句块的字节码拼接到每一个catch块后面，换句话说，程序中存在多少个catch块，就会在每一个catch块后面重复多少个finally语句块的字节码。|


> 每个类的常量池都有许多值相同的常量，那么这些常量是共享的吗？类的常量池和我们平常所说的常量池有什么区别？


对于Java类中的每一个实例方法（非static方法），其在编译后所生成的字节码当中，方法参数的数量总是会比源代码中方法参数的数量多一个（this），它位于方法的第一个参数位置处。




栈帧（stack frame）
栈帧是一种用于帮助虚拟机执行方法调用与方法执行的数据结构。
栈帧本身是一种数据结构，封装了方法的局部变量表、动态链接信息、方法的返回地址以及操作数栈等信息。

有些符号引用是在类加载阶段或是第一次使用时就会转换为直接引用，这种转换叫做静态解析；另外一些符号引用则是在每次运行期转换为直接引用，这种转换叫做动态链接，这体现为Java的多态性。

静态解析的4种情形：
1.静态方法
2.父类方法
3.构造方法
4.私有方法（无法被重写）
以上4类方法称作非虚方法，他们是在类加载阶段就可以将符号引用转换为直接引用的。
> 公有方法有可能被overwrite，所以不能被静态解析。

方法的静态分派Demo
```java
public class Test6 {
    public void test(Grandpa grandpa){
        System.out.println("Grandpa");
    }

    public void test(Father father){
        System.out.println("Father");
    }

    public void test(Son son){
        System.out.println("Son");
    }

    public static void main(String[] args) {
        Grandpa p1 = new Father();
        Grandpa p2 = new Son();
        Test6 test6 = new Test6();
        test6.test(p1); //输出Grandpa
        test6.test(p2); //输出Grandpa
    }
}

class Grandpa{

}

class Father extends Grandpa{

}

class Son extends Father{

}
```
以上代码，g1的静态类型是Grandpa，而g1的实际类型（真正指向的类型）是Father。
变量的静态类型是不会发生变化的，而变量的实际类型则是可以发生变化的（多态的一种体现），实际类型是在运行期方可确定。
方法重载，是一种静态的行为，编译期就可以完全确定。




方法调用
1.invokeinterface：调用接口中的方法，实际上是在运行期决定的，决定到底调用实现该接口的哪个对象的特定方法。
2.invokestatic：调用静态方法。
3.invokespecial：调用自己的私有方法、构造方法（<init>）以及父类的方法。
4.invokevirtual：调用虚方法，运行期动态查找的过程。
5.invokedynamic：动态调用方法。