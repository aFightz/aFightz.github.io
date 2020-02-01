---
title: 15 | 原子变量与非阻塞同步机制
date: 2020-02-01 21:19:23
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

CAS的含义
我认为V的值应该为A，如果是，那么将V的值更新为B，否则不修改并告诉V的值实际为多少。

volatile
如果不加volatile，那么会写入变量更新到主内存吗？更新的规则会是什么？

A线程
A段逻辑
写入volatile V

B线程
读取volatile V
B段逻辑

假设时间上的执行顺序是A->B，那么A段逻辑对于B段逻辑是可见的吗？ 

DCL标准实现
```java
public class DoubleCheckLocking{
    private static volatile Resource resource;
    
    public static Resource getInstance(){
        if(resource == null){
            synchronized (DoubleCheckLocking.class){
                if(resource == null){
                    resource = new Resource();
                }
            }
        }
    }
}  
```    

初始化过程中，对于可以通过某个final城到达的任意变量（例如某个final数组中的元素，或者由一个final域引用的HashMap的内客）将同样对于其他线程是可见（即不会被重排序）的。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              