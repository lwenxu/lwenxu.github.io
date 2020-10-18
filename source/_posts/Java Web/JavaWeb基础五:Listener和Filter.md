---
title: Java Web 基础五：Listener和Filter
date: 2017-08-21 15:49:13
tags: Java Web
categories:
  -Java Web
---
Filter接口和Servlet的接口是非常类似的，它里面只有三个方法。



## 1. 监听器

三大域对象的监听器，每一个域对象有两种监听器，分别是生命周期监听，和属性监听。也就是对于对象的出生死亡监听以及对域中放入键值对时监听。

​`ServletContext`、`HttpSession`、`ServletRequest`  这三个域对象就占用了六个监听器。还有一个域对象就是 `PageContext` 这个只是我们没有对应的监听器。

我们要使用这些东西的话，我们首先需要创建一个类实现相应的接口就是上面的哪些监听器，名字就是这些名字加上 Listner、AttributeListner，然后重写里面的方法，接着需要在 web.xml 中配置 listener 中配置一下。具体的配置方式如下：

```xml
    <listener>
        <listener-class>three.listener.MyListener</listener-class>
    </listener>	
```

```java
public class MyListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("init..........");
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println("des............");
    }
}
```



感知监听器，这两个监听器是和 session 相关的，并且需要添加到 bean 上(bean 需要实现HttpSessionBindingListener 接口)，不需要再 web.xml 中配置。就是只要我们在 session 中删减了这个 java bean 我们就会触发对应的方法。

## 2. Filter接口

### 1. 生命周期

* `init`  Filter 创建的时候，启动服务器的时候创建 Filter
* `doFilter`  过滤的时候，放行 `chain.doFilter(request,response);`
* `destroy` 销毁之前，销毁费内存资源，关闭服务器
Servlet 他也是单例的。

### 2. 配置
他的配置和Servlet一模一样，有两个一个是注解，另外一个就是web.xml
并且配置名称都一样不过一个叫做servlet一个叫做filter

```xml
    <filter>
        <filter-name>MyFilter</filter-name>
        <filter-class>three.filter.MyFilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>MyFilter</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>REQUEST</dispatcher>
        <dispatcher>FORWARD</dispatcher>
        <dispatcher>INCLUDE</dispatcher>
        <dispatcher>ERROR</dispatcher>
    </filter-mapping>
```



```java
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
		filterChain.doFilter(servletRequest,servletResponse);//放行
    }

    @Override
    public void destroy() {

    }
}
```

多过滤器执行的时候是当第一个放行以后才能进入下一个拦截器，也就是拦截器必须放行才能来到下一个拦截器。

### 3. 四种拦截方式
对于Filter有四种拦截方式，也就是针对不同的请求有的类型可以拦截有的类型不拦截。
* REQUEST 请求，他也是默认的
* FORWARD 转发
* INCLUDE 包含
* ERROR 错误

多拦截的执行顺序就是 mapping 中的顺序。

