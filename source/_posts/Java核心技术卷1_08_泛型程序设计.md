---
title: 08 | 泛型程序设计
date: 2019-09-11 14:15:35
tags: [Java基础]
categories :
- 学习笔记
- Java
- Java核心技术卷1
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">泛型的定义</font></center>
**1、泛型类的定义**
```java
public class MyPair <T,U>{
    private T data;
}
```
泛型类可以有多个类型变量。像上面的T、U。
类型变量使用大写形式，且比较短。

**2、泛型方法的定义**
```java
public static <T> T say(T... a){
    return a[0];
}

int result = MyMethod.<Integer>say(1,2,3); //这种调用可以省略 -> MyMethod.say(1,2,3)。
```


```java
double result = MyMethod.say(3.14,1);
```
上面的方法会报错，原因是：编译器将会自动打包参数为1个Double和1个Integer对象，而后寻找这些类的共同超类型，以确定T究竟是什么类型。

```java
public final class Double extends Number implements Comparable<Double>;
public final class Integer extends Number implements Comparable<Integer>;
```
由上面代码可知，Double和Integer的共同超类时Number，所以只要改成如下既不会报错：
```java
Number result = MyMethod.say(3.14,1);
```


**3、类型变量的限定**
```java
public static <T extends Parent&Serializable> void say1(T a){
    a.say();
}
```
如果要调用这个方法，T必须要实现**Parent接口**和**Serializable接口**。
限定中，可以有限定多个接口，但是只能限制一个父类，且父类必须放到**第一位**。

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">泛型代码和虚拟机</font></center>
**1、原始类型**
无论何时定义一个泛型类型， 都自动提供了一个相应的**原始类型**（raw type)。

`Pair<T>`的原始类型如下：
```java
public class Pair {
    private Object data;
}
```
因为T没有限定变量，所以T就被替换成Object类型，如果有限定变量，就替换为限定变量，如果有多个限定变量，就替换为**第一个限定变量**。
`Pair<T extends Parent>`的原始类型如下：
```java
public class Pair {
    private Parent data;
}
```

```java
class Interval<T extends Serializable & Comparable>
```
上面的声明是低效率的，因为原始类型用Serializable替换T, 而编译器在必要时要向Comparable插入强制类型转换。为了提高效率，应该将标签（tagging) 接口（即没有方法的接口，如上例中的Serializable）放在边界列表的末尾。

**2、翻译泛型表达式**
```java
Pair<Employee> buddies = ...
Employee buddy = buddies.getFirst();
```
编译器把这个方法调用翻译为两条虚拟机指令：
- 对原始方法Pair.getFirst的调用，返回Object。
- 将返回的Object 类型强制转换为Employee类型。

当存取一个泛型域时也要插人强制类型转换。

**3、翻译泛型方法**
和上面Pair的翻译方式类似。

**4、桥方法**

```java
public class Employee<T> {
    public T salary;
    //getter、setter
}
```

```java
public class Manager extends Employee<Integer>{
    
    public void setSalary(Integer salary) {
        this.salary = 10*salary;
    }
    public Integer getSalary(){
        return this.salary;  
    }
}
```
按照虚拟机的翻译方式，Employee里的`setSalary(T salary)`会被翻译成`setSalary(Object salary)`，此时Manager继承Employee就有了两个setSalary方法：`setSalary(Integer salary)`和`setSalary(Object salary)`。

按下面的调用方式：
```java
Manager manager = new Manager();
Employee<Integer> employee = manager;
employee.setSalary(11);
```
Employee的setSalary被翻译成`setSalary(Object salary)`，由于Manager也有`setSalary(Integer salary)`这个方法，所以按照多态，`employee.setSalary(11)`将会调用Employee的`setSalary(Object salary)`方法。

为了解决这个问题，编译器在Manager类中生成一个桥方法，如下：
```java
public void setSalary(Object salary){
    set((Integer)salary);
}
```
这样的话就会调用Manager的setSalary(Integer salary)方法了。

同样道理，在虚拟机里Manager也会有Integer getSalary()方法和Object getSalary()方法，在Java里我们不能这样编写，因为方法签名是由方法名+参数组成，但是在虚拟机里用参数类型和返回类型确定一个方法，所以虚拟机可以正确处理这个情况。

Java 泛型转换的事实：
- 虚拟机中没有泛型，只有普通的类和方法。 
- 所有的类型参数都用它们的限定类型替换。 
- 桥方法被合成来保持多态。 
- 为保持类型安全性，必要时插人强制类型转换。

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">约束性与局限性</font></center>
**1、不能用基本类型实例化类型参数**
不能用类型参数代替基本类型。因此，没有Pair<double>, 只有Pair<Double>。原因是类型擦除之后， Pair类含有Object类型的域， 而Object不能存储double。

**2、运行时类型查询只适用于原始类型**
```java
if (a instanceof Pair<String>) // Error
if (a instanceof Pair<T>) // Error
if (a instanceof Pair) //true
```

值得注意的点：
```java
Pair pair = new Pair<Employee>();
pair.setData("aa");
System.out.println(pair.getData().getClass()); //class java.lang.String
```

```java
Pair<String> stringPair = . .
Pair<Employee〉employeePair = . .
if (stringPair.getClassO == employeePair.getClassO) // true
```
两次调用getClass都将返回Pair.class。


**3、不能创建参数化类型的数组**、
```
Pair<String>[] table = new Pair<String>[10]; // Error
```

不能创建参数化类型数组的原因:
- 假设这是合法的:
```java
Pair<String>[] table = new Pair<String>[10];
```
   擦除之后，table的类型是Pair[]。

- 那么可以把它转换为Object[]: 
```java
Object[] objarray = table；
```
- 数组会记住它的元素类型（Pair），如果试图存储其他类型的元素，就会抛出一个ArrayStoreException异常（但是编译可以通过）：
```java
objarray[0] = "Hello"; // Error component type is Pair
```

- 不过对于泛型类型， 擦除会使这种机制无效。以下赋值： 
```java
objarray[0] = new Pair<Employee>;
```
   能够通过数组存储检査，不过这是一个类型错误。所以不允许这样子创建。

> 注意：只是不允许创建这些数组， 而声明类型为`Pair<String>[]`的变量仍是合法的。只是不能用`new Pair<String>[10]`初始化这个变量。