---
title: SpringBoot 笔记(七):搜索
date: 2018-05-21 14:01:28
tags: SpringBoot
categories:
 - SpringBoot
---
# SpringBoot 笔记 (七):搜索

这里我们的搜索就是用 ElasticSearch 这个工具，这个其实是在 Lauce 的基础上构建的一个搜索引擎。

# 1. ES入门

## 1.docker安装

```bash
docker pull registry.docker-cn.com/library/elasticsearch
docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 --name ES_dev
```

这里指定了es的初始的java堆大小和最大堆大小，否则就是默认的 2G 。

## 2.基本概念

ES 是一个面向文档的数据库，他的文档的格式就是JSON的格式。

### 1.索引

这个索引指的是动词，也就是我们把数据存放到ES中的一个过程。

### 2.索引

这个索引是名词，指的是我们的ES中的一个数据库。

### 3.类型

这个就相当与我们的一个表

### 4.文档

就是我们的数据记录。

简单的可以用一张表来表示：

![image](https://user-images.githubusercontent.com/22151420/40353449-1ffca3e6-5de4-11e8-8b83-2b396388fc6b.png)

