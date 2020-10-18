---
title: Java Web 基础二:Request和Response对象
date: 2017-08-20 22:01:13
tags: Java Web
categories:
  -Java Web
---
### 1.Response
这个是服务器返回给客户端的对象，然后被tomcat解析以后发送给浏览器。
#### 1.与响应头相关的方法
* sendError() 发送错误状态码 如404 405 等等
* setIntHeader("key",value) 设置适用于int类型的响应头
* setHeader("key","value") 设置头
* setStatus() 设置状态码例如302 301 200 等
#### 2.与响应体相关的对象
* getOutputStream  获取一个向浏览器输出的字节流对象
* getWriter 获取一个浏览器输出的字符流对象
#### 3.重定向
* sendRedirect("url")  直接重定向，底层第调用了setHeader("Location","url")和setStatus(302)这两个方法。

### 2.Request
#### 1.获取客户端的信息
* getRemoteAddr() 获取对方的ip
* getMethod 获取请求方式get/post
#### 2.获取请求头
* getHeader()
* getHeaders()
#### 3.获取请求url
* getScheme 获取协议
* getServerName 获取服务器地址
* getServerPort 获取端口
* getContextPath 获取项目名路径，有斜杠
* getServletPath 获取Servlet路径，有斜杠
* getQueryString 获取参数字符串
* getRequestURI  获取请求URI 也就是项目名加上Servlet路径
* getRequestURL  获取请求URL 不包含参数的整个路径
#### 4.获取请求体
* getParameter  返回string
* getParameterValues  返回string[]
* getParameterNames 返回string[]
* getParameterMap 返回map<string,string[]>

### 3.请求转发请求包含：
总的来说就是一个请求，但是有多个Servlet来完成这个请求。但是如果是重定向的话就是直接多个请求。  
请求转发：最终返回给客户端的信息是原Servlet的响应头加上被转发的Servlet的响应体
请求包含：返回的东西是两者的响应头和响应体的组合。
```java
RequestDispatcher dispatcher=request.getRequestDispatcher("/index.jsp");
dispatcher.forward();//转发
dispatcher.include();//包含
```

### 4.request域
Servlet的三大域对象：
* request 一个请求范围内
* session session存活时期
* application 整个服务器存活时期  也就是ServletContext对象

## 5.编码



#### 1.服务器响应编码  (服务器给客户端发送的数据)
服务器响应编码就是在服务器响应客户端数据，然后在客户端上显示数据的过程。但是注意这个地方分了两步，首先就是服务器在向客户端发送数据之前需要设置服务器的响应编码也就是服务器使用怎样的解码方式来把字符串变成字节，默认采用的是 iso。之后客户端接收到了以后还需要知道使用怎样的编码去编码还原成字符串，否则客户端使用了错误的编码之后还是会导致乱码。
其中设置服务器编码的方法是
```java
Response.setCharacterEncoding("utf-8");
```
而告诉客户端的解析方式，也即是告诉客户端服务器使用的解码方式：
```java
Response.setHeader("Content-Type:text/html;charset=utf-8");
```
这样才能达成一致，保证不会出现问题。注意其实我们调用了第二个方法的时候就不用调用第一个方法，因为他会自动的调用第一个方法。
有一个更加方便的方法来完成这个编码工作，只需要依据就能同时设置编码方式和客户端解码方式，他和 setHeader 是等效的。
```java
Response.setContentType("text/html;charset=utf-8");
```
这样一句话就能搞定。

#### 2.请求编码
请求编码就是客户端提交给服务器的数据，也就是我们使用getParameter方法获取的get或者post数据。  
一般在地址栏的参数都是gbk的，而在页面中点击链接或者提交表单的时候一般就是utf-8，其实最终还是看你这个页面的编码，就是看你的 html 里面的 `<mate content-type="utf-8"/ >` 头
这个其实和上面的是类似的，都是客户端先编好码，然后服务器需要解码，也就是我们还是需要设置两个方面的编码。页面的编码显然就是使用一个mate头设置成utf-8就可以，但是服务器的getParameter这个方法默认使用的就是iso编码所以在此之前需要设置。
##### 1.post方式
```java
Request.setCharacterEncoding("utf-8")
Request.getParameter("");
```
##### 2.get方式

也就是先进行使用默认的字符集进行解码，然后在使用 utf-8编码。

```java
string name=Request.getParameter("");
byte[] bytes=name.getBytes("iso-8859-1");
name =new String(bytes,"utf-8");
```

### 6.URL编码
在使用form表单提交数据的时候他会把数据进行URL编码，然后服务器会自动识别并且解码。为了方便传输，早起的浏览器直接发会出问题，所以使用了这种方式，减少数据的丢失。

### 7.服务器路径
#### 1.请求转发和请求包含（服务器端的URL）
* 以 / 开头，就是相对于   localhost:8080/项目名/  [绝对路径 —> 最常用]
* 不以 / 开头，就是相当于当前的Servlet  [相对路径]
#### 2.重定向，超链接，表单(客户端的URL)
* 以 / 开头，相对于当前的主机  localhost:8080/
* 不以 / 开头，就是相对于当前的html页面
