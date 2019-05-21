---
title: 02 | 日志系统：一条SQL更新语句是如何执行的？
date: 2018-12-08 22:30:46
tags: [MySQL]
categories :
- 学习笔记
- MySQL
- MySQL实战45讲
---



> **概要**：主要介绍了redolog与Binlog的机制与区别

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">Redo Log</font></center>

**1、原理**
redo log是InnoDB引擎特有的日志，它使用了 <font color = "#CD5555">**WAL（Write-Ahead Logging）技术**</font>，即先写日志，再写磁盘。

**2、更新过程**
当有一条记录需要更新的时候，InnoDB 引擎就会先把记录写到 redo log 里面，并更新内存，这个时候更新就算完成了。同时，InnoDB 引擎会在适当的时候，将这个操作记录更新到磁盘里面。

**3、redo log的日志文件**
InnoDB 的 redo log 是固定大小的，比如可以配置为一组 4 个文件，每个文件的大小是 1GB：

![](MySQL实战45讲_02_日志系统：一条SQL更新语句是如何执行的？\redolog.jpg)

<font color = "#CD5555">**write pos**</font>是当前记录的位置，一边写一边后移，写到第3号文件末尾后就回到0号文件开头。<font color = "#CD5555">**checkpoint**</font> 是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。
write pos和checkpoint之间<font color = "#CD5555">**空着的部分**</font>，可以用来记录新的操作。如果write pos追上checkpoint，这时候不能再执行新的更新，得停下来先擦掉一些记录，把checkpoint推进一下。
有了redo log，InnoDB就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为<font color = "#CD5555">**crash-safe**</font>。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">BinLog</font></center>
binlog（归档日志）是属于Server层的日志，只能用于归档，没有crash-safe的能力。
**1、Binlog的两种模式**
- statement 格式：是记sql语句。
-  row格式：记录行的内容。

>
> **思考**：为什么Binlog不具备crash-safe的能力，该怎么从数据层面去理解。
>

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">两种日志的不同点</font></center>

- redo log是InnoDB引擎特有的，而binlog是MySQL的Server层实现的，所有引擎都可以使用。
- redo log是<font color = "#CD5555">**物理日志**</font>，记录的是<font color = "#ADADAD">“在某个数据页上做了什么修改”</font>，binlog 是<font color = "#CD5555">**逻辑日志**</font>，记录的是这个语句的原始逻辑，比如<font color = "#ADADAD">“给ID=2这一行的c字段加1”</font>。
- redo log是循环写的，空间color =固定会用完。binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。



#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">Update时的执行流程</font></center>

![](MySQL实战45讲_02_日志系统：一条SQL更新语句是如何执行的？\update语句执行流程图.png)

- 执行器先找引擎取 ID=2 这一行。ID 是主键，引擎直接用树搜索找到这一行。如果 ID=2 这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。
- 执行器拿到引擎给的行数据，把这个值加上 1，比如原来是 N，现在就是 N+1，得到新的一行数据，再调用引擎接口写入这行新数据。
- 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务。
- 执行器生成这个操作的 binlog，并把 binlog 写入磁盘。
- 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成。



#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">两阶段提交</font></center>

将 redo log 的写入拆成了两个步骤：`prepare` 和 `commit`，这就是"两阶段提交"。

两阶段提交是为了让redo log与binlog的数据状态<font color = "#CD5555">**保持一致**</font>。



#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">参数设置</font></center>

`innodb_flush_log_at_trx_commit`： 这个参数设置成 1 的时候，表示每次事务的 redo log 都直接持久化到磁盘。（建议设置为1）
`sync_binlog` ：这个参数设置成 1 的时候，表示每次事务的 binlog 都持久化到磁盘。（建议设置为1）

