---
title: LinkedList 源码分析
date: 2018-03-26 15:15:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---
# LinkedList 源码分析

> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1.结构

### 1. 继承
&emsp;&emsp;该类继承自 AbstractSequentialList 这个是由于他是一个顺序的列表，所以说继承的是一个顺序的 List
### 2. 实现
这个类实现的接口比较多，具体如下：

1. 首先这个类是一个 List 自然有 List 接口
2. 然后由于这个类是实现了 Deque 这个接口是双端队列的接口，所以说它是具有双端队列的特性的。后面我们会看到很多关于双端队列的方法。
3. 然后就是两个集合框架肯定会实现的两个接口 Cloneable, Serializable 。
<!--more-->

### 3. 主要字段
#### 1. 属性字段

```java
    transient int size = 0;
    //指向链表的头指针和尾指针
    transient Node<E> first;
    transient Node<E> last;
```

#### 2. Node 节点

Node 节点是主要存放数据的地方这个节点数据结构也比较简单就是一个泛型加上前驱后继指针。也就是一个双向链表。

```java
 private static class Node<E> {
   E item;
   Node<E> next;
   Node<E> prev;

   Node(Node<E> prev, E element, Node<E> next) {
       this.item = element;
       this.next = next;
       this.prev = prev;
   }
 }
```
 
### 4. 主要方法概览

1. ctor-2
2. addFirst
3. addLast
4. addAll
5. add
6. indexOf
7. lastIndexOf
8. peek 获取第一个元素，是 null 就返回 null
9. peekFirst/Last  获取第一个最后一个元素
10. poll 删除第一个元素并返回 没有返回 null
11. pollFirst/Last 
12. offer 调用了 add
13. offerFirst/Last
14. push
15. pop
16. set
17. remove(noArgs) == removeFirst  继承自 deque
18. remove(E e) 查找删除
19. read/writeObject  还是手动的序列化，原因一样，直接序列化元素而没有 pre/next

## 2. 构造方法分析
只有两个构造方法。其中一个是默认的空构造也就是生成一个空的 LinkedList 另外一个就是接受一个 Collection 接口。里面调用了 PutAll 方法。

```java
    public LinkedList() {
    }

    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```
## 3. 主要方法分析
### 1. add
这个方法就直接调用了 `linkLast` 而 `linkLast` 里面就是直接把元素添加到元素的结尾。

```java
 public boolean add(E e) {
        linkLast(e);
        return true;
}

void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
}
       
```

### 2. addFrist/Last
这两个方法同上还是调用了 `linkFirst` 和 `linkLast` 所以说这几个添加修改的方法基本都是靠底层的同样的方法实现的。

```java
public void addFirst(E e) {
   linkFirst(e);
}

public void addLast(E e) {
   linkLast(e);
}
```

### 3. addAll
 该方法我们在构造方法中也看到了，在它里面实现的时候和 ArrayList 一样是直接把集合转成数组，然后进行创建新的节点插入进来。
 
 ```java
 public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
 ```
### 4. indexOf
这个方法里面采用 for 循环遍历，遍历的时候是从头结点开始遍历，只要找到那个元素立即返回，而不继续进行下去。

```java
public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```
### 5. lastIndexOf
这个方法和上面的方法实现的方式一样的，但是注意这个方法的意思是找到最后一个与之匹配的元素，他并不是从头开始找，而是直接从尾节点开始遍历。做法同理找到即停止。
### 6. peek/peekFirst/peekLast
`peek` 方法的意思就是返回最顶端的元素，如果这个元素不存在，那么直接返回 `null` 。之后还有 `peekFirst` 这类的就是返回第一个的意思。底层调用的就是头结点的属性。这些方法其实在 Collection 接口中是不存在的，主要就是因为他实现了 Deque 所带来的的新特性。
### 7. poll/pollFirst/pollLast
`poll` 用来删除头结点并返回，如果不存在就返回 `null`
剩下的两个方法同理。 
### 8. offer/offerFirst/offerLast
插入头结点。
### 9. push/pop
底层的方法就是 addFirst 和 removeFirst 
### 10. remove(noargs)/remove(E e)
无参的调用 removeFirst 有参数的就是去查找然后删除。
### 11. read/writeObject
这里同 `ArrayList` 自己手动的进行了序列化。序列化的时候只是对 Node 节点里面的元素进行序列化，而前驱后继直接省略，也是节约空间的想法。

## 4.总结
好，其实在完全理解 `ArrayList` 的基础之上看这篇文章就比较好理解，里面的操作更加简单。只是注意一下两者的区别，实现了 `Deque` 带来的不少新的方法。


