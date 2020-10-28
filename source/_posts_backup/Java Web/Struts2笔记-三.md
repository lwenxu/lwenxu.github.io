---
title: Struts2笔记(三)
date: 2017-08-23 10:39:26
tags: struts2
categories:
  -JAVA
---
拦截器，struts2方庄了很多功能，这些功能都是使用拦截器来实现的。一般的拦截器只是执行默认的拦截器，就是在struts-core这个包里面，然后李 main 有一个struts-default.xml这个文件李爱你就定义了很多默认的拦截器，interceptor-strck这个标签里面就是默认的拦截器。例如里面有ModelDriven。
### 1.拦截器执行的时机
拦截器执行的时机就是action创建完成之后在action调用的方法之前调用。
### 2.底层原理
#### 1.aop思想
aop就是面向切面编程。如果我们需要扩展一个类的功能，一般来说我们会写一个父类，<!--more-->让子类继承父类这样子类的功能就能获得拓展。这就是纵向编程，而所谓的横向的切面编程就在这个类的某个方法之前我们执行某些操作，或者在某个方法之后执行某些操作。蕾丝与thinkphp中的__before()或者__after()。
#### 2.责任链模式
责任链就类似于过滤链，过滤链就需要首先方形才能够继续进行。也就是如果我们需要进行一系列的操作的话，首先操作一然后必须放行才能进行下一个操作。  
再多拦截器的时候就需要责任链模式。
#### 3.原理概述
首先需要创建 action 的代理对象，所谓代理对象就不是真正的对象，而是一个具有被代理的对象的全部功能的对象，然后在代理对象执行 action 的方法之前，做了一个 aop 的操作也就是执行各种默认的拦截器，在每一个拦截器执行完毕以后需要在执行放行操作执行下一个拦截器，最后才执行那个被调用的方法。
### 3.过滤器与拦截器
过滤器是服务器启动的时候创建，然后能过滤所有的内容，只要有路径就可以拦截。  
但是拦截器虽然叫做拦截器，他只能拦截action
### 4.自定义拦截器
#### 1.写逻辑的类
首先需要继承 AbstractInterceptor 类，然后里面有三个方法，分别就是 init 初始化的操作，destroy 销毁的操作，interceptor 拦截的操作。
实际在开发的时候尽量继承的 MethodFilterInterceptor 类，重写里面的拦截方法。最后注册拦截器，配置拦截器和 action 的关系。
```java
public class MyInterceptor extends MethodFilterInterceptor{
    @Override
    protected String doIntercept(ActionInvocation actionInvocation) throws Exception {
        //逻辑

        //if 成功 ->放行操作
        actionInvocation.invoke();
        //if fail -> 返回一个字符串，就回到 result 中找这个结果就和 action 一样
        return "fail";
    }
}
```
在 action 中的所在的包配置拦截器的声明。
```xml
<interceptors>
            <interceptor name="MyInterecpot" class="demo2.MyInterceptor"/>
</interceptors>
```
再具体的 action 标签里面使用配置的拦截器
```xml
<action name="hello" class="demo01.Hello">
            <interceptor-ref name="MyInterecpot"/>
            <result name="success">/index.jsp</result>
</action>
```
把默认拦截器写上，不然默认拦截器会失效。
```xml
<interceptor-ref name="defaultStack"/>
```
不拦截某个方法
```xml
<interceptor-ref name="MyInterecpot">
                <param name="excludeMethods">login,add</param>
</interceptor-ref>
```      
