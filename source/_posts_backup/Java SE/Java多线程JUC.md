---
title: Java多线程JUC
date: 2017-08-19 10:49:28
tags: 多线程
categories:
 -Java
---
### 1. volatile 关键字
多线程访问的时候，一个比较严重的问题就是内存不可见，其实在内存访问的时候每一个线程都有一个自己的缓冲区，每次在做修改的时候都是从主存取到数据，然后放到自己的缓冲区中，在做完修改之后放回主存。这样每一个线程之间的变量是不可见的。造成读到的数据可能始终就是错误的，因此有一个关键字可以使得这个共享变量称为透明的。就好像所有的操作就直接是在内存中操作一样，因为他一直不停的去同步主存的数据。<!--more-->
### 2.原子性
i++ 这个运算，其实在底层低用的就是临时变量的方式，这样的话虽然是一个表达式，但是在多线程的时候就会出现安全问题。
```java
package atomic;

/**
 * @Author: lwen
 * @Date: Created in 2017/8/19 15:26
 * @Description:
 */

class Test implements Runnable{
    private volatile int i=0;
    @Override
    public void run() {
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(i++);
    }
}
public class Main {
    public static void main(String[] args) {
        Test t=new Test();
        for (int i = 0; i < 10; i++) {
            new Thread(t).start();
        }
    }
}
```
最后运行的结果，显然就是 volatile 这个关键字他只是让变量课件也就是都是在主存中操作，但是他没有互斥的作用简单来说没有锁来控制，他们都及时的从主存里面拿到了数据，但是他们拿到最新的数据了，正在运算的时候有人提交了结果，所以导致重复元素。最根本原因是 i++ 有三步操作，读，运算，写
>>
1
3
2
0
4
7
6
5
8
8
Process finished with exit code 0

java 底层提供了一些原子性的变量，例如 AtomicInteger 这些东西他们既然是源自的首先就是可见的。这些东西的底层主要是使用了 CAS ( CompareAndSet ) 算法，CAS 算法主要是操作系统在硬件上提供的支持。
这个算法有三个重要的参数：第一个就是 V 就是运算前从内存读取的值 A 写入前从内存中读取的值 B 最终需要写入的值。在写入前做一次判断当且仅当 V == A  时才会写入 B 否则什么操作也不做。

### 3.ConcurrentHahsMap 安全的 HashMap
这是一个线程安全的 HashMap ，说道线程安全的　HahsMap 自然就有 HashTable 但是这个效率非常的低，主要就是因为他的封锁粒度太大，他锁的是整个 HashTable 也就是两个不相干的 HashTable 也是互斥访问的，在 jdk1.5 以后使用的就是 ConcurrentHahsMap 这个东西那个时候主要使用的锁分段机制，也就是在原来的基础上把 HashTable 分16段，每一段对应一个 HashMap 这样的话两个互不相干的 HashMap 是可以同步访问的。

### 4.CountDownLatch 闭锁
所谓的闭锁说白了就是该线程会等到洽谈所有线程的代码都运行结束了才开始运行，他的底层实现就是维护一个变量，这个变量就是当前存活的线程的数量，当他减成0了也就是其他的线程都运行完了，此时这个闭锁线程可以开始。例如说我们开十个线程做某一件事情，我们在主线程中统计这十个线程的总的运行时间。
```java
package latch;

import com.sun.javafx.sg.prism.web.NGWebView;

import java.util.concurrent.CountDownLatch;

/**
 * @Author: lwen
 * @Date: Created in 2017/8/19 16:06
 * @Description:
 */
class Latch implements Runnable{
    private CountDownLatch latch;

    public Latch(CountDownLatch latch) {
        this.latch = latch;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread());
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        latch.countDown();  //线程结束以后，需要给维护的变量减一
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        int start= (int) System.currentTimeMillis();
        CountDownLatch downLatch= new CountDownLatch(10);  //定义十个线程
        Latch latch=new Latch(downLatch);
        for (int i = 0; i < 10; i++) {
            new Thread(latch).start();
        }
        downLatch.await();  //当线程没有完全结束的时候，主线程需要等待
        int end= (int) System.currentTimeMillis();
        System.out.println(end-start);
    }
}
```

### 4.线程的第三种创建方式
一般我们创建线程我们使用的都是继承 Thread 类，或者实现 Runnable 接口，但是还是主要使用的是实现 Runnable 接口，但是注意这两个方式他们嗾使没有返回值的东西。也就是我么不能多线程没有办法返回一些结果。这里就出现了创建线程的第三种方式，也就是可以得到返回值的方式，就是使用 Callable 接口，这个接口的使用需要使用 FutureTask 类，而这个类实现了 Runnable 和 Future 接口，他的具体的使用方式和 Runnable 接口还是有一点不一样的。
```java
package future;

import org.omg.PortableInterceptor.INACTIVE;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/**
 * @Author: lwen
 * @Date: Created in 2017/8/19 16:23
 * @Description:
 */
class Future implements Callable<Integer>{

    @Override
    public Integer call() throws Exception {
        int sum=0;
        for (int i = 0; i < 1000; i++) {
            sum+=i;
        }
        return sum;
    }
}

public class Main {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Future future=new Future();
        FutureTask<Integer> task=new FutureTask<Integer>(future);
        new Thread(task).start();

        System.out.println(task.get());;//获取最终的返回值，但是注意这个地方的这个方法其实就是一个闭锁，前面的那个县城没有执行完，这个地方是不会执行的
    }
}
```

### 5.高级同步
解决同步问题总共就有三种方式，分别就是同步代码块，同步函数，和同步锁。
也就是手动的声明 lock 加锁，然后使用 unlock 释放锁

```java
package lock;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author: lwen
 * @Date: Created in 2017/8/19 16:33
 * @Description:
 */

class Ticket implements Runnable{
    private int ticket=100;
    Lock lock=new ReentrantLock();
    @Override
    public void run() {
        while (true) {
            lock.lock();
            try {
                if (ticket != 0) {
                    try {
                        Thread.sleep(20);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    ticket--;
                    System.out.println(Thread.currentThread().getName() + ":" + ticket);

                }
            }finally {
                lock.unlock();  //解锁必须放在finally里面保证能够执行到
            }
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Ticket t=new Ticket();
        new Thread(t,"窗口一").start();
        new Thread(t,"窗口二").start();
        new Thread(t,"窗口三").start();
    }
}
```

### 6.生产者和消费者的线程同步问题
```java
package product;

import java.util.logging.Level;

/**
 * @Author: lwen
 * @Date: Created in 2017/8/19 16:57
 * @Description:
 */
class Clerk {
    private int production=0;

    public synchronized void get() throws InterruptedException {
        while (production>=1){  //notifyAll 可能会形成假唤醒问题，所以说必须要使用while一直判断而不能使用if
            System.out.println("已满");
            this.wait();
        }
        System.out.println(Thread.currentThread().getName()+":"+ ++production);
        this.notifyAll();

    }

    public synchronized void sale() throws InterruptedException {
        while (production<=0){
            System.out.println("卖完");
            this.wait();
        }
        System.out.println(Thread.currentThread().getName()+":"+ --production);
        this.notifyAll();

    }
}


class Product implements Runnable{
    private Clerk clerk;

    public Product(Clerk clerk) {
        this.clerk = clerk;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            try {
                Thread.sleep(20);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            try {
                clerk.get();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class Consumer implements Runnable{
    private Clerk clerk;

    public Consumer(Clerk clerk) {
        this.clerk = clerk;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            try {
                Thread.sleep(20);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            try {
                clerk.sale();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
public class Main {
    public static void main(String[] args) {
        Clerk clerk=new Clerk();
        Product p=new Product(clerk);
        Consumer consumer=new Consumer(clerk);
        new Thread(p,"生产者1").start();
        new Thread(p,"生产者2").start();
        new Thread(consumer,"消费者1").start();
        new Thread(consumer,"消费者2").start();
    }
}
```
### 7.读写锁
排斥写写和读写同时进行
```java
package readandwrite;

import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * @Author: lwen
 * @Date: Created in 2017/8/19 17:29
 * @Description:
 */

class ReadWriteLockDemo {
    private int number=0;
    private ReadWriteLock lock=new ReentrantReadWriteLock();

    public void read(){
        lock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+":"+number);
        }finally {
            lock.readLock().unlock();
        }
    }

    public void write(int number) throws InterruptedException {
        lock.writeLock().lock();
        Thread.sleep(20);
        try {
            this.number=number;
            System.out.println("write");
        }finally {
            lock.writeLock().unlock();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        ReadWriteLockDemo readWriteLockDemo=new ReadWriteLockDemo();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    readWriteLockDemo.write(20);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        for (int i = 0; i < 20; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    readWriteLockDemo.read();
                }
            }).start();
        }
    }
}
```

### 4.线程池
```java
package pool;

import java.util.concurrent.*;

/**
 * @Author: lwen
 * @Date: Created in 2017/8/19 17:56
 * @Description:
 */

class Test implements Runnable{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}


class Test1 implements Callable<Integer>{

    @Override
    public Integer call() throws Exception {
        int sum=0;
        for (int i = 0; i <= 100; i++) {
            sum+=i;
        }
        return sum;
    }
}
public class Main {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        //注意线程池里面装的都是线程，这些线程到时候再次使用的时候不需要再次创建，也就是代表操作系统不需要再次分配资源，分配pid等等
        //而不是说里面装的是任务，这里仅仅就是线程而已
        ExecutorService executor= Executors.newFixedThreadPool(10);  //固定大小的线程池
        ExecutorService executor1=Executors.newSingleThreadExecutor(); //一个线程的线程池
        ExecutorService executor2=Executors.newCachedThreadPool(); //动态变化的线程池
        for (int i = 0; i < 20; i++) {
            executor.submit(new Test());
        }

        Future<Integer> future=executor1.submit(new Test1());
        System.out.println(future.get());

        executor.shutdown(); //只有当线程池关闭的时候  程序才会停止
        executor1.shutdown();
        executor2.shutdown();
    }
}
```

### 8.线程调度：
线程调度可以决定在多久之后执行什么操作。
```java
package schedule;

import java.util.Random;
import java.util.concurrent.*;

/**
 * @Author: lwen
 * @Date: Created in 2017/8/19 18:11
 * @Description:
 */


public class Main {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ScheduledExecutorService service= Executors.newScheduledThreadPool(5);
        for (int i = 0; i < 5; i++) {
            Future<Integer> future=service.schedule(new Callable<Integer>() {
                @Override
                public Integer call() throws Exception {
                    int random=new Random().nextInt(100);
                    System.out.println(Thread.currentThread().getName()+":"+random);
                    return random;
                }
            },1, TimeUnit.SECONDS);
            System.out.println(future.get());
        }
        service.shutdown();
    }
}
```
