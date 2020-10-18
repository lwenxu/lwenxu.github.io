---
title: Java之StringBuffer
date: 2017-08-09 14:33:06
tags: Java
categories:
  -Java
---
### 1.存储：
 * append(data)  添加在最后
 * insert(index,data)  在制定位置添加
### 2.删除：
 * delete(start,end)  删除某一段字符串
 * deleteCharAt(index)  删除置顶字符
 <!--more-->
### 3.获取：
 * charAt(index)
 * indexOf(data)
 * lastIndexOf(data)
 * length()
 * subString(start,end)
### 4.修改：
 * replace(start,end,data)
 * setCharAt(index,data)
### 5.翻转：
 * reverse()
### 6.获取缓冲区的制定数据放在指定字符串中
 * getChars(start,end,dstChar,dstIndex)

还有一个容器叫做 StringBuilder 其实一般来说这个用的更加频繁，因为 StringBuilder 他是线程不安全的，速度会更快。
