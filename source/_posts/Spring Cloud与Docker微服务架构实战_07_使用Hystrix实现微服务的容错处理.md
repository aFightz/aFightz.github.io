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

断路器状态转换图


整合Hystrix
- 引入spring-cloud-starter-netflix-hystrix
- 在启动类上添加@EnableCircuitBreaker或@EnbaleHystrix
- 在mapping方法上添加如下注解
@HystrixCommand(fallbackMethod = "findByIdFallback")

findByIdFallback是一个自定义方法，若业务发生异常则则回调用此方
法。
public User findByIdFallback(Long id){
...
}
- @HystrixCommand下的commandProperties可配置超时等属性。
- findByIdFallback的参数与mapping注解下的方法参数需一致，若想对异常
  进行一些处理，可以再加上Throwable参数，如下
  ... findByIdFallback(Long id ,Throwable e)
  - 若在某些异常情况下，不回调findByIdFallback方法，可以通过
    @HystrixCommand下的ignoreExceptions属性去设置。

Hystrix与Actuator
- 可以通过Actuator去监控Hystrix状态，但服务正常时，状态应该是up，但是当我们关闭服务时再访问，Hystrix会由于业务失败执行回退逻辑，但是此时Hystrix的状态仍旧是up（断路器还未被打开），这是由于失败率还没达到阈值（默认是5S内20次失败）。

Hystrix线程隔离策略
- Hystrix线程隔离策略有两种
- THREAD：HystrixCommand将在将在单独的线程执行。
- SEMAPHORE：HystrixCommand将在调用线程上执行。
- Hystrix默认的策略是THREAD。
- 在同一个线程，才能访问request上下文，所以使用THREAD策略会出现	
  访问不到上下文的情况。
- 可通过execution.isolation.strategy属性指定隔离策略。

Feign使用Hystrix
Srping Cloud 默认已经为Feign整合了Hystrix。
- feign.hystrix.enbaled=true
- 为FeignClient接口创建一个实现类。
- 然后在FeigonClient接口上使用@FeignClient的fallback属性指定实现
  类，即可实现hystrix的回滚功能。

如果需要知道回退错误类型，则
- 实现FallbackFactory<FeignClient>方法，复写
public FeignClient create(Throwable cause){
}
这个方法。
- 在FeignClient接口上使用@FeignClient的fallbackFactory属性指定实现
  类。

- 为指定Feign禁用Hystrix
- FeignClient默认是不开启Hystrix的，只是我们之前在全局加了开启的	全局的配置。所以要为指定Feign禁用Hystrix，只需要为特定的Feign注入一个初始的FeignClient(没开启Hystrix)而不注入全局的FeignClient即可。
@Configuration
public class xxxConfiguration{
@Bean
public Feign.Builder feignBulider(){
Feign.builder();
}
}
之后在FeignClient的configuration属性指定这个配置即可。

Hystrix的监控
- 只要添加了actuator模块，就能通过/hystrix.stream看到相应的监控信息。
- 可使用Hystrix Dashborad可视化监控数据。
- 可使用Turbine聚合监控数据。


