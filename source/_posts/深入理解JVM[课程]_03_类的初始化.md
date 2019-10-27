---
title: 03 | 类的初始化
date: 2019-10-20 16:19:55
tags: [JVM]
categories :
- 学习笔记
- JVM
- 深入理解JVM[课程]
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">初始化的功能</font></center>
为类的**静态变量**赋予正确的初始值。

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">初始化的时机</font></center>
每个类或接口在被Java程序“**首次主动使用**”时才被初始化，首次使用分为以下7种情况：
- 创建类的实例。
- 访问某个类或接口的静态变量，或者对该静态变量赋值。
- 调用类的静态方法。
- 反射。
- 初始化这个类的子类。
- Java虚拟机启动时被标明为启动类的类（带有main方法的类）。
- JDK1.7开始提供的动态语言支持：**java.lang.invoke.MethodHandle**实例的解析结果REF_getStatic，REF_putStatic，REF_invokeStatic句柄对应的类如果没有初始化，则初始化。

>调用**ClassLoader**类的`loadClass`方法加载一个类，并不是对类的主动使用，不会导致类的初始化。

**1、子类与父类初始化Demo**
```java
class Parent{
    public static int parent = 1;

    static {
        System.out.println("parent");
    }
}
class Child extends Parent{
    public static int child = 1;

    static {
        System.out.println("child");
    }
}

public class Test{
    static {
            System.out.println("main");
    }
    public static void main(String[] args) {
        /**首先会初始化main方法所在的类，之后使用到了Child的静态变量，所以Child需要被初始化，Child被初始化之前Child的父类也必须被初始化。输出：
         * main
         * parent
         * child
         * 1
         */
        System.out.println(Child.child);
    }
}
```
> 上述例子中的类加载顺序和类初始化顺序其实是一样的，可以通过开启VM参数去观察一下这三个类的加载顺序。
    
```java
public class Test2{
    public static void main(String[] args) {
        /**此时没有直接使用到Child的静态变量而使用了Parent的静态变量，所以Child没有被初始化而Parent被初始化了。输出：
         * parent
         * 1
         */
        System.out.println(Child.parent);
    }
}
```

**2、反射与类加载器初始化Demo**
```java
public class Test {

    public static void main(String[] args) throws ClassNotFoundException {
        ClassLoader loader = ClassLoader.getSystemClassLoader();
        loader.loadClass("CL");
        System.out.println("-----------");
        Class.forName("CL");
        /**
        *  输出：
        *  -----------
        *  CL
        *  
        *  说明类加载器加载类不会去初始化类，而反射会去初始化类。
        */
    }

}

class CL {
    static {
        System.out.println("CL");
    }
}
```

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">初始化顺序</font></center>
类的初始化顺序是**从上往下**的，如下Demo可以说明：
```java
public class MyTest6 {
    
    public static void main(String[] args){
        Singleton singleton = Singleton.getInstance(); 
        System.out.println("counter1: "+Singleton.counter1);
        System.out.printin("counter2: "+Singleton.counter2);
        /**
        *  输出的结果是:
        *  1
        *  0
        */
    } 
}

class Singleton {
    public static int counter1; 
    private static Singleton singleton = new Singleton(); 
    private Singleton(){
        counter1++;
        counter2++;      
    }

    public static int counter2 = 0;
    
    public static singleton getInstance(){
        return singleton;
    }
}
```
当singleton类被加载完连接完后，counter1与counter2都被赋为了0，Singleton被赋为了null，此时的调用getInstance()这个静态方法（主动使用），Singleton将被初始化，从上往下赋值：
- singleton变量被赋值，触发构造方法Singleton()，此时counter1为1，counter2为1。
- counter2被赋值为0。


> 注：类被加载进内存后，也可以被卸载。
