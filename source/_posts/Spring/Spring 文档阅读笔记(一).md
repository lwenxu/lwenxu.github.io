---
title: Spring 文档阅读笔记（一）
date: 2020-01-30 15:02:37
tags: [Spring 源码 , Spring ,源码]
categories:
	- Spring 源码分析
	- Spring
---

# Spring 文档阅读笔记 一





## 1.2 概览

### 1.2.1 配置元数据

元数据配置就是能够让你明白如何进行应用开发，容器的实例化，配置，以及应用中对象的组织

传统配置元数据的方式就是使用 xml，在我们大部分的章节中会介绍一些关键的概念和 Ioc 容器的特性

当然目前不仅仅支持 xml ，目前推荐的是 Java 代码的方式以及注解方式：

- 2.5 开始支持注解驱动
- 3.0 开始建议使用 JavaConfig 方式配置 具体可以看看  [`@Configuration`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html), [`@Bean`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html), [`@Import`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html), and [`@DependsOn`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html) 注解.



## 2.3 使用场景

### 2.3.2 日志

spring的强依赖是是 JCL (Jakarta Commons Logging API ) 以及他的实现 commons-logging (JCL的标准实现)。更加确切的说他是在spring-core 中被依赖的。

现在选择的话 spring 并不会使用 JCL 而是  slf4j 有两种方案关闭 common-logging：

1. 从`spring-core`模块排除依赖（因为它是唯一的显式依赖）`commons-logging`的模块

2. 或者依赖于一个特定的`commons-logging`依赖，用一个空jar替换这个依赖（更多细节可以在[SLF4J FAQ](https://link.jianshu.com/?t=http://slf4j.org/faq.html#excludingJCL)中找到）。

排除包之后还需要将 jcl 的调用连接到 slf4j 上，并且添加 slf4j 的具体实现：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.0.0.BUILD-SNAPSHOT</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>1.5.8</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.5.8</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.5.8</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.14</version>
    </dependency>
</dependencies>
```