---
title: SpringBoot 笔记(十一):Servlet容器
date: 2018-05-24 10:01:28
tags: SpringBoot
categories:
 - SpringBoot
---
# SpringBoot 笔记 ( 十一 ):Servlet容器

SpringBoot默认使用Tomcat作为嵌入式的Servlet容器

## 1）、定制和修改Servlet容器的相关配置

### 1、修改配置文件中的和 server 有关的配置

ServerProperties【也是EmbeddedServletContainerCustomizer】

<!--more-->

```properties
server.port=8081
server.context-path=/crud
server.tomcat.uri-encoding=UTF-8
//通用的Servlet容器设置
server.xxx
//Tomcat的设置
server.tomcat.xxx
```

### 2、编写一个 EmbeddedServletContainerCustomizer

嵌入式的Servlet容器的定制器，来修改Servlet容器的配置

```java
@Bean  //将这个定制器加入到容器中
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
    return new EmbeddedServletContainerCustomizer() {
        //定制嵌入式的Servlet容器相关的规则
        @Override
        public void customize(ConfigurableEmbeddedServletContainer container) {
            container.setPort(8083);
        }
    };
}
```

## 2）、注册Servlet三大组件【Servlet、Filter、Listener】

由于SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文件。

注册三大组件用以下方式：

### 1.ServletRegistrationBean

```java
//注册三大组件
@Bean
public ServletRegistrationBean myServlet(){
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(new MyServlet(),"/myServlet");
    return registrationBean;
}

```

### 2.FilterRegistrationBean

```java
@Bean
public FilterRegistrationBean myFilter(){
    FilterRegistrationBean registrationBean = new FilterRegistrationBean();
    registrationBean.setFilter(new MyFilter());
    registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
    return registrationBean;
}
```

### 3.ServletListenerRegistrationBean

```java
@Bean
public ServletListenerRegistrationBean myListener(){
    ServletListenerRegistrationBean<MyListener> registrationBean = new ServletListenerRegistrationBean<>(new MyListener());
    return registrationBean;
}
```



### 4、DIspatcherServlet 配置

SpringBoot帮我们自动SpringMVC的时候，自动的注册SpringMVC的前端控制器，DIspatcherServlet

DispatcherServletAutoConfiguration中：

```java
@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
@ConditionalOnBean(value = DispatcherServlet.class, name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
public ServletRegistrationBean dispatcherServletRegistration(
      DispatcherServlet dispatcherServlet) {
   ServletRegistrationBean registration = new ServletRegistrationBean(
         dispatcherServlet, this.serverProperties.getServletMapping());
    //默认拦截： /  所有请求，包静态资源，但是不拦截jsp请求；   /*会拦截jsp
    //可以通过server.servletPath来修改SpringMVC前端控制器默认拦截的请求路径
    
   registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
   registration.setLoadOnStartup(
         this.webMvcProperties.getServlet().getLoadOnStartup());
   if (this.multipartConfig != null) {
      registration.setMultipartConfig(this.multipartConfig);
   }
   return registration;
}

```

## 3）、替换为其他嵌入式Servlet容器

默认支持：

### 1.Tomcat（默认使用）

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   引入web模块默认就是使用嵌入式的Tomcat作为Servlet容器；
</dependency>
```

### 2.Jetty（长链接类的服务）

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-jetty</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

### 3.Undertow（高并发的不支持JSP）

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-undertow</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

## 4）、嵌入式Servlet容器自动配置原理

EmbeddedServletContainerAutoConfiguration：嵌入式的Servlet容器自动配置

```java
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Configuration
@ConditionalOnWebApplication
@Import(BeanPostProcessorsRegistrar.class)
//导入BeanPostProcessorsRegistrar：Spring注解版；给容器中导入一些组件
//导入了EmbeddedServletContainerCustomizerBeanPostProcessor：
//后置处理器：bean初始化前后（创建完对象，还没赋值赋值）执行初始化工作
public class EmbeddedServletContainerAutoConfiguration {
    
    @Configuration
	@ConditionalOnClass({ Servlet.class, Tomcat.class })//判断当前是否引入了Tomcat依赖；
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)//判断当前容器没有用户自己定义EmbeddedServletContainerFactory：嵌入式的Servlet容器工厂；作用：创建嵌入式的Servlet容器
	public static class EmbeddedTomcat {

		@Bean
		public TomcatEmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() {
			return new TomcatEmbeddedServletContainerFactory();
		}

	}
    
    /**
	 * Nested configuration if Jetty is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Server.class, Loader.class,
			WebAppContext.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedJetty {

		@Bean
		public JettyEmbeddedServletContainerFactory jettyEmbeddedServletContainerFactory() {
			return new JettyEmbeddedServletContainerFactory();
		}

	}

	/**
	 * Nested configuration if Undertow is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Undertow.class, SslClientAuthMode.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedUndertow {

		@Bean
		public UndertowEmbeddedServletContainerFactory undertowEmbeddedServletContainerFactory() {
			return new UndertowEmbeddedServletContainerFactory();
		}

	}
```



EmbeddedServletContainerFactory（嵌入式Servlet容器工厂）,主要是用来创建 EmbeddedServletContainer 也就是嵌入式的Servlet容器。这个是一个接口，也就是我们有对应的实现类，就是上面的三种嵌入式的容器。

```java
public interface EmbeddedServletContainerFactory {

   //获取嵌入式的Servlet容器
   EmbeddedServletContainer getEmbeddedServletContainer(
         ServletContextInitializer... initializers);

}
```

下面我们以 TomcatEmbeddedServletContainerFactory 分析

```java
@Override
public EmbeddedServletContainer getEmbeddedServletContainer(
      ServletContextInitializer... initializers) {
    //创建一个Tomcat
   Tomcat tomcat = new Tomcat();
    //配置Tomcat的基本环节
   File baseDir = (this.baseDirectory != null ? this.baseDirectory
         : createTempDir("tomcat"));
   tomcat.setBaseDir(baseDir.getAbsolutePath());
   Connector connector = new Connector(this.protocol);
   tomcat.getService().addConnector(connector);
   customizeConnector(connector);
   tomcat.setConnector(connector);
   tomcat.getHost().setAutoDeploy(false);
   configureEngine(tomcat.getEngine());
   for (Connector additionalConnector : this.additionalTomcatConnectors) {
      tomcat.getService().addConnector(additionalConnector);
   }
   prepareContext(tomcat.getHost(), initializers);
    
    //将配置好的Tomcat传入进去，返回一个EmbeddedServletContainer
    //并且启动Tomcat服务器
   return getTomcatEmbeddedServletContainer(tomcat);
}
```

但是我们对嵌入式容器的配置修改是怎么生效？实际上是  `ServerProperties、EmbeddedServletContainerCustomizer`	：定制器帮我们修改了Servlet容器的配置，也就是我们在上面导入的 `@Import(BeanPostProcessorsRegistrar.class)` 然后导入了EmbeddedServletContainerCustomizerBeanPostProcessor（后置处理器）：bean初始化前后（创建完对象，还没赋值赋值）执行初始化工作。

容器中导入了**EmbeddedServletContainerCustomizerBeanPostProcessor**，他的配置原理

```java
//初始化之前
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName)
      throws BeansException {
    //如果当前初始化的是一个ConfigurableEmbeddedServletContainer类型的组件
   if (bean instanceof ConfigurableEmbeddedServletContainer) {
       //
      postProcessBeforeInitialization((ConfigurableEmbeddedServletContainer) bean);
   }
   return bean;
}

private void postProcessBeforeInitialization(
			ConfigurableEmbeddedServletContainer bean) {
    //获取所有的定制器，调用每一个定制器的customize方法来给Servlet容器进行属性赋值；
    for (EmbeddedServletContainerCustomizer customizer : getCustomizers()) {
        customizer.customize(bean);
    }
}

private Collection<EmbeddedServletContainerCustomizer> getCustomizers() {
    if (this.customizers == null) {
        // Look up does not include the parent context
        this.customizers = new ArrayList<EmbeddedServletContainerCustomizer>(
            this.beanFactory
            //从容器中获取所有这葛类型的组件：EmbeddedServletContainerCustomizer
            //定制Servlet容器，给容器中可以添加一个EmbeddedServletContainerCustomizer类型的组件
            .getBeansOfType(EmbeddedServletContainerCustomizer.class,
                            false, false)
            .values());
        Collections.sort(this.customizers, AnnotationAwareOrderComparator.INSTANCE);
        this.customizers = Collections.unmodifiableList(this.customizers);
    }
    return this.customizers;
}

ServerProperties也是定制器
```

步骤：

1）、SpringBoot根据导入的依赖情况，给容器中添加相应的 EmbeddedServletContainerFactory 比如这里就是TomcatEmbeddedServletContainerFactory

2）、容器中某个组件要创建对象就会调用后置处理器：EmbeddedServletContainerCustomizerBeanPostProcessor。

只要是嵌入式的容器工厂，后置处理器就工作。

3）、后置处理器，从容器中获取所有的 **EmbeddedServletContainerCustomizer**，调用定制器的定制方法，而我们的`ServerProperties` 就是我们要的定制器。



## 5）、嵌入式Servlet容器启动原理

什么时候创建嵌入式的Servlet容器工厂？什么时候获取嵌入式的Servlet容器并启动Tomcat；

获取嵌入式的Servlet容器工厂：

### 1）、SpringBoot应用启动运行run方法

### 2）、refreshContext(context)

SpringBoot刷新IOC容器，也就是创建IOC容器对象，并初始化容器，创建容器中的每一个组件，这里需要判断如果是web应用创建 `AnnotationConfigEmbeddedWebApplicationContext`，否则创建`AnnotationConfigApplicationContext`

### 3）、refresh(context)

刷新刚才创建好的ioc容器

```java
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         initMessageSource();

         // Initialize event multicaster for this context.
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         onRefresh();

         // Check for listener beans and register them.
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         finishRefresh();
      }

      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // Destroy already created singletons to avoid dangling resources.
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
```

### 4）、  onRefresh()

 web的ioc容器重写了onRefresh方法，web 的 ioc容器会创建嵌入式的Servlet容器，createEmbeddedServletContainer()。

### 5）、获取嵌入式的Servlet容器工厂

createEmbeddedServletContainer() 这个方法的第一步就是去创建这个工厂，getEmbeddedServletContainerFactory。

EmbeddedServletContainerFactory containerFactory = getEmbeddedServletContainerFactory();

从ioc容器中获取EmbeddedServletContainerFactory 组件，TomcatEmbeddedServletContainerFactory 创建对象，后置处理器就获取所有的定制器来先定制Servlet容器的相关配置。完成配置和启动。

### 6）、使用容器工厂获取嵌入式的Servlet容器

this.embeddedServletContainer = containerFactory.getEbeddedServletContainer(getSelfInitializer());

### 7）、嵌入式的Servlet容器创建对象并启动Servlet容器

先启动嵌入式的Servlet容器，再将ioc容器中剩下没有创建出的对象获取出来，这个时候我们自己写的Controller Service 以及各种配置类才会生效。

## 6）、使用外置的Servlet容器

我们采用嵌入式Servlet容器，这样我们的应用打成可执行的jar

优点：简单、便携

缺点：默认不支持JSP、优化定制比较复杂（使用定制器ServerProperties、自定义EmbeddedServletContainerCustomizer，自己编写嵌入式Servlet容器的创建工厂EmbeddedServletContainerFactory）

所以我们可以使用外置的Servlet容器：外面安装Tomcat---应用war包的方式打包；

### 1.步骤

1）、必须创建一个war项目，我们还是使用 SpringBootInitlizer 这个工具创建项目，但是注意我们在创建项目的时候我们选择部署的方式为 war 就可以生成 war的形式。

然后我们需要去项目的结构地方的web模块点击我们的目录结构让他自动生成webapp 目录。生成在 src/main 下面。然后就是生成 web 应用描述符，也就是生成 web.xml

接着我们需要修改应用的启动配置，自己添加一个tomcat的服务器。然后在 deploy 这个地方加入我们的 war包。

2）、将嵌入式的Tomcat指定为provided；

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <scope>provided</scope>
</dependency>
```

3）、必须编写一个**SpringBootServletInitializer**的子类类名随意但是必须继承，并调用configure方法

```java
public class ServletInitializer extends SpringBootServletInitializer {

   @Override
   protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
       //传入SpringBoot应用的主程序
      return application.sources(SpringBoot04WebJspApplication.class);
   }

}
```

4）、启动服务器就可以使用；

### 2.启动原理

#### 1. 打包方式

1. jar包：执行SpringBoot主类的main方法，启动ioc容器，创建嵌入式的Servlet容器。
2. war包：启动服务器，然后服务器启动SpringBoot应用，使用的是 SpringBootServletInitializer，启动ioc容器。

servlet3.0一个规范：

​	1）、服务器启动（web应用启动）会创建当前web应用里面每一个jar包里面 ServletContainerInitializer 实例

​	2）、ServletContainerInitializer的实现放在jar包的META-INF/services文件夹下，有一个名为javax.servlet.ServletContainerInitializer的文件，内容就是ServletContainerInitializer的实现类的全类名

​	3）、还可以使用 @HandlesTypes，在应用启动的时候加载我们感兴趣的类

#### 2.启动流程：

##### 1）、启动Tomcat

##### 2）、SpringServletContainerInitializer 的调用

org\springframework\spring-web\4.3.14.RELEASE\spring-web-4.3.14.RELEASE.jar!\META-INF\services\javax.servlet.ServletContainerInitializer：Spring的web模块里面有这个文件：**org.springframework.web.SpringServletContainerInitializer** 也就是我们应用启动的时候会创建这个对象。

##### 3）、创建 @HandlesTypes 的对象

SpringServletContainerInitializer 将 @HandlesTypes(WebApplicationInitializer.class) 标注的所有这个类型的类都传入到 onStartup 方法的 Set<Class<?>>，为这些 WebApplicationInitializer 类型的类创建实例

##### 4）、调用 onStartup 方法

每一个 WebApplicationInitializer 都调用自己的onStartup；

##### 5）、SpringBootServletInitializer 的 onStartup

我们的 SpringBootServletInitializer 的类是继承自 WebApplicationInitializer 并且被放到了@HandlesTypes里，所以会被创建对象，并执行onStartup方法，也就是我们采用 war 包工程生成的这个类。

##### 6）、创建 ICO 容器

SpringBootServletInitializer实例执行onStartup的时候会 createRootApplicationContext 创建容器

```java
protected WebApplicationContext createRootApplicationContext(
      ServletContext servletContext) {
    //1、创建SpringApplicationBuilder
   SpringApplicationBuilder builder = createSpringApplicationBuilder();
   StandardServletEnvironment environment = new StandardServletEnvironment();
   environment.initPropertySources(servletContext, null);
   builder.environment(environment);
   builder.main(getClass());
   ApplicationContext parent = getExistingRootWebApplicationContext(servletContext);
   if (parent != null) {
      this.logger.info("Root context already created (using as parent).");
      servletContext.setAttribute(
            WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, null);
      builder.initializers(new ParentContextApplicationContextInitializer(parent));
   }
   builder.initializers(
         new ServletContextApplicationContextInitializer(servletContext));
   builder.contextClass(AnnotationConfigEmbeddedWebApplicationContext.class);
    
    //调用configure方法，子类重写了这个方法，将SpringBoot的主程序类传入了进来
   builder = configure(builder);
    
    //使用builder创建一个Spring应用
   SpringApplication application = builder.build();
   if (application.getSources().isEmpty() && AnnotationUtils
         .findAnnotation(getClass(), Configuration.class) != null) {
      application.getSources().add(getClass());
   }
   Assert.state(!application.getSources().isEmpty(),
         "No SpringApplication sources have been defined. Either override the "
               + "configure method or add an @Configuration annotation");
   // Ensure error pages are registered
   if (this.registerErrorPageFilter) {
      application.getSources().add(ErrorPageFilterConfiguration.class);
   }
    //启动Spring应用
   return run(application);
}
```

##### 7）、Spring的应用就启动并且创建IOC容器

```java
public ConfigurableApplicationContext run(String... args) {
   StopWatch stopWatch = new StopWatch();
   stopWatch.start();
   ConfigurableApplicationContext context = null;
   FailureAnalyzers analyzers = null;
   configureHeadlessProperty();
   SpringApplicationRunListeners listeners = getRunListeners(args);
   listeners.starting();
   try {
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
      ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);
      Banner printedBanner = printBanner(environment);
      context = createApplicationContext();
      analyzers = new FailureAnalyzers(context);
      prepareContext(context, environment, listeners, applicationArguments,
            printedBanner);
       
       //刷新IOC容器
      refreshContext(context);
      afterRefresh(context, applicationArguments);
      listeners.finished(context, null);
      stopWatch.stop();
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass)
               .logStarted(getApplicationLog(), stopWatch);
      }
      return context;
   }
   catch (Throwable ex) {
      handleRunFailure(context, listeners, analyzers, ex);
      throw new IllegalStateException(ex);
   }
}
```

先启动Servlet容器，然后容器根据我们的Servlet的3.0 的标准，去启动我们SpringBoot 下生成的一个类，这个类再启动SpringBoot应用。

