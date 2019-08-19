---
title: 05 | 接口、lambda表达式与内部类
date: 2019-08-19 17:23:20
tags: [Java基础]
categories :
- 学习笔记
- Java
- Java核心技术卷1
---
#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">接口</font></center>
**1、接口概念**
接口中的所有方法**自动地属于public**。因此，在接口中声明方法时，不必提供关键字public。
在实现接口时，**必须**把方法声明为public。
接口中不能包含实例域，但却可以包含常量。与接口中的方法都自动地被设置为public一样，接口中的域将被自动设为**pubic static final**。
每个类只能够拥有**一个超类**，但却可以**实现多个接口**。
在Java SE 8中，允许在接口中增加**静态方法**。

**2、Comparable**
如果希望使用`Arrays类的sort方法`对A对象数组进行排序，那么A类就必须实现`Compareble`接口。

```java
//用这个对象与oj进行比较。如果对象小于o则返回负值，相等返回0，大于返回正值。
java.lang.Comparable<T> public int compareTo(T o);
```

```java
/**
* java.lang.Integer
*/
static int compare(int x , int y) //如果x<y返回一个负整数，相等返回0，大于返正整数。
```

```java
/**
* java.lang.Double
*/
static int compare(double x , double y) //如果x<y返回一个负数，相等返回0，大于返正数。
```

语言标准规定：对于任意的x和y，实现必须能够保证`sgn(x.compareTo(y)) = sgn(y.compareTo(x))`。（也就是说，如果y.compare(x)抛出一个异常，那么x.compareTo(y)也应该抛出一个异常。）这里的“sgn”是一个数值的符号：如果n是负值，sgn(n)等于-1；如果n是0，sgn(n) 等于0；如果n是正值，sgn(n) =1。
与equals方法一样，在继承的时候可能会出现问题，解决的方式也与equals一样。

可以对一个字符串数组排序，因为`String`类实现了`Comparable<String>`，而且`String.compareTo`方法可以按**字典顺序**比较字符换。

现在假设要按照长度对字符串进行排序，那么肯定不能让String类用两种不同的方式实现compareTo方法，而且String类也不应由我们来修改。要处理这种情况的时候，Arrays.sort方法还有第二个版本，如下：
```java
public class LengthComparator implements Comparator<String> {
    @Override
    public int compare(String first, String second) {
        return first.length() - second.length();
    }
}


public static void main(String [] args){
    String[] friends = {"1","111","11"};
    Arrays.sort(friends,new LengthComparator());
    System.out.println( Arrays.toString(friends));
}
```

**3、默认方法**
可以为接口方法提供一个默认实现。必须用**`defalut`**修饰符标记这样一个方法。

默认方法的一个重要用法是“接口演化”。假设A实现了B接口后，此时如果再为B接口添加一个非默认方法b的话，将不能保证“源代码兼容”，A类将不能成功编译。若将b声明为默认方法则不会有这个问题。

**解决默认方法的冲突**
如果先在一个接口将一个方法定义为默认方法，然后又在超类或另一个接口中定义了同样的方法，会发生什么情况呢？
- 超类优先。如果超类提供了一个具体方法，同名而且有相同参数类型的默认方法会被忽略。
- 接口冲突。如果一个接口 提供了一个默认方法，另一个接口提供了一个同名并且参数类型相同的方法(不论是否是默认方法)，必须覆盖这个方法来解决冲突。

**两个接口都是默认方法的解决方法Demo**
```java
public class SomeOne implements Worker,Employee{
    @Override
    public String getName() {
        return Worker.super.getName();
    }
}
```
千万不要让一个默认方法重新定义Object类中的某个方法。由于“类优先”原则，这样的方法绝对无法超越Object的方法（因为所有类都继承Object）。



**4、对象克隆**
如果希望copy是一个新对象，它的初始状态与original相同，但是之后他们各自会有自己不同的状态，这种情况下就可以使用clone方法。

**浅复制对象克隆Demo**
```java
public class CloneDemo implements Cloneable{

    private Date date = new Date();
    private boolean flag = false;

    //getter and setter...

    public CloneDemo clone() throws CloneNotSupportedException {
        CloneDemo copy = (CloneDemo)super.clone();
        return copy;
    }
}
```
由于Object的`clone()`方法是`protected`的，所以自己创建的对象想要实现克隆方法则**必须要重载**。**`Cloneable`**本身只是一个定义没有任何方法interface，实现它只是为了表明这个类可以被克隆，若不实现，逻辑上没有错误，但是在**运行时克隆会抛错**。

上述的只是**浅复制克隆**，值能对一些基本类型的复制，而并不能对对象进行复制。上面的例子中，如果原始对象对Date进行而更改了，克隆对象的Date也会随之改变。

**深度复制对象克隆Demo**
```java
public class CloneDemo implements Cloneable{

    private Date date = new Date();
    private boolean flag = false;

    //getter and setter...

    public CloneDemo clone() throws CloneNotSupportedException {
        CloneDemo copy = (CloneDemo)super.clone();
               copy.setDate((Date)this.getDate().clone());
        return copy;
    }
}
```
这个例子当原始对象的Date改变了，克隆对象的Date也不会改变，这称之为深度复制。
其实Object的clone只是new一个对象，然后把原始对象的基本类型set进去，而对象引用再没做额外操作的情况下，还是会引用原来的。
> 也就是说原始的Object的clone方法是**浅复制**。

所有数组类型都有一个public的clone方法。可以用这个方法建立一个新的数组，包含原数组所有元素的副本。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">lambda表达式</font></center>
**1、Demo**
```java
(String first , String second)
        ->first.length() - second.length();
```
如果代码要完成的计算无法放在一个表达式中，就可以像写方法一样，把这些代码放在{}中，如下：
```java
(String first , String second) -> {
    if(first.length() < second.length()){
        return -1;
    }else if(first.length() > second.length()){
        return 1;
    }else{
        return 0;
    }
}
```
即使lambda表达式没有参数，仍然要提供空括号，就像无参数方法一样：
```java
()->{
   //do something
}
```
如果可以推导一个lambda表达式的参数类型，则可以忽略其类型。如下：
```java
LambdaComp<String> comp = (first,second) ->
        first.length() - second.length();

public interface LambdaComp<T> {
    int compare(T a , T b);
}
```
如果方法只有一个参数，而且这个参数的类型可以推导得出，那么甚至还可以省略小括号：
```java
Comparable<String> comp = first ->
        first.length();
```
无需指定lambda表达式的返回类型。lambda表达式的返回类型总是会由上下文推导得出。
其实lambda表达式可以看做是对只有一个**抽象方法**的interface的实现。
如果设计你自己的接口，其中只有一个抽象方法，可以用@FunctionalInterface注解来标记这个接口。表明这个接口是一个函数式接口。当无意中增加了另一个非抽象方法时会编译错误。
在Java中，**lambda表达式就是闭包**。

**2、函数式接口**
对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个lambda表达式。这种接口称为**函数式接口**。

需要注意的是，一定是“**一个抽象方法**”。接口完全有可能重新声明Object类的方法，如toString或clone，这些声明可能会让方法不再是抽象的。而且，在JavaSE8中，接口可以声明非抽象方法。

**3、方法引用**

```java
LambdaComp<String> comp = System.out::println;
comp.test("aa");  //输出aa
```
相当于
```java
LambdaComp<String> comp = x-> System.out.println(x);
```


方法引用主要有3种情况：
- Object::instanceMethod
- Class::staticMethod
- Class::instanceMethod

前两种情况中，方法引用等价于提供方法参数的lambda表达式。  
**`System.out::println`**等价于**`x->System.out.println(x)`**。**`Math::pow`**等价于**`(x,y)->Math.pow(x,y)`**。
第三种情况，第一个参数会成为方法的目标。
**`String::compareToIgnoreCase`**等同于**`(x,y)->x.compareToIngnoreCase(y)`**。

可以在方法引用中使用this或者super参数。
>当一个Lambda表达式调用了一个已存在的方法,就可以用方法引用。

**4、构造器引用**
构造器引用与方法引用很类似，只不过方法名为**`new`**。例如，**`Person::new`**是Person构造器的一个引用。

可以用数组类型建立构造器引用。例如。**`int[]::new`**是一个构造器引用。它有一个参数：即数组的长度，等价于**`x->new int[x]`**。

**5、变量作用域**

不合法的Demo
```java
for(int i = 0;i<10;i++){
    ActionListener listener = event ->{
        System.out.println(i);
    };
}
```
但是这样又是合法的：
```java
for(int i = 0;i<10;i++){
    int finalI = i;
    ActionListener listener = event ->{
        System.out.println(finalI);
    };
}
```
    

**6、Comparator接口的静态方法**
Comparator接口包含很多方便的静态方法来创建比较器。

例如，假设有一个Person对象数组，可以如下（提取一个键）按名字对这些对象排序。
```java
Arrays.sort(person,Comparator.comparing(Person::getName));
```
可以把比较器与thenComparing方法串起来。
```java
Arrays.sort(person ,Comparator.comparing(Person::getLastName).thenComparing(Person::getFirstName));
```
也可以为键指定一个比较器，按照名字长度倒序排序。
```java
Arrays.sort(employees, comparing(Employee::geteFlag,(s,t)->t-s));
```


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">内部类</font></center>
- 内部类方法可以访问该类定义所在的作用域的数据，包括私有数据。
- 内部类可以对同一个包中的其他类隐藏起来。
- 当想要定义一个回调函数且不想编写大量代码时，使用匿名内部类比较便捷。

**1、使用内部类访问对象状态**
- 内部类既可以访问自身的数据域，也可以访问创建它的外围类对象的数据域。
- 内部类的对象总有一个隐式引用，它指向了创建它的外部类对象。这个引用在内部类的定义是不可见的。
- 外围类的引用要在构造器中设置。编译器修改了所有的内部类的构造器，添加一个外围类引用的参数。
- 只有内部类可以是私有类（private），而常规类只可以具有包可见性(default)，或公有可见性(public)。

**2、内部类的特殊语法规则**
- 外部类引用：**`OuterClass.this`**
- 创建内部类：**`outerObject.new InnerClass()`**
- 在外围类的作用域之外，可以这样引用内部类：**`OuterClass.InnerClass`**
- 非静态内部类不能有静态方法也不能有静态变量。

**3、内部类是否有用、必要和安全**
编译器会将内部类翻译成用$(美元符号)分隔外部类名与内部类名的常规类文件，而虚拟机则对此一无所知。

假设现在将Outer定义为常规类，将Inner定义为Outer的内部类，那么Inner在创建的时候，Outer的对象会传给Inner，但是即使是这样，Inner也访问不了Outer私有域，那么内部类是如何管理这些额外的访问特权呢？

用反射的方法去查看Outer类会发现：

class Outer{
   private int interval;
   private boolean beep;
   
   public Outer(int , boolean);
   static boolean access$0(Outer);
   public void start();
}

access$0将返回beep。（方法名可能稍有不同，如access$000，这取决于你的编译器。）



局部内部类

Demo

public class Outer {
    public void start(){
        class Inner implements ActionListener{

            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("hello world");
            }
        }

        ActionListener listener = new Inner();
        Timer t = new Timer(1,listener);
        t.start();
    }
}

局部内部类不能用public或private访问说明符进行声明。

局部类有一个优势，除了start方法之外，没有任何类知道Inner方法的存在。

由外部方法访问变量
与其他内部类相比，局部内部类还有一个优点。它们不仅能够访问包含它们的外部类，还可以访问局部变量。不过那些局部变量不可以被改变。（Java 8之前这些局部变量必须声明为final）。

Demo

public class Outer {
    public void start(int interval , boolean beep){
        class Inner implements ActionListener{
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("hello world");
                if(beep){
                    System.out.println("beep");
                }
            }
        }
        //beep = false;    //如果加上这句编译会错误
        ActionListener listener = new Inner();
        Timer t = new Timer(interval,listener);
        t.start();
    }
}
在创建Inner类时有保存interval与beep两个参数的局部副本。


匿名内部类

public void start(int interval , boolean beep){

    ActionListener listener = new ActionListener() {
        @Override
        public void actionPerformed(ActionEvent e) {
            System.out.println("hello world");
            if(beep){
                //do nothing
            }
        }
    };
    Timer t = new Timer(interval,listener);
    t.start();
}
如果构造参数的括号后面跟一个大括号，正在定义的就是匿名内部类。

双括号初始化

invite(new ArrayList<String>(){{add("Harry");add("Tony");}});

静态内部类
只有内部类可以声明为static

在内部类不需要访问外围类对象的时候，应该使用静态内部类。
与常规内部类不同，静态内部类可以有静态域和方法。
声明在接口中的内部类自动成为static和public类。
静态内部类可以直接new。