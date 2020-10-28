---
title: Spring笔记(一)
date: 2017-08-23 15:12:40
tags: Spring
categories:
  -JAVA
---
### 1.基本介绍
spring 是一个一站式框架，也就是有了它 web 层，service 层还有 dao 层都能直接搞定而不需使用其他的框架。
这三层分别就是：
* SpringMVC
* ioc
* JdbcTemplate
<!--more-->
### 2.ioc基本原理
ioc 原理就是使用配置文件解析到需要创建的类的 class 然后使用工厂类的静态方法获取这个类的对象。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="domain.User"/>

</beans>
```
```java
@Test
    void fun1(){
        ApplicationContext context= new ClassPathXmlApplicationContext("user.xml");
        User user= (User) context.getBean("user");
        user.say();
    }
```
### 3.spring 差UN感觉爱你对象的三种方式
#### 1.ioc无参构造
也就是上面的那个方法，首先配置 bean 然后读取配置文件，根据 id 获取类的对象，这个实例化的时候都是使用该类的默认构造函数。默认构造函数是无参的，当我们使用有参的构造函数覆盖了以后就会报错。这个方法也是最常用的一种方法。
#### 2.静态工厂
首先创建一个静态工厂的类，这个类里面需要有静态方法，静态方法返回类的实例，最后配置 xml 文件，也和上面一样，但是这里的 class 值写得是静态工厂的类，还有一个 factory-method 属性，就写那个静态方法。
#### 3.实例工厂
实例工厂和上面的静态工厂不同的方式就是这个工厂需要实例化，然后实例化就交给 ioc 来解决其他的和静态工厂一样。

### 4.bean 标签的常用属性
#### 1.id
id 值就类似于对像的名字
#### 2.class
需要创建的类的全路径
#### 3.name
和 id 属性一样，获取对象也是 getBean 方法，他和 id 的不同的在于他可以写一些特殊符号。
#### 4.scope
* singleton 单例对象，默认值
* prototype 多例
* request  创建的对象都放在了request 域
* session    创建对象放在 session 域

### 5.ioc 属性注入
#### 1.构造函数注入
```xml
<bean id="user" class="domain.User">
        <constructor-arg name="id" value="1"/>
        <constructor-arg name="name" value="lwen"/>
</bean>
```
#### 2.set 方法
```xml
<bean id="user1" class="domain.User">
        <property name="id" value="1"/>
        <property name="name" value="lwen"/>
</bean>
```

#### 3.注入对象
属于 set 方法的一种 ，但是他和注入普通的字符串还是不一样的。
```xml
<bean id="user1" class="domain.User">
        <property name="userDao" ref="Dao 的 id 值"/>
</bean>
```

#### 4.复杂类型的注入
还是使用 set 方法
* 数组类型
* list
* map
* properties
```xml
<bean id="user2" class="domain.User">
        <!--数组-->
        <property name="array">
            <list>
                <value>1</value>
                <value>1</value>
                <value>1</value>
            </list>
        </property>
        <!--list-->
        <property name="list">
            <list>
                <value>hello</value>
                <value>hello</value>
                <value>hello</value>
            </list>
        </property>
        <!--map-->
        <property name="map">
            <map>
                <entry key="a" value="a"/>
                <entry key="b" value="b"/>
            </map>
        </property>
        <!--properties-->
        <property name="properties">
            <props>
                <prop key="url">jdbc:mysql:///java</prop>
            </props>
        </property>
</bean>
```
### 5.配置文件自动加载    
在服务器创建的时候会创建一个 ServletContext 对象，这个对象有一个监听器，监听他什么时候创建。
我们可以在这个对象创建的时候把配置文件给加载，再把加载的文件对象放在 ServletContext 域中。
这个监听器就叫做 ServletContextListener 。
这个东西目前 spring 已经封装好了，我们只需要进行 xml 的配置即可。
<!-- 配置配置文件路径 -->
<context-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath:aop.xml</param-value>
</context-param>
<!-- 配置监听器 -->
<listener>
       <listener-class>org..web.context.ContextLoaderListener</listener-class>
</listener>
