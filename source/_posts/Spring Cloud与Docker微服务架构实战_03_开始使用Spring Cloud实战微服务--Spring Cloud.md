---
title: 03 | 开始使用Spring Cloud实战微服务
date: 2019-06-04 10:57:39
tags: [Spring Cloud]
categories :
- 学习笔记
- Java
- Spring
- Spring Cloud与Docker微服务架构实战
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">Maven项目转Gradle项目</font></center>
```
gradle init --type pom
```

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">注解</font></center>
- `@GetMapping`相当于`@RequestMapping(method = RequestMethod.GET)`。类似的还有`@PutMapping`、`@DeleteMapping`等。
- `@SpringBootApplication`整合了
   - `@Configuration`
   - `@EnableAutoConfiguration`
   - `@ComponentScan`
   

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">YML</font></center>
yml有严格的缩进、冒号后的空格不能少。



#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">Actuator</font></center>
要让端点显示详情，可以用以下三种方式之一实现：
- 添加`spring-boot-starter-security`这个starter pom。
- 设置`management.security.enabled=false`。
- 设置`management.endpoint.health.show-details=always`。

可以用`info.*`属性定义info端点公开的数据。