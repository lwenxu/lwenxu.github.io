---
title: Java Web 基础一:Servlet详解
date: 2017-08-20 20:07:27
tags: Java Web
categories:
  -Java Web
---
### 1.Servlet接口
1.最顶层的Servlet就是一个接口，也就是Servlet接口，要实现这个接口必须要实现其中的五个方法，这五个方法中有三个是生命周期方法，这三个生命周期方法都是没有返回值的，分别就是：
* init 在servlet创建的时候被调用一次，并且始终只调用一次，servlet的创建时在第一次被访问的时候，但是servlet是单例的，不会重复创建，他和后面要学的struts2的action不一样。但是我们可以配置他让他在服务器启动的时候被创建。避免第一次惩戒。就加一行
```xml
<load-on-start>0</load-on-start>
```
就能然他在服务器启动的时候创建。
* service 这个方法会被多次调用，主要就是每次这个servlet被浏览器访问的时候他都会被调用一次，这个方法接受两个参数，一个是ServletRequest这个是tomcat将从浏览器获取到的数据封装到了这个类的对象中提供给我们的Servlet，两外一个就是Response这个就是将我们Servlet返回的数据封装起来然后返回给tomcat，最后返回给浏览器。
* destroy 这个方法是在servlet销毁之前被调用一次，这个作用不是销毁servlet而是用来吹资源用的，尤其是数据库连接，文件句柄等等。
剩下还有两个方法基本不会被调用，这里也稍微说一下。
* getServletInfo 这个方法是用来返回servlet的信息，这个信息是一个字符串，也牛市由开发者来随便写，这个servlet是关于什么的servlet，但是注意servlet都是由tomcat来创建，并调用里面的方法的，一般他们是不会调用这个方法的。
* getServletConfig 这个方法稍微有用一点，主要就是获取servlet的配置信息，也就是在web.xml中的关于servlet的配置。而这个配置信息又是怎么来的呢？其实看看init方法就会发现这个方法被传入了一个参数，参数就是 ServletConfig 类型的，所以说这个配置信息其实是由tomcat传递给init方法的，然后可以使用这个方法来获取。但是注意这个 ServletConfig 是一个接口，也就是这个东西的具体实现类我们没写，那么就是有tomcat来提供他的具体实现类。

### 2.Servlet 配置
#### 1.web.xml配置方式
这中配置方式就是使用xml标签来配置Servlet的信息，也是最原始的一种，这里也介绍一下。
<servlet>
    <servlet-name>AServlet</servlet-name>
    <servlet-class>AServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>AServlet</servlet-name>
    <url-pattern>/AServlet</url-pattern>
</servlet-mapping>
这样的话，首先就是浏览器找到url-pattern的位置发现能够匹配就去找名叫AServlet的Servlet，然后就能找到具体的类，实例化类，调用service方法

#### 2.注解方式
其实在j2ee中很多能够使用xml配置的都可以使用注解来解决，但是这个不是绝对的，有很多时候必须使用web.xml来配置。
```java
@WebServlet(name = "AServlet",urlPatterns = "/AServlet")
```
只需要在这个servlet的类前面添加一行注解，就能达到和刚才的web.xml配置一样的效果


注意上面的url-pattern是可以配置成通配符，但是通配符只能出现一次，并且只能出现在最前面或者最后面。
### 3. ServletConfig 接口
这个接口上面也说了，具体的实现类是由tomcat提供的，然后这个接口其实提供了四个方法，其中有一个特别重要的方法。
java.lang.String	getInitParameter(java.lang.String name)
         Gets the value of the initialization parameter with the given name.
java.util.Enumeration<java.lang.String>	getInitParameterNames()
         Returns the names of the servlet's initialization parameters as an Enumeration of String objects, or an empty Enumeration if the servlet has no initialization parameters.
这前面两个方法主要就是获取一下初始化参数，一般就是配置在servlet中，然后具体的标签就是
```xml
<servlet>
    <servlet-name>AServlet</servlet-name>
    <servlet-class>AServlet</servlet-class>
    <init-param>
        <param-name>a</param-name>
        <param-value>hello</param-value>
    </init-param>
</servlet>
```
类似于这种，基本上不会怎么使用。
ServletContext	getServletContext()
         Returns a reference to the ServletContext in which the caller is executing.
java.lang.String	getServletName()
         Returns the name of this servlet instance.
下面的两个用的最多的就是 getServletContext ，这个方法很重要，后面会专门来说这个对象。

### 4.GenericServlet 类
这个类实际上就是一个封装了一个 ServletConfig 对象的包装类，这个类里面的核心就是 ServletConfig 对象，他多出来的方法就是来源于 ServletConfig 的四个方法，以及相关方法。具体的可以查看源码，写的比较简单。

### 5.HTTPServlet 类
这个类才是我们最常用的类，因为他封装的是关于协议的对象，对service方法进行了包装，他对ServletRequest对象进行了强转，转化为HTTPServletRequest对象，然后根据这个对象的信息获取具体的请求方式，从而衍生出了两个方法doGet和doPost，这两个方法就是分别在请求方式为get和post的时候被调用，如果我们没有重写某个方法，而我们访问的时候恰好使用的是这个类型就会报错，报错代码就是405，服务器不支持的请求类型。

### 6.全局 web.xml
这个其实就是在服务器下面有一个web.xml，这个xm里面的内容会出现在每一个项目的web.xml中，也就相当于全局的web.xml。  
这个全局的web.xml配置了默认的servlet和jsp的处理，也就是以jsp结尾的都会被web.xml拦截然后进行处理。  
里面还有session的过期时间，一般默认的就是30分钟。  
以及各种mime类型，也就是各种文件的mime。  

### 7.ServletContext 对象
一个项目只有一个 ServletContext 对象，这个对象也就是 application 。这个东西就可以成为各个 Servlet 之间的信息传递。  
ServletContext 一般就是和tomcat的生命周期一模一样，服务器启动的时候产生，关闭的时候这个对象才会被销毁。  
这个对象的获取就是通过ServletConfig的getServletContext方法获取，其实在GenericServlet中就提供了getServletConfig方法获取这个对象。另外除了这两个对象可以获取这个对象以外，还有就是HTTPSession也是可以获取的，以及ServletContextEvent也可以，他们的方法名称都是一样的。也就是目前有四种方式获取这个对象。

### 8.域对象
在JavaWeb中有四个域对象，jsp中继承了JavaWeb的四个域对象，而Servlet中只有三个。
* PageContext
* HttpSession
* ServletRequest
* ServletContext
他们都有公共的两个方法就是setAttribute和getAttribute就是存取数据。

注意我们最常用的ServletContext的功能就是获取项目路径：  
* ServletContext.getRealPath("index.jsp") 这个返回的最后是服务器上位置，也就是硬盘位置
* ServletContext.getResourceAsStream("index.jsp")  这个是获取这个文件的一个流InputStream
还有一个就是获取类路径的的资源：  
所谓类路径就是WEB_INF下面的classes文件夹以及lib文件夹下的jar包。注意不包含子文件夹。
* 使用ClassLoader
a.txt在src下面，首先就是获取ClassLoader，然后这个对象就可以访问类路径下面的任意内容
```java
ClassLoader loader=this.getClass().getClassLoader();
InputStream in=loader.getResourceAsStream("a.txt");
System.out.println(IOUtils.toString(in));
```
这样就可以直接获取src文件夹下面的一个a.txt文件，因为src最后直接出现在WEB_INF下面。
* 使用Class
a.txt文件直接出现在了该类的包中，而不是src下面。
```java
ClassLoader loader=this.getClass();
InputStream in=loader.getResourceAsStream("a.txt");
System.out.println(IOUtils.toString(in));
```
也就是以上的这两个方式他们的相对路径是不一样的，一个相当于WEB_INF一个相对于当前类所在路径。
