---
title: ConcurrentHashMap 源码分析
date: 2018-03-27 16:15:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---

# ConcurrentHashMap 源码分析 
> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1. 前言
&emsp;&emsp;  终于到这个类了，其实在前面很过很多次这个类，因为这个类代码量比较大，并且涉及到并发的问题，还有一点就是这个代码有些真的晦涩，不好懂。前前后后大概花了三天的时间看完的一些重要操作，接着今天来整理一下。
<!-- more -->
&emsp;&emsp;  好了首先介绍一个个人的感受：

1. 首先这个类很多操作和 `HashMap` 是类似的，但是麻烦就麻烦在 `锁分离技术` 和 `并发处理` 
2. 底层还是采用的 `数组` + `链表` + `红黑树` 来实现的，但是红黑树的 `TreeNode` 改成了 `TreeBin`
3. 里面有很多 CAS （Compare And Swap）操作，比如说 `unsafe.compareAndSwapInt(this, valueOffset, expect, update) `意思是如果 `valueOffset` 位置包含的值与 `expect` 值相同，则更新 `valueOffset` 位置的值为update，并返回true，否则不更新，返回false。
4. 不仅仅是 CAS 还有一些重量级的锁。也就是 `synchronized代码块` 用来保证操作同一数组元素下的节点的一致性，后面会看到。 

## 2. 基本结构

### 1. 继承
AbstractMap
### 2. 实现
ConcurrentMap<K, V> 和 Serializable 没有克隆。应该也是出于安全考虑！
### 3. 属性

#### 1. 基本属性
```java
    // 最大容量，同 hashmap
    private static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认大小  16
    private static final int DEFAULT_CAPACITY = 16;
    // 数组的最大值
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    // 并发数，不可修改   默认 16
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
    // 负载因子 0.75
    private static final float LOAD_FACTOR = 0.75f;
    // 转成树链表最大长度 8
    static final int TREEIFY_THRESHOLD = 8;
    // 转链表的节点数
    static final int UNTREEIFY_THRESHOLD = 6;
    // 最小转换步长 16 常量不可修改
    private static final int MIN_TRANSFER_STRIDE = 16;
    //
    private static int RESIZE_STAMP_BITS = 16;
    // 最大  transfer helper 数 默认 2^16-1
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
    //
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
    // 标志位
    static final int MOVED = -1; // 正在 transfer 的节点的 hash 值
    static final int TREEBIN = -2; //  树根的 hash 值
    static final int RESERVED = -3; // 不进行序列化的 hash 值
    // 可用 cpu 数
    static final int NCPU = Runtime.getRuntime().availableProcessors();
    // 底层的数组 注意是 volatile 的
    transient volatile Node<K, V>[] table;
    // 在 resize 的时候使用的数组，只有在 resize 的时候才不是 null  一样volatile
    private transient volatile Node<K, V>[] nextTable;

    // 在没有竞争的时候使用，或者在初始化的时候作为一个反馈
    private transient volatile long baseCount;

    // sizeCtl 是一个控制标志
    // -1 表示初始化
    // -N 表示有 N-1 个线程在 resize
    // 正数或0代表hash表还没有被初始化，这个数值表示初始化
    // 大于 0 表示下一次进行扩容的时候的阈值
    // 它的值始终是当前ConcurrentHashMap容量的0.75倍，这与loadfactor是对应的。
    private transient volatile int sizeCtl;
```

其中尤其要注意那个 `sizeCtl` 现在不知道他的意思后面可能就看不懂了！！
#### 2. Node
和 `HashMap` 一样的。但是注意这个地方采用了 `volatile` 关键字，其他的真的一样，只有两个是 `volatile` 原因在于只有这两个是可变的，会变的。

```java
    static class Node<K, V> implements Map.Entry<K, V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K, V> next;
        
        // hashcode 比较有意思 HashMap 中使用了 Objects.hashCode(key) ^ Objects.hashCode(value)
        // 但是 Objects 里面还是调用了 key 和 val 的 hashCode 所以原理一样
        public final int hashCode() {
            return key.hashCode() ^ val.hashCode();
        }


        // 虽然 val 是 volatile 的变量但是不提供修改方法，否则抛异常  
        public final V setValue(V value) {
            throw new UnsupportedOperationException();
        }
        }
```

#### 3. Segment
&emsp;&emsp;  这个以前是特别重要的一个数据结构，基本所有的查删改都是依靠他的，但是在 `1.8` 中他的作用被削弱了，里面什么方法都没有。

```java
static class Segment<K, V> extends ReentrantLock implements Serializable {
        private static final long serialVersionUID = 2249069246763182397L;
        final float loadFactor;

        Segment(float lf) {
            this.loadFactor = lf;
        }
    }
```
#### 4. TreeNode 
由于直接继承了 `Node 节点` 所以相比 `HashMap` 中的 `TreeNode` 少了 before 和 after 属性。他的方法比较少主要是因为红黑树已经不用这个数据结构了而是采用的 `TreeBin` ，但是它存在是因为在转成红黑树的时候是先把 `Node` 封装成 `TreeNode` 然后再封装到 `TreeBin` 中的。

```java
 static final class TreeNode<K, V> extends Node<K, V> {
        TreeNode<K, V> parent;  // red-black tree links
        TreeNode<K, V> left;
        TreeNode<K, V> right;
        TreeNode<K, V> prev;    // needed to unlink next upon deletion
        boolean red;
        }
```
#### 5. TreeBin

```java
 static final class TreeBin<K, V> extends Node<K, V> {
        TreeNode<K, V> root;  //用了上文中的 TreeNode
        volatile TreeNode<K, V> first;
        volatile Thread waiter;
        volatile int lockState;
        // values for lockState
        static final int WRITER = 1; // set while holding write lock
        static final int WAITER = 2; // set when waiting for write lock
        static final int READER = 4; // increment value for setting read lock
        
        //  大量的 rb-tree 的方法  不分析了
        }
```
### 4. 主要方法概览
#### 1. ctor-4 
&emsp;&emsp;  在构造方法中类似于 `HashMap ` 的做法，在构造方法里面只进行一下参数的判断以及对一些属性进行赋值，但是没有对数组进行初始化。还是那个原理：延时加载，在 `put` 方法中肯定会对数组进行初始化。
&emsp;&emsp;  在这里主要设置的值肯定就是我们前面提到的 `sizeCtl` 属性，当他为整数的时候就是阈值，我们也看到阈值字段但是没有见到那个字段被使用，主要就是被 `sizeCtl` 实现了！还有需要设置的属性就是负载因子和初始的数组大小，默认是 `0.75` 和 `16` 。
&emsp;&emsp;  好具体说一下每一个方法的实现:
1. 第一个无参的就是空方法，所有的值采用默认
2. 有初始大小的，就按照 1.5*n 转成 2^n 这个格式。
3. 如果传入一个 Map 就和 HashMap 一样，调用 `putAll` 方法，然后 `putAll` 就是循环原来的数组，然后 `put` 到新的数组中。
4. 其他的就是手动设置大小和阈值。


```java
// 空方法，注释说会创建大小为 16 的数组，估计是延时加载 在 put 里面
    public ConcurrentHashMap() {
    }

    // 设置了 sizeCtl 就是下一次扩容的容量
    public ConcurrentHashMap(int initialCapacity) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException();
        // 如果要开的数组比最大的一半还大，那就直接分配最大容量
        // 否则分配 1.5n+1 向上取 2^n
        int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY : tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
        this.sizeCtl = cap;
    }

    // 同 HashMap
    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        this.sizeCtl = DEFAULT_CAPACITY;
        putAll(m);
    }


    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, 1);
    }

    // 设置容量和负载因子
    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        // 容量不能小于并发数
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long) (1.0 + (long) initialCapacity / loadFactor);
        int cap = (size >= (long) MAXIMUM_CAPACITY) ?
                MAXIMUM_CAPACITY : tableSizeFor((int) size);
        this.sizeCtl = cap;
    }
```
#### 2. size/sumCount
&emsp;&emsp;  `size` 方法直接调用了 `sumCount` ，然后 `sumCount` 作用就是统计一下 `cellCount` 数组中的值的和，这时候我们会发现 `cellCount` 是一个类，自己定义了一个静态内部类，作用就是专门来统计节点的数量。可见统计节点在并发情况是一件很困难的事，这里还专门设计了一个类来进行统计。里面就一个字段 volatile 的 long 。

```java
    public int size() {
        long n = sumCount();
        // 如果有一些奇怪的值，比如大于最大小于最小就设置为最大，或者最小
        // 否则就是正常的值
        return ((n < 0L) ? 0 : (n > (long) Integer.MAX_VALUE) ? Integer.MAX_VALUE : (int) n);
    }

    static final class CounterCell {
        volatile long value;

        CounterCell(long x) {
            value = x;
        }
    }

    final long sumCount() {
        CounterCell[] as = counterCells;
        CounterCell a;
        long sum = baseCount;
        if (as != null) {
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null)
                    sum += a.value;
            }
        }
        return sum;
    }
```     
#### 3. get
&emsp;&emsp;  get 方法就和 HashMap 差不多，因为我们会发现他连锁都没有加，直接就以前的那个套路，首先计算 hash 然后找到数组的节点，看第一个是不是，然后看是否是红黑树或者是链表，它们采用了顺序查找，就 while 循环。
&emsp;&emsp;  那么我们需要考虑一下为什么不需要加锁，难道我们在读元素的时候同时有别的线程在写不会出现安全问题？举个例子，当我们在遍历一个链表寻找元素的时候刚好有线程在链表的结尾做插入操作，要么我们读到链表结尾的时候，写线程没有更新链表的结尾元素那么我们就认为读先于写也就没有安全问题，因为我们在读的时候不会发现尾节点指针正好发生变化，简单来说写线程的节点指针操作是原子的，对其他线程也是可见的，这时你应该清楚为什么链表的 `next` 是 `volatile` 。

```java
public V get(Object key) {
        Node<K, V>[] tab;
        Node<K, V> e, p;
        int n, eh;
        K ek;
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 && (e = tabAt(tab, (n - 1) & h)) != null) {
            // 判断头结点是不是
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            // 红黑树查找
            } else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            // 顺序查找
            while ((e = e.next) != null) {
                if (e.hash == h && ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```
#### 4. containsKey/Value（Traverser对象）
&emsp;&emsp;  `containsKey` 这个方法比较简单，因为我们直接可以调用 `get` 方法就可以判断。因此它的实现就是调用了 `get` 方法。
&emsp;&emsp;  `containsValue` 方法必然是需要遍历节点的，因为他需要一个个的比较元素的值。里面并没有像 `get` 一样遍历，而是采用了一个遍历器 `Traverser` 这个遍历器的主要作用是如果发现正在 transfer ，就切换到新的数组里面，获取最新插入的值。

```java
 public boolean containsKey(Object key) {
        return get(key) != null;
    }

    public boolean containsValue(Object value) {
        if (value == null)
            throw new NullPointerException();
        Node<K, V>[] t;
        if ((t = table) != null) {
            Traverser<K, V> it = new Traverser<K, V>(t, t.length, 0, t.length);
            for (Node<K, V> p; (p = it.advance()) != null; ) {
                V v;
                if ((v = p.val) == value || (v != null && value.equals(v)))
                    return true;
            }
        }
        return false;
    }
```
#### 5. put/putVal
&emsp;&emsp;  好了，现在就准备开始介绍最重要的，put 方法了。这个方法里面直接调用了 putVal ，其实 putVal 的逻辑有点复杂，就单看代码来说，代码量还是比较大的。
&emsp;&emsp;  那么我们先介绍一下 putVal 的大概的逻辑然后再看代码。

1. 首先就判断了 key 和 value 不能为空的情况，一开始觉得还挺奇怪的 HashMap 就允许 null 作为键值为什么 ConcurrentHashMap 却不行，这里我查了一下这段源码作者 Doug Lea 的解释是 “ 在普通的非并发的 HashMap 中我们可以存放 null 然后至于这个地方是否是真的为 null 还是由于键不存在呢，我们可以采用 `containsKey` 来判断。但是在并发情况下在调用方法时可能会发生引用关系的变化导致歧义 ”。感觉还是有点抽象难懂，我自己的理解就是：当我们调用 `map.get(key)` 时候返回 `null` 的时候假如根本不存在这个 `key` ，但是我们由于不清楚这个地方的值到底是 `null` 还是这个 `key` 根本不存在，我们就会调用 `containsKey` 来检查，但是在这个时间间隙中有其他线程往里面放了一个 `key -> null` 导致我们检测的结果是有这个键值对，从而误判！

2. 然后我们前面也提到了，在构造方法中没有任何初始化表的操作，所以说我们在 putVal 中如果判断到表空，就需要进行初始化工作。这个初始化调用了一个 `initTable()` 这个方法稍后专门分析。

3. 接着就是利用 hash 值来获取数组的索引了，首先还是判断那个对应的位置有没有元素，如果没有的话就简单了，采用 CAS 操作添加一个新节点此时添加工作就完成了可以返回了。如果不是这种情况就继续往下看。

4. 我们看头结点的 Hash 值是否是 `MOVED` ，如果是就说明当前的表正在进行 `transfer` 我们就让当前线程去帮助 `transfer` 。现在没看懂没关系等看到 `transfer` 方法的时候就知道为什么了。

5. 否则的话就是一个正常的链表或者红黑树了，这时候我们就和 HashMap 一样，如果是一个链表我们就遍历链表，然后遇到相同的 key 进行 value 的替换，否则插入到链表的结尾。 如果是一个红黑树就执行红黑树的插入操作。 注意这个操作是在同步代码块中进行的，因为我们不能保证多线程的修改安全，但是这个代码块的锁是头结点，也就是数组有多少元素我们就可以同时操作多少把锁，这样并发数就是数组的长度。而前面定义的并发数没有实质的作用。

6. 最后由于我们插入了一个节点需要判断一下当前的节点数是不是大于转红黑树的阈值(默认为8)。是则调用 `treeifyBin` ，这个方法也比较复杂，待会专门来说。 


```java
    // 直接调用
    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    final V putVal(K key, V value, boolean onlyIfAbsent) {
        // 不允许 null 键值 解释是在我们没办法判断是没有对应的值还是值为 null 。
        // 在非并发的情况下我们可以使用 containsKey/Value(null) 来明确知道 是不是有 null key/val
        // 这也就解释了为什么 hashmap 在查找的时候采用了先使用 null 来查找的策略
        // 但是并发的话，他底层调用了 get 方法，而 get 方法不是同步的，有可能会发生改变产生歧义
        if (key == null || value == null) throw new NullPointerException();
        // 得到 hash 值
        int hash = spread(key.hashCode());
        // 用于记录相应链表的长度
        int binCount = 0;
        for (Node<K, V>[] tab = table; ; ) {
            Node<K, V> f;
            int n, i, fh;
            // 如果数组为 null 也就是没有初始化(延时加载)或者数组没有元素，进行数组初始化
            if (tab == null || (n = tab.length) == 0)
                // 初始化数组，后面会详细介绍
                tab = initTable();
            // 找该 hash 值对应的数组下标，得到第一个节点 f
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                // 如果数组该位置为空， 用一次 CAS 操作将这个新值放入其中即可，这个 put 操作差不多就结束了，可以拉到方法的最后面了
                // 如果 CAS 成功，那就是没有并发操作 方法可以结束了  有并发操作就进行下一次循环，注意外面的循环是死循环
                if (casTabAt(tab, i, null, new Node<K, V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            // hash 居然可以等于 MOVED，这个需要到后面才能看明白，不过从名字上也能猜到，肯定是因为在扩容
            else if ((fh = f.hash) == MOVED)
                // 帮助数据迁移，这个等到看完数据迁移部分的介绍后，再理解这个就很简单了
                tab = helpTransfer(tab, f);
            // 到这里就是说，f 是该位置的头结点，而且不为空  也就是一般情况
            else {
                V oldVal = null;
                // 获取数组该位置的头结点的监视器锁
                synchronized (f) {
                    // 判断是否有新的节点插入到头部，或者删除头部节点造成不匹配  进行下一次循环
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) { // 头结点的 hash 值大于等于 0，说明是链表
                            binCount = 1;
                            // 遍历链表
                            for (Node<K, V> e = f; ; ++binCount) {
                                K ek;
                                // 如果发现了"相等"的 key，直接覆盖他的 value ，方法结束
                                if (e.hash == hash && ((ek = e.key) == key || (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                // 到了链表的最末端，将这个新值放到链表的最后面
                                Node<K, V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K, V>(hash, key,
                                            value, null);
                                    break;
                                }
                            }
                        } else if (f instanceof TreeBin) { // 红黑树
                            Node<K, V> p;
                            binCount = 2;
                            // 调用红黑树的插值方法插入新节点
                            if ((p = ((TreeBin<K, V>) f).putTreeVal(hash, key,
                                    value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                // 退出链表/红黑树的操作  的同步代码块
                // binCount != 0 说明上面在做链表操作
                if (binCount != 0) {
                    // 判断是否要将链表转换为红黑树，临界值和 HashMap 一样，也是 8
                    if (binCount >= TREEIFY_THRESHOLD)
                        // 这个方法和 HashMap 中稍微有一点点不同，那就是它不是一定会进行红黑树转换，
                        // 如果当前数组的长度小于 64，那么会选择进行数组扩容，而不是转换为红黑树
                        // 具体源码我们就不看了，扩容部分后面说
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```

#### 6. initTable
&emsp;&emsp;  好，上面已经分析了关于 put 方法，我们还有两个问题没有解决，一个是初始化表，另外一个是转成红黑树，先分析表的初始化。那先回顾一下 HashMap 他是采用同样的延时加载然后在 `resize` 方法中进行了表的初始化工作。也就是 HashMap 的 `resize` 方法同时进行了初始化和扩容以及迁移的工作，在 ConcurrentHashMap 中划分的更细致，初始化就只进行初始化，因为他是并发的要考虑到更多。
&emsp;&emsp; 一样的，我们先分析一下大致的思路。

1. 首选先需要看是不是有别的线程在进行初始化，如果是我们就不要进行初始化了让出 cpu 资源让别的线程继续初始化。这个如何判断别的线程是否在初始化？就涉及到了前面的 `sizeCtl` 属性，当他是 -1 的时候就说明在进行表的初始化工作。

2. 显然当别的线程没有初始化，当前线程就要初始化。并且不让别的线程进行争夺，就把 `sizeCtl` 用 CAS 置为 -1，并开始初始化。

3. 初始化的工作有两个，一是 new 一个容量为 16 的新数组，其次设置扩容的阈值也就是 `sizeCtl` 的值，设置好了也说明初始化完毕。

```java
    private final Node<K, V>[] initTable() {
        Node<K, V>[] tab;
        int sc;
        while ((tab = table) == null || tab.length == 0) {
            // 别的线程已经初始化好了或者正在初始化 sizeCtl 为 -1
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // 让出线程的执行权
            // CAS 一下，将 sizeCtl 设置为 -1，代表抢到了锁 基本在这就相当于同步代码块
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        // DEFAULT_CAPACITY 默认初始容量是 16
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        // 初始化数组，长度为 16 或初始化时提供的长度
                        Node<K, V>[] nt = (Node<K, V>[]) new Node<?, ?>[n];
                        // 将这个数组赋值给 table，table 是 volatile 的  他的写发生在别人的读之前
                        table = tab = nt;
                        // 如果 n 为 16 的话，那么这里 sc = 12 其实就是 0.75 * n
                        sc = n - (n >>> 2);
                    }
                } finally {
                    // 设置下次扩容的时候的阈值
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```

#### 7. treeifyBin
&emsp;&emsp;  行，解决了初始化接下来就要看看链表转红黑树的操作了。这个操作没有像我想的那样直接进行转树的操作，而是做了一系列的判断。
&emsp;&emsp;  当链表长度大于等于 16  但数组长度小于 64 时，需要进行一次扩容操作，扩容操作又委托给了 `tryPresize` 扩容是预计扩容到原来的两倍。注意区分链表长度和数组长度不要弄混了！！！
&emsp;&emsp;  接下来就是真正的链表转成树的操作了。

```java
private final void treeifyBin(Node<K, V>[] tab, int index) {
        Node<K, V> b;
        int n, sc;
        if (tab != null) {
            // MIN_TREEIFY_CAPACITY 为 64
            // 所以，如果数组长度小于 64 的时候，其实也就是 32 或者 16 或者更小的时候，会进行数组扩容
            if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
                // 扩容操作 大小扩容为 (3*n+1)--> 2^n
                tryPresize(n << 1);
                // b 是头结点
            else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                // 加锁
                synchronized (b) {

                    if (tabAt(tab, index) == b) {
                        // 下面就是遍历链表，建立一颗红黑树
                        TreeNode<K, V> hd = null, tl = null;
                        for (Node<K, V> e = b; e != null; e = e.next) {
                            TreeNode<K, V> p =
                                    new TreeNode<K, V>(e.hash, e.key, e.val,
                                            null, null);
                            if ((p.prev = tl) == null)
                                hd = p;
                            else
                                tl.next = p;
                            tl = p;
                        }
                        // 将红黑树设置到数组相应位置中
                        setTabAt(tab, index, new TreeBin<K, V>(hd));
                    }
                }
            }
        }
    }
```

#### 8. tryPresize
&emsp;&emsp;  继续分析上面遗留下来的问题，扩容操作。

1. 首先判断表是否为空，空则进行初始化，代码同 `initTable` 

2. 如果 `3*tableSize+1 < 扩容阈值` 就不扩容，这个情况基本不会发生。

3. 将 `sizeCtl` 设置为下次要进行扩容的阈值，然后进行 `transfer` ，这个方法里面才是真正的将数组大小扩充为原来的两倍并且进行数据迁移。

&emsp;&emsp;  整体看起来还是以前的 HashMap 的套路，他的 `resize` 进行初始化、扩容、数据迁移。而这里把这三步拆成了三个方法分别来做，以保证高并发。

```java
private final void tryPresize(int size) {
        // c：size*1.5+1 ，再往上取最近的 2 的 n 次方。
        int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY : tableSizeFor(size + (size >>> 1) + 1);
        int sc;
        // sizeCtl > 0 表示下次扩容的阈值
        while ((sc = sizeCtl) >= 0) {
            Node<K, V>[] tab = table;
            int n;
            // 数组初始化
            // 这个 if 分支和之前说的初始化数组的代码基本上是一样的，在这里，我们可以不用管这块代码
            if (tab == null || (n = tab.length) == 0) {
                n = (sc > c) ? sc : c;
                if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                    try {
                        if (table == tab) {
                            @SuppressWarnings("unchecked")
                            Node<K, V>[] nt = (Node<K, V>[]) new Node<?, ?>[n];
                            table = nt;
                            sc = n - (n >>> 2); // 0.75 * n
                        }
                    } finally {
                        sizeCtl = sc;
                    }
                }
            // 如果扩容后的值还小于等于阈值或者，当前数组长度已经达到最大了 不进行扩容
            } else if (c <= sc || n >= MAXIMUM_CAPACITY)
                break;
            // 进行扩容操作
            else if (tab == table) {
                int rs = resizeStamp(n);
                // 不可能发生
                if (sc < 0) {
                    Node<K, V>[] nt;
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs ||
                            sc == rs + 1 ||
                            sc == rs + MAX_RESIZERS ||
                            (nt = nextTable) == null ||
                            transferIndex <= 0
                            )
                        break;
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                // 1. 将 SIZECTL 设置为 (rs << RESIZE_STAMP_SHIFT) + 2  这个值等于
                //  (n的32位二进制中空位数 << 16 ) + 2 肯定是正数 也就是下次扩容的阈值
                //  调用 transfer 方法，此时 nextTab 参数为 null

                // 2. 这个 cas 操作保证了只有一个线程会最先进行 transfer 操作
                else if (U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
            }
        }
    }
```

#### 9.  transfer
&emsp;&emsp;  好了，该进入正题了，`transfer` 是这个类中最麻烦也是最精巧的一个方法，还是先说思路再阅读代码。

1. 如果当前的 `nextTab` 是空，也就是说需要进行扩容的数组还没有初始化，那么初始化一个大小是原来两倍的数组出来，作为扩容后的新数组。

2. 我们分配几个变量，来把原来的数组拆分成几个完全相同的段，你可以把他们想象成一个个大小相同的短数组，每个短数组的长度是 `stride` 。

3. 我们先取最后一个短数组，用 `i` 表示一个可变的指针，可指向短数组的任意一个位置，最开始指向的是短数组的结尾。`bound` 表示短数组的下界，也就是开始的位置。也就是我们在短数组选择的时候是采用从后往前进行的。

4. 然后使用了一个全局的属性 `transferIndex`（线程共享），来记录当前已经选择过的短数组和还没有被选择的短数组之间的分隔。

5. 那么当前的线程选择的这个短数组其实就是当前线程应该进行的数据迁移任务，也就是说当前线程就负责完成这一个小数组的迁移任务就行了。那么很显然在 `transferIndex` 之前的，没有被线程处理过的短数组就需要其他线程来帮忙进行数据迁移，其他线程来的时候看到的是 `transferIndex` 那么他们就会从 `transferIndex` 往前数 `stride` 个元素作为一个小数组当做自己的迁移任务。

&emsp;&emsp;  好，现在可能感觉有点乱，来总结一下。当前的数组的迁移被分为很多的任务包，每一个任务包中有 `stride` 个元素，然后这些任务包需要被从后往前的分配给不同的线程。分配过程依赖于共享的全局变量 `transferIndex` 。这样做的原因就是为了高并发，不得不佩服写这个类源码的大师！ 下面用一张图在来总结一下上面所说的内容。
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-29/28767841.jpg)

&emsp;&emsp;  现在线程收到自己的任务包了，肯定就需要进行数据迁移的工作了。迁移工作就比较简单了，由于是需要对链表或红黑树节点进行操作，必须要对过程同步，锁还是头结点。进行节点迁移的时候，就是和 HashMap 一样，把原来的链表和树拆成两部分，分别放到 i 和 i+n 上。
&emsp;&emsp;  我们还是主要看链表的拆分，采用的看 hash 值的某一特定位是 0 还是 1 来决定放在哪个位置，节点采用的头插法，也就是部分的节点的顺序是反的。为什么说部分是反的？那是因为他在里面一大段代码都在干一件事，去找原链表的最后一段特定位相同的完整序列保持顺序不变。这个比较难说清楚还是用一张图来说明一下。
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-29/4080400.jpg)


```java
// 把节点拷贝到新数组里面
    // 调用情况： 1.在 put 中的转红黑树的时候，如果大小不足 64 试着扩容 时候调用了 transfer
    //           2.在调用 put 完成以后，有一个 addCount 操作里面也调用了 transfer
    //           3.helpTransfer 中
    private final void transfer(Node<K, V>[] tab, Node<K, V>[] nextTab) {
        int n = tab.length, stride;
        // 将这些节点分成若干个任务迁移，每一个任务里面的节点数就是 stride
        // 所以当只有一个 cpu 的时候就是一个任务，多个 cpu 按下面的规则
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range

        // 如果 nextTab 为 null，先进行一次初始化，在属性字段里面我们就看到了这个字段是只有在迁移的时候不是 null
        // 其他时间都是 null

        //由于前面的 cas 操作已经保证了只有一个线程进行 transfer 所以不担心每个线程都会 new 出自己的新数组
        if (nextTab == null) {
            try {
                // 容量翻倍，所以说前面那个 tryPresize 方法里面计算的希望要扩容的大小只是为了与阈值比较一下
                // 真正在扩容的时候只扩充为原来的 2 倍
                Node<K, V>[] nt = (Node<K, V>[]) new Node<?, ?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            // volatile 赋值
            nextTable = nextTab;
            // 用于控制迁移的位置
            transferIndex = n;
        }
        // 新数组长度
        int nextn = nextTab.length;
        // 标志节点
        // 这个构造方法会生成一个Node，key、value 和 next 都为 null，关键是 hash 为 MOVED
        // 后面我们会看到，原数组中位置 i 处的节点完成迁移工作后，
        // 就会将位置 i 处设置为这个 ForwardingNode，用来告诉其他线程该位置已经处理过了
        ForwardingNode<K, V> fwd = new ForwardingNode<K, V>(nextTab);
        // advance 指的是做完了一个位置的迁移工作，可以准备做下一个位置的了
        boolean advance = true;
        // 在对 nextTable 赋值后记得清理节点  待会说
        boolean finishing = false;



        // 死循环
        for (int i = 0, bound = 0; ; ) {
            Node<K, V> f;
            int fh;

            // 说了这么多 很多都是废话，其实就一句话就是：我们从数组的结尾开始把元素分成任务，其中每一个任务的节点数就是 stride
            // 任务是从后往前分配的，也就是最先分出去的是数组结尾的那一段。任务开始的元素下标是 i 结束的下标是 bound 注意 bound < i
            // 因为是从后往前来的
            while (advance) {
                int nextIndex, nextBound;
                // 刚领取的任务完成了 也就是 i>=bound  一开始我找了半天的 --i 竟然藏在这里
                if (--i >= bound || finishing)
                    advance = false;
                // 任务领完了 transferIndex 本来是数组的长度，现在都成0 了说明任务都分派完了
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                // 实施任务分派，更新了 TRANSFERINDEX 其他线程能看到
                } else if (U.compareAndSwapInt(this, TRANSFERINDEX, nextIndex, nextBound = (nextIndex > stride ? nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }

            // 主要就是判断 i<0 ， 也就是是不是都迁移完了  至于 i>=n 是 i = n 导致的
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                // 所有的迁移操作已经完成
                // 将新的 nextTab 赋值给 table 属性，完成迁移
                // 重新计算 sizeCtl = 1.5*n 下次的阈值 结束迁移方法
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }

                // 之前我们说过，SIZECTL 在迁移前会设置为 (rs << RESIZE_STAMP_SHIFT) + 2
                // d但是怎么会出现这种情况？？？？？
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    // 任务结束，方法退出
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;

                    // 到这里，说明 (sizeCtl - 2) == resizeStamp(n) << RESIZE_STAMP_SHIFT，
                    // 也就是说，所有的迁移任务都做完了，也就会进入到上面的 if(finishing){} 分支了
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            // 如果原数组 i 处是空的，没有任何节点，那么放入 ForwardingNode 作为标志
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            // 该位置处是一个 ForwardingNode，代表该位置已经迁移过了
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            // 原数组处有值
            else {
                // 对数组该位置处的结点加锁，开始处理数组该位置处的迁移工作
                // 迁移干的事同样的事 就是把链表拆成两部分，树也分裂成两部分  和 HashMap 真的几乎一样
                synchronized (f) {
                    // 重复判断防止多线程修改
                    if (tabAt(tab, i) == f) {
                        Node<K, V> ln, hn;
                        // 头结点的 hash 大于 0，说明是链表的 Node 节点
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K, V> lastRun = f;
                            // p.hash & n 其实是取某一特定的位，只能是 0/1 所以这个做法是把原来一个链表分成两个
                            // lastRun 指向最后一段 runBit 相同的连续的节点的开始
                            // 感觉这一段代码写得有点蠢，就是为了找到最后一段完整的同类型的节点遍历了整个链表 ？ 还是我理解错了？
                            for (Node<K, V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }

                            // 分链表的准备工作
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            } else {
                                hn = lastRun;
                                ln = null;
                            }
                            // 最后还是按照那个特定的位是 0/1 分成两个链表
                            // 链表的顺序翻转了，采用的头插
                            for (Node<K, V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash;
                                K pk = p.key;
                                V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K, V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K, V>(ph, pk, pv, hn);
                            }
                            // 链表放在新数组的位置 i 和 i+n
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            // 将原数组该位置处设置为 fwd，代表该位置已经处理完毕，
                            // 其他线程一旦看到该位置的 hash 值为 MOVED，就不会进行迁移了
                            setTabAt(tab, i, fwd);
                            // advance 设置为 true，代表该位置已经迁移完毕
                            advance = true;
                        } else if (f instanceof TreeBin) {
                            // 红黑树的迁移
                            TreeBin<K, V> t = (TreeBin<K, V>) f;
                            TreeNode<K, V> lo = null, loTail = null;
                            TreeNode<K, V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K, V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K, V> p = new TreeNode<K, V>
                                        (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                } else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }

                            // 如果一分为二后，节点数少于 8，那么将红黑树转换回链表
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                    (hc != 0) ? new TreeBin<K, V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                    (lc != 0) ? new TreeBin<K, V>(hi) : t;
                            // 处理同链表
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```
#### 10. helpTransfer 
&emsp;&emsp;这个方法主要是让其他线程在对表做操作的时候刚好遇到了，表在扩容，数据迁移。就让这些线程帮助完成这个数据迁移，也就是去领取 `transfer` 中的数据迁移的任务包。


```java
//1. putVal 当有 MOVED 节点的时候  2.replaceNode 也就是在 remove 中 当有 MOVED 节点的时候
    //3. clear 4.conpute/IfPresent 干嘛的？ 5.merge 干嘛的？
    // 好了发现规律了就是当我们在有遍历节点的操作的时候遇到了 MOVED 就去 helpTransfer 也就是 transfer
    final Node<K, V>[] helpTransfer(Node<K, V>[] tab, Node<K, V> f) {
        Node<K, V>[] nextTab;
        int sc;
        // 条件判断 + 把正在 transfer 的 nextTab 赋值给 nextTab
        if (tab != null && (f instanceof ForwardingNode) && (nextTab = ((ForwardingNode<K, V>) f).nextTable) != null) {
            int rs = resizeStamp(tab.length);
            // 当 transfer 还没有结束  sizeCtl < 0 表示在迁移
            while (nextTab == nextTable && table == tab && (sc = sizeCtl) < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    break;
                // 把 SIZECTL 加一 然后 transfer
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                    transfer(tab, nextTab);
                    break;
                }
            }
            return nextTab;
        }
        return table;
    }
```

#### 11. tableAt/casTableAt/setTableAt
&emsp;&emsp;  这些方法,不细看，里面主要就是封装了一些对表的 CAS 操作。只是用的比较多。
#### 11. remove/replaceNode、
&emsp;&emsp;  `remove` 方法也是调用了 `replaceNode` 方法，这个方法里面就是常规思路。如果看到有节点在 `transfer` 就调用 `helpTransfer`， 由于需要操作节点需要进行同步。对树和链表进行删除操作。


```java
final V replaceNode(Object key, V value, Object cv) {
        int hash = spread(key.hashCode());
        for (Node<K, V>[] tab = table; ; ) {
            Node<K, V> f;
            int n, i, fh;
            if (tab == null || (n = tab.length) == 0 || (f = tabAt(tab, i = (n - 1) & hash)) == null)
                break;
            // 有正在移动的节点就帮助移动
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                boolean validated = false;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        // 链表节点
                        if (fh >= 0) {
                            validated = true;
                            for (Node<K, V> e = f, pred = null; ; ) {
                                K ek;
                                if (e.hash == hash && ((ek = e.key) == key || (ek != null && key.equals(ek)))) {
                                    V ev = e.val;
                                    if (cv == null || cv == ev || (ev != null && cv.equals(ev))) {
                                        oldVal = ev;
                                        //value不为空，则更新值
                                        if (value != null)
                                            e.val = value;
                                        //value为空，则删除此节点
                                        else if (pred != null)
                                            pred.next = e.next;
                                        //符合条件的节点e为头结点的情况
                                        else
                                            setTabAt(tab, i, e.next);
                                    }
                                    break;
                                }
                                pred = e;
                                if ((e = e.next) == null)
                                    break;
                            }
                        //  红黑树
                        } else if (f instanceof TreeBin) {
                            validated = true;
                            TreeBin<K, V> t = (TreeBin<K, V>) f;
                            TreeNode<K, V> r, p;
                            if ((r = t.root) != null &&
                                    (p = r.findTreeNode(hash, key, null)) != null) {
                                V pv = p.val;
                                if (cv == null || cv == pv ||
                                        (pv != null && cv.equals(pv))) {
                                    oldVal = pv;
                                    if (value != null)
                                        p.val = value;
                                    else if (t.removeTreeNode(p))
                                        setTabAt(tab, i, untreeify(t.first));
                                }
                            }
                        }
                    }
                }
                // 进行了节点的删除  cas 更新 Count
                if (validated) {
                    if (oldVal != null) {
                        if (value == null)
                            addCount(-1L, -1);
                        return oldVal;
                    }
                    break;
                }
            }
        }
        return null;
    }
```
#### 12. clear
清空操作也要说一下，因为不再和以前一样就直接把所有元素置为 null 等待垃圾回收。因为这里涉及到表的大小统计，以及其他的线程同时在操作表可能会导致混乱。他是一个元素下的所有节点统计一遍然后修改容器的大小，也就是他是对每一个数组下的每一个节点进行回收的。显然回收节点需要同步。当然由于涉及到了修改表如果遇到了在扩容的情况也是需要帮助数据迁移。

```java
public void clear() {
        // 这个变量记录了要删除的节点数的负数
        long delta = 0L;
        int i = 0;
        Node<K, V>[] tab = table;
        while (tab != null && i < tab.length) {
            int fh;
            Node<K, V> f = tabAt(tab, i);
            // 空节点 进行下一趟
            if (f == null)
                ++i;
            // 帮助移动
            else if ((fh = f.hash) == MOVED) {
                tab = helpTransfer(tab, f);
                i = 0; // restart
            //  删除节点
            } else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        // 如果是链表节点就返回链表头
                        // 如果是树节点就返回他的下一个 否则返回 null
                        Node<K, V> p = (fh >= 0 ? f : (f instanceof TreeBin) ? ((TreeBin<K, V>) f).first : null);
                        // 遍历链表或者树
                        while (p != null) {
                            --delta;
                            p = p.next;
                        }
                        // 直接把数组对应的位置设置为 null
                        setTabAt(tab, i++, null);
                    }
                }
            }
        }
        if (delta != 0L)
            addCount(delta, -1);
    }
```
#### 13. read/writeObject

&emsp;&emsp;他的序列化也是自己实现的，只存放 key 和 value，在反序列化的时候进行整个表的重建。


## 3.总结
&emsp;&emsp;  这个类确实比较麻烦，但是整体思路确实和 HashMap 差不多，但是由于需要做到高并发，访问安全做了很多的更细节的操作，把原来 HashMap 中一步能做完的拆成好几步，并且并发度感觉比 1.7 版本的更高，因为在加锁的时候只加了一个元素，而 1.7 是加一个 Segment 的锁，锁粒度更小，并发度更高。以及在 `transfer` 中的任务分包真的很巧妙，加速扩容。其实写文字说明只是一个大概的流程，我对源码写了大量的注释，看完说明在看源码就会轻松很多，并且很多源码里面的细节在说明里面没有提到，所以了解大致思路后一定要仔细阅读源码。


