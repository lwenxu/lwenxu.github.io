---
title: Linux下安装shadowsocks
date: 2017-05-04 20:04:20
tags:
categories:
  -Linux
---
![](http://i4.buimg.com/567571/32f51545bce7861e.jpg)
## Fedora/Red Hat Enterprise Linux

使用dnf安装:
```bash
sudo dnf copr enable librehat/shadowsocks
sudo dnf update
sudo dnf install shadowsocks-qt5
```
如果你要使用copr repo可能还需要安装下面这个插件：
<!--more-->
```bash
sudo dnf install dnf-plugins-core
```
当然你的Linux版本不是特别高的话可能用的不是dnf，这时候用yum就行：
```bash
sudo yum update
sudo yum install shadowsocks-qt5
```

## Ubuntu

> PPA is for Ubuntu >= 14.04.

```bash
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```
## Debian
首先需要安装一系列的ss-qt5的依赖：
```bash
sudo apt-get install qt5-qmake qtbase5-dev libqrencode-dev libqtshadowsocks-dev libappindicator-dev libzbar-dev libbotan1.10-dev
```
然后执行下面的命令，就能得到一个Deb包
```bash
dpkg-buildpackage -uc -us -b
```
之后直接使用dpkg安装即可：
```bash
sudo dpkg -i shadowsocks-qt5-<VER_ARCH_ETC>.deb.
```
