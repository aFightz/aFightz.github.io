---
title: 07 | 使用Hystrix实现微服务的容错处理
date: 2019-06-04 17:35:40
tags: [Spring Cloud]
categories :
- 学习笔记
- Java
- Spring
- Spring Cloud与Docker微服务架构实战
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">断路器状态转换图</font></center>
![](Spring Cloud与Docker微服务架构实战_07_使用Hystrix实现微服务的容错处理\断路器状态转换图.png)


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">整合Hystrix</font></center>
- 引入`spring-cloud-starter-netflix-hystrix`
- 在启动类上添加`@EnableCircuitBreaker`或`@EnbaleHystrix`
- 在Mapping方法上添加注解:
```java
@HystrixCommand(fallbackMethod = "findByIdFallback")
```

    - **<font color = "#CD5555">findByIdFallback</font>**是一个自定义方法，若业务发生异常则则回调用此方法。
```java
public User findByIdFallback(Long id){
...
}
```

    - `@HystrixCommand`下的`commandProperties`可配置<font color = "#CD5555">超时</font>等属性。
    - findByIdFallback的参数与Mapping注解下的方法参数需<font color = "#CD5555">一致</font>，若想对异常进行一些处理，可以再加上<font color = "#CD5555">Throwable</font>参数，如下:
```java
public User findByIdFallback(Long id ,Throwable e){
...
}
```
    - 若在某些异常情况下，不回调findByIdFallback方法，可以通过`@HystrixCommand`下的`ignoreExceptions`属性去设置。

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">Hystrix与Actuator</font></center>

可以通过Actuator去监控Hystrix状态。服务正常时，状态是up，但是当我们关闭服务时再访问，Hystrix会由于业务失败执行回退逻辑，此时Hystrix的状态仍旧是up，<font color = "#CD5555">这是由于失败率还没达到阈值，断路器还未被打开</font>（默认是<font color = "#CD5555">5S内20次失败</font>）。


#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">Hystrix线程隔离策略</font></center>
- Hystrix线程隔离策略有两种：
  - `THREAD`：HystrixCommand将在将在<font color = "#CD5555">单独的线程</font>执行。
  - `SEMAPHORE`：HystrixCommand将在<font color = "#CD5555">调用线程</font>上执行。
- Hystrix<font color = "#CD5555">默认的策略</font>是`THREAD`。
- 在同一个线程，才能访问request上下文，所以使用THREAD策略会出现访问不到上下文的情况。
- 可通过`execution.isolation.strategy`属性指定隔离策略。

#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">Feign使用Hystrix</font></center>

Srping Cloud默认已经为Feign整合了Hystrix。

**1、feign开启hystrix**
- 设置`feign.hystrix.enbaled=true`
- 为FeignClient接口创建一个实现类。
- 然后在FeignClient接口上使用`@FeignClient`的`fallback`属性指定<font color = "#CD5555">实现类</font>，即可实现hystrix的回滚功能。

- 如果需要知道回退错误类型，则：
    - 实现`FallbackFactory<FeignClient>`类，复写create方法:
    ```java
    public FeignClient create(Throwable cause){
    }
    ```
    - 在FeignClient接口上使用`@FeignClient`的`fallbackFactory`属性指定实现类。

**2、为指定Feign禁用Hystrix**
FeignClient默认是不开启Hystrix的，只是我们之前在全局加了开启Hystrix配置。所以要为指定Feign禁用Hystrix，只需要为特定的Feign注入一个初始的FeignClient(没开启Hystrix)而不注入全局的FeignClient即可。
```java
@Configuration
public class xxxConfiguration{
    @Bean
    public Feign.Builder feignBulider(){
        Feign.builder();
    }
}
```
之后在FeignClient的`configuration`属性指定这个配置即可。

#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">Hystrix的监控</font></center>
- 只要添加了actuator模块，就能通过`/hystrix.stream`看到相应的监控信息。
- 可使用`Hystrix Dashborad`可视化监控数据。
- 可使用`Turbine`聚合监控数据。


