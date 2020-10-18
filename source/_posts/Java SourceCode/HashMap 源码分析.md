---
title: HashMap 源码分析
date: 2018-03-26 16:15:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---
# HashMap 源码分析

> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1.结构

### 1. 继承
&emsp;&emsp;该类继承自 `AbstractMap` 这个类似于 `ArrayList`
### 2. 实现
具体如下：

1. 首先这个类是一个 Map 自然有 Map 接口
3. 然后就是两个集合框架肯定会实现的两个接口 Cloneable, Serializable 。
<!--more-->

### 3. 主要字段

#### 1. 属性字段

```java
   // 默认大小 16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
    // 最大容量 2^30
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 链表转成树，链表的长度 8
    static final int TREEIFY_THRESHOLD = 8;
    // 树转成链表时树大节点数 6
    static final int UNTREEIFY_THRESHOLD = 6;
    // Node 数组
    transient Node<K,V>[] table;
    // 大小
    transient int size;
    // 阈值 他等于容量乘以负载因子
    int threshold;
```
#### 2. Node 
​&emsp;&emsp; 这个其实就是在 JDK1.7 中我们常说的 Entry ，但是在 Java8 把 Entry 更进一步抽象了，放到了 Map 接口里面，那里面的内部接口。里面并没有定义任何的字段，只有一些公共的方法。

​&emsp;&emsp; 然后这个  `Node` 是实现了  `Entry` 接口，里面定义了四个属性，这几个属性也就是 `HashMap ` 的关键了，分别就是 `hash` 、`key`、`value` 、`next` 。下面具体的看下代码。

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
    }
```



#### 3. TreeNode 

&emsp;&emsp;  `TreeNode` 着很明显，我们在上面的属性字段提到了关于链表转成树的操作，那么我们就需要把 `Node` 节点包装成 ` TreeNode` 。这里有一个比较有意思的事情就是这个 `TreeNode` 是继承自 `LinkedHashMap` 的 `Entry` 但是他又继承自 `HashMap` 的 `Node` ，而 那个 `Entery` 在 `Node` 基础上添加了属性就是 `before` 和 `after` 。有点绕，那么简单来说就是 `TreeNode`  在 `Node` 里面添加了 `before` 、`after`  还有其他的红黑树的信息。来具体看一下结构。

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
    	//before after inhert from Entry
}
```



### 4. 主要方法概览

1. cotr-4
2. put/putVal
4. resize
5. putAll/putMapEntries
6. get/getNode/containsKey 
7. remove/removeNode/clear
8. containsValue
9. read/writeObject

## 2. 主要方法分析
###1. cotr-4	

&emsp;&emsp;  首先介绍一下构造方法，这里我们会看到四个构造方法，他们在里面做的事情都差不多，主要是设置容器的 `容量` 和`阈值` 。其中在上面的字段中我们看到了一些常量，其中就有说明初始大小就是 16 ，然后负载因子是 0.75 ，还有提到最大容量 2^30 。

&emsp;&emsp;  在进行数组大小设置的时候有一个比较有意思的方法，`tableSizeFor(int size)` 这个方法能够保证最后返回出来的值是一个比 `size` 大的最小的 2^n 这样一个数。这样说可能有点不好理解，举个例子吧。 假如我们传入一个 18 那么返回的就是 32 ，因为 32 是 `2 的 n 次方` 这类的数，然后他是最接近 18 的  `2 的 n 次方`  。

&emsp;&emsp;  然后你可能会发现为什么没有初始化  `Node`  数组，  这是因为在 `jdk1.8` 里面 `HashMap` 的实现他的空间是延时分配的，也就是在我们真正往里面 `put` 值的时候才会真的创建 `Node数组` ，这个到我们分析 `put方法` 的时候我们会看到这一机制。

   ​	

   ```java
   // 设置负载因子和初始大小
       public HashMap(int initialCapacity, float loadFactor) {
           // 参数判断
           if (initialCapacity < 0)
               throw new IllegalArgumentException("Illegal initial capacity: " +
                                                  initialCapacity);
           if (initialCapacity > MAXIMUM_CAPACITY)
               initialCapacity = MAXIMUM_CAPACITY;
           if (loadFactor <= 0 || Float.isNaN(loadFactor))
               throw new IllegalArgumentException("Illegal load factor: " +
                                                  loadFactor);
           this.loadFactor = loadFactor;
           // 这个方法就是把一个不是 2 的次幂的数转换成一个大于当前数的最小的 2 的次幂的数
           this.threshold = tableSizeFor(initialCapacity);
       }

       // 大小设置一下，负载因子默认
       public HashMap(int initialCapacity) {
           this(initialCapacity, DEFAULT_LOAD_FACTOR);
       }

       // 设置负载因子为默认的 0.75
       public HashMap() {
           this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
       }

       // putMapEntries 这个方法就是 new 新数组然后 foreach 拷贝
       public HashMap(Map<? extends K, ? extends V> m) {
           this.loadFactor = DEFAULT_LOAD_FACTOR;
           putMapEntries(m, false);
       }
   ```

   ​

### 2. put/putVal

   ​	`put` 方法是今天的重头戏，因为大部分的代码其实都集中在这里，put 方法直接调用了 `putVal` 这个方法里面就进行了真正的存放元素的操作。 

   ​	我先大概说一下 `putVal` 的逻辑，然后再看代码就不会那么头疼了。

1. 一开始判断当前的 `Node 数组` 是否是空，如果是空则进行初始化，也就是分配空间（这里就是啊前面提到的延时分配空间）

2. 接着需要计算这个插入的值在数组中的位置，计算的方法就是  `hash % capacity` ，但是你可能看到的代码不是这样而是采用的 `hash & (capacity-1)` ，但是他们是等价的！！！不过这个等价是有条件的，那就是 `capacity`  的值必须是 `2 ^ n` 。所以你现在可能理解为什么 `HashMap` 的大小一直需要为  `2 ^ n`  以及 `tableSizeFor` 的作用。这个等价是可以证明的，比较简单不再赘述。

3. 找到需要插入元素的位置以后，如果说这个位置没有元素那好，我们直接把这个元素插入即可。

4. 但是如果这个地方的元素并不是空的，那么我们要么就是插入了完全一样的 `key` 要么就是 `key` 不一样但是 `hash`  函数发生了冲突。

5. 如果是完全一样的 `key` 那我们就用新的 `value` 替换掉原来的 `value` 返回老值即可。

6. 但是如果是发生了 hash 冲突我们就需要解决冲突。在 `jdk1.8` 里面采用的解决冲突的方法就是在这个节点上生成一个链表或红黑树。至于具体生成哪种据坎节点的数量了，节点数量少链表就很快多了的话我们肯定采用平衡二叉树（红黑树）。这个分水岭的节点数是 8 ，在上面的数据域可以看到他是一个常量。

7.  对红黑树直接调用红黑树的 `putTreeVal` 方法插入，而链表的话我们直接插入到链表的尾部即可。对链表插入完成以后需要检测一下是不是需要转成红黑树。

8. 最后进行一下扩容判断，毕竟有新的节点加入。



​		以上就是 `putVal` 的全部过程， 其中有一个扩容操作没有说，一会会单独讲这个方法。下面看看这个方法的源码。

```java
	public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }


	// 第三个参数 onlyIfAbsent 如果是 true，那么只有在不存在该 key 时才会进行 put 操作
	// 第四个参数 evict 我们这里不关心
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 第一次 put 值的时候，会触发下面的 resize()。这是因为我们调用了默认的无参的构造方法导致的，无参的里面只设置了负载因子
        // 第一次 resize 和后续的扩容有些不一样，因为这次是数组从 null 初始化到默认的 16 或自定义的初始容量
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 找到具体的数组下标，如果此位置没有值，那么直接初始化一下 Node 并放置在这个位置就可以了
        // 这个地方采用的 (n - 1) & hash 来寻找数组的下标，他和 hash%n 的效果一样的 但是仅仅限制在 n 是 2 的次幂
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);//newNode 方法不用看  就直接 new

        else {// 数组该位置有数据
            Node<K,V> e; K k;
            // 首先，判断该位置的第一个数据和我们要插入的数据，key 是不是"相等"，如果是，把这个节点放到 e 里面
            // 最后还有一个判断 e 是不是 null 然后对这个节点的 value 进行替换也就是说，如果 key 一样的话直接替换 vaule key 不一样说明是碰撞
            if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 下面两种情况都是碰撞处理
            // 如果该节点是代表红黑树的节点，调用红黑树的插值方法，本文不展开说红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 数组该位置上是一个链表
            else {
                // 循环找链表的最后一个节点
                for (int binCount = 0; ; ++binCount) {
                    // 找到尾部就插入尾部 (Java7 是插入到链表的最前面)
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // TREEIFY_THRESHOLD 为 8，所以，如果新插入的值是链表中的第 9 个，会触发下面的 treeifyBin，也就是将链表转换为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 不可能发生，所以直接 break 了 ，这个条件在前面就过滤掉了，也就是 key 相同的情况应该进行 value 的替换
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // e!=null 说明存在旧值的key与要插入的key"相等"，不是碰撞情况而是一致的 key 需替换返回老值
            if (e != null) {
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);  //这个操作是空操作  模板设计模式   是给 LinkedHashMap 使用的
                return oldValue;
            }
        }
        ++modCount;
        // 如果 HashMap 由于新插入这个值导致 size 已经超过了阈值，需要进行扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict); // 同 afterNodeAccess
        return null;
    }
```



### 3.  resize

​	扩容方法也有点复杂，方法体有点长，没关系我们慢慢分析，先了解思路在看源码。

1. 虽然这个方法是扩容方法，但是他也承担着初始化的任务。前面我们提到在 `putVal方法` 中有为 `Node 数组` 分配空间的事情，但是这个分配空间是委托给了 这个方法进行的。

2. 所以开始确认当前是分配空间还是在扩容，如果是扩容我们要判断当前的容量是不是已经到达极限了也就是最大容量 ` 2^3` ，如果大于等于这个值我们不进行扩容把阈值设置为最大的整数，防止下次再进行扩容操作。否则的话我们正常扩容把容量调整为原来的二倍，这样做的原因很明显容量要是 `2 ^ n` 。

3. 接下来我们就可以 new 一个新数组了，当然如果是这个操作是初始化那么我们的工作就完成了，但是如果是扩容操作我们还需要把原来的数组中的元素迁移到新的数组中。

4. 接下来的操作就是数据迁移工作。迁移就是遍历原来的数组，然后如果这个位置只有一个元素那直接迁移，如果不是的话就只能是红黑树或者单链表了。

5. 遇到红黑树我们就调用红黑树的迁移方法，单链表就把原来的链表拆成两部分。挂在新的数组的位置，拆分的方法也很巧妙源码中会看到。



&emsp;&emsp;  所以流程大概清楚了就看源码，源码注释的比较清楚。

```java
final Node<K,V>[] resize() {
        //前面这一大堆都是关于计算扩容的操作   不管他
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        // 对应数组扩容
        if (oldCap > 0) {
            //到极限了不扩容 修改阈值防止下一次又进入扩容操作
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 将数组大小扩大一倍 将阈值扩大一倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        // 对应使用两个有参的构造方法初始化后，第一次 put 的时候  也就是说 HashMap 在初始化的时候没有分配数组，空间是延时分配的
        else if (oldThr > 0)
            newCap = oldThr;
        // 对应使用 new HashMap() 初始化后，第一次 put 的时候
        else {
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }

        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;


        // 用新的数组大小初始化新的数组
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        // 如果是初始化数组，到这里就结束了，返回 newTab 即可  接下来的操作就是数据迁移
        table = newTab;


        if (oldTab != null) {
            // 开始遍历原数组，进行数据迁移。
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 如果该数组位置上只有单个元素，那就简单了，简单迁移这个元素就可以了
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    // 如果是红黑树，就进行分裂操作
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    // 链表的话就要把这些数据迁移到对应的位置 ，注意不是直接把整个链表放到数组的新位置  而是拆成两个链表
                    else {
                        // 这块是处理链表的情况，需要将此链表拆成两个链表，放到新的数组中，并且保留原来的先后顺序
                        // loHead、loTail 对应一条链表，hiHead、hiTail 对应另一条链表，代码还是比较简单的
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;

                        //一条链表拆成两条
                        do {
                            next = e.next;
                            //这里就是用了一个条件拆分成了两条链表
                            //他代码这样写的原因在于：oldCap 是一个2的次幂，那么也就是说 他是形如 "100000000...000" 这个格式的
                            //那么任何一个数和他相与必然只有两种结果 0 / 非0 就看最高位，其他位出来肯定是0  这样就区分了两个链表  巧妙！
                            if ((e.hash & oldCap) == 0) {
                                //这里面的逻辑其实就是链表按照原来的顺序连接  也就是说原来 a 在 c 前面只要 ac 在同一条链表上 a 就在 c 前面
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                // 同上
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);

                        //用来把两条链表分别挂在正确的数组位置
                        if (loTail != null) {
                            loTail.next = null;
                            // 第一条链表新位置就是原来的位置
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            // 第二条链表的新的位置是 j + oldCap，这个很好理解
                            newTab[j + oldCap] = hiHead;
                        }

                    }
                }
            }
        }
        return newTab;
    }
```



### 4. putAll/putMapEntries

&emsp;&emsp;  前面分析完比较困难的 `putVal` 和 `resize` 方法后接下来的方法都很轻松了。

&emsp;&emsp;    这个 `putAll` 方法调用了 `putMapEntries` ，在构造函数中也调用了这个方法的。其具体的就是 `foreach` 拷贝元素。

### 5. get/getNode/containsKey 

​	这几个方法底层调用的都是 `getNode` 方法，它的原理就是判断第一个元素是不是，然后看是红黑树还是单链表再遍历得到结果。

```java
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
}


final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (first = tab[(n - 1) & hash]) != null) {
            // 判断第一个节点是不是就是需要的。
            // 至于为什么要判断第一个元素我认为可能是作者觉得虽然说我们可能会碰到冲突，但是元素不多的情况下真的不会冲突.
            //也就是最理想的情况就是一个元素。
            if (first.hash == hash && // always check first node
                    ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                // 判断是否是红黑树
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);

                // 链表遍历
                do {
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
}
```



### 5. remove/removeNode/clear

这几个方法也很简单，`remove` 就是底层依赖的 `removeNode` 就是先遍历找到对应的节点，然后在遍历去删除。

​		`clear` 方法和前面介绍的 `ArrayList`  一样就是把数组元素设置为 `null` 让他去 gc

```java
	public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }

    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        //首先 hash 对应的位置是有东西的 否则直接返回 null
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            //这个 if else 是用来寻找那个要删除的节点的
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            //这个 if 是用来删除上面找到的那个节点
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }

	public void clear() {
        Node<K,V>[] tab;
        modCount++;
        if ((tab = table) != null && size > 0) {
            size = 0;
            for (int i = 0; i < tab.length; ++i)
                tab[i] = null;
        }
    }
```



### 6. containsValue

​	这个方法还是采用遍历的方法，他没有区分是树还是链表统一的采用了 `next` 指针，这是因为 `key` 是作为红黑树的索引条件但是 `value` 并不是，并且在 `TreeNode` 中是有 `next` 的因为他间接继承了 `Node`。

```java
public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
        if ((tab = table) != null && size > 0) {
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    if ((v = e.value) == value ||
                        (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }
```

  

### 7. read/writeObject

​	最后还是序列化的问题，`Node数组`  并没有采用默认的序列化可以看到他加了 `transient` 关键字。这里手动序列化只是序列化了 `key` `value` 其他的一概不存储。原因还是节省空间。

```java
private void writeObject(java.io.ObjectOutputStream s)
        throws IOException {
        int buckets = capacity();
        // Write out the threshold, loadfactor, and any hidden stuff
        s.defaultWriteObject();
        s.writeInt(buckets);
        s.writeInt(size);
        internalWriteEntries(s);
    }

 void internalWriteEntries(java.io.ObjectOutputStream s) throws IOException {
        Node<K,V>[] tab;
        if (size > 0 && (tab = table) != null) {
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    s.writeObject(e.key);
                    s.writeObject(e.value);
                }
            }
        }
    }
```



## 3. Hashtable

​	&emsp;&emsp;    就像是 `ArrayList` 和 `Vector` 一样我们需要讨论一下 `Hashtable` 和 `HashMap` 之间的异同。

1. 继承结构他们的实现接口一致，继承的类却不同，`Hashtable` 继承的是 `Dictionary` 

2. 里面采用的结构是 `Entry 数组` ,没有采用延时空间分配，默认大小是 11 ，容量也不是 2^n 扩容是 `2n+1` 的增长 。他的扩容叫做 `rehash` ,但是注意在扩容的时候把元素的位置颠倒了，也就是链表反插。 迭代接口含有比较古老的 `Enumeration` 


```java
protected void rehash() {
        int oldCapacity = table.length;
        Entry<?,?>[] oldMap = table;

        // overflow-conscious code
        int newCapacity = (oldCapacity << 1) + 1;
        if (newCapacity - MAX_ARRAY_SIZE > 0) {
            if (oldCapacity == MAX_ARRAY_SIZE)
                // Keep running with MAX_ARRAY_SIZE buckets
                return;
            newCapacity = MAX_ARRAY_SIZE;
        }
        Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];

        modCount++;
        threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
        table = newMap;

        for (int i = oldCapacity ; i-- > 0 ;) {
            // 倒序插入到新的数组中，也就是老链表从头到尾遍历的节点头插到新的链表的头。
            for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
                Entry<K,V> e = old;
                old = old.next;

                int index = (e.hash & 0x7FFFFFFF) % newCapacity;
                e.next = (Entry<K,V>)newMap[index];
                newMap[index] = e;
            }
        }
    }
```

3. 确定数组的下标采用的直接 ` (hash & 0x7FFFFFFF) % tab.length` ，并且在计算 `hash` 的时候是直接采用了 `key.hashCode()` 而在 `HashMap` 中还使用了扰动计算用来降低冲突的概率。 简单来说就是结合高低位来降低 `Hash冲突`


```java
h ^= k.hashCode();
h ^= (h >>> 20) ^ (h >>> 12);
return h ^ (h >>> 7) ^ (h >>> 4);
```

4. 不允许 null 的键值，他在源码中的做法就是先判断 `value` 是不是 `null` ，如果是直接抛异常。至于为什么没有判断 `key` 这是由于一会我们进行 `put` 操作的时候自然会调用 `key.hashCode()` 异常妥妥的抛出来了。


```java

public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        /*
        关键在于一个对象的 HashCode可以为负数，这样操作后可以保证它为一个正整数
        0x7FFFFFFF is 0111 1111 1111 1111 1111 1111 1111 1111
        (hash & 0x7FFFFFFF) 将会得到一个正整数
        因为hash是要作为数组的index的，这样可以避免出现下标为负数而出现异常
         */
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }

```


5. 扩容迁移采用链表倒序插入，只有链表没有红黑树。

6. 全部的方法都是 `synchronized` 的方法。

&emsp;&emsp;   关于为什么 `&0x7ffffffff` 这里提一下，关键在于一个对象的 HashCode可以为负数，这样操作后可以保证它为一个正整数0x7FFFFFFF is 0111 1111 1111 1111 1111 1111 1111 1111(hash & 0x7FFFFFFF) 将会得到一个正整数。因为hash是要作为数组的index的，这样可以避免出现下标为负数而出现异常。


```java

```


