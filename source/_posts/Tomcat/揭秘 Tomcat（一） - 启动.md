---
title: 揭秘 Tomcat - 启动
date: 2021-06-27 14:39:26
tags: 源码分析
categories:
  - Tomcat

---

## 前言



tomcat 对于每一个 Java 工程师都是一个既熟悉又神秘的黑盒，什么 web 容器，什么三大组件，什么web.xml ..... 这些令人迷惑的问题都会在 《揭秘 Tomcat》系列文章中掰扯的清清楚楚。



 ![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1590165360512-927ede67-214d-460c-a928-90bdca6bbd88-20210627194357458.png)



## 从启动说起



启动 tomcat 非常简单 ， `./startup.sh` 一行命令，起飞 ~ 打开这个 shell 脚本我们发现最终启动了一个 Java 的 main 方法，也就是整个tomcat 的入口  `org.apache.catalina.startup.Boostrap` 瞅一眼简化后的代码 ~



![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1590165821369-01da8b92-bedb-495b-a856-d02aa16fbba6-20210627194354707.png)



简要的做一下分析启动参数，main 方法可以传入 startd、stopd、start、stop，startd 作为参数。其中 start 和 startd 区别就是将 await 设置为 true ，即程序是需要后台运行，还是在前台等待用户操作。



启动主要做了以下三件事情:

1. 1. 实例化 Boostrap ，并初始化
   2. 调用 load 方法
   3. 调用 start 方法



接下来就从以上三个步骤，来揭秘 tomcat 启动流程。



## 初始化启动器【init】



直接上代码看起来会比较蛋疼，我们先整理下思路，调用 `Boostrap#init` 方法主要完成三件事：



1. 初始化类加载器：初始化 Tomcat 核心加载器：`commonLoader`、`catalinaLoader`、`sharedLoader` ，后续会详细介绍这三个类加载器的作用以及使用场景。



1. 设置主线程 classLoader 为catalinaLoader，安全管理的 classLoad 为catalineLoader。



1.  通过反射生成Catalina对象，设置 Catalina 父加载器为 sharedLoader 





初始化的核心代码如下：

![image-20210627194240506](https://lwen-pic.oss-cn-beijing.aliyuncs.com/image-20210627194240506.png)



总结下来就是初始化了类加载器，即关键的一行就是 initClassLoaders()，它加载了/conf/catalina.properties 中的 common.loader、shared.loader、server.loader 对应的jar包和 properties 等文件，最后初始化了Catalina类，这个类将是加载我们 web 的 classloader。所以我们的自定义的文件或者 jar 包 都可以通过配置common.loader来加载 ，比如公共配置文件，日志文件等等。



看看 initClassLoaders() 方法代码，我们发现这几个类加载器默认情况下都是指向的 commonLoader，从源码可以看到，tomcat commonLoader 的父加载器为null，违背了java自己的类加载机制。那么问题来了 tomcat 为什么要定义自己的 classLoader ？

![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1593849559785-0bc2e39b-fed7-4e30-8f27-9003362b99af-20210627194253547.png)





​    我们知道 Java 默认是遵循双亲委派模型的，听起来高大上的名词，用一句话来解释就是所有类加载器优先使用父加载器进行类的加载。



![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1593523834557-c7ea4c3e-9bf6-408c-9b5a-d73d5c68cfd3-20210627194258231.png)



Java 体系中默认的类加载器遵循上图的父子关系，也就是说大部分的类都是最顶层的 BootStrap 这个类加载器进行加载的。这样做的好处就是保证 JDK 中的基础类不会被改写，另外防止一个类被不同的类加载器重复加载多次。



但是为什么 tomcat 没有使用默认的类加载机制，而是自己另起一套呢？看下下面几个场景：



- - tomcat 中可以部署多个项目，但是这多个项目可能依赖同一个类库的不同版本假如都使用 bootstrap 去加载那么就会出现类冲突谁知道要加载哪个版本； -> 衍生出 WebAppClassLoader
  - tomcat 内部依赖的 jar 和类库如果不和应用程序隔离的话应用程序中的类可以轻易覆盖 tomcat 中的核心类库造成安全问题； -> 衍生出 catalinaLoader
  - 应用程序和tomcat 都需要的类库大家可以共用的就不需要重复加载；-> 衍生出 Shared ClassLoader

  

简单看下 tomcat 的类加载体系：



![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1593524986228-d3bcd9f8-38c9-4b4f-aa4b-e18bc54071df-20210627194303384.png)



他们的功能从名字就能看出来时干嘛的，简单列举下功能：



- - commonLoader：Tomcat最基本的类加载器，加载路径中的class可以被Tomcat容器本身以及各个Webapp访问；
  - catalinaLoader：Tomcat容器私有的类加载器，加载路径中的class对于Webapp不可见；
  - sharedLoader：各个Webapp共享的类加载器，加载路径中的class对于所有Webapp可见，但是对于Tomcat容器不可见；
  - WebappClassLoader：各个Webapp私有的类加载器，加载路径中的class只对当前Webapp可见；







## 加载容器组件 【load】



紧接着调用了 Bootstrap 的 load 方法，看下简化后的代码：



![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1593528839395-d21d5374-f77b-4877-8c8f-4d14cef4a3e0-20210627194310073.png)



直接调用到了 Catalina#load 方法



![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1593527238438-3eb5e324-8c86-43fb-bc26-52ff6ca1fa1d-20210627194314428.png)



 简单介绍下上面代码的作用，configFile() 方法返回了 server.xml 文件, Digester 解析该文件，得到容器的配置，并创建容器组件和容器。这个地方的解析 xml 生成一系列的对象的逻辑很牛逼，也很复杂后面会专门有一篇文章来解析这部分逻辑！

 

依次创建的组件为 ：StandardServer、 StandardService、StandardEngine、StandardHost。然后拿到 StandardServer 实例调用 init() 方法初始化 Tomcat 容器的一系列组件。一些容器初始化的时候，都会调用其子容器的 init() 方法，初始化它的子容器，相当于每个容器都在初始化自身相关设置的同时，将子容器初始化。所以着重看下 init 方法



init 方法并不是 Server 类中的，而是他从 Lifecycle 接口的子类中继承的。就拿 StandardServer 举例， 他继承了 LifecycleMBeanBase ，可以看到他的初始化逻辑(下图的 init 方法) 并没有直接写出来，而是通过一个抽象方法让子类实现，而在本类中实现通用的逻辑，比如状态机的设定，异常处理。这就是大名鼎鼎的 `模板设计模式` ，读 tomcat 还能学学设计模式，血赚有没有~~~

![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1593856082499-5d86fe50-e9f4-423f-b0ae-c64d44d28a23-20210627194322188.png)



而这个类里面的模板占位符 `initInternal` 被 20 多个类进行了覆盖，都是我们的 tomcat 组件

![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1593855896060-60bdadcb-0ab5-4055-8933-7e80ac35dcab-20210627194328259.png)



同理  destroy -> destroyInternal ，start -> startInternal ，stop -> stopInternal 都是使用了相同的手法。



## 启动容器 【start】



最后一步就是 start ，调用后就能成功看到  `Server startup in 1222 ms ` 这样一行标志性的日志了



![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1593870761232-b3ae56f5-a65b-4b9a-8fc1-42c4d57cdf00-20210627194333667.png)



同样的，他还是调用了 `org.apache.catalina.startup.Catalina#start` 代码判断异常的逻辑较多，可以简化为下面的形式

![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1593871002514-5b7fb7ad-59bb-44d9-942e-deb6f1ee275d-20210627194337111.png)



如果没有初始化 server 那么重新初始化一下，这是一个好习惯，做一个兜底防止程序马上就挂了；之后调用了 server 组件的 start 方法，前面我们知道 server 组件初始化会调用内部的 initInternal ，start 也是同理的会调用内部的 start 他们都是生命周期接口中的方法。主要的事情就是启动子组件；



接着就注册了一个钩子函数，这是 jdk 中自带的线程停止后会执行另外一个线程帮他清理数据；



最后就是调用 await () 进行 web 端口的监听，启动后台线程池接受 http 请求，这样整个 tomcat 就启动完毕了，在 await 过程中方法是不会返回的所以 stop 不会被执行，而程序接收到 stop 的命令后会返回执行 stop 方法清理数据



## 总结

这里和大家主要从宏观的角度了解 tomcat 的启动流程，主要分为 init，load，start 三部分，而且很多中间件都是这三个部分组成的，不论是后面会讲到的 spring 还是 dubbo 大致流程都是如此，对我们后期看其他中间件源码会有一个大致的思路。

而且对于整个启动过程可以总结为三句话：

- 初始化 toncat 类加载器系统

- 解析 server.xml 初始化 tomcat 系统组件

- tomcat 组件启动，钩子注册，端口监听

1. 1. 

后续的文章会对这三部分进行详细的解读，里面使用的设计模式 ， 优秀的写法， 良好的风格我都会和大家一起探讨进步。关注我，在技术的道路上十年如一日的进步 ！



另外：大家也可以关注下我的微信公众号哦~ 技术分享和个人思考都会第一时间同步！

![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1610289331606-52e03a47-f431-4b20-8b57-6a4cc0f70345.png)
