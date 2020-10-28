---
title: Hexo
date: 2016-03-09 13:01:47
tags:
---
![](http://p1.bpimg.com/567571/76cdc0b676e4f4aa.png)
<h2>动因</h2>
新学期开始了，这个学期准备养成开始写博客的好习惯。以前什么东西都是在QQ空间里，
后来觉得技术性稍微强一点的就是CSDN和博客园。但是啊我觉得他们的广告和版式确实
有点让人不忍直视，毕竟是搞技术的不如自己搭一个博客吧。于是就有了下面的折腾了。
能踩的坑我都帮你们踩过了，大家按照我的步骤一步步往下搭就好，那么现在开始吧！
# 正文
<!--more-->
## 首先声明一下：
平台：windows
hexo版本：3.X
## 所需文件:
1. [Node.js] [http://nodejs.cn/]
2. [Git] (https://git-scm.com/downloads/)
（下载后直接安装就好:next，next，finsh）

## 配置hexo：
找到刚才安装的git bash
然后你需要在本地硬盘里面创建一个文件夹用来存放hexo的配置文件，例如我在这里创建的地方是F：/hexo ，
那么我们现在打开git bash然后输入

```bash
cd F：
cd hexo
```

现在你就会进入到你刚才创建的hexo文件夹了。然后就开始安装hexo了。 
依次在git bash中输入以下内容，时间会因为网速而不同：

``` bash
1.npm install hexo-cli -g
2.hexo init
2.npm install hexo-deployer-git --save
3.hexo g
4.hexo s
```

这样hexo就安装完成了，下面我们要看看hexo给我们的初始界面是什么样的就执行以下命令：

(以//开始的是注释，不要复制，给你们说明用的。)

``` bash
1.hexo g   //生成静态文件
2.hexo s   //开启本地服务器，用于页面预览
```

之后我们就到浏览器中看看我们的hexo长什么样子了，就在浏览器地址栏输入localhost:4000回车是不是看到了一篇hello world而且界面很漂亮。

## 配置Github：
安装完hexo只是我们在能看到啊，要让所有人都看到我们要把它同步到github上面，就相当于吧本地文件上传到一个免费服务器，别人都看得到。
首先你需要注册一个github账号，搜索github然后点击注册就好，按照他的提示一步一步来，注意要把用户名记住啊，那个倒时很有用的。
这里我的就是xpf199741，密码是自然的。注册完了就去添加一个仓库就是叫repository的东西
然后取一个名字，**注意仓库名字必须是“用户名.github.io”**，例如我的就是**xpf199741.github.io**前面一定要是用户名后面的也要遵循，否则后面步骤肯定通不过。
	
``` bash
	ssh-keygen -t rsa -C "你的注册github的电子邮箱地址"
```

大家在复制的时候把电子邮箱更改一下啊，有停顿时候回车就是了，然后在本地的c盘用户目录下面产生一个.ssh文件夹然后进去，打开id_rsa.pub，复制里面的全部内容。然后回到网页版的github上进入Settings，左边选择SSH Keys，Add SSH Key,title随便填，粘贴key（就是刚才复制的内容）。

然后回到git bash再输入：

``` bash
	ssh -T git@github.com 
```

如果如果是第一次的会提示是否continue，输入yes就会看到：You’ve successfully authenticated, but GitHub does not provide shell access 。这就表示已成功链接。

## 链接Github与hexo：
编辑F：\hexo下的_config.yml文件
在行末添加一下内容：

```	bash
	deploy: 
  		type: git
 		repository: git@github.com:xpf199741/xpf199741.github.io.git
 	 	branch: master
```

把xpf199741改成你的就好了，但是注意一下：**hexo的配置文件中任何’:’后面都是带一个空格的**
在git bash里输入：

``` bash	
	hexo g
	hexo d
```

执行上面的第二个命令，可能会要你输入用户名和密码，输入密码是不显示任何东西的，输完回车就可以了。
然后在浏览器中输入

``` bash	
	xpf199741.github.io.git
```

是不是在浏览器中看到了hexo的内容，不同于第一次看到的是，这次是在线的，所有人都可以看到。
至此你应该已经学会了搭建hexo咯。
## 尾声
再遇到什么问题可在下面评论,之后我会继续更新hexo的美化，配置，使用。