---
title: 深入浅出Spring Boot 2.x（三）
date: 2018-12-02 23:58:46
tags: [Spring Boot]
categories :
- 学习笔记
- Java
- Spring
- 深入浅出Spring Boot 2.x
---

### @Configuration

```java
@Configuration
public class AppConfig {

    @Bean(name = "user")
    public User initUser(){
        User user = new User();
        user.setName("hjl");
        return user;
    }
}
```

@Configuration代表这是一个Java配置文件。
如果没有配置@Bean的name属性，那么方法名"initUser"将作为Bean的名称保存到IOC容器。
**UnitTest**

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    User user = ctx.getBean(User.class);
    System.out.println(user.getName());
}
```



### @ComponentScan

```java
@Configuration
@ComponentScan
public class AppConfig {
 
}
```

默认情况下只会扫描AppConfig所在的当前包和子包。
也可以自定义扫描的范围：

```
@Configuration("com.hjl.demo.springboot.*")

@Configuration(basePackges = {"com.hjl.demo.springboot.pojo"})

@Configuration(basePackageClasses = {User.class})
```



