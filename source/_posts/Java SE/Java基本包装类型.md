---
title: Java基本包装类型
date: 2017-08-09 10:14:07
tags: Java
categories:
  -Java
---
基本类型的对象包装，也就是将常用的基本数据类型包装成对象
* byte  Byte
* short Short
* int  Integer
* long  Long
* boolean Boolean
* float Float
* double Double
* char Character
<!--more-->
最常用的作用就是基本数据类型与字符串的转换
### 1. 基本数据类型转字符串：
基本数据类型+""  
基本数据类型类.toString(基本类型的数值)
### 2.字符串转成基本数据类型：
Integer.parseInt()
Long.parseLog()
对character不用转就是string
### 3.进制转换：
向十进制转：toHexString（）
向其他进制转换：parseInt("",radax)  radax指的是字符串的进制
### 4.自动拆箱和装箱：
1.5版本 的新特性，自动装箱与拆箱以前要这么写：  
```Java
Integer x=new Integer(1)  
Integer x=new Integer("1")
```
现在可以自动装箱：
```Java
  Integer x=5;  //自动装箱
  x=x+2 //先拆箱后和装箱  拆箱原理就是x.intValue()
```
1.5后对于在byte范围（-128~+127）内的数 如果一个数已经存在  则不会重新开辟新空间,也就是
```Java
  Integer x=127,y=127; //x===y
  Integer m=128,n=128; //m!==n
```
还有一点需要注意的就是 new String 和普通的 String = “” 这两个差别很大前者属于一个对象放在了堆内存中，而后者则是直接就在常量池中，不仅仅是字符串，其他都如此。
