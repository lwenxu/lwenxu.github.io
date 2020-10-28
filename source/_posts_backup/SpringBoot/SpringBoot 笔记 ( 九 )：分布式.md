---
title: SpringBoot 笔记(九):分布式
date: 2018-05-23 22:01:28
tags: SpringBoot
categories:
 - SpringBoot
---
# SpringBoot 笔记(九):分布式

我们可以使用 SpringBoot构建分布式应用，也就是我们在开发的时候可以进行多个模块的拆分，每一个功能做一个模块，然后我们使用一些分布式的框架，进行远程调用，所谓的远程调用就是RPC调用，而不是以前的WebService这个形式的.。其实这种分布式的RPC框架有很多，除了我们创常见的Doubbo还有就是我们Spring 项目自带的 SpringCloud 也是类似的东西。<!--more-->

器基本理念就是：我们一个模块写好我们的业务逻辑然后，把我们写好的逻辑当作服务发布出去，然后发布出去的具体位置我们称之为注册中心，现在注册中心主要是通过 zookeeper 了来完成的。

## 1.zookeeper的安装

```shell
docker pull registry.docker-cn.com/library/zookeeper
docker run -d -p 2181:2181 --name zk-01 bf5cbc9d5cac 
```

## 2.添加依赖

```xml
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.4.6</version>
        <type>pom</type>
    </dependency>

    <dependency>
        <groupId>com.alibaba.boot</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>0.1.0</version>
    </dependency>
    
    <dependency>
        <groupId>com.github.sgroschupf</groupId>
        <artifactId>zkclient</artifactId>
        <version>0.1</v	ersion>
    </dependency>
```

## 3.配置文件

```yaml
dubbo:
  application:
    name: producer
  registry:
    address: zookeeper://219.245.18.94:2181
  scan:
    base-packages: com.example.producer.service
```

## 4.代码

### 开启分布式注解

```java
@EnableDubbo
@SpringBootApplication
public class ProducerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProducerApplication.class, args);
    }
}
```

### 发布服务

``` java
package com.example.service;

public interface TickService {
    public String sell();
}
```

```java
package com.example.service;

import com.alibaba.dubbo.config.annotation.Service;
import org.springframework.stereotype.Component;

@Component
@Service
public class TickServiceImpl implements TickService {
    @Override
    public String sell() {
        return "hello dubbo";
    }
}
```

注意使用的注解，发布服务的时候使用的是阿里的注解。

### 消费服务

```java
package com.example.service;

public interface TickService {
    public String sell();
}
```

必须有一样的接口才行。

``` java
package com.example.service;

import com.alibaba.dubbo.config.annotation.Reference;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    @Reference
    TickService tickService;

    public void getTick(){
        System.out.println(tickService.sell());
    }
}
```

使用    @Reference 注解注入远程的 服务。

