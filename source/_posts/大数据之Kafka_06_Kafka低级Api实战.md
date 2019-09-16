---
title: 05 | 大数据之Kafka_06_Kafka低级Api实战
date: 2019-09-16 15:04:16
tags: [Kafka]
categories :
- 学习笔记
- Kafka
- 大数据之Kafka
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">消费使用步骤</font></center>
- 获取指定的分区的leader
- 获取分区最新的消费进度
- 从leader拉取分区的消息
- 识别leader的变化，重试

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">API实现</font></center>
**1、参数定义**
```java
//定义相关参数
ArrayList<String>brokers=new ArrayList<>();
//kafika集群
brokers.add("hadoop102");
brokers.add("hadoop103");
brokers.add("hadoop104");
//端口号
int port=9092；
//主题
String topic="second"；
/分区
int partition=0；
//offset 
long offset=2;
```

**2、获取分区leader**
```java
private BrokerEndPoint findLeader(List<String> brokers, int port, String topic, int partition){
    for(String broker : brokers){
        //创建获收分区leader的消费者对象
        SimpleConsumer getLeader=new SimpleConsumer(broker , port , 1000, 1024*4 , "getLeader");
        //创建一个主题元数据信息请求
        TopicMetadataRequest topicMetadataRequest = new TopicMetadataRequest (Collections.singletonlist(topic));
        
        //获取主题元数据返回信
        TopicMetadataResponse metadataResponse = getLeader.send(topicMetadataRequest);
        
        //解析元数据返回值
        List<TopicMetadata> topicsMetadata = metadataResponse.topicsMetadata();
        
        //遍历主题元数据
        for(TopicMetadata topicMetadatum : topicsMetadata){
            //获取多个分区的元数据信息
            List<PartitionMetadata> partitionsMetadata = topicMetadatum.partitionsMetadata();
            
            //遍历分区元数据
            for(PartitionMetadata partitionMetadata : partitionsMetadata){
                //partition  = 0
                if(partition == partitionMetadata.partitionld()){
                    return partitionMetadata.leader()；
                }
            }
        }
    }
    return null;
}
```

**3、获取分区数据**
```java
private void getData(List<String> brokers, int port, String topic, int partition, long offset){
    //获取分区leader 
    BrokerEndPoint leader = findLeader(brokers, port, topic, partition);
    if(leader == null){
        return;
    }
    
    String leaderHost = leader.host();
    //获取数据的消费者对象
    SimpleConsumer getData = new SimpleConsumer (leaderHost, port, 1000, 1024*4, "getData");
    
    //创建获取数据的对象,100表示获取返回的字节数
    FetehRequest fetchRequest = new FetehRequestBuilder().addFetch(topic, partition, offset, 100).build();
    
    //获取数据返回值
    FetchResponse fetchResponse=getData.fetch(fetchRequest);
    
    //解析返回值
    ByteBufferMessageSet messageAndoffsets = fetchResponse.messageSet(topic，partition);
    
    for(MessageAndOffset messageAndOffset : messageAndOffsets){
        long offset1 = messageAndOffset.offset();
        ByteBuffer payload = messageAndOffset.message().payload();
        byte[] bytes = new byte[payload.limit()];
        payload.get(bytes);
        System.out.println(offset1+"—"+new String(bytes));
    }
}
```