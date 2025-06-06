---
title: SpringFu DSL 设计思想
date: 2021-07-11 15:02:37
tags: [SpringFu,源码]
categories:
	- SpringFu
	- Spring

---

让我们再回头看看这个启动类：

```java
public class Application {

    public static JafuApplication app = Jafu.webApplication(ioc ->
            ioc.beans(beanDefinition ->
                    beanDefinition.bean(SampleHandler.class)
                            .bean(SampleService.class)
                            .bean(Sample.class)
            ).enable(WebMvcServerDsl.webMvc(dsl ->
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
        ConfigurableApplicationContext context = app.run(args);
    }
}
```



从上面的代码中可以看到，不论是 `Jafu.webApplication` 还是 `ConfigurationDsl.enable()` 或者 `WebMvcServerDsl.webMvc()` 他们都有一个共同的特点就是通过 DSL 来描述设置、加载配置、定义 Bean ，可以说 `DSL` 就是整个 SpringFu 的驱动器和灵魂。

那么对于这些 DSL 又究竟是什么，里面都有哪些神奇的小组件，DSL 之间又有哪些使用小技巧呢？带着这些问题可以一起来扒一扒 SpringFu 的源码。



![image-20210710194145037](https://lwen-pic.oss-cn-beijing.aliyuncs.com/image-20210710194145037.png)



看到上面这个图一切就明朗起来了，如果说 SpringBoot 的自动配置是通过 `xxxAutoConfiguration` 那么SpringFu 的核心就在于 `xxxDsl` 。是的他将所有的组件都通过一份 `DSL代码`  来做配置和组合。而 DSL 的本质其实是 `ApplicationContextInitializer` 如果你恰好也熟悉 SpringBoot 的自动配置原理的话，你会顿时明白他们本质是同一个东西，都是通过 `ApplicationContextInitializer` 这个钩子函数来魔改 Spring 定义框架行为的。

如果你还不太熟悉 `ApplicationContextInitializer` 可以参看我写的这篇文章：xxx ，浅显易懂的介绍了他是什么，怎么用，为什么这么用。



## 顶级 Dsl

​	这里说的顶级 Dsl 并不是指他在类层级上属于顶层，而是说他是用于配置一个 SpringBoot 应用的 Dsl ，要启动一个 SpringBoot 应用首先要定义这个 Dsl。

​	这个类的代码也是非常简单，核心功能点在于执行用户自定义的 DslFunction。

```java
public class ApplicationDsl extends ConfigurationDsl {

	private final Consumer<ApplicationDsl> dsl;

	ApplicationDsl(Consumer<ApplicationDsl> dsl) {
		super(configurationDsl -> {});
		this.dsl = dsl;
	}

	@Override
	public void initialize(GenericApplicationContext context) {
		super.initialize(context);
		this.dsl.accept(this); // 执行自定义 DSL
		new MessageSourceInitializer().initialize(context); // 注册国际化的 AutoConfigurationBean 
	}

}
```

追溯下 `ApplicationDsl` 的父类 `ConfigurationDsl` 看起来方法不少，但要注意一个点我们可以把所有的 Dsl 类都分为两部分：`配置` 和 `执行` 。配置即组装当前 Dsl 的上下文，执行则是根据前面组装的上下文执行 initialize 方法。

![image-20210710200135821](https://lwen-pic.oss-cn-beijing.aliyuncs.com/image-20210710200135821.png)

​		上图两个红框的部分，分别是 `提供给用户的配置方法` 和 `系统执行的初始化方法` 。核心方法如下：



###  logging

​	配置日志等级

```java
public ConfigurationDsl logging(Consumer<LoggingDsl> dsl) {
   new LoggingDsl(dsl);
   return this;
}
```

###  configurationProperties

使用配置类

```java
public <T> ConfigurationDsl configurationProperties(Class<T> clazz, String prefix) {
   context.registerBean(clazz.getSimpleName() + "ConfigurationProperties", clazz, () -> new FunctionalConfigurationPropertiesBinder(context).bind(prefix, Bindable.of(clazz)).get());
   return this;
}
```

###  beans

注册 bean

```java
public ConfigurationDsl beans(Consumer<BeanDefinitionDsl> dsl) {
   new BeanDefinitionDsl(dsl).initialize(context);
   return this;
}
```

### enable

​	立即执行某个具体的 `Dsl` 

```java

public ConfigurationDsl enable(ApplicationContextInitializer<GenericApplicationContext> configuration) {
   return (ConfigurationDsl) super.enable(configuration);
}

protected AbstractDsl enable(ApplicationContextInitializer<GenericApplicationContext> dsl) {
  dsl.initialize(context);
  return this;
}
```

同时他还有一个重载的方法，立即执行 `ConfigurationDsl` 

```java
public ConfigurationDsl enable(Consumer<ConfigurationDsl> configuration) {
   new ConfigurationDsl(configuration).initialize(context);
   return this;
}
```

### listener

添加监听器

```java
public <E extends ApplicationEvent> ConfigurationDsl listener(Class<E> clazz, ApplicationListener<E> listener) {
   context.addApplicationListener(e -> {
      // TODO Leverage SPR-16872 when it will be fixed
      if (clazz.isAssignableFrom(e.getClass())) {
         listener.onApplicationEvent((E)e);
      }
   });
   return this;
}
```



在追溯 `ConfigurationDsl` 的父类 `AbstractDsl` 可以看到他提供了几个获取 Bean 以及获取环境变量的方法提供给所有的 Dsl



```java
public abstract class AbstractDsl implements ApplicationContextInitializer<GenericApplicationContext> {

   protected GenericApplicationContext context;

   public <T> T ref(Class<T> beanClass) {
      return this.context.getBean(beanClass);
   }

   public <T> T ref(Class<T> beanClass, String name) {
      return this.context.getBean(name, beanClass);
   }

   public Environment env() {
      return context.getEnvironment();
   }

   public List<String> profiles() {
      return Arrays.asList(context.getEnvironment().getActiveProfiles());
   }

   protected AbstractDsl enable(ApplicationContextInitializer<GenericApplicationContext> dsl) {
      dsl.initialize(context);
      return this;
   }
  
   @Override
   public void initialize(GenericApplicationContext context) {
      this.context = context;
   }

}
```

## DSL 设计思路

​		看了一个顶级 Dsl 的源码后我们大致能够窥探到整个 SpringFu 的 Dsl 的设计思路了：`执行 xx 操作就返回 xxDsl，并且在执行 xx 操作时传递 xxDsl 中预置方法的组合，来定义这个 Dsl 的行为。` 看文字还是比较抽象的，结合一个例子和图就会很明白了。

```java
WebMvcServerDsl webDsl = WebMvcServerDsl.webMvc(dsl ->
                            dsl.port(dsl.profiles().contains("test") ? 8181 : 8080)
                                .router(router -> {
                                    SampleHandler handler = dsl.ref(SampleHandler.class);
                                    router.GET("/sayHello", handler::hello);
                                })
                                .converters(c ->
                                        c.string().jackson()
                                )
                    	);
```

上面这段代码就是用于构造一个：

- 端口号为 8181 （测试环境） 8080 （非测试环境）
- 路由为 `/sayHello` 访问方式为 `Get` 
- 返回格式为 json

的 WebDsl 整体流程我们可以理解如下的流图

<img src="https://lwen-pic.oss-cn-beijing.aliyuncs.com/ab33ce32-7af0-4176-962c-c353ae173f82.png" alt="WebDsl 执行流程图" style="zoom: 50%;" />

<center>[WebDsl 执行流程图]</center>

