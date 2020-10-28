---
title: Mac安装包版本的Python
date: 2017-05-06 23:18:33
tags:
categories:
  -Mac
---

1. 安装配置Python版本管理器pyenv

brew install pyenv

2. 查看已经安装了哪些版本的Python

pyenv versions
其中版本号前面有*号的就是当前生效的版本,查看当前生效的版本也可以用
pyenv version

3. 安装指定版本的Python

pyenv install 3.7-dev
>安装完成后必须rehash
pyenv rehash


4. 切换和使用指定的版本Python版本有3种方法

系统全局用系统默认的Python比较好，不建议直接对其操作
pyenv global system
用local进行指定版本切换，一般开发环境使用。
pyenv local 2.7.10
对当前用户的临时设定Python版本，退出后失效
pyenv shell 3.7-dev
取消某版本切换
pyenv local 3.5.0 --unset
优先级关系：shell——local——global

>另外最重要的一点，当执行了pyenv shell/local/global 命令后，一定要执行 source ~/.zshrc ，不然切换版本就不会生效。

最后执行
```bash
➜  ~ pyenv versions
  system
* 3.7-dev
➜  ~
➜  ~ python -V
Python 3.7-dev
```
