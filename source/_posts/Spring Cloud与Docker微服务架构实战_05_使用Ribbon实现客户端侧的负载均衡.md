---
title: 05 | 使用Ribbon实现客户端侧的负载均衡
date: 2019-06-04 16:04:21
tags: [Spring Cloud]
categories :
- 学习笔记
- Java
- Spring
- Spring Cloud与Docker微服务架构实战
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">获得当前选择的服务节点</font></center>
```java
@Autowired
private LoadBlancerClient loadBlancerClient;
```
```java
ServiceInstance instance = this.loadBlancerClient.choose("service-name");
instance.getHost()/instance.getPort();
```

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">虚拟主机名称</font></center>
默认情况下，虚拟主机名称即服务名称（也可通过`application-name`指定服务名称）。
也可配置`eureka.instance.virtual-host-name`或`eureka.instance.security-virtual-host-name`设置虚拟主机名称。
虚拟主机名不能含有“_”这个字符，否则会报异常。
不能将`restTemplate.getForObject(...)`与`loadBalanceClient.choose(...)`写在同一个方法，会有冲突。


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">自定义Ribbon</font></center>
**1、使用Java配置方式配置自定义Ribbon**
Ribbon默认的配置类是`RibbonClientConfiguration`。使用自定义配置会覆盖这个默认配置。所以我们在为某一个Ribbon设置一个自定义配置时，不应该让这个自定义配置被@ComponentScan扫描到。

```java
@Configuration
public class RibbonConfiguration{
    @Bean
    public IRule ribbonRule(){
        return new RandomRule();
    }
}
```

```java
//为特定service-name的ribbon client指定自定义配置类。
@Configuration
@RibbonClient(name = "xxx-service" , configuration = 
    RibbonConfiguration.class)
public class TestConfiguration{
}
```

也可以用`@RibbonClients`为所有Ribbon Client提供默认配置。（让@RibbonClient的配置被扫描到，不就是全局配置了？）


**2、使用属性自定义Ribbon配置**

属性的前缀是service-name.ribbon
- NFLoadBanlancerClassName：配置ILoadBalancer的实现类。
- NFLoadBanlancerRuleClassName：配置IRule的实现类。
- NFLoadBanlancerPingClassName：配置Iping的实现类。
- NIWSServerListClassName：配置ServerList的实现类。
- NIWSServerListFilterClassName：配置ServerListFilter的实现类。
属性配置的方式比Java配置方式的<font color = "#CD5555">优先级更高</font>。

**3、脱离Eureka使用Ribbon**
```
ribbon.eruka.enabled=false
service-name.ribbon.listOfServer= xxx,xxx
```

**4、指定名称的Ribbon请求特定URL，其他Ribbon使用Eureka**
```
service-name.ribbon.NIWSServerListClassName = com.netflix.loadbalancer.ConfigurationBasedServerList
service-name.ribbon.listOfServers = xxx,xxx
```

#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">Ribbon的加载</font></center>
Ribbon上下文的加载默认是<font color = "#CD5555">懒加载</font>的（第一次请求才会加载），可通过如下方式配置成<font color = "#CD5555">饥饿加载</font>。
```
ribbon:
   eager-load:
      enabled: true
      clients: client1,client2
```