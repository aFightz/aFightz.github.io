---
title: 03 | 事务隔离：为什么你改了我还看不见？
date: 2018-12-08 23:25:43
tags: [MySQL]
categories :
- 学习笔记
- MySQL
- MySQL实战45讲
---

事务的四大特性（ACID）
原子性（Atomicity）
一致性（Consistency）
隔离性（Isolation）
持久性（Durability）


多个事务可能会导致的问题：
脏读(dirty read)
不可重复读(non-repeatable read)
幻读(phantom read)



隔离级别
读未提交(read uncommitted)：一个事务还没提交时，它锁做的变更能被别的事务看到。
读提交(read committed)：一个事务提交后，它做的变更才能被其他事务看到。
可重复读(repeatable read)：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。且未提交变更对其他事务不可见。
串行化(serializable)：对同一行记录，读会加读锁，写会加写锁。

![](MySQL实战45讲（三）\demo.jpg)

读未提交(read uncommitted)：V1=2 , V2=2 , V3 = 2。
读提交(read committed)：V1=1 , V2=2 , V3 = 2。
可重复读(repeatable read)：V1=1 , V2=1 , V3 = 2。
串行化(serializable)：V1=1 , V2=1 , V3 = 2。


