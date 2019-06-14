---
title: 03 | Java的基本程序设计结构
date: 2019-06-14 16:26:40
tags: [Spring Cloud]
categories :
- 学习笔记
- Java
- Java核心技术卷1
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">规范</font></center>
**1、类名的规范**
名字必须以<font color = "#CD5555">字母开头</font>，后面可以跟<font color = "#CD5555">字母和数字的任意组合</font>，长度基本上没有限制。(不能用java保留字)。
    
**2、main方法**
main方法必须声明为<font color = "#CD5555">public</font>。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">数据类型</font></center>
在Java中，一共有<font color = "#CD5555">8种</font>基本类型，其中<font color = "#CD5555">4种</font>整型、<font color = "#CD5555">2种</font>浮点类型、<font color = "#CD5555">1种</font>字符类型char、<font color = "#CD5555">1种</font>boolean类型。

**1、整型**
整型用于表示没有小数部分的数值，它允许是负数。在Java中，整型的范围与与进行Java代码的机器无关。具体内容如下:

| 类型  | 存储需求(单位：byte) |
| :---: | :------------------: |
| byte  |          1           |
| short |          2           |
|  int  |          4           |
| long  |          8           |


**2、长整型数值**
- 长整型数值有一个`后缀L`(如40000000000L)。
- 十六进制数值有一个`前缀0x`（如0xCAFE）。
- 八进制有一个`前缀0`，(如010对应八进制中的8)，但是很容易混淆，所以<font color = "#CD5555">最好不要用</font>。
- 二进制有一个`前缀0b`（从java7开始），如（0b1001就是9）。

从java7开始还可以为数字字面量加下划线（如1_000_000）方便阅读，java编译器会去除这些下划线。
Java没有任何无符号类型(unsigned)。

**3、浮点类型**
浮点类型用于表示有小数部分的数值。在Java中有两种浮点类型。如下表:

|  类型  | 存储需求(单位：byte) |
| :----: | :------------------: |
| float  |          4           |
| double |          8           |

double的数值精度是float类型的两倍,所以也被称为双精度数值）
float类型的数值有一个`后缀F`（例如3.14F）。没有后缀F的浮点数值（如3.14）<font color = "#CD5555">默认为double</font>类型。(`3.14D`也表示double类型)。
jdk5.0中，可以用<font color = "#CD5555">十六进制表示浮点数值</font>。例如，0.125可以表示成`0x1.0p-3`，在十六进制表示法中，使用p表示指数而不是e。尾数采用十六进制，指数采用十进制。且指数的基数是2（p是2）而不是10。
所有的浮点数值计算都遵循<font color = "#CD5555">IEEE754规范</font>。

**1、double在内存里的表现形式**
![](Java核心技术卷1_03_Java的基本程序设计结构\double在内存里的表现形式.jpeg)


**2、float在内存里的表现形**
①第一位s代表符号为，1代表负数，0代表正数。
② 第二个域是指数域。 单精度数的偏差 值为 -127，而双精度double类型的偏差值为 -1023。
③ 第三个域为尾数域。 尾数为1.xxxx…xxx的形式。

计算公式（双精度）


Demo:将存储在内存的2进制转换为float类型。


s: 1-表示负数 
e: 10000010->130->(130-127)=3,也即尾数部分小数点向右移3位。 
m: 10110000000000000000000,尾数加上前面省略的1和小数点为：1.10110000000000000000000.
现在三个部分都已经算出来了，通过指数值调整小数点的位置，得到结果为： 
1101.10000000000000000000 
转换成10进制,整数部分为:1×2^3+1×2^2+1×2^0=13; 小数部分为：1×2^-1=0.5. 
最后整数部分加上小数部分，再加上符号，所表示的实数就是：−13.5。


下面是用于表示溢出和出错情况的三个特殊的浮点数值。
①正无穷大                      Double.POSITIVE_INFINITY
②负无穷大                      Double.NEGATIVE_INFINITY
③NaN（不是一个数字） Double.NaN

所有非数值的数字都是不相同的，所以不能像下面这样检测一个特定值是都等于Double.NaN。
if(x == Double.NaN)
但可以用以下代码判断是都是NaN:
if(Double.isNaN(x))

浮点数值不适用于出现舍入误差的金融计算中。（带小数点的计算）

e.g System.out.println(2.0-1.1);
输出的将会是
0.8999999999999999

其主要原因是浮点数值采用二进制系统表示，而在二进制系统中无法精确的表示分数1/10。

小数转换二进制的方法:
十进制数的整数位是二进制数的整数位，十进制数的小数位是二进制数的小数位。两部分分开转换。
①整数部分 除以2取余，逆序排列。
②小数部分 乘 2 取整，顺序排列。

e.g  十进制的0.1转换为二进制
 0.1×2=0.2 .....................0
 0.2×2=0.4 .....................0
 0.4×2=0.8 .....................0
 0.8×2=1.6......................1
 0.6×2=1.2......................1
 0.2×2=0.4......................0
......无限循环。

从上面过程可以知道，在将一个十进制的小数的小数部分转换为二进制的小数时，除非最后的一位是5，否则是不可能乘尽的，也就意味着这个十进制的小数是无法用二进制来精确表示的。

若需要在数值计算中不含有任何误差，就应该使用BigDecimal类。



问题集锦
①double类型的计算中有时精确有时不精确的问题。
②直接赋值小数给double类型，按道理来讲尾数不为5的应该都表示不了，但实际上可以表示。
③https://www.zhihu.com/question/56545018
这个问题里怎么知道一个double有2个近似值，从而取有效位比较小的那个近似值。


char类型    
char类型用来表示单个字符，通常用来表示字符常量。
'A'是编码65对应的字符常量。"A"是一个包含字符A的字符串。
Unicode编码单元可以表示为十六进制值，范围从\u0000到\uffff。
char类型是基于Unicode编码的，所以占用两个字节。

用于表示特殊字符的转义序列符如下:

所有的转义字符都必须出现在字符常量或字符串的引号内，如'\u2122'或'Hello\n'。

boolean 类型
数值型和布尔值之间不能进行相互转换。


变量
变量名必须是一个以字母开头的由字母或数字构成的序列。其中字母包括'A'~'Z'、''a'~'z'、'_'、'$'或在某种语言中代表字母的任何Unicode字符(实际上并不能用Unicode)。变量名的长度没有限制。

尽管$是一个合法的Java字符，但最好不要用。它只用在Java编译器或其他工具生成的名字中。


常量
关键字final表示这个变量只能被赋值一次，赋值以后就不可以再被更改了。

运算符
Java中。当参与/运算的两个操作数都是整数时，表示整数除法；否则表示浮点除法。
整数除以0将会产生一个异常，而浮点（不包括0.0）除以0将会得到无穷大结果。
计算0.0/0或者负数的平方根结果为NaN。

位运算符
位运算符包括:
①&      与
②|        或
③^      异或
④~       非
⑤<<    左位移运算符
⑥>>    右位移运算符
⑦<<<  无符号右移运算符

运算符在位模式下工作。 
运算符中，除~以外，其他均为二元运算符。操作数只能为整型和字符型数据。

位运算符运算规则
与运算


或运算


非运算


异或运算


左位移
符号位不变，低位补0。

右位移
符号位不变，并用符号位补溢出的高位。

无符号右移
符号位也跟着改变，高位补0。

没有无符号左移的运算符。

&与|运算符与&&和||非常相似，只是不按短路方式计算。

数学函数
在Math类中，为了达到最快的性能，所有方式都使用计算机浮点单元中的例程。如果想要结果完全正确，那么就应该使用StrictMath类。

数值类型之间的合法转换

5个实心箭头表示无信息丢失的转换。3个虚箭头表示可能有精度损失的转换。
当用上面两个数值进行二元操作时：
①如果两个操作数有一个是double类型的，另一个操作数就会转换成double类型。
②否则如果其中一个操作数是float类型，另一个操作数就会转换成float类型。
③否则如果其中一个操作数是long类型，另一个操作数就会转换成long类型。
④否则，两个操作数都将被转换成int类型。


强制转换
如果试图将一个数值从一种类型强制转换成另一种类型而又超出了目标类型的表示范围，结果就会被截断成一个完全不同的值。
e.g (byte)300的结果为44。


java字符串
Java字符串就是Unicode的字符序列。

由于不能修改Java字符串中的字符，所以在Java文档中将String类兑现称为不可变字符串。
不可变字符串却有一个优点：编译器可以让字符串共享。

在Java的栈中，有共享池的概念，会把一些常量放到这个共享池中，包括字符串常量和基本类型常量。共享的操作在编译时由编译器完成的，可以节省内存，并提高效率。例如语句String str = "hello"，首先在栈中创建字符串引用变量str，再看看常量池中有没有"hello"，如果有str就直接指向它，没有就创建"hello"并放在常量池中，然后指向它。
对于int之类的基本类型的变量也差不多都是这样的。
而对于String str =  new String("hello")，则是创建新的对象，并放在堆内存中。是在runtime的时候分配的。这样做效率和节省内存的方面不如String str = "hello"，但是更灵活。如果在编译时不知道要创建什么样的字符串，就只能运行时创建了。

代码点与代码单元
代码点>=代码单元。
Java字符串由char序列组成。char数据类型是一个采用UTF-16编码表示Unicode代码点的代码单元。
 大多数的常用Unicode字符使用一个代码单元就可以表示，而辅助字符需要一对代码单元表示。
String的length方法将返回采用UTF-16编码表示的代码单元数量。而想要得到实际的长度，即代码点数量。需要如下方法:
int count = str.codePointCount(0,str.length());
调用charAt(n)方法返回的是位置n的代码单元。

遍历代码点的两种方法
①
String str = "\uD835\uDD6B"+"a";
//代码点数量
int count = str.codePointCount(0,str.length());
System.out.println("code length :"+count);  //2

int i = 0;
while(i<count){
    //得到第i个代码点
    int index = str.offsetByCodePoints(0,i);
    int cp = str.codePointAt(index);
    System.out.println(Character.toChars(cp));  //𝕫 、a
    i++;
}

②
//第一个代码点的index
int index = str.offsetByCodePoints(0,0);
while(index<str.length()){
    int cp = str.codePointAt(index);
    System.out.println(Character.toChars(cp));   //𝕫 、a
    if(Character.isSupplementaryCodePoint(cp)){
        index+=2;
    }else{
        index++;
    }
}



构建字符串
字符串连接时若用String的话每次连接一个字符串都会构建一个新的String对象（Jdk1.8之后的连接符'+'会使用StringBuilder ，所以可以直接拼接不再需要额外构建StringBuilder来拼接），效率会比较低下。可以采用StringBuilder类避免这个问题。
e.g
StringBuilder builder = new StringBulider();
builder.append(ch);
builder.append(str);
String completedString = builder.toString();

StringBuilder与StringBuffer的区别
①这两个类的API是相同的。
②StringBuffer的效率比较低，但是是线程安全的。

Java8中，用String拼接字符串默认使用StringBuilder进行拼接。

可以使用标签的形式跳出循环，但是只能跳出语句不能跳进语句。
e.g
label:
{
   if(condition) break label;
}


大数值
BigDecimal实现了任意精度的浮点数运算。BigInteger实现了任意精度的整数运算。


数组
创建一个数字数组时，所有元素都初始化为0。boolean数组的元素都会初始化为false。对象数组的元素则初始化为一个特殊值null。
一旦创建了数组，就不能再改变它的大小。

打印数组中的所有值
e.g
Arrays.toString(a);
结果为[2,3,5]

数组初始化
①静态初始化 
int[] smallPrimes = {2,3,5};
②动态初始化
int[] smallPrimes = new int[2];
smallPrimes[0] = 1;

匿名数组
new int[]{2,3,5} ;
Java中数组长度可以为0。
数组长度为0与null不同。


数组拷贝
在Java中，允许将一个数组变量拷贝给另一个数组变量。这时，两个变量将引用同一个数组。
e.g
int[] luckyNumbers = smallPrimes;
luckyNumbers[5] = 12//now smallPrimes[5] is also 12

如果只是希望拷贝数组的值而不是变量，可以用Arrays类的copyOf方法。

数组排序
Arrays.sort(a)
这个方法使用了优化的快速排序算法。

二维数组
int[][] magicSquare =
   {
       {1,2,3},
       {2,3,4}
   };

快速打印一个二维数组可以用Arrays.deepToString(a)方法。

不规则数组
int[][] odds = new int[NMAX + 1][];
for(int n = 0;n<=NMAX; n++)
    odds[n] = new int[n + 1];






