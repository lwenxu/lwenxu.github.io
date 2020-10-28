---
title: AQS 与 Sync 源码分析
date: 2018-04-1 20:05:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---

# ReentrantReadWriteLock 源码分析
> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！


&emsp;&emsp;  虽然前面几篇文章，已经分析过很多次 AQS 但是有时候分析分析着就会陷入一种调用链分不清楚的情况。为了更好的理解 AQS 中的锁机制，专门用一篇文章分析 AQS 结构，理清调用关系！
<!-- more -->
## 1. AQS 结构分析
![](http://ojrw3x8j2.bkt.clouddn.com/18-4-1/98760195.jpg)

&emsp;&emsp;好了，一开始我们就能明显的看到有两个内部类 Node 和 ConditionObject 。这是 AQS 中的重点，其中 Node 是 AQS 中的线程表示，而 ConditionObject 则是等待唤醒机制的核心。

&emsp;&emsp;  接着就是一些重要方法：

* enq
* addWaiter
* setHead
* unparkSuccessor
* doReleaseShared
* setHeadAndPropagate
* cancelAcquire
* acquireQueued
* doAcquireInterruptibly
* doAcquireShared
* doAcquireSharedInterruptibly
* tryAcquire
* tryRelease
* tryAcquireShared
* tryReleaseShared
* acquire
* acquireInterruptibly
* acquireShared
* acquireSharedInterruptibly
* release
* releaseShared
* hasQueuedThreads
* hasQueuedPredecessors
* transferForSignal
* fullyRelease


## 2. Node 节点和 ConditionObject
### 1. Node 节点
&emsp;&emsp;  Node 节点其实就是我们对线程的一个封装，让线程能够被加入到我们设定的队列中进行等待或唤醒，其中里面有几个特别重要的属性：

1. next，pre 这是对于同步队列中的线程来说的，构建的双向链表
2. nextWaiter 则是给等待队列使用的单向链表
3. thread 线程引用
4. waitStatus 等待状态 

### 2. ConditionObject 
&emsp;&emsp;  这个对象就是一个等待队列的管理者，里面有对线程进行等待唤醒机制的操作。

&emsp;&emsp;  重要属性：first/lastWaiter 就是用来指示链表的首尾节点。

&emsp;&emsp;  重要方法：

1. await 首先需要判断是否被中断，然后加入等待队列(addConditionWaiter)，释放他具有的锁 (fullyRelease).然后用死循环检测自己是否在同步队列，并将自己 park。清理等待队列中的非 Condition 的任务。
2. signal->doSignal ，用 while 循环将线程从等待队列中移除，然后调用尝试放入同步队列(transferForSignal)并unpark。否则的话继续在等待队列中找。

## 3. 重要方法调用过程
### 1. acquire 
&emsp;&emsp;  获取独占锁，首先使用尝试获取锁(tryAcquire)，失败则加入同步队列(addWaiter),然后如果是队列中的第一个等待者自旋获取锁。
&emsp;&emsp;  一般来说 tryAcquire 是一个模板方法，在 AQS 中没有实现，然后其他的并发组件中使用了 AQS 后必然实现自己的 tryAcquire 因为不同的锁类型实现方式不同。使用图例解释一下执行过程。
![](http://ojrw3x8j2.bkt.clouddn.com/18-4-3/62101431.jpg)

### 2. releas
&emsp;&emsp;  这个逻辑更加简单，首先尝试释放锁(tryRelease)，成功则唤醒后继节点(unparkSuccessor)。

### 3. acquireShared
&emsp;&emsp;  先尝试获取共享锁(tryAcquireShared),失败则加入同步队列如果是头结点自旋获取锁，并尝试唤醒后继节点(doAcquireSharedInterruptibly)。基本是 acquire 中的 addWaiter 和 acquireQueued 的组合。
### 4. releaseShared
&emsp;&emsp;  这个和 release 有点类似，但是里面方法不太一样，首先还是调用了 tryReleaseShared 成功后，传播释放后继节点 doReleaseShared。这里面主要还是调用了 unparkSuccessor，但是还有标记传播的阶段。




