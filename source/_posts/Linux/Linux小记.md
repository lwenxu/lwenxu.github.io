---
title: 'Linux小记'
date: 2016-03-20 22:53:08
tags:
category: "Linux"
categories:
 Linux
---
# 正文
## 定义别名：
	
``` bash
	alias vi='vim'
```

## 查看别名：

``` bash
	alias 
```

## 让某个用户永久生效
<!--more-->
``` bash
	vim ~/.bashrc
```

## 删除别名

``` bash	
	ualias vi
```

## 快捷键：

``` bash
	ctrl+u  删除或剪切光标左侧的所有字符
	ctrl+y  粘贴
	ctrl+r 	搜索历史命令
	ctrl+d	推出登陆
	ctrl+z  暂停放入后台
	ctrl+l 	清屏
	ctrl+a 	光标移到开头
	ctrl+e  光标移到结尾
```
	
## *是任意多个任意字符(任意可以是0)

``` bash	
	ls *abc
```	
会显示所有以abc结尾的或者就是abc的文件

## []匹配括号中的任意一个，必须是一个。

``` bash
	ls [abc]df
```
	
匹配以abc其中一个开始，以df结尾的，文件名是三个字符

## ？这个是匹配任意一个字符

``` bash
	ls ?asc
```

	四个字符，以asc结尾

## [^]与2同只是取反


## ''单引号中所有的特殊符号都没有特殊的含义

## ""双引号特殊符号都有特殊意义

## ``反引号等价于$()里面的系统命令会先执行反引号和括号里面的命令

``` bash
	echo `ls`
```

显示本目录下的文件名

## \去掉特殊符号的意义

``` bash	
	echo \$date
```

显示的是$date去掉了$的特殊意义 

## 清空历史命令：

``` bash	
	history -c 
```

## 强制保存到历史命令文件(~/.bash_history)中：

``` bash
	history  -w
```

## 更改历史命令保存文件存的条数

``` bash
	vim /etc/profile   （profile是环境变量的配置文件）
	HISTZISE=1000
```

## 历史命令的调用：

```	bash
	!n  //重复执行第n条命令
	!!  //重复执行上一条命令
	!字符串  //重复执行最后一条的含有字符串的命令
```

## 将错误与正确信息都存在文件里

``` bash
	ls &>> abc   (以追加的形式)
	ls &>  abc  （以覆盖的方式）
```

## 普通的重定向
	
``` bash
	ls >> abc
	ls >  abc
```

## 只执行命令不进行结果的输出，在脚本中经常使用

``` bash	
	ls &>/dev/null
```

## 分开重定向：

```bash		
	ls >> abc 2>>def (正确的是放在abc，错误的放在def)
```