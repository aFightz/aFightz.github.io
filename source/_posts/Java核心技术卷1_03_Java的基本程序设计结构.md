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
- 二进制有一个`前缀0b`（从java7开始，如0b1001就是9）。

从java7开始还可以为数字字面量加下划线（如1_000_000）方便阅读，java编译器会去除这些下划线。
Java没有任何无符号类型(unsigned)。

**3、浮点类型**
浮点类型用于表示有小数部分的数值。在Java中有两种浮点类型。如下表:

|  类型  | 存储需求(单位：byte) |
| :----: | :------------------: |
| float  |          4           |
| double |          8           |

double的数值精度是float类型的两倍，所以也被称为双精度数值。
float类型的数值有一个`后缀F`（例如3.14F）。没有后缀F的浮点数值（如3.14）<font color = "#CD5555">默认为double</font>类型。(`3.14D`也表示double类型)。
jdk5.0中，可以用<font color = "#CD5555">十六进制表示浮点数值</font>。例如，0.125可以表示成`0x1.0p-3`，在十六进制表示法中，使用p表示指数而不是e。尾数采用十六进制，指数采用十进制。且指数的基数是2（p是2）而不是10。
所有的浮点数值计算都遵循<font color = "#CD5555">IEEE754规范</font>。

<font color = "#36648B">double在内存里的表现形式</font>
![](Java核心技术卷1_03_Java的基本程序设计结构\double在内存里的表现形式.jpeg)


<font color = "#36648B">float在内存里的表现形式</font>
![](Java核心技术卷1_03_Java的基本程序设计结构\float在内存里的表现形式.jpeg)
- 第一位s代表符号为，1代表负数，0代表正数。
- 第二个域是指数域。 单精度数的偏差值为<font color = "#CD5555">-127</font>，而双精度double类型的偏差值为<font color = "#CD5555">-1023</font>。
- 第三个域为尾数域。 尾数为1.xxxx…xxx的形式。

<font color = "#36648B">计算公式（双精度）</font>
![](Java核心技术卷1_03_Java的基本程序设计结构\计算公式.png)

<font color = "#36648B">Demo：将存储在内存的2进制转换为float类型</font>
![](Java核心技术卷1_03_Java的基本程序设计结构\demo题目.png)

- s：1 -> 表示负数。 
- e：10000010 -> 130 -> (130-127) = 3,也即尾数部分小数点向右移3位。 
- m：10110000000000000000000,尾数加上前面省略的1和小数点为：1.10110000000000000000000。
- 现在三个部分都已经算出来了，通过指数值调整小数点的位置，得到结果为：1101.10000000000000000000。 
- 转换成10进制,整数部分为：1×2^3+1×2^2+1×2^0 = 13; 小数部分为：1×2^-1 = 0.5。
- 最后整数部分加上小数部分，再加上符号，所表示的实数就是：−13.5。


<font color = "#36648B">用于表示溢出和出错情况的三个特殊的浮点数值</font>
- 正无穷大 -> `Double.POSITIVE_INFINITY`
- 负无穷大 -> `Double.NEGATIVE_INFINITY`
- NaN（不是一个数字）-> `Double.NaN`

所有非数值的数字都是<font color = "#CD5555">不相同的</font>，所以不能像下面这样检测一个特定值是都等于Double.NaN。
```java
if(x == Double.NaN)
```
但可以用以下代码判断是不是NaN:
```java
if(Double.isNaN(x))
```

<font color = "#36648B">浮点数值不适用于出现舍入误差的金融计算中（带小数点的计算）</font>
```java
System.out.println(2.0-1.1);
```
输出的将会是 0.8999999999999999。
其主要原因是浮点数值采用二进制系统表示，而在二进制系统中<font color = "#CD5555">无法精确的表示分数1/10</font>。

<font color = "#36648B">小数转换二进制的方法</font>
十进制数的整数位是二进制数的整数位，十进制数的小数位是二进制数的小数位。两部分分开转换。
- 整数部分。除以2取余，逆序排列。
- 小数部分。乘2取整，顺序排列。

<font color = "#36648B">Demo： 十进制的0.1转换为二进制</font>
```
 0.1 × 2 = 0.2 ......0
 0.2 × 2 = 0.4 ......0
 0.4 × 2 = 0.8 ......0
 0.8 × 2 = 1.6.......1
 0.6 × 2 = 1.2.......1
 0.2 × 2 = 0.4.......0
 ......无限循环
```

从上面过程可以知道，在将一个十进制的小数的小数部分转换为二进制的小数时，<font color = "#CD5555">除非最后的一位是5，否则是不可能乘尽的，</font>也就意味着这个十进制的小数是无法用二进制来精确表示的。
若需要在数值计算中不含有任何误差，就应该使用`BigDecimal`类。


<font color = "#36648B">问题集锦</font>
- double类型的计算中有时精确有时不精确的问题。
- 直接赋值小数给double类型，按道理来讲尾数不为5的应该都表示不了，但实际上可以表示。
- @see https://www.zhihu.com/question/56545018 这个问题里怎么知道一个double有2个近似值，从而取有效位比较小的那个近似值。


**4、char类型**
char类型用来表示单个字符，通常用来表示字符常量。
`'A'`是编码65对应的字符常量。`"A"`是一个包含字符A的字符串。
Unicode编码单元可以表示为<font color = "#CD5555">十六进制值</font>，范围从`\u0000`到`\uffff`。
char类型是<font color = "#CD5555">基于Unicode编码</font>的，所以占用两个字节。

用于表示特殊字符的转义序列符如下:

| 转义序列 |  名称  | Unicode值 |
| :------: | :----: | :-------: |
|   `\b`   |  退格  |  \u0008   |
|   `\t`   |  制表  |  \u0009   |
|   `\n`   |  换行  |  \u000a   |
|   `\r`   |  回车  |  \u000d   |
|   `\"`   | 双引号 |  \u0002   |
|   `\'`   | 单引号 |  \u0027   |
|   `\\`   | 反斜杠 |  \u005c   |

所有的转义字符都必须出现在字符常量或字符串的引号内，如`'\u2122'`或`'Hello\n'`。

**5、boolean 类型**
数值型和布尔值之间不能进行相互转换。


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">变量</font></center>
变量名必须是一个<font color = "#CD5555">以字母开头</font>的由字母或数字构成的序列。其中字母包括`'A'~'Z'、'a'~'z'、'_'、'$'`或在某种语言中代表字母的任何Unicode字符(实际上并不能用Unicode)，变量名的长度没有限制。
尽管$是一个合法的Java字符，但最好不要用。它只用在Java编译器或其他工具生成的名字中。


#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">常量</font></center>
关键字final表示这个变量只能被赋值一次，赋值以后就不可以再被更改了。

#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">运算符</font></center>
当参与`/`运算的两个操作数都是整数时，表示整数除法，否则表示浮点除法。
整数除以0将会产生一个异常，而浮点（不包括0.0）除以0将会得到无穷大结果。
计算0.0/0或者负数的平方根结果为`NaN`。

#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">位运算符</font></center>
**1、位运算符列表**

| 符号 |       名称       |
| :--: | :--------------: |
|  &   |        与        |
|  I   |        或        |
|  ^   |       异或       |
|  ~   |        非        |
|  <<  |   左位移运算符   |
|  >>  |   右位移运算符   |
| <<<  | 无符号右移运算符 |


运算符中，除~以外，其他均为二元运算符。操作数只能为整型和字符型数据。

**2、位运算符运算规则**

**与运算**

| 操作数1 | 操作数2 | 结果 |
| :-----: | :-----: | :--: |
|    0    |    0    |  0   |
|    0    |    1    |  0   |
|    1    |    0    |  0   |
|    1    |    1    |  1   |


**或运算**

| 操作数1 | 操作数2 | 结果 |
| :-----: | :-----: | :--: |
|    0    |    0    |  0   |
|    0    |    1    |  1   |
|    1    |    0    |  1   |
|    1    |    1    |  1   |

**非运算**

| 操作数 | 结果 |
| :----: | :--: |
|   0    |  1   |
|   1    |  0   |

**异或运算**

| 操作数1 | 操作数2 | 结果 |
| :-----: | :-----: | :--: |
|    0    |    0    |  0   |
|    0    |    1    |  1   |
|    1    |    0    |  1   |
|    1    |    1    |  0   |

**左位移**
符号位不变，低位补0。

**右位移**
符号位不变，并用符号位补溢出的高位。

**无符号右移**
符号位也跟着改变，高位补0。

没有无符号左移的运算符。
&与|运算符与&&和||非常相似，只是不按短路方式计算。

#### <center><font color = "#36648B">✎✎✎✎✎✎✎</font><br/><font color = "#36648B">数学函数</font></center>
在Math类中，为了达到最快的性能，所有方式都使用计算机浮点单元中的例程。如果想要结果完全正确，那么就应该使用`StrictMath`类。

**1、数值类型之间的合法转换**
![](Java核心技术卷1_03_Java的基本程序设计结构\数值转换.png)

5个实心箭头表示<font color = "#CD5555">无信息丢失</font>的转换。3个虚箭头表示<font color = "#CD5555">可能有精度损失</font>的转换。
当用上面两个数值进行二元操作时：
- 如果两个操作数有一个是double类型的，另一个操作数就会转换成double类型。
- 否则如果其中一个操作数是float类型，另一个操作数就会转换成float类型。
- 否则如果其中一个操作数是long类型，另一个操作数就会转换成long类型。
- 否则，两个操作数都将被转换成int类型。


**2、强制转换**
如果试图将一个数值从一种类型强制转换成另一种类型而又超出了目标类型的表示范围，结果就会被截断成一个完全不同的值。
> e.g (byte)300的结果为44。


#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">java字符串</font></center>
Java字符串就是Unicode的字符序列。
由于不能修改Java字符串中的字符，所以在Java文档中将String类兑现称为不可变字符串。
不可变字符串却有一个优点：编译器可以让<font color = "#CD5555">字符串共享</font>。

在Java的栈中，有共享池的概念，会把一些常量放到这个共享池中，包括字符串常量和基本类型常量。共享的操作在<font color = "#CD5555">编译时</font>由编译器完成的，可以节省内存，并提高效率。例如语句`String str = "hello"`，首先在栈中创建字符串引用变量str，再看看常量池中有没有"hello"，如果有str就直接指向它，没有就创建"hello"并放在常量池中，然后指向它。(对于int之类的基本类型的变量也差不多都是这样的。)
而对于`String str =  new String("hello")`，则是创建新的对象，并放在堆内存中。是在<font color = "#CD5555">runtime</font>的时候分配的。这样做效率和节省内存的方面不如`String str = "hello"`，但是更灵活。如果在编译时不知道要创建什么样的字符串，就只能运行时创建了。

**1、代码点与代码单元**
代码点 >= 代码单元。
Java字符串由char序列组成。char数据类型是一个采用UTF-16编码表示Unicode代码点的代码单元。
 大多数的常用Unicode字符使用一个代码单元就可以表示，而辅助字符需要一对代码单元表示。
String的length方法将返回采用UTF-16编码表示的代码单元数量。而想要得到实际的长度，即代码点数量。需要如下方法:
```java
int count = str.codePointCount(0,str.length());
```
调用`charAt(n)`方法返回的是位置n的代码单元。

**2、遍历代码点的两种方法**
```java
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
```

```java
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
```



**3、构建字符串**

<font color = "#36648B">StringBuilder与StringBuffer的区别</font>
- 这两个类的API是相同的。
- StringBuffer的效率比较低，但是是线程安全的。

Java8中，用String拼接字符串默认使用`StringBuilder`进行拼接。

可以使用标签的形式跳出循环，但是只能跳出语句不能跳进语句。
```java
label:
{
   if(condition) break label;
}
```


#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">数组</font></center>
创建一个数字数组时，所有元素都初始化为`0`。boolean数组的元素都会初始化为`false`。对象数组的元素则初始化为一个特殊值`null`。
一旦创建了数组，就不能再改变它的大小。

**1、打印数组中的所有值**
```java
Arrays.toString(a);
```

**2、数组初始化**

<font color = "#36648B">静态初始化</font>
```java
int[] smallPrimes = {2,3,5};
```

<font color = "#36648B">动态初始化</font>
```java
int[] smallPrimes = new int[2];
smallPrimes[0] = 1;
```


<font color = "#36648B">匿名数组</font>
```java
new int[]{2,3,5} ;
```
Java中数组长度可以为0。
数组长度为0与null不同。


**3、数组拷贝**
在Java中，允许将一个数组变量拷贝给另一个数组变量。这时，两个变量将引用同一个数组。
```java
int[] luckyNumbers = smallPrimes;
luckyNumbers[5] = 12 //now smallPrimes[5] is also 12
```

如果只是希望拷贝数组的值而不是变量，可以用`Arrays`类的`copyOf`方法。

**4、数组排序**
```java
Arrays.sort(a);
```
这个方法使用了优化的<font color = "#CD5555">快速排序</font>算法。

**5、二维数组**
```java
int[][] magicSquare =
   {
       {1,2,3},
       {2,3,4}
   };
```
快速打印一个二维数组可以用`Arrays.deepToString(a)`方法。

**6、不规则数组**
```java
int[][] odds = new int[NMAX + 1][];
for(int n = 0;n<=NMAX; n++)
    odds[n] = new int[n + 1];
```






