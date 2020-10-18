---
title: Spring笔记(二)
date: 2017-08-23 16:05:53
tags: Spring
categories:
  -JAVA
---
### 1.注解ioc
#### 1.开启注解扫描
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--开启注解扫描-->
    <context:component-scan base-package="domain1"/>
</beans>
```
<!--more-->
#### 2.在类上面做注解：
```java
@Component(value = "user")
@Scope(value="singleton")
```
注解有四个，但是却别都不大：
* Component
* Service
* Controller
* Repository

#### 3.注入属性
注解的时候 set 方法也无需生成，直接注解就能搞定。  
直接在属性上面写@Autowired或者@Resource(name = "")  
第一种方式我们不用去指定对象的 id 值，而第二个需要些 id 值。主要用的就是第二个，因为他更加的严谨。

### 2.AOP
#### 1.基本介绍
aop 也叫做面向切面编程，或者面向方面编程。他的主要方式就是横向抽取，与以前的继承方式的纵向抽取不同。
这里就主要说说横向抽取，aop 主要还是用了动态代理的方式来拓展对象的功能。而动态代理分为两种：
* jdk 的动态代理
这种动态代理就是一个接口，然后我们需要使用 aop 来增强这个接口的实现类的对象的功能，那么这个动态代理方式就是新建一个类，然后这个类实现那个接口，也就是创建一个与被增强的类的平级的一个类，所以称之为横向抽取。之后只需要再次类中进行方法的增强即可，但是这个地方就是用了动态代理的方式而不是切切实实的写一个类，我们代理原来的需要被增强的类，然后在他的基础上做一些拓展。
* gclib 动态代理方式
这种动态代理是针对的没有接口的代理。主要就是通过集成需要被增强的类，然后进行动态代理。

#### 2.相关定义
* 连接点：类中的哪些类可以被增强的方法就称之为连接点。
* 切入点：我们实际增强了的方法。
* 增强：我们增强的功能。分为前置增强（方法之前），后置增强（方法之后），异常增强（抛出异常执行），最终增强（后置之后），环绕增强（之前之后都执行）
* 切面：把增强应用到方法上的过程。
* 目标对象：代理的目标对象

#### 3.xml 配置方式
增强类：
```java
package aop;

import org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint;

public class Improve {
    public void before_say(){
        System.out.println("before");
    }

    public void after_say(){
        System.out.println("after");
    }

    public void around_say(MethodInvocationProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("before");
        joinPoint.proceed();
        System.out.println("after");
    }
}
```
被增强类
```java
package aop;

public class User {
    public void say(){
        System.out.println("hello");
    }
}
```
配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd ">

    <bean id="improve" class="aop.Improve"/>
    <aop:config>
        <!--配置切入点  方法-->
        <aop:pointcut id="point1" expression="execution(* aop.User.say(..))"/>
        <!--配置切面 应用增强-->
        <aop:aspect ref="improve">
            <aop:before method="before_say" pointcut-ref="point1"/>
        </aop:aspect>
    </aop:config>


    <bean id="user" class="aop.User"/>
</beans>
```
### 2.log4j
导包：
* log4j
* common-loging
配置文件：
* src/log4j.properties
