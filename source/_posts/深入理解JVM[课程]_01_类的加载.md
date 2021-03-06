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
> 但是实际情况下，在编译完之后，删除某个类的.class文件，如果需要加载它，会直接报错，而不是等到初始化才报错。

类加载器并不需要等到某个类被“首次主动使用”时再加载它。
> 这里有例子可以证明吗？

若有一个类加载器能够成功加载Test类，那么这个类加载器被称为**定义类加载器**，所有能成功返回Class对象引用的类加载器（包括定义类加载器）都被称为**初始类加载器**。



#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">类加载器的类型</font></center>
- Java虚拟机自带的加载器
  - **根类加载器（Bootstrap）**：该加载器没有父加载器，它负责加载虚拟机的核心类库，如java.lang.*等。根类加载器从系统属性`sun.boot.class.path`所指定的目录中加载类库。根类加载器的实现依赖于底层操作系统（由C++编写），属于虚拟机的实现的一部分，它并**没有继承**`java.lang.ClassLoader`类
  - **扩展类加载器（Extension）**:它的父加载器为**根类加载器**。它从`java.ext.dirs`系统属性所指定的目录中加载类库，或者从JDK的安装目录的`jre\lib\ext`子目录（扩展目录）下加载类库，如果把用户创建的JAR文件放在这个目录下，也会自动由扩展类加载器加载。扩展类加载器是纯Java类，**是java.lang.ClassLoader类的子类**。
  > 扩展类加载器只会去加载.jar结尾的文件，而不会去加载.class结尾的文件。
  
  - **系统/应用类加载器（System）**（AppClassLoader）:也称为应用类加载器，它的父加载器为**扩展类加载器**。它从环境变量`classpath`或者系统属性`java.class.path`所指定的目录中加载类，它是**用户自定义的类加载器的默认父加载器**。系统类加载器是纯Java类，**是java.lang.ClassLoader类的子类**。
- 用户自定义的类加载器
  
  - **用户自己定制的java.lang.ClassLoader的子类。**

> **根类加载器 > 扩展类加载器 > 系统/应用类加载器 > 自定义的类加载器**(箭头左边为箭头右边的父加载器，但是它们之间的关系并不是继承)
  拓展类加载器与应用加载器是由根类加载器去加载的。



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
如果数组元素是原生类型，那么调用数组的`getClassLoader()`方法将会返回**null**，否则返回的是**加载元素的类加载器**。

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
Class.forName的类加载器**默认是调用者的类加载器**。且所加载的类**默认会被初始化**。
> `Reflection.getcallerClass()`可以知道调用者是哪个类。

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

> **问题1**：jvm在启动时，类加载器A根据.class文件创建相应的Class对象，如果后续此类加载器通过LoadClass手动生成Class对象，这两个Class对象有无联系？
  答：LoadClass在加载类的时候，会判断此类是不是已经在命名空间内，如果在，则直接返回Class对象。所以这两个Class对象其实是指向同一个地址的。
  **问题2**：假设有对象TT，首先`new TT()`（也就是首次主动调用），后续通过LoadClass手动生成TT的Class对象，再通过`newInstance()`方法生成一个TT对象，那么后者还算是首次主动调用吗？
  答：如果这两个Class对象是同一个，那么就不算是“首次主动使用”。
  **问题3**：上述代码要怎么体现双亲委托机制？如果用MyClassLoader去加载类，因为它的Parent ClassLoader是AppClassLoader（默认情况下），从而会一直往上委托，那么实际上加载类的应该是BootStrap ClassLoader？
  答：每一个类加载器都有自己指定加载类的路径。每一个类加载器都执行这样的逻辑：如果Parent ClassLoader加载不到才由自己本身去加载.class。
  **问题4**：在什么情况下ClassLoader才会判断自己无法加载类？
  答：在自己的findClass方法逻辑中定义的。
  **问题5**：发现一个很严重的问题：上述的Demo中findClass方法没有被调用。
  答：因为LoadClass这个方法会先调用父加载器（本例中为AppClassLoader）去加载class。而不会往下走（执行findClass方法）。把上述例子中的class文件放到classpath外,就会调用findClass方法了。
  **问题6**：`Class<?> clazz = TT.class`这段代码会导致类的加载吗？如果会被加载，那么是被哪个加载器加载？
  答：会导致类的加载，类加载器将是当前线程上下文类加载器。
  **问题7**：如果新建两个MyClassLoader实例去加载classpath外的.class文件，会调用findClass两次（说明“类只会加载一次”的说法不成立？）。
  答：由于实例不同，此时命令空间已经不同了，当然会去加载两次。
  **问题8**：如果系统加载器与自定义加载器均加载了同样的类（内存中存在两个Class对象），那么`Class<?> clazz = TT.class`这样子调用的流程又会是怎么样？
  答：如果是加载了此全类名相同的类，那么说明自定义加载器与系统加载器没有任何关系。那么这一句代码使用的类加载器将会是线程上下文类加载器。
  **问题9**：自定义加载器加载的类可以引用系统加载器加载的类吗？
  答：如果自定义加载器与系统加载器无任何关系，那么两者的类是完全隔离的。

> 若有两个jar包，两个包都存在全类名相同的类，那么会加载路径在前的jar包。


#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">命名空间</font></center>
**1、定义**
每个类加载器都有自己的命名空间，命名空间由**该加载器及所有父加载器所加载的类**组成。
在同一个命名空间中，只会存在一个类。
> 本质上就是同一个加载器（同一个实例）不会加载一个类两次。

#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">类之间的引用</font></center>
每个类都会使用自己的类加载器（即加载自身的类加载器）来去加载所依赖的类。
> 比如，如果A引用了B类，假设A类的定义类加载器是a，那么此时B类也会被a去加载（前提是a未被加载）。

> **问题1**：如果.class（全类名名相同）由两个不同的类加载器加载，生成两个Class对象，那么由这两个Class对象生成的实例会有什么不同？
  1、既然全类名相同的两个.class能被两个不同的类加载器加载，说明这两个加载器不存在任何关系，否则在       委托加载的机制下，其中一个加载器肯定会委托另外一个加载器加载的。
  2、由这两个全类名相同的Class对象生成的实例实际上**不是同一个类型**。
  3、所以实际上**类型 = classloader实例 + 全类名**。
  **问题2**：如果用同一个classloader实例去加载全类名相同的两个类（存放路径不同），那么结果会是如何？
  答：以先加载的那个.class为准。后面的会被忽略。因为在后面加载时，加载器会认为此类已经被加载过了（即使存放的路径不同）。



<center> <h4><font color = "#36648B">✎✎✎✎✎✎✎✎✎✎</br>双亲委托机制的好处</center>

- 因为所有的类加载的上层加载器中都**必然会有bootstrap classloader**。所以能确保java核心类库（`sun.boot.class.path`下的类）是由bootstrap classloader 所加载的，确保了核心库的安全。
- 除了`sun.boot.class.path`下的类，我们可以通过自定义的加载器创建其他类的独立命名空间。



<center> <h4><font color = "#36648B">✎✎✎✎✎✎✎✎✎✎✎</br>Launcher</center>

Launcher是由**根类加载器**去加载的。

**1、自定义系统类加载器**
根据`java.lang.ClassLoader#getSystemClassLoader`的描述可知，我们执行以下步骤指定自定义的类加载器为系统类加载器。

- 自定义类加载器必须要有一个构造方法，它的参数是ClassLoader。（用来指定自定义类加载器的parent classloader）。
  
  > 自定义类加载器由原来默认的系统类加载器加载。且它的parent classloader是默认的系统类加载器。
- 通过`java.system.class.loader`  属性指定自己实现的加载器为系统类加载器。

**2、Launcher的构造**
在Launcher的构造方法中，构造了**ExtClassLoader**与**AppClassLoader**。

<center> <h4><font color = "#36648B">✎✎✎✎✎✎✎✎✎✎✎✎</br>线程上下文类加载器</center>

Thread类的类加载器是**BootStrap ClassLoader**。

线程上下文类加载器是从JDK 1.2开始引入的，Thread类中的`getcontextclassLoader()`与`setContextClassLoader(ClassLoader cl)`分别用来获取和设置上下文类加载器。
如果没有显示对上下文类加载器进行设置的话，线程将**继承其父线程的上下文类加载器**。Java应用运行时的初始线程的上下文类加载器是**系统类加载器**。

在双亲委托机制下，父加载器是不能使用子加载器所加载的类的。但是如果使用父加载器的线程中设置线程上下文加载器为子加载器，就可以解决这个问题了。
> 这在SPI（Service Provider Interface）中很常见，比如说JDBC等。对于SPI来说，有些接口是Java核心库所提供的，而Java核心库是由启动类加载器来加载的，而这些接口的实现却来自于不同的jar包（厂商提供），一般情况下Java的启动类加载器是会加载其他来源的jar包，这样传统的双亲委托模型就无法满足SPI的要求。而通过给当前线程设置上下文类加载器，就可以由设置的上下文类加载器来实现对于接口实现类的加载。

> 这种场景如果用spring去实现是不是也能很好的解决呢？只关注实例bean，而不关注类。

线程上下文器的使用的代码示例
```java
public class Test{
    public void use(){
        Claastoader originClassloader = Thread.currentehread().getcontextclaastoader();
        try{
            //设置为目标类加载器
            Thread.currentThread().setcontextclassloader(targetClassLoader);
            //dosomething  做一些类加载的操作
        }finally{
            Thread.currentThread().setcontextclassloader(originClassloader);
        }
    }
}
```



<center> <h4><font color = "#36648B">✎✎✎✎✎✎✎✎✎✎✎✎✎</br>jar hell问题的定位</center>

```java
public class Test {

    public static void main(String[] args) throws  IOException {
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        String resourceName = "Test.class";
        Enumeration<URL> urls = cl.getResources(resourceName);

        while (urls.hasMoreElements()){
            URL url = urls.nextElement();
            System.out.println(url);
        }
    }

}
```
上述demo可以打印出`Test.class`存在哪几个jar包中。

