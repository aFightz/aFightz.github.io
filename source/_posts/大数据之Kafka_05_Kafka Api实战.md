---
title: 05 | 大数据之Kafka_05_Kafka Api实战
date: 2019-08-09 14:41:04
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- 大数据之Kafka
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">生产者Api使用</font></center>
**1、pom文件**
```
<dependencies>
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>0.11.0.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka_2.12</artifactId>
        <version>0.11.0.0</version>
    </dependency>
</dependencies>
```

**2、生产者Demo**
```java
public static void main(String[] args) {
    Properties props = new Properties();
    // Kafka服务端的主机名和端口号
    props.put("bootstrap.servers", "hadoop103:9092");
    // 等待所有副本节点的应答
    props.put("acks", "all");
    // 消息发送最大尝试次数
    props.put("retries", 0);
    // 一批消息处理大小
    props.put("batch.size", 16384);
    // 请求延时
    props.put("linger.ms", 1);
    //达到linger.ms的时间或者达到batch.size就会提交。
    // 发送缓存区内存大小
    props.put("buffer.memory", 33554432);
    // key序列化
    props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    // value序列化
    props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
     
    Producer<String, String> producer = new KafkaProducer<>(props);
    for (int i = 0; i < 50; i++) {
        producer.send(new ProducerRecord<String, String>("first", Integer.toString(i), "hello world-" + i));
    }
    producer.close();
}
```


#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">带回调函数的生产者</font></center>
```java
producer.send(new ProducerRecord<String, String>("first", "hello" + i), new Callback() {
 
    @Override
    public void onCompletion(RecordMetadata metadata, Exception exception) {
     
        if (metadata != null) {
            System.err.println(metadata.partition() + "---" + metadata.offset());
        }
    }
});
```

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">自定义分区的生产者</font></center>
```java
public class CustomPartitioner implements Partitioner {
 
    @Override
    public void configure(Map<String, ?> configs) {
    }
     
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        // 控制分区
        return 0;
    }
     
    @Override
    public void close() {
    }
}
```

```java
props.put("partitioner.class", "com.atguigu.kafka.CustomPartitioner");
```