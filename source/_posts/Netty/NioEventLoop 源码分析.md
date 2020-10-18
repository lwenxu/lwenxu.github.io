---
title: NioEventLoop 源码分析
date: 2018-04-16 17:40:24
tags:
categories:
  - Netty 源码分析
---
# NioEventLoop 源码分析
## 1. SingleThreadEventExecutor 的 execute 方法 
NioEventLoop 的核心就在于它的 `run()` 他是在第一次添加任务的时候开始执行。那我们先看看第一次添加任务的地方，其实第一次添加任务的地方是在父类中的 `execute()` 方法。所以先去分析一下 `SingleThreadEventExecutor` 的`execute()` 方法。我把代码精简了贴出来，只看核心的部分。

```java
public void execute(Runnable task) {
        boolean inEventLoop = inEventLoop();
        // 线程已经启动  直接加入任务
        if (inEventLoop) {
            addTask(task);
        // 否则先启动任务 再添加任务
        } else {
            startThread();
            addTask(task);
        }

    }
```
&emsp;&emsp;  很明显，也就是我们往线程池中添加任务的时候，首先要看看我们的线程是不是已经启动了，没有的话首先我们需要启动一下线程。接下来要看看 `startThread()` 方法干了什么事。里面做了一些检查也就是线程只能被 `start` 一次，然后直接调用了 `NioEventLoop` 封装的 `thread` 的 `start()` 很简单！

&emsp;&emsp;  但是，等等这个线程是从哪来的？我们并没有显示的传入，来到 `SingleThreadEventExecutor` 构造方法，我们会发现他在构造器中进行了初始化，但是不是直接 `new Thread` 而是使用了我们传的线程工厂，然后在工程里面 new 了这个线程需要执行的任务。

&emsp;&emsp;  他的任务就是先执行一下 run 方法，然而他的 run 方法是抽象的，自然就调用到子类去了，这也就解释了为什么说是第一次添加任务的时候调用了 `NioEventLoop` 的 `run` 方法。

&emsp;&emsp;  贴一下对 `thread` 初始化的代码(精简过后的)：

```java
    // new 了一个新的线程
        thread = threadFactory.newThread(new Runnable() {
            @Override
            public void run() {
                boolean success = false;
                updateLastExecutionTime();
                try {
                   // 调用了 run 方法，这个 run 方法在这个类中是抽象的，显然在子类中实现了
                    SingleThreadEventExecutor.this.run();
                    success = true;
                } catch (Throwable t) {
                    logger.warn("Unexpected exception from an event executor: ", t);
                } finally {
                    // 让线程关闭
                }
        });
```

## 2. 再回到 NioEventLoop 的 run 方法
&emsp;&emsp;  那好，我们在上面已经看到了我们在创建一个 `NioEventLoop` 的时候会创建一个线程，这个线程的任务就是去调用子类的 run 方法。当我们执行 `execute( task )` 方法，添加一个新任务去运行的时候，就会判断当前线程是不是启动了，否则启动我们一开始创建的那个线程。用一张图说明一下！！！
[![kUQnB.md.png](https://s1.ax2x.com/2018/04/16/kUQnB.md.png)](https://simimg.com/i/kUQnB)

&emsp;&emsp;  好的现在正式的看一下 run 方法，还是贴一下核心代码：

```java
protected void run() {
        for (;;) {

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

                           
        }
    }
```

&emsp;&emsp;  可以看到在代码里面的死循环中值做了三件事：select、processSelectedKeys、 runAllTasks .借一张图来看：

[![kUjcp.md.png](https://s1.ax2x.com/2018/04/16/kUjcp.md.png)](https://simimg.com/i/kUjcp)

#### 1. 首先轮询注册到reactor线程对用的selector上的所有的channel的IO事件
#### 2. 处理产生网络IO事件的channel
#### 3. 处理任务队列

具体做的事情放到下面一一道来！
### 1. select()
&emsp;&emsp;  如果有任务的话直接去 selectNow() 也就是不进行等待的 select() ,而没有任务的时候就进行自旋等待的 select() 。下面是 select() 的核心代码,可以看待里面调用了 selectNow() 所以说这个就是一个自旋的 selectNow() 。

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


             if (SELECTOR_AUTO_REBUILD_THRESHOLD > 0 &&
                        selectCnt >= SELECTOR_AUTO_REBUILD_THRESHOLD) {
                    // 解决死循环问题，重建 selector
                    rebuildSelector();
                    selector = this.selector;
                    //  直接是 selectNow()
                    selector.selectNow();

                }
            }
```
`wakenUp` 表示是否应该唤醒正在阻塞的 select 操作，netty在进行一次新的loop之前，都会将 wakeUp 被设置成false，标志新的一轮loop的开始。
### 2. processSelectedKeys()
### 3. runAllTasks()


