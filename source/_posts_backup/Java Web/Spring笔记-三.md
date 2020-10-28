---
title: Spring笔记(三)
date: 2017-08-23 17:37:03
tags: Spring
categories:
  -JAVA
---
### 1.使用注解实现 aop
* 配置 xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd ">
        <!--开启 aop 操作-->
        <aop:aspectj-autoproxy/>
        <bean id="user" class="aop1.User"/>
        <bean id="improve" class="aop1.Improve"/>

</beans>
```<!--more-->
* 在增强类中注解
```java
package aop1;

import org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Aspect
public class Improve {
    @Before(value="execution(* aop1.User.say(..))")
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

### 2.JdbcTemplate
#### 1.增
```java
//配置 dataSource
        DriverManagerDataSource dataSource=new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///JAVA");
        dataSource.setUsername("root");
        dataSource.setPassword("12345678");
        //设置模板
        JdbcTemplate jdbcTemplate=new JdbcTemplate(dataSource);
        //操作
        String sql="insert into user values(?,?)";
        jdbcTemplate.update(sql,1,"lwen");
```
#### 2.删
还是 update 操作只是 sql 不一样
#### 3.改
同增加
#### 4.查
类似于 QueryRunner 也是使用接口的实例，不过这里没有准备好的实现类，实现类需要我们自己书写。
##### 1.查询单值
```java
@Test
    void fun2(){
        //配置 dataSource
        DriverManagerDataSource dataSource=new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///JAVA");
        dataSource.setUsername("root");
        dataSource.setPassword("12345678");
        //设置模板
        JdbcTemplate jdbcTemplate=new JdbcTemplate(dataSource);
        //操作
        String sql="select count(*) from user";
        Object object=jdbcTemplate.queryForObject(sql,Integer.class);
        System.out.println(object);
    }


    class MyRowMapper implements RowMapper{

        public Object mapRow(ResultSet resultSet, int i) throws SQLException {
            User user=new User();
            user.setId(resultSet.getInt("id"));
            user.setName(resultSet.getString("name"));
            return user;
        }
    }

    @Test
    void fun3(){
        //配置 dataSource
        DriverManagerDataSource dataSource=new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///JAVA");
        dataSource.setUsername("root");
        dataSource.setPassword("12345678");
        //设置模板
        JdbcTemplate jdbcTemplate=new JdbcTemplate(dataSource);
        //操作
        String sql="select * from user where name=?";
        Object object=jdbcTemplate.queryForObject(sql, new MyRowMapper(),"lwen");
        System.out.println(object);
    }
```
##### 2.多值查询
```java
class MyRowMapper1 implements RowMapper{

        public Object mapRow(ResultSet resultSet, int i) throws SQLException {
            User user=new User();
            user.setId(resultSet.getInt("id"));
            user.setName(resultSet.getString("name"));
            return user;
        }
    }
    @Test
    void fun4(){
        //配置 dataSource
        DriverManagerDataSource dataSource=new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///JAVA");
        dataSource.setUsername("root");
        dataSource.setPassword("12345678");
        //设置模板
        JdbcTemplate jdbcTemplate=new JdbcTemplate(dataSource);
        //操作
        String sql="select * from user";
        List<User> users=jdbcTemplate.query(sql,new MyRowMapper1());
        System.out.println(users);
    }
```

### 2.事务管理

#### 1.xml方式配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
                            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd ">
    <!--jdbcTemplate-->
    <bean id="jdbc" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--配置事务管理器-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入 dataSource-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--配置事务的增强-->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <tx:method name="add"/>
        </tx:attributes>
    </tx:advice>
    <!--配置切面-->
    <aop:config>
        <!--切入点-->
        <aop:pointcut id="point1" expression="execution(* com.lwen.Service.add(..))"/>
        <!--切面-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="point1"/>
    </aop:config>
</beans>
```

#### 2.注解方式
* 开启注解
```xml
<!-- 开启注解 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```
* 在事务类上面写注解
@Transactional
