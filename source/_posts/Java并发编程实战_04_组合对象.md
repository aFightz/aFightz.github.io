---
title: 04 | 组合对象
date: 2019-04-03 17:49:12
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

<center> <h4><font color = "#36648B">✎</br>私有锁与对象锁</center>

*<font color = "#36648B">对象锁会暴露给外部，而私有锁不会。</font>*

**对Collections.unmodifiableMap的一些探讨**

```java
public class Test7 {

    public static void main(String[] args) {
        Map<Integer , Location> map = new HashMap<>();
        map.put(1 , new Location(1 ,1));

        Map<Integer , Location> otherMap = Collections.unmodifiableMap(map);
        System.out.println("map "+map.get(1));   //输出 map 1,1
        System.out.println("othermap "+otherMap.get(1)); //输出 othermap 1,1

        map.get(1).setX(11);
        System.out.println("othermap "+otherMap.get(1)); //输出 othermap 11,1

        otherMap.get(1).setX(12);
        System.out.println("othermap "+otherMap.get(1)); //输出 othermap 12,1

        otherMap.put(2 , new Location(1 ,1)); //抛出java.lang.UnsupportedOperationException

    }

    public static class Location{
        private int x;
        private int y;

        public Location(int x , int y){
            this.x = x;
            this.y = y;
        }

        public String toString(){
            return x+","+y;
        }
        //getter and setter ...
    }
}
```
由上面的输出可知，unmodifiableMap的value执行的是`深层复制`，而且<u>*可以对value对象里面的属性进行更改。但是不能对引用进行更改*</u>。

任何的线程问题都可以从这个角度出发：<font color = "red">**内存是否一致**</font>。

<center> <h4><font color = "#36648B">✎✎</br>由于锁不一样导致的问题</center>

```java
public class ListHelper<E> {
    public List<E> list = Collections.synchronizedList(new ArrayList<>());

    public synchronized boolean putIfAbsent(E x){
        boolean absent = !list.contains(x);
        if(absent){
            list.add(x);
        }
        return absent;
    }
}
```

synchronizedList这个方法提供了同步的方法，但是锁的对象是内置的一个obejct。
由于以下两点原因：
- list的get/set的锁与putIfAbsent的锁不是同一个锁。
- list是由public修饰的，所以可以直接被访问。

导致putIfAbsent其实是线程不安全的。

解决这个问题也很简单：
将list修改为private修饰，不让外界访问它的get/set方法，或者将它的get/set用ListHelper这个对象锁保护，保证与putIfAbsent使用的是同一个锁。（当然，这样就没必要用synchronizedList了）
以下Demo中，让putIfAbsent使用synchronizedList所使用的锁，（本质上也是保持使用锁的一致）：

```java
public class ListHelper<E> {
    public List<E> list = Collections.synchronizedList(new ArrayList<>());

    public boolean putIfAbsent(E x){
        synchronized (list){
            boolean absent = !list.contains(x);
            if(absent){
                list.add(x);
            }
            return absent;
        }
    }
}
```

synchronizedList的构造方法中，如果没有传锁对象，则会把本对象(list)，作为锁对象。
```java
SynchronizedCollection(Collection<E> c) {
    this.c = Objects.requireNonNull(c);
    mutex = this;
}

SynchronizedCollection(Collection<E> c, Object mutex) {
    this.c = Objects.requireNonNull(c);
    this.mutex = Objects.requireNonNull(mutex);
}
```

> 是否会存在这种情况：对象创建时引用已经赋值给变量了，但是对象方法还未被初始化？
答：如果不是final类型或volatile类型的对象初始化，确实会有这样的情况。

> ListHelper应该也是会有线程安全问题？

使用`装饰器方法`保护非线程安全的类：
> e.g 使用**Collections.synchronizedList**保护ArrayList。

