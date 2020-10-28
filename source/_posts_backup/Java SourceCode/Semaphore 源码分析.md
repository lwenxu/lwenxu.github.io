---
title: Semaphore 源码分析
date: 2018-03-31 21:35:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---

# Semaphore 源码分析
> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1. Semephore 简单介绍
&emsp;&emsp;  一般来说，在 Java 中比较常用的同步工具就是 Lock 和 Synchronized 但是 Java 也加入了很多新的机制比如这里提到的信号量。他其实给人的感觉就是操作系统中的信号量，如果一个线程要运行需要获取一个资源进行一次 P 操作，在这里就是调用 acquire 方法，然后运行结束后调用 V 操作，释放这个资源以供其他线程使用，这里的 release 方法。那么如果资源都被其他线程抢光了，那么这个线程只能处于等待状态。也就是 P 操作返回的 -1 。

&emsp;&emsp;  上面我们所说的信号量是多值信号量，主要用于进程之间的同步。还有一种称之为二值信号量，也就是操作系统中常提到的 mutex 。对，此时他就是锁！因为做了一个 P 操作后，只有进行 V 操作其他线程才能进入临界区。此时和 Lock 作用一样了。
<!-- more -->
&emsp;&emsp;  下面写个简单的程序来感受一下这个同步工具。

```java
public class SemaphoreTest {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(2);
        for (int i = 0; i < 4; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName());
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }
            }).start();
        }
    }
}

```

最后输出的如下：
先打印了
```
Thread-0
Thread-1
```
隔了一秒打印了

```
Thread-2
Thread-3
```

&emsp;&emsp;  可以看到临界区内只有两个线程同时执行。其实当我们理解了上一篇文章中的 AQS 我们大概也能猜到 Semaphore 的实现原理，也是借助于 AQS 中的 state 属性，下面具体分析一下 Semaphore 的结构及原理。

## 2. 基本结构
### 1. 继承
    null
### 2. 实现
    Serializable  感觉好多类都能序列化啊！！有没有觉得。
### 3. 主要字段
    
```java
    private final Sync sync;
```

&emsp;&emsp;  好的，就一个锁字段，真干脆，那么证明前面的猜测也是对的，这个 Sync 肯定是一个内部类，继承了 AQS。在上面一篇文章中我们看到了一些 AQS 中的共享锁我没有分析，其实共享锁就是放在这使用的，所以在这篇文章中主要就是介绍 AQS 中剩下的共享锁部分。看一下整体结构。
![](http://ojrw3x8j2.bkt.clouddn.com/18-3-31/98562096.jpg)
&emsp;&emsp;  有没有觉得很熟悉，简直和 ReentrantLock 结构极其相似。Semaphore的内部类公平锁(FairSync)和非公平锁(NoFairSync)各自实现不同的获取锁方法即tryAcquireShared(int arg)，毕竟公平锁和非公平锁的获取稍后不同，而释放锁tryReleaseShared(int arg)的操作交由Sync实现，因为释放操作都是相同的，因此放在父类Sync中实现。

### 4. 方法概览
1. acquire()
2. release()
3. ctor-2
4. hasQueuedThreads() 
5. tryAcquire() 

## 3. 主要方法分析
### 1. ctor-2
&emsp;&emsp;  两个构造方法，分别指定了底层采用公平锁还是非公平锁！其中 permits 的值直接传给了 AQS 父类，也就是设置了 AQS 的 state 属性。

```java
//默认创建公平锁，permits指定同一时间访问共享资源的线程数
public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }

public Semaphore(int permits, boolean fair) {
     sync = fair ? new FairSync(permits) : new NonfairSync(permits);
 }
```

### 2. acquire 
&emsp;&emsp;  这个方法里面就直接调用了，acquireSharedInterruptibly(1) 。先不看代码，介绍一下过程：当线程调用了 acquire ， state 值代表的资源数足够使用，那么请求线程将会获得同步状态即对共享资源的访问权，并更新 state 的值(一般是对state值减1)，但如果state值代表的许可数已为0，则请求线程将无法获取同步状态，线程将被加入到同步队列并阻塞，直到其他线程释放同步状态(一般是对state值加1)才可能获取对共享资源的访问权。

&emsp;&emsp;  那么现在分析一下 acquireSharedInterruptibly 这个方法显然是可以被中断的，也就是说我们在 acquire 时也可以被中断。进入方法的第一件事就是判断是否有中断事件发生。然后调用了 tryAcquireShared 来获取共享锁。如果失败调用 doAcquireSharedInterruptibly 加入等待队列。 doAcquireSharedInterruptibly 这个方法真的和 acquireQueued 方法如出一辙，大部分代码是一样的！！！仅仅小部分不一样我已经注释出来了。

```java
// 这个方法真的是似曾相识，感觉就是 acquireQueued 的方法稍稍修改了一下
    /*
        1. 那个方法不抛异常
        2. 那个方法在调用前把节点加入了等待队列，封装了独占锁
        3. 那个方法设置标志位，这里直接抛异常

     */
    private void doAcquireSharedInterruptibly(int arg) throws InterruptedException {
        // 加入等待队列
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                // 前驱
                final Node p = node.predecessor();
                // 如果前驱是 head 。自旋获取锁  简直一样！！！
                // 就是写法不一样，意思一模一样
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                // 那个方法设置标志位，这里直接抛异常
                if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

&emsp;&emsp;  接着又是那个模板方法的问题了， tryAcquireShared ，在公平锁和非公平锁中的实现也不同。下面详细介绍一下两个的区别。在公平锁中还是和前面的 ReentrantLock 中的操作一样，先判断一下同步队列中是不是还有其他的等待线程，有则直接返回失败。否则对 state 值进行减操作并返回剩下的信号量。注意一点的是，如果我们请求的信号量大于可用信号量，返回的值是负值，而在 doAcquireSharedInterruptibly 中要求我们的方法返回正值才会继续执行。举个例子：当我们一开始有 2 信号量，而第一个线程直接请求了 10 个，这个线程就会被挂起，让其他线程来抢资源。并且如果发现请求后的资源数为负，他是不会对 state 进行更新的。

&emsp;&emsp;  然后再分析非公平锁，非公平锁直接调用了父类中的 nonfairTryAcquireShared 和 ReentrantLock 一样，调用父类的 nonfairTryAcquire ，这个方法比较简单，就是把上文中的 tryAcquireShared 里面的同步队列的判断删除了。注意一下，上面的这些方法都是死循环直到有条件对应时才会跳出循环。


```java
// 非公平锁的获取方式
protected int tryAcquireShared(int acquires) {
            return nonfairTryAcquireShared(acquires);
        }
final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
        
// 公平锁获取
protected int tryAcquireShared(int acquires) {
            for (;;) {
                if (hasQueuedPredecessors())
                    return -1;   // 仅仅多了这一行代码而已。
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 || compareAndSetState(available, remaining))
                    return remaining;
            }
        }
```

### 3. release
&emsp;&emsp;  release 方法，主要作用就是释放资源，提醒一下一定要保证 release 的执行，否则线程退出但是资源没有释放。所以一般这个代码写在 finally 中是最好的。并且获取多少资源就要释放多少资源，否则还是资源没被争确释放，举个例子 一开始执行了 acquire(10) 最后释放的时候不能只写一个 release() 而是 release(10) 才对。

&emsp;&emsp;  前面我们也看到了，其实 release 方法中调用的释放锁的操作肯定是一个公共操作，毕竟释放锁就不涉及公平和非公平一说了。在 release 中调用了 AQS 中的releaseShared 。这个方法毫无疑问首先会调用 tryReleaseShared 。这个又是模板方法等子类来实现，事实上就是在 Sync 中实现的。逻辑也很简单就是对 state 做增量操作。

&emsp;&emsp;  如果 tryReleaseShared 成功，执行 doReleaseShared ，就是唤醒后继线程。注意到前面的 ReentrantLock 中有 release 方法，他是直接尝试释放锁，如果成功就唤醒后继节点，两者逻辑基本一样。实际上 release 和 releaseShared 方法代码结构及逻辑完全一致，不同的就是在 tryAcquire 后的唤醒同步队列中的节点的操作不同。不然也就没必要弄两个方法了。


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
    // 为了方便对比把两个代码放在一块 可以看到 release 中的结构完全一样
    // 区别就在于 doReleaseShared 中有更多的判断操作
    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();  //在里面执行的 unparkSuccessor(h)
            return true;
        }
        return false;
    }
```

&emsp;&emsp;  重点看一下 doReleaseShared ，一个比较特殊的方法，由于共享的特性，在获取锁和释放锁的过程都需要唤醒后继节点，因为可以有多个线程同时进入临界区。这个方法的主要作用是：
 
 1. 在执行 acquireShared 申请资源时，执行 tryAcquireShared 失败则执行 doAcquireShared 方法将线程加入同步队列，然后判断同步队列中是否只有当前这一个等待线程，是则自旋获取锁，如果获取到了锁，就把当前节点设为头结点。然后 setHeadAndPropagate 继续唤醒后继节点，因为上次释放的资源可能特别多，现在的资源支持较多的线程同时执行，所以要唤醒后继节点。唤醒后继节点的条件是 `资源数 > 0 ` 或者 当前节点的后继节点的 `waitStatus < 0` 在这里只能是 `PROPAGATE` 或者 `SIGNAL`。这里也就解释了为什么在 doReleaseShared 中需要设置头为 PROPAGATE 状态，就是为了后续的唤醒。
 
 2. 而在执行 releaseShared 释放资源时，首先执行 tryReleaseShared ，如果成功则执行 doReleaseShared 释放后继节点，因为释放了资源让更多的线程来运行。如果说现在队列中没有任何节点在等待就把头结点设置为 PROPAGATE ，说明有过剩的资源可用。如果此时刚好遇到上面执行 acquireShared 需要唤醒后继节点的时候先要判断头结点的 `waitStatus` 的值，如果是 `PROPAGATE` 肯定可以唤醒后面的。如果说等待队列中有等待线程那么就唤醒他们就行。
 
 3. 好了这个 doReleaseShared 方法需要在以上两种情况下分析，否则始终不清楚为什么要设置 `PROPAGATE` 。

```java
// 这个方法中与 release 中的唤醒不同点在于他保证了释放动作的传递
    // 如果后继节点需要唤醒，则执行唤醒操作，如果没有后继节点则把头设置为 PROPAGATE
    // 这里的死循环和其他的操作中的死循环一样，为了检测新的节点进入队列
    // 其实这个方法比较特殊，在 acquireShared 和 releaseShared 中都被执行了，主要就是共享模式允许多个线程进入临界区
    private void doReleaseShared() {
        for (;;) {
            Node h = head;
            // 等待队列中有正在等待的线程
            if (h != null && h != tail) {
                // 获取头节点对应的线程的状态
                int ws = h.waitStatus;
                // 如果头节点对应的线程是SIGNAL状态，则意味着头结点正在运行，后继结点所对应的线程需要被唤醒。
                if (ws == Node.SIGNAL) {
                    // 修改头结点，现在后继节点要成为头结点了，状态设置初始值
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;
                    // 唤醒头结点h的后继结点所对应的线程
                    unparkSuccessor(h);
                }
                // 队列中没有等待线程，只有一个正在运行的线程。
                // 将头结点设置为 PROPAGATE 标志进行传递唤醒
                else if (ws == 0 && !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            // 如果头结点发生变化有一种可能就是在 acquireShared 的时候会调用 setHeadAndPropagate 导致头结点变化，则继续循环。
            // 从新的头结点开始唤醒后继节点。
            if (h == head)                   // loop if head changed
                break;
        }
    }

    //  被 acquireShard 调用
    private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; // Record old head for check below
        // 当前线程设为头结点
        setHead(node);
        // propagate 代表的是当前剩余的资源数，如果还有资源就唤醒后面的共享线程，允许多个线程获取锁，
        // 或者还有一个比较有意思的条件就是 h.waitStatus < 0 他其实是说 h.waitStatus 要么是 signal 要么是 propagate
        // 从而唤醒后继节点
        if (propagate > 0 || h == null || h.waitStatus < 0 || (h = head) == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }
```

## 4. 总结
&emsp;&emsp;  好了，其实到了这里基本就分析的差不多了，其他的一些很少用的方法，其实里面的写法类似，比如说不进行中断的判断等等。
&emsp;&emsp;  小结一下前面所说的，Semaphore 是一个信号同步工具，线程通过向 Semaphore 申请资源和释放资源，来进入临界区。一般情况下资源的个数比较多，所以我们允许多个线程同时进入临界区，而当我们资源个数只有一个的时候他就退化成锁也就是只允许一个线程进入临界区。但我们用的比较多的还是多个线程进入，毕竟要使用锁的话 Lock 可能做得更好。
&emsp;&emsp;  好，Semaphore 一般被认为是多个线程可同时进入临界区，那么我们底层的实现就不能采用独占锁，而是使用了共享锁，也就是 AQS 中的另外一部分 “江山” 共享锁的实现。资源数就直接采用了 AQS 中的 state 变量来维护。

&emsp;&emsp;  下面完整的跑一遍流程：

1. 使用构造方法创建一个 Semaphore 对象，默认底层 new 了一个非公平锁，传入的资源数被父类(AQS)的构造方法使用，初始化了 `state` 变量。

2. 调用 acquire 方法，这个方法直接调用了 `acquireSharedInterruptibly()` 他首先调用了 Sync 子类中的 tryAcquireShared（因为公平锁和非公平锁的缘故） 如果获取资源失败需要调用 `doAcquireSharedInterruptibly()` 这个方法和独占锁中的 `acquireQueued` 方法如出一辙，首先加入等待队列，然后如果等待队列中只有这一个等待的线程则自旋获取锁。如果获取到了锁并且还有可用就尝试唤醒他之后等待的线程，也就是调用了`setHeadAndPropagate()` ，否则的话直接等待。

3. 这里很巧妙的一点，当我们自旋获取到了锁就说明在自旋的这段时间中有线程释放了资源，然后我们发现资源数还是大于 0 ，那么可以让更多的线程运行（这个时候肯定是新的线程加入了同步队列，也就解释了为什么要使用死循环来执行这段代码），所以才调用了 `setHeadAndPropagate()` 这个方法里面不仅仅设置了头结点，还调用了 `doReleaseShared()` 这看起来是需要在 release 方法中调用的在 acquire 中也调用了。

4. 好了，现在如果调用 `release` 方法 ，他首先调用 `releaseShared` 之后，里面的逻辑是：先释放锁调用 `tryReleaseShared() ` ，如果成功需要唤醒同步队列中等待的线程。所以会调用 `doReleaseShared` 。这里面的代码我们已经说过好几次了，因为在很多地方都调用了他。

好，看懂了这些基本对于 Semaphore 了解的差不多了，对于 AQS 中的共享锁部分也算是有了一个完整的介绍。

