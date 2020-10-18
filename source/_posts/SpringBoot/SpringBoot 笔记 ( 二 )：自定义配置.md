---
title: SpringBoot笔记 ( 二 )：自定义配置
date: 2018-04-13 22:01:28
tags: SpringBoot
categories:
 - SpringBoot
---

# SpringBoot 笔记 ( 二 )：自定义配置

### 1. 配置文件
SpringBoot使用一个全局的配置文件，配置文件名是固定的：

- application.properties
- application.yml

&emsp;&emsp;  修改SpringBoot自动配置的默认值，因为在所有的自动配置类中他们都会去读取我们的配置文件，如果说有配置这些项目就按照我们配置的，没有则使用自动配置。
<!-- more -->
&emsp;&emsp;  支持两种格式，我们主要说说后面一种，前面比较简单就是采用的点的方式定义的。yml 其实也是一种标记语言，YAML 他的语法比较简洁，写起来没有 xml 那么臃肿。

### 2. YAML 语法

#### 1、基本语法

k:(空格)v：表示一对键值对（空格必须有）

以**空格**的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的同时属性和值也是大小写敏感；

#### 2. 字面量：普通的值（数字，字符串，布尔）

​	k: v：字面直接来写，字符串默认不用加上单引号或者双引号；

#### 3. 对象、Map（属性和值）（键值对）：

​	k: v：在下一行来写对象的属性和值的关系，注意缩进

​		对象还是k: v的方式

```yaml
friends:
		lastName: zhangsan
		age: 20
```

行内写法：

```yaml
friends: {lastName: zhangsan,age: 18}
```



#### 4. 数组、表（List、Set）：

用- 值表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets: [cat,dog,pig]
```

### 3. 配置文件注入

#### 1.@ConfigurationProperties

配置文件：

```yaml
person:
    lastName: hello
    age: 18
    boss: false
    birth: 2017/12/12
    maps: {k1: v1,k2: 12}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 12
```

对应的Bean：

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *      prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
 *
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;

```

@ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定  prefix = "person"：配置文件中哪个下面的所有属性进行一一映射 ，这样我们就明白了我们的 springboot 的配置文件的一些属性是绑定的我们的配置类的属性，那么我们的配置类就可以读取我们的配置文件里面的内容了！

#### 2.@Value

除了使用 `@ConfigurationProperties` 让我们的bean和配置文件绑定，我们还可以使用 @Value("配置文件属性") 注解去获取配置文件的内容。这个注解是支持spEL表达式的，这是他的优点。

#### 3.@PropertySource

这个注解的作用就是用来导入一些指定的配置文件里面的内容到我们当前的 JavaBean 中，与之关联。也就是方便我们的配置文件的拆分。 比如： `@PropertySource(value={"classpath:person.properties"})` 就是加载类路径下面的  person.properties 里面的内容。

#### 4.@ImportResource

 导入spring的xml配置文件然后让他生效。其实这个就是Spring的基本注解，导入配置文件配置类。这个东西要标注在主类上，而不是bean中。

注意，如果我们的配置文件和bean关联以后没有自动提示的话我们就需要导入pom ，在导入配置文件处理器以后，编写配置就有提示了。

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
```

一个小问题就是配置文件中配置的中文读出来会乱码，这主要是 idea 的 properties 文件默认采用的 utf-8 ，我们需要使用自动转 ascii 码。
![](https://s1.ax2x.com/2018/04/13/N7KhX.png)

### 4. 配置类
SpringBoot推荐给容器中添加组件的方式，推荐使用全注解的方式

1. 配置类 **@Configuration** 等价于 Spring 配置文件
2. 使用 **@Bean** 给容器中添加组件

```java
/**
 * @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
 *
 * 在配置文件中用<bean><bean/>标签添加组件
 *
 */
@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名,也可以自己指定。
    @Bean
    public HelloService helloService02(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```

### 5. 多环境配置文件

#### 1. 多Profile文件

我们在主配置文件编写的时候，文件名可以是`application-{profile}.properties/yml`默认使用`application.properties`的配置；

#### 2. 文档块方式

```yml

server:
  port: 8081
spring:
  profiles:
    active: prod

---
server:
  port: 8083
spring:
  profiles: dev


---

server:
  port: 8084
spring:
  profiles: prod  #指定属于哪个环境
```

#### 3. 激活指定profile

1. 在配置文件中指定  spring.profiles.active=dev
2. 命令行：`java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；`可以直接在测试的时候，配置传入命令行参数
3. 虚拟机参数；
			`-Dspring.profiles.active=dev`

#### 4. 指定外部配置文件
所有的配置都可以在命令行上进行指定
`java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc`

### 6. 自动配置原理

1. SpringBoot启动的时候加载主配置类，开启了自动配置功能 `@EnableAutoConfiguration`

2. **@EnableAutoConfiguration 作用：**

-  EnableAutoConfigurationImportSelector 给容器中导入一些组件
- selectImports()方法中的 `List<String> configurations = getCandidateConfigurations(annotationMetadata,      attributes);` 获取候选的配置

- `SpringFactoriesLoader.loadFactoryNames()`扫描所有jar包类路径下 `META-INF/spring.factories` 把扫描到的这些文件的内容包装成 `properties` 对象，从 `properties` 中获取到 `EnableAutoConfiguration.class` 类（类名）对应的值，然后把他们添加在容器中

**总之一句话将类路径下  META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中**
如下：
```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
```

每一个这样的  xxxAutoConfiguration 类都是容器中的一个组件，都加入到容器中，用他们来做自动配置


### 7.以HttpEncodingAutoConfiguration为例解释自动配置原理

```java
//表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
@Configuration   
//启动指定类的ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中   所有能在我们的配置文件中配置什么就是看我们的 Properties 中的 prefix和属性有什么。
@EnableConfigurationProperties(HttpEncodingProperties.class) 

//Spring底层@Conditional注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效,判断当前应用是否是web应用，如果是，当前配置类生效@ConditionalOnWebApplication 
//判断当前项目有没有这个类CharacterEncodingFilter,SpringMVC中进行乱码解决的过滤器；
@ConditionalOnClass(CharacterEncodingFilter.class)  

//判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的,也就是说即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)  

public class HttpEncodingAutoConfiguration {
  
  	//他已经和SpringBoot的配置文件映射了
  	private final HttpEncodingProperties properties;

   //只有一个有参构造器的情况下，参数的值就会从容器中拿
  	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
  
    @Bean   //给容器中添加一个组件，这个组件的某些值需要从properties中获取
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```

总结：

**1）、SpringBoot启动会加载大量的自动配置类**
**2）、我们看我们需要的功能有没有SpringBoot默认写好的自动配置类；**
**3）、我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件有，我们就不需要再来配置**
**了）**
**4）、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指**
**定这些属性的值**

  根据当前不同的条件判断，决定这个配置类是否生效，一但这个配置类生效，这个配置类就会给容器中添加各种组件，这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的。
&emsp;&emsp;  也就是如下两个类非常重要：

- xxxxAutoConfigurartion：自动配置类。
- xxxxProperties: 封装配置文件中相关属性。

### 8.@Conditional注解

这个是Spring中原生的，就是用一个Class这个Class中重写了 match 方法，来判断这个conditional是否为真。然后我们的SpringBoot 是自己写了很多新的 @Conditional 的新版，就拓展了原来的功能。

- @ConditionalOnJava系统的java版本是否符合要求
- @ConditionalOnBean容器中存在指定Bean；
- @ConditionalOnMissingBean容器中不存在指定Bean；
- @ConditionalOnExpression满足SpEL表达式指定
- @ConditionalOnClass系统中有指定的类
- @ConditionalOnMissingClass系统中没有指定的类
- @ConditionalonSingleCandidate容器中只有一个指定的Bean，或者这个Bean是首选Bean
- @ConditionalOnProperty系统中指定的属性是否有指定的值
- @ConditionalOnResource类路径下是否存在指定资源文件
- @ConditionalOnWebApplication 当前是web环境
- @ConditionalOnNotWebApplication 当前不是web环境
- @ConditionalOnjndiJNDI存在指定项

也就是自动配置类是在**一定条件下才会生效**，不是说每一个自动配置类都会被加载！！我们可以开启 debug 模式然后我们就可以看到自动配置报告。使用 `debug:true` 在主配置文件中。下面就是自动配置报告！

```log
=========================
AUTO-CONF IGURATION REPORT
=========================

Positive matches:(自动配置类启用的）

DispatcherServletAutoconfiguration matched:
-@ConditionalonClass found required class
‘org.springframework.web.servlet.Dispatcherservlet“；ConditionalonMissingclass did not find
unwanted class (Onclasscondition)
-@conditionalOnwebApplication(required)found StandardServletEnvironment
(onwebApplicationcondition)


Negative matches:(没有启动，没有匹配成功的自动配置类）

ActiveMQAutoconfiguration:
Did not match:
-@ConditionalOnClass did not find required classes‘javax.jms.ConnectionFactory“，
‘org.apache.activemq.ActivemQConnectionFactory“(Onclasscondition)

```

