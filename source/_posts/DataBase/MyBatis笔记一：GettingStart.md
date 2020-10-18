---
title: MyBatis笔记一：GettingStart
date: 2018-05-30 18:56:49
categories: MyBatis
tags:
- MyBatis
- 数据库
- Java
---

# MyBatis笔记一：GettingStart

## 1.MyBatis优点

![image](https://user-images.githubusercontent.com/22151420/40716284-07d07a7e-643b-11e8-8a15-3e8448bc0e5f.png)

我们的工具和各种框架的作用就是为了我们操作数据库简洁，对于一些数据库的工具能帮我们少写一些处理异常等等的代码，但是他们并不是自动化的，很多的操作还是需要我们自己进行，所以我们的框架就帮我们把中间黑色的部分封装起来了，减少我们的负担，但是SQL也是重中之重，我们需要把这些东西自己来控制就有 MyBatis 这个半自动框架，以及我们需要学习更多的关于 HQL 的内容。<!--more-->

相对于Hibernate 他的优点就是可以进行SQL 的定制化，能让我们的SQL更加优化，虽然 Hibernate 也可以这么做但是有一点就是我们需要在原来的框架的基础上学习更多的 HQL 相关的东西，加重学习负担。

## 2.Getting Start

### 1.引入依赖

首先我们需要建立一个新的工程，第一步就是进入 MyBatis 的依赖，当然也少不了数据库的依赖，这里我们为了方便实体类的编写我们还加上了 Lombok ，以及log4j ：

```xml
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.5</version>
    </dependency>
    
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.35</version>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.16.20</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
   	 	<version>1.2.17</version>
    </dependency>
</dependencies>
```

### 2.编写配置文件

需要在类路径下编写配置文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///spring_data"/>
                <property name="username" value="root"/>
                <property name="password" value=""/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```



### 2.log4j配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">

<log4j:configuration>

    <appender name="myConsole" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern"
                   value="[%d{dd HH:mm:ss,SSS\} %-5p] [%t] %c{2\} - %m%n" />
        </layout>
        <!--过滤器设置输出的级别-->
        <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <param name="levelMin" value="debug" />
            <param name="levelMax" value="warn" />
            <param name="AcceptOnMatch" value="true" />
        </filter>
    </appender>

    <appender name="myFile" class="org.apache.log4j.RollingFileAppender">
        <param name="File" value="D:/output.log" /><!-- 设置日志输出文件名 -->
        <!-- 设置是否在重新启动服务时，在原有日志的基础添加新日志 -->
        <param name="Append" value="true" />
        <param name="MaxBackupIndex" value="10" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%p (%c:%L)- %m%n" />
        </layout>
    </appender>

    <appender name="activexAppender" class="org.apache.log4j.DailyRollingFileAppender">
        <param name="File" value="E:/activex.log" />
        <param name="DatePattern" value="'.'yyyy-MM-dd'.log'" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern"
                   value="[%d{MMdd HH:mm:ss SSS\} %-5p] [%t] %c{3\} - %m%n" />
        </layout>
    </appender>

    <!-- 指定logger的设置，additivity指示是否遵循缺省的继承机制-->
    <logger name="com.runway.bssp.activeXdemo" additivity="false">
        <appender-ref ref="activexAppender" />
    </logger>

    <!-- 根logger的设置-->
    <root>
        <priority value ="debug"/>
        <appender-ref ref="myConsole"/>
        <appender-ref ref="myFile"/>
    </root>
</log4j:configuration>
```

### 3.编写实体类

```java
package lwen.entries;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.experimental.Accessors;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Accessors(chain = true)
public class Employee {
    private int id;
    private String name;
    private int age;
}
```

### 4.编写测试类

```java
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;

public class GettingStart {
    @Test
    public void sessionFactoryTest() throws IOException {
        String resource = "mybatis.xml";
        InputStream stream = Resources.getResourceAsStream(resource);
        SqlSession sqlSession = null;
       try {
           //获取sql工厂
           SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(stream);
           //获取sql session
           sqlSession = sessionFactory.openSession();

           // 1.sql标识 2.绑定的参数
           Object employee = sqlSession.selectOne("selectOneEmployee", 3);
           System.out.println(employee);
       }finally {
           if (sqlSession != null) {
               sqlSession.close();
           }
       }

    }
}
```

### 5.编写mapper文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC " -//mybatis.org//DTD Mapper 3.e//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.mybatis.example.BlogMapper">
    <select id="selectOneEmployee" resultType="lwen.entries.Employee">
      select * from employee where id=#{id}
    </select>
</mapper>
```

注意这里其实我们的mapper和测试类里的参数必须是需要匹配的才行。比如我们的 id 就是我们的测试类中的 select 方法需要传的参数，然后我们的 resultType 就是一个实体类的具体路径。

### 6.总结基本套路

1. 根据xml配置文件（全局配置文件）创建一个Sq1SessionFactory对象，xml是用来配置数据源一些运行环境信息

2. sql映射文件：配置了每一个sql，以及sql的封装规则等。

3. 将sq1映射文件注册在全局配置文件中

4. 写Java代码：

   1）、根据全局置文件得到Sq1SessionFactory；
   2）、使用sqlSession工厂，获取到sqlSession对象使用他来执行增改查，一个sqlSession就是代表和数据库的一次会话，用完关闭
   3）、使用sql的唯一标志来告诉MyBatis执行哪个sql，语句都是保存在Sql映射文件中的。

**上面我们使用的方式是以前比较老的方式，现在不再使用那种方式了，我们现在都是面向的接口编程的，所以我们使用Mapper 类来完成数据库访问。**

## 3.面向接口编程

### 1.编写接口Mapper

创建一个mapper接口，然后里面写方法即可：

```java
package lwen.dao;

import lwen.entries.Employee;

public interface EmployeeMapper {
    public Employee getEmployeeById(Integer id);
}
```

可以看到就是这么简单，但是我们一开始是通过方法的参数和我们的 xml 的查询语句链接起来的，这里我们又怎么做这个映射关系呢？其实就是需要在我们的 xml 中配置我们的 这个Mapper 的路径，以及我们的方法作为查询语句的 id 属性，还有返回值类型，等等。接下来就来看这些配置。

### 2.配置Mapper.xml

接下来就是配置我们的 mapper 的xml了，让他和我们的接口挂钩。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC " -//mybatis.org//DTD Mapper 3.e//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="lwen.dao.EmployeeMapper">
    <select id="getEmployeeById" resultType="lwen.entries.Employee">
      select * from employee where id=#{id}
    </select>
</mapper>
```

**注意事项：**

1. 第一点就是我们的 namespace 以前事随便写，但是这个地方就是需要我们和Mapper 接口绑定，也就是我们的接口的全类名。
2. 然后就是我们的id属性，就是我们的Mapper的方法名。
3. 还有一点疑问就是我们的参数，我们都没传，其实这里他做了默认绑定，参数名和这里的sql模板参数名一致就行。

### 3.测试类

```java
@Test
public void InterfaceMapperTest() throws IOException {
    String resource = "mybatis.xml";
    InputStream stream = Resources.getResourceAsStream(resource);
    SqlSession sqlSession = null;
   try {
       //获取sql工厂
       SqlSessionFactory sessionFactory = getSessionFactory("mybatis.xml");
       //获取sql session
       sqlSession = sessionFactory.openSession();

       // 获取 mapper 接口的实例对象
       EmployeeMapper employeeMapper = sqlSession.getMapper(EmployeeMapper.class);
       //直接调用的 mapper 里的方法
       System.out.println(employeeMapper.getEmployeeById(3));

   }finally {
       if (sqlSession != null) {
           sqlSession.close();
       }
   }

}
```

### 4.一个神奇的点

这里我们会发现我们根本没有写接口的实现类，我们怎么获取到我们的 Mapper 的对象，并且还可以调用他的方法呢？实际上我们去打印一下我们使用 `sqlSession.getMapper(EmployeeMapper.class)` 这句话获取到我们的对象，并且能执行相应的操作呢？

我们打印出来这个对象我们就会发现，获取到的对象实际上是一个代理对象，那么也就是说我们 Mybatis 底层采用的代理方式做的具体的功能。

其实我们通过SqlSession.getMapper(XXXMapper.class) 方法，**MyBatis** 会根据相应的接口声明的方法信息，通过动态代理机制生成一个**Mapper** 实例，我们使用**Mapper** 接口的某一个方法时，**MyBatis** 会根据这个方法的方法名和参数类型，确定**Statement Id**，底层还是通过`SqlSession.select("statementId",parameterObject);`或者`SqlSession.update("statementId",parameterObject);` 等等来实现对数据库的操作，（至于这里的动态机制是怎样实现的，我想专门用一篇文章来讨论） 

**MyBatis** 引用 **Mapper** 接口这种调用方式，纯粹是为了满足面向接口编程的需要。（其实还有一个原因是在于，面向接口的编程，使得用户在接口上可以使用注解来配置SQL语句，这样就可以脱离XML配置文件，实现“0配置”）。 因为面向接口编程其实有非常多的好处。因为现在我们的 dao 代码就是我们的 mapper 接口了，但是如果有一天我们希望不采用 MyBatis 来做数据访问了，我们在底层就可以直接写对应的实现类使用 Hibernate 等等其他框架来顶替，也是一件很容易的事。

## 4.总结

1. SqlSession代表和数据库的一次会话：用完必须关闭，这也是一个资源，所以我们用完了必须进行关闭操作，避免数据库不必要的问题。
2. SqlSession和connection一样都是非线程安全，每次使用都应该去获取新的对象，不可以当作一个成员变量 ，不然自然会导致紊乱，也就是我们需要把他放在一个局部方法中，用完关闭。其实说清楚点他的底层就是 connection。
3.  mapper接口没有实现类，但是mybatis会为这个接口生成一个代理对象。（将接口和xml进行绑定）
4. 两个重要的配置文件：
   1. mybatis的全局配置文件：包含数据库连接池信息，事务管理器信息等。。。系统运行环境信息
   2. sql映射文件：保存了每一个sq1语句的映射信息：将sql抽取出来。