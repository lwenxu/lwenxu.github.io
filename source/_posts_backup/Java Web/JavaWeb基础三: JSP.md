---
title: JavaWeb基础二:JSP
date: 2017-08-21 10:26:18
tags: Java Web
categories:
  -Java Web
---
## 1. JSP 标签

我们常用的 jsp 标签有两种，实际上有三种 jsp 标签。

1. <% %> 这种就是可以放 java 代码
2. <%= %> 这种就是输出语句，类似 PHP 中的简写语法
3. <%! %> 放各种生命代码，基本不用。

其实JSP最终还是一个Servlet，主要他的优点在于，可以在一个Servlet中直接写html代码，防止我们去写很多 out.print("html代码") ，或者说我们可以在html中写 java 脚本。但是并不是说我们使用了jsp 就不再使用Servlet，他们之间是有区别的，分工不一样。

* jsp主要就用来显示表单，显示数据结果
* Servlet就是用来处理数据，返回给jsp
## 2. JSP 原理
服务器启动后的第一次访问时，服务器会把这个文件转化为java文件，并且这个java文件是实现了servlet接口的一个类然后再把此java文件，转化成一个class文件，实例化对象，然后调用service方法。而在第二次访问的时候就是直接调用service方法。这也就是第一次惩戒。jsp被编译成的java放在了tomcat的work目录下面。  
具体的看一下jsp编译出来的文件的时候就会发现里面是有六个private对象，还有两个分别就是request和response被tomcat传入。也就是有8个对象 ，还有一个exception没在这。所以这里就知道了jsp的九大内置对象。

## 3. JSP指令

### 1. page指令

##### 1. pageEncoding

指定jsp的编码

##### 2. contextType

设置响应头这两个东西其实使用任意一个即可。

##### 3. errorPage

如果这个页面抛出异常以后重定向到哪一个页面。

##### 4. IsErrorPage=true

标记这个页面可使用 expection 对象，其他页面不行。

##### 5. session="true"

当前页可使用 session 对象

### 2. 静态包含 include

他是静态包含和RequestDispatcher类似，但是就是包含的时期不一样。  静态包含就是在jsp编译成java的时候形成的，也就是最终是两个文件合并成了一个class，最后形成一个class文件。
RequestDispatcher则是动态包含，他们在显示之前始终是两个java文件，两个class，在最后需要显示的时候才把内容整合发送给客户端。

### 3. 导入标签库 import

prefix制定标签库的前缀，uri就是标签库的位置。

### 4.九大内置对象

- out  jsp的输出流，向浏览器输出数据
- page  当前的jsp对象，也就是在编译成大java中有page=this
- config  对应的servletConfig对象
- pageContext  代理其他的所有域对象，在标签之间传数据。
- request   
- response
- exception  
- session （HttpSession）
- application  （servletContext）

### 5.四大对象作用范围

- ServletContext  一个应用
- Session 一个会话
- Request 一个请求
- pageContext 一个jsp页面 ，一般用来jsp标签的数据传输

### 6.pageContext作用

1. 代理其他的三大域对象
   pageContext.setAttribute("key","value",pageContext.SESSION_SCOPE); 存放在session中代理了session
2. 全域查找
   pageContext.findAttribute("key") 在这四大域对象中依次查找
3. 获取其他的jsp八大内置对象

### 7.JSP动作标签

- <jsp:forward page=""/>  转发
- <jsp:include page=""/>  包含
- <jsp:param name="" value=""/>  为其他的标签传递参数

### 8.EL表达式

EL表达式主要就是用来代替JSP中的 <%= %> 这个标签的，他可以简单的用于输出语句

1. 输出四大域对象中的内容
   ${key}  这样就可以全域查找到四大域对象中的key变量  如果key是一个对象的话，我们希望获取这个对象里面的某个get方法的返回值，我们只需要key.key1  key1就是getKey1()这个方法的返回值。
2. 精确的四大域对象查找
   ${pageScope.key}
   ${requestScope.key}
   ${sessionScope.key}
   ${applicationScope.key}
3. 其他7个内置对象

- param
- paramValue
- header
- headerValue
- cookie
- pageContext  他可以获取其他所有的10个对象，因为它里面有其他十个对象的get方法。

1. EL 函数库
   导入对应的库，然后使用标签调用函数库

### 9.JSTL

JSTL是EL的扩展，因为EL只是进行输出而已，但是有一些判断，遍历等等，这些操作就是JSTL。他需要引入jstl.jar
他有四大库，但是常用的只有两个一个是core另外一个就是formate标签库
注意导入的时候uri是jsp/core  或者 jsp/formate

#### 1.core标签库(c标签)

- out 输出标签  value就是要输出的变量
- set 设置某个变量的值  var变量名  value变量值
- url url格式化的标签  value 自动添加上项目名  里面如果加param标签那么就可以传递参数 name/value
- remove 删除域变量  var变量名  scope域范围，不写的话删除全域的对象中的此值
- if if语句  test 判断的条件 ${not empty key} 如果key不是空
- forEach
  - 计数方式  var循环变量  begin循环变量从几开始  end到几结束  step设置步长
  - 用来遍历  items需要迭代的变量  var每一次迭代的变量
- choose/when  多分支  

```xml
<c:choose>
  <c:when test></c:when>
  <c:when test></c:when>
  <c:when test></c:when>
  <c:when test></c:when>
</c:choose>
```

#### 2.formate标签库

- formateDate  value需要格式化的变量   pattern yyyy-MM-dd HH:mm:ss
- formateNumber value变量   pattern 0.00 需要小数点两位  四舍五入  