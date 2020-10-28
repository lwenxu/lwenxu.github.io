---
title: SpringMVC 基础
date: 2018-04-18 22:01:28
tags: SpringMVC
categories:
 - SpringMVC
---

## 1.SpringMVC 基础原理

1. C 	前端控制器 ——> DispatcherServlet
2. M    数据对象
3. V      视图处理器ViewResvor

<!—more-->

### 处理步骤：

1. 发起请求到前端控制器 `DispatcherServlet` 
2. 然后这个控制器会调用 `HandlerMapping`  查找对应的 `Controller`或者说 `Handler` 
3. 找到了对应的 `Controller` 就让 `HandlerAdaptor` 去执行 `handler` 
4. 执行了 `handler` 以后返回的就是 `ModelAndView` 对象。
5. 对象返回给前端控制器，然后前端控制器会丢给 `ViewResovr` 去解析
6. 然后继续返回给前端控制器并返回给用户。



## 2.基础程序 

#### 1. 配置基础的 web.xml 加载 SpringMVC

在 web.xml 中我们需要配一个 Servlet 和一个 Listener ，这个 Servlet 其实就是我们的路由调度器，然后 Listener 则是上下文监听器。还有一个初始化参数就是指定 spring 的配置文件的位置。

其实这些配置基本都是固定的，在使用 idea 建立 SpringMVC 项目的时候他会自动的帮我们配置好，但是我们还是需要在进行一些配置。主要的配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--配置 spring 的配置文件-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <!--1. /  所有的都进行解析，但是不管 jsp
            2. *.from
            3. /* 不能用 所有的都拦截
        -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>


    <filter>
        <filter-name>HttpHiddenMethods</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>HttpHiddenMethods</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

</web-app>
```



### 2. 配置基础的 spring 配置文件

接着就是配置 spring 配置文件，可以看到在上面的 web.xml 中我们在初始化参数中指定了 spring 的配置文件就是 applicationContext.xml 放在了 WEB-INF 路径下面。

然后需要配置一些核心的 bean 让 spring 进行自动加载，以需要配置扫描的包。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!--&lt;!&ndash;处理器映射器&ndash;&gt;-->
    <!--<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>-->
    <!--&lt;!&ndash;处理器适配器&ndash;&gt;-->
    <!--<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>-->

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB_INF/templates/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <!-- 配置需要扫描的包-->
    <context:component-scan base-package="com.mvc"/>
    <!--必须要的，配置了 springMVC 的注解生效，不然的话我们的 url 映射还是通过配置文件的方式生效的
        不然一直处于 404 状态，这也就是下面的 dispatcher-servlet.xml 的作用配置 url 映射
    -->
    <mvc:annotation-driven />
</beans>
```

### 3. 控制器

```java
@Controller
public class Test {
    
    @RequestMapping("/helloTest")
    public String hello(){
        return "hello";
    }
}
```

这样我们的程序就能够跑起来了。

## 3. RESTful 风格的请求

一般的我们无法直接使用 RESTful 风格的请求，但是在 SpringMVC 中有一个过滤器可以帮我们把一个 post 请求转化成为 `PUT` 或者 `DELETE` 请求。具体的做法如下：

#### 1. 配置过滤器（HiddenHttpMethodFilter）

​	

```xml
    <filter>
        <filter-name>HttpHiddenMethods</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>HttpHiddenMethods</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```



#### 2. 在 post 的表单域中封装一个 hidden 的 input 标签，用来说明提交方式

```html
<form method="post">
    <input type="hidden" name="_method" value="PUT">
</form>
<form method="post">
    <input type="hidden" name="_method" value="DELETE">
</form>
```

## 4. @RequestMapping 及参数传递

### 1. @RequestMapping 注解

这个注解主要就是用来做方法和类的 url 映射的，也就是相当于一个路由映射文件。这个注解可以使用在类上面也可以使用在方法上面，在类上面的话就是我们访问每一个方法的时候都需要加上类的 url 前缀。

他有几个比较重要的属性，用来管理 url 的：

1. value: 这个是默认的，是 url 地址

2. method：这个是用来指定请求方式的 RequestMethod.GET/POST/DELETE/PUT ….

3. params：这个是用来规定我们的 url 中携带的参数的他是一个数组，可以放多个值 

   ```java
   params = {"username","age!=10"}  params 是包含 username age!=10   headers 也是如此只是规定了请求头而已
   ```

### 2. @PathVariable 注解

这个注解的作用是用来传递参数的，我们不仅仅可使用 `？` 来传递参数，还可以使用更优雅的 `/paramName/value` 的方式来传递参数，并且能够直接在方法中绑定这些参数。

```java
@RequestMapping(value = "/helloworld/{id}") 
public String hello(@PathVariable("id") String id){
    System.out.println("hello controller");
    return "hello";
}
```

这里的 id 必须和 url 中的保持一致，但是无需和方法的参数保持一致。

### 3. @RequestParam 注解

但是有一个问题，当我们传递过来的参数是通过 `？` 的形式传递过来的，那么我们又该怎么去获取他呢？是的这里我们可以使用 `@RequestParam` 方法来获取这些值，当我们有些参数不是必须传递的我们就可以使用 `require=false` 来规定不用必传，这个时候如果我们没有传值，这个参数刚好是一个引用类型的就会是 `null` 但是如果是一个基本数据类型，就会报错。我们必须手动的设置一个默认值。

```java
@RequestParam(value = "age",required = false,defaultValue = "0") Integer age
```

上面的参数就可以获取到 `http://localhost/hello?age=10` 这种的 url 的参数了、

### 4. @RequestHeader 注解

用法同上！

```java
@RequestHeader("Content-Type") String content
```

### 5. @CookieValue 注解

这个使用同上，获取 Cookie 的值。

### 6. POJO 参数（Java 类）

我们的表单提交过来的数据会自动的被封装到 POJO 对象中，我们只需要配置好表单的 name 值和 Bean 的属性值一致即可，如果说里面有级联的属性，我们就使用 `proA.proB` 来封装。例如：

```html
<form method="post">
    <input type="text" name="name"/>
    <input type="text" name="age"/>
    <input type="text" name="address.code">
    <input type="text" name="address.name">
</form>
```

这个表单就会被封装成一个 POJO 对象，这个对象里面有另外一个类的引用就导致了级联属性的出现，我们是就是使用了点的方式完成的封装。

### 7.原生的 Servlet API 参数

它支持比较多的原生的 Servlet 的 API ，其实是在他内部调用了 request 对象的一些方法获取到的。

1. HttpServletRequest
2. HTTPServletResponse
3. HttpSession
4. Locale
5. InputStream
6. OutputStream
7. Reader
8. Writer
9. Principal

## 5. 控制器与视图间数据交互

一般我们需要在控制器里面绑定一些数据到视图中，然后我们可以在视图里面采用标签来获取 Controller 里面的数据从而展示这些数据，在 SpringMVC 中有几种方法可以达到这个目的。

### 1. ModelAndView

这个东西其实就像他的名字一样，他是 Model 数据和模型的结合体，我们可以往里面添加数据（Model），也可以把要转发的页面放在里面让视图解析器去渲染。所以说这个对象里面有一个 Model 属性，这个属性就是用来存放数据的，其实就是一个 Map 。Map 里面的这些数据都会被遍历然后放到 request 域对象之中，我们只需要在请求域中获取就好。

```java
    /**
     * ModelAndView 来用作 Controller 与 View 之间的数据交互的介质
     * 也就是 Model 的载体
     * @return
     */
    public ModelAndView modelAndViewTest(){
        ModelAndView modelAndView = new ModelAndView("hello");
        modelAndView.addObject("time", new Date());
        return modelAndView;
    }
```

### 2. Model/Map/ModelMap

其实三个东西类型都是 Map 类型的，然后SpringMVC 在真正的传入的对象显然就是他们的实现类，这里我们不过分纠结，基层肯定是一个 Map 。Map 里面的这些数据都会被遍历然后放到 request 域对象之中，我们只需要在请求域中获取就好。

这个用起来也比较简单，就是在方法的入参里面传入这个一个东西就行了，而不是采用的返回值。Map 的具体的泛型就是 string 和 object。看下面的例子。

```java
 public String modelMap(Map<String,Object> map){
        map.put("time", new Date());
        return "hello";
    }
```



### 3. @SessionAttributes 注解

这个注解只能放在类上面，然后我们使用 value 属性或者 types 属性来规定哪些属性需要被放在 Session 域中，这个两个属性其实都是一个数组，所以我们可以方多个值。

value 这个属性，就是当我们在放入 map 中的一个键名的时候我们就可以把它放到 session 域中。而 types 属性则是当我们放一个 class 的时候他会自动抓取处于 map 中的同类型的数据。

```java
@SessionAttributes(value = {"time","username"},types = {String.class,Integer.class})
```

### 4. @ModelAttribute 注解

这个注解是标识在方法上的，这个注解标识的方法会在所有的方法调用前被调用。在这个被标识的方法里面我们需要从数据库中获取对应的对象，然后把这个对象放到 map 里面，但是注意 map 中的键必须要是我们的 Model 类对应的小写的一个字符串才能起作用。当我们使用其他的方法来进行某个 model 的修改动作的时候我们某个字段不传的话这时候在 map 中的那个对象的对应字段挥起一个补充作用，把对应字段填上。

执行流程：

1. 首先是执行了被这个注解标识的方法，将数据库中获取到的值放到了 map 里面，然后这个 map 是被放到了一个implicitModel 里面
2. 然后在我们提交一个表单的时候，我们对应的方法的参数会到 implicitModel 里面查找对应的对象，查找的 key 就是先看看我们的这个方法的参数是否被 ``@ModelAttribute(value="...")` 修饰。如果是的话我们采用的是直接使用它的 value 属性作为 key 去查找。
3. 如果没有这个注解修饰参数，则采用这个 POJO 的类名第一个字母小写作为 key 查找。
4. 如果没有则看看是否这个类被  @SessionAttributes 注解 注释，如果是则是去 session 中查找，如果没有找到抛异常。
5. 如果上面的情况都没找到，则是使用 POJO 反射创建一个新的对象把表单数据封装进去，而如果上面有找到的话我们就使用那个  Model 然后设置对应的属性值。
6. 接着把这个修改后的 model 放到 implicitModel 进而放到 request 域中。

## 6.视图

视图的解析步骤：

1. 首先我们是访问了我们的控制器。
2. 然后我们的控制器会返回 string 或者 VIewAndModel 对象。
3. 他们都会被视图解析器（我们在 spring 中配置的 bean）包装成一个 ModelAndView 对象。
4. 最后渲染视图，并转发到对应的视图。

### 1. 手动路由配置

可以手动配置路由，不经过 controller 就可以访问到对应的视图。在 spring 配置文件里写上如下内容：

```xml
    <mvc:annotation-driven />
    <!--手动配置路由   直接路由到视图无需经过 controller-->
    <mvc:view-controller path="/success" view-name="hello"/>
```

那么我们访问 `http://localhost:8080/success` 就被转发到 WEB_INF/templates/hello.jsp  具体的目录取决于我们配置的视图解析器的前缀和后缀。

### 2. 自定义视图解析器

我们一般采用的就是 `InternalResourceViewResolver` 这个视图解析器，我们也可以自定义视图。自定义视图则需要一个特殊的视图解析器完成解析视图的工作，就是 `BeanNameViewResolver`  就是通过视图的 bean 的 name 来获取视图的。由于我们配置了多个视图解析器则需要定义一个优先级，哪个视图解析器先工作，使用 order 属性。

```xml
<!--自定义的 Bean视图解析器   直接通过 bean 的 name 获取视图-->
<bean class="org.springframework.web.servlet.view.BeanNameViewResolver">
    <property name="order" value="100"/>
</bean>
```

下面是我们使用 bean 定义的一个视图。

```java
@Component
public class MyViewRevsor implements View {
    @Override
    public String getContentType() {
        return "text/html";
    }

    @Override
    public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        response.getWriter().write("hello");
    }
}
```

### 3. 转发和重定向

我们一般在 controller 中写的东西默认都会转发的，我们需要重定向的话我们只需要在人绘制前面加上 redirect 就可以。

```java
    @RequestMapping("/beanView")
    public String beanView(){
        return "redirect:/myViewRevsor";
    }
```

### 4. 静态资源映射

对于静态资源我们需要直接获取而不需要进行映射，所以说我们在获取静态资源的时候回出现 404 ，我们就配置一个

`<mvc:default-servlet-handler/>` 这个就会自动的处理没有映射的 url 。

### 5. 表单数据到 controller 的映射

表单数据在提交以后实际上是依赖于 SpringMVC 里面一个 WebDataBinder 类进行的数据绑定，这个 WebDataBinder 里面有很多其他的对象的引用其中就有数据格式化、数据校验、数据转换的对象，也就是说在这个数据转换的过程我们是可以添加一些对象来手动的控制数据的绑定的。

1. converter 这个是用来数据转换的，具体的参照文档，就比如我们把前端的一个字符串转成方法入参的一个 bean 对象，就经过这个 converter 来完成。
2. @initBinder 被这个注解表示的方法，会在数据绑定之前进行运行，其功能就是对 binder 进行一些设置比如忽略一些字段。修改字段，拒绝字段等等。

```java
  @InitBinder
    public void initBinder(WebDataBinder binder) {
        // 拒绝 name 字段
        binder.setDisallowedFields("name");
    }
```

3. 数据的格式化，采用注解注解在对应的 bean 的字段上。常用的有时间还有数子。
4. JSR303 校验规范，这只是 JavaEE 的规范，真正的实现类是 Hibernate ValidData ，然后我们进行数据校验也是使用注解的方式，具体的注解在规范里面都有，都比较简单。并且我们需要在方法的入参的 bean 上加上 @Valid 注解。
5. 错误消息的回显，显然如果我们的校验生效并且有错误的话我们需要回显到表单。我们就需要在方法的参数里面加上一个 BindResult 对象，然后错误的数据都会被放到这个东西里面，同时 BindResult 是一个 Error 类型的对象，所以我们亦可以放这个对象。最后放到 map 里面在前端回显即可。

### 6. 返回 JSON 数据

只需要在方法上加上 @ResponseBody 就可以，返回值是一个 List 或数组。

## 7. 拦截器（Interceptor）

```xml
<!--配置拦截器-->
    <mvc:interceptors>
        <!--自定义的拦截器组件-->
        <bean class="com.mvc.MyInterceptor"/>
        <!--更详细的配置可以针对 url-->
        <mvc:interceptor>
            <mvc:mapping path="/hello"/>
            <mvc:exclude-mapping path="/hah"/>
            <bean class="com.mvc.MyInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>
```

```java
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```



## 8. 统一异常处理

统一异常处理就采用对应的 handler 来处理异常，使用 @ExceptionHandler 注解，然后标注要处理的异常类型，但是如果说我们需要把异常带到错误页面我们不能使用 map 而只能使用 ModelAndView 不然那就会报错。

```java
@ExceptionHandler({ArithmeticException.class})
public ModelAndView error(Exception ex){
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.addObject("ex", ex);
    return modelAndView;
}
```

如果他在当前的 controller 中找不到对应的 ExceptionHandler 就去查找对应的 @ControllerAdvise 注解表示的类，中的ExceptionHandler 注解方法。也就是默认的处理器。