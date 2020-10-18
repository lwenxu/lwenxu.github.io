---
title: Mac配置PHP7环境
date: 2017-05-07 21:40:24
tags:
categories:
  -Mac
---

>在Mac下面本来就安装有Apache2和PHP5.5.3
这里为了配置更新的环境就稍微折腾了一下，主要配置的就是MySQL和PHP，Apache就使用自带的就行。

## 配置MySQL：
MySQL在MySQL的官网上直接下载dmg包之后安转就行了。
这里Mac版本的MySQL下载地址是
>https://dev.mysql.com/downloads/file/?id=461365

下载之后在安装完成以后注意他会提示有一个默认的密码，一定要给记下来，一会改密码用的上。
再安装完成了，打开偏好设置，最下面有一个MySQL的控制面板，在里面开启MySQL
然后在终端中修改MySQL的密码
```bash
ln -s /usr/local/mysql/bin/mysql /usr/local/bin
mysql -uroot -p     #输入刚才提示的密码，登录以后开始修改密码
set password for root@localhost=password('password');
exit
```

## 配置PHP7环境
添加PHP7的源
```bash
brew tap homebrew/dupes  
brew tap homebrew/versions  
brew tap homebrew/homebrew-php  
brew install php70  
brew unlink php55
brew link php70 --with-httpd24
```
需要等待比较久的时间，因为是直接去官方下载。

## 配置Apache
直接vim Apache的配置文件，记得使用sudo，否则没权限写入。
当然不用sudo也可以，在修改完成以后执行 :w !sudo tee %
然后就可以使用管理员权限写入了。
然后打开Apache的配置文件主要得目的就是修改DocumentRoot和PHP模块文件
```bash
sudo vim /etc/apache2/httpd.conf
```
找到DocumentRoot改成你想要的目录，但是这个目录必须要有权限，一般给个777绝对ok
并且找到下面的地方，把PHP5的模块改成下面的内容：
```bash
#Enable PHP 7 module  
LoadModule php7_module /usr/local/opt/php70/libexec/apache2/libphp7.so
```
