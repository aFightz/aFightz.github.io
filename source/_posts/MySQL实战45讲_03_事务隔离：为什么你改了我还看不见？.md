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

![](MySQL实战45讲_03_事务隔离：为什么你改了我还看不见？\demo.jpg)

读未提交(read uncommitted)：V1=2 , V2=2 , V3 = 2。
读提交(read committed)：V1=1 , V2=2 , V3 = 2。
可重复读(repeatable read)：V1=1 , V2=1 , V3 = 2。
串行化(serializable)：V1=1 , V2=1 , V3 = 2。(这样来看串行化更像是读写都是同一把锁，但是后面作者在评论上又说读读不互斥)

隔离级别的实现
读未提交(read uncommitted)：直接返回记录上的最新值。（这个最新值从InnoDB buffer pool读取）
读提交(read committed)：每个SQL语句开始执行时创建一个视图。
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


可重复读隔离级别的实现
![](MySQL实战45讲_03_事务隔离：为什么你改了我还看不见？\可重复的读的实现.png)
每个事务在启动时都会创建一个视图，每条记录在更新时都会记录一条回滚操作，事务A将1改成2,事务B将2改成3，事务C将3改成4。此时为了保证事务前后读取都是一致的，就得保存回滚段。所以如果使用长事务，回滚段就会变得非常长。
当事务A提交或回滚后，回滚段会被相应删除一部分。
所以尽量别使用长事务，否则会吃内存。因为长事务会在内存保存许多回滚记录。



事务提交方式
显示启动事务语句。begin或start transaction，配套的提交语句时commit，回滚语句时rollback。(事务并不是在beain就启动的，而是在之后的第一个DML语句才启动。)
set autocommit=0。意味着在只执行一个select语句的时候，这个事务就启动了，且不会自动提交，直到主动执行commit或rollback语句，或者断开连接。（自动提交的意思是“在没有begin或start transaction 的时候”，自动提交。）

commit work and chain语句可以在提交commit后，立刻执行begin,减少客户端与Mysql的交互次数。

查询长事务
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60



