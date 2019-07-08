---
title: 02 | Docker的安装
date: 2019-07-02 20:03:01
tags: [Docker]
categories :
- 学习笔记
- Docker
- Docker核心技术
---

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">系统要求</font></center>
Docker运行在<font color = "#CD5555">CentOs-6.5</font>或更高版本的CentOs上，要求系统为64位、系统内核版本为<font color = "#CD5555">2.6.32-431</font>或者更高版本。

**查看自己的内核**
```
uname -r
```
**查看centos版本**
```
cat /etc/reahat-release
```
 


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">Centos6.8安装Docker</font></center>

**1、Docker使用EPEL发布，RHEL系的OS首先要确保已经持有EPEL仓库，否则先检查OS的版本，然后装相应的EPEL包。**
```
yum install-y epel-release
```
**2、安装Docker**
```
yum install-y docker-io
```
**3、安装后的配置文件路径**
```
/etc/sysconfig/docker
```

**4、启动Docker后台服务**
```
service docker start
```

**5、验证Docker版本**
```
docker version
```
















