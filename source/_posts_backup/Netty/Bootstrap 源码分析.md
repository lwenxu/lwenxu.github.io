---
title: Bootstrap 源码分析
date: 2018-04-07 21:40:24
tags:
categories:
  - Netty 源码分析
---
# Netty 源码分析: Bootstrap

## 1. 结构
先看一个这个类的类层次结构，
![](http://ojrw3x8j2.bkt.clouddn.com/18-4-8/42956996.jpg)
好，这个结构还是比较明晰的，然后看他的主要字段，因为这些字段比较重要，在后面的代码分析中是用的上的。
<!-- more -->

```java
    // options 选项
    private final Map<ChannelOption<?>, Object> childOptions = new LinkedHashMap<ChannelOption<?>, Object>();
    // 属性
    private final Map<AttributeKey<?>, Object> childAttrs = new LinkedHashMap<AttributeKey<?>, Object>();
    // worker 线程池，为啥没有 boss ？ 待会说
    private volatile EventLoopGroup childGroup;
    // handler
    private volatile ChannelHandler childHandler;
```
&emsp;&emsp;  可以看到很多参数就是在我们进行 bootstrap 配置的时候进行设置的。但是我们注意到我们一般配置线程池的时候是有一个 boss 一个 worker ，但是这里只有一个引用是怎么回事？接着我们看一下构造方法。有一行注释 “ Specify the {@link EventLoopGroup} which is used for the parent (acceptor) and the child (client).” 这个意思就是说这里把线程池分为了 parent 和 child ，其中 parent 就是我们说的 boss 他的作用就是用来接收请求的连接，然后 worker 线程池就是用来处理读写事件的。最后我们找到两个参数的构造方法，用来初始化这两个参数，这里明显的能看到原来 boss 是在父类中定义的。所以调用父类的构造方法初始化了。过程和这个里对 Worker 初始化一样，做了检测，就不贴代码了。

```java
public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup) {
        // 初始化 boss
        super.group(parentGroup);
        if (childGroup == null) {
            throw new NullPointerException("childGroup");
        }
        if (this.childGroup != null) {
            throw new IllegalStateException("childGroup set already");
        }
        // 初始化 worker
        this.childGroup = childGroup;
        return this;
    }
```

## 2. 常用方法分析
&emsp;&emsp;  好的，上面看到了 bootstrap 里面的属性，接着分析一下我们常用的方法，主要就是 `group` 、 `option` 、`handler` 、`childHandler` 、 `bind` 、 `connect` 。

### 1. option
&emsp;&emsp;  逻辑比较简单，如果 value 为 null 就说明要删除这个 option ，否则就放到表里面。

```java
public <T> B option(ChannelOption<T> option, T value) {
        if (option == null) {
            throw new NullPointerException("option");
        }
        if (value == null) {
            synchronized (options) {
                options.remove(option);
            }
        } else {
            synchronized (options) {
                options.put(option, value);
            }
        }
        return (B) this;
    }
```

### 2. handler/childHandler
&emsp;&emsp;  这个是直接设置了引用，就把 handler 设置为传入的参数。childHandler 同理。

```java
public B handler(ChannelHandler handler) {
        if (handler == null) {
            throw new NullPointerException("handler");
        }
        this.handler = handler;
        return (B) this;
    }
```

### 3. channel
&emsp;&emsp;  这里面直接调用了 channelFactory ，也就是在 3.x 中设置 channel 工厂，这里做了进一步的封装。这个 channel 工厂的作用就是利用反射 newInstance 。

### 4. bind
&emsp;&emsp;  好吧，这个可能是现在遇到的最麻烦的一个方法了，因为如果要说的话，里面牵扯的东西太多了，又再次引入的了新的类和方法。不着急先了解大概的思路，再看源码。

![](http://ojrw3x8j2.bkt.clouddn.com/18-4-9/93982359.jpg)
&emsp;&emsp;  在 bind 方法中写出来只有几行代码，但是每一个方法里面涉及的东西就比较多。首先是进行了验证，验证就是检测线程池 和 channelFactory 是不是正常被赋值了。接着就是重要的 doBind 方法。

&emsp;&emsp;  doBind 分为两个重要操作，第一个就是：对 channel 进行初始化和注册操作，然后把执行绑定操作。也就是这里涉及到的两个重要的方法： `initAndRegister` 和 `doBind0` 。




我的博客即将搬运同步至腾讯云+社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan

