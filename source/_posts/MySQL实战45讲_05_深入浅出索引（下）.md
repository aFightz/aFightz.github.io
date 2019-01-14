---
title: 05 | 深入浅出索引（下）
date: 2019-01-08 16:43:43
tags: [MySQL]
categories :
- 学习笔记
- MySQL
- MySQL实战45讲
---


### 覆盖索引
假设现在t_person表如下：

| 主键 | 索引 | 普通字段 | 普通字段 |
| :--: | :--: | :------: | :------: |
|  id  | name |   age    |  ismale  |

```sql
#以下语句会导致回表
select * from t_person where name = 'a';
```
```sql
#以下将不会导致回表,这就叫做覆盖索引
select id from t_person where name = 'a';
```


如果有很高频的通过name来找age这样的请求，我们也可以为name与age作联合索引，这样也可以避免回表。 


### 索引下推
假设现在给加了索引(name , age)
那么下面语句将会先过滤，减少回表次数

```sql
select * from t_user where name like '张%' and age = 10 and ismale = 1 ;
```

![](MySQL实战45讲_05_深入浅出索引（下）\索引下推.jpg)









