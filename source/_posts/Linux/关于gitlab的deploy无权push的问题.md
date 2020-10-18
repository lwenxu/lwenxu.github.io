---
title: 关于gitlab的deploy无权push的问题
date: 2017-04-27 17:48:58
tags: 工具
categories:
  - Web
---
今天准备两个人协同开发一个项目，然后建立了一个组。一开始是给了deploye权限，
但是他说没有权限上传，我就把权限提高但是我把权限提到master之后还是不行，清缓存
什么的都做了还是不行，提交的时候一直出现以下错误：
错误提示：
<!--more-->
>git -c diff.mnemonicprefix=false -c core.quotepath=false push -v origin master:master
Pushing to http://xxx/xxx/xxx_HTML.git

>POST git-receive-pack (47642 bytes)

>remote: GitLab: You don't have permission[K

>To http://xxx/xxx/xxx_HTML.git
! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'http://xxx/xxx/xxx_HTML.git'

最后还是找到了解决方案了，主要就是项目一般对于master分支对组中的成员是受保护的，具体方案如下：
找到项目：
![](http://i1.piimg.com/567571/9c5d32d60ef4a803.png)
在项目的【Setting】中的【Protected branches】可以设置哪些分支是被保护的，默认情况下【master】分支是处于被保护状态下的，develop角色的人是无法提交到master分支的，在下面的【Developers can push】打上钩就可以了。
![](http://i4.buimg.com/567571/f6d4e2adbfb1487a.png)
