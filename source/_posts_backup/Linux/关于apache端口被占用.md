---
title: 关于apache端口被占用
date: 2016-04-04 17:59:44
tags:
categories:
  - Web
---
![](http://p1.bpimg.com/567571/d081845d56c7f263.jpg)
# 正文
无论我们在安装单个的apache还是装集成环境xampp都是有可能遇到apache意外停止。查看错误日志会发现一般都是端口被占用，一般是被虚拟机占用了，这时一般有两种方法，改apache的端口号，或者把占用apache的端口号的那个服务干掉。
<!--more-->

## 方案一：改端口号
在apache的配置文件里找到httpd-ssl.conf文件然后ctrl+f查找port然后改成444或者其他的，只要没被占用的就可以，然后重启apache。MySQL出问题了也是这样的解决方案。

## 方案二：杀占用的服务

win+r打开运行，输入cmd打开命令提示符，输入

``` bash
netstat -ano
```
 
找到占用443端口的服务，找到他的pid ，然后输入

``` bash
kill /f /pid 端口号
```
  
就能结束此服务，然后重启apache即可。

也可以使用任务管理器，停掉对应的服务。 