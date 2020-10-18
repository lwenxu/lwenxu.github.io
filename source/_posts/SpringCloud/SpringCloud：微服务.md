---
title: SpringCloud：微服务
date: 2018-05-25 20:01:28
tags: SpringCloud
categories:
 - SpringCloud
---

## 1.微服务

微服务其实就是我们的以前的整个应用拆分的一个个小的应用服务，也就是我们的一个个模块。每一个微服务就是一个进程，运行在一个独立的进程之上，然后通过网络进行通讯互联。

## 2.微服务架构

微服务架构是一种架构模式，他是把传统的单体应用(All in one)，拆分成多个项目独立的微服务，然后每一个为服务都是一个独立进程，拥有自己的独立的数据库。然后各个微服务的整合形成整个的微服务架构。

## 3.微服务的优缺点

### 1.优点：

- 每一个服务就是一个聚焦一个功能
- 单个服务开发简单
- 服务解耦
- 小而精

### 2.缺点：

- 部署困难，部署依赖
- 排查困难
- 服务通讯的成本
- 数据的一致性
- 集成测试
- 性能监控

## 4.微服务技术栈

| 微服务条目                               | 技术                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| 服务开发                                 | Springboot、Spring、SpringMVC                                |
| 服务配置与管理                           | Netflix公司的Archaius、阿里的Diamond等                       |
| 服务注册与发现                           | Eureka、Consul、Zookeeper等                                  |
| 服务调用                                 | Rest、RPC、gRPC                                              |
| 服务熔断器                               | Hystrix、Envoy等                                             |
| 负载均衡                                 | Ribbon、Nginx等                                              |
| 服务接口调用（客户端调用服务的简化工具） | Feign等                                                      |
| 消息队列                                 | Kafka、RabbitMQ、ActiveMQ等                                  |
| 服务配置中心管理                         | SpringCloudConfig、Che等                                     |
| 服务路由（API网关)                       | Zuul                                                         |
| 服务监控                                 | Zabbix、Nagios、Metrics、Spectator等                         |
| 全链路追踪                               | Zipkin,Brave、Dapper等                                       |
| 服务部署                                 | Docker、OpenStack、Kubernetes等                              |
| 数据流操作开发包                         | SpringCloud Stream（封装与Redis,Rabbit、Kafka等发送接收消息） |
| 事件消息总线                             | Spring Cloud Bus                                             |



## 5.SpringCloud

这里我们的SpringCloud就是一个微服务架构的框架的具体实现了。同样的还有很多其他的微服务架构，比如现在非常出名的Dubbo 以及京东的 JSF 新浪的 Motai