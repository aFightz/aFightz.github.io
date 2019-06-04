---
title: 06 | 使用Feign实现声明式REST调用
date: 2019-06-04 16:47:51
tags: [Spring Cloud]
categories :
- 学习笔记
- Java
- Spring
- Spring Cloud与Docker微服务架构实战
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">整合Feign</font></center>

- 添加Feign的依赖`spring-cloud-starter-openfeign`
- 创建Feign接口
```java
@FeignClient(name = "service-name")
public interface UserFeignClient{
   @RequestMapping(value = "/{id}" , method = RequestMethod.GET)
   User findById(@PathVariable Long id);
}
```
- 在启动类添加`@EnbaledFeignClients`注解。

因为使用了Eureka，所以Ribbon会把service-name解析成注册中心列表里的服务。
可以使用`service.ribbon.listOfServers`属性配置服务列表。
也可以通过`@FeignClient`的`url`属性配置特定的地址。


#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">自定义Feign配置</font></center>

Feign的默认配置是`FeignClientsConfiguration`。
可以通过`@FeignClient`的`configuration`属性自定义Feign的配置，自定义配置的优先级比FeignClientsConfiguration高。

**1、自定义配置Demo**
```java
public class FeignConfiguration{
   @Bean
   public Contract feignContract(){
      return new feign.Contract.Default();
   }
}
```
这个配置<font color = "#CD5555">不能加</font>`@Configuration`注解，否则如果被扫描到会被替换成全局配置。

**2、基于Http Basic认证的Demo**
```java
public class FeignConfiguration{
   @Bean
   public BasicAuthRequestInterceptor basicAuthRequestInterceptor(){
      return new BasicAuthRequestInterceptor("user" , "password");
   }
}
```

**3、全局配置**
用`@EnableFeignClients`的`defaultConfiguration`属性用来指定默认的配置类。


**4、配置指定名称的Feign Client**
```
feign.client.config.feignName.*(各种属性)
```

**5、配置通用的Feign Client**
```
feign.client.config.default.*(各种属性)
```

属性配置的方式比java配置的优先级更高，如果想让java配置的优先级更高，可以配置如下属性，
```
feign.client.default-to-properties=false
```

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">手动创建一个Feigon</font></center>

- FeigonClient接口上的@FeignClient注解以及启动类上的@FeigonClients移除。
- Controller Demo
```java
@Import(FeignClientsConfiguration.class)
@RestController
public class MovieController{
   private UserFeignClient userFeignClient;
   private adminFeignClient adminFeignClient;

   @Autowired
   public MovieController(Decoder decoder , Encoder encoder , Client client , Contract contract){
      this.userFeignClient = Feign.builder().client(client).encoder(encoder).decoder(decoder).contract(contract)
      .requestInterceptor(new BasicAuthRequestInterceptor("user" , "password1"))
      .target(UserFeignClient.class , "http://xxx-service/");
      //adminFeignClient类似上面
   }
}
```

`FeignClientsConfiguration`是Spring Cloud为Feign提供的默认配置类。

Feign可以设置为对请求或响应进行压缩，还能对压缩进行一些详细的设置。
Feign默认的日志级别是DEBUG，可以通过配置进行一些自定义的日志级别配置，甚至可以对某一特定服务进行单独的日志级别配置。

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">构造多参数请求</font></center>
- 使用@RequestParam和Map构造get的多参数请求。
- 使用@RequestBody构造post的多参数请求。


#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">使用Feign上传文件</font></center>
- 引入feign-form（需要使用它的Encoder）。
- Hystrix的超时时间需要设长一点。不然文件还没上传完就超时了。



