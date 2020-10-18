---
title: VirtualEnv
date: 2017-05-06 23:18:33
tags:
categories:
  -Mac
---



### 修改配置文件

1）检查pip.conf文件是否存在
```
    >> cd ~
    >> mkdir .pip
    >> ls ~/.pip
```
2）直接编辑pip.conf
```
    sudo vi ~/.pip/pip.conf 
```
修改成如下的格式

```
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host = mirrors.aliyun.com
```
### 临时作用的方式
```
pip3 install 包名 -i https://mirrors.aliyun.com/pypi/simple/
```