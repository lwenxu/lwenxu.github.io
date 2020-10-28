---
title: Flume 入门
date: 2018-05-18 16:01:28
tags: Spark
categories:
 - Spark
---

# 1. 基本架构

### 1. 结构

source	源数据：目录/

channel	数据管道

sink		数据输出

### 2. 几种结构形式

1. 单个的Flume
2. 多个Flume串行工作，前一个Flume的sink作为后一个的source
3. 多个Flume并行工作然后最后用一个Flume进行合并这行并行收集的数据
4. 多个Flume并行工作，分别传送到不同的目的地

# 2. 安装

## 1. 安装JDK

解压到 ~/app
将java配置系统环境变量中：~/.bash_profile
export JAVA_HOME=/home/hadoop/app/jdk1.8.0_144
export PATH=$JAVA_HOME/bin:$PATH
source下让其配置生效
检测：java -version

## 2. 安装Flume

### 1.  安装

export FLUME_HOME=/home/hadoop/app/apache-flume-1.6.0-cdh5.7.0-b

export PATH=$FLUME_HOME/bin:$PATH
source下让其配置生效

### 2.  配置

flume-env.sh的配置：export JAVA_HOME=/home/hadoop/app/jdk1.8.0_144

# 3. 使用Flume

使用Flume的关键其实就是书写配置文件，然后启动不同的配置文件的进程就是不同的任务。

## 1. 配置

- 配置Source
- 配置Channel
- 配置Sink
- 把上面的组件串联起来

给一个简单的Conf的例子，并做一些注释。

```properties
#a1:agent名称 r1:source的名称 k1:sink的名称 c1:channel的名称
#Name the components on this agent
al.sources=r1
al.sinks=k1
al.channels=c1
#Describe/configure the source
al.sources.rl.type=netcat
al.sources.r1.bind=localhost
al.sources.r1.port =44444
#Describe the sink
al.sinks.k1.type=logger
# Use a channel which buffers events in memory
al.channels.c1.type=memory
#Bind the source and sink to the channel
al.sources.r1.channel.=c1
al.sinks.k1.channel=c1

```

## 2. 启动

```bash
flume-ng agent \
--name a1 \
-c $FLUME_HOME/conf \
-f $FLUME_HOME/conf/example.conf \
-Dflume.root.logger=INFO,console
```

注意 --name 选项指定的名字必须要和配置文件里面的agent名字一致才行，否则会提示找不到对应的配置。

启动以后使用 telnet 127.0.0.1 4444 访问，然后输入文字flume就可以收集到数据。具体数据如下：

```json
2018-05-18 17:01:21,693 Event: { headers:{} body: 68 65 6C 6C 6F 0D   hello. }              
2018-05-18 17:01:26,912 Event: { headers:{} body: 77 6F 72 6C 64 0D  world. }                 
```

可以看到他每一条消息都是一个Event 也就是说在Flume中每一个Event都是一个基本的传送单元。

Event = header + body

# 3.  实战

## 1. 从文件中读取数据

````properties
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /root/data.log
a1.sources.r1.shell = /bin/bash -c

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
````

## 2.  A机向B机传输采集的数据

首先配置A机 ，我们采用的是 exec 监控一个文件，然后使用 memory channel 输出到 avro ，所以有配置文件 exec-memcory-avro.conf

```properties
exec-memcory-avro.sources = exec-source
exec-memcory-avro.sinks = avro-sink
exec-memcory-avro.channels = memory-channel

exec-memcory-avro.sources.exec-source.type = exec
exec-memcory-avro.sources.exec-source.command = tail -F /root/data.log
exec-memcory-avro.sources.exec-source.shell = /bin/bash -c

exec-memcory-avro.sinks.avro-sink.type = avro
exec-memcory-avro.sinks.avro-sink.hostname = master
exec-memcory-avro.sinks.avro-sink.port = 4444

exec-memcory-avro.channels.memory-channel.type = memory
exec-memcory-avro.channels.memory-channel.capacity = 1000
exec-memcory-avro.channels.memory-channel.transactionCapacity = 100


exec-memcory-avro.sources.exec-source.channels = memory-channel
exec-memcory-avro.sinks.avro-sink.channel = memory-channel
```

执行如下的命令可以启动，但是不要现在启动否则会报错

```bash
flume-ng agent \
--name exec-memcory-avro \
-c $FLUME_HOME/conf \
-f $FLUME_HOME/conf/exec-memcory-avro.conf \
-Dflume.root.logger=INFO,console
```



那么B机就需要使用avro source来接受了，然后使用memory做channel，最后使用logger做输出。建立配置文件avro-memory-logger.conf

```properties
avro-memory-logger.sources = avro-source
avro-memory-logger.sinks = logger-sink
avro-memory-logger.channels = memory-channel

avro-memory-logger.sources.avro-source.type = avro
avro-memory-logger.sources.avro-source.bind = master
avro-memory-logger.sources.avro-source.port = 4444

avro-memory-logger.sinks.logger-sink.type = logger

avro-memory-logger.channels.memory-channel.type = memory
avro-memory-logger.channels.memory-channel.capacity = 1000
avro-memory-logger.channels.memory-channel.transactionCapacity = 100


avro-memory-logger.sources.avro-source.channels = memory-channel
avro-memory-logger.sinks.logger-sink.channel = memory-channel
```

启动

```bash
flume-ng agent \
--name avro-memory-logger \
-c $FLUME_HOME/conf \
-f $FLUME_HOME/conf/avro-memory-logger.conf \
-Dflume.root.logger=INFO,console
```

我们要先启动后面的那个，也就是B机，不然的话我们先启动A机就会报错。

