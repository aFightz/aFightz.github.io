---
title: 04 | 对象与类
date: 2019-07-09 15:01:16
tags: [Java基础]
categories :
- 学习笔记
- Java
- Java核心技术卷1
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">对象与对象变量</font></center>

下面代码里deadLine变量与birthDay变量引用的是同一个变量。
```java
Date deadLine = new Date();
Date birthDay = deadLine;
```

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">GregorianCalendar类</font></center>

```java
Calendar c = new GregorianCalendar(); //Calendar是抽象类
        
//1.直接设置年月日时分秒
//c.set(2015, Calendar.AUGUST, 2); //2015.08.02
        
//2.通过块分别设置相应的年月日时分秒
//注：可以按这种格式继续设置时分秒，如果省略，则按照本地默认设置
c.set(Calendar.YEAR, 2015); //2015年
c.set(Calendar.MONTH, 1); //2月，0为1月
c.set(Calendar.DAY_OF_MONTH, 2); //Calendar.DATE == Calendar.DAY_OF_MONTH
                
Date d = c.getTime();
System.out.println(d); //Mon Feb 02 21:15:13 CST 2015
        
//获取相应的年月日时分秒
System.out.println(c.get(Calendar.YEAR)); //2015
    
//测试日期计算
c.add(Calendar.YEAR, 10); //增加10年,减的话把10变成负的即可
System.out.println(c.getTime()); //Sun Feb 02 21:15:13 CST 2025
```


不要编写如下的返回引用对象的访问器方法，因为容易被外部修改。
```java
class Enployee{
    private Date hireDay;
    
    public Date getHireDay(){
        return hireDay;
    }
}
```
如果要返回一个对象的引用，应该首先对它进行克隆。如下：
```java
class Enployee{
    private Date hireDay = new Date();

    public Date getHireDay(){
        return hireDay==null?null:(Date)hireDay.clone();
    }
}
```
Date实现了clone方法。


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">基于类的访问权限</font></center>
一般来说，如果把属性设置为private后，外部在调用的时候是不可以直接通过“点”的方式调用的，但是以下两种例外。
```java
public class Employee {
    private String name;

    public boolean equals(Employee other){
        return name.equals(other.name);
    }
}
```
因为other本来就属于Employee，所以可以在这个类中可以直接调用。

```java
public class Demo {
    public static void main(String [] args) {
        Employee employee = new Employee();

        System.out.println(employee.name);
    }

    static class Employee {
        private String name;

        public boolean equals(Employee other){
            return name.equals(other.name);
        }
    }
}
```
因为Employee属于Demo的内部类，所以在Demo的方法里可以访问Employee的私有属性。



#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">final 实例域</font></center>
对象的final属性在构建对象时必须被初始化。


#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">静态常量</font></center>
`System.out`是一个静态常量，如下:

```java
public class System{
   public static final PrintStream out =  . . . ;
}
```
System类里面有有一个setOut方法可以将System.out设置为不同的流，因为setOut是一个本地方法（native），而不是用Java语言实现的。本地方法可以绕过Java语言的存储控制机制，从而对final属性进行修改。自己编写程序时，不应该这样处理。



#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">静态方法</font></center>
当对象里有静态方式时，建议使用类名，而不是通过构建的对象来调用静态方法。

可以通过调用方法对属性进行初始化（这种方式有点意思 ），如下:
```java
public class Employee {
    private static int nextId;
    private int id = assignId();

    private static int assignId(){
        int r = nextId;
        nextId++;
        return r;
    }
}
```
这样写就让assignId相当于一个初始化方法。
 

#### <center><font color = "#36648B">✎✎✎✎✎✎✎</font><br/><font color = "#36648B">调用另一个构造器</font></center>
如果构造器的第一个语句如this(...)，这个构造器将调用同一个类的另一个构造器,如下
```java
public class Employee {
    private int id ;
    public Employee(){
        this(1);
    }
    public Employee(int id){
        this.id = id;
    }
}
```


#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">对象析构与finalize方法</font></center>
可以为任何一个类添加finalize方法。finalize方法将在垃圾回收器清除对象之前调用。在实际应用中，不要依赖于使用这个方法回收任何短缺的资源，因为很难知道这个方法什么时候才能够调用。


#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">类的导入</font></center>
只能使用星号`*`导入一个包，而不能使用`Import java.*`或者`Import java.*.*`导入以java为前缀的所有包。


#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">包作用域</font></center>
- **private** :      只能被定义它们的类访问。
- **default** :      只能被同一包中的类访问。
- **protected** : 具有default的权限，并且可以被不同包中所继承的子类访问。
- **public** ：     可以被任意类访问。


#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">文档注释</font></center>
JDK包含一个很有用的工具，叫做javadoc，它可以由源文件生成一个HTML文档。

javadoc从下面几个特性中抽取信息：
- 包
- 公共类与接口
- 公有的和受保护的构造器及方法
- 公有的和受保护的域


#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">通用注释</font></center>
- `@author`:用在类文档的注释中，表示后作者姓名。可以用多个`@author`标记。

- `@version`:对当前版本的描述。

- `@since`:始于什么版本。

- `@deprecated`:对类、方法或变量不再使用的标识。

- `@see`:链接到某个方法或某个超链接。
```java 
//将链接到Test类的test(double)方法。
@see com.huangjunlong.Test#test(double)
```


- `@link`:同`@see`。














