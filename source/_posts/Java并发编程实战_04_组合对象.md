---
title: 04 | 对象的共享
date: 2019-04-03 17:49:12
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

可以使用装饰器方法保护非线程安全的类。
e.g 使用Collections.synchronizedList保护ArrayList。

私有锁与对象锁
对象锁会暴露给外部，而私有锁不会。

对Collections.unmodifiableMap的一些浅显探讨
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
由上面的输出可知，unmodifiableMap的value执行的是深层复制，而且可以对value对象里面的属性进行更改。但是不能对引用进行更改。








