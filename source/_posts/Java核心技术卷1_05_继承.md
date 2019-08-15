---
title: 05 | 继承
date: 2019-07-09 16:13:20
tags: [Java基础]
categories :
- 学习笔记
- Java
- Java核心技术卷1
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">子类构造器</font></center>
如果子类的构造器没有显示地调用超类的构造器，则将自动地调用超类默认（没有参数）的构造器。如果超类没有默认的构造器，而且子类又没有显示地调用超类其他的构造器，那么会编译错误。
> 所以这就是有些类的构造方法中为什么一定要有super的原因

一个对象变量可以指示多种实际类型的现象被称为<font color = "#CD5555">多态</font>，在运行时能够自动地选择调用哪个方法的现象称为<font color = "#CD5555">动态绑定</font>。
- 子类数组的引用可以转换成超类数组的引用，而不需要采用强制类型的转换。
- 同样的子类对象的引用可以转换成超类对象的引用，而不需要采用强制类型的转换。

> 总的来说就是：
引用一定要“>=”对象。
赋值时，左边的引用要“>=”右边的引用，否则需要强转。（有可能强转失败）

Employee类（父类）
```java
public class Employee {

}
```
Manager类（子类）
```java
public class Manager extends Employee{
    public void say(){}
}
```
以下代码会导致`ArrayStoreException`：
```java
Manager[] managers = new Manager[10];
Employee[] staff = managers;
staff[0] = new Employee(); //出错在这一步
managers[0].say();
```

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">理解方法调用</font></center>
**1、方法调用过程**
假设要调用`x.f(args)`，x声明为类C的一个对象。下面是调用过程：
- 如果是private方法，static方法，final方法或者构造器，那么编译器可以直接知道在哪个类中调用方法（因为这些方法不能重写）。这种调用方式称为<font color = "#CD5555">静态绑定</font>。其他的都称为<font color = "#CD5555">动态绑定</font>。如果是动态绑定则继续按下个步骤走。
- 编译器查看对象的声明类型和方法名。编译器将会列举C类中名为f的方法和其超类中访问属性为public(protected与default不行？)且名为f的方法以及子类中名为f的方法。至此，编译器已经获得所有候选方法。
- 编译器查看调用方法时提供的参数类型。在候选方法中找到与之参数类型完全匹配的方法。否则将会报错。至此，编译器已经获得了需要调用的方法名以及参数类型。

- 当采用动态绑定调用方法时。假设x的实际类型是D，它是C类的子类。如果D类定义了f(args)，就直接调用；否则将在C类查找f(args)。

Demo：调用`e.getSalary()`的详细过程
- 由于`getSalary`不是`private`方法、`static`方法或`final`方法，所以将采用<font color = "#CD5555">动态绑定</font>。虚拟机为`Employee`和`Manager`两个类生成方法表（如果Employee有子类，也会为子类生成方法表）。在`Employee`的方法表中，列出了这个类定义的所有方法：
```
Employee：
   getName() -> Employee.getName()
   getSalay() - > Employee.getSalary()
   getHireDay() -> Employee.getHireDay();
   raiseSalary(double) -> Employee. raiseSalary(double)
```
  Manager方法表：
```
Manager：
   getName() -> Employee.getName()
   getSalay() - > Manager .getSalary()
   getHireDay() -> Employee.getHireDay();
   raiseSalary(double) -> Employee. raiseSalary(double)
   setBonus(double) -> Manager.setBonus(double)
```
  三个方法是继承Employee的，一个方式是重新定义的，一个方法是新增加的。
  > 方法表略去了Object的方法。

- 虚拟机搜索定义getSalary签名的类。此时虚拟机已经知道该调用哪个方法。
- 虚拟机调用方法。


**2、覆盖方法时的注意事项**
- 子类方法不能低于超类方法的可见性。
- 允许子类将覆盖方法的返回类型定义为原返回类型的子类型。如下：


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">阻止继承 ： final类与方法</font></center>

**1、final**
final修饰的类与方法不能被继承。

将一个类定义为final后，这个类的**全部方法都将变为final，但属性并不会变成final**。

早期的Java中，有些程序员为了避免动态绑定带来的系统开销而使用final关键字。

**2、内联**
如果一个方法**没有被覆盖**、被调用很频繁并且很短，编译器就能够对它进行优化，这个过程被称为内联。

```
e.g 如果getName()方法没有被覆盖，内联调用e.getName()将被替换为访问e.name域。
```
如果getName()在另外一个类中被覆盖，编译器无法知道被覆盖的代码将做什么操作，也就不能对它进行内联处理了。

当虚拟机加载了另外一个子类，而这个子类中包含了对内联方法的覆盖，那么优化器将取消对覆盖方法的内联。

在继承链上进行向下的转换会报`ClassCastException`错误，所以在强转前最好先事先用`instanceof`判断一下。
e.g.
```java
if(C instanceof D){//假设C是D的子类
    ...
}
```
**如果C为null，也不会产生异常**。

**3、equals**

重写equals Demo
```java
public class Employee {
    int flag;

    public boolean equals(Object otherObj){
        if(this == otherObj){
            return true;
        }
        if(otherObj == null){
            return false;
        }
        if(getClass() != otherObj.getClass()){
            return false;
        }
        Employee other = (Employee)otherObj;
        return this.flag == other.flag;
    }
}
```
`Objects.equals(a,b)`：可以避免a为空导致a.equals(b)出错的情况。
`static boolean equals(Object a ,Object b)`：如果a和b都为null，返回true；如果只有其中之一为null，则返回false；否则返回a.equals(b)。（这个方法在实际中还是挺好用的）

在子类定义equals方法时，首先调用超类的equals(前提是超类已经重写了equals方法)。如果检测失败，对象就不可能相等。

子类重写equals Demo
```java
public class Manager extends Employee{
    private int mFlag;

    public boolean equals(Object otherObj){
        if(!super.equals(otherObj)){
            return false;
        }
        Manager other = (Manager)otherObj;
        return mFlag == other.mFlag;
    }
}
```

Java语言规范要求equals方法具有下面的特性：
- 自反性：对于任何非空引用x，x.equals(x)应该返回true。
- 对称性。对于任何引用x和y，当且仅当y.equlas(x)返回true，x.equals(y)也应该发挥true。
- 传递性。对于任何引用x、y和z，如果x.equals(y)返回true，y.equals(z)返回true，x.equals(z)也应该返回true。
- 一致性。如果x和y引用的对象没有发生变化，反复调用x.equals(y)应该返回同样结果。
- 对于任意非空引用x，x.equals(null)应该返回false。

在上述的Employee类中，如果用

```java
if (!(otherObj instanceof Employee)) {
    return false;
}
```
代替
```java
if(getClass() != otherObj.getClass()){
    return false;
}
```
这样会违反对称性原则，原因如下：
假设e是Employee对象，而m是一个Manager对象，假设两个对象的属性都相同。当调用e.quals(m)时，会返回true。而调用m.equals(e)时，会出现ClassCastException错误，因为在检验过程中Employee无法转换成为Manager。

如果用getClass的方式检测的话，`e.quals(m)返回false，m.equals(e)返回false`，符合对称性原则，但是又会违反里氏置换原则。所以这是一个矛盾的点。


> **里氏置换原则**：所有引用基类的地方必须能够透明的使用其子类对象。也就是说，只要父类出现的地方子类就能够出现，而且替换为子类不会产生任何错误或异常。但是反过来，子类出现的地方，替换为父类就可能出现问题了。




Java中有一个`AbstractSet`类的equals方法检测两个集合是否有相同元素。AbstractSet类有两个具体的实现类：TreeSet和HashSet，它们分别使用不同的算法实现查找集合的操作。无论集合采用何种方式实现，都需要拥有对任意两个集合进行比较的功能。集合是一个相当特殊的例子，因为没有一个子类需要重新定义集合是否相等的语义，所以AbstractSet的equals方法应该被声明为final(实际上并没有)。所以有一个隐患就是如果AbstractSet的子类重写了equals方法，则会出现上述的矛盾问题。

所以可以总结出以下两条法则：
- 如果超类有自己的相等概念且本类也拥有自己的的相等概念（即“上头 ”有自己的相等概念），则对称性需求将强制采用`getClass`进行检测。
- 如果只由本类决定是否相等。（即“上头 ”没有自己的相等概念 ），那么就可以使用`instanceof`，并且将equals方法声明为final。这种情况不会违反任何原则。


对于数组类型的域，可以使用静态的Arrays.equals方法检测相应的数组元素是否相等。
`static Boolean equals(type[] a ,type[] b)`
如果两个数组成都相同，并且在对应的位置上数据元素也均相同，将返回true。数组的类型可以是Object、int、long、short、char、byte、boolean、float或double。（也可以是他们相应的包装器类）


#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">hashCode方法</font></center>
String类使用下列算法计算散列码:

```java
public int hashCode() {
    int h = hash; //hash默认为0
    if (h == 0 && value.length > 0) {
        char val[] = value;  //value就是String值，用char[]类型存储。
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```
> **为什么是31？** 
- 31*i = (i<<5)-i，jvm可以对这个乘法进行优化，用移位以及减法代替。
- 31不会太大也不会太小。

可以知道，String的散列值是**根据String值**算出来的。
由于hashCode方法定义在`Object`类中，因此**每个对象都有一个默认的散列码，其值为对象的存储地址**。

用以下例子可以验证上面的结论：
```java
String s = "OK";
String t = new String("OK");
StringBuffer sb = new StringBuffer(s);
StringBuffer tb = new StringBuffer(t);
System.out.println(s.hashCode() +" "+sb.hashCode());//2524 1163157884
System.out.println(t.hashCode() +" "+tb.hashCode());//2524 1956725890
```
如果要重新定义equals方法，就必须重新定义hashCode方法，以便用户可以将对象插入到**散列表**中。

hashCode方法应该返回一个整型数值（可以为负数）。

以下是Employee方法的hashCode方法Demo
```java
public class Employee {
    public int hashCode(){
        return 7 * name.hashCode() + 11 * number.hashCode() + 13 * new Double(salary).hashCode();
    }
}
```
不过上面的Demo有一个缺陷，就是name有可能为空，为此可以用以下方法代替:

`static int hashCode(Object a)`:
如果a为null就返回0，否则就返回a.hashCode()，如下：
```java
public class Employee {
    public int hashCode(){
        return 7 * Objects.hashCode(name ) + 11 * Objects.hashCode (number )  + 13 * Double.hashCode (salary);
    }
}
```
还有更好的做法，如下:
```java
public class Employee {
    public int hashCode(){
        return Objects.hash(name,salary,number);
    }
}
```

`static int hash(Object... objects)`:
返回一个散列码，由提供的所有对象的散列码组合而成。

Equals与hashCode的定义必须一致：如果x.equals(y)返回true，那么x.hashCode()就必须与y.hashCode()具有相同的值。 如果equals方法返回false，hashcode可以相等，但是这样不利于哈希表的性能。
hashcode相等的对象未必相等。因为hashcode是取余得出的。
哈希表判断对象是否相同的依据是equals与hashCode都相同。
如果存在数组类型的域，那么可以使用静态的`Arrays.hashCode`方法计算一个散列码，这个散列码由数组元素的散列码组成。

#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">toString方法</font></center>
Object类定义了`toString`方法，用来打印输出**对象所属的类名和散列码**。



#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">泛型数组列表</font></center>
如果已经清楚或能够估计出数组可能存储的元素数量，就可以在填充数组之前调用`ensureCapacity`方法：

```java
staff.ensureCapacity(100);
```
ArrayList如果没有一次性扩到想要的最大容量的话，它就会在添加元素的过程中，一点一点的进行扩容，而对数组扩容是要进行数组拷贝的，这就会浪费大量的时间。而ensureCapacity可以一次性扩容到指定的数量。如果指定的容量小于原来数组容量的1.5倍+1，那么扩容的容量将会是原来的1.5倍+1，否则数组容量将被扩容到指定的容量。
还可以把初始容量传递给ArrayList构造器：
```java
ArrayList<Employee> staff = new ArrayList<>(100);
```

一旦确认数组列表的大小不再发生变化，就可以调用`trimToSize`方法。这个方法将存储区域的大小调整为当前元素数量所需要的存储空间数目。垃圾回收器将回收多余的存储空间。

只有i小于或等于数组列表的大小时，才能够调用list.set(i,x)。以下这段代码是错误的：
```java
List<String> employeeList = new ArrayList<>();
employeeList.set(0,"1");
```
会报错：`IndexOutOfBoundsException`


#### <center><font color = "#36648B">✎✎✎✎✎✎✎</font><br/><font color = "#36648B">对象包装器与自动装箱</font></center>
对象包装器类拥有很明显的名字：**Integer、Long、Float、Double、Short、Byte、Character、Void和Boolean**（前6个类派生于公共的超类**Number**）。
对象包装类是**不可变**的，即一旦构造了包装器，就不允许更改包装在其中的值。同时，对象包装器类还是**final**，因此**不能定义它们的子类**。

自动装箱Demo：
```java
list.add(3);
```
将自动变换成
```java
list.add(Integer.valueof(3))
```


自动拆箱Demo:
```java
int n = list.get(i);
```
会被翻译器自动翻译成
```java
int n = list.get(i).intValue();
```

甚至在算数表达式中也能自动地装箱和拆箱：
```java
Integer n = 3;
n++;
```
编译器将自动插入一条对象拆箱的指令，然后进行自增计算，最后再将结果装箱。

#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">直接赋值与new的区别</font></center>
```java
String s1 = "a";
String s2 = "a";
String s3 = new String("a");
String s4 = new String("a");
```
上面的值中`s1==s2 , s3!=s4 , s1!=s3`。原因是:
- 直接赋值时，编译器会先去常量池（不是堆也不是栈）找是否存在`a`,没有的话在常量池创建值为`a`的对象，此时`s1`再引用这个对象，此时常量池已经存在了值为`a`的对象，所以s1与s2引用的也是同一个对象。
- 而用new操作时，会在堆中创建一个新的对象。

> **注**：==比较的是对象的内存地址,与hashcode没有任何一点关系。

**==陷阱**
当在以下值的范围内，采用直接赋值方式且value相等的包装箱类，使用“==”的情况:
- **Integer**。在`-128~127`范围内返回true。
编译器会把值-128~127的Integer类缓存,以后当要赋值在此范围的Integer时，直接去缓存里面找。

- **Short**。在`-128~127`范围内返回true。
类似Integer。

- **Byte**。永远都返回true。   
类似Integer，并且byte的范围是-128~127，所以肯定会返回true。

- **Long**。在`-128~127`范围内返回true。
类似Integer。

- **Character**。在`0~127`范围内返回true。
编译器会把值0~127的Character类缓存,以后当要赋值在此范围的Character时，直接去缓存里面找。

- **Boolean**。永远返回true。（用字符串形式赋值时，不区分大小写，非"true"的其他值都为"false"）
Boolean类里边定义了值为true和false的两个final类。不管Boolean的值是什么都只会引用这两个类。

- **Float**。永远返回false。
返回值是直接根据float值new出来的Float对象

- **Double**。永远返回false。
类似Float。

如果在一个条件表达式中混合使用Integer和Double类型，Integer就会拆箱，提升为double，再装箱为Double。

如果想编写可以改变一个可以修改数值参数（假设为int类型）的方法，不能用Integer，因为Integer类是不可改变的。可以用`IntHolder`。


#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">参数数量可变的方法</font></center>


public void test(Object...){
}
Object..参数类型与Object[]完全一样。
调用时像以下这样调用

test(Object1 , Object2 ...);


#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">枚举类</font></center>

public enum TestEnum {
    ONE,
    TWO
}
这个声明定义的类型是一个类，它刚好有2个实例，所以在比较两个枚举类型的值时，永远不需要调用equals，而直接使用“==”就可以了。

可以在枚举类型中添加一些构造器、方法和属性：

public enum TestEnum {
    ONE(1),
    TWO(2);
    private Integer value;
    TestEnum(Integer value){
        this.value = value;
    }
    public void setValue(Integer value) {
        this.value = value;
    }
}


static Enum valueOf(Class enumClass , String name)
返回指定名字、给定类的枚举常量


String toString()
返回枚举常量名


int ordinal()
返回枚举常量在enum声明中的位置，位置从0开始计数


int compareTo(E other)
如果枚举常量出现在other之前，则返回一个负值；如果this==other，则返回0；否则返回正值。枚举常量的出现次序在enum声明中给出。

#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">Class类</font></center>
可以根据存储在字符串中的类名创建一个对象：

String s  = "java.util.Random";
Object m = Class.forName(s).newInstance();


利用反射分析类的能力

java.lang.Class

public Field[] getFields()         //返回记录了这个类或其超类的公有域的Field对象数组。
public Field[] getDeclaredFields() //返回记录了这个类全部域的数组（不包含超类的）

//返回方法
public Method[] getMethods()                
public Method[] getDeclaredMethods()

//返回构造器
public Constructor<?>[] getConstructors()
public Constructor<?>[] getDeclaredConstructors()


java.lang.reflect.Field
java.lang.reflect.Method
java.lang.reflect.Constuctor

public Class<T> getDeclaringClass()   //返回一个描述类中定义的构造器、方法或域的Class对象。
public Class<?>[] getExceptionTypes() //返回一个用于描述方法抛出的异常类型的Class对象数组。（在Constructor和Method类中）
public int getModifiers() //返回一个用于描述构造器、方法或域的修饰符的整型数值。使用Modifier类中的方法可以分析这个返回值。
public String getName()   //返回一个用于描述构造器、方法或域名的字符串
public Class<?>[] getParameterTypes() //返回一个用于描述参数类型的Class对象数组。（在Constructor和Method类中）
public Class<?> getReturnType()  //返回一个用于描述返回类型的Class对象。（在Method类中）



java.lang.reflect.Modifier

public static String toString(int mod)    //返回对应modifiers中位设置的修饰符的字符串标识。
public static boolean isXXX (int mod)   //是否是此修饰符的判


在运行时使用反射分析对象

e.g

Employee harry = new Employee();
harry.setName("harry hacker");

Class cl = harry.getClass();
Field f = cl.getDeclaredField("name");
Object v = f.get(harry);//返回name field 的object
System.out.println(v);  //harry hacker
上面的代码会有问题，因为name属性是private的，所以不可以直接用get访问到，还得再加上一句。

f.setAccessible(true);
setAccessible方法时AccessibleObject类中的一个方法，它是Field、Method和Constructor类的公共超类。
上述代码的egt方法还有一个需要解决的问题。因为name是String类型，属于Object，所以调用get返回一个Object没什么问题，假设要访问一个类型为double(double是基本类型，不属于Object)，那么该怎么办呢？
可以调用getDouble方法（但是调用get方法也可以，只是返回的是Object，需要自己另外强转）。


java.lang.reflect.AccessibleObject

public void setAccessible(boolean flag) //为反射对象设置可访问标志。flag为true表面屏蔽Java语言的访问检查，使得对象的私有属性也可以被查询和是遏制
public boolean isAccessible()  //返回反射对象可访问标志的值
public static void setAccessible(AccessibleObject[] array, boolean flag) 



java.lang.Class

public Field getField(String name)  //返回指定名称的公有域
public Field[] getFields()
Field getDeclaredField(String name) //返回类中给定名称的域。
public Field[] getDeclaredFields()


java.lang.reflect.Field

public Object get(Object obj)  //返回obj对象中表示的域值
public void set(Object obj, Object value) //用newValue设置Obj对象中Field对象表示的域

使用反射编写泛型数组代码

动态扩建数组Demo

public static Object[] badCopyOf(Object[] a , int newLength){
    Object[] newArray = new Object[newLength];
    System.arraycopy(a,0,newArray,0,Math.min(a.length,newLength));
    return newArray;
}
这种转换有一个问题，一个对象数组不能转换为雇员数组（Employee[]）。Java数组会记住每个元素的类型，即创建数组时new表达式使用的元素类型。将一个Employee[]临时地转换成Object[]数组，然后再把它转换回来是可以的，但从一个开始就是Object[]的数组却永远不能转换成Employee[]数组。

以下的Demo可以适用于任何类型的数组扩建

public static Object goodCopyOf(Object a , int newLength){
    Class cl = a.getClass();
    if(!cl.isArray()){
        return null;
    }
    Class componentType = cl.getComponentType();
    int length = Array.getLength(a);
    Object newArray = Array.newInstance(componentType,newLength);
    System.arraycopy(a,0,newArray,0,Math.min(length,newLength));
    return newArray;
}



java.lang.reflect.Array

public static native Object get(Object array, int index) 
public static native xxx getXxxx(Object array, int index)  //xxx是8种基本类型种的一种，返回指定位置上的内容
public static native void set(Object array, int index, Object value)
public static native void seXxx(Object array, int index, Object value)
public static native int getLength(Object array)
public static Object newInstance(Class<?> componentType, int length)
public static Object newInstance(Class<?> componentType, int... dimensions) //返回一个具有指定类型，指定维数的新数组

调用任意方法
假设用m1代表Employee类的getName方法，下面这条语句显示了如何调用这个方法：

String n = (String)m1.invoke(harry);


下面说明了如何获得Employee类的getName方法和raiseSalary方法的方法指针。

Method m1 = Employee.class.getMethod("getName");
Method m2 = Employee.class.getMethod("raiseSalary",double.class);


使用反射获得方法指针的代码要比仅仅直接调用方法明显慢一下。










