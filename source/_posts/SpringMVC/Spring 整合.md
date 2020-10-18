---
title: SpringMVC 整合
date: 2018-04-21 20:01:28
tags: SpringMVC
categories:
 - SpringMVC
---

今天一开始直接用了 Idea 来创建一个 SpringMVC+Spring+Mybatis+Thymeleaf 的项目，一开始还是挺顺利的，除了在 Thymeleaf 那个地方卡了一下，后面项目还是顺利跑起来了。

接着想用 Maven 搭建，因为一开始用 Idea 生成的项目使用的手动导入 jar 这样非常费力，为了一劳永逸和简单就采用了 maven 来构建项目。最后发现自己陷入了一个大坑，好久没有跳上来。

接着我就把用 Maven 搭建 SpringMVC + Spring + Mybatis + Thymeleaf 项目过程写下来，免得日后再采坑，以后项目直接拷贝就可以了不用再配置了！



## 1. 创建 Maven 项目

[![lo8Gl.md.png](https://s1.ax2x.com/2018/04/21/lo8Gl.md.png)](https://simimg.com/i/lo8Gl) 



## 2.添加 web 模块

![loSLJ.png](https://s1.ax2x.com/2018/04/21/loSLJ.png)

在这里面勾选上 web 模块，接着我们会看到我们的项目中多了一个 web 目录也就是最重要的 WEB-INF 等等。

## 3. web 模块移动

![CMJk0U.png](https://s1.ax1x.com/2018/04/21/CMJk0U.png)

我们把 web 模块移动到这里，方便观看。以及在 maven 项目之内，接着我们需要改一下项目配置，因为我们移动了这个模块。如果移动的时候更新了依赖就无需这步操作。

[![CMJZtJ.md.png](https://s1.ax1x.com/2018/04/21/CMJZtJ.md.png)](https://imgchr.com/i/CMJZtJ)

## 4. 配置文件

![CMJeh9.png](https://s1.ax1x.com/2018/04/21/CMJeh9.png)

创建配置文件如上。

1. spring 的配置文件 applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--配置包扫描-->
    <context:component-scan base-package="com.lwen"/>
    <!--配置静态资源访问-->
    <mvc:default-servlet-handler/>
    <!--配置注解-->
    <mvc:annotation-driven/>
    <!--配置 thymeleaf 视图解析器  类似于 jsp 解析器-->
    <bean id="templateResolver" class="org.thymeleaf.spring4.templateresolver.SpringResourceTemplateResolver">
        <property name="prefix" value="/WEB-INF/templates/" />
        <property name="suffix" value=".html" />
        <property name="templateMode" value="HTML5" />
    </bean>
    <bean id="templateEngine" class="org.thymeleaf.spring4.SpringTemplateEngine">
        <property name="templateResolver" ref="templateResolver" />
    </bean>
    <bean class="org.thymeleaf.spring4.view.ThymeleafViewResolver">
        <property name="templateEngine" ref="templateEngine" />
    </bean>

    <!--配置外部配置文件-->
    <context:property-placeholder location="classpath:properties.properties"/>
    <!--配置druid数据库连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <!-- 数据库基本信息配置 -->
        <property name="url" value="${url}" />
        <property name="username" value="${username}" />
        <property name="password" value="${password}" />
        <property name="driverClassName" value="${driverClassName}" />
        <!-- 最大并发连接数 -->
        <property name="maxActive" value="${maxActive}" />
        <!-- 初始化连接数量 -->
        <property name="initialSize" value="${initialSize}" />
    </bean>
    <!--配置 spring 事务管理器-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--配置注解事务-->
    <tx:annotation-driven/>
    <!--配置手动路由-->
    <mvc:view-controller path="/" view-name="index"/>


</beans>
```

1. springMVC 配置文件 dispatcher-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">



</beans>
```

1. 数据库配置文件 properties.properties

   ```properties
   url=jdbc:mysql://localhost:3306/dbspring
   username=root
   password=12345678
   driverClassName=com.mysql.jdbc.Driver
   maxActive=500
   initialSize=5
   ```

## 5. pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lwen</groupId>
    <artifactId>springmvc</artifactId>
    <version>1.0-SNAPSHOT</version>

    <profiles>
        <profile>
            <id>jdk‐1.8</id>
            <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>1.8</jdk>
            </activation>
            <properties>
                <maven.compiler.source>1.8</maven.compiler.source>
                <maven.compiler.target>1.8</maven.compiler.target>
                <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
            </properties>
        </profile>
    </profiles>

    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.1</version>
        </dependency>

        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf</artifactId>
            <version>3.0.9.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring4</artifactId>
            <version>3.0.9.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>aopalliance</groupId>
            <artifactId>aopalliance</artifactId>
            <version>1.0</version>
        </dependency>

        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.9</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>4.3.14.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>4.3.13.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-instrument</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-instrument-tomcat</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jms</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-messaging</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-oxm</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc-portlet</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-websocket</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>


    </dependencies>


</project>
```



## 6. 配置 jar 依赖

[![CMJ10O.md.png](https://s1.ax1x.com/2018/04/21/CMJ10O.md.png)](https://imgchr.com/i/CMJ10O)

这步很重要，否则会一直报错，说找不到 class。