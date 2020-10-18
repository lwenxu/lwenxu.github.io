---
title: ReentrantReadWriteLock 源码分析
date: 2018-04-1 16:05:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---

# ReentrantReadWriteLock 源码分析
> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1. 概述
&emsp;&emsp;  这个类听名字好像是和 ReentrantLock 差不多，但是实际上他们两没有任何关系，他并没有直接或间接的继承 ReentrantLock。ReentrantLock 属于独占锁，也就是我们前面所说的在临界区之内只能有一个线程运行。比如说我们的 Hashtable 采用的就是这种方式，哪怕在 get 元素的时候都对表加了锁，其他线程希望读取都没办法，但事实上我们知道多个线程同时读不会引起安全问题。至于什么时候会出现安全问题，这里介绍一个操作系统中常提到的 `Bernstein条件` ，概括的来说就是两个线程对同一个资源不能同时进行如下操作： `读写`、`写读`、`写写` 。所以我们对数据进行并发访问是不会有问题的，于是诞生了 读锁 和 写锁的概念，在 Java 中提供的 `ReentrantReadWriteLock` 就是一个具体实现。
<!-- more -->
&emsp;&emsp;  对于 ReentrantReadWriteLock，当写操作时，其它线程无法读取或写入数据，而当读操作时，其它线程无法写数据，但却可以读取数据。

&emsp;&emsp;  介绍一下线程进入读写锁的条件。

1. 读锁：没有其他线程的写锁，没有其他线程的写请求。
2. 写锁：没有其他线程的读锁，没有其他线程的写锁。

&emsp;&emsp;  这个锁有以下的特性：

1. WriteLock 中可以加 ReadLock。反之不可！
2. WriteLock 可以降级为 ReadLock，反之不可！
3. 获取锁可被中断。
4. 锁数量有限制。

### 1. 实现
实现了 ReadWriteLock 接口，里面就两个方法让返回读写锁。
### 2. 字段
三个字段，一个读锁，一个写锁，一个锁的实现核心 sync。

```java
    // 维护两个锁，这两个锁里面的实现就是 sync
    private final ReentrantReadWriteLock.ReadLock readerLock;
    private final ReentrantReadWriteLock.WriteLock writerLock;
    final Sync sync;
```
### 3. 结构
![](http://ojrw3x8j2.bkt.clouddn.com/18-4-1/29922994.jpg)

&emsp;&emsp;  还是似曾相识的结构，里面采用了 AQS 衍生出来的 Sync 以及两个公平锁和非公平锁。接着就是两个新的内部类，分别是读锁，和写锁。里面引用了 sync 核心组件。现在可以说明的是，读锁采用的是共享锁，而写锁使用独占锁。也就是把 AQS 中的两类方法都用上了。


## 2. ReadLock 实现
### 1. lock
&emsp;&emsp;  lock 方法直接调用了 acquireShared ，在前面我们已经分析过好多次 acquireShared 方法，这里再大概说一下逻辑：先调用 tryAcquireShared 尝试获取锁，如果获取失败，则调用 doAcquireShared 加入等待队列，尝试自旋获取锁，并且唤醒同步队列中的线程。

&emsp;&emsp;  这里重点说一下 tryAcquireShared 方法，因为在公平锁和非公平锁中实现不同，所以放到了子类中实现，但是这里公平和非公平是一样的，都在 Sync 中实现，在 `FairSync/NonfairSync` 子类中只是实现了是否需要阻塞读写线程的判断条件。在 tryAcquireShared 中核心思想是这样的：

1. 如果没有非当前线程的写锁，则可以继续开始获取，否则返回失败。但是当前线程的写锁和要加读锁不冲突，这也就解释了上面提到的读锁中可重入写锁，反之不可以，不可以的情况待会再解释。
2. 如果当前线程不用等待，并且未达到读上限，读数量更新成功，没有并发抢占此方法则可以开始获取锁。
3. 更新读锁的重入次数。里面采用了缓存机制。
4. 出现等待，达到读上限，有并发抢占，再接着重试获取锁，重试获取锁和这里的逻辑一样，只是做了更加详细的判断在并发情况下更适用。

```java
// 获取读锁
        protected final int tryAcquireShared(int unused) {
            Thread current = Thread.currentThread();
            int c = getState();
            // 有其他线程的写锁，直接失败   自己线程的写锁可以允许
            if (exclusiveCount(c) != 0 && getExclusiveOwnerThread() != current)
                return -1;
            int r = sharedCount(c);
            // 如果当前线程不用等待，并且未达到读上限，没有并发抢占此方法     可以获取读锁
            if (!readerShouldBlock() && r < MAX_COUNT && compareAndSetState(c, c + SHARED_UNIT)) {
                // 唯一一个读线程
                if (r == 0) {
                    firstReader = current;
                    firstReaderHoldCount = 1;
                // 可重入读线程
                } else if (firstReader == current) {
                    firstReaderHoldCount++;
                //  加入读线程，更新重入次数
                } else {
                    // 这一系列操作只是为了获取到当前线程的重入次数，本来直接用 readHolds.get() 就能搞定的，但是这里写了一大堆
                    // 是为了缓存，减少 readHolds.get() 开销
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            // 出现等待，达到读上限，有并发抢占，再接着重试获取锁
            return fullTryAcquireShared(current);
        }

        // 这个代码就是上面的代码的重复，但是他在非并发情况下会更简单 条件判断的更加详细，其余真的没什么了
        final int fullTryAcquireShared(Thread current) {
            HoldCounter rh = null;
            for (;;) {
                int c = getState();
                // 与上面等价  不能有其他线程的写
                if (exclusiveCount(c) != 0) {
                    if (getExclusiveOwnerThread() != current)
                        return -1;
                // 准备获取锁
                } else if (readerShouldBlock()) {
                    // 不请求重入锁，只是为了判断 firstReaderHoldCount > 0  ？？为啥
                    if (firstReader == current) {
                        // assert firstReaderHoldCount > 0;
                    } else {
                        // 这么大一段就是为了删除重入数为 0 的线程
                        if (rh == null) {
                            rh = cachedHoldCounter;
                            if (rh == null || rh.tid != getThreadId(current)) {
                                rh = readHolds.get();
                                if (rh.count == 0)
                                    readHolds.remove();
                            }
                        }
                        if (rh.count == 0)
                            return -1;
                    }
                }
                // 不能超限
                if (sharedCount(c) == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // 重复代码
                if (compareAndSetState(c, c + SHARED_UNIT)) {
                    if (sharedCount(c) == 0) {
                        firstReader = current;
                        firstReaderHoldCount = 1;
                    } else if (firstReader == current) {
                        firstReaderHoldCount++;
                    } else {
                        if (rh == null)
                            rh = cachedHoldCounter;
                        if (rh == null || rh.tid != getThreadId(current))
                            rh = readHolds.get();
                        else if (rh.count == 0)
                            readHolds.set(rh);
                        rh.count++;
                        cachedHoldCounter = rh; // cache for release
                    }
                    return 1;
                }
            }
        }
```
### 2. unlock
&emsp;&emsp;  unlock 也是调用了 `releaseShared(1)` ,然后里面的逻辑也是说过的，首先尝试释放当前线程执行 `tryReleaseShared` ，如果成功 `doReleaseShared ` 唤醒后继线程。

&emsp;&emsp;  还是 `tryReleaseShared` 方法。就是对读锁的重入次数进行减，删除那些计数值为0 的线程。

```java
// 释放读锁
        protected final boolean tryReleaseShared(int unused) {
            Thread current = Thread.currentThread();
            // 第一个读锁
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
                if (firstReaderHoldCount == 1)
                    firstReader = null;
                else
                    firstReaderHoldCount--;
            // 查找锁
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                int count = rh.count;
                if (count <= 1) {
                    readHolds.remove();
                    if (count <= 0)
                        throw unmatchedUnlockException();
                }
                --rh.count;
            }
            for (;;) {
                int c = getState();
                int nextc = c - SHARED_UNIT;
                // 没有读锁对读线程没有影响，但是对写线程有影响的
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
```

## 3. WriteLock 实现

### 1. lock
&emsp;&emsp;  调用 `acquire(1)` ，这个方法逻辑我们已经清楚了，现在就看一下 `tryAcquire()` 。先介绍一下基本思路：

1. 如果没有任何读写线程直接获取。
2. 要等待或者有竞争更新则失败
3. 有读线程或者非当前写、超过写重入限失败，否则更新重入次数，也就是设置 state 的值。这里也就解释了写锁中可以重入读锁（必须为当前线程的写锁），但是写锁中不允许有任何的读锁。


```java
 // 读锁获取
        /*
        1. 没任何锁直接获取
        2. 有读锁或非当前写，失败（也就是为什前面提到的在读锁中不能重入写锁的原因，但是反过来可以必须是当前写）
        3. 如果有等待条件失败
         */
        protected final boolean tryAcquire(int acquires) {
            Thread current = Thread.currentThread();
            int c = getState();
            int w = exclusiveCount(c);
            // 可能有读写线程
            if (c != 0) {
                // (Note: if c != 0 and w == 0 then shared count != 0)
                // 有读线程或者非当前写线程不可获取写锁 巧妙！！！
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                // 超过锁容量
                if (w + exclusiveCount(acquires) > MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // 重入写锁，获取成功
                setState(c + acquires);
                return true;
            }
            // 是否要等待
            if (writerShouldBlock() || !compareAndSetState(c, c + acquires))
                return false;
            // 没有读写线程 直接获取
            setExclusiveOwnerThread(current);
            return true;
        }
```
### 2. unlock
&emsp;&emsp;  调用 `release(1)` ，还是看 `tryRelease` ，减去那个值如果为0说明释放成功。


```java
 //读锁释放
        protected final boolean tryRelease(int releases) {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            // 低位为写锁，可以直接减
            int nextc = getState() - releases;
            boolean free = exclusiveCount(nextc) == 0;
            // 写锁个数为 0 释放线程
            if (free)
                setExclusiveOwnerThread(null);
            // state 设置为 0
            setState(nextc);
            return free;
        }
```

## 4. 总结
&emsp;&emsp;  好了现在算是把独占锁和共享锁来了一个大整合。说到底四个重要的方法，然后里面的调用链必须要清楚，一会我会再写一篇文章分析调用链。不然很容易就蒙了，方法有点多。
&emsp;&emsp;  对于这个类需要明白以下几点：

1. 也是采用了 state 变量来维护锁，高位读，低位写

2. 读锁全是共享锁，写锁全是独占锁了一下。

3. 锁重入问题，读不可重入写，反之可以。 

