---
title: HashSet 源码分析
date: 2018-03-27 14:15:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---

# HashSet 源码分析

> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1. 基本结构
### 1. 继承
这个类简直适合 `HashMap` 如出一辙，他们继承的类都极其相似。继承的是 `AbstractSet` 。
### 2. 实现
实现了 `Set` 接口，之后还有两个普遍的接口 Cloneable, java.io.Serializable 。
### 3. 主要字段
&emsp;&emsp;  这个字段非常的少，只有一个 `HashMap` 和一个常量。估计你已经猜到这个容器的底层是采用`HashMap` 实现的了，但是具体又是怎么实现的呢？
<!-- more -->
&emsp;&emsp;  我们都知道，`set 集合` 是不允许有重复元素的，那么他怎么让他没有重复元素呢，答案就是把元素存放在 `HashMap` 的键中，因为 `HashMap` 的键是用来唯一区别两个 `Node` 节点是否相同的，而 `value` 就存放那个常量。

```java
    private transient HashMap<E,Object> map;
    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
```
### 4. 主要方法
1. ctor-5
2. add
3. remove
4. read/writeObject

## 2.主要方法分析
### 1. 构造方法
&emsp;&emsp;  这里提供了五个构造方法，其实有一个构造方法是比较特殊的，他在方法里面 new 了一个 `LinkedHashMap` ，也就是底层的数据结构是双链表。你可能会注意到上面我们声明的是一个 `HashMap` 的引用，new 的 `LinkedHashMap` 不会有问题吗？别忘了 后者继承了前者，并在他的基础上添加了一个双向指针。其他的四个方法其实都没什么好说的，全是调用了 `HashMap` 的构造，然后默认大小还是 16.


```java
    public HashSet() {
        map = new HashMap<>();
    }


    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }


    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }


    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }

    // 包内可用！ 底层采用 LinkedHashMap
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
```
### 2. add
&emsp;&emsp;  就是由于上面也提到了底层采用的 `HashMap` 结构，所以 add 就调用了 put 方法，然后看返回的值是不是 `null` 从而判断是否是重复元素。

```java
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```
### 3. remove
&emsp;&emsp;  remove 和上面的add 一样的操作。


```java
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
```
### 4. read/writeObject
&emsp;&emsp;  和其他的容器一样重写了序列化反序列化的操作。
### 5.其他方法
&emsp;&emsp;  其他方法都是直接调用了 `HashMap` 的方法，所以比较简单。总共代码才三百多行，只要我们注意他的底层实现是 `HashMap` 就够了。

