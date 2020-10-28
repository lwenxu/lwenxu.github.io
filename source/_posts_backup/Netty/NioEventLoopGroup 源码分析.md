---
title: NioEventLoopGroup 源码分析
date: 2018-04-07 21:40:24
tags:
categories:
  - Netty 源码分析
---
# NioEventLoopGroup 源码分析

> #### 1. 在阅读源码时做了一定的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限。为了方便 IDE 查看、跟踪、调试 代码，所以在 [github](https://github.com/lwenxu/nettySourceCode) 上提供 netty 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> #### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

&emsp;&emsp;  从今天开始，就准备进军 ne    tty 了，主要的想法是看看 netty4 中一些比较重要的实现，也就是能经常出现在我们面前的东西。主要是： 线程池、通道、管道、编解码器、以及常用的工具类。

&emsp;&emsp;  然后现在看源码应该不会像之前的 jdk 那么细致了，主要是看了一个类以后就发现 netty 对代码封装太强了，基本一个功能可能封装了七八个类去实现，很多的抽象类但是这些抽象类中的功能还非常的多。所以说主要看这个流程，以及里面写的比较好的代码或者比较新的思想会仔细的去看看。具体的子字段，每个方法不可能做到那么细致。
<!-- more -->
&emsp;&emsp;  好，正式开始 netty 源码征战 ！

## 1. 基本思路
&emsp;&emsp;  这里首先讲一下结论，也就是先说我看这个类的源码整理出来的思路，主要就是因为这些类太杂，一个功能在好几个类中才完全实现。

&emsp;&emsp;  我们在 new 一个 worker/boss 线程的时候一般是采用的直接使用的无参的构造方法，但是无参的构造方法他创建的线程池的大小是我们 CPU 核心的 2 倍。紧接着就需要 new 这么多个线程放到线程池里面，这里的线程池采用的数据结构是一个数组存放的，每一个线程需要设置一个任务队列，显然任务队列使用的是一个阻塞队列，这里实际采用的是 `LinkedBlockQueue` ，然后回想一下在 jdk 中的线程池是不是还有一个比较重要的参数就是线程工厂，对的！这里也有这个东西，他是需要我们手动传入的，但是如果不传则会使用一个默认的线程工厂，里面有一个 `newThread` 方法，这个方法实现基本和 jdk 中的实现一模一样，就是创建一个级别为 5 的非 Daemon 线程。对这就是我们在创建一个线程池时候完成的全部工作！

&emsp;&emsp;  好现在来具体说一下，我们每次创建的是 `NioEventLoopGroup` 但是他又继承了 n 个类才实现了线程池，也就是线程池的祖先是 `ScheduledExecutorService` 是 jdk 中的线程池的一个接口，其中里面最重要的数据结构就是一个 children 数组，用来装线程的。

&emsp;&emsp;  然后具体的线程他也是进行了封装的，也就是我们常看到的 `NioEventLoop` 。这个类里面有两个比较重要的结构：taskQueue 和 thread 。很明显这个非常类似 jdk 中的线程池。

## 2. NioEventLoopGroup 线程池分析
&emsp;&emsp;  首先要创建线程池，传入的线程数为 0，他是一直在调用 `this()` 最后追溯到 `super(nThreads,threadFactory,selectorProvider)`  也就是使用了 `MultithreadEventLoopGroup` 的构造方法，在这一步确定了当传入的线程数为 0 时应该设置的线程数为 CPU 核心的两倍。然后再次上调，调用了 `MultithreadEventExecutorGroup` 的构造方法，在这里才是真正的开始了线程池的初始化。

&emsp;&emsp;  首先设置了线程池工厂，然后初始化 chooser ，接着创建 n 个线程放到 children 数组中，最后设置线程中断的监听事件。


```java
/**
     * 这个方法流程：
     * 1、设置了默认的线程工厂
     * 2、初始化 chooser
     * 3、创建nTreads个NioEventLoop对象保存在children数组中
     * 4、添加中断的监听事件
     * @param nThreads
     * @param threadFactory
     * @param args
     */
    protected MultithreadEventExecutorGroup(int nThreads, ThreadFactory threadFactory, Object... args) {
        if (nThreads <= 0) {
            throw new IllegalArgumentException(String.format("nThreads: %d (expected: > 0)", nThreads));
        }

        // 默认使用线程工厂是 DefaultThreadFactory
        if (threadFactory == null) {
            threadFactory = newDefaultThreadFactory();
        }

        children = new SingleThreadEventExecutor[nThreads];
        // 二的平方的实现是看  n&-n==n
        //根据线程个数是否为2的幂次方，采用不同策略初始化chooser
        if (isPowerOfTwo(children.length)) {
            chooser = new PowerOfTwoEventExecutorChooser();
        } else {
            chooser = new GenericEventExecutorChooser();
        }
        //产生nTreads个NioEventLoop对象保存在children数组中
        for (int i = 0; i < nThreads; i ++) {
            boolean success = false;
            try {
                children[i] = newChild(threadFactory, args);
                success = true;
            } catch (Exception e) {
                // TODO: Think about if this is a good exception type
                throw new IllegalStateException("failed to create a child event loop", e);
            } finally {
                // 没成功，把已有的线程优雅关闭
                if (!success) {
                    for (int j = 0; j < i; j ++) {
                        children[j].shutdownGracefully();
                    }
                    // 没有完全关闭的线程让它一直等待
                    for (int j = 0; j < i; j ++) {
                        EventExecutor e = children[j];
                        try {
                            while (!e.isTerminated()) {
                                e.awaitTermination(Integer.MAX_VALUE, TimeUnit.SECONDS);
                            }
                        } catch (InterruptedException interrupted) {
                            Thread.currentThread().interrupt();
                            break;
                        }
                    }
                }
            }
        }

        // 对每一个 children 添加中断线程时候的监听事件，就是将 terminatedChildren 自增
        // 判断是否到达线程总数，是则更新 terminationFuture
        final FutureListener<Object> terminationListener = new FutureListener<Object>() {
            @Override
            public void operationComplete(Future<Object> future) throws Exception {
                if (terminatedChildren.incrementAndGet() == children.length) {
                    terminationFuture.setSuccess(null);
                }
            }
        };

        for (EventExecutor e: children) {
            e.terminationFuture().addListener(terminationListener);
        }
    }
```

&emsp;&emsp;  其中有一个 if 分支用来初始化 chooser ，这个 chooser 就是用来选择使用哪个线程来执行哪些操作的。这里用到了判断一个数是否为 2 的次幂的一个方法 `isPowerOfTwo()`  实现比较有意思，贴出来。

```java
    private static boolean isPowerOfTwo(int val) {
        return (val & -val) == val;
    }
```

&emsp;&emsp;  接下来目光要转向 `newChild(threadFactory, args)` ，因为在这个类里面这个方法是抽象的，在 `NioEventLoopGroup` 得到了实现。其实看到了也非常的简单粗暴，直接 new 了一个 `NioEventLoop` ，接下来就应该分析这个线程的包装类了。

```java
    @Override
    protected EventExecutor newChild(
            ThreadFactory threadFactory, Object... args) throws Exception {
        // 这里才是重点  也就是真正的线程   被放在自己的 children 数组中
        return new NioEventLoop(this, threadFactory, (SelectorProvider) args[0]);
    }
```
## 3. NioEventLoop 线程分析

&emsp;&emsp;  上面已经看到了，`newChild` 方法就是 new 了一个 `NioEventLoop` 。所以有必要好好看看这个线程包装类。

&emsp;&emsp;  这个类的构造方法是调用了父类 `SingleThreadEventLoop` 的构造,接着继续上调 `SingleThreadEventExecutor` 构造，在这个类中才真正的实现了线程的构造。里面就做了两件事 ： 

1. new 了一个新的线程，新的线程还分配了一个任务，任务的内容就是调用本类中的一个 run 方法，在 `NioEventLoop` 中实现。

2. 设置任务队列为 `LinkedBlockQueue`

```java
/**
     * 构造方法主要完成了：
     * 1、new 一个新的线程执行一个 run 方法
     * 2、用 LinkedBlockQueue 初始化 taskQueue
     * @param parent
     * @param threadFactory
     * @param addTaskWakesUp
     */
    protected SingleThreadEventExecutor(EventExecutorGroup parent, ThreadFactory threadFactory, boolean addTaskWakesUp) {

        if (threadFactory == null) {
            throw new NullPointerException("threadFactory");
        }

        this.parent = parent;
        this.addTaskWakesUp = addTaskWakesUp;
        // new 了一个新的线程
        thread = threadFactory.newThread(new Runnable() {
            @Override
            public void run() {
                boolean success = false;
                updateLastExecutionTime();
                try {
                    // 调用一个 run 方法
                    SingleThreadEventExecutor.this.run();
                    success = true;
                } catch (Throwable t) {
                    logger.warn("Unexpected exception from an event executor: ", t);
                } finally {
                    // 让线程关闭
                    for (;;) {
                        int oldState = STATE_UPDATER.get(SingleThreadEventExecutor.this);
                        if (oldState >= ST_SHUTTING_DOWN || STATE_UPDATER.compareAndSet(
                                SingleThreadEventExecutor.this, oldState, ST_SHUTTING_DOWN)) {
                            break;
                        }
                    }
                    // Check if confirmShutdown() was called at the end of the loop.
                    if (success && gracefulShutdownStartTime == 0) {
                        logger.error(
                                "Buggy " + EventExecutor.class.getSimpleName() + " implementation; " +
                                SingleThreadEventExecutor.class.getSimpleName() + ".confirmShutdown() must be called " +
                                "before run() implementation terminates.");
                    }

                    try {
                        // Run all remaining tasks and shutdown hooks.
                        for (;;) {
                            if (confirmShutdown()) {
                                break;
                            }
                        }
                    } finally {
                        try {
                            cleanup();
                        } finally {
                            STATE_UPDATER.set(SingleThreadEventExecutor.this, ST_TERMINATED);
                            threadLock.release();
                            if (!taskQueue.isEmpty()) {
                                logger.warn("An event executor terminated with non-empty task queue (" + taskQueue.size() + ')');
                            }
                            terminationFuture.setSuccess(null);
                        }
                    }
                }
            }
        });

        // 使用 LinkedBlockQueue 初始化 taskQueue
        taskQueue = newTaskQueue();
    }
```

&emsp;&emsp;  然后看一下他要执行的 run 方法在 `NioEventLoop` 中得到了实现。

```java
/**
     *'wakenUp.compareAndSet(false, true)' 一般都会在 select.wakeUp() 之前执行
     * 因为这样可以减少 select.wakeUp() 调用的次数，select.wakeUp() 调用是一个代价
     * 很高的操作
     * 注意：如果说我们过早的把 wakenUp 设置为 true，可能导致线程的竞争问题，过早设置的情形如下：
         1) Selector is waken up between 'wakenUp.set(false)' and
            'selector.select(...)'. (BAD)
         2) Selector is waken up between 'selector.select(...)' and
            'if (wakenUp.get()) { ... }'. (OK)
     在第一种情况中 wakenUp 被设置为 true 则 select 会立刻被唤醒直到 wakenUp 再次被设置为 false
     但是wakenUp.compareAndSet(false, true)会失败，并且导致所有希望唤醒他的线程都会失败导致
     select 进行不必要的休眠

     为了解决这个问题我们是在 wakenUp 为 true 的时候再次对 select 进行唤醒。
     */

    @Override
    protected void run() {
        for (;;) {
            // 获取之前的线程状态，并让 select 阻塞
            boolean oldWakenUp = wakenUp.getAndSet(false);
            try {
                // 有任务在线程创建之后直接开始 select
                if (hasTasks()) {
                    selectNow(); //直接调用了 select 的 selectNow 然后再次唤醒同下面的代码
                // 没有任务
                } else {
                    // 自旋进行等待可进行 select 操作
                    select(oldWakenUp);
                    // 再次唤醒，解决并发问题
                    if (wakenUp.get()) {
                        selector.wakeup();
                    }
                }

                cancelledKeys = 0;
                needsToSelectAgain = false;
                final int ioRatio = this.ioRatio;
                // 都是处理 selected 的通道的数据，并执行所有的任务，只是在 runAllTasks 传的参数不同
                if (ioRatio == 100) {
                    processSelectedKeys();
                    runAllTasks();
                } else {
                    final long ioStartTime = System.nanoTime();
                    processSelectedKeys();
                    final long ioTime = System.nanoTime() - ioStartTime;
                    runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
                }

                if (isShuttingDown()) {
                    closeAll();
                    if (confirmShutdown()) {
                        break;
                    }
                }
            } catch (Throwable t) {
                logger.warn("Unexpected exception in the selector loop.", t);
                // Prevent possible consecutive immediate failures that lead to
                // excessive CPU consumption.
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    // Ignore.
                }
            }
        }
    }
```
&emsp;&emsp;  紧接着就是分析这个 run 方法，也就是线程在被创建之后进行的一系列操作。里面主要做了三件事：

1. 进行 select 
2. 处理 selectedKeys
3. 唤醒队列中所有的任务

&emsp;&emsp;  上面的操作都是在一个循环里面一直执行的，所以说 `NioEventLoop` 这个线程的作用就只有一个那就是：进行任务处理。在这个线程被 new 出来时我们就给他分配了线程的任务就是永不停歇的进行上面的操作。

&emsp;&emsp;  上面的过程说的是有线程安全问题，也就是如果我们过早的把 wakenUp 设置为 true，我们的 select 就会苏醒过来，而其他的线程不清楚这种状态想要设置为 wakenUp 的时候都会失败，导致 select 休眠。主要感觉有点是因为这个东西不是线程间可见的，要是采用 volatile 可能就会解决这个问题，但是 wakenUp 是 final 的不能使用 volatile 关键字修饰。所以作者采用的解决方案就是再次手动唤醒，防止由于其他线程并发设置 wakenUp 的值导致的不必要的休眠。

&emsp;&emsp;  然后要说一下 select 方法，这个方法的调用主要因为在队列中没有任务，所以就暂时不用 select ，这个方法里面做的就是自旋的去 select ，没有任务就 等待一段时间再去 select。

```java
/**
     * 这个方法主要干的事情：
     * 1、如果不需要等待就直接 select
     * 2、需要等待则等一个超时时间再去 select
     * 这个过程是不停进行的也就是死循环直达有任务可进行 select 时 select 完毕退出循环
     * @param oldWakenUp
     * @throws IOException
     */
    private void select(boolean oldWakenUp) throws IOException {
        Selector selector = this.selector;
        try {
            int selectCnt = 0;
            long currentTimeNanos = System.nanoTime();
            long selectDeadLineNanos = currentTimeNanos + delayNanos(currentTimeNanos);
            for (;;) {
                // 不用等待进行一次 select 操作
                long timeoutMillis = (selectDeadLineNanos - currentTimeNanos + 500000L) / 1000000L;
                if (timeoutMillis <= 0) {
                    if (selectCnt == 0) {
                        selector.selectNow();
                        selectCnt = 1;
                    }
                    break;
                }
                // 等一个超时再去选择
                int selectedKeys = selector.select(timeoutMillis);
                selectCnt ++;

                if (selectedKeys != 0 || oldWakenUp || wakenUp.get() || hasTasks() || hasScheduledTasks()) {
                    // - Selected something,
                    // - waken up by user, or
                    // - the task queue has a pending task.
                    // - a scheduled task is ready for processing
                    break;
                }
                if (Thread.interrupted()) {
                    // Thread was interrupted so reset selected keys and break so we not run into a busy loop.
                    // As this is most likely a bug in the handler of the user or it's client library we will
                    // also log it.
                    //
                    // See https://github.com/netty/netty/issues/2426
                    if (logger.isDebugEnabled()) {
                        logger.debug("Selector.select() returned prematurely because " +
                                "Thread.currentThread().interrupt() was called. Use " +
                                "NioEventLoop.shutdownGracefully() to shutdown the NioEventLoop.");
                    }
                    selectCnt = 1;
                    break;
                }

                long time = System.nanoTime();
                if (time - TimeUnit.MILLISECONDS.toNanos(timeoutMillis) >= currentTimeNanos) {
                    // timeoutMillis elapsed without anything selected.
                    selectCnt = 1;
                } else if (SELECTOR_AUTO_REBUILD_THRESHOLD > 0 &&
                        selectCnt >= SELECTOR_AUTO_REBUILD_THRESHOLD) {
                    // The selector returned prematurely many times in a row.
                    // Rebuild the selector to work around the problem.
                    logger.warn(
                            "Selector.select() returned prematurely {} times in a row; rebuilding selector.",
                            selectCnt);

                    rebuildSelector();
                    selector = this.selector;

                    // Select again to populate selectedKeys.
                    selector.selectNow();
                    selectCnt = 1;
                    break;
                }

                currentTimeNanos = time;
            }

            if (selectCnt > MIN_PREMATURE_SELECTOR_RETURNS) {
                if (logger.isDebugEnabled()) {
                    logger.debug("Selector.select() returned prematurely {} times in a row.", selectCnt - 1);
                }
            }
        } catch (CancelledKeyException e) {
            if (logger.isDebugEnabled()) {
                logger.debug(CancelledKeyException.class.getSimpleName() + " raised by a Selector - JDK bug?", e);
            }
            // Harmless exception - log anyway
        }
    }
```

&emsp;&emsp;  接着就是 `processSelectedKeys();` 和`runAllTasks();` 这两个方法，前一个方法不说就是和我们写 Nio 的时候的步骤差不多，遍历 selectedKeys 处理，然后 `runAllTasks()` 执行所有的任务的 run 方法。


```java
protected boolean runAllTasks() {
        fetchFromDelayedQueue();
        Runnable task = pollTask();
        if (task == null) {
            return false;
        }

        // 这个循环就是用来循环任务队列中的所有任务
        for (;;) {
            try {
                task.run();
            } catch (Throwable t) {
                logger.warn("A task raised an exception.", t);
            }

            task = pollTask(); // 循环条件
            if (task == null) {
                lastExecutionTime = ScheduledFutureTask.nanoTime();
                return true;
            }
        }
    }
```

## 4. 总结
&emsp;&emsp;  好了其实到这里线程池其实分析的已经差不多了，对于很多的细节问题并没有仔细的去看，单丝我们清楚流程以及里面的结构基本就差不多了。

&emsp;&emsp;  在 `NioEventLoopGroup` 中包装了 `NioEventLoop` 线程任务。具体包装在了 children 数组中，然后使用 newThread 工厂创建线程，接着给线程分配任务，任务就是进行 select 操作。


我的博客即将搬运同步至腾讯云+社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan

