---
title: JavaWeb基础
date: 2017-08-09 15:49:28
tags: Web
categories:
 -JavaWeb
---
### 1. XML
1. xml一般就用来存放少量的数据，或者是作为配置文件。
2. xml的声明<?xml version="1.0" encoding="utf-8"?>  必须放在首行的首列（也就是顶头写）
3. 有且仅有一个根标签，其他的都是他的子标签
4. xml中的换行和空格都当做内容来解析，所以对于缩进来说一定要注意。
5. xml中的内容区分大小写，不能以数字和下划线开始,不能以xml开始，里面不能包含空格和冒号<!--more-->
6. 一个元素可以有多个属性，名字自定义，属性不能冲突
7. 对于一些特殊字符需要转义，使用实体来表示，就和html里面的一样
8. CDATA区就是里面的内容无需转义  <![CDATA[ content ]]>
9. 对于xml的内容的约束有两个 dtd和scheme两种
    --dtd：
        1.创建一个dtd文件
        2.有几个元素就写几个<!ELEMENT>
        3.看元素分别是简单元素还是复杂元素，就是嵌套否
            --简单元素<!ELEMENT 元素名 (#PCDATA) >
            --复杂<!ELEMENT 元素名 (子元素，逗号隔开) >
        4.在xml中引入dtd
            <!DOCTYPE 根元素名 SYSTEM "dtd文件路径">
10. xml的解析方法有两个分别就是dom和sax   （dom说白了就是构建dom树，这个方便的就是增删操作，但是文件过大可能造成内存溢出；sax则是从上往下读，一行行的解析，然后返回对象，但是不可以增删改，但是方便查询）
11. 三个常用的xml的解析器  ：dom4j(重要)  jaxp  jdom

### 2. HTTP 基础：
1. 请求头：
<pre>
请求行（GET /hello/index.html HTTP/1.1 ）
请求头 （多个键值对，其中有一个Keep-Alive这个东西是由于http无状态协议，所以说一个网站加载的时候肯定不止一次请求，像图片等等的东西，我们为了一次请求就让他多连接一会）
        （Content-Type:application-x-www-from-urlencoded  ->表示表单的数据会使用url编码）
一个空行
请求体 （只有POST才有请求体，GET没有）
</pre>
2. 响应头：
<pre>
    响应行 （HTTP/1.1 200 OK）  2开头的都是成功  3开头的重定向  4开头的客户端错误   5开头的服务器错误
    请求头 （Content-Type:text/html ;charset=ISO-8859-1 文本的话必须要有编码，像图片之类的可以不要）
    空行
    响应正文
</pre>    

3. Reference 显示的从哪个网站过来的一般的作用就是防盗链和统计广告

4. 重定向就是当客户端给服务器发送求以后，服务器返回了一个带有新的地址的返回，然后客户端去请求这个新的地址（302），而转发则是直接接通到新的服务器客户端不须在请求

5. 304就是缓存  首先浏览器发了一个Last-Modify头   然后服务器比对Last-Modify-Since  如果一样那么返回304  没有请求体

6. Tomcat是支持java web而不支持java ee  因为java ee范围很广泛  主要就是不支持java ee的JEB（企业级的java 应用）  但是我们有ssh可以代替

### 3. Servlet
1. Servlet的创建有三种方式，分别就是实现Servlet接口，继承GenericServlet和继承HttpServlet类
```java
public class AHttpServlet extends HttpServlet{
    /**
     * Http servlet首先就是继承的是GenericServlet，它里面有更丰富的方法，但是说到底还是要运行service方法，但是这个类里面的service方法其实有两个
     * 第一个是从上面继承下来的，另一个是自己的实际要用的，他们的不同就在于参数，自己的那个参数是与Http协议相关的，也就是说这个东西绑定了Http协议
     * 但是以前的那个参数是与协议无关的，但是最终tomcat要调用的是父类里面的service方法，所以说在继承的service方法中首先把参数都强转成http类型的参数
     * 也就是自己的service方法的参数，然后去调用自己的service方法，自己的service方法里面会判断来自客户端的请求究竟是post还是get然后调用doPost或者doget
     * 方法，所以说最终我们只需要复写doGet或者doPost即可
     * HttpServlet这个类是抽象的，但是里面的方法没有一个是抽象的，doGet和doPost默认都是给客户端返回一个405也就是不支持的请求类型，所以如果不复写
     * 这两个方法就会出问题405
     *
     */
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("doGet");

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("doPost");
    }
}
```
仿写的 GenericServelt 的代码
```java
public class SrcGenericServlet implements Servlet{
    private ServletConfig servletConfig;  //很重要的一个对象，里面有五个方法，前面已经说了两个方法就是关于初始化配置的两个
    //还有就是Context,以及getServletName()获取Servlet的名字
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        this.servletConfig=servletConfig;
        //这个就是钩子，如果我们写好的这个类然后有别人去继承，那么如果他想在init方法里面添加一些自己的功能
        //这个init有参数的如果被覆盖的话，子类中的servletConfig就会指向空指针，因为没有初始化。
        //所以为了拓展，子类能够写自己的代码我们写了一个新的函数，子类只需要覆盖这个函数就可以到到目的
        //不过还有一个方式就是在子类中我们先写自己的代码，然后使用super.init()也是能够搞定的
        init();
    }

    public void init(){

    }

    public ServletContext getServletContext(){
        return this.servletConfig.getServletContext();
    }

    @Override
    public ServletConfig getServletConfig() {
        return this.servletConfig;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {

    }

    @Override
    public String getServletInfo() {
        return "hello";
    }

    @Override
    public void destroy() {

    }
}
```
2. 首先说Servlet接口，这个接口里面总共有五个抽象方法需要进行复写，其中有三个声明周期方法init【在第一次访问这个Servlet的时候】 server【其他每一次访问的时候】 destroy【tomcat关闭的时候】
这个是单例的，一个类只有一个对象
而且这个是线程不安全的，所以效率高

3. Servlet要被浏览器访问必须要配置一下 web.xml 让 tomcat 能够调用这个 Servlet   具体就有两块分别就是Servlet的配置也就是配置Servlet的名字和类
  第二个就是配置浏览器访问的路径，和对应的Servlet的名称
  ```xml
<servlet>
    <servlet-name>AServlet</servlet-name>
    <servlet-class>Servlet.AServlet</servlet-class>
</servlet>
    <servlet-mapping>
        <servlet-name>AServlet</servlet-name>
        <url-pattern>/AServlet</url-pattern>
    </servlet-mapping>
```
或者直接在 Servlet 的类上面写 @WebServlet( name="", url-pattern="" ) 而不用写 web.xml 的配置

4. 由于Servlet是线程不安全的，所以说最好不要创建一些成员，而一般的创建局部变量。或者是无状态成员（一个类里面没有属性，然后这个类对象作为自己类成员），或者只读的成员，只有get没有set

5. 一个Servlet一般的就是第一请求的时候创建，也可以在tomcat启动的时候创建Servlet。就是在Servlet标签里面load-on-startup标签，里面是一个序号，不要重复

6. web.xml的继承，其实每一个项目都有一个web.xml，但是要注意每一个web.xml都是继承了tomcat的conf下的web.xml这个xml里面主要定义了三个东西。第一个就是默认Servlet
  这个默认的Servlet是匹配所有的页面，当我们访问一个页面的时候没有Servlet响应就轮到defaultServlet，他处理的方法也很简单就是返回一个404
  然后就是一个JSPServlet这个Servlet是用来处理所有的jsp页面的请求的，还有定义了很多的MIME类型


7. ServletConfig是javaWeb四大域对象之一，他们的功能就是在Servlet中传递数据，这个对象就是存取属性用于Servlet交互。
  getAttribute()   setAttribute()  removeAttribute()

8. 设置公共的初始化配置在Servlet标签外面写context-param

9. 获取的ServletConfig对象里面的getRealPath("/index.jsp)这个就是获取他在服务器上得真是位置，然后需要转化为流对象的话我们可以使用getResourceAsStream()直接返回的流对象

## 4. ServletContext 对象
&emsp;&emsp;  这个对象其实就是我们整个的项目，所以一般我们把它称之为 application ，他在整个项目中只有一个，并且他的生命周期和 Tomcat 一模一样。一般用它来做多个 Servlet 之间的数据传递。
&emsp;&emsp;  可以使用 ServletConfig 获取这个对象。或者使用 `getServletContext()` 直接获取 。我们还可以使用 HttpSession 获得这个对象。还能使用 ServletContextEvent 获取这个对象。
&emsp;&emsp;  可以获取 Web 根目录下的资源路径，也就是 webapps 下面的项目下的 web 目录，`getRealPath()` 、`getResourceAsStream()` ，同样的我们也可以使用 class 和 classloader 来获取类路径下的资源。
`className.class.getClassLoader().getResourceAsStream()` 或者 `className.class.getResourceAsStream()` 。

## 5. Servlet 四大域对象
首先说一下域对象的功能，简单来说就是一个用来在多个 Servlet 之间传递数据的对象，类似于一个 Map 容器。

1. PageContext
2. ServletRequest
3. HttpSession
4. ServletContext

他们都有 set/get/removeAttribute("",obj) 方法来存取数据。

### 4.反射：
1. 获取class，有三种方法：
    * 类名.class
    * 对象.getClass()
    * Class.forName("具体路径（包名）")
2. 对于一个类如果我们不使用new我们可以使用class来获取

```Java
    Class clazz=Class.forName("cn.lwen.Person");
    Person p=(Person) clazz.newInstance();  //p就是person的实例，这个显然也就是无参数的构造方法来实现的然后对于有参数的构造方法则是使用getConstructor来获取这个实例
    Class clazz=Class.forName("cn.lwen.Person");
    Constructor cs=clazz.getConstructor(类型的class（如：String.class）由于这个地方是可变参数，所以说参数的个说可以有多个)
    Person p=new (Person) cs.newInstance("lwen");  
```

3. 操作类的属性：类的属性是由一个叫做getDeclaredField()的方法获得的，得到的类型是Field类型，然后可以使用返回的类型的set方法类设置，第一个参数是；类的实例，第二个就是属性的内容
  但是这里只允许操作公共的变量，私有变量是不允许的，filed上面的一个setAccessible(true)则表示可以直接操作底油属性

4. 对于方法的操作就是和上面的属性很类似，首先使用getDeclaredMethod("",parm)获得方法然后返回的Method类型的，如果是私有的则和上面执行一样的操作，然后使用
  返回的Method对象的invoke(obj,parmar)执行函数

### 5. 两大重要对象：
#### 1. 在service这个方法中有两个很重要的对象response和request这两个，现在主要就说说这两个东西怎么给客户端请求以及响应

### response  
1. 首先就有一个发送状态信息的方法：response.sendError(404,"访问的页面存在但是不给你看");//发送一个错误的返回信息

2. 然后就是设置头信息，头信息就是键值对这里有三个常用的设置头信息的方法：  
        
```java
   setHeader("","")
   setIntHeader("",int)
   setDateHeader("",second)
```
然后这个方法把set改成add就是一个键多个值
设置重定向setRedirect("URL")  快捷的方法  也可使用设置头来做. `setHeader("Location","index.jsp")` 、 `setStatus(302,"ss")`

3. response里面的两个流，字符流和字节流，但是这两个流 不能同时存在否则会抛异常
        getOutputStream()
        getWriter()
        
4. 设置浏览器的缓存


### request  
1. 获取客户端的信息，获取客户端的url信息

2. 获取客户端的请求的参数，无论是get还是post对于单值的属性都可以使用getParameter("name")   然后对于多值则是getParameterValues()返回数组
      getParameterNames()获取所有的键名，是枚举类型

3. 请求转发和请求包含。
        请求转发：如果我们访问A，然后A做了请求转发到B，那么最终返回给客户端的就是保留了A的请求头和B的请求体（留头不留体）这次过程中url不变
        请求包含：同上，只是这次包含A的头和体还有B的体
        他们与重定向是不同的，浏览器不知道A请求了B，这始终就是一个请求
        
```Java
     request.getRequestDispatcher("/AServlet").forward(request,response);  //请求转发
            request.getRequestDispatcher("/AServlet").include(request,response);  //请求包含
            //request域   在Servlet中有三大域对象，在javaweb中有四个
            //分别就是request,session,application,Context
            //他们的生命周期就和这个对象的生命周期相同
            //他们都用相同的方法setAttribute()  getAttribute()  removeAttribute()  然后供其他的Servlet使用这些存在这些对象的信息
```

#### 2. 编码问题：
    编码分为响应编码和请求编码
    响应编码：就是服务器向客户端发送的数据的编码，我们首先要自己发送的是utf-8并且还要浏览器用utf-8去解析，我们采用的就是http头信息
              setHeader("Content-Type:text/html;charset:utf-8")  或者直接一个简单的方法就是setContentType("text/html;charset:utf-8")
    请求编码：则是服务器接受客户端过来的请求，进行编码，客户端默认使用的是iso，我们需要转成utf-8，而且这个地方是区分post和get请求方式的
              post就直接使用setCharacterEncoding("utf-8")而get就需要先转化成byte数组然后使用string转成utf-8
              byte[] by str.getBytes("ISO-8859-1");
              string =new String(byte,"utf-8")


### 6. JSP
#### 1. JSP三大指令 ： page  include  taglib
一个jsp页面可以使用多个指令，不一定放在第一行   
##### ---page：
       pageEncoding  设置页面的编码   在服务器把jsp编译成java时使用这个属性
       contentType   设置返回响应头  就和setContentType()方法一样
       以上两个属性只要设置了一个属性另一个属性可以不设置
       如果都不设置就是iso8859-1
       import 导包  可以用逗号隔开
       errorPage   当前页面如果抛出异常要转发到哪个页面，不是跳转  状态码200
       isErrorPage  设置当前页面是否为处理错误的页面   只有这个页面可以使使用9大内置对象的exception（当标签的内容为true）   状态码为500
       isELignore   是否忽略el表达式
##### ---include  静态包含
        他是在编译成java文件的时候完成的  他们共同编译成一个java文件  然后生成一个class
        他和ResponseDispatcher的include作用类似  只是他是动态包含  编译成两个不一样的文件class  最后运行的时候才合并  动态合并
        静态包含的作用就是页面分解
##### ---taglib
        引入标签库
        prefix引入前缀
        uri 标签的地址
 <%@ page .....%>
       在web.xml中配置errorPage    

```xml
<error-page>
    <error-code>404</error-code>
    <localtion>/error.jsp</localtion>
</error-page>
<error-page>
    <error-code>500</error-code>
    <localtion>/error.jsp</localtion>
</error-page>
<error-page>
    <exception-type>java.lang.RuntimeException</exception-type>   //页面抛出任何异常都会显示500   有事为了更加细致的分  我们可以定异常来处理
    <localtion>/error.jsp</localtion>
</error-page>

```


#### 2. JSP 九大内置对象和四大域对象：
* out   就是response.getWriter  用于想浏览器输出
* config   ServletConfig   就是xml中的内容
* page  当前jsp对象   引用类型就是Object    Object page=this
* pageContext  一个顶9个
* response   HttpServletResponse
* request    HttpServletRequest
* exception   异常
* session  
* application 整个tomcat存活期有效   ServletContext


Servlet三大jsp四大域对象  

* ServletContext   整个应用
* session  整个会话
* request  一个请求
-------------------jsp专有
* pageContext  一个jsp页面   用于jsp标签中的数据传递   代理其他域可以其他的域查找设置pageContext.setAttribute("","",PageContext.SESSION_SCOPE)   就是设置到session中
               全域查找  findAttribute("xx")   从小到大（域范围），依次查找



#### 3.JSP的动作标签：
<jsp:forward>   他和RequestDispatcher中的forward一样  就是一个在Servlet中使用一个在jsp中使用
<jsp:include>   同上  include方法一样
<jsp:param>   用来作为前两个的子标签 用来给包含的或者转发的页面传递参数


#### 4.JavaBean：
 必须要为成员提供get和set方法
 必须要有默认构造器
 某个属性有get和set方法但是没有该属性 也是可以的
 boolean的属性get方法可以直接就是is。。


#### 5.EL表达式：
 $(pageScope.xxx)   获取并输入该域中的xxx字段
 $(requestScope.xxx)   
 $(sessionScope.xxx)   
 $(applicationScope.xxx)   
如果不写那么就是全域查找

EL表达式就是用来输出的  用来代替<%= %>


##### EL的11个内置对象：
其中只有pageContext不是map类型  因为她就是pageContext类型其他的都是map类型   在El表达式中 Map的值访问可以使用点  或者数组方式  map['key']  map.key
上面已经有4个就是pageScope....
然后就是
param   key为参数名value为参数值   和getParam方法一样   适用于单值的
paramValues   适用于多值的  和上面的一样  value是一个数组
header:
headerValues:
这两个同上
initParam  获取<context-param></context-param>里面的参数
cookie  value是cookie对象  所以在获得cookie对象以后必须要使用value才能获得值
pageContext  例如${pageContext.request.contextPage}  获取request域中的contextPage内容



### 4.EL的函数库（由JSTL提供）
首先需要导入JSTL的函数库
${fn:substring("123123",1,2)}


### 5.自定义函数库：
  写一个java类  类中的方法可多个  但是必须为 public  static
  然后写tld文件,ide可以生成在xml下

### 8. JSTL ：
#### 1. 回顾：
<pre>
jsp：
    三大指令
    include  page   taglib
    九个内置对象
    response cookie out page pageContext request Exception config application
    动作标签
    forward include param
javaBean：
    默认构造器
    set，get方法

EL表达式：
    基本语法
    11个内置对象pageScope...Header，headerValuers,initParam,cookie,pageContext,param,paramValues
    函数库
</pre>    


#### 2.JSTL标签库：
    四大标签库：core核心库（重要），fmt（格式化），sql，xml（后两个过时）
    导入标签，使用taglib指令
#####    --core：【C标签】

* out <c:out value="aaa">字符常量  <c:out value="${aaa}">全域查找    default表示默认的值  escapeXml是否转义特殊字符  默认true
* set  <c:set var="" value="" scope="">   var变量名   value可以是EL表达式  scope域  默认page
* remove  <c:remove var="" scope="">  给出scope就删指定的   不然就是所有域
* url <c:url value="" >  在路径前面自动加项目名  但是value一斜杠开头  <c:param name="" value=""> 子标签用于传参，并对参数进行url编码   var参数用来保存生成的url而不进行输出，否则就直接输出
* if  <c:if test=${not empty a}/>  判断a不是空就输出  在全域内查找   test中放bool类型
* choose <c:choose />  表示if....elseif()..else  这个标签主要用于容纳when标签  when标签里面有test属性
* forEach <c:forEach var="" begin="" end="" step=""></c:forEach> 包含头也包含尾  items 需要遍历的对象(这里一定不要有空格) var 迭代到这个变量上
        循环状态变量   vsStatus=“vs”  vs.index  vs.count   vs.first  vs.last vs.current
    --fmt标签：【格式化标签】
        <fmt:formatData value="" pattern="">  例如 yyyy-MM-DD
        <fmt:formatNumber value="" pattern="">  例如0.00  两位小数  四舍五入  

#### 3.JSTL自定义标签
    1. 步骤：
        * 标签处理类  实现SimpleTag接口  或者继承simpleTagSupport他做了很多处理 tomcat传 给simpleTag的参数都被保存了
        * tld文件（xml）标签和类相关联  都放在WEB-INF中  防止别人直接访问
        * 在页面中引入
    2. 标签处理类
        * 这些方法都是有tomcat调用  最后调用的是doTag方法  和Servlet非常类似
        * doTag  执行标签的时候调用  
        * getParent 获得父标签
        * setParent  设置父标签
        * setJspBody  设置标签体
        * setJSPContext  设置pageContext
    3. 跳过某部分
        直接在处理类中抛出一个跳过页面异常，最终编译的代码就是一个标签一个函数，由于这个函数也是直接抛出异常，所以该标签后面的内容就不执行了
        而是跑到调用该函数的try块中的catch中执行响应的方法，这个catch对SkipPageException特别的照顾了一下，然后就结束了
    4. 自定义标签属性：
        1. 在处理类中创建属性
        2. 在tld文件中定义    

#### 4.三层架构
mvc是bs架构的公共的东西
而三层架构则是java web的东西：
    web层  与web相关的  Servlet  jsp
    业务层 功能【登陆，注册，转账 等等...】(service)
    数据层（dao 【data access Object】）(dao)


