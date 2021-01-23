---
title: Logback 日志框架详解
date: 2020-03-07 10:39:26
tags: log
categories:
  - JAVA
  - LOG
---

## 配置 pattern

|        符号        |                             功能                             | 性能损耗 |
| :----------------: | :----------------------------------------------------------: | :------: |
|         %c         |                          打印全类名                          |    大    |
|         %n         |    换行，一定要在 pattern 的最后加上不然展示的就是一行了     |    小    |
|        %ex         | 打印出异常: 1. %ex{full} 全部异常堆栈  2. %ex{short} 最上面一层堆栈 |    小    |
| %d{MM-dd HH:mm:ss} |                          格式化时间                          |    小    |
|       %level       |                         打印日志级别                         |    小    |
|         %m         |                         打印方法名称                         |    小    |



举个简单的例子:

```xml
<property name="CONSOLE_LOG_PATTERN" value="%d{MM-dd HH:mm:ss} |-%level  %c#%m - %msg %ex{full}%n"/>

<!-- ConsoleAppender 控制台输出日志 -->
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
  <target>System.out</target>
  <encoder>
    <pattern>${CONSOLE_LOG_PATTERN}</pattern>
    <charset class="java.nio.charset.Charset">UTF-8</charset>
  </encoder>
</appender>
```





可参考连接：

- [具体每个的标签作用](https://juejin.im/post/5b51f85c5188251af91a7525)
- 