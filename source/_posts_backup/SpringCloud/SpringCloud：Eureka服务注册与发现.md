---
title: SpringCloud：Eureka服务注册与发现
date: 2018-05-26 10:51:28
tags: SpringCloud
categories:
 - SpringCloud
---

# SpringCloud：Eureka服务注册与发现

Eureka 其实就是一个 服务注册与发现的中心，也就是相当于我们前面做的一些生产者的服务需要注册到我们的注册中心，那么我们的消费者就不用把代码写死，而是可以去服务中心订阅对应的服务，获取服务的最新地址，并且进行逻辑解耦。

说的更简单一点它就相当于我们的 Dubbo 中的zookeeper 的功能就是用来服务发现的和注册的。他是一个CS架构的一个应用，也就是我们会有客户端和服务端，接下来就准备使用这个服务注册中心。

![image](SpringCloud：Eureka服务注册与发现.assets/40571908-d4d43274-60d3-11e8-88fa-965317a12105.png)

那么现在我们就只需要在我们以前的那个项目上在进行进一步的操作，也就是加上注册中心就好，那么现在就开始搭建！

<!--more-->

## 1.配置Eureka服务端

前面也提到了Eureka是一个CS架构的应用，所以说我们的服务器端说白了就是一个新的SpringBoot的应用，所以我们可以直接开一个Eureka 的服务端并且它带有一个Dashboard 。接着我们就需要整合我们的客户端，客户端说白了就是我们的服务提供者，我们的服务消费者，我们的服务提供者以及消费者需要去Eureka上面看哪些服务注册了能够消费哪些服务。

ok，还是先建一个module，然后再这个module中创建我们的服务端。

### 1.pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-cloud</artifactId>
        <groupId>com.lwen</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>eureka-server-7001</artifactId>

    <dependencies>
        <!--eureka-server服务端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>
        <!-- 修改后立即生效，热部署 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>

</project>
```

### 2.配置文件

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址（单机）。
      defaultZone: http://eureka7003.com:7003,http://eureka7003.com:7002
```

注意一下这个地方我们使用了 `http://eureka7003.com` url 这个东西其实就是我们在 host 中配置了 `127.0.0.1` 形成的。

### 3.启动类

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServer7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001.class, args);
    }
}
```

这里唯一需要注意的就是我们这是一个Eureka的服务端，所以我们需要配置这个注解 `@EnableEurekaServer` 。然后同样的方法我们在建一个 Eureka的server端，就形成一个集群。

最后启动的时候我们就能发现我们的Eureka集群是有备份的。

![1527409895357](SpringCloud：Eureka服务注册与发现.assets/1527409895357.png)

## 2.配置Eureka客户端

### 1.配置生产者

#### 1.配置pom

```xml
<!-- actuator监控信息完善 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<!-- 将微服务provider侧注册进eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

#### 2.配置启动类

```java
@EnableEurekaClient
@SpringBootApplication
@MapperScan("lwen.dao")
public class DeptApp8001 {
    public static void main(String[] args) {
        SpringApplication.run(DeptApp8001.class, args);
    }
}
```

主要就是添加 `@EnableEurekaClient` 注解。

#### 3.配置yml

```yaml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      #defaultZone: http://localhost:7001/eureka
       defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
  instance:
    instance-id: cloud-dept8001
    prefer-ip-address: true     #访问路径可以显示IP地址
```

### 3.配置消费者

#### 1.配置pom

同上

#### 2.配置启动类

需要加上Ribbon 的配置，因为我们需要使用为服务名去调用对应的服务，而不再采用以前的url地址了。`@RibbonClient(name = "CLOUD-DEPT")`

#### 3.配置RestTemplate

```java
@Configuration
public class RestConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

进行负载均衡

#### 4.配置yml

```yaml
eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```

这里就是相对于上面我们没有配置 instance 这是因为我们的消费者只是在上面寻找服务提供者消费这些提供者，而不用去注册这些消费者。

## 3.配置负载均衡

`Ribbon` 就是用来做负载均衡的。只是和一般的负载均衡的处理方式不同的是这个东西是客户端的负载均衡，所谓的客户端负载均衡就是我们的服务消费者会过来消费服务的时候对当前的生产者的状态进行判断，然后消费者主动去找负载比较轻的服务端去做消费。

所以说我们的客户端（消费者）就自带了负载均衡的策略，而不是说我们的客户端来了以后我们有一个中间的件，例如我们常用的 Nginx ，来判断我们采用什么策略来完成这个客户端的访问请求。

那么很明显，我们的代码必须是写在客户端这一方，其实真正的负载均衡在 SpringCloud 中非常简单，就是开启负载均衡，然后在我们的 rest 客户端添加一个负载均衡的标志，也就是我们的 rest 客户端是最后真正去访问我们的客户端的，所以说配置起来就两步，最后我们可以对他们的负载均衡的策略进行调整。

### 1.开启负载均衡

`@RibbonClient(name = "CLOUD-DEPT")` 我们只需要在主类上加上这个我们就算是开启了负载均衡的功能，需要注意的一点就是我们需要在上面说明对哪个服务访问的时候采用负载均衡，因为有的服务可以负债均衡有的根本不需要，接下来我们只需要对 rest 客户端修改修改即可。

```java
@RibbonClient(name = "CLOUD-DEPT")
@EnableEurekaClient
@SpringBootApplication
public class DeptConsumerApp81 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumerApp81.class, args);
    }
}
```

### 2.rest 客户端启用负载均衡

可以看到下面的代码我们只需要在这个rest客户端的获取的时候添加一个注解我们的rest客户端就拥有了负载均衡的功能。

```java
@Configuration
public class RestConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### 3.自定义负载均衡算法

对于Nginx我们肯定熟悉的就是他有很多的自定义算法，也就是用来做负载均衡的。当然这个地方默认的负载均衡的算法就是轮询，然后我们还可以自定义一些：

![1527507569023](SpringCloud：Eureka服务注册与发现.assets/1527507569023.png)

可以看到这里有很多的负载均衡的算法，那些具体的类其实都是算法，然后可以看到有一个 MyLBRule1 这个是我自定义的一个负载均衡算法，其实很简单就是拷贝一个类然后照着他的样子重写 `choose` 方法即可。

```java
package rules;

import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.Server;

import java.util.List;

public class RandomRule_ZY extends AbstractLoadBalancerRule
{

	// total = 0 // 当total==5以后，我们指针才能往下走，
	// index = 0 // 当前对外提供服务的服务器地址，
	// total需要重新置为零，但是已经达到过一个5次，我们的index = 1
	// 分析：我们5次，但是微服务只有8001 8002 8003 三台，OK？
	// 
	
	
	private int total = 0; 			// 总共被调用的次数，目前要求每台被调用5次
	private int currentIndex = 0;	// 当前提供服务的机器号

	public Server choose(ILoadBalancer lb, Object key)
	{
		if (lb == null) {
			return null;
		}
		Server server = null;

		while (server == null) {
			if (Thread.interrupted()) {
				return null;
			}
			List<Server> upList = lb.getReachableServers();
			List<Server> allList = lb.getAllServers();

			int serverCount = allList.size();
			if (serverCount == 0) {
				/*
				 * No servers. End regardless of pass, because subsequent passes only get more
				 * restrictive.
				 */
				return null;
			}

//			int index = rand.nextInt(serverCount);// java.util.Random().nextInt(3);
//			server = upList.get(index);

			
//			private int total = 0; 			// 总共被调用的次数，目前要求每台被调用5次
//			private int currentIndex = 0;	// 当前提供服务的机器号
            if(total < 5)
            {
	            server = upList.get(currentIndex);
	            total++;
            }else {
	            total = 0;
	            currentIndex++;
	            if(currentIndex >= upList.size())
	            {
	              currentIndex = 0;
	            }
            }			
			
			
			if (server == null) {
				/*
				 * The only time this should happen is if the server list were somehow trimmed.
				 * This is a transient condition. Retry after yielding.
				 */
				Thread.yield();
				continue;
			}

			if (server.isAlive()) {
				return (server);
			}

			// Shouldn't actually happen.. but must be transient or a bug.
			server = null;
			Thread.yield();
		}

		return server;

	}

	@Override
	public Server choose(Object key)
	{
		return choose(getLoadBalancer(), key);
	}

	@Override
	public void initWithNiwsConfig(IClientConfig clientConfig)
	{
		// TODO Auto-generated method stub

	}

}
```

这是一个简单的算法，也就是用来每个服务器轮询5次的算法。



## 4.面向接口的服务调用

前面我们都看到了关于我们的微服务的调用其实就是使用我们的 rest 客户端去请求我们的生产者的 controller ，那么我们是不是能够使用更简洁的方式去调用我们的生产者的服务呢？也就是我们不再用手动的注入我们的 restTemplate 而是直接类似于我们以前那样直接调用我们的服务端的controller。

并且当我们可以使用这个面向接口的服务以后我们又如何进行负载均衡呢，因为我们以前的负载均衡的方式就是采用的对 rest 客户端添加注解，这里我们因为采用了接口的调用方式又如何使用负载均衡，这里 Feign 就帮助我们解决了这一系列的问题。

首先可以知道的是： Feign 也是一个基于客户端的面向接口的服务，所以我们需要在客户端进行配置，由于他是一个发布的controller 服务我们直接把他定义到 api 层面即可。然后我们在需要调用这个服务的地方也需要进行 Feign 的配置，也就是需要配置消费者。这样我们才能识别到 Feign 才对。

### 1.配置 API

我们建立一个 service 包，然后我们在这个包里写一个`接口` 注意是一个接口，因为我们是面向接口编程 的，这里主要要注意的就是我们的 url 就是我们以前的 rest 客户端的请求的 url 说啊比了就是我们服务提供者的方法的 url 。

那么另外一个非常重要的地方就是我们的 `@FeignClient` 这个说明了我们的接口就是一个 Feign 的 rest 调用接口 .

```
package lwen.service;

import lwen.entries.Dept;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import java.util.List;
@FeignClient("CLOUD-DEPT")
public interface DeptClientService {

    @GetMapping("/dept/list")
    List<Dept> findAll();

    @PostMapping("/dept")
    Boolean insertDept(Dept dept);
}
```

### 2.配置消费者

我们配置好了客户端其实我们就可以进行服务访问了,这里我们为了可以直接访问接口服务我们创建一个新的服务端的服务。

首先我们需要在主类上开启我们的 Feign 也就是使用注解 `@EnableFeignClients` 

```java
@EnableFeignClients("lwen.service")
@EnableEurekaClient
@SpringBootApplication
public class DeptConsumerFeignApp82 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumerFeignApp82.class, args);
    }
}
```

然后就是我们的，controller了这里就是去请求我们的后台的的服务提供者的 controller 层。

```java
package lwen.controller;

import lwen.entries.Dept;
import lwen.service.DeptClientService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class DeptController {

    @Autowired
    private DeptClientService deptClientService;

    @GetMapping("/dept/list")
    public List findAll() {
        return deptClientService.findAll();
    }

    @PostMapping("/dept")
    public Boolean insertDept(Dept dept) {
        return deptClientService.insertDept(dept);
    }
}
```

其实当我们写完这些东西配置都正确了以后我们就会发现我们的 ide 会有非常智能的提示，也就是把我们暴露的服务的controller 的方法显示成可运行的状态。

![1527511615518](SpringCloud：Eureka服务注册与发现.assets/1527511615518.png)

很神奇，接着我们就可以准备解决负载均衡的问题了。

### 3.负载均衡

好的，我们的 rest 客户端被省略了，那么我们如何进行负载均衡这是一个问题。但是事实上我们完全不用担心如何进行负载均衡，这是因为我们的 Feign 已经帮我们整合好了这些东西。

是的，也就是我们在标注 `@FeignClient` 这个注解的时候我们已经配置了负载均衡，在我们的每一个方法之上，并且他的默认的负载均衡的策略就是轮训方式，现在我们就可以启动我们的服务然后测试一下。的确是这样。

可是我们以前还可以对服务是可以进行负载均衡的策略进行定制的，其实这个也很简单，我们只需要在我们的API端写好我们的配置类即可。

```java
@FeignClient(value = "CLOUD-DEPT",configuration = MyRuleConfig.class)
public interface DeptClientService {

    @GetMapping("/dept/list")
    List<Dept> findAll();

    @PostMapping("/dept")
    Boolean insertDept(Dept dept);
}
```

可以看到我们的注解上加了新的属性，就是用来配置负载均衡的，并且值得注意的是他和 Ribbon 的负载均衡的属性一模一样。

## 5.服务熔断与服务降级

### 1.基本概念

其实在我们的应用中很有可能会出现服务压力太大的情况，这时候为了避免我们服务发生雪崩我们需要对他们进行必要的处理，比如这里提到的服务熔断和服务降级。

`服务熔断`：服务熔断一般是指软件系统中，由于某些原因使得服务出现了过载现象，为防止造成整个系统故障，从而采用的一种保护措施，所以很多地方把熔断亦称为过载保护。 这个其实就是在我们服务被熔断的时候还需要给客户端返回一个符合预期的友好的结果。

[^系统崩溃]: 这里提到了关于单个应用可能会导致整个系统的崩溃，其实这是存在的，也就是当我们的某个服务的负载太高，然后很多请求去请求我们的服务，我们的服务又去调用别的服务，然后这样的话就可能会导致整个系统的所有子模块都被调动起来，也就是整个系统的服务过载。上文提到的单个应用导致整个服务过载甚至崩溃的情况就是我们所说的扇出，或者雪崩。``

`服务降级` ：旅行箱是必备物，平常走走近处绰绰有余，但一旦出个远门，再大的箱子都白搭了，怎么办呢？常见的情景就是把物品拿出来分分堆，比了又比，最后一些非必需品的就忍痛放下了，等到下次箱子够用了，再带上用一用。而服务降级，就是这么回事，整体资源快不够了，忍痛将某些服务先关掉，待渡过难关，再开启回来。 但是这个过程中我们的关闭的服务也必须要返回一些友好的提示，不能让客户端调用抛出异常或者长时间等待。

### 2.熔断降级的异同

所以从上述分析来看，两者其实从有些角度看是有一定的类似性的：

1. **目的很一致**，都是从可用性可靠性着想，为防止系统的整体缓慢甚至崩溃，采用的技术手段；
2. **最终表现类似**，对于两者来说，最终让用户体验到的是某些功能暂时不可达或不可用；
3. **粒度一般都是服务级别**，当然，业界也有不少更细粒度的做法，比如做到数据持久层（允许查询，不允许增删改）；
4. **自治性要求很高**，熔断模式一般都是服务基于策略的自动触发，降级虽说可人工干预，但在微服务架构下，完全靠人显然不可能，开关预置、配置中心都是必要手段；

而两者的区别也是明显的：

1. **触发原因不太一样**，服务熔断一般是某个服务（下游服务）故障引起，而服务降级一般是从整体负荷考虑；
2. **管理目标的层次不太一样**，熔断其实是一个框架级的处理，每个微服务都需要（无层级之分），而降级一般需要对业务有层级之分（比如降级一般是从最外围服务开始）
3. **实现方式不太一样**，这个区别后面会单独来说

### 3.服务熔断配置

服务熔断的话一般来说我们只需要对于我们的生产者的某个方法进行熔断处理，说的简单点就是但我们的服务出现问题，需要被熔断的的时候我们需要有一个保险机制然后给客户端符合预期的结果，也就是我们的熔断以后的句柄（Handler）。

首先创建一个新的 Module ，然后我们需要在主启动类上面加上对熔断的支持。`@EnableCircuitBreaker` 

```java
package lwen.controller;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import lwen.entries.Dept;
import lwen.service.DeptService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;
@RestController
public class DeptController {
    @Autowired
    DeptService deptService;
    @HystrixCommand(fallbackMethod = "findAllFallBack")
    @GetMapping("/dept/list")
    public List<Dept> findAll() {
        return deptService.findAll();
    }
    @PostMapping("/dept")
    public Boolean insertDept(Dept dept) {
        return deptService.insertDept(dept);
    }
    public Dept findAllFallBack() {
        return new Dept().setName("No such data ...");
    }
}
```

可以看到上面的这个controller 就是我们对 findAll 这个方法进行了服务熔断的处理，也就是当我们服务出现问题的时候我们的熔断句柄就会被调用。这里就是我们设置的 fallback 。

因为这个方法肯定会抛出异常，所以我们也自然会调用我们的 fallback 

### 4.服务降级配置

服务降级一般是对于整个的一个服务来说的，所以我们的降级配置直接就配置在API中就好。

首先就是添加一个 `FallbackFactory`  主要就是对我们的服务每一个方法进行降级的句柄。

```java
package lwen.service;

import feign.hystrix.FallbackFactory;
import lwen.entries.Dept;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

@Component // 不要忘记添加，不要忘记添加
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService>
{
    @Override
    public DeptClientService create(Throwable throwable)
    {
        return new DeptClientService() {
            @Override
            public List<Dept> findAll() {
                ArrayList<Dept> list = new ArrayList<>();
                list.add(new Dept().setName("No such data"));
                return list;
            }

            @Override
            public Boolean insertDept(Dept dept) {
                return null;
            }
        };
    }
}
```

在我们的 API 中添加 `@FeignClient(value = "CLOUD-DEPT", configuration = MyRuleConfig.class, fallbackFactory = DeptClientServiceFallbackFactory.class)` 也就是我们的 `fallbackFactory` 

## 6.服务网关

服务网关就相当于我们网络中的网关一样，住哟啊就用着请求的转发代理作用。然后这里我们的目的就是把所有的微服务都放在内网，然后我们只对外暴露网关即可，这样我们的微服务才会更加的安全。

在 SpringCloud 中我们使用的网关是 Zuul ，他是一个独立的微服务，所以我们自己建立一个新的服务。这个服务其实只需要一个启动类和一个配置文件即可。

### 1.配置主类

```java
package lwen;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@SpringBootApplication
public class DeptZuul9527 {
    public static void main(String[] args) {
        SpringApplication.run(DeptZuul9527.class, args);
    }
}
```

### 2.配置文件

```yaml
server:
  port: 9527

spring:
  application:
    name: microservicecloud-zuul-gateway

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    instance-id: gateway-9527.com
    prefer-ip-address: true


zuul:
  #ignored-services: microservicecloud-dept
  prefix: /lwen  #服务前缀
  ignored-services: "*"
  routes:
    mydept.serviceId: cloud-dept
    mydept.path: /mydept/**   #服务路径  后面接的就是我们的controller 的地址
```

之后我们就可以访问对应的地址 `http://localhost:8080/lwen/mydept/controller_url`

## 7.统一配置服务

我们的开发过程中有时候为了更方便的股那里我们所有的微服务的配置文件我们可以单独启动一个微服务专门用来管理我们其他的微服务的配置文件，这里我们要做的就是这么一件事，采用的东西就叫做 Config 组件。

### 1.创建服务

首先还是创建一个新的服务，因为我们是使用一个独立的微服务来管理其他的微服务。这个微服务基本和我们一个普通的SpringBoot 应用没区别，但是我们还是需要加入我们的配置模块。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-cloud</artifactId>
        <groupId>com.lwen</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dept-config-3344</artifactId>

    <dependencies>
        <!-- springCloud Config -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <!-- 避免Config的Git插件报错：org/eclipse/jgit/api/TransportConfigCallback -->
        <dependency>
            <groupId>org.eclipse.jgit</groupId>
            <artifactId>org.eclipse.jgit</artifactId>
            <version>4.10.0.201712302008-r</version>
        </dependency>
        <!-- 图形化监控 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!-- 熔断 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <!-- 热部署插件 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

### 2.主类

```java
package lwen;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DeptConfigApp3344 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConfigApp3344.class, args);
    }
}
```

### 3.配置文件

```yaml
server:
  port: 3344

spring:
  application:
    name:  cloud-config
  cloud:
    config:
      server:
        git:
          uri: https://github.com/lwenxu/springcloudconfig.git #GitHub上面的git仓库名字
```

其实我们的主类基本没有任何的特别的地方，最主要的就是我们的配置文件，这里我们配置了我们 config 的server 也就是我们仓库的地址。这个东西并不需要我们配置 ssh ，这是因为我们的仓库主要是公开的我们的服务就可以访问到。

### 4.配置客户端

客户端我们需要一个新的配置文件，做一些通用的不可变的配置，这是 boostrap.yml 配置文件

```yaml
spring:
  cloud:
    config:
      name: application #需要从github上读取的资源名称，注意没有yml后缀名
#      profile: test   #本次访问的配置项
      label: master   
      uri: http://config-3344.com:3344  #本微服务启动后先去找3344号服务，通过SpringCloudConfig获取GitHub的服务地址
```

下面是我们的 springboot 的配置文件：

```yaml
spring:
  application:
    name: microservicecloud-config-client
```