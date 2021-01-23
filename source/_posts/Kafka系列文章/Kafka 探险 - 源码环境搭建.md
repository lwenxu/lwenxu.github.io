---
title: Kafka 探险 - 源码环境搭建
date: 2021-01-14 17:20:28
tags: Kafka 探险
categories:
 - Kafka 探险
---

# Kafka 探险 - 源码环境搭建

> 这个 Kafka 的专题，我会从系统整体架构，设计到代码落地。和大家一起杠源码，学技巧，涨知识。希望大家持续关注一起见证成长！
> **我相信：技术的道路，十年如一日！十年磨一剑！**

**
**
## 前言


在阅读源码之前，首先要做的就是搭建一套源码调试环境，这是最基本的一步，不要觉得麻烦或者简单就不去做，也许你会像我一样搭源码的过程中得到一些教训和经验。同时在后面阅读源码的过程中，很多看不懂的地方 debug 一下也许就明朗了。


记录了搭建 Kafka 源码环境的简单过程，为大家提供一个步骤参考，同时记录搭建环境中可能会遇到的问题及解决方案。


这个环境搭建过程也会提到一个非常实用，并且很多人都不知道的源码 debug 技巧，对阅读源码和 debug 系统很有帮助哦！


## 源码下载


笔者下载的 Kafka 版本是 0.11.0.1 ，源码下载地址是 ：[https://kafka.apache.org/downloads](https://kafka.apache.org/downloads)
下载时选择，源码下载：


![image.png](https://cdn.nlark.com/yuque/0/2021/png/171275/1610295908149-3d876882-2f6c-4e16-acfe-402b957d4660.png#align=left&display=inline&height=423&margin=%5Bobject%20Object%5D&name=image.png&originHeight=846&originWidth=3026&size=470703&status=done&style=none&width=1513)




## 解压工程&安装插件


解压下载好的源码包，直接使用 Idea 打开项目即可。另外由于 Kafka 代码是 Scala 写的，所以需要安装一个 Scala 插件。


到 Idea 的插件市场下载 Scala 插件，这个插件不仅仅有语法提示而且可以帮你下载 Scala SDK，切换 SDK 非常方便，必装！


![image.png](https://cdn.nlark.com/yuque/0/2021/png/171275/1610296138821-fff657cf-7d63-4a49-8730-56489f5b38d5.png#align=left&display=inline&height=539&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1456&originWidth=2324&size=836993&status=done&style=none&width=860)


## 仓库初始化


养成一个好习惯，对于这种直接下载的源码包，先用 git 进行初始化，后续有什么改动也能够进行回溯，防止直接把源码改瓢了，之前做的注释也很难再拷贝出来。


```bash
git add . && git commit -m 'init'
```


## 构建项目


修改项目根目录下的 build.gradle ，将所有的 `mavenCentral()` 替换成  ` maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}` 加快 gradle 导入包的速度。
完事以后开始进行 Gradle 构建


![image.png](https://cdn.nlark.com/yuque/0/2021/png/171275/1610296394234-61bc8d50-cd52-4c92-9f59-c7cf8141b0ce.png#align=left&display=inline&height=323&margin=%5Bobject%20Object%5D&name=image.png&originHeight=646&originWidth=3486&size=739288&status=done&style=none&width=1743) 
构建完成后，所有的 Kafka 些模块会被自动导入，如下图是导入完成时的工程模块结构


![image.png](https://cdn.nlark.com/yuque/0/2021/png/171275/1610296724147-b6075fae-0c69-4293-a46c-86ae8af2a7fe.png#align=left&display=inline&height=616&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1232&originWidth=920&size=354544&status=done&style=none&width=460)


## 启动


找到 kafka.Kafka 这个类，然后运行 Main 方法，添加启动参数
```bash
vmOptions ->  -Dkafka.logs.dir=/Users/lwen/logs/kafka   # 这个目录需要修改一下，是 kafka 消息文件目录

program arguments ->  config/server.properties  # kafka 的配置文件路径
```


下图展示配置完毕时的参数


![image.png](https://cdn.nlark.com/yuque/0/2021/png/171275/1610383541230-9bf79e7f-8ea9-414b-ba1b-ffa27573b721.png#align=left&display=inline&height=682&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1364&originWidth=2142&size=674967&status=done&style=none&width=1071)


我遇到了很多编译警告⚠️，不过只要还能继续编译就不用 care。
令人悲伤的是程序启动不起来，main 方法直接退出了，没有任何的提示。


![image.png](https://cdn.nlark.com/yuque/0/2021/png/171275/1610297238930-cad4aec6-ae09-439e-a8f4-9a3ecc2dbf30.png#align=left&display=inline&height=364&margin=%5Bobject%20Object%5D&name=image.png&originHeight=728&originWidth=3662&size=1228401&status=done&style=none&width=1831)


## 排查问题


遇到上面那个问题后，找不到任何的日志看出是因为什么导致的，当时看网上的教程是把 log4j 配置文件拷贝到 kafka 目录，日志就能生效，但是我尝试过了也不 OK。


所以我就开始 debug，找出为什么这个地方会出现 exit with 1 ，这里介绍一个调试源码的技巧：我们看到代码是遇到了异常才退出的，但是我们没有异常堆栈和错误提示，可以肯定的是程序肯定遇到异常了。


所以我们在 Idea 中，断点所有会发生异常的位置具体操作：


cmd+shift+f8 打开断点窗口


![image.png](https://cdn.nlark.com/yuque/0/2021/png/171275/1610885167175-17d6d533-5b46-496f-9f25-8b1d6da3921c.png#align=left&display=inline&height=623&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1246&originWidth=1406&size=454735&status=done&style=none&width=703)


勾选上 Any Exception ，并在 Catch Class Filter 中去掉 ClassNotFoundException 因为在程序运行的时候会有双亲委派的类加载过程，肯定会触发 ClassNotFoundException 。这样配置以后，程序抛出任何非 ClassNotFoundException 的位置都会停下来

![image.png](https://cdn.nlark.com/yuque/0/2021/png/171275/1610885340427-2fb68d6f-3280-4733-8e58-2322de105227.png#align=left&display=inline&height=623&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1246&originWidth=1702&size=504033&status=done&style=none&width=851)


以 debug 的方式启动程序，最后我发现程序在  initZk() 的地方异常了，那就很清晰了，zk 配置问题


![image.png](https://cdn.nlark.com/yuque/0/2021/png/171275/1610885430723-f3324af0-23e5-4371-b71f-ce5ce921329a.png#align=left&display=inline&height=350&margin=%5Bobject%20Object%5D&name=image.png&originHeight=700&originWidth=2022&size=435325&status=done&style=none&width=1011)


这个有点坑！主要是因为没有开启日志，所以一行日志没有直接抛出异常结束进程了，后来我也找到打印日志的方法，按照我上面的启动参数配置就可以。


所以原因是没有启动 zk，那么下一步就是安装 zk。


## 安装 ZK


```bash
brew install  zookeeper
```
 安装完了以后启动 zk ，我采用的是 后台运行的方式：
```bash
brew services start zookeeper
```
      当然也可以直接前台启动，看到日志输出:
```bash
zkServer start
```
## 再次启动


![image.png](https://cdn.nlark.com/yuque/0/2021/png/171275/1610383610463-3abb66b5-9aee-4181-8ca5-2eb77ce68fb7.png#align=left&display=inline&height=465&margin=%5Bobject%20Object%5D&name=image.png&originHeight=930&originWidth=2752&size=1154182&status=done&style=none&width=1376)


## 唠叨


本来以为搭建源码挺简单的，但是还是自己把自己坑了一把。日志没配，zk 没配。不过好在这个过程中，就算没有任何日志和堆栈也能分析到问题的原因，也是调试的一个小技巧，相当实用。


下篇文章要开始分析 Producer 的架构啦，首先我们会尝试自己实现一个 Producer ，然后再和官方的对比，看看优秀的代码在设计中更关注的点以及是如何实现的。


另外：大家也可以关注下我的微信公众号哦~ 技术分享和个人思考都会第一时间同步！


![image.png](https://cdn.nlark.com/yuque/0/2021/png/171275/1610289331606-52e03a47-f431-4b20-8b57-6a4cc0f70345.png#align=left&display=inline&height=430&margin=%5Bobject%20Object%5D&name=image.png&originHeight=430&originWidth=430&size=69429&status=done&style=none&width=430)
