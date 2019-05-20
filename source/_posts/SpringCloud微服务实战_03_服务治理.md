---
title: 03 | 服务治理 ：Spring Cloud Eureka
date: 2019-05-05 10:43:35
tags: [Spring Cloud]
categories :
- 学习笔记
- Java
- Spring
- SpringCloud微服务实战
---

Spring Cloud Eureka是Spring Cloud Netfix的一部分，基于Netfix Eureka做了二次封装。

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">服务治理</font></center>
服务治理包含了两部分的内容：<font color = "#CD5555">服务注册</font>与<font color = "#CD5555">服务发现</font>。
**1、什么叫做服务注册？**
服务注册就是服务单元向注册中心注册自己的服务。其中注册中心维护一个服务清单（一个服务可能有多个实例），还需要以心跳的方式去监测清单中的服务是否可用。

**2、什么叫做服务发现？**
服务调用方不会直接调用服务提供方，而是向注册中心请求这个服务的实例列表，之后通过某种策略从这列表中选出一个实例进行调用。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">Eureka服务器端(服务注册中心)</font></center>
Eureka服务器端可以以集群方式部署，并且服务注册中心之间<font color = "#CD5555">以异步模式互相复制各自的状态</font>。当某个分片（注册中心）故障时，继续提供服务的发现和注册。当故障分片恢复运行时，集群中的其他分片会把他们的状态同步回故障分片。


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">搭建服务注册中心</font></center>

**1、maven配置**
在第二章maven配置的基础上，加上如下配置：

```markdown
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Greenwich.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**2、注解**
在Application上加上`@EnableEurekaServer`

**3、配置**

```markdown
#服务器端口，默认为8761
server.port=1111
#是否向注册中心注册自己，默认为true
eureka.client.register-with-eureka = false
#是否去检索服务（服务发现），默认为true
eureka.client.fetch-registry = false
#默认为http://localhost:8761/eureka/
eureka.client.serviceUrl.defaultZone = http://127.0.0.1:1111/eureka/
#应用名
spring.application.name=center1
```

**4、访问注册中心页面**
`http://127.0.0.1:1111/`

#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">搭建服务提供者</font></center>

**1、注解**
在Application上加上`@EnableEurekaClient`

**2、配置**
```markdown
#如有多个则以逗号隔开
eureka.client.serviceUrl.defaultZone = http://localhost:1111/eureka/
spring.application.name = hello.server
```

**3、查看是否生效**
可在服务中心的`Instances currently registered with Eureka`一栏看到服务的信息。


#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">高可用注册中心</font></center>

**1、配置**
只需要理解`eureka.client.serviceUrl.defaultZone`这个参数的意义即可，它相当于把注册中心用一条线连起来。只要N个注册中心可以用一条线连起来（线路可达），那么就形成了高可用集群。当超过2个注册中心时，集群中的节点需要相互关联，否则会导致某个注册中心不可达。

> 有个问题：当服务注册到注册中心后，再停掉服务，从注册中心页面并不能看出这个服务已经下线。
>


#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">服务发现与消费</font></center>

**1、maven配置**
```markdown
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

**2、代码**
```java
@SpringBootApplication
@EnableDiscoveryClient
public class Application {

    @Bean
    @LoadBalanced
    RestTemplate restTemplate(){
        return  new RestTemplate();
    }
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class , args);
    }
}
```

```java
@Service
public class UserServiceImpl implements UserSerivce {

    @Autowired
    private RestTemplate restTemplate;
    
    @Override
    public String say() {
         return restTemplate.getForEntity("http://HELLO.SERVER/hello",String.class).getBody();
    }
}
```
需要注意的是上述的`HELLO.SERVER`是服务提供者模块的`spring.application.name`属性名。



