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
串行化(serializable)：V1=1 , V2=1 , V3 = 2。(这样来看串行化更像是读写都是同一把锁)

隔离级别的实现
读未提交(read uncommitted)：直接返回记录上的最新值。
读提交(read committed)：每个SQL语句开始执行时创建一个视图。（怎么保证事务提交后，才提交更改。）
可重复读(repeatable read)：事务启动时创建一个视图，整个事务存在期间都使用此视图。
串行化(serializable)：加锁的方式。

数据库默认的隔离级别
Oracle：读提交(read committed)。
Mysql：可重复读(repeatable read)。

设置Mysql的隔离级别
全局修改
修改mysql.ini配置文件
#可选参数有：READ-UNCOMMITTED, READ-COMMITTED, REPEATABLE-READ, SERIALIZABLE.
transaction-isolation = REPEATABLE-READ

当前session修改
set session transaction isolation level read uncommitted;

查看全局与当前session事务级别
SELECT @@global.tx_isolation; 
SELECT @@session.tx_isolation; 




