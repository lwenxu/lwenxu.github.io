---
title: LinkedHashMap 源码分析
date: 2018-03-26 16:15:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---

# LinkedHashMap 源码分析

> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1. 基本结构
### 1. 实现
实现的接口是 `Map` 
### 2. 继承
&emsp;&emsp;  继承的是 `HashMap` 这个就比较熟悉了，事实上我们会看到 `LinkedHashMap` 代码量非常的少，主要就是因为他继承的 `HashMap` ，继承了大多数的操作。 仔细一点的都会发现 `HashMap` 里面有非常多的空白方法，这些方法其实是模板方法，为了让继承 `HashMap` 的类重写一些自己的特性。而不破坏代码结构。
<!-- more -->
### 3. 数据域
#### 1. 基本字段

&emsp;&emsp;  在 `HashMap` 的基础上他添加了三个字段，这三个字段都非常重要，首先就是关于双向链表的两个字段 以及决定是否进行 LRU 的标志位。

```java
    // 双向链表的头结点
    transient LinkedHashMap.Entry<K,V> head;
    // 双向链表的尾节点
    transient LinkedHashMap.Entry<K,V> tail;
    //  决定是否进行 LRU 算法
    final boolean accessOrder;
```
#### 2. Entry 节点
&emsp;&emsp;  可能看过前面关于 `HashMap` 源码分析的都清楚，里面有一个 `TreeNode` 节点，他继承的就是 `LinkedHashMap` 中的 `Entry` 节点。这个节点里是在 `HashMap` 的  `Node` 节点上添加了 `before 和 after` ,也就是用来构造双向链表的指针。

### 4. 重点方法概览
1. ctor-5  最重要的能实现 LRU 的是设置 accessOrder 的那个
2. afterNodeRemoval
3. afterNodeInsertion
4. afterNodeAccess
5. containsValue
6. get
7. removeEldestEntry

## 2. 重要方法分析
### 1. 构造方法
&emsp;&emsp;  这个类有 5 个构造方法，一开始我以为和 `HashMap` 一样只有四个，后来又找到一个隐藏的比较深的方法，也是实现 LRU 最重要的一个方法。

&emsp;&emsp;  那么我们重点分析最后一个特殊的方法，前面几个方法我们都见过和 `HashMap` 中的差不多，就是多了一个设置 `accessOrder=false` 的操作。最后那个方法如果我们设置了 `accessOrder=true` 那么我们在访问一个元素的时候，这个元素会自动的排到双向链表的结尾。也就是所谓的 LRU。

&emsp;&emsp;  既然提到构造方法我们顺带说一下 `reinitialize` 方法这个方法是设置了头结点和尾节点。这些方法是在 `clone` 和 `反序列化` 的时候使用的。

```java
 public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
    
// 初始化双链表
void reinitialize() {
        super.reinitialize();
        head = tail = null;
    }
```
### 2. afterNodeRemoval
这个方法比较简单，也是模板方法，里面的主要作用就是修改一下双向链表，从而达到删除一个节点的作用。

```java
    // 这个方法还 ok 就是修改一下双向指针
    void afterNodeRemoval(Node<K,V> e) { // unlink
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.before = p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a == null)
            tail = b;
        else
            a.before = b;
    }
```
### 3. afterNodeInsertion
&emsp;&emsp;  在插入节点以后，如果说我们实现的 LRU 算法是需要删除一些旧的节点时候，这个方法就会在插入节点完成之后自动删除旧节点。删除的逻辑是 `removeEldestEntry` 方法决定的，如果要启用删除这个方法里面做删除逻辑，并且返回 `true` 。这里没有任何实现！

```java
    // 插入新的节点以后，如果说定义了删除老元素的方法，这个方法返回 true 的话就直接删除原来的旧元素 注意老元素是在头部  所以删除头部的元素即可
    // 在这个地方是不做人事事情的 removeEldestEntry  返回了 false
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            // 删除头部元素
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
```
### 4. afterNodeAccess
&emsp;&emsp;  在节点访问以后，如果说我们没有开启 LRU 算法，那没什么关系，访问了以后顺序不变，而如果`accessOrder=true` 那么访问的元素必须要放到双向链表的结尾。


```java
// 如果accessOrder 为 true，也就是支持 LRU 算法，那么就把这个元素先从双向链表中删除（在数组中的位置不变），然后插到链表的头部作为最新的元素
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }

```

### 5. containsValue
重写了父类的方法，主要是因为有了 双向链表的支持，遍历会更快。


```java
// 重写了 containsValue 因为有了双链表了遍历起来更方便
    public boolean containsValue(Object value) {
        // after 指针
        for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
            V v = e.value;
            if (v == value || (value != null && value.equals(v)))
                return true;
        }
        return false;
    }
```
### 6. get
get 方法属于访问元素，还是 LRU 的问题、


```java
// 重写了，获取元素以后，如果用了 LRU 需要重新排列该元素的位置
    public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        // 重新排列该元素位置
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
```
### 7. removeEldestEntry
空实现。


```java
    // 这个方法是用来被覆盖的，也就是子类来用，本类用不着。
    // 如果有必要，则删除掉该近期最少使用的节点，
    // 这要看对removeEldestEntry的覆写,由于默认为false，因此默认是不做任何处理的。
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }

```


## 3. 总结
其实在理解这个容器的时候我们就把他当做 `HashMap` 加上一个无关的双向指针即可。 然后注意一下他的 LRU 算法的利用！

