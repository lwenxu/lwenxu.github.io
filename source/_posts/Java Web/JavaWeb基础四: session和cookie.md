---
title: JavaWeb基础四:session和cookie
date: 2017-08-21 10:46:59
tags: Java Web
categories:
  -Java Web
---
## 1. Cookie （Cookie 对象）
#### 1. cookie的存取  

Cookie是存放在客户端的小文件，文件里是服务器保存的一些数据。
设置cookie，首先需要创建一个cookie然后保存。
```java
Cookie cookie=new Cookie("key","value");
response.addCookie(cookie);
```
这样就已经向客户端保存了cookie，然后需要获取cookie。 response.getCookies()这是一个cookie数组，需要便利之。cookie[]



#### 2.cookie的存取时间
maxAge > 0 浏览器关闭后制定的秒数死亡
maxAge < 0 浏览器关闭后立即死亡
maxAge = 0 立即死亡

```java



​```java
cookie.setMaxAge(10);
```
#### 3.cookie的path
注意这个path并不是存放在磁盘上的路径，而是项目访问路径。如果满足这个路径的话，Request时就会设置一个cookie的头里面就是cookie的值。  注意这个路径是包含就满足，也就是访问路径包含了cookie的路径就会返回给服务器对应的cookie。比如下面几个例子

```tiki wiki
http://baidu.com/hello     --->  cookie: http://baidu.com/hello/index/  不包含 cookie 匹配失败
http://baidu.com/hello/index     --->  cookie: http://baidu.com/hello  	包含 cookie 匹配成功
```



## 2.Session (HttpSession对象)
session是存放在服务器端的文件，用于跟踪会话。但是注意HttpSession是和http无关的是java web提供的对象。session对象还是四大域对象之一，直接就可以使用这个对象。
#### 1. session的获取
在Servlet中获取HttpSession对象
```java
HttpSession session=Request.getSession();
session.setAttribute();
```
而在jsp中他是属于九大内置对象的，我们直接使用即可，无需创建。
#### 2. HttpSession的原理
在访问网站的时候，服务器并不会马上就创建session而是等到我们调用了request.getSession()方法的时候才开始创建，但是也不一定就是直接创建，而是分几种情况：
获取Cookie中的JSESSIONID
* 1.如果有JSESSIONID不存在就创建SESSION然后保存JSESSIONID，是保存到了 cookie 中，如果禁用了他就通过 url 重写。服务器会通过 JSESSIONID 来找服务器中对应的 session 文件。
* 如果有JSESSIONID，有SESSION的话就不会创建，没有SESSION就会继续创建。
在Servlet中我们需要调用request.getSession()才会创建session，但是jsp中我们不需要做这件事，只要访问了jsp立即就会创建session，因为jsp在被包装的时候就已经调用了request.getSession()方法，因为session是九大内置对象。

#### 3. session的URL重写
就是当我们的cookie被禁用了以后我们需要JSESSIONID，但是没有地方获取，这里我们这个就会自动使用URL重写，也就是在每一个URL中自动添加了一个JSESSIONID的参数。也就是说 session 就是这样的。

#### 4. session 的序列化

也就是当我直接关闭服务器的时候在服务器的 temp 中会有一个 SESSION 的临时文件。再次启动的时候就会加载这个文件恢复 SESSION 。