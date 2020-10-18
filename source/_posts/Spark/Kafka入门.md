---
title: Kafka 入门
date: 2018-05-018 19:01:28
tags: Spark
categories:
 - Spark
---

#  1. Kafka架构

1. 生产者（producer）：生产资源的
2. 消费者（consumer）：消费资源
3. broker（缓冲）：中间缓冲器
4. topic（标记）：谁来消费

# 2. 配置Kafka

## 1. 配置ZK

1. 解压
2. 配置ZK_HOME
3. 配置PATH
4. 修改 zk.cfg 修改存储位置
5. zkServer.sh 启动

## 2. 配置Kafka

1. 配置HOME以及PATH
2. 修改 server.porperties 注意以下几个条目

```properties
$KAFKA_HOME/config/server.properties
broker.id=0
listeners
host.name
1og.dirs
zookeeper.connect
```

3. 启动Kafka

```bash
kafka-server-start.sh $KAFKA_HOME/config/server.properties
```

4. 生产消息

```bash
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic hello
```

5. 启动消费者

```bash
kafka-console-consumer.sh --zookeeper localhost:2181 --topic hello --from-beginning
```

