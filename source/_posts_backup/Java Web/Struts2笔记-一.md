---
title: Struts2笔记(一)
date: 2017-08-22 18:52:06
tags: Struts2
categories:
  -JAVA
---
### 1.创建Action
创建action需要继承ActionSupport类，然后就会有一些常量以及一些方法。   
struts2的action默认执行的方法就是execute()方法<!--more-->
```java
package demo01;

import com.opensymphony.xwork2.ActionSupport;

public class Hello extends ActionSupport{
    public String execute(){
        return SUCCESS;
    }
}
```
### 2.配置核心配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
        "http://struts.apache.org/dtds/struts-2.3.dtd">

<struts>
  <!-- name随便配  extends就只能是这个  namespace是将来要访问的路径的前缀 -->
    <package name="struts" extends="struts-default" namespace="/">
      <!-- name以后和namespace合并形成访问的url  class就是具体的action的类 -->
        <action name="hello" class="demo01.Hello">
          <!-- 看返回的值是什么然后转发到对应的页面 -->
            <result name="success">/index.jsp</result>
        </action>
    </package>
</struts>
```

#### 1.package
就相当于java中的包，为了区分不同的action。
* name和功能无关，name值不能重复
* extends 他的值是固定的，是继承了struts，只有这样类才有action的特性
* namespace 和name构成访问路径  不写默认就是斜杠
#### 2.action
* name就是访问路径
* class action的class路径
* method 这个就是要执行的方法
#### 3.result
* name  回去返回值
* type  就是跳转还是转发
#### 4.配置常量
```xml
<constant name="" value=""/>
<!-- 例如 -->
<constant name="struts.i18n.encoding" value="UTF-8"/>
```
### 3.配置web.xml过滤器
我们的web服务器并不知道我们访问的是一个action而不是一个Servlet，所以我们要使用一个Struts2的过滤器来处率action的请求。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```
### 4.struts2基本原理
#### 1.过滤器加载配置文件
在web服务器启动的时候，过滤器就会创建，然后在init方法就会加载struts2的核心配置文件，以及它自带的jar包中的配置文件
#### 2.访问action执行方法
在访问的时候都被过滤器拦截到，然后转发到对应的类，让这些类的对应方法执行

### 5.配置文件的拆分
可以拆分配置文件，但是子配置文件还是需要dtd，struts标签，然后主配置文件也是如此。然后在主配置文件中的struts标签中写include标签。  

主配置文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
        "http://struts.apache.org/dtds/struts-2.3.dtd">
<struts>
    <constant name="struts.i18n.encoding" value="UTF-8"/>
    <include file="struts.xml"/>
</struts>
```
子配置文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
        "http://struts.apache.org/dtds/struts-2.3.dtd">

<struts>
    <constant name="struts.i18n.encoding" value="UTF-8"/>
    <package name="struts" extends="struts-default" namespace="/">
        <action name="hello" class="demo01.Hello">
            <result name="success">/index.jsp</result>
        </action>
    </package>
</struts>
```
