---
title: 08 | 使用Zuul构建微服务网关
date: 2019-06-05 13:51:29
tags: [Spring Cloud]
categories :
- 学习笔记
- Java
- Spring
- Spring Cloud与Docker微服务架构实战
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">Zuul使用的客户端</font></center>
- Apache HTTP Client(<font color = "#CD5555">默认</font>)
- RestClient(设置`ribbon.restclient.enabled=true`)
- okhttp3.OkHttpClient(设置`ribbon.okhttp.enabled=true`)   

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">Zuul的整合</font></center>
- 引入`spring-cloud-starter-netfilx-zuul`
- 在启动类上添加`@EnbaleZuulProxy`
- 在配置加上Eureka注册中心的地址（zuul整合了Ribbon与Hystrix）。
- 访问形式`http://ZUUL_HOST:ZUUL_PROT/微服务名称/**`


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">管理端点</font></center>
- 通过`/routes`访问端点。
- 通过`/routes?format=details`可以访问更多的详细信息。
- 使用GET方式请求，可以得到当前映射的路由列表。
- 使用POST方式请求，强制刷新当前路由。

- 通过`/filters` 返回Zuul中所有的过滤器详情。

#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">路由配置</font></center>
**1、配置指定微服务的访问路径**
```
zuul.routes.xx-service-name: /user/**
```
或
```
zuul:
   routes:
      user-route:     #user-route只是给路由起名字，可以随意起
         service-id: xx-service-name
         path: /user/**
```

**2、忽略指定微服务，只代理其他服务**
```
zuul.ignored-services: xx-service-name
```


**3、忽略所有微服务，只代理指定服务**
```
zuul:
  ignored-services: '*'
  routes:
      xx-service-name: /user/**
```

**4、指定path和url**
```
#将 /user/**映射到http//localhost:8080/,但是会破坏ribbon与hystrix
zuul:
   routes:
      user-route:
         path: /user/**
         url: http//localhost:8080/
```
```
#不破坏ribbon与hystrix
zuul:
   routes:
      user-route:    
         service-id: xx-service-name
         path: /user/**
         
ribbon:
   eureka:
      enabled: false
xx-service-name:
   ribbon:
      listOfServers: localhost:8000,localhost:8001
```

**5、可借助PatternServiceRouteMapper实现正则映射。**

**6、路由前缀**
- `zuul.strip-prefix`决定在访问时是否带上`zuul.prefix`。
- `zuul.routes.<service-name>.strip-prefix`决定在访问时是否带上`zuul.routes.<service-name>.path`

**7、忽略某些路径**
```
#在访问service-name这个服务时，会忽略/admin/路径。
zuul:
   ignoredPatterns: /**/admin/**
   routes:
      <service-name>： /user/**
```

**8、本地转发**
```
#在访问/path-a/**时，会转发到/path-b。
zuul:
   routes:
      <route-name>:
         path: /path-a/**
         url: forward:/path-b
```

#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">ZUUL日志级别</font></center>
```
logging.level.com.netflix
```


#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">ZUUL的安全与Header</font></center>
**1、为特定service设置敏感header列表**

```
zuul.routes.<service-name>.sentive-headers
```

**2、覆盖全局配置**
```
zuul.routes.*.sentive-headers
```

**3、忽略（丢弃）Header**
```
zuul.ignored-headers
```
ignored-headers默认为空，但是在在zuul配合spring security时默认值会是Expires等<font color = "#CD5555">Spring Security的Header</font>，如果需要使用它们，则可以设置`zuul.ignoreSecurityHeaders`为false。

#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">Zuul与文件上传</font></center>
- 文件小于1M。无须任何处理。
- 文件大于10M。在访问zuul时需要加上`zuul/`前缀。
- 对于超大文件，要将hystrix和ribbon的超时时间设置长一点。
```
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds
ribbon.ConnectTimeout.ReadTimeout
```

#### <center><font color = "#36648B">✎✎✎✎✎✎✎</font><br/><font color = "#36648B">Zuul的过滤器</font></center>
**1、过滤器类型**
- PRE
- ROUTING
- POST
- ERROR

**2、ZUUL请求的生命周期图**


**3、编写zuul过滤器**
- 继承抽象类`ZuulFilter`，实现其抽象方法。
- 将这个新编写的filter注册成一个bean。

**4、可使用过滤器做以下事情**
- 限流
- 安全认证（传播安全token）
- 灰度发布

**5、禁用过滤器**
```
zuul.<SimpleClassName>.<filterType>.disable = true
```

**6、注解**
`@EnableZuulProxy`是`@EnableZuulServer`的增强版。




