---

title: MapStruct 简易教程
date: 2020-03-07 17:39:26
tags: Java 工具包
categories:
  - JAVA
  - Java工具包
---

## 使用场景

我们经常会遇到 DO 和 DTO 以及 DTO 和 VO 之间的转换，一般我们的做法是使用 BeanUtil.copy 然后对于一些特殊的字段进行 Set 。但是这样做有几个痛点：

1. 其实这个过程很没有技术含量，基本都是 CV 操作
2. 在业务代码中做这种类型转换的业务无关代码，让代码显得比较杂乱
3. 如果是几万的 beanList 的拷贝会有性能问题
4. 希望能抽出工具类但是好像也不太合适，每个都是一个工具类，而且对于复杂的转换垃圾代码太多
5. 希望能够通过简单的注解或者 xml 配置完成这些操作

## 解法

Map-struct 就是这样一个基于注解的编译后置处理器，让你写一些简单的注解满足数据转换的需求，类似于 lombok 是编译时期完成的，也没有什么性能问题。

## 安装

```xml
<properties>  
  <org.mapstruct.version>1.3.0.Final</org.mapstruct.version>
</properties>

<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-jdk8</artifactId>
    <version>${org.mapstruct.version}</version>
</dependency>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>${org.mapstruct.version}</version>
    <scope>provided</scope>
</dependency>
```



## 使用

### @mapper 

1. 当我们使用 `componentModel = "spring"` 意味着我们可以使用 spring 进行依赖注入

2. `componentModel= "default"`：默认，可以通过 Mappers.getMapper(Class) 方式获取实例对象


### @mappings 和 @mapping

1. @Mapping(source = "birthday", target = "birth")
2. 带有时间格式化的 @Mapping(source = "birthday", target = "birthDateFormat", dateFormat = "yyyy-MM-dd HH:mm:ss")
3. Java 代码格式化 @Mapping(target = "birthExpressionFormat", expression = "java(org.apache.commons.lang3.time.DateFormatUtils.format(person.getBirthday(),\"yyyy-MM-dd HH:mm:ss\"))")
4. 忽略字段  @Mapping(target = "email", ignore = true)


## 简单的 Demo

### DO

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table
public class Worker implements Serializable {
    private static final long serialVersionUID = 629213888727713294L;
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private String email;
    @JsonIgnore
    private String password;
    private String type;
}
```

### VO

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class WorkerVO {
    private Long id;
    private String name;
    private String email;
    private String password;
    private String type;
    private String token;
}



```

### 转换 Mapper

```java
@Mapper(componentModel = "spring")
public interface UserMapper {

    @Mappings(
            @Mapping(source = "worker.password",target = "token")
    )
    public WorkerVO convert(Worker worker);
}
```





