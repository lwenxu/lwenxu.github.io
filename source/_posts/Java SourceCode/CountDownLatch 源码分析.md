---
title: CountDownLatch 源码分析
date: 2018-04-3 20:05:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---
# CountDownLatch 源码分析

> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1. 基本介绍
&emsp;&emsp;  `Latch` 这个单词的意思就是 “闭锁” ，这也是 jdk1.5 引入的一个并发组件，名字听上去并没有让我们对他的功能并没有什么直观的感受。先粗略解释一下他的功能：我们定义了一个带有数值 n 的 Condition 对象，然后我们调用这个对象上的 `await` 方法，如果说我们要这个线程继续执行，我们不是调用 `signal` 而是调用 n 次 `countDown` 方法。这样等待在这个 Condition 上的所有线程都将被唤醒。
<!-- more -->
&emsp;&emsp;  好先来举个例子理解一下上面的文字。


```java
public class CountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch1 = new CountDownLatch(1);
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                try {
                    latch1.await();
                    System.out.println(Thread.currentThread().getName());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
        System.out.println("main must done something !");
        Thread.sleep(1000);
        latch1.countDown();
    }
}
```

先打印了

```
main must done something !
```
隔了一秒钟打印了

```
Thread-0
Thread-1
Thread-2
Thread-3
Thread-4
Thread-5
Thread-8
Thread-7
Thread-6
Thread-9
```

&emsp;&emsp;  很明显，这个组件非常适合我们在一些线程开始之前需要完成 n 件其他的准备工作，每完成一件我们就执行一次 `countDown()` 也就是计数值减一，唯有在计数值为 0 时我们的线程才会被唤醒继续执行。

## 2. 结构
&emsp;&emsp;  好的，先给一张类图看看他的结构。
![](http://ojrw3x8j2.bkt.clouddn.com/18-4-3/35332474.jpg)

&emsp;&emsp;  毫无疑问，底层还是采用了 AQS 作为基础并发组件。并且这个类要比我们想像中的简单的多。这个类没有继承和实现任何的接口及父类，他但一个独立的组件，里面就封装了 AQS。

### 3. 主要方法分析
&emsp;&emsp;  其实这这个类里面只有两个有价值的方法，就是 await 和 countDown ，但是这两个方法全都是委托给了 AQS 的 acquireSharedInterruptibly 和 releaseShared 。好了又回到了 AQS ，那么上面的这个 Sync 肯定需要对 tryAcquireShared 和 tryReleaseShared 进行重写。所以说我们只需要分析两个方法。

#### 1. tryAcquireShared
&emsp;&emsp;  逻辑超简单，当 state 变量为 0 就返回 1 ，否则返回 -1 也就是失败。很明显我们调用了 await 肯定是当时的 state 值不为 0，自然会阻塞，并尝试唤醒后面的线程。

```java
 protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }
```
#### 2. tryReleaseShared
countDown 就是让信号量减一操作，所以说他就是进行了减一操作，当为 0 的时候，就会触发唤醒后继节点的操作。

```java
protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
```


