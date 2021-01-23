---
title: Git 骚操作
date: 2020-03-07 22:39:26
tags: git
categories:
  - git
---

## 跳到之前的分支

```
git checkout -
```

## 查看历史

```
# 每个提交在一行内显示
git log --oneline

# 在所有提交日志中搜索包含「homepage」的提交
git log --all --grep='homepage'

# 获取某人的提交日志 
git log --author="Maxence"
```

## 修正

比方说想在提交 fed14a4c 加上一些内容。

![git 提交分支](https://segmentfault.com/img/remote/1460000021643076)

git 提交分支

```
git add .
git commit --fixup HEAD~1
# 或者也可以用提交的哈希值（fed14a4c）替换 HEAD~1

git rebase -i HEAD~3 --autosquash
# 保存并退出文件（VI 中输入 `:wq`）
```

## 从远程分支检出新的本地分支

```
git checkout origin/aa aa
```

