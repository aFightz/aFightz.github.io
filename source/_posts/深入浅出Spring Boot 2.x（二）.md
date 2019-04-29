---
title: 深入浅出Spring Boot 2.x（二）
date: 2018-11-28 01:28:46
tags: [Spring Boot]
categories :
- 学习笔记
- Java
- Spring
- 深入浅出Spring Boot 2.x
---

> **概要：**
>
> - SpringBoot参数的相关说明
>
> - 与配置有关的注解的关系说明

SpringBoot的默认配置文件是classpath下的`application.properties`。

如果要改服务器端口，只需要在`application.properties`加上以下配置即可。

```java
server.port = 8090
```



### 加载参数的优先级顺序

- 命令行参数
- 来自java:comp/env的JNDI属性
- Java系统属性（System.getProperties()）
- 操作系统环境变量
- RandomValuePropertySource配置的random.*属性值
- jar包外部的application-{profile}.properties或application.yml（带spring.profile）配置文件
- jar包内部的application-{profile}.properties或application.ym（带spring.profile）配置文件
- jar包外部的application.properties或application.yml（不带spring.profile）配置文件
- jar包内部的application.properties或application.ym（不带spring.profile）配置文件
- @Configuration注解类上的@PropertySource
- 通过SpringApplication.setDefaultProperties指定的默认属性



### 常见配置注解的关系

@SpringBootApplication包含了以下三个注解：

- @SpringBootConfiguration
- @EnableAutoConfiguration
- @ComponentScan

而@SpringBootConfiguration又包含了@Configuration注解。