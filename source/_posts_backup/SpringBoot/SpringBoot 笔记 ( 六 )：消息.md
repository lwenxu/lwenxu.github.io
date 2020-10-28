---
title: SpringBoot 笔记 ( 六)：消息
date: 2018-05-20 15:01:28
tags: SpringBoot
categories:
 - SpringBoot
---
# SpringBoot 笔记 (六): 消息

# 1.基本概念

## 1.应用场景

<!--more-->

![image](https://user-images.githubusercontent.com/22151420/40310511-a4d4be00-5d3f-11e8-9e30-0014fd99d851.png)

![image](https://user-images.githubusercontent.com/22151420/40310554-beba6612-5d3f-11e8-99da-35ff7787e56f.png)

## 2.重要概念

- 消息代理（broker）：消息队列服务器
- 目的地：消息消费者

### 1.消息队列的两种目的地：

- 队列：点对点的通讯，这种就是消息生产者把消息发送到消息队列中，然后消息接受者去获取消息，获取后**这个消息就被移除**，消息的接受者可以有多个也就是可以有多个消费者，但是注意**一个消息只能被其中的一个消费者消费**。
- 主题：发布订阅通讯，广播形式。所有的接受者都可以收到消息。

### 2.JMS

这是一个基于JVM的消息代理的规范，但是他只是一个规范真正的实现则是 ActiveMQ和HornetMQ等等

### 3.AMQP

高级消息队列，他也是消息队列的规范但是他是兼容JMS的，他的具体实现就是我们常常听到的RabbitMQ

### 4.JMS和AMQP对比

![image](https://user-images.githubusercontent.com/22151420/40310950-fe85a152-5d40-11e8-9304-fb2f976a8a97.png)

### 5.Spring支持

spring-jms提供了对JMS的支持
spring-rabbit提供了对AMQP的支持
需要ConnectionFactory的实现来连接消息代理
提供JmsTemplate、Rabbit Template来发送消息
@JmsListener(JMS）、@RabbitListener(AMQP)注解在方法上监听消息代理发
布的消息
@EnableJms、@EnableRabbit开启支持

### 6.Spring Boot自动配置

JmsAutoConfiguration
RabbitAutoConfiguration

### 7.RabbitMQ基本结构

![image](https://user-images.githubusercontent.com/22151420/40311287-f1cd9932-5d41-11e8-849b-952b3d650a87.png)

首先我们产生消息的我们叫做 `生产者(Publisher)` 然后我们生产者会把消息发送给 我们的`消息服务器(Broker)` 中的一个`虚拟主机(Virtual Host)`，虚拟主机中有一个专门用来接受生产者消息的组件就是 `交换机Exchange` ，在接收到消息以后我们的路由器就和我们真正的某条 `消息队列Queue` 通过路由键连接起来，这里的消息队列有多个，然后消息队列与路由器的链接也是多对多的。最后就是我们的消费者去消息队列中消费消息，采用的就是建立TCP链接，但是我们为了节省资源开销，采用了一条TCP的多路复用，也就是在一个TCP中开辟了多个信道来传输数据。

### 8.Exchange类型

- direct 直连模式，在我们消息带过来的类型和消息键完全匹配的时候我们直接转发到对应的队列  ---- 点对点
- fanout 广播模式，对所有的消息都会广播到每一个队列中，这是最快的 ---- 广播
- topic 模式匹配方式，单词级别的匹配，**消息的消费者使用的**   '# '表示匹配0个或者多个单词  ’*‘ 表示匹配一个单词

# 2.RabbitMQ

## 1.docker安装RabbitMQ

```shell 
docker pull registry.docker-cn.com/library/rabbitmq:3.7-management   #获取具有web管理面板的镜像+使用加速
docker run -d -p 5672:5672 -p 8002:15672 --name rabbitmq_dev c51d1c73d028
```

## 2.管理界面

输入 ip:15672 输入 guest/guest 登录进入管理界面

## 3.SpringBoot整合RabbitMQ

### 1.添加starter

```xml 
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 2.自动配置原理

```java 
@Configuration
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
    @Configuration
    @ConditionalOnMissingBean(ConnectionFactory.class)
    protected static class RabbitConnectionFactoryCreator {
         @Bean
		public CachingConnectionFactory rabbitConnectionFactory(RabbitProperties config){}
    }
       
    @Configuration
    @Import(RabbitConnectionFactoryCreator.class)
    protected static class RabbitTemplateConfiguration{
         @Bean
		@ConditionalOnSingleCandidate(ConnectionFactory.class)
		@ConditionalOnMissingBean(RabbitTemplate.class)
		public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {}
        
         @Bean
		@ConditionalOnSingleCandidate(ConnectionFactory.class)
		@ConditionalOnProperty(prefix="spring.rabbitmq", name = "dynamic", matchIfMissing = true)
		@ConditionalOnMissingBean(AmqpAdmin.class)
		public AmqpAdmin amqpAdmin(ConnectionFactory connectionFactory) {}
			
    }
        
    @Configuration
    @ConditionalOnClass(RabbitMessagingTemplate.class)
    @ConditionalOnMissingBean(RabbitMessagingTemplate.class)
    @Import(RabbitTemplateConfiguration.class)
    protected static class MessagingTemplateConfiguration {
         @Bean
		@ConditionalOnSingleCandidate(RabbitTemplate.class)
		public RabbitMessagingTemplate rabbitMessagingTemplate{}
    }
}
```

可以看到我们整个的自动配置类其实是我们的三个自动配置类的组合，在这个配置类里面我们配置了链接工厂，以及两个供我们便捷操作RabbitMQ的两个Template 这个和 redis 以及 jdbc 的 template 非常的类似。

#### 1.rabbitConnectionFactory 

这个bean就是配置了他的配置信息，包括地址端口什么的，这些可以看到是从我们的 `RabbitProperties` 中获取的，也就是 `@EnableConfigurationProperties(RabbitProperties.class)` 导入的 properties ，我们点进去就可以看到它对应的配置项以及关联的配置文件的前缀。其中前缀就是 "spring.rabbitmq" 下面就是具体的配置了。

```java
@ConfigurationProperties(prefix = "spring.rabbitmq")
public class RabbitProperties {
    private String host = "localhost";
	private int port = 5672;
	private String username;
	//....
}
```

那么我们就在配置文件中填写配置项：

```yaml
spring:
  rabbitmq:
    host: 219.245.18.94
    username: guest
    password: guest
    virtual-host: /
    port: 5672
```

#### 2.rabbitTemplate

这个就是操作RabbitMQ的接口组件。

#### 3.amqpAdmin

而这个是RabbitMQ的系统管理组件

#### 4.rabbitMessagingTemplate

消息相关的操作接口。

### 3.简单的API操作

```java
@Test
public void rabbitSend(){
    //自定义消息头和消息体形式
    rabbitTemplate.send("lwen.direct",
            "lwen.q1",
            new Message("hello".getBytes(),new MessageProperties()));

    //简单一点直接传输消息体，并且自动进行序列化  对象默认使用jdk序列化
    HashMap<String, Object> msg = new HashMap<>();
    msg.put("m1", "hello");
    msg.put("m2", Arrays.asList("hello", "world"));
    rabbitTemplate.convertAndSend("lwen.direct",
            "lwen.q1",
            msg);

}

@Test
public void receiveMsg(){
    //获取消息进行反序列化
    Object q1 = rabbitTemplate.receiveAndConvert("q1");
    System.out.println(q1.getClass());
    System.out.println(q1);
}
```

上面就是简单的读写消息队列的操作。

以上我们看到的是单播的例子，其实广播以及模式匹配我们只是使用了不同的exchange就能达到目的。

### 4.配置序列化规则

可以看到上面的自动序列化出来的东西我们是没办法看的，我们更希望使用json等序列化工具。我们看一下他的序列化的配置。

在 `public RabbitTemplate rabbitTemplate()` 方法中 可以看到 `MessageConverter messageConverter = this.messageConverter.getIfUnique();` 就是获取消息转换的工具。

![image](https://user-images.githubusercontent.com/22151420/40348682-f21c0582-5dd6-11e8-9b7e-cf67afe255cd.png)

可以看到有这么多 MessageConverter 的具体实现，我们就是使用json的。

```java
@Configuration
public class AMQPConfig {
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

之后我们可以在RabbitMQ的管理面板上看到这种格式的数据了，而不是二进制：

```json
{"m1":"hello","m2":["hello","world"]}
```

### 5.消息监听

我们对一些应用解耦的话我们就需要使用消息队列，那么消息队列就需要有通知的功能，这里我们只需要两个注解就能搞定这个问题。

1. 首先我们需要开启RabbitMQ的注解 @EnableRabbit

```java
@SpringBootApplication
@EnableRabbit
public class BootMqApplication {

    public static void main(String[] args) {
        SpringApplication.run(BootMqApplication.class, args);
    }
}
```

1. 然后我们对接受消息的方法加上@RabbitListener注解进行监听

```java
@Service
public class BookService {
    @RabbitListener(queues = "q1")
    public void bookReceive(Book book){
        System.out.println(book);
    }
}
```

### 6.amqpAdmin

前面我们看到在自动配置的时候他们帮我们创建了一个 amqpAdmin 这个组件，然后我们是可以拿这个组件进行创建路由、队列、以及绑定规则，也就是我们在管理面板上能完成的操作。

```java
@Autowired
AmqpAdmin amqpAdmin
@Test
public void rabbitAdmins(){
    //创建路由  DirectExchange 是Exchange的实现类  类自己找  有很多
    amqpAdmin.declareExchange(new DirectExchange("amqpAdmin.exchange"));
    //创建队列
    amqpAdmin.declareQueue(new Queue("amqpAdmin.queue"));
    //绑定
    amqpAdmin.declareBinding(new Binding("amqpAdmin.queue",
            Binding.DestinationType.QUEUE,
            "amqpAdmin.exchange",
            "amqp.key",
            null));
}
```

