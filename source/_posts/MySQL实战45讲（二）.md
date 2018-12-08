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

### redo log

redo log  使用了 `WAL（Write-Ahead Logging）技术`，即先写日志，再写磁盘。

当有一条记录需要更新的时候，InnoDB 引擎就会先把记录写到 redo log 里面，并更新内存，这个时候更新就算完成了。同时，InnoDB 引擎会在适当的时候，将这个操作记录更新到磁盘里面。

InnoDB 的 redo log 是固定大小的，比如可以配置为一组 4 个文件，每个文件的大小是 1GB：

![](MySQL实战45讲（二）\redolog.jpg)

<font color = "red">write pos</font> 是当前记录的位置，一边写一边后移，写到第 3 号文件末尾后就回到 0 号文件开头。<font color = "red">checkpoint</font> 是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。

write pos 和 checkpoint 之间空着的部分，可以用来记录新的操作。如果 write pos 追上 checkpoint，这时候不能再执行新的更新，得停下来先擦掉一些记录，把 checkpoint 推进一下。

有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为<font color = "red">crash-safe</font>。

### binlog

redo log 是 InnoDB 引擎特有的日志，而 Server 层也有自己的日志，称为 binlog（归档日志）。

因为最开始 MySQL 里并没有 InnoDB 引擎。MySQL 自带的引擎是 MyISAM，但是 MyISAM 没有 crash-safe 的能力，binlog 日志只能用于归档。而 InnoDB 是另一个公司以插件形式引入 MySQL 的，既然只依靠 binlog 是没有 crash-safe 能力的，所以 InnoDB 使用另外一套日志系统——也就是 redo log 来实现 crash-safe 能力。



**两种日志的不同点：**

- redo log 是 InnoDB 引擎特有的，binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。

- redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。
- redo log 是循环写的，空间固定会用完。binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。



**update时的执行流程**

![](MySQL实战45讲（二）\update语句执行流程图.png)

- 执行器先找引擎取 ID=2 这一行。ID 是主键，引擎直接用树搜索找到这一行。如果 ID=2 这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。


- 执行器拿到引擎给的行数据，把这个值加上 1，比如原来是 N，现在就是 N+1，得到新的一行数据，再调用引擎接口写入这行新数据。

- 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务。

- 执行器生成这个操作的 binlog，并把 binlog 写入磁盘。

- 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成。



### 两阶段提交

将 redo log 的写入拆成了两个步骤：`prepare` 和 `commit`，这就是"两阶段提交"。

两阶段提交是为了让redo log与binlog的数据状态<font color = "red">保持一致</font>。



#### 参数设置

`innodb_flush_log_at_trx_commit`： 这个参数设置成 1 的时候，表示每次事务的 redo log 都直接持久化到磁盘。（建议设置为1）
`sync_binlog` ：这个参数设置成 1 的时候，表示每次事务的 binlog 都持久化到磁盘。（建议设置为1）





### redo log 与binlog记录的格式

Redo log不是记录数据页“更新之后的状态”，而是记录这个页 “做了什么改动”。
Binlog有两种模式：

- statement 格式：是记sql语句。
-  row格式：记录行的内容。

 

### binlog不能去掉的原因

- 一个原因是，redolog只有InnoDB有，别的引擎没有。
- 另一个原因是，redolog是循环写的，不持久保存，binlog的“归档”这个功能，redolog是不具备的。  