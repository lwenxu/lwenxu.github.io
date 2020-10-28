---
title: 七牛云关联Windows图床
date: 2019-01-03 15:14:53
tags: 工具
categories:
	- 工具
---

# 1. 注册七牛云

[七牛云](https://portal.qiniu.com/create) 地址，需要在这里进行注册

# 2.完成实名认证

需要上传身份证的正反面以及支付宝做一下认证即可。

首先进入个人中心

![](http://images.heniankj.com/20190103152010.png)

然后进行实名认证

![](http://images.heniankj.com/20190103152132.png)

由于我已经认证过了，所以显示认证完成，未认证的用户需要按照提示认证，一般来说 5分钟就能完成认证。

# 3. 创建对象存储

![](http://images.heniankj.com/20190103152527.png)

只需要填一下名字，然后因为是图床所以肯定是公开的访问权限。

![](F:\OneDrive\Blog\source\images\20190103152647.png)

# 4. 绑定域名

配置完空间以后就是需要关联域名，配置 CNAME 

![](http://images.heniankj.com/20190103152918.png)

绑定域名，这个域名需要是一个已经备案过的域名。

![](F:\OneDrive\Blog\source\images\20190103153019.png)

只需要填写域名这一项即可，最后就是需要到你的域名管理后台配置一个域名的解析，首先打开这个地址 https://portal.qiniu.com/cdn/domain/ 然后找到你的域名选择复制CNAME

![](http://images.heniankj.com/20190103153215.png)

然后需要做一个域名解析，比如到阿里云上

![](http://images.heniankj.com/20190103153419.png)

等一会会 https://portal.qiniu.com/cdn/domain/ 打开这个链接如果看到解析成功则说明配置完成了。

# 5. 关联 PicGo

这款软件是一个跨平台的图床软件个人觉得很好用。直接百度就可以下载到。

![](http://images.heniankj.com/20190103153719.png)

做完配置，然后确定，设置为默认图床就可以在上传区上传文件了！