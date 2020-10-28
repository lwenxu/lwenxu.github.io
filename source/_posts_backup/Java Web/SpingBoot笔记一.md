---
title: SpingBoot笔记一
date: 2017-10-23 22:09:38
tags: Spring
categories:
	-SpringBoot

---

## 一.目录配置

1、Application.java 建议放到跟目录下面,主要用于做一些框架配置

2、domain目录主要用于实体（Entity）与数据访问层（Repository）

3、service 层主要是业务类代码

4、controller 负责页面访问控制

##二.pom.xml 配置

**引入web模块**

1、pom.xml中添加支持web的模块：

```xml
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
 </dependency>
```

pom.xml文件中默认有两个模块：

spring-boot-starter：核心模块，包括自动配置支持、日志和YAML；

spring-boot-starter-test：测试模块，包括JUnit、Hamcrest、Mockito。

##三.编写controller内容

```java
@RestController
public class HelloWorldController {
    @RequestMapping("/hello")
    public String index() {
        return "Hello World";
    }
}
```

@RestController的意思就是controller里面的方法都以json格式输出，不用再写什么jackjson配置的了！

## 四.单元测试

***

打开的src/test/下的测试入口，编写简单的http请求来测试；使用mockmvc进行，利用MockMvcResultHandlers.print()打印出执行结果。

