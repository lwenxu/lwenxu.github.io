---
title: 两个 python 的虚拟管理工具
date: 2017-05-06 23:18:33
tags:
categories:
  -Mac
---



# 两个 python 的虚拟管理工具

<hr>

## virtualenv

#### 安装 virtualenv
```
    sudo apt-get install python-virtualenv
```
#### 创建 env
```
    virtualenv ENV
```
#### 启动 env
```
    cd ENV
    source ./bin/activate
```

#### 退出虚拟环境
```
    deactivate
```

## virtualenv 的加强版 virtualenvwrapper

#### 优点：
1. 将所有虚拟环境整合在一个目录下
2. 管理（新增，删除，复制）虚拟环境
3. 切换虚拟环境

#### 安装：
```
    sudo easy_install virtualenvwrapper  
```
此时还不能使用virtualenvwrapper，默认virtualenvwrapper安装在/usr/local/bin下面，实际上你需要运行virtualenvwrapper.sh文件才行，先别急，打开这个文件看看,里面有安装步骤，我们照着操作把环境设置好。


#### 创建目录用来存放虚拟环境

```
    mkdir $HOME/.virtualenvs
```

#### 配置脚本文件

在 bashrc 文件中进行如下操作
```
    export WORKON_HOME=$HOME/.virtualenvs
    source /usr/local/bin/virtualenvwrapper.sh
    source ~/.bashrc
​````
此时virtualenvwrapper就可以使用了。

#### 基本操作
1. 列出基本环境
```
    workon/lsvirtualenv
```
2. 新建虚拟环境
```
    mkvirtualenv [虚拟环境名称]
```
3. 启动/切换虚拟环
```
    workon [虚拟环境名称]
```
4. 删除虚拟环境
```
    rmvirtualenv [虚拟环境名称]
```
5. 离开虚拟环境
```
    deactivate
```

```