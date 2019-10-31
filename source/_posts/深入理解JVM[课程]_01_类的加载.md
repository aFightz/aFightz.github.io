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

#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">获取类加载器的途径</font></center>
- **获得当前类的ClassLoader**：`clazz.getClassLoader()`。
- **获得当前线程上下文的ClassLoader**：`Thread.currentThread().getContextClassLoader()`。
- **获得系统的ClassLoader**：`ClassLoader.getSystemClassLoader()`。
- **获得调用者的ClassLoader**：`DriverManager.getCallerClassLoader()`。


#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">数组的类加载器</font></center>
数组的Class对象不是由类加载器加载的，而是由虚拟机**动态生成**的。
如果数组元素是原生类型，那么调用数组的`getClassLoader()`方法将会返回**null**（表示没有类加载器），否则返回的是**加载元素的类加载器**。

#### <center><font color = "#36648B">✎✎✎✎✎✎✎</font><br/><font color = "#36648B">代码示例</font></center>

**1、获取当前类的ClassLoader**
```java
public class Test{
    public static void main(String[] args){
       Class<?> clazz = Class.forName("java.lang.String");
       //获得String类的加载器
       ClassLoader classLoader = clazz.getClassLoader();
    }    
}
```
加载java.lang.String的类加载器是根类加载器，所以HotSpot虚拟机输出为null。（但是有些虚拟机不是输出为Null，并没有严格的规定）

**2、获取父加载器**
```java
public class Test{
    public static void main(String[] args){
       //获得系统类加载器（AppClassLoader）
       ClassLoader classLoader = ClassLoader.getSystemClassLoader();
       //获得系统类加载器的父加载器，也就是拓展类加载器（ExtClassLoader）
       classLoader = classLoader.getParent();
    }    
}
```

**3、自定义ClassLoaderDemo1**
```java
public class MyClassLoader extends ClassLoader{

    private String classLoaderName;

    public MyClassLoader(String classLoaderName) {
        super();//将AppClassLoader作为父加载器
        this.classLoaderName = classLoaderName;
    }

    public MyClassLoader(ClassLoader parent, String classLoaderName) {
        super(parent);//将parent作为父加载器
        this.classLoaderName = classLoaderName;
    }

    public Class<?> findClass(String className) throws ClassNotFoundException {
        //byte [] bytes = ...  通过className读取到字节数组
        return this.defineClass(className , bytes , 0 ,bytes.length);
    }
    
    public static void main(String[] args) throws Exception {
        MyClassLoader classLoader = new MyClassLoader("MyClassLoader");
        Class<?> clazz =classLoader.loadClass("LoaderTest");
        Object object =clazz.newInstance();
    }
}
class LoaderTest{
    
}
```
**loadClass的流程**：
- 若已经加载过此class，则直接返回内存里的Class对象。
- 否则用父加载器去加载此class。
- 若还是加载不成功，则调用findClass方法去加载class。

> **问题1**：jvm在启动时，会根据.class文件创建相应的Class对象，如果后续通过LoadClass手动生成Class对象，这两个Class对象有无联系？
  答：LoadClass在加载类的时候，会判断此类是不是已经被加载过，如果已经被加载过，则直接返回Class对象。所以这两个Class对象其实是指向同一个地址的。
  **问题2**：假设有对象TT，首先`new TT()`（也就是首次主动调用），后续通过LoadClass手动生成TT的Class对象，再通过`newInstance()`方法生成一个TT对象，那么后者还算是首次主动调用吗？
  答：通过代码实验可知，这已经不算是“首次主动使用”了。
  **问题3**：上述代码要怎么体现双亲委托机制？如果用MyClassLoader去加载类，因为它的Parent ClassLoader是AppClassLoader（默认情况下），从而会一直往上委托，那么实际上加载类的应该是BootStrap ClassLoader？
  **问题4**：在什么情况下ClassLoader才会判断自己无法加载类？
  **问题5**：发现一个很严重的问题：上述的Demo中findClass方法没有被调用。
  答：因为LoadClass这个方法会先调用父加载器（本例中为AppCladdLoader）去加载class。而不会往下走（执行findClass方法）。把上述例子中的class文件放到classpath外,就会调用findClass方法了。
  **问题6**：`Class<?> clazz = TT.class`这段代码会导致类的加载吗？
  **问题7**：如果新建两个MyClassLoader实例去加载classpath外的.class文件，会调用findClass两次（说明“类只会加载一次”的说法不成立？）。

#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">命名空间</font></center>
**1、定义**
每个类加载器都有自己的命名空间，命名空间由**该加载器及所有父加载器所加载的类**组成。
在同一个命名空间中，只会存在一个类。
> 本质上就是同一个加载器（同一个实例）不会加载一个类两次。