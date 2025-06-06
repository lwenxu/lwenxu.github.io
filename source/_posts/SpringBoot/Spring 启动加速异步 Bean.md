---
title: SpringBoot 启动加速
date: 2021-05-27 22:39:26
tags: Spring
categories:
  - SpringBoot

---

### 前言



在 2021 年这个小学作文中的未来年份，没有想象中的汽车满天飞，也没有实现机器人满地跑。但牛逼的是我们都有一个共识： 知乎达到了人均 “谢邀~ 人在美国刚下飞机”的生活水平，虎扑的人均收入也在 30W+ ，还有就是程序员都人均精通 SpringBoot ，哪怕和算法聊技术一言不合就满嘴 SpringCould 分布式、微服务，然而实际操作可能是 **分步试** 、**伪服务** ... 你一个小小系统开这么多应用启动不难受？(不难受因为可以装 13)   SpringBoot 这启动速度也确实令人捉急，每个应用 5 分钟改个小功能，编码五分钟部署俩小时。 so 如何更加优雅快速的  启动 SpringBoot 应用 (满足装 13 的欲望) 呢？





![IMG_3485.jpg](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1618727885214-dd6dfd2d-7c9f-4e06-9b68-f7ea14e1d360-20210627195044309.jpeg)









其实我们都比较清楚大部分的启动时间是由于 Spring 需要加载各种 Bean 导致启动速度下降的，那么对于那些不是特别重要的 Bean 我们是不是可以让他再起一个线程做 Bean 初始化，不阻塞主线程启动，这样启动速度不就起来了么，说干就干！



那你肯定会问，我们咋判断一个 Bean 加载时间长短，找出那些 `拉跨的Bean` 呢？内心一阵嘲讽，嘴角泛出一阵冷笑后 不慌不忙的道来：“这对于精通 Spring 框架的我来说不是小菜一碟么 ”。我掐指一算，就知道是哪几个 Bean 拉跨。

![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1619414062442-1cb5a1cc-3702-4fa8-937d-0dd75fb37ecc-20210627195048353.png)



### 统计 Bean 初始化时间 



简单做法就是通过 BeanPostProcessor ，祭出这个神器就能干倒一半的 LSP，很多使用 Spring 的 LSP 其实都不太知道这个玩意，那他到底是个什么玩意呢？简单来讲就是 Spring 初始化 Bean 的钩子函数。 Bean 是否加载，加载的是哪个 Bean 以及修改 Bean 的相关属性都可以通过这个钩子函数搞定。这就是一个活生生的 Spring 后门，学会了他 Spring 框架任你摆布。



后面我会专门开一篇文章去讲解 BeanPostProcessor  如何帮你加载 Bean，以及如何利用这个后门搞事情。



![unnamed.jpg](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1619265336642-31eb298f-f2d5-4025-b431-64262d48f1b7-20210627195051841.jpeg)





所以我们现在讲讲如何用 BeanPostProcessor  统计 Bean 的初始化时间。不多 BB 直接上代码：



![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1619266023651-dc9b9630-a24a-4847-9bbe-3278ca3203c5-20210627195055164.png)



我们编写了 BeanInitCostTimeBeanPostProcessor 继承了 BeanPostProcessor   实现了 postProcessBeforeInitialization 和 postProcessAfterInitialization 分别代表了 Bean 初始化之前和之后，简单记录了 Bean 初始化开始时间，然后用结束时间减去开始时间就得到了具体的初始化时间。



我专门在 Spring 容器中放了一个加载时间很长的测试 Bean ，代码如下



![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1619267993440-bc8b9a0d-0a58-443c-8bec-93f209ebbc46-20210627195101201.png)



效果如下：

![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1619267872243-88f9a69e-f3ea-446e-aca7-151469cd7a7e-20210627195104385.png)





### 异步 Bean 实操



异步 Bean 异步 Bean ，首先这个 Bean 需要是个异步的，那如何将一个 Bean 包装成异步的呢？首先异步 Bean 肯定需要继承原来的 Bean 需要保证是同一种类型的，然后重写他的 init 方法。代码操作如下：



![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1619270159828-93a6a66e-2d1b-4095-9b36-b2d9a34f6c9a-20210627195107701.png)



但是这么写并不是很优雅，假如我们还需要在初始化 Bean 的时候做一些其他操作，直接继承是不方便的。所以必须祭出 Spring 中的另外一个组件了，工厂 Bean，学名 FactoryBean 这个要和另外一个叫做 BeanFactory 的大佬区分开。 工厂 Bean 顾名思义就是这个玩意也是一个 Bean 只是他的主要职责是作为工厂，通常用于生成特定类型的 Bean ，那么我们就有一个骚操作了，让这个工厂生产的 Bean 就是它自己，而且中间还能定义很多额外的逻辑，不多 BB 上代码



![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1619270390154-8e7ca014-acab-4e52-a16a-df28fd77ae02-20210627195111231.png)



然后通过异步 Bean 去加载我们看看前后的差别



![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1619270486905-7201f393-7ac0-4935-bb8c-f44115429fb1-20210627195114682.png)



优化之前：

![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1619270568290-278de048-c70a-4406-a7a6-5c1f37a4e85b-20210627195118350.png)



优化之后：

![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1619270528731-9322ec40-94ed-4225-b74b-fcf9d6975189-20210627195120945.png)



可以看到优化后 1.4s 应用就启动完成了，而且耗时的 Bean 可以让他慢慢加载，在应用启动完成后这个 Bean 才完全被加载。



### 唠叨



最近有点忙成狗，从四月就断更到现在了，今天抽个周末的时候终于写完了。



不过最近看了好几本书，也准备抽点时间写写读后感了，非常推荐大家看看 《零秒思考》 ，同时写点个人的小思考： 一旦适应了用语言表达脑海中的意向和感觉，渐渐地就不会为如何表达自己的心情和想法而苦恼，流畅清晰的表达能够更快的与对方沟通让对方心领神会。表达不善往往便会产生一种想也是白想的放弃心理，由此甚至连思考本身也放弃了，不思考人就不会成长，不经历思考解决手头的事情人就会失去干劲变得无聊。

另外：大家也可以关注下我的微信公众号哦~ 技术分享和个人思考都会第一时间同步！

![image.png](https://lwen-pic.oss-cn-beijing.aliyuncs.com/1610289331606-52e03a47-f431-4b20-8b57-6a4cc0f70345-20210627195142247.png)