---
title: JavaWeb 基础
date: 2020-02-28 16:00:52
tags: JavaWeb
categories:
  -Java Web
---

## Spring

### bean 属性注入方式

 1. 通过 setter 方法，直接使用 property 标签；p 标签，1. 导入 p 空间 2. 使用 p:属性 = 'xx'

     1. 使用 property 是会进行自动 的类型转换，起作用的就是 PropertyExtend。

     2. 如果要对属性赋值为 null 的话，我们不能使用 name ,value ，必须将类型写在标签体里面

        <property name = ""> <null/></property>

    3. 如果注入的是一个对象的话那么需要用 ref 引用
    4. 如果我们需要注入一个新对象，而不是 ioc 中原有的那么我们就在 property 标签中写一个 bean 标签，然后不需要 id 就可以，注意内部 bean 不会被放到 ioc 中
    5. list ，在 porperty 中写 list 标签，然后里面写对应的元素即可，如果是对象就用 bean，或者使用 ref 标签引用外部对象。
    6. map ，使用 map 标签，然后里面写 entry 标签 key - > value / value-ref ,同样的如果不写 value 可以在 entry 中写一个 bean 作为内部 bean 引用
    7. value 标签其实就是基本类型了
    8. props 标签就是对应了 java 中 property 对象，里面使用 prop 标签
    9. 使用 util 命名空间，帮我们创建一些集合，而且被放到 ioc 中，方便其他的地方可以引用，比如 \<util:map\> \<util:list\>  
    10. 级联属性（属性的属性的属性 ...） \<property name = 'a.b.c' value = 'xx'> 注意级联属性的时候可以修改 ioc 中的 bean 的属性
    11. 属性继承，如果创建两个同样 class 下的 bean 我们可以通过 bean 的 parent 属性来继承一个 bean 的属性值，父 bean 如果设置了 abstract=true 那么这个 bean 只能被继承不能从 ioc 中获取
    12. 
    13. 

 2. 通过构造器，使用 constructor-arg 标签  name , value ，如果不指定 name那么按照顺序

### 依赖注入

1. dependce-on = "a,b,c" 这个并不能说谁依赖谁 而是只能改变 bean 的创建顺序

### bean 的作用域

1. prototype 多实例，不在容器启动的时候创建而是在获取的时候被创建
2. singleton 单利，在容器启动之前就创建好，任何时候获取都是获取创建好的
3. request 每次请求的时候创建
4. session 同一个会话一个 bean

### 工厂创建 bean：

1. 静态工厂，不需要创建工厂对象，直接调用静态方法
   - 在创建 bean 的时候 class 写工厂的全类名，然后 factory-method 写对应的静态的方法名
2. 实例工厂，创建一个工厂对象，然后调用对应的方法
   - 先配置实例工厂，就是配置一个工厂 bean 也就是说的 factory-bean 
   - 然后在配置我们需要的 bean，class 写我们目标 bean 的 class，然后需要配置 factory-bean 和 factory-method 两个属性
   - 如果们继承了spring 中的 FactoryBean 接口 然后我们在配置 class 的时候配置这个 类，也不需要配置其他的一些东西直接出来就是我们需要的 bean，spring 就默认自动的调用工厂方法创建实例
   - 但是注意实现了 factorybean 接口的工厂不管你是单实例还是多实例他都是不在启动时创建而是在调用时创建

### 生命周期方法

1. init-method ，destory-method  分别是容器启动的时候和容器关闭的时候调用 这是单实例的情况，注意初始化方法是在创建实例之后才调用的，也就是 new 之后，但是此时 bean 并没有完全创建出来因为还没有走完生命周期
2. 如果是多实例的时候在获取的时候回调用 init 永远不会调用 destory 需要自己管理生命周期。

### bean 后置处理器

1. 需要实现 BeanPostProcessor 然后实现两个方法，分别是在 init 之前和之后调用的两个方法
2. 注意实现了 beanPostProcessor 的类需要被放到 ioc 容器中

### 引用外部属性文件 properties 文件

1. 使用 context:property-placeholder 标签,使用 location 指定属性文件的位置
2. 使用 ${xx} 获取属性
3. **注意：不要使用 username 这个 key，因为这个是 spring 中的一个关键字，他是系统用户名**
4. **注意：在使用 ${xx} 时候前后左右不要有空格，如果的话会将这个变量连接一个空格**

### 基于 xml 的自动装配

1. 在 bean 标签上写 autowire = 'xx' , 具体类型有 ：
2. byName [按照 id 和属性名进行匹配]
3. byType 按照类型，如果发现有多个那么会抛异常
4. default 不进行自动装配
5. constructor 按照构造器进行赋值，如果按照 constructor 进行装配有一个匹配过程，首先使用类型匹配，如果匹配的时候有多个结果不会抛异常而是通过 id 进行装配

### SPEL

1. 使用 #{12*12} 做计算 
2. 使用 #{aa.name+" "} 获取属性做拼接
3. 调用静态方法 #{T(全类名).静态方法名(参数)}
4. 调用非静态方法  #{obj.method(params)}

### 包扫描

1. 使用 \<compont-scan base-package = '' >
2. 但是如果需要使用这种注解扫描必须使用 aop 包不然会报错
3. 过滤包扫描，对于一些包不进行扫描 \<context:exclude-filter type='xx' expression='xx'>
   1. type  可以是注解，annotation，assignable 全类名 (前面两个常用，后面基本不使用)，aspectj j 表达式，custom 自己写代码，regex正则
4. 只扫描 \<context:inculde-filter type='xx' expression='xx'>
5. 上面两个必须要写到 component-scan 标签里面



## classpath 变量



classpath 和 classpath* 区别：

1.  classpath：只会到你的class路径中查找找文件;
2. classpath*：不仅包含class路径，还包括jar文件中(class路径)进行查找.

src路径下的文件在编译后会放到WEB-INF/classes路径下。默认的classpath是在这里。直接放到WEB-INF下的话，是不在classpath下的。





