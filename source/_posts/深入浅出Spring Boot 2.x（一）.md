---
title: 深入浅出Spring Boot 2.x（一）
date: 2018-11-26 13:45:46
tags: [Spring Boot]
categories :
- 学习笔记
- Spring
- 深入浅出Spring Boot 2.x
---

> **概要**：SpringBoot的快速搭建运行Demo。
>

Servlet3.0后，Web容器可以脱离web.xml的部署，可以完全基于注解开发。

### Spring Boot运行Demo

- maven结构



```
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.1.0.RELEASE</version>
</parent>

<dependencies>

  <!--测试包-->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>

  <!--web开发包，包含spring mvc，且内嵌tomcat-->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <!--aop包-->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
  </dependency>

</dependencies>

<!--引入插件-->
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```





- java代码

```java
@Controller
@EnableAutoConfiguration
public class Application {

    @RequestMapping("/test")
    @ResponseBody
    public Map<String,String> test(){
        Map<String,String> map = new HashMap<String,String>();
        map.put("name","joke");
        return map;

    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }

}
```

之后访问

```
http://localhost:8080/test
```

就可以看到返回参数了，非常之简单。