---
title: SpringBoot 笔记(八):任务
date: 2018-05-22 17:01:28
tags: SpringBoot
categories:
 - SpringBoot
---
# SpringBoot 笔记 (八):任务

# 1.异步任务

## 1.开启异步任务注解

@EnableAsync

## 2.对Service层方法开启异步

@Async

```java
@Service
public class TaskService {
    @Async
    public void task() throws InterruptedException {
        System.out.println("start");
        Thread.sleep(3000);
        System.out.println("end");
    }
}
```

```java
@EnableAsync
@SpringBootApplication
public class TaskApplication {

    public static void main(String[] args) {
        SpringApplication.run(TaskApplication.class, args);
    }
}
```

# 2.定时任务

## 1.开启调度

@EnableScheduling

## 2.@Scheduled(cron = "") 

调度注解，放在Service层即可。

## 3.cron格式

秒(0-59)  分(0-59)   时(0-23)  日(1-31)  月(1-12)   周(0-7  其中0/7表示周日)

这几个位置可写的值不仅仅是上面的数字，还可以是表达式：

- 枚举：1,2,3,4
- 范围：2-5
- 任意： *
- 步长： /3  每3步
- 冲突：？ 当日和星期回冲突的时候

一些例子：

1 * * *？ *  2-7  每周的周二到周天的每一分钟的第一秒触发

1-7 * * ？ *  3  每周三的每分钟的1-7秒触发

0  0/5  14,18  ？ *  1-6  每周的1-6 14点和18点 每个五分钟执行一次

0 15 10？* 1-6 每个月的周一至周六10:15分执行一次

0 0 2？* 6 每个月的最后一个周六凌晨2点执行一次

# 3.邮件发送

## 1.配置原理

配置pom文件

```xml
 		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
        <dependency>
            <groupId>com.sun.mail</groupId>
            <artifactId>javax.mail</artifactId>
        </dependency>
```

查看MailSenderAutoConfiguration的自动配置类

```java
@Bean
public JavaMailSenderImpl mailSender() {
   JavaMailSenderImpl sender = new JavaMailSenderImpl();
   if (this.session != null) {
      sender.setSession(this.session);
   }
   else {
      applyProperties(sender);
   }
   return sender;
}
```

可以看到主要就是注入了一个 JavaMailSenderImpl 这个类的实例，然后我们就可以借助这个实例进行发送邮件。

## 2.配置属性

```yaml
spring:
  mail:
    host:
    username:
    password:
    default-encoding: utf-8
    protocol: smtp
```

## 3.发送邮件

```java
@Autowired
JavaMailSenderImpl javaMailSender;

/**
 * 简单的文本文件的发送
 */
@Test
public void senderSimple(){
    SimpleMailMessage message = new SimpleMailMessage();
    message.setSubject("开会");
    message.setText("你好开会");
    message.setFrom("lwenxu");
    message.setTo("xpf199741@outlook.com");
    javaMailSender.send(message);
}

@Test
public void senderMulti() throws MessagingException {
    MimeMessage mimeMessage = javaMailSender.createMimeMessage();
    MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, true);

    mimeMessageHelper.setSubject("开会");
    mimeMessageHelper.setText("你好开会");
    mimeMessageHelper.setFrom("lwenxu");
    mimeMessageHelper.setTo("11@qq.com");
    mimeMessageHelper.addAttachment("1.jpg", new File("C:\\asd.jpg"));
    
    javaMailSender.send(mimeMessage);
}
```