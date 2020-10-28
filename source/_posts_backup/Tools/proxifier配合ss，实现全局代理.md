---
title: proxifier配合ss，实现全局代理
date: 2017-07-30 21:05:34
tags: shadowsocks
categories:
  -工具类
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;proxfixer配合ss的话，基本可以实现全局代理，分应用代理，或者玩外服的游戏（一般的游戏默认不走代理，本软件可以强制应用代理）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于ss使用的是sockets5代理，一般情况下只有浏览器支持，可以实现科学上网。但很多用户希望自己的应用软件，像outlook或游戏之类的软件也实现科学上网。这就需要proxifier的配合。
软件可以在官网下载，https://www.proxifier.com/
官网无法下载的也可在如下百度网盘下载，
链接: https://pan.baidu.com/s/1c11bs8C 密码: 44qj
目前仅支持windows和mac os，不支持手机。
此软件为收费软件，这里提供两个注册码, 软件分为Standard Edition和Portable Edition版本，注册码不通用，注册用户名任意。
```bash
L6Z8A-XY2J4-BTZ3P-ZZ7DF-A2Q9C（Portable Edition）
5EZ8G-C3WL5-B56YG-SCXM9-6QZAP（Standard Edition）
P427L-9Y552-5433E-8DSR3-58Z68（MAC）
```

### 1.打开软件，首先配置代理服务器。
如下图，添加地址127.0.0.1，以及ss里配置的本地端口，默认为1080，选择socks version 5
![](http://ojrw3x8j2.bkt.clouddn.com/17-7-30/93346575.jpg)
在配置完成之后需要测试一下，看配置的是否有问题。
![](http://ojrw3x8j2.bkt.clouddn.com/17-7-30/33843553.jpg)
### 2.接下来就要添加规则，来确定哪些软件是走代理的，哪些不用
按如图所示的添加，这里有个default规则，如果default旁边的action里边选择的时proxy socks5…则本机所有软件都会走代理。一般default会选direct，然后把你需要走代理的软件选成proxy socks5…
![](http://ojrw3x8j2.bkt.clouddn.com/17-7-30/18134133.jpg)
