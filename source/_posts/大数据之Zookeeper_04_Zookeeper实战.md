---
title: 04 | Zookeeper实战
date: 2019-07-15 20:02:50
tags: [Zookeeper]
categories :
- 学习笔记
- Zookeeper
- 大数据之Zookeeper
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">分布式安装部署</font></center>
**1.集群规划**
A、B和C三个节点上部署Zookeeper。

**2.解压安装。**
- 在A的服务器解压安装zk。
- 同步zk到B、C。
```
xsync [ZK_PATH]/
```
> xsync是怎么同步A到B和C上的，在其他地方是否有相关设置？

**3.配置服务器编号。**
- 创建目录`[dataDir_path]`。
- 在`[dataDir_path]`目录下创建一个`myid`的文件。
- 编辑`myid文件`，在文件中添加与server对应的唯一编号`[SERVER_ID]`。
- 拷贝配置好的zookeeper到其他机器上。
```
xsync myid
```
并分别在B、C上修改myid文件中内容为自己的`[SERVER_ID]`。

**4.配置 zoo.cfg文件**
- 修改配置文件为zoo.cfg。
- 修改数据存储路径配置dataDir为`[dataDir_path]`。
- 增加如下配置。
```
server.[A_ID]=A:2888:3888
server.[B_ID]=B:2888:3888
server.[C_ID]=C:2888:3888
```
- 同步zoo.cfg配置文件。
```
xsync zoo.cfg
```
> 这里的A、B、C是怎么映射到IP的？

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">配置参数解读</font></center>
`server.A=B:C:D`
- A是一个数字，表示这个是第几号服务器。集群模式下在dataDir目录下配置一个文件myid，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。
- B是这个服务器的ip地址。
- C是这个服务器与集群中的Leader服务器交换信息的端口。
- D是用来执行**重新选举**时服务器相互通信的端口。

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">集群操作</font></center>
- 分别启动Zookeeper。
- 分别查看状态。
  可从结果中知道这个节点是follower还是leader。
  
  
#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">客户端命令行操作</font></center>
**1、显示所有操作命令**
```
help
```

**2、查看znode中所包含的内容**
- 查看当前znode
```
ls /
```
- 查看sanguo znode
```
ls /sanguo
```

**3、查看当前节点详细数据**
```
ls2 /
```

**4、创建普通节点**
```
create /sanguo "jinlian"
```
也可创建多级目录：
```
create /sanguo/shuguo "liubei"
```

**5、获得节点的值**
```
get /sanguo/shuguo
```

**6、创建短暂节点**
```
create -e /sanguo/wuguo "zouyu"
```

**7、创建带序号的节点**
```
create -s /sanguo/weiguo "caocao"
```

**8、修改节点数据值**
```
set /sanguo/shuguo "diaocan"
```

**9、节点的值变化监听**
```
get /sanguo watch(只监听一次)
```

**10、节点的子节点变化监听（路径变化）**
```
ls /sanguo watch(只监听一次)
```

**11、删除节点**
```
delete /sanguo/banzhang
```

**12、递归删除节点**
```
rmr /sanguo
```

**13、查看节点状态**
```
stat /sanguo
```



#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">API</font></center>
**1、创建zk客户端**
```java
public class TestZookeeper{
    private String connectString="hadoop102:2181,hadoop103:2181,hadoop104:2181"; 
    private int sessionTimeout=2000; 
    private Zookeeper zkClient;
    
    @Test 
    public void init()throws IOException{
        zkClient=new Zookeeper(connectstring,sessionTimeout,new Watcher(){
            @Override 
            public void process(WatchedEvent event){
                //do something
            }
        });
    }
}
```
当有数据变化时，process会被调用。

**2、创建一个节点**
```java
@Test 
public void createNode()throws KeeperException，InterruptedException{
    String path =zkClient.create（
        "/atguiku",
        "dahaigezuishuai".getBytes(),
        Ids.OPEN_ACL_UNSAFE,
        CreateMode.PERSISTENT
    );
    System.out.print1n(path);
}
```


**3、获取子节点**
```Java
List<String> children = zkClient. getchildren("/", true);
```
true表示注册监听。



**4、判断节点是否存在**
```Java
Stat stat=zkclient. exists("/atguigu", false);
```

**5、监听服务器节点动态上下线**
流程如下：
- 服务端启动时去注册信息（创建都是临时节点）。
- 客户端启动就去getChildren，获取到当前在线服务器列表，并且注册监听。
- 服务器节点下线。
- 服务器节点上下线事件通知。
- process()重新再去获取服务器列表，并注册监听。

重点在于服务器向zk注册信息时，创建的节点是**临时的且带编号**的。
> 问题：如果多台机创建的path相同（节点相同），那么如果节点是带编号的，就不会重合，但如果不带编号的，又会是什么状况呢？