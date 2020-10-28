---
title: SpringBoot 笔记(十二):数据访问
date: 2018-05-24 10:01:28
tags: SpringBoot
categories:
 - SpringBoot
---
# SpringBoot 笔记(十二):数据访问

## 1、JDBC

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
```



```yaml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.15.22:3306/jdbc
    driver-class-name: com.mysql.jdbc.Driver
```

### 1.默认情况下：

​	1. 用org.apache.tomcat.jdbc.pool.DataSource作为数据源；

​	2. 数据源的相关配置都在DataSourceProperties里面；

### 2.自动配置原理：

org.springframework.boot.autoconfigure.jdbc包下就是进行自动配置的包：

#### 1、数据库连接池

参考 DataSourceConfiguration，根据配置创建数据源，默认使用Tomcat连接池，可以使用spring.datasource.type指定自定义的数据源类型

#### 2、创建数据源

SpringBoot默认可以支持：`org.apache.tomcat.jdbc.pool.DataSource、HikariDataSource、BasicDataSource`

```java
/**
 * Generic DataSource configuration.
 */
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type")
static class Generic {

   @Bean
   public DataSource dataSource(DataSourceProperties properties) {
       //使用DataSourceBuilder创建数据源，利用反射创建响应type的数据源，并且绑定相关属性
      return properties.initializeDataSourceBuilder().build();
   }

}
```

#### 3、DataSourceInitializer：ApplicationListener

一个监听器，主要用于sql脚本的运行

​	作用：

​		1）、runSchemaScripts();运行建表语句；

​		2）、runDataScripts();运行插入数据的sql语句；

默认只需要将文件命名为： `schema-*.sql 或者 data-*.sql` 默认规则：`schema.sql，schema-all.sql`

当然我们也可以在配置文件中指定，然后就加载指定的sql 脚本，这里是一个 list 也就是脚本可以指定多个。

```properties
 	schema:
      - classpath:department.sql
```

## 2、整合Druid数据源

### 1.配置 yml

```properties
spring.datasource.initialSize:5
spring.datasource.minIdle:5
spring.datasource.maxActive:20
spring.datasource.maxlait:60800
spring.datasource.timeBetweenEvictionRunsMillis:60000
spring.datasource.minEvictableIdleTimeMillis:300000
spring.datasource.validationQuery:SELECT 1 FROM DUAL
spring.datasource.testWhileIdle:true
spring.datasource.testOnBorrow:false
spring.datasource.testOnReturn:false
spring.datasource.poolPreparedStatements:true
#配置监控统计拦截的filters，去掉后监控界面sql无法统计，‘walL“用于防火墙
spring.datasource.filters:stat,wall,log4j
spring.datasource.maxPoolPreparedStatementPerConnectionSize:20
spring.datasource.useGTobalDataSourceStat:true
spring.datasource.connectionProperties:druid.stat.mergeSql=true；druid.stat.slowsqlMillis=5e0
```

上面的配置文件中的数据实际上我们没用上，这里我们需要创建一个 DruidConfig 然后和当前的配置文件绑定。

### 2.JavaConfig

```java

@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
       return  new DruidDataSource();
    }
    //配置Druid的监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();

        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        initParams.put("allow","");//默认就是允许所有访问
        initParams.put("deny","192.168.15.21");

        bean.setInitParameters(initParams);
        return bean;
    }


    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");

        bean.setInitParameters(initParams);

        bean.setUrlPatterns(Arrays.asList("/*"));

        return  bean;
    }
}

```

## 3、整合MyBatis

```xml
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.1</version>
		</dependency>
```

![](../../../../%E8%B5%84%E6%96%99/%E6%BA%90%E7%A0%81%E3%80%81%E8%B5%84%E6%96%99%E3%80%81%E8%AF%BE%E4%BB%B6/%E6%96%87%E6%A1%A3/Spring%20Boot%20%E7%AC%94%E8%AE%B0/images/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180305194443.png)

### 1、注解版

```java
//指定这是一个操作数据库的mapper
@Mapper
public interface DepartmentMapper {

    @Select("select * from department where id=#{id}")
    public Department getDeptById(Integer id);

    @Delete("delete from department where id=#{id}")
    public int deleteDeptById(Integer id);

    @Options(useGeneratedKeys = true,keyProperty = "id")  //这是为了获取自动增长的id
    @Insert("insert into department(departmentName) values(#{departmentName})")
    public int insertDept(Department department);

    @Update("update department set departmentName=#{departmentName} where id=#{id}")
    public int updateDept(Department department);
}
```

### 2.自定义配置

自定义MyBatis的配置规则，给容器中添加一个ConfigurationCustomizer；

```java
@org.springframework.context.annotation.Configuration
public class MyBatisConfig {

    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return new ConfigurationCustomizer(){
            @Override
            public void customize(Configuration configuration) {
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```

使用MapperScan批量扫描所有的Mapper接口，这样的话我们就不用在Mapper接口上写 @Mapper 注解了。就属于批量的注解了这些注解。

```java
@MapperScan(value = "com.springboot.mapper")
@SpringBootApplication
public class SpringBootMybatisApplication {
	public static void main(String[] args) {
		SpringApplication.run(SpringBootMybatisApplication.class, args);
	}
}
```

### 3、配置文件版

```yaml
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml 指定全局配置文件的位置
  mapper-locations: classpath:mybatis/mapper/*.xml  指定sql映射文件的位置
```

更多使用参照 http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

## 4、整合SpringData JPA

### 1）、SpringData简介

![](../../../../%E8%B5%84%E6%96%99/%E6%BA%90%E7%A0%81%E3%80%81%E8%B5%84%E6%96%99%E3%80%81%E8%AF%BE%E4%BB%B6/%E6%96%87%E6%A1%A3/Spring%20Boot%20%E7%AC%94%E8%AE%B0/images/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180306105412.png)

### 2）、整合SpringData JPA

JPA:ORM（Object Relational Mapping）；

1）、编写一个实体类（bean）和数据表进行映射，并且配置好映射关系；

```java
//使用JPA注解配置映射关系
@Entity //告诉JPA这是一个实体类（和数据表映射的类）
@Table(name = "tbl_user") //@Table来指定和哪个数据表对应;如果省略默认表名就是user；
public class User {

    @Id //这是一个主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增主键
    private Integer id;

    @Column(name = "last_name",length = 50) //这是和数据表对应的一个列
    private String lastName;
    @Column //省略默认列名就是属性名
    private String email;
```

2）、编写一个Dao接口来操作实体类对应的数据表（Repository）

```java
//继承JpaRepository来完成对数据库的操作
public interface UserRepository extends JpaRepository<User,Integer> {
}

```

3）、基本的配置JpaProperties

```yaml
spring:  
 jpa:
    hibernate:
#     更新或者创建数据表结构
      ddl-auto: update
#    控制台显示SQL
    show-sql: true
```