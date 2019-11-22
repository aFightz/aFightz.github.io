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
|       名称       | 所占空间(字节) |
| :--------------: | :------------: |
|   Magic Number   |       4        |
|     Version      |       4        |
|  Constant Pool   |      2+n       |
|   Access Flags   |       2        |
| This Class Name  |       2        |
| Super Class Name |       2        |
|    Interfaces    |      2+n       |
|      Fields      |      2+n       |
|     Methods      |      2+n       |
|    Attributes    |      2+n       |

> 每个类的常量池都有许多值相同的常量，那么这些常量是共享的吗？类的常量池和我们平常所说的常量池有什么区别？

<center> <h4><font color = "#36648B">✎✎</br>方法</center>
对于Java类中的每一个**实例方法**（非static方法），其在编译后所生成的字节码当中，方法参数的数量总是会比源代码中方法参数的数量多一个（**this**），它位于方法的第一个参数位置处。

> 入参一定会存放到局部变量表中。


<center> <h4><font color = "#36648B">✎✎✎</br>异常</center>
假设方法中有`try catch`的语法，那么在**运行时**局部变量表会多一个记录异常的变量。

**1、JVM对异常的处理方式**

- 采用**异常表**的方式来对异常进行处理。
- 当异常处理存在`finally`语句块时，现代化的JVM采取的处理方式是将finally语句块的字节码拼接到每一个catch块后面，换句话说，程序中存在多少个catch块，就会在每一个catch块后面重复多少个finally语句块的字节码。


<center> <h4><font color = "#36648B">✎✎✎✎</br>栈帧（stack frame）</center>
栈帧是一种用于帮助虚拟机执行方法调用与方法执行的数据结构,封装了方法的局部变量表、动态链接信息、方法的返回地址以及操作数栈等信息。



<center> <h4><font color = "#36648B">✎✎✎✎✎</br>方法调用</center>
- **invokeinterface**：调用接口中的方法，实际上是在运行期决定的，决定到底调用实现该接口的哪个对象的特定方法。
- **invokestatic**：调用静态方法。
- **invokespecial**：调用自己的私有方法、构造方法以及父类的方法。
- **invokevirtual**：调用虚方法，运行期动态查找的过程。
- **invokedynamic**：动态调用方法。

<center> <h4><font color = "#36648B">✎✎✎✎✎</br>静态解析与动态链接</center>
有些符号引用是在类加载阶段或是第一次使用时就会转换为直接引用，这种转换叫做静态解析；另外一些符号引用则是在每次运行期转换为直接引用，这种转换叫做动态链接，这体现为Java的多态性。

**1、静态解析的4种情形**

- 静态方法
- 父类方法
- 构造方法
- 私有方法（无法被重写）

以上4类方法称作非虚方法，他们是在类加载阶段就可以**将符号引用转换为直接引用**的。

> 公有方法有可能被重写，所以不能被静态解析。

**2、方法的静态分派Demo**

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
        Grandpa g1 = new Father();
        Grandpa g2 = new Son();
        Test6 test6 = new Test6();
        test6.test(g1); //输出Grandpa
        test6.test(g2); //输出Grandpa
    }
}

class Grandpa{

}

class Father extends Grandpa{

}

class Son extends Father{

}
```
以上代码，g1的**静态类型是**Grandpa，而g1的实际类型（真正指向的类型）是Father。
变量的静态类型是**不会发生变化**的，而变量的实际类型则是可以发生变化的（多态的一种体现），实际类型是在运行期方可确定。
**方法重载，是一种静态的行为**，编译期就可以完全确定。

**3、方法动态分派Demo**
```java
public class Test {
    public static void main(String[] args) {
        Fruit f1 = new Apple();
        Fruit f2 = new Banana();
        f1.test(); //输出Apple
        f2.test(); //输出Banana
    }

}

class Fruit{
    public void test(){
        System.out.println("Fruit");
    }
}

class Apple extends Fruit{
    public void test(){
        System.out.println("Apple");
    }
}

class Banana extends Fruit{
    public void test(){
        System.out.println("Banana");
    }
}
```

**方法重写是一种动态的行为**，在运行期才可以决定。

> 总的来说就是：
调用哪个方法（重载导致有多个同名方法），是由所传入的引用类型决定的。
确定完调用方法之后，再去寻找调用这个方法的实例，这是由实际类型所决定的。






<center> <h4><font color = "#36648B">✎✎✎✎✎✎</br>操作数栈</center>
```java
public class Test {
    public static void main(String[] args) {
        int a = 1;
        int b = 2;
        int c = 3;
        int max = a+b+c;
    }

}
```

如上代码显示了一个很简单的相加运算，用jclasslib打开它的.class文件，展示的字节码逻辑如下：


```
0 iconst_1
1 istore_1
2 iconst_2
3 istore_2
4 iconst_3
5 istore_3
6 iload_1
7 iload_2
8 iadd
9 iload_3
10 iadd
11 istore 4
13 return
```

iconst_1表示将1入栈，istore_1表示将栈顶元素存储到局部变量表index为1的地方，以此类推。
iload_1表示读取局部变量表index为1的变量，压入栈。
iadd表示连续两次将栈顶弹出，并将弹出的元素相加，得出的结果压入栈。
istore 4表示将栈顶元素赋值给局部变量表中index为4的变量。

