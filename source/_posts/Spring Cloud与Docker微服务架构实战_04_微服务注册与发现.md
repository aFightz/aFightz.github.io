---
title: 04 | 微服务注册与发现
date: 2019-06-04 11:18:43
tags: [Spring Cloud]
categories :
- 学习笔记
- Java
- Spring
- Spring Cloud与Docker微服务架构实战
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">Eureka Server</font></center>
微服务启动后，会周期性(<font color = "#CD5555">默认为30s</font>)的向Eureka Server发送心跳，以续约自己的租期。
Eureka Server在一定时间（<font color = "#CD5555">默认为90s</font>）没有接收到微服务实例的心跳，将注销该实例。
默认情况下Eureka Server也是Eureka Client。
Eureka Client会缓存服务注册表中的信息。
如果是一个单点的Eureka Server，可以把`eureka.client.registerWithEureka`设为false，因为不需要同步其他的Eureka Server节点的信息。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">注册</font></center>
若不配置`eureka.instance.prefer-ip-address`这个属性或者将其设为false，则表示注册微服务<font color = "#CD5555">所在操作系统的hostname</font>到Eureka Server。
在Spring Cloud Edgware以及更高版本中，只需要添加相关依赖，即可自动注册。
若不想将服务注册到Eureka Server，只需设置`spring.cloud.service-registry.auto-registration.enabled = false`或`@EnableDiscoverClient(autoRegister=false)`。


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">Eureka Server高可用</font></center>
使用yml实现的比较新奇的配置：
```
spring:
  application:
    name : xxxx
---
spring:
  profiles: peer1
server:
  port: 8761
eruka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: http://peer2:8762/eureka/
---
spring:
  profiles: peer2
server:
  port: 8762
eruka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/ 
```
上面使用---将yml文件分为三段，第一段未指定profile，表示是公用的，对所有profile都生效。


#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">为Eureka Server开启认证</font></center>
添加spring-boot-security-starter。

**1、通过配置方式开启Basic认证**
yml添加以下内容：
```
security:
   basic:
      enabled: true
user:
   name: user
      password: password@123 
eureka:
   client:
      serviceUrl:
      defaultZone: http://user:password@123localhost:8761/eureka/
```

**2、通过生命Bean的方式开启Basic认证**

```java
@Bean
public DiscoveryClientOptionalArgs discoveryClientOptionalArgs() {
    DiscoveryClientOptionalArgs discoveryClientOptionalArgs = new DiscoveryClientOptionalArgs();
    List<CilentFilter> additionalFilters = new ArrayList<>();
    additionalFilters.add(new HTTPBasicAuthFilter("user" , "password123"))
    discoveryClientOptionalArgs.setAdditionalFilters(additionalFilters);
    return discoveryClientOptionalArgs;
}
```



#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">Eureka的元数据</font></center>
Eureka的元数据有两种，分别是<font color = "#CD5555">标准元数据</font>和<font color = "#CD5555">自定义元数据</font>。
- 标准元数据：主机名、IP地址、端口号、状态页和健康检查等信息。
- 自定义元数据：eureka.instance.metadata-map配置（key-value）。

`DiscoveryClient.getInstance(serviceId)`可查询指定微服务在Eureka上的实例列表。包括标准元数据和自定义元数据。
`/eureka/apps`可查看Eureka的metadata。


#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">手动对服务进行操作</font></center>
可通过 `/eureka/apps`下的端点，进行服务注册，服务查询，服务注销等操作（用xml交互）。


#### <center><font color = "#36648B">✎✎✎✎✎✎✎</font><br/><font color = "#36648B">Eureka的自我保护</font></center>
进入自我保护模式后，Eureka Server的首页会显示告警。
之前有说到，在一定时间（<font color = "#CD5555">默认90s</font>）内，如果没收到客户端的心跳，那么server会注销这个客户端，但是如果在短时间丢失过多的客户端（出现网络故障的原因），那么server就会进入自我保护模式，不会删除注册列表里的客户端。当网络故障恢复后，该server会自动退出自我保护模式。
禁用自我保护模式。`eureka.server.enable-self-perservation = false`。


#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">指定注册到server上的IP</font></center>

**1、忽略指定名称的网卡**
```
spring:
   cloud:
      inetutils:
         ignored-interfaces:
            - docker0
            - veth.*
eureka:
   instance:
      prefer-ip-address: true
```

**2、使用正则表达式，指定使用的网络地址**
```
spring:
   cloud:
      inetutils:
         preferredNetworks:
            - 192.168
            - 10.0
eureka:
   instance:
      prefer-ip-address: true
```
**3、只使用站点本地地址**
```
spring:
   cloud:
      inetutils:
         userOnlySiteLocalInterfaces: true
eureka:
   instance:
      prefer-ip-address: true
```
**4、手动指定IP地址**
```
eureka:
   instance:
      prefer-ip-address: true
      ip-address: 127.0.0.1
```  
      
#### <center><font color = "#36648B">✎✎✎✎✎✎✎✎✎</font><br/><font color = "#36648B">健康检查</font></center>
**1、开启健康检查（不仅仅是心跳保持正常）**
`eureka.client.healthcheck.enbaled: true`
加了上述配置时，/pause端点无法正常工作。也无法将状态标记为	down。这个bug未修复。

**2、实现自己的健康检查**
实现HealthCheckHandler接口。