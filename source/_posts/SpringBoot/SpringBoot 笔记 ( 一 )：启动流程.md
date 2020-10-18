---
title: SpringBoot笔记(一):启动流程
date: 2018-04-13 21:01:28
tags: SpringBoot
categories:
 - SpringBoot
---

# SpringBoot 笔记(一): 启动流程

## 1. 配置开发环境

### 1. 创建 Maven 项目
然后我们首先在项目里面加上编译环境，防止每一次更新 Maven 的时候导致项目的语言级别自动被改成 Java5 然后导致编译不通过的问题。
<!-- more -->
```xml
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
```
### 2. Maven 中添加依赖
&emsp;&emsp;  一开始加上最基础的依赖就是 parent 父项目和 web 的 starter 。

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

&emsp;&emsp;其实很明显父工程的作用就是用来版本控制的，为什么这么说？我们跟踪一下父工程，来到父工程的 pox.xml 看到如下的配置。

```xml
<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-dependencies</artifactId>
		<version>1.5.9.RELEASE</version>
		<relativePath>../../spring-boot-dependencies</relativePath>
	</parent>
```
&emsp;&emsp;  继续跟踪这个父工程可以看到以下内容。

```xml
<properties>
		<!-- Dependency versions -->
		<activemq.version>5.14.5</activemq.version>
		<antlr2.version>2.7.7</antlr2.version>
		<appengine-sdk.version>1.9.59</appengine-sdk.version>
		<artemis.version>1.5.5</artemis.version>
		<aspectj.version>1.8.13</aspectj.version>
		<assertj.version>2.6.0</assertj.version>
		<atomikos.version>3.9.3</atomikos.version>
		....
		....
</properties>
```
&emsp;&emsp;  好的，现在终于算是找到源头了，这些 properties 就是用来规定 springboot 整合的这些框架的版本，所以说我们只用管好 `springboot` 的版本，对于里面封装的各个组件我们不用操心，他都帮我们配置好了。但是我们也是可以自己继续进行定制的，后面会说到相关的东西。

&emsp;&emsp;  可以看到里面添加的另外一个 starter 就是 web 的 starter ，其实在 springboot 中的 starter 的命名很规范一眼就能看出是干嘛的。什么是 starter ？ 简单来说就是 springboot 用来整合各个框架的包，我们需要什么框架什么功能就添加上这些对应的 starter 在 springboot 中就可以享受开箱即用的快感了！

### 3. 编写主类
&emsp;&emsp;  做完了这些最简单的配置，我们就可以开始写启动代码了，代码非常简单只有几行。

```java

// @SpringBootApplication 来标注一个主程序类
@SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        // Spring启动！
SpringApplication.run(MainApplication.class,args);
    }
}
```
&emsp;&emsp;  很神奇这样我们就写好了一个完全可以跑起来的 springboot 应用，我们只是写了一个注解和一行代码而已，为什么会这么简单，让我们看看 `@SpringBootApplication` 注解。

#### 1. @SpringBootApplication 注解
&emsp;&emsp;  首先可以看到的是这是一个组合注解，里面有如下内容：
```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
```
&emsp;&emsp;  好一个个的来看:

1. `@SpringBootConfiguration` 这个注解就是标注这是一个 `springboot` 的配置类，也就是可以进行 `springboot` 的配置，配置类就等于配置文件，我们要有这个等价的概念。然后我们在打开这个注解发现里面就是 `spring`的一个 Java Config 注解 `@Configuration` 所以说这个实际上是 `SpringBoot ` 的主配置类。
2. `@EnableAutoConfiguration` 他是用来开启自动配置，也就就是 `springboot` 的核心，其实最终起作用的就是 `spring-boot-configuration-processor-1.5.9.RELEASE.jar` 这个包在起作用，后面会分析到。看看这个注解里面的内容！
```java
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
```
两个注解:

##### 1.@AutoConfigurationPackage

第一个是自动导组件的注解，底层调用了Spring 底层注解(@Import)给容器中导入一个组件。也就是 `@Import(AutoConfigurationPackages.Registrar.class)` 可以看到他导入了一个自动配置包的注册器。Import 是Spring 的一个底层的注解，就是用来导入组件。然后我来查看这个组件到底做了什么：

```java
@Order(Ordered.HIGHEST_PRECEDENCE)
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

   @Override
   public void registerBeanDefinitions(AnnotationMetadata metadata,
         BeanDefinitionRegistry registry) {
      register(registry, new PackageImport(metadata).getPackageName()); //获取元数据中的包信息，然后导入这些包里面的组件 核心！！！！
   }

   @Override
   public Set<Object> determineImports(AnnotationMetadata metadata) {
      return Collections.<Object>singleton(new PackageImport(metadata));
   }
}
```

​	如果在上面的那个注册组件的位置打上断点的话，我们就会发现这个地方计算的报名就是我们主类所在的包的包名，也 就是需要导入我们主类所在的包以及子包所对应的组件，我们自己的组件。所以说我们必须把其他的类放到与主类平行的其他包中，否则没办法加载这些类到容器里。

##### 2.@Import(EnableAutoConfigurationImportSelector.class)

第二个就是导入一个 Seletor 组件，先看一下这个组件的代码，里面很简单就是一个方法，但是我们发现他是继承了一个类的所以说我们需要看看父类的方法。

```java
public class EnableAutoConfigurationImportSelector  extends AutoConfigurationImportSelector{
          //..........
      }
```

然后我们看到父类中的一个非常重要的方法， `selectImports` 这个方法就完成了我们的自动配置！

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
   if (!isEnabled(annotationMetadata)) {
      return NO_IMPORTS;
   }
   try {
      AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
            .loadMetadata(this.beanClassLoader);
      AnnotationAttributes attributes = getAttributes(annotationMetadata);
      List<String> configurations = getCandidateConfigurations(annotationMetadata,
            attributes);
      configurations = removeDuplicates(configurations);
      configurations = sort(configurations, autoConfigurationMetadata);
      Set<String> exclusions = getExclusions(annotationMetadata, attributes);
      checkExcludedClasses(configurations, exclusions);
      configurations.removeAll(exclusions);
      configurations = filter(configurations, autoConfigurationMetadata);
      fireAutoConfigurationImportEvents(configurations, exclusions);
      return configurations.toArray(new String[configurations.size()]);
   }
   catch (IOException ex) {
      throw new IllegalStateException(ex);
   }
}
```

可以看到里面有个 configurations 变量这个就是我们的自动配置的一些包。可以打断点看到里面的内容。

这个组件底层就是各种 beanFactory 也就是我们 Spring 底层的 Ioc 容器，然后就是向容器里面导入我们的自动配置类，`xxxAutoConfiguration` 这种类，也就是 `spring-boot-configuration-processor-1.5.9.RELEASE.jar` 包里面的内容。

那么具体把这个包里面的哪些 `xxxAutoConfiguration` 加入到容器里面呢？ 这就需要看一个方法。在这个 Selector 的底层有一个 `getCandidateConfigurations` 方法，这个方法就是去扫描所有包下面的 `META-INF/spring.factories` 文件，然后这些文件中的 `EnableAutoConfiguratio` 属性就会有需要配置组件要导入哪些 `xxxAutoConfiguration` 类对这个组件进行自动配置。可以看一下`spring-boot-configuration-processor-1.5.9.RELEASE.jar` 里面的具体内容：

```xml
org.springframework.boot.test.autoconfigure.orm.jpa.AutoConfigureDataJpa=\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration
```
&emsp;&emsp;  可以看到这就是他的自动配置机制！
#### 2. run 方法
在这个方法的底层 new 了一个 `SpringApplication` 的东西，这个很明显就是真正的 `springboot` 的启动类了。最后调用到了 ` public ConfigurableApplicationContext run(String... args)` 这里面做了一些列的启动操作，就不具体看了。然后我们传入的那个类被放到了 `sources = new LinkedHashSet();` 这个数据结构里面。感觉没怎么被使用。

&emsp;&emsp;  这样其实我们的 springboot 应用已经可以开始跑起来了。但是我们不能访问任何页面，主要是因为我们没有 controller ！好的接着编写一个简单的 controller。

### 4. controller 编写
```java
@Controller //标注他是 controller
@ResponseBody //能够返回数据
public class HelloController {
    @GetMapping("/hello") //url映射
    public String hello(){
        return "Hello World!";
    }
}
```
&emsp;&emsp;  @Controller 、@ResponseBody 这两个注解我们可以合并采用 @RestController 这个其实就是上面两个注解的组合注解。然后下面的那个 @GetMapping 就是采用的 get 方式请求时候的映射，然后对应的还有 post、put、delete 等等，这些都是 restful 接口中有规定的在何种情况使用何种请求方式。

&emsp;&emsp;  此时我们去访问 `localhost:8080/hello` 就会显示 “Hello World！” 了！

### 5. 打包部署
&emsp;&emsp;  这个在 springboot 中打包部署非常容易就是采用一个 Maven 插件我们就可以打包成 jar 而非 war 。那么也就是说我们可以直接使用 `java -jar` 命令来运行这个 jar 包。等等 ！难道不需要 tomcat 之类的 web 容器吗？ 对的！因为`springboot` 是内嵌了 tomcat 所以我们根本没有部署到 web 容器这一说，所以说部署简洁。那么我们需要加入打包插件。

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```
&emsp;&emsp;  接下来只需要在 Maven 中执行 package 命令就可以在 target 目录生成 jar 包。并且我们可以使用 `java -jar ` 来执行这个 jar 。


### 6. resource 目录结构
resources文件夹中目录结构：

- static：保存所有的静态资源如 js css  images。
- templates：保存所有的模板页面 html 模板。
- application.properties：Spring Boot应用的配置文件。

