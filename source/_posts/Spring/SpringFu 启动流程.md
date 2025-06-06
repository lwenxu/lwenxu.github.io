---
title: SpringFu 启动流程
date: 2021-07-11 15:02:37
tags: [SpringFu,源码]
categories:
	- SpringFu
	- Spring

---

整个项目的启动是通过 `Jafu.webApplication().run()` 完成的，与 SpringBoot 的 `SpringApplication.run(SpringfuDemoApplication.class, args)` 有异曲同工之妙。那么核心点就是 `webApplication()` 和` run()` 方法。

通过 JaFu 的成员结构和注释我们能看出来，他其实是一个客户端类或者说工厂类。

<img src="https://lwen-pic.oss-cn-beijing.aliyuncs.com/image-20210711164338078.png" alt="image-20210711164338078" style="zoom: 33%;" />

<center>[JaFu 成员结构]</center>

其实这个写法在日常代码中也是有大量应用场景的，比如我们封装一个二方包或者客户端给其他人用，他们使用起来可能需要 new 好多类，配置很多参数之后再做组装。这样一系列操作对于不熟悉这个二方包的人是非常不友好的，接入成本太高。为了防止被祭天，我们尽量采用这种工厂类或者富客户端的方式减少用户负担，屏蔽细节。

<img src="https://lwen-pic.oss-cn-beijing.aliyuncs.com/image-20210711165132287.png" alt="image-20210711165132287" style="zoom: 50%;" />

<center>[反面教材]</center>

## 配置 Web APp



```java
	public static JafuApplication webApplication(Consumer<ApplicationDsl> dsl) {
		return new JafuApplication(new ApplicationDsl(dsl)) {
			@Override
			protected ConfigurableApplicationContext createContext() {
				return new ServletWebServerApplicationContext();
			}
		};
	}
```

核心就是 new 了一个 JafuApplication ，并且指明了当前的运行环境为 `Web` 所以创建了 `ServletWebServerApplicationContext` 
