---
title: Java多线程
date: 2017-08-09 15:49:28
tags: 多线程
categories:
 -Java
---
1.复写run方法的目的在于，把要运行的代码放到run方法里面，也就是新的线程要跑什么内容  
这也就是第一种多线程的方法，其主要的步骤如下：
1. 继承Thread类
2. 复写run方法
3. 创建对象
4. start  
<!--more-->
2.任何一个程序至少有一个线程就是主线程，主线程也是main方法的线程，这个线程是由jvm启动的，当我们自己创建新的线程的时候实际上是在主线程之外另开的新的线程和主线程并行工作
```java
class DemoRun extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 60; i++) {
            System.out.println("thread---"+i);
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) {
        DemoRun demoRun=new DemoRun(); //创建线程
        demoRun.start();//start方法有两个作用，一个是启动线程，第二个就是调用run方法
        //如果仅仅是使用run方法的话，也就仅仅调用了run方法里面的代码，而并没有开启多线程，run方法的线程还是在主线程
        for (int i = 0; i < 60; i++) {
            System.out.println("main---"+i);
        }
    }
}
```

3.第一种创建线程的方式其实会有很大的局限性，例如说，我们说java是单继承的语言，那么也就会出现一个class继承了父类，无法在继承Thread类
  而java却是多实现的，我们就可以继承runnable接口完成。但是注意，runnable接口并不是一个Thread类的对象，说白了他不是一个线程，那么我们
  就不知道我们多线程到底要运行哪的代码，不明确run方法。所以我们就先建立Thread的对象，然后把runnable接口的对象传递给Thread类，这样一来Thread类就明确了
  run方法的位置，也就是多线程要运行的代码的位置。
```java
class RunnableTest implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 60; i++) {
            System.out.println(Thread.currentThread().getName()+"-----"+i);
        }
    }
}

public class RunnableDemo {
    public static void main(String[] args) {
        RunnableTest runnableTest=new RunnableTest();
        Thread run=new Thread(runnableTest);
        Thread run1=new Thread(runnableTest);
        run.start();
        //注意如果实现的runnable接口的话一个线程可以启动多次，相当于多个线程，里面的属性可以当做共享变量来使用，因为是同一个对象
        run1.start();
    }
}
```
4.在说完多线程的创建以后最重要的就是线程的安全问题，主要就是共享变量的问题。例如上面的说到的实现runnable接口的时候，我们的属性就可以是共享变量
  一般来说就是使用进程同步代码块来搞定，安全问题。当如具体我们要同步哪一个代码块就看我们那个块使用了共享
```java
class TestDemo implements Runnable{
    private int num;
    Object obj=new Object();
    TestDemo(){
        num=100;
    }
    @Override
    public void run() {

        while (true){
            synchronized (obj) {    //这个地方可以随便放一个对象，但是注意如果是同一个对象就是同一个锁否则就基本没有同步功能
                if (num > 0) {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "=====" + num);
                    num--;
                }
            }
        }
    }
}

public class SyncDemo {
    public static void main(String[] args) {
        TestDemo testDemo=new TestDemo();
        new Thread(testDemo).start();
        new Thread(testDemo).start();
        new Thread(testDemo).start();
    }
}
```

5.同步函数：
```java

class Bank{
    public int sum;
    Bank(){
        sum=0;
    }

    public void add(int n){
        sum+=n;
    }
}

class TestDemo_01 implements Runnable{
    private Bank bank=new Bank();
    @Override
    //1.首先要明确哪些代码是多线程代码，这里显然就是run里面的，然后run里面调用了add函数，所以add方法也是多线程
    //2.之后就是哪些是共享变量，这里就是sum，bank
    //3.这样我们同步的代码就知道是哪些了既可以笼统的把run里面的操作bank的同步起来
    //这样的话我们就同步了一个函数，函数里面的那些局部变量就被同步了，这样和单线程没却别
    //所以我们同步add方法里面的内容
    //或者说我们可以直接同步add方法  就在方法的前面添加synchronized即可
    //也就是说我们的代码同步有两种方法一个就是同步代码块，一个就是同步函数
    public void run() {
        for (int i = 0; i < 3; i++) {
            synchronized (bank) {
                bank.add(100);
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(bank.sum);
            }
        }
    }
}

public class SyncFunction {
    public static void main(String[] args) {
        TestDemo_01 testDemo_01=new TestDemo_01();
        new  Thread(testDemo_01).start();
        new  Thread(testDemo_01).start();
    }
}

```

6.如果在我们进行了代码同步以后，代码还挂了那就说明我们的代码没有满足两个前提：
* 第一个就是必须是两个及两个以上的线程
* 第二个就是使用的必须是同一个锁

  这里顺带说同步函数使用的锁其实就是this
  然后如果是静态同步函数的话，我们知道静态函数不可能使用this，因为他不属于对象而是属于类
  这里也就是类的字解码文件  类名.class   就是这个


7.线程通讯同步，所谓线程同步通讯就是，在多个线程同时对同一个资源进行操作的时候我们使用同步代码让他们同步，但是这样可能会造成两个线程各自执行到底
    而非交替执行，为了交替执行我们使用同步，一个执行完了以后就睡眠唤醒另外一个，但是还是要注意这两个线程称一定要使用同一把锁，两个线程的代码都要同步
    具体方法就是：
    wait(),notify(),notifyAll()这几个函数都是使用在同步中的，因为他们需要对有监视器（锁）的对象进行操作
    所以只有在同步中他们才有锁、
    这些方法在操作线程的时候必须要标识他们所操作的锁，也就是说被某个锁同步的代码只能由此锁的wait，notify操作不可以对不同的锁的对象进行等待唤醒
    由于锁是任意对象所以说这些方法都被定义在Object中
```Java
class Resource{
    private int count;
    public boolean flag;
    Resource(){
        count=0;
        flag=true;
    }

    public synchronized void in(){
        if (flag) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("in===" + count++);
        flag=true;
        notify();
    }

    public synchronized void out(){
        if (!flag){
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("out------------"+count);
        flag=false;
        notify();
    }
}

class Producer implements Runnable{
    private Resource resource;

    public Producer(Resource resource) {
        this.resource = resource;
    }

    @Override
    public void run() {
        while (true){
            resource.in();
        }
    }
}

class Consumer implements Runnable{
    private Resource resource;

    public Consumer(Resource resource) {
        this.resource = resource;
    }

    @Override
    public void run() {
        resource.out();
    }
}

public class ProducerAndConsumer {
    public static void main(String[] args) {
        Resource resource=new Resource();
        Producer producer=new Producer(resource);
        Consumer consumer=new Consumer(resource);
        new Thread(producer).start();
        new Thread(producer).start();
        new Thread(consumer).start();
        new Thread(consumer).start();
    }
}
```
