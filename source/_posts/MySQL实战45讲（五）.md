---
title: 05 | 深入浅出索引（下）
date: 2019-01-08 16:43:43
tags: [MySQL]
categories :
- 学习笔记
- MySQL
- MySQL实战45讲
---


索引的常见模型
哈希表。适合插入数据与做等值查询，不适合做区间查询。
有序数组。适合做等值查询与区间查询（使用二分法），不适合插入数据。所以这种适合做静态存储引擎。
搜索树。二叉搜索树->平衡二叉树->N叉树（B+树）
二叉搜索树：左孩子节点<父节点<右孩子节点
平衡二叉树：左右两棵子树的高度差绝对值不超过1，并且左右两棵子树都是平衡二叉树。（维持 O(log(N)) 的查询复杂度）
N叉树：（为了查询尽量少地读磁盘）。


覆盖索引
假设现在t_person有两个字段，id、name、age、ismale,其中id是主键，name加了索引。
那么
select * from t_person where name = 'a';
会导致回表。
但是
select id from t_person where name = 'a';
将不会导致回表，这就叫做覆盖索引。

如果有很高频的通过name来找age这样的请求，我们也可以为name与age作联合索引，这样也可以避免回表。 


索引下推
假设现在给加了索引(name , age)
那么下面语句将会先过滤，减少回表次数
select * from tuser where name like '张%' and age=10 and ismale = 1 ;
![](MySQL实战45讲（五）\索引下推.jpg)








