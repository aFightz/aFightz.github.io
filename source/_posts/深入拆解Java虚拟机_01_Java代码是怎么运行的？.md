---
title: 01 | Java代码是怎么运行的？
date: 2019-01-09 16:43:43
tags: [Java虚拟机]
categories :
- 学习笔记
- Java虚拟机
- 深入拆解Java虚拟机
---

JRE就是Java运行时环境。JRE包括了Java虚拟机以及Java核心类库。

Java虚拟机内存划分
![](深入拆解Java虚拟机_01_Java代码是怎么运行的？）\Java虚拟机的内存划分.png)
本地方法是用C++写的native方法。
PC寄存器存放各个线程的执行位置。

Java 虚拟机具体是怎样运行Java字节码的？
虚拟机视角：
执行Java代码首先需要将它编译而成的class文件加载到Java虚拟机中，加载后的Java类会被存放于方法区（Method Area）中。
在运行过程中，每当调用进入一个Java方法，Java虚拟机会在当前线程的Java方法栈中生成一个栈帧，用以存放局部变量以及字节码的操作数。这个栈帧的大小是提前计算好的，而且Java虚拟机不要求栈帧在内存空间里连续分布。当退出当前执行的方法时，不管是正常返回还是异常返回，Java虚拟机均会弹出当前线程的当前栈帧，并将之舍弃。
硬件视角：
HotSpot默认采用混合模式，综合了解释执行和即时编译两者的优点。它会先解释执行字节码，而后将其中反复执行的热点代码，以方法为单位进行即时编译。


使用asmtools.jar在查看在虚拟机中boolean类型的表示方式
public class Test3 {

    public static void main(String[] args) {
        boolean flag = true;
        if(flag) System.out.println("Hello , Java");
        if(flag == true) System.out.println("Hello ,Jvm");
    }
}
在idea中生成这段java代码的class文件Test3.class。

生成jasm文件 
java -jar asmtools.jar jdis Test3.class >>Test.jasm

super public class Test3
	version 49:0
{


public Method "<init>":"()V"
	stack 1 locals 1
{
		aload_0;
		invokespecial	Method java/lang/Object."<init>":"()V";
		return;
	
}

public static Method main:"([Ljava/lang/String;)V"
	stack 2 locals 2
{
		iconst_1;//改为iconst_2
		istore_1;
		iload_1;
		ifeq	L14;
		getstatic	Field java/lang/System.out:"Ljava/io/PrintStream;";
		ldc	String "Hello , Java";
		invokevirtual	Method java/io/PrintStream.println:"(Ljava/lang/String;)V";
	L14:	iload_1;
		iconst_1;
		if_icmpne	L27;
		getstatic	Field java/lang/System.out:"Ljava/io/PrintStream;";
		ldc	String "Hello ,Jvm";
		invokevirtual	Method java/io/PrintStream.println:"(Ljava/lang/String;)V";
	L27:	return;
	
}

} // end Class Test3
将iconst_1改为iconst_2，如上注释所示。

执行如下命令重新自动生成Test3.class
java -jar asmtools.jar jasm Test.jasm

在idea打开重新生成的Test3.class，可以很清楚的看到虚拟机是把boolean当做int来处理的。
public class Test3 {
    public Test3() {
    }

    public static void main(String[] var0) {
        byte var1 = 2;
        if (var1 != 0) {
            System.out.println("Hello , Java");
        }

        if (var1 == 1) {
            System.out.println("Hello ,Jvm");
        }

    }
}



