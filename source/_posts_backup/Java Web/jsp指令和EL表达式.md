---
title: jsp指令和EL表达式
date: 2017-08-21 11:22:27
tags: java
categories:
  -JAVA
---
### 1.page指令
#### 1.pageEncoding
指定jsp的编码
#### 2.contextType
设置响应头
这两个东西其实使用任意一个即可。<!--more-->
#### 3.errorPage
如果这个页面抛出异常以后重定向到哪一个页面。
### 2.静态包含 include
他是静态包含和RequestDispatcher类似，但是就是包含的时期不一样。  
静态包含就是在jsp编译成java的时候形成的，也就是最终是两个文件合并成了一个class，最后形成一个class文件
RequestDispatcher则是动态包含，他们在显示之前始终是两个java文件，两个class，在最后需要显示的时候才把内容整合发送给客户端。
### 3.导入标签库 import
prefix制定标签库的前缀，uri就是标签库的位置。
### 4.九大内置对象
* out  jsp的输出流，向浏览器输出数据
* page  当前的jsp对象，也就是在编译成大java中有page=this
* config  对应的servletConfig对象
* pageContext  包含其他的所有 域对象之一
* request   
* response
* exception  
* session HttpSession
* application  servletContext

### 5.四大对象作用范围
* ServletContext  一个应用
* Session 一个会话
* Request 一个请求
* pageContext 一个jsp页面 ，一般用来jsp标签的数据传输

### 6.pageContext作用
1. 代理其他的三大域对象
pageContext.setAttribute("key","value",pageContext.SESSION_SCOPE); 存放在session中代理了session
2. 全域查找
pageContext.findAttribute("key") 在这四大域对象中依次查找
3. 获取其他的jsp八大内置对象

### 7.JSP动作标签
* <jsp:forward page=""/>  转发
* <jsp:include page=""/>  包含
* <jsp:param name="" value=""/>  为其他的标签传递参数

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
* param
* paramValue
* header
* headerValue
* cookie
* pageContext  他可以获取其他所有的10个对象，因为它里面有其他十个对象的get方法。
4. EL 函数库
导入对应的库，然后使用标签调用函数库

### 9.JSTL
JSTL是EL的扩展，因为EL只是进行输出而已，但是有一些判断，遍历等等，这些操作就是JSTL。他需要引入jstl.jar
他有四大库，但是常用的只有两个一个是core另外一个就是formate标签库
注意导入的时候uri是jsp/core  或者 jsp/formate
#### 1.core标签库(c标签)
* out 输出标签  value就是要输出的变量
* set 设置某个变量的值  var变量名  value变量值
* url url格式化的标签  value 自动添加上项目名  里面如果加param标签那么就可以传递参数 name/value
* remove 删除域变量  var变量名  scope域范围，不写的话删除全域的对象中的此值
* if if语句  test 判断的条件 ${not empty key} 如果key不是空
* forEach
  * 计数方式  var循环变量  begin循环变量从几开始  end到几结束  step设置步长
  * 用来遍历  items需要迭代的变量  var每一次迭代的变量
* choose/when  多分支  
```xml
<c:choose>
  <c:when test></c:when>
  <c:when test></c:when>
  <c:when test></c:when>
  <c:when test></c:when>
</c:choose>
```
#### 2.formate标签库
* formateDate  value需要格式化的变量   pattern yyyy-MM-dd HH:mm:ss
* formateNumber value变量   pattern 0.00 需要小数点两位  四舍五入  
