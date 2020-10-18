---
title: LinkedHashSet 源码分析
date: 2018-03-27 13:15:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---

# LinkedHashSet 源码分析
> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1. 基本结构
&emsp;&emsp;  如果你看了 `HashSet` 现在基本一分钟就能弄明白 `LinkedHashSet` 的底层原理。里面没有任何字段，只有一个序列化 id，这个我们不说。然后他继承的是 `HashSet` ，有没有注意到简直就和 `LinkedHashMap` 一个套路，然后就调用一下父类的构造方法，但是调用的父类的构造方法都是同一个，很明显肯定是那个比较特殊的构造，也就是只能在包内访问，并且比其他的方法多一个 bool 参数的那个，因为它能指定底层的数据结构是 `LinkedHashMap` ，这也就解释了为什么 `LinkedHashSet` 中的四个方法都是 `public` 的，而最后一个是 `default`。
<!-- more -->
## 好的介绍完了！！！  ：）

