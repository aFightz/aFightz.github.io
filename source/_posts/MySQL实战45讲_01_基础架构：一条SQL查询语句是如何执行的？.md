---
title: 01 | 基础架构：一条SQL查询语句是如何执行的？
date: 2018-12-03 00:15:46
tags: [MySQL]
categories :
- 学习笔记
- MySQL
- MySQL实战45讲
---



#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">MySQL的逻辑架构图</font></center>

![](MySQL实战45讲_01_基础架构：一条SQL查询语句是如何执行的？\MySQL的逻辑架构图.png)

MySQL 可以分为 `Server 层`和`存储引擎层`两部分。

存储引擎是插件式的，MySQL5.5.5版本之后默认的存储引擎是<font color = "#CD5555">**InnoDB**</font>。



#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">连接器</font></center>

用以下命令可以登录Mysql。

```mysql
mysql -h 127.0.0.1 -P 3306 -u root -p database_name
```
之后需要输入你的密码即可。

一个用户成功建立连接后，即使你用管理员账号对这个用户的权限做了修改，也<font color = "#CD5555">**不会影响**</font>已经存在连接的权限。

以下命令查看有哪些空闲连接：

```mysql
show processlist;
```

长时间没操作时，连接会断开，这个时间默认为<font color ="#CD5555">**8小时**</font>，是由`wait_timeout`控制的。

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">长连接</font></center>
**1、弊端**

但是全部使用长连接后，你可能会发现，有些时候 MySQL 占用内存涨得特别快，这是因为 MySQL 在执行过程中<font color ="#CD5555">**临时使用的内存**</font>是管理在<font color="#CD5555">**连接对象**</font>里面的。这些资源会在连接断开的时候才释放。所以如果长连接累积下来，可能导致内存占用太大，被系统强行杀掉（OOM），从现象看就是 MySQL 异常重启了。

**2、解决方案**

- 定期断开长连接。使用一段时间，或者程序里面判断执行过一个占用内存的大查询后，断开连接，之后要查询再重连。
- 如果你用的是 MySQL 5.7 或更新版本，可以在每次执行一个比较大的操作后，通过执行 `mysql_reset_connection`来重新初始化连接资源。这个过程不需要重连和重新做权限验证，但是会将连接恢复到刚刚创建完时的状态。

>
> **思考**：客户端每次查询用的是不同的连接吗？什么情况下的操作占用内存比较大？怎么查看MYSQL占用的内存，以及内存上限？怎么reset connection？
>

#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">查询缓存</font></center>

8.0版本后就没有这个功能了。



