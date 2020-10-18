---
title: ReentrantLock 与 AQS 源码分析
date: 2018-03-29 22:55:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---

# ReentrantLock 与 AQS 源码分析
> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1. 基本结构
&emsp;&emsp;  重入锁 ReetrantLock，JDK 1.5新增的类，作用与synchronized关键字相当，但比synchronized更加灵活。ReetrantLock本身也是一种支持重进入的锁，即该锁可以支持一个线程对资源重复加锁，但是加锁多少次，就必须解锁多少次，这样才可以成功释放锁。
<!-- more -->
### 1. 继承
没有继承任何类，因为很多操作都使用了组合完成。
### 2. 实现
Lock, java.io.Serializable 
&emsp;&emsp;这里着重介绍一下 Lock 接口，接口定义了几个必要的方法，也是在 ReentrantLock 中的重点需要分析的方法。
&emsp;&emsp;  三类方法：获取锁、释放锁、获取条件。

```java
public interface Lock {
    // 阻塞获取锁，如果获取不到锁就一直等待
    void lock();
    // 可中断获取锁，在获取锁的过程可以被中断，但是 Synchronized 是不可以
    void lockInterruptibly() throws InterruptedException;
    // 非阻塞获取锁，没有获取到锁立即返回
    boolean tryLock();
    // 超时获取锁，没获取到锁等待一段时间
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    // 解锁
    void unlock();
    // 等待唤醒机制的条件
    Condition newCondition();
}
```
从上面可以看到 Synchronized 和 Lock 的一些重要区别：

1. Lock 的获取锁的过程是可以中断的，Synchronized 不可以，Synchronized 只能在 wait或同步代码块执行过程中才可以被中断。

2. 由于 Lock 显示的加锁，锁可以横跨几个方法，也就是临界区的位置可以更加自由。

3. Lock 支持超时获取锁。

4. 后面会看到 Lock 还支持公平及非公平锁。

5. 绑定多个 Condition 条件 

### 3. 主要字段
&emsp;&emsp;很好，这个类的字段非常的少，真正起作用的字段只有一个 “锁” 字段。

```java
    // 同步锁
    private final Sync sync;
```
&emsp;&emsp;  这个锁（Sync）是一个继承自 AQS 的抽象内部类，说明一下 AQS (AbstractQueuedSynchronizer) 一般被称为队列同步器，他是并发包中的核心组件，绝大多数锁机制都是采用的这个类来实现的。虽然看到他是一个抽象类，但是你会发现里面没有一个方法是抽象方法，他实现了锁机制中的必要的通用的方法，待会会专门讲这个类。不然 ReentrantLock 没办法说，ReentrantLock 里面的锁操作都是依赖于 AQS。

&emsp;&emsp;  然后这个锁是有两个子类，分别是 `NonfairSync` 和 `FairSync` 从名字上也可以看出这两个类分别代表了 `公平锁` 和 `非公平锁` 。何为锁的公平性？ 实际上就是新来的线程需要征用锁必须要要等到先于他到达的线程获取并释放锁。也就是获取锁的过程是按照下来后到的顺序进行的，反之就称为非公平锁。后面我们会看到其实这两种锁不同就在于非公平锁在新线程创建后首先会直接进行锁的获取，如果没有获取到会进行一段时间的自旋，始终没获取到锁才进行等待状态。

&emsp;&emsp;  一般而言，公平锁开销比非公平锁大，这也是比较符合我们的直观感受。公平锁是需要进行排队的，但在某些场景下，可能更注重时间先后顺序，那么公平锁自然是很好的选择。

&emsp;&emsp;  好总结一下，在 ReentrantLock 中只维护了一个 “锁” 变量，这个锁是继承了 AQS 同步器，然后这个锁又有两种派生的锁：公平锁，非公平锁。那么 ReentrantLock 实现其实就有两种方式：公平锁，非公平锁。
### 4. 主要方法概览
1. ctor-2
2. lock
3. lockInterruptibly
4. tryLock
5. tryLock(time)
6. unlock
7. newCondition

## 2. 基础并发组件 AQS
### 1. 基本字段
#### 1. 重要字段
&emsp;&emsp;  AQS 是维护了一个同步队列（双向链表），这个队列里面线程都是需要竞争锁的，没有竞争到的就在同步队列中等待。` head` 和 `tail` 就指向队列的首尾。`state` 是一个标志字段，表示当前有多少线程在临界区。一般来说 `state` 只能是 0 或 1 但是由于锁是可重入的，所以也有大于 1 的情况。

&emsp;&emsp;  除了一个同步队列还有 0~n 个等待队列，等待队列就是调用了 `await` 方法的线程，会被挂到调用了 `await` 的 `condition` 上面的等待队列，所以有多少个 `condition` 就有多少等待队列。

```java
    //同步队列头指针
    private transient volatile Node head;
    // 同步队列尾指针
    private transient volatile Node tail;
    // 状态标志，0 则没有线程在临界区，非零表示有 state 个线程在临界区（由于锁可重入）
    private volatile int state;
```
#### 2. Node 节点
&emsp;&emsp;Node 节点也就是上文所提到的 `同步队列` 和 `等待队列` 中的元素，注意两个队列之间的元素类型是一样的因为他们之间会有相互移动转换的动作，这两个队列中的元素自然是线程，为了方便查找和表示 AQS 将线程封装到了 Node 节点中，构成双向队列。


```java
static final class Node {
        // 共享非 null/独占为 null  
        static final Node SHARED = new Node();
        static final Node EXCLUSIVE = null;

        /**
         * 线程状态
         */
        static final int CANCELLED =  1;
        static final int SIGNAL    = -1;
        static final int CONDITION = -2;
        static final int PROPAGATE = -3;
        volatile int waitStatus;
       // 双向链表  这两个指针用于同步队列构建链表使用的   下面还有一个 nextWaiter 是用来构建等待单链表队列
        volatile Node prev;
        volatile Node next;
        // 线程
        volatile Thread thread;
        // 等待队列单链表
        Node nextWaiter;

        /**
         * Returns true if node is waiting in shared mode.
         */
        final boolean isShared() {
            return nextWaiter == SHARED;
        }

    }
```

&emsp;&emsp;  可以看到上面有一个 `waitStatus` 属性，代表了线程当前的状态，状态标识就是那些常量。具体如下：

1. SIGNAL:     正在执行的线程结束释放锁或者被取消执行，他必须唤醒后续的状态为 SIGNAL 节点

2. CANCELLED:  在同步队列中等待的线程等待超时或被中断，需要从同步队列中取消该Node的结点， 其结点的waitStatus为CANCELLED，即结束状态，进入该状态后的结点将不会再变化。

3. CONDITION:  该标识的结点处于等待队列中（不是同步队列），结点的线程等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将从等待队列转移到同步队列中，等待获取同步锁。

4. PROPAGATE:在共享模式中，该状态标识结点的线程处于可运行状态。

5. 0:代表初始化状态。

&emsp;&emsp;  可以看到，Node 里面的主要字段就是一个状态标志位、一个线程的引用、用于构建链表的指针。注意，有三个指针，其中前两个 `next` 和 `pre` 是用来构建同步队列的（双向链表），后面 `nextWaiter` 是用来构建等待队列。所以说虽然同步队列和等待队列使用的同一个数据类型，数据结构是不同的，并且在后面我们会看到等待队列中的节点只有两种状态 `Condition` 和 `CANCELLED` 前者表示线程已结束需要从等待队列中移除，后者表示条件结点等待被唤醒。

&emsp;&emsp;下面画图说明一下同步队列和等待队列的情况。
等待队列
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-30/81487339.jpg)
同步队列
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-30/91260714.jpg)
#### 3. ConditionObject

&emsp;&emsp;  这个内部类是等待唤醒机制的核心，在他上面绑定了一个等待队列。在这个类中使用了两个指针（ `firstWaiter/lastWaiter` ）指向队列的首尾。这里主要看一下 `await` 、`signal` 和 `signalAll` 方法。

1. await
&emsp;&emsp;  当一个线程调用了await()相关的方法，那么首先构建一个Node节点封装当前线程的相关信息加入到等待队列中进行等待，并释放锁直到被唤醒（移动到同步队列）、中断、超时才被队列中移出。被唤醒后的第一件事是抢锁和检查是否被中断，然后才是移除队列。被唤醒时候的状态应该为 SIGNAL ，而在方法中执行的移除队列的操作就是移除状态非 Condition 的节点。

```java
public final void await() throws InterruptedException {
            // 等待可中断
            if (Thread.interrupted())
                throw new InterruptedException();
            // 加入等待队列， new 新的 Node 做一个尾插入
            Node node = addConditionWaiter();
            // 释放当前线程的锁，失败则将当前线程设置为取消状态
            int savedState = fullyRelease(node);

            int interruptMode = 0;
            // 如果没在同步队列就让线程等待也就是看是否被唤醒
            // 如果有中断或者被唤醒那么退出循环
            while (!isOnSyncQueue(node)) {
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            // 运行到此处说明已经被唤醒了，因为结束了循环
            // 唤醒后，首先自旋一下获取锁，同时判断是否中断
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            // 清理队列中状态不是 Condition 的的任务，包括被唤醒的 SIGNAL 和 被取消的 CANCELLED
            if (node.nextWaiter != null)
                unlinkCancelledWaiters();
            //被中断 抛异常
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }
```

2. signal/doSignal/signalAll
&emsp;&emsp;  执行 signal 首先进行锁的判断，如果没有获取到独占锁就直接抛出异常。这也就是为什么只有拥有锁的线程才能执行 signal ，然后获取等待队列中的第一个节点执行 doSignal。

```java
        public final void signal() {
            // 获取独占锁
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            // 唤醒等待队里中的第一个线程
            Node first = firstWaiter;
            if (first != null)
                doSignal(first);
        }
```
&emsp;&emsp;  doSignal 方法主要就干了三个事 ：
1. 将被唤醒的节点从等待队列中移除（while 循环体），如果被唤醒的节点被取消了就继续唤醒后面的节点（transferForSignal 返回 false）
2. 否则把这个节点加入到同步队列 （ enq 方法 ）
3. 当同步队列中当前节点的前驱被取消或者没办法唤醒时则唤醒这个线程 （ unpark ），这时候调用了 unpark 正好和 await 中的 park 相对应使得 await 的线程被唤醒，接着执行循环体判断自己已经被移入到同步队列了，接着就可以执行后面的获取锁的操作。

```java
 private void doSignal(Node first) {
            do {
                // 头指针指向唤醒节点的下一个节点，并顺便判断等待队列是否空
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                // 解除引用
                first.nextWaiter = null;
            } while (!transferForSignal(first) && (first = firstWaiter) != null); //移入同步队列失败则继续唤醒下一个线程，否则唤醒成功
            // 唤醒成功的线程不一定马上能开始执行，只有在前驱节点被取消或者没办法被唤醒时
   }
   
   
   //  将节点从等待队列移动到同步队列   成功返回 true 失败 false
    final boolean transferForSignal(Node node) {
        // 在等待队列中的节点只有 condition 和 cancelled 两种状态，如果状态更新失败说明任务被取消
        // 否则更新为初始状态   直接返回的话上面的 doSignal 就会继续唤醒后面的线程
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;
        // 把当前节点加入同步队列
        Node p = enq(node);
        // 获取同步队列中倒数第二个节点的状态，当前节点的前驱
        int ws = p.waitStatus;
        // 如果前驱节点被取消或者在设置前驱节点状态为Node.SIGNAL状态失败时，唤醒被通知节点代表的线程
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }
    
    
    // 插入一个节点到同步队列，如果同步队列是空的则加入一个空节点做为头结点
    // 死循环保证肯定能插入    返回插入节点的前驱
    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                // 这一步不需要 cas 是因为并发没关系，只是指向链表结尾，不会多线程更新问题
                node.prev = t;
                // 可能有多个线程抢
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```

&emsp;&emsp;  有一个小问题,就是在某个线程中执行了别人的 signal 不会导致当前线程立即放弃锁，之所以会这样正是由于 `ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL)` 这个判断，即前驱线程都结束了。比如下面的例子：

```java
package util.AQSTest;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

// test signal 执行后不会导致当前线程立即释放锁
public class AQSTest {
    static Lock lock = new ReentrantLock();
   static Condition run1Cond = lock.newCondition();
    static Condition run2Cond = lock.newCondition();

    static class Runner1 implements Runnable {
        @Override
        public void run() {
            lock.lock();
            try {
                System.out.println("runner 1 start");
                run1Cond.await(1, TimeUnit.SECONDS);
                run2Cond.signal();
                System.out.println("runner 1 exit");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }

        }
    }

    static class Runner2 implements Runnable {
        @Override
        public void run() {
            lock.lock();
            try {
                System.out.println("runner 2 start");
                run2Cond.await();
                System.out.println("runner 2 exit");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                lock.unlock();
            }

        }
    }
    public static void main(String[] args) {
        new Thread(new Runner1(),"runner1").start();
        new Thread(new Runner2(),"runner2").start();
    }
}

```

输出的结果始终是：

```
runner 1 start
runner 2 start
runner 1 exit
runner 2 exit
```

&emsp;&emsp;  我使用了工具对上面的代码进行了调试，大致说一下流程，顺便用来捋一捋等待唤醒机制。

&emsp;&emsp;  首先 runner1 启动，获取到锁，打印出 “runner1 start” ，然后调用了 await 方法，此时 runner1 线程就执行了 AQS 中的 ConditionObject 中的 await 方法，该方法首先 new 了一个新的节点，把 runner1 封装到这个节点里面。挂在了 run1Con 的等待队列上，然后执行了释放锁并判断中断。紧接着 runner1 线程执行循环体判断是否被唤醒也就是是否在同步队列，显然这时候不在，就直接调用了 park 方法，执行休眠 1 秒钟操作， park 方法是 native 方法由操作系统实现。在上面线程释放锁的时候执行的操作是 `fullyRelease` 这个方法调用了 ` release` 方法，而 `release` 方法中释放了锁之后，会检查同步队列中是否还有以前因为没抢到锁而等待的线程，如果有执行 `unparkSuccessor` 也就是唤醒同步队列中的后继线程。那么此时 runner2 会被唤醒，唤醒后就去抢锁，获取到 lock 锁后输出了 “runner2 start” ，然后 runner2 线程又会因为调用 `await` 处于和 runner1 同样的境地，也就是被放入 run2Con 的等待队列。好！此时 runner1 的超时时间到了，就会被 unpark 这个 unpark 是被操作系统调用的，之后继续执行循环体发现超时时间小于等于 0 ，则调用 `transferAfterCancelledWait` 里面调用了 `enq` 就是加入同步队列，接着开始竞争锁，开始执行 run2Con 上的 signal 此时 signal 调用 doSignal 先执行 do while 中的循环体，runner2 从 run2Con 的等待队列上移除，然后执行 `transferForSignal` 里面又调用了 `enq` 将他加入同步队列，并返回同步队列中的前驱，前驱节点状态不是 Cancelled 或者 可以被置为 SIGNAL 则 signal 方法结束。接着打印了 “runner1 exit” 。接着需要执行 finally 里面的释放锁的操作了，显然 unlock 肯定调用了 release ，而 release 会唤醒同步队列中的后继的线程，那么位于同步队列中的 runner2 之前的 park 状态就会被打断，从而跳出 while 循环，执行获取锁的操作。打印出 “runner2 exit” ，最后释放锁整个程序结束。


&emsp;&emsp;  现在总算是吧 Condition 的等待唤醒机制弄清楚了。也把 AQS 中的两个内部类的功能都解释完了。接下来就看 AQS 中的方法。


### 2. 重要方法

1. get/setState
2. release/tryRelease/unparkSuccessor/fullyRelease
3. acquire/tryAcquire/addWaiter/tryQueued
4. acquireShared
5. releaseShared

&emsp;&emsp;  这些属于 AQS 中常用的方法，但是里面的核心方法都是模板方法，也就是说由继承他的子类来实现，所以只能看个大概的逻辑。一会等到讲 ReentrantLock 时再详细说这里面的方法。

## 3. ReentrantLock 内部类 Sync/fairSync/noFairSync

### 1. Sync
&emsp;&emsp;  这三个内部类实际上是继承自 AQS ，也就是说 ReentrantLock 是采用了 AQS 作为自己的核心并发控制组件完成的一系列的锁操作，及等待唤醒机制。

&emsp;&emsp;  首先看一下 Sync 他是后面两个的父类，他直接继承自 AQS 。AQS 中留了几个比较重要的模板方法 tryAcquire 、tryRelease 。这个方法直接实现了一些在公平锁和非公平锁中的通用操作，也就是释放锁的操作 tryRelease 。

&emsp;&emsp;  tryRelease 的实现很简单，主要就是依赖于 AQS 中的 state 属性，如果state 值减去要释放的信号量为 0 则释放成功，否则失败。

```java
        // 释放锁的公共操作
        protected final boolean tryRelease(int releases) {
            // 释放锁首先就是使用 AQS 中的 state 的值减去信号量 判断是否为0
            // 如果是 0 则表明成功释放锁，独占线程设为 null，否则说明还占用锁
            int c = getState() - releases;
            // 必须获取到锁才能解锁，否则抛异常
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```

### 2. fairSync
    &emsp;&emsp;  公平锁执行 lock 操作就是执行了 AQS 中的 acquire(1) 也就是请求一个锁资源。但是注意，在 AQS 中的 acquire 中的 tryAcquire 方法没有实现，所以必须由当前类实现。
    
    &emsp;&emsp;  在 tryAcquire 中做的事情就是看是否有代码在临界区。没有则还要看同步队列中是否有线程等待，当只有这一个线程在获取锁的时候才能正常的获取锁，其他情况都失败。
    
```java
// 公平锁
    static final class FairSync extends Sync {
        final void lock() {
            acquire(1);
        }

        // 没有代码在临界区或者是当前线程的重入 则获取成功，否则失败
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            // 如果当前线程在获取锁的过程没有其他线程在临界区
            if (c == 0) {
                // 如果同步队列中没有等待的线程，就设置 state ，并且当前线程设为独占线程
                if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            // 有程序在临界区，如果是当前线程可重入，加上请求的资源数
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            // 竞争锁失败，因为他是公平的锁竞争
            return false;
        }
    }
```
### 3. noFairSync
同理，这个方法也需要实现 lock 和 tryAcquire 操作。在 lock 中直接判断是否有代码在临界区，没有则直接获取到锁，与公平锁不同的是：公平锁还判断了等待队列中是否有等待的线程。有在临界区的情况时执行 acquire 操作。同样的，首先要执行 tryAcquire 如果失败，加入同步队列并自旋获取锁。还是 tryAcquire 的实现，这里又调用了 nonfairTryAcquire。


```java
    // 非公平锁
    static final class NonfairSync extends Sync {
        final void lock() {
            // 如果没有代码在临界区 直接获取锁，独占
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
            // 有代码在临界区则执行尝试获取锁
                acquire(1);
        }

        // 和公平锁中的 tryAcquire 一模一样只是少了关于同步队列中是否有等待线程的判断
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
    
    final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            // 没有线程获取锁 直接获取到锁  和公平锁中的 tryAcquire 一模一样只是少了关于同步队列的判断
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            // 重入锁
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

&emsp;&emsp;  好了，现在我们 AQS 中的空的核心方法也被子类实现了，那么现在 fairSync 和 noFairSync 就算是一个完整的 AQS 了。此时看一下加解锁的流程。

只说公平锁，因为非公平锁就只是少了一个判断。

1. 首先 sync 调用 lock 方法，让后 lock 调用了 AQS 的 acquire(1) 也就是获取一个锁资源。

2. acquire 就先调用 tryAcquire(1) 尝试获取锁，这时候代码又回调到 sync 中的实现的 tryAcquire 方法，这个方法先判断锁是否已经被别的线程使用，然后需要确定没有更早的线程在同步队列等待获取锁，才把当前线程设置为独占线程，并设置 state 值获取锁。但是如果有代码在临界区需要判断是否为当前线程，因为锁是可重入的。如果是当前线程则 state 加上请求锁的个数，返回。

3. 这时候又回到 AQS 中，如果上面尝试获取锁的过程失败，就需要调用 addWaiter 将当前线程封装成一个独占节点，等待状态默认为 0，并且返回当前节点。

4. 加入同步队列后，再调用 acquireQueued 方法，当此线程是同步队列中等待的第一个线程则自旋尝试获取锁，毕竟很可能正在执行的线程马上就会释放锁了，再进行休眠不合适。如果自旋获取锁失败则判断节点状态是否为 SIGNAL 然后执行等待操作。

5. 锁获取成功则把当前节点设置为头结点，把 thread = null

6. 至此，Acquire 方法执行结束。

然后调用 unlock 方法解锁操作。

1. 解锁操作就没那么麻烦，首先还是调用到了 AQS 中的 release 方法，这个方法首先尝试解锁当前线程，又回调到了 sync 中的 tryRelease 。

2. tryRelease 逻辑比较简单，使用 AQS 中的 state 减去释放的资源数，等于 0 代表完全释放，否则释放失败。

3. 如果 tryRelease 成功执行就要去唤醒同步队列中的后继节点，继续执行。

4. 至此，release 方法执行完毕。


## 4. AQS 中的要方法
### 1. get/setState
&emsp;&emsp;这两个方法主要是对 state 变量的 volatile 的读写，其实里面就就是普通的 get/set 方法。但是注意的一点就是 state 是 volatile 的。

```java
    // 对状态变量的 volatile 读写
    protected final int getState() {
        return state;
    }
    protected final void setState(int newState) {
        state = newState;
    }
```
### 2. release/tryRelease/unparkSuccessor/fullyRelease
&emsp;&emsp;  这几个方法在一起说主要是因为他们之间存在调用链，首先来看 release 这个方法我们在上面也分析了，里面调用了 tryRelease 、unparkSuccessor。 也就是首先调用 tryRelease 来释放当前线程的锁，如果释放成功就调用 unparkSuccessor 来唤醒同步队列中后继节点。其中 tryRelease 是由子类来实现，里面的主要逻辑就是看当前的 state 变量的值在修改过后是否为0 。这里还有一个 fullRelease 主要是在 ConditionObject 中调用的，当执行 await 的操作的时会执行此方法释放锁。

```java
 //  尝试释放锁
    public final boolean release(int arg) {
        // 如果释放锁成功 唤醒同步队列中的后继节点
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
    
    // 唤醒同步队列中的后继节点
    private void unparkSuccessor(Node node) {
        // node 一般就是当前正在运行的线程
        int ws = node.waitStatus;
        // 当前线程置为初始状态   可以失败
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        // 找到同步队列中的下一个节点
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {  //没有下一个节点或者被取消
            s = null;
            // 从后往前找第一个没有被取消的线程
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        // 唤醒那个线程
        if (s != null)
            LockSupport.unpark(s.thread);
    }
    
    final int fullyRelease(Node node) {
        boolean failed = true;
        try {
            int savedState = getState();
            if (release(savedState)) {
                failed = false;
                return savedState;
            } else {
                throw new IllegalMonitorStateException();
            }
        } finally {
            if (failed)
                node.waitStatus = Node.CANCELLED;
        }
    }
```
### 3. acquire/tryAcquire/addWaiter/acquireQueued
这个和上面的一样，在执行了 acquire 后，会去调用子类复写的 tryAcquire 方法，这个方法就是看有否有代码块在临界区，没有的话直接获取锁（非公平锁），设置 state，有的话要判断是不是当前线程能否进行重入操作，否则就获取失败。失败后会调用 addWaiter ，new 一个新的节点加入到同步队列，接着调用了 acquireQueued 如果这个节点是同步队列中的第一个等待的线程（但不是第一个节点，因为第一个节点是 thread=null 的运行中的线程）就自旋一段时间看能否获取到锁。不能则 park 等待。

```java
// 获取锁
    public final void acquire(int arg) {
        // 尝试获取锁 失败则加入同步队列 如果是同步队列中的第一个线程就自旋获取锁
        // 上面的步骤的自旋获取锁阶段，返回的是是否需要中断，所以下面就进行 selfInterrupt
        // tryAcquire 是模板方法，因为对于公平锁和非公平锁获取锁方式不同
        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    
    
    // 创建一个节点放入到同步对列中   可传入是否为独占锁   返回当前节点
    private Node addWaiter(Node mode) {
        // 默认的 status 是 0
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            // 把 tail 设置为 node 成功说明没有竞争
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        // 失败则就说明空队列   创建头结点
        enq(node);
        return node;
    }
    
     final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            // 自旋获取锁
            for (;;) {
                // 获取前驱节点
                final Node p = node.predecessor();
                // 如果前驱是空的头结点，那么也就是说当前线程就是队列中的第一个线程 并尝试获取锁  成功的话方法返回中断情况
                if (p == head && tryAcquire(arg)) {
                    // 把当前节点设置为头结点  thread=null 也就可以看做当前线程在运行，所以就不在同步队列
                    setHead(node);
                    // gc
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                // 如果获取锁失败，检测为 SIGNAL 或者设置为 SIGNAL 然后让此线程等待 等待操作在 parkAndCheckInterrupt 中完成
                if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            // 失败 取消
            if (failed)
                cancelAcquire(node);
        }
    }
```
## 5. 总结
&emsp;&emsp;  其实到这里 ReentrantLock 已经讲完了，因为他底层全部调用的是 Sync 中的方法，也就是全都是调用了 AQS 中的方法。而 AQS 中的大部分重要的方法都已经看过了。


