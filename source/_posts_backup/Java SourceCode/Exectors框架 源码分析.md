---
title: Exectors框架 源码分析
date: 2018-04-3 22:28:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---
# Exectors框架 源码分析

> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1. 基本结构
&emsp;&emsp;  由于 Exector 这个家族还是比较大的，所以先导出一下类图，对这个家族有一个大概的认识。
![](http://ojrw3x8j2.bkt.clouddn.com/18-4-3/89479215.jpg)
<!-- more -->

## 2. Exector
&emsp;&emsp;  可以看到，Exector 属于一个接口，其实它里面只有一个 `void execute(Runnable)` 方法。注意它里面的参数是 `Runnable` ，并且返回值是 void 类型。

## 3. ExectorService

&emsp;&emsp;  紧接着就是另外一个 `ExectorService` 接口，这个接口继承了上面的 `Exector` 但是他在原来的接口上添加了非常多的方法，其中有两类方法比较重要。 其中一类就是`submit()` 这里提供了三个重载的方法：

```java
    // 返回一个 Future 对象，表示将要执行的线程的状态
    // Future 对象的 get 方法会一直阻塞直到算出值
    <T> Future<T> submit(Callable<T> task);
    // Runnable 本来是不返回 Future 对象的，这里使用了一个参数传递
    <T> Future<T> submit(Runnable task, T result);
    // 普通的执行，类似于 execute 方法
    Future<?> submit(Runnable task);
```

&emsp;&emsp;  注意，这三个方法都是有返回值的，并且返回值都是一个 Future 对象，然后就是参数的问题，他们的参数可以是 Runnable 也可以是 Callable 。Runnable 我们都比较熟悉了，Callable 我们就可以理解成带有返回值的 Runnable。

&emsp;&emsp;  然后另外一类方法就是用来判断线程状态的方法，以及操作线程池中的线程的方法。

```java
    // 启动一次顺序关闭，执行以前提交的任务，但不接受新任务。
    void shutdown();
    // 尝试终止正在执行的线程，暂停处理正在等待的任务，并返回等待执行的任务列表。
    List<Runnable> shutdownNow();
    //当前线程结束
    boolean isShutdown();
    //全部结束
    boolean isTerminated();
```

## 4. AbstractExectorService
&emsp;&emsp;  虽然又是一个抽象类，但是他完全没有一个抽象方法，就像 AQS 一样。然后主要讨论这里面的几个重要方法。他增加了两个重载方法，然后实现了接口的三个 `submit()` 。

&emsp;&emsp;  先说 `submit()` 方法，这个方法如果接受的是 `Runnable` 参数的话就直接调用了 `newTaskFor(Runnable runnable, T value)` ，在前面会看到对于 `Runnable` 是有两个重载方法的，那么这里就给 `T` 传参不同，可以是 `null` 。然后另外一个传参为 `callable` 的就调用 `newTaskFor(Callable<T> callable)` 。可以看到，他们底层都是调用了 `execute` ，最后返回的还是 `RunnableFuture`。`RunnableFuture` 这个就是继承了 `Runnable` 和 `Future` 接口的一个空接口。

```java
    //  调用的都是 newTaskFor 的 Runnable 参数方法
    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }
    public <T> Future<T> submit(Runnable task, T result) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task, result);
        execute(ftask);
        return ftask;
    }
    //  调用的 newTaskFor 的 Callable 方法
    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }
```
&emsp;&emsp;  好，既然上面的方法都调用了 `newTaskFor` ，那么这个方法就是 `submit` 的执行基础，接着看一下这个方法。

```java
    // 这两个是 AES 的核心方法，主要就是创建任务  是 submit 的依赖
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }
```

&emsp;&emsp;  可以看到上面的方法全都是采用了 `FutureTask` 创建的新任务。返回后交给 `execute` 方法执行。那很明显， `FutureTask` 肯定是 `RunnableFuture` 的子类，确实 `FutureTask` 只实现了 `RunnableFuture` 这一个接口。

## 5. FutureTask 介绍
&emsp;&emsp;  在前面看到，这个类实现了 RunnableFuture 然后 RunnableFuture 又继承了 Runnable 和 Future ，这个名字取的还真是可以，直接把两个接口组合了。Runnable 我们知道他就只有一个 Run 方法，用来放任务代码。而 Future 又是干嘛的呢。我们来看一下里面的方法，比较少。

```java
    // 取消任务
    boolean cancel(boolean mayInterruptIfRunning);
    // 状态判断
    boolean isCancelled();
    boolean isDone();
    // 阻塞获取结果
    V get() throws InterruptedException, ExecutionException;
    // 超时阻塞获取结果
    V get(long timeout, TimeUnit unit)
```
&emsp;&emsp;  好的，其实就是在原来的任务的基础上加上了关于状态判断和结果获取的方法，主要是结果的阻塞获取是非常重要的，这也是 Runnable 的一大缺陷。

&emsp;&emsp; FutureTask 其实就是 RunnableFuture 的一个实现类。所以说我们大致的看一下这个实现类就好。

```java
    // 任务状态
    private volatile int state;
    private static final int NEW          = 0;
    private static final int COMPLETING   = 1;
    private static final int NORMAL       = 2;
    private static final int EXCEPTIONAL  = 3;
    private static final int CANCELLED    = 4;
    private static final int INTERRUPTING = 5;
    private static final int INTERRUPTED  = 6;
    // 任务
    private Callable<V> callable;
    // 结果
    private Object outcome; // non-volatile, protected by state reads/writes
    // 执行任务的线程，他是 volatile 的
    private volatile Thread runner;
```
里面的一些其他的方法就是用来对任务进行操作的，主要就是实现了父类的一些方法。里面也采用了 CAS 对线程的设定等待。

## 6. ThreadPoolExector
&emsp;&emsp;  然后介绍完了 AES 以后，就是真正的可用的类了，这里我们首先介绍的是 `ThreadPoolExector` 也就是我们常说的线程池。但是我们很少看到看到我们直接的去 new 一个 `ThreadPoolExector` 而更常用的是采用 `Executors` 这个类去创建线程池。主要就是 `Executors` 算是一个工厂方法，new 一个线程池的过程是比较麻烦的，需要配置很多的参数，这个工厂方法把一些我们会经常用到的的线程池都通过一个工厂方法返回给我们，减少我们的工作量。

&emsp;&emsp;  这个类其实也是比较复杂的，他的代码比较多，这里只分析一部分比较重要的参数和方法。

### 1. 字段

#### 1. 基本字段
```java
// 任务的后备队列
    private final BlockingQueue<Runnable> workQueue;
    // 锁
    private final ReentrantLock mainLock = new ReentrantLock();
    // 用来支持等待中断的
    private final Condition termination = mainLock.newCondition();
    // 存放的工作线程，只有当获取到锁的时候才能访问这个 Set
    private final HashSet<Worker> workers = new HashSet<Worker>();
    // 线程池最大数量
    private int largestPoolSize;
    // 完成的线程数，只有在获取锁的时候才能更新这个值
    private long completedTaskCount;

    //==============================================================================
    // 这里有提到用户自定义的变量我们都是用 volatile 来修饰 以保证获取到最新的值
    //==============================================================================
    // 线程创建工厂类
    private volatile ThreadFactory threadFactory;
    // 当任务队列饱和或者线程池关闭后 再往里面提交任务时候的执行策略
    private volatile RejectedExecutionHandler handler;
    // 默认的执行策略是采用的 AbortPolicy (这是一个函数式接口的子类，里面实现的方法默认是抛异常)
    private static final RejectedExecutionHandler defaultHandler = new AbortPolicy();
    // 非核心线程的存活时间
    private volatile long keepAliveTime;
    // 是否允许核心线程具有存活时间，允许则上面的参数也会作用于核心线程
    private volatile boolean allowCoreThreadTimeOut;
    // 核心线程的大小
    private volatile int corePoolSize;
    // 最大线程数
    private volatile int maximumPoolSize;
    // 池控参数  非常重要！！！！
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    // ctl 的解包 -> workerCount 和 runState
    private static int runStateOf(int c)     { return c & ~CAPACITY; }
    private static int workerCountOf(int c)  { return c & CAPACITY; }
    // 打包操作 两个变量或一下
    private static int ctlOf(int rs, int wc) { return rs | wc; }
```

&emsp;&emsp;  其中比较重要的属性有： workQueue(任务队列)、一个 Set<Worker>(线程集合)、池参数、线程工厂类、拒绝策略、池控参数。

&emsp;&emsp;  其中池控参数是将两个只是变量都打包进去了，分别是 workerCount 和 runState 。

&emsp;&emsp;    workerCount有效线程数.runState表明线程池的状态是否为运行，还是关闭。为了方便表示我们把 workerCount 和 runState 打包到了一个变量里面就是 ctl。
runState 表示生命周期，有以下状态：

1. RUNNING：接受新任务并处理排队的任务
2. SHUTDOWN：不接受新任务，但处理排队的任务
3. STOP：不接受新任务，不处理排队的任务，并中断正在进行的任务
4. TIDYING：所有任务都已终止，workerCount为零，线程转换到状态TIDYING 将运行terminate() 勾子
5. TERMINATED：terminated()已完成
 
 &emsp;&emsp; 并且这些值之间顺序很重要，以允许有序的比较。 runState在整个过程中是单调递增的但不需要经过每一个状态，具体规律如下：

* RUNNING -> SHUTDOWN  在执行 shutdown()的时候
* (RUNNING or SHUTDOWN) -> STOP    在执行shutdownNow()
* SHUTDOWN -> TIDYING      当任务队列和线程池为空的时候
* STOP -> TIDYING      当池为空的时候
* TIDYING -> TERMINATED    钩子方法调用完毕

#### 2. Worker 
&emsp;&emsp;  这个类其实比较有意思，一般我们会想到它里面必然是有一个线程的引用的，也就是把线程用 Worker 类来包装一下。然后还有一个就是他当前正在执行的任务引用。
&emsp;&emsp;  但是我们看这个类的继承及实现的时候就会发现比较有意思的事情，他继承了 AQS 实现了 Runnale 也就是他此时从结构来看既是一个 `同步器` 还是一个 `任务`。
##### 1. 字段

```java
        // 线程
        final Thread thread;
        // 该线程当前领到的任务
        Runnable firstTask;
```

##### 2. 方法
&emsp;&emsp;  里面比较重要的就是一些锁的实现，这里采用的全是互斥锁。然后实现的 Runnable 的 run 就是执行当前的任务。待会看运用这些方法再具体分析，里面的代码不多。

#### 3. BlockQueue
&emsp;&emsp;  这里采用了 BlockQueue 来存放任务，然后 BlockQueue 其实是一个接口，他有很多子类，而我们使用 Exectors 工厂方法 new 出新的线程池的时候其实不同种类的线程池采用的也是不同的 BlockQueue 。
&emsp;&emsp;  在 doc 中提到的子类有：ArrayBlockingQueue, DelayQueue, LinkedBlockingDeque, LinkedBlockingQueue, PriorityBlockingQueue, SynchronousQueue 但是我们真正常用的就是 ：ArrayBlockingQueue,LinkedBlockingQueue, SynchronousQueue 而我们在 Exectors 中就用了 LinkedBlockingQueue, SynchronousQueue 这两种。这个到时候会专门写一篇关于阻塞队列的博客，不然这个文章的篇幅太大了。

#### 4. ThreadFactory
&emsp;&emsp;  好很明显，这是一个函数式接口，里面就只有一个 newThread 方法，在线程池中，如果我们没有自己传入的话采用的都是 `Executors.defaultThreadFactory()`  创建一个 非 Daemon 优先级为 NORM_PRIORITY 的线程。
这样主线程退出时不会直接退出JVM，而是等待线程池中的线程结束。所有线程都调为同一个级别，这样在操作系统角度来看所有系统都是公平的，不会导致竞争堆积。
### 2. 主要方法
#### 1. ctor

```java
    public ThreadPoolExecutor(int corePoolSize,  //核心线程容量
                              int maximumPoolSize,//线程池的允许线程的最大容量
                              long keepAliveTime,//存活时间
                              TimeUnit unit,//时间单位
                              BlockingQueue<Runnable> workQueue,//任务队列
                                                                                 ThreadFactory threadFactory) {//线程工厂
           }
```

#### 2. execute
&emsp;&emsp;  这个方法是把提交到线程池中的任务在将来的某个时间运行起来，但是不一定能够保证这个任务肯定能执行到，为什么这么说？这是由于我们在前面看到提交到线程池中的线程不一定被马上执行，如果说线程池中线程都在忙现有的任务，新提交的就会被放到任务队列中 (BlockQueue)，然而我们在上面还看到了一个拒绝策略，就是当线程池饱和或者线程池马上关闭了提交的任务会被拒绝，在这里就是抛出异常！

&emsp;&emsp;  好，现在来理一下 `execute` 方法的执行过程。

1. 如果少于corePoolSize线程正在运行，开始一个新的线程领取任务。 对addWorker的调用会自动检查runState和workerCount，从而防止可能会添加的错误警报线程时它不应该通过返回false。

2. 把任务放入后备队列，如果成功我们还是需要再次检查，因为自上次检查依赖我们可能遇到线程池关闭
   如果有必要的话需要回退任务队列

3. 如果排队失败，我们会尝试添加一个新的线程。而线程添加失败则说明线程池关闭了或者
   已经处于饱和状态


```java
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
        // 如果正在运行的线程数小于核心池大小 可以添加一个新的线程领取这个任务
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            // 如果说添加新线程失败了我们需要重新获取线程池的工作程数量，可能有变化
            // 等看到 addWorker 的时候再说
            c = ctl.get();
        }
        // 如果线程池在工作状态，我们就把这个任务放到后备队列
        if (isRunning(c) && workQueue.offer(command)) {
            // 重新检查 ctl
            int recheck = ctl.get();
            // 如果线程池关闭了，我们需要做回退动作，也就是撤销刚才放入的任务
            // 如果撤销成功，执行拒绝策略
            if (! isRunning(recheck) && remove(command))
                reject(command);
            // 如果撤销失败，并且没有工作线程不管他
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        // 线程池已经关闭了 添加线程也失败则执行拒绝
        else if (!addWorker(command, false))
            reject(command);
    }
```

#### 3. addWorker
&emsp;&emsp;  这个方法的功能是：根据当前线程池的状态和给的边界条件来检测是否需要添加新的线程，如果是，则添加到线程队列中并调整工作线程数并启动线程执行第一个任务。 如果该方法检测到线程池处于STOP状态或者是察觉到将要停止，则返回false。 如果线程工厂创建线程失败(可能是由于发生了OOM异常)则也返回false。

今天有点累  ，先写到这里！明天继续，清明小长假还要继续学习  ：( 


