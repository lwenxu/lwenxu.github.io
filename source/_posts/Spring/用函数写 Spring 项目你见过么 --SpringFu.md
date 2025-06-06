---

title: 用函数写 Bean 你见过么
date: 2021-07-10 15:02:37
tags: [Spring]
categories:
	- Spring Fu

---



​	函数是很多编程语言中的一等公民，并且在近些年提的很火的 `函数式变成` 以及 `Serverless` 都是通过函数来表达，每个函数提供一个单一功能的服务。他们相互作用，相互引用组装出复杂的功能。当云计算重构整个 IT 产业的同时， 也赋予了企业崭新的增长机遇。正如集装箱的出现加速了贸易全球化进程，以容器为代表的云原生技术作为云计算的服务新界面加速云计算也推动着软件架构向云原生演进。在当下处处都在呼吁 `云原生` 和 `反应式架构`的大背景下， Java 以及他的好搭档 Spring 又在做出怎么样的改变去紧跟这个新浪潮呢 ？



<center> <img src="https://lwen-pic.oss-cn-beijing.aliyuncs.com/cloud-native-architecutre-mindnode-20210710150004646.jpg" alt="Cloud native思维导图" style="zoom: 33%;" /></center>
  <center> [云原生概念导图] </center>



​	这里我们就要请出我们的主角 `SpringFu` , 据官方声称使用 SpringFu 搭建 SpringBoot 应用比普通自动配置的 SpringMVC 项目要快上 40% ，并且得益于很少使用反射特性能够非常好的适应 `GraalVM native` 同时还有内存占用量低等等优势。



##  GayHub 

​		其实 SpringFu 项目分为  [JaFu](https://github.com/spring-projects-experimental/spring-fu/tree/main/jafu) (Java DSL) and [KoFu](https://github.com/spring-projects-experimental/spring-fu/tree/main/kofu) (Kotlin DSL) ，也就是他既做了 Java 版本也做了 Kotlin 版本，作者也考虑了极为先进的卡特琳了，虽然这个语言我学过几天后再也没使用过，在平时的工作中我使用 Groovy 的频率都比 Kotlin 高。

<img src="https://lwen-pic.oss-cn-beijing.aliyuncs.com/image-20210710152019688.png" alt="image-20210710152019688" style="zoom: 50%;" />



先创建一个简单的 SpringBoot 工程，可以使用 https://start.spring.io/  这个 Spring 官方的脚手架搭建工具，当然如果你是跟我一样的 Idea 的重度用户也可以使用 Idea 的 `SpringInitializr` 

<img src="https://lwen-pic.oss-cn-beijing.aliyuncs.com/image-20210710153046306.png" alt="image-20210710153046306" style="zoom:50%;" />



## 加载依赖的小插曲

再到主 POM 里加下下面这这段朴实无华的代码：

```xml
<dependency>
    <groupId>org.springframework.fu</groupId>
    <artifactId>spring-fu-jafu</artifactId>
    <version>0.4.3</version>
</dependency>
```

但是当你更新 Maven 依赖的时候发现这个包并不会被拉下来，然后我果断换掉了 阿里云  的 Maven 镜像源，变成了 http://maven.net.cn/content/groups/public/ 然后赤裸裸的 Error 又把我打入地狱，什么玩意中央仓库都没有这个包？于是我拿着这个 artifactId 去 https://mvnrepository.com/ 全网 Maven 仓库查询了下，然后我注意到他有这么个 Tag ，呵 ！！！给我来这套

![image-20210710160236238](https://lwen-pic.oss-cn-beijing.aliyuncs.com/image-20210710160236238.png)



于是乎我经历了第三次变更 Maven 镜像源，我也懒得配置 mirrorOf 了，直接把这个镜像源提到第一个按照优先级先到这个镜像仓库取拉包，这次果然没有问题了。

```xml
<mirrors>  
    <!-- mirror | Specifies a repository mirror site to use instead of a given   
              repository. The repository that | this mirror serves has an ID that matches   
              the mirrorOf element of this mirror. IDs are used | for inheritance and direct   
              lookup purposes, and must be unique across the set of mirrors. | -->
    <mirror>
      <id>nexus-sp</id>
      <mirrorOf>*</mirrorOf>
      <name>Nexus sp</name>
      <url>https://repo.spring.io/milestone/</url>
    </mirror>
    <mirror>
      <id>nexus-163</id>
      <mirrorOf>*</mirrorOf>
      <name>Nexus 163</name>
      <url>http://mirrors.163.com/maven/repository/maven-public/</url>
    </mirror>
    <mirror>
      <id>nexus-aliyun</id>
      <mirrorOf>central</mirrorOf>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </mirror>
    <mirror>  
      <id>net-cn</id>  
      <mirrorOf>central</mirrorOf>  
      <name>Nexus net</name>  
      <url>http://maven.net.cn/content/groups/public/</url>
    </mirror>  
</mirrors>  
```



​	因为我勾选了 SpringData ，所以我必须配置一个数据源，这里我就用本地的 MySQL 当做测试数据源，让项目先跑起来。

<img src="https://lwen-pic.oss-cn-beijing.aliyuncs.com/image-20210710161001751.png" alt="image-20210710161001751" style="zoom: 50%;" />

​	不出意外，MacBook Pro 运行这种小程序就是快！

## 改造主程序



​	我们都清楚，SpringBoot 的加载是通过主类来完成的，想让 SpringFu 来作为整个项目的启动器我们就需要对 `SpringfuDemoApplication ` 做做改造了。注意如下代码请不要抄官方文档，你不可能跑起来的，因为他们的 demo 使用了一个 protected 的方法，如果你不是通过源码依赖是不可能调用到这个方法的，所以这里我换了一个写法，加载完 ioc 容器后在外部手动 enable 。

```java
public class Application {
    
    public static JafuApplication app = webApplication(ioc ->
            ioc.beans(beanDefinition ->
                    beanDefinition.bean(SampleHandler.class).bean(SampleService.class)
            ).enable(webMvc(dsl ->
                    dsl.port(dsl.profiles().contains("test") ? 8181 : 8080)
                            .router(router -> {
                                SampleHandler handler = dsl.ref(SampleHandler.class);
                                router.GET("/sayHello", handler::hello);
                            })
                            .converters(c ->
                                    c.string().jackson()
                            )
            ))
    );

    public static void main(String[] args) {
        app.run(args);
    }
}
```



其中依赖了 `SampleHandler` 这个处理器 代码如下：

```java
public class SampleHandler {
	
	public ServerResponse hello(ServerRequest request) {
		return ok().body("hello");
	}
	
}
```

​	启动后访问，大功告成。



![image-20210710175442941](https://lwen-pic.oss-cn-beijing.aliyuncs.com/image-20210710175442941.png)



## 新旧比较

### 加载 bean



在传统的 Spring 加载 Bean 一般是通过 xml 或者 JavaConfig 的方式，这里演示下相对比较简单的 JavaConfig 声明 Bean 的方式。通过一个 `@Configuration`  注解告诉 Spring 他是一个配置类，其中的每个 `@Bean` 声明的方法都是容器中的一个 Bean

```java
@Configuration
public class TestConfig {

    @Bean
    public User user() {
        return new User();
    }

    @Bean
    public Student student() {
        return new Student();
    }
}
```



而在 SpringFu 中使用就变得简单起来了，通过 `ConfigurationDsl` 的 beans 方法注册 bean，方法接受了一个 `Lambda` 表达式来加载 bean 。所以说在 SpringMVC 或者 SpringBoot 中需要使用 `@Component` 的地方最终都会使用 `ConfigurationDsl#beans` 来代替

```java
ioc.beans(beanDefinition ->
              beanDefinition.bean(SampleHandler.class)
              .bean(SampleService.class)
              .bean(Sample.class)
         )
```



### Web 路由



在 SpringMVC 中是通过 `@RequestMapping("/api/demo")` 这个注解来表名当前哪个路径会路由到当前类或者方法，而在 SpringFu 中则有专门的 DSL 来描述这个映射关系，比如我们上面用到的 `WebMvcServerDsl` 并且他定义了一系列的方法来配置 web 相关信息，比如用到的 `port` 和 `router`

```java
@RestController
@RequestMapping("/hello")
public class TestController {

    @RequestMapping("/echo")
    public String echo() {
        return "hello";
    }
}
```



在 SpringFu 中我们把 Controller 叫做 Handler , 路由配置方式也与之前有些差异：

```java
webMvcDsl.router(router -> {
      SampleHandler handler = dsl.ref(SampleHandler.class);
      router.GET("/sayHello", handler::hello);
})
```



整体来看，从 SpringMVC 到 SpringFu 改动的不仅仅是表达方式，而是一种编程思维，通过 `DSL` 抽象组件再通过 `Function` 的组合以及注册完成框架配置和业务展开。其实 SpringFu 的定位并不是替代 SpringMVC 而是在云原生以及在`Serverless`等以函数为主体的轻应用场景中发挥巨大作用。

本篇主要介绍 SpringFu 项目搭建，体验了一把总体感觉还是不错的除了有不少坑之外，下篇会看看关于 SpringFu 所用到的设计思想以及类关系简单的源码分析。



