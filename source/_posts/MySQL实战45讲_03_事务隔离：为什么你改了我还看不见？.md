---
title: 03 | 事务隔离：为什么你改了我还看不见？
date: 2018-12-08 23:25:43
tags: [MySQL]
categories :
- 学习笔记
- MySQL
- MySQL实战45讲
---

> **概要**：主要介绍了四种隔离级别。

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">事务</font></center>
**1、事务的四大特性（ACID）**
- 原子性（Atomicity）
- 一致性（Consistency）
- 隔离性（Isolation）
- 持久性（Durability）


**2、多个事务可能会导致的问题：**
- 脏读(dirty read)。事务A读到了事务B未提交的数据。
- 不可重复读(non-repeatable read)。事务A第一次查询得到一行记录row1，事务B提交修改后，事务A第二次查询得到row1，但列内容发生了变化。
- 幻读(phantom read)。事务A第一次查询得到一行记录row1，事务B增加一条记录row2并提交修改后，事务A第二次查询得到两行记录row1和row2。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">隔离级别</font></center>
**1、隔离级别的分类**
- 读未提交(read uncommitted)：一个事务还没提交时，它锁做的变更能被别的事务看到。
- 读提交(read committed)：一个事务提交后，它做的变更才能被其他事务看到。
- 可重复读(repeatable read)：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。且未提交变更对其他事务不可见。
- 串行化(serializable)：对同一行记录，会加读写锁。

**2、隔离级别的区别**

![](MySQL实战45讲_03_事务隔离：为什么你改了我还看不见？\demo.jpg)

|          |  v1  |  v2  |  v3  |
| :------: | :--: | :--: | :--: |
| 读未提交 |  2   |  2   |  2   |
|  读提交  |  1   |  2   |  2   |
| 可重复读 |  1   |  1   |  2   |
|  串行化  |  1   |  1   |  2   |

<font color = "#ADADAD">这样来看串行化更像是读写都是同一把锁，但是后面作者在评论上又说读读不互斥。</font>

**3、隔离级别的实现**
- 读未提交：直接返回记录上的最新值。（这个最新值从InnoDB buffer pool读取）
- 读提交：每个SQL语句开始执行时创建一个视图。
- 可重复读：事务启动时创建一个视图，整个事务存在期间都使用此视图。
- 串行化：加锁的方式。

**4、数据库默认的隔离级别**
Oracle：读提交(read committed)。
Mysql：可重复读(repeatable read)。

**5、设置Mysql的隔离级别**
- 全局修改。修改mysql.ini配置文件：
  ```sql
  #可选参数有:READ-UNCOMMITTED, READ-COMMITTED, REPEATABLE-READ, SERIALIZABLE.
  transaction-isolation = REPEATABLE-READ
  ```
- 当前session修改。
  ```sql
  set session transaction isolation level read uncommitted;
  ```


**6、查看全局与当前session隔离级别**
```sql
SELECT @@global.tx_isolation; 
SELECT @@session.tx_isolation; 
```




**7、可重复读隔离级别的实现**
![](MySQL实战45讲_03_事务隔离：为什么你改了我还看不见？\可重复的读的实现.jpg)
每个事务在启动时都会创建一个视图，每条记录在更新时都会记录一条回滚操作，事务A将1改成2,事务B将2改成3，事务C将3改成4。此时为了保证事务前后读取都是一致的，就得保存回滚段。所以如果使用长事务，回滚段就会变得非常长。
当事务A提交或回滚后，回滚段会被相应删除一部分。
所以尽量别使用长事务，否则会吃内存。因为长事务会在内存保存许多回滚记录。



**8、事务提交方式**
- 显示启动事务语句。begin或start transaction，配套的提交语句时commit，回滚语句时rollback。(事务并不是在beain就启动的，而是在之后的第一个DML语句才启动。)
- set autocommit=0。意味着在只执行一个select语句的时候，这个事务就启动了，且不会自动提交，直到主动执行commit或rollback语句，或者断开连接。（自动提交的意思是“在没有begin或start transaction 的时候”，自动提交。）

**`commit work and chain`<font color = "#CD5555">语句可以在提交commit后，立刻执行begin,减少客户端与Mysql的交互次数</font>。**

**9、查询长事务**
```sql
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```


**疑问：**
- rr隔离如何利用gap锁防止幻读。
- rr隔离与串行化隔离的优劣。
- rr隔离、mvcc、快照、视图这几个之间的关系。

可重复读实现原理
https://www.jianshu.com/p/17967b72139a





