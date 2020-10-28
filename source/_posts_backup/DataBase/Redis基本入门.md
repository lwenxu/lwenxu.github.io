---
title: Redis基本入门
date: 2017-07-28 16:56:49
tags: Redis NoSql 数据库
categories:
  -数据库
---
## 1.Redis简介
&nbsp;&nbsp;&nbsp;&nbsp;Redis 是一种基于内存亦可持久化的日志型，Key-Value 数据库。可持久在于他的部分数据是存放在内存上，而当数据库重启以后他的数据不会立刻丢失，而是会存放在磁盘上，通过日志去加载磁盘上的数据文件。所以说它不仅仅是一种内存型键值数据库。  
&nbsp;&nbsp;&nbsp;&nbsp;另外 Redis 所支持的数据结构也非常丰富，不仅仅就是一种简单的 NoSql ，而是被称作为数据结构服务器。他所支持的数据结构有：字符串（string），哈希（Map），列表（List），集合（Set），有序集合（ZSet），BitMap，HyperLogLog（超小内存唯一值计数），GEO（地理信息位置）。以上所说的均是值所支持的数据结构，键必然为字符串。
<!--more-->
## 2.Redis安装
&nbsp;&nbsp;&nbsp;&nbsp; Redis 最初是由标准 C 写的，但是后来社区非常的活跃，所以现在什么版本的 Redis 都是有的，所以说这个数据库是可以跑在任何平台的，这里主要就说 Windows 下的安装， Linux 与 Mac 下安装很简单，只需要一个命令就能搞定。
&nbsp;&nbsp;&nbsp;&nbsp;下载好Redis的二进制文件后，使用 cmd 切到 Redis 安装目录下，然后运行
```bash
redis-server.exe redis.conf
```
就能跑起一个 Redis 的服务端程序，然后不要关闭这个程序。另开一个 cmd 依然切换到安装目录执行
```bash
redis-cli.exe -h 127.0.0.1 -p 6379
```
这样就开启了一个客户端。
## 3.Redis的配置

我们有三种启动 redis 的方式：

- 最简启动，无需配置文件 `redis-server`
- 加载配置文件 `redis-server configpath`
- 动态参数的配置

客户端的连接方式：

`redis-cli -h ip -p port`

&nbsp;&nbsp;&nbsp;&nbsp;Redis 可以使用客户端进行配置也可以使用配置文件，Redis 的配置文件位于 Redis 安装目录下，文件名为 redis.conf。
使用客户端基本格式如下：
```bash
CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE
```
这样的好处在于配置起来方便，但是缺点显然就是服务器重启，配置将不再生效，也就是说现在的配置放在了内存中，然后需要重启还生效的东西必须放在永久存储介质上，一般来说就是磁盘。这个规则不仅仅适用于 Redis ，对于 Linux 以及其他数据库也是如此。  
&nbsp;&nbsp;&nbsp;&nbsp;接下来主要说一下配置文件里面的重要选项：
```bash
1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程

  daemonize no

2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定

  pidfile /var/run/redis.pid

3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字

  port 6379

4. 绑定的主机地址

  bind 127.0.0.1

5.当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能

  timeout 300

6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose

  loglevel verbose

7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null

  logfile stdout/filename

8. 设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id

  databases 16

9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合

  save <seconds> <changes>

  Redis默认配置文件中提供了三个条件：

  save 900 1

  save 300 10

  save 60 10000

  分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。



10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大

  rdbcompression yes

11. 指定本地数据库文件名，默认值为dump.rdb

  dbfilename dump.rdb

12. 指定本地数据库,以及存放目录

  dir ./

13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步

  slaveof <masterip> <masterport>

14. 当master服务设置了密码保护时，slav服务连接master的密码

  masterauth <master-password>

15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭

  requirepass foobared

16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息

  maxclients 128

17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区

  maxmemory <bytes>

18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no

  appendonly no

19. 指定更新日志文件名，默认为appendonly.aof

   appendfilename appendonly.aof

20. 指定更新日志条件，共有3个可选值：     no：表示等操作系统进行数据缓存同步到磁盘（快）     always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）     everysec：表示每秒同步一次（折衷，默认值）

  appendfsync everysec



21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）

   vm-enabled no

22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享

   vm-swap-file /tmp/redis.swap

23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0

   vm-max-memory 0

24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值

   vm-page-size 32

25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。

   vm-pages 134217728

26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4

   vm-max-threads 4

27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启

  glueoutputbuf yes

28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法

  hash-max-zipmap-entries 64

  hash-max-zipmap-value 512

29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）

  activerehashing yes

30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件

  include /path/to/local.conf
```

## 4. 通用命令

1. keys 所有的键 O(n)
2. dbsize 复杂度为O(1)
3. exists key
4. del key
5. expire key second
6. type key
7. ttl/persist key 存活时间，让他一直存活

### 时间复杂度

![image](https://user-images.githubusercontent.com/22151420/39404881-00ca9fa0-4bce-11e8-90e1-85596195fa3a.png)





## 5. 字符串

最大是512M，主要用于缓存、计数器、分布式锁。

### 1.API

- get/set/del
- Incr/decr/incrby/decrby
- Set/setnx/set .. xx 
- Mget/mset … 批量操作原子的
- getset key newvalue
- Append key value
- Strlen key

## 6. Hash

### 1.API

- Hget/hset/hdel
- Hexits/hlen field 数
- hmget/hmset
- 同上

## 7. List

- lpush/lpop/rpush/rpop

- Linsert key before/after 

- lrem key count value  根据count值，从列表中删除所有value相等的项

  (1) count >0，从左到右，删除最多count个value相等的项
  (2) count<0,从右到左，删除最多Math.abs(count)个value相等的项
  (3) count=0,删除所有value相等的项
  lrem listkey 0 a

- Itrim key start end   按照索引范围修剪列表,删除不在范围之内的

- lrange key start end (包含end)  获取列表指定索引|范围所有item

- lindex key index 获取列表指定位置的值

- len key 长度

- lset key index val

- blpop/brpop key timeout 阻塞直到部位空返回

## 8. set

- sadd/sdel/srem
- scrad/sismember/srandmember/spop/smembers


- sdiff user:1:follow user:2:follow = music his #差集
- sinter user:1:follow user:2:follow = it sports #交集
- sunion user:1:follow user:2:follow = it music his sports news ent #并集
- sdifflsinterlsuion + store destkey..#将差集、交集、并集结果保存在destkey中


- SADD= Tagging
- SPOP/SRANDMEMBER = Random item
- SADD + SINTER = Social Graph

## 9. zset

- zadd key score element(可以是多对)添加score和element
- zrem key element
- zsrore key ele
- zincrby key increscore element增加或减少元素的分数
- Zrank 排名
- zrange key start end
- zrangebyscore 
- zrembyscore
- zrevarank 反向的排序

## 10. 发布订阅

## 11.RDB

1. RDB是Redis内存到硬盘的快照，用于持久化。


2. save通常会阻塞Redis。


3. bgsave不会阻塞Redis,但是会fork新进程。


4. save自动配置满足任一就会被执行。
5. 有些触发机制不容忽视

