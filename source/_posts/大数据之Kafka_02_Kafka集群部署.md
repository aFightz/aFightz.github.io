---
title: 02 | Kafka集群部署
date: 2019-08-01 23:35:15
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- 大数据之Kafka
---


**1、解压安装包**
```
tar -zxvf kafka_2.11-0.11.0.0.tgz -C /opt/module/
```

**2、修改解压后的文件名称**
```
mv kafka_2.11-0.11.0.0/ kafka
```
 
**3、在/opt/module/kafka 目录下创建 logs 文件夹**
```
mkdir logs
```

**4、修改配置文件**
```
cd config/
vi server.properties
```
输入以下内容：
```properties
#broker的全局唯一编号，不能重复
broker.id=0

#删除topic功能使能
delete.topic.enable=true

#处理网络请求的线程数量
num.network.threads=3

#用来处理磁盘IO的现成数量
num.io.threads=8

#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400

#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400

#请求套接字的缓冲区大小
socket.request.max.bytes=104857600

#kafka运行日志存放的路径
log.dirs=/opt/module/kafka/logs

#topic在当前 broker上的分区个数
num.partitions=1

#用来恢复和清理 data下数据的线程数量
num.recovery.threads.per.data.dir=1

#已关闭segment文件保留的最长时间，超时将被删除，单位是小时，即是7天。
log.retention.hours=168

#segment文件超过这个大小就会被关闭，等待删除（1个G）
log.segment.bytes=1073741824

#配置连接Zookeeper集群地址
zookeeper.connect=hadoop102:2181 , hadoop103:2181 , hadoop104:2181
```
> 黑科技：可以用sublime text直接连上服务器的文件修改。（安装sftp插件）
> **问题**：kafka配置文件中没有指定kafka的集群配置，只指定了zk的集群配置。这其中是怎么实现kafka集群的？

**5、配置环境变量**
（这步其实可以不用配置，只是方便而已，但是可以了解一下配置环境变量的步骤）
```
sudo vi /etc/profile
```

```
#KAFKA_HOME 
export KAFKA_HOME=/opt/module/kafka 
export PATH=$PATH:$KAFKA_HOME/bin 
source /etc/profile
```

**6、同步**
```
xsync kafka/
```

**7、分别在各台服务器启动**
（启动之前需要先启动zk）
```
bin/kafka-server-start.sh config/server.properties
```

**8、关闭**
```
bin/kafka-server-stop.sh stop
```


