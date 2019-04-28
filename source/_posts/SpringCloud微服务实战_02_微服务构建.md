---
title: 02 | 微服务构建
date: 2019-04-16 11:28:12
tags: [Spring Cloud]
categories :
- 学习笔记
- Java
- SpringCloud微服务实战
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">SpringBoot的Maven配置</font></center>

构建SpringBoot工程，只需要引入spring-boot-starter-parent与若干Starter POM，并且只需要指定spring-boot-starter-parent的版本，而不需要指定Starter POM的版本。
spring-boot-starter-parent定义了Spring Boot版本的基础依赖以及一些默认配置内容（如配置文件application.properties的位置等）。
Starter POM采用“spring-boot-starter-*”的命名方式。




#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">单元测试</font></center>

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

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">配置文件详解</font></center>

1、配置文件
SpringBoot的默认配置文件路径：src/main/resources/application.properties

2、YAML配置文件
environments:
    dev:
        url : http://dev.bar.com
        name : Developer Setup
    prod:
        url : http://foo.bar.com
        name : My Cool App

YAML配置文件有以下优缺点：
- 优点：将属性加载到内存中保存的时候是有序的。
- 缺点：无法通过@PropertySource注解来加载配置。


#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">属性详解</font></center>

1、属性说明
指定端口：server.port
指定应用名：spring.application.name
定义多个不同的环境配置：spring.profiles.active

2、用@Value加载属性
@Value的两种加载配置方式
- PlaceHolder方式：${...}
- SpEL表达式：#{...}

3、参数引用
book.name = SpringCloud
book.author = hjl
book.desc = ${book.author} is writing《${book.name}》

4、用${random}设置随机数
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

5、使用命令行参数指定属性
java -jar xxx.jar --server.port=8888


#### <center><font color = "#36648B">✎✎✎✎✎</font><br/><font color = "#36648B">多环境配置</font></center>
多环境配置的文件名需要满足application-{profile}.properties的格式。
通过spring.profiles.active参数指定不同的配置文件。
默认的profile是dev。

1、多环境配置思路
将通用的配置内容放到application.properties。
将特定的内容放到application-{profile}.properties。
通过spring.profiles.active加载不同环境的配置文件。

#### <center><font color = "#36648B">✎✎✎✎✎✎</font><br/><font color = "#36648B">配置文件加载顺序</font></center>
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

#### <center><font color = "#36648B">✎✎✎✎✎✎✎</font><br/><font color = "#36648B">监控与管理</font></center>

1、引入actuator
