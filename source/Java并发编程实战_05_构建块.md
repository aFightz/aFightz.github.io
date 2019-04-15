---
title: 05 | 构建块
date: 2019-04-15 19:26:35
tags: [并发]
categories :
- 学习笔记
- Java
- Java并发编程实战
---

Iterator与for-each在循环的过程中，会监听一个计数器，如果计数器被修改（集合被修改），则会抛出ConcurrentModificationException。
值得一提的是修改计数器并不是一个线程安全的操作，所以有可能线程A在迭代，线程B修改了集合，但是线程A没抛出则会抛出ConcurrentModificationException。


public class HiddenIterator {
    private final Set<Integer> set = new HashSet<>();

    public synchronized void add(Integer i){
        set.add(i);
    }

    public synchronized void remove(Integer i){
        set.remove(i);
    }

    public void addThenThings(){
        Random random = new Random();
        for(int i = 0 ; i < 10 ; i ++){
            add(random.nextInt());
        }
        System.out.println(set);
    }
}
上面的代码中，由于迭代（println会打印toString的内容，而hashSet的toString方法就是使用Iterator拼接Set的所有元素）与容器内元素的改变（HiddenIterator#get/set）没有使用同一个锁，所以在迭代时候有可能会出现ConcurrentModificationException。
解决的方法很简单，保持这两者的锁一致即可。


