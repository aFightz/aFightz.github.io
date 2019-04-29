---
title: 02 | 微服务构建
date: 2019-04-16 11:28:12
tags: [Spring Cloud]
categories :
- 学习笔记
- Java
- Spring
- SpringCloud微服务实战
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">SpringBoot的Maven配置</font></center>

构建SpringBoot工程，只需要引入`spring-boot-starter-parent`与若干`Starter POM`，并且只需要指定`spring-boot-starter-parent`的版本，而不需要指定`Starter POM`的版本。因为`spring-boot-starter-parent`已经定义了<font color = "#CD5555">**Spring Boot版本的基础依赖**</font>以及一些<font color = "#CD5555">**默认配置**</font>内容（如配置文件application.properties的位置等）。
`Starter POM`采用**spring-boot-starter-***的命名方式。




#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">用MockMvc进行单元测试</font></center>
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringJUnitWebConfig(classes = Application.class)
public class HelloControllerTest {
    private MockMvc mockMvc;

    @Before
    public void setUp(){
        mockMvc = MockMvcBuilders.standaloneSetup(new HelloController()).build();
    }
    
    @Test
    public void hello() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
        .andExpect(status().isOk()).andExpect(content().string(("Hello World")));
    }
}
```

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">配置文件详解</font></center>

**1、配置文件**
SpringBoot的默认配置文件路径：`src/main/resources/application.properties`

**2、YAML配置文件**
```
environments:
    dev:
        url : http://dev.bar.com
        name : Developer Setup
    prod:
        url : http://foo.bar.com
        name : My Cool App
```

YAML配置文件有以下<font color = "#CD5555">**优缺点**</font>：
- 优点：将属性加载到内存中保存的时候是`有序`的。
- 缺点：无法通过`@PropertySource`注解来加载配置。


#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">属性详解</font></center>

**1、属性说明**
指定端口：`server.port`
指定应用名：`spring.application.name`
定义多个不同的环境配置：`spring.profiles.active`

**2、用`@Value`加载属性**
`@Value`的两种加载配置方式
- PlaceHolder方式：`${...}`
- SpEL表达式：`#{...}`

**3、参数引用**
```
book.name = SpringCloud
book.author = hjl
book.desc = ${book.author} is writing《${book.name}》
```

**4、用`${random}`设置随机数**
```
#随机字符串
com.didispace.blog.value=${random.value}
#随机int
com.didispace.blog.value=${random.int}
#随机long
com.didispace.blog.value=${random.long}
#10以内的随机数
com.didispace.blog.value=${random.int(10)}
#10~20的随机数
com.didispace.blog.value=${random.int[10,20]}
```
可使用随机数生成随机端口避免端口冲突。

**5、使用命令行参数指定属性**
`java -jar xxx.jar --server.port=8888`


#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">多环境配置</font></center>
多环境配置的文件名需要满足`application-{profile}.properties`的格式。可以通过`spring.profiles.active`参数指定不同的配置文件。若不指定，则默认的profile是<font color = "#CD5555">**dev**</font>。

**多环境配置思路**
- 将<font color = "#CD5555">**通用的配置**</font>内容放到`application.properties`。
- 将<font color = "#CD5555">**特定的内容**</font>放到`application-{profile}.properties`。
- 通过`spring.profiles.active`加载不同环境的配置文件。

#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">配置文件加载顺序</font></center>
```
命令行参数
SPRING_APPLICATION_JSON中的属性
来自java:comp/env的JNDI属性
Java系统属性（System.getProperties()）
操作系统环境变量
RandomValuePropertySource配置的random.*属性值
jar包外部的application-{profile}.properties或application.yml（带spring.profile）配置文件
jar包内部的application-{profile}.properties或application.ym（带spring.profile）配置文件
jar包外部的application.properties或application.yml（不带spring.profile）配置文件
jar包内部的application.properties或application.yml（不带spring.profile）配置文件
@Configuration注解类上的@PropertySource
通过SpringApplication.setDefaultProperties指定的默认属性
```

#### <center><font color = "#36648B">✎✎✎✎✎✎✎</font><br/><font color = "#36648B">监控与管理</font></center>

首先需要引入引入<font color = "#CD5555">**actuator**</font>的starter pom。
默认情况下，只有`/health`和`/info`这两个端点通过http暴露了出来。所以当我们在访问`http://localhost:8080/actuator`这个接口时，会得到如下json：

![](SpringCloud微服务实战_02_微服务构建\default.png)

以下是Spring官网给出的所有端点描述：[<font color = "#36648B">**spring官网关于Endpoints的资料**</font>][spring_url]

[spring_url]: https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html	"[spring_url]:"

![](SpringCloud微服务实战_02_微服务构建\list.png)

可以通过这个设置`management.endpoints.web.exposure.include = *`，将其他端口也通过http暴露出来。
`/shutdown`这个端点默认是不开启的，还需要再加上配置`management.endpoint.shutdown.enabled=true`开启这个端点。
当我们访问`/health`这个端点的时候，加上`management.endpoint.health.show-details=always`会将更详细的检验器信息打印出来。Spring内置了许多校验器，具体的可以去[<font color = "#36648B">**spring官网**</font>][spring_url]查一下。我们也可以实现自己的校验器，如下

```java
@Component
public class MyServiceHealthIndicator implements HealthIndicator {
    private boolean flag = false;
    @Override
    public Health health() {
        if(flag){
            //模拟校验成功
            return Health.up().build();
        }
        //模拟校验失败
        return Health.down().withDetail("status" , "500").withDetail("message" , "服务器错误").build();
    }
}
```