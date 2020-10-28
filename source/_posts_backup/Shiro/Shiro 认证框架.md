---
title: Shiro安全框架:认证
date: 2018-04-13 21:01:28
tags: Shiro
categories:
 - Shiro
---

# Shiro 安全框架

## 1. 认证

### 1. 采用简单的对象登陆认证(SimpleAccountRealm)

```java
public class AuthenticationTest {
    // 创建一个简单的认证 realm 也就是认证信息存放在对象中的
    SimpleAccountRealm simpleAccountRealm = new SimpleAccountRealm();

    @Before
    public void addUser(){
        simpleAccountRealm.addAccount("lwen", "1234");
    }
    @Test
    public void authentication(){
        // 创建一个 securityManager 环境
        SecurityManager securityManager = new DefaultSecurityManager(simpleAccountRealm);
        // 设置环境
        SecurityUtils.setSecurityManager(securityManager);
        // 获取认证主体
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken("lwen", "1234");
        subject.login(token);
        System.out.println(subject.isAuthenticated());

    }
}
```

<!--more-->

### 2. 简单的角色认证

```java
public class AuthorizationTest {
    // 创建一个简单的认证 realm 也就是认证信息存放在对象中的
    SimpleAccountRealm simpleAccountRealm = new SimpleAccountRealm();

    @Before
    public void addUser(){
        simpleAccountRealm.addAccount("lwen", "1234","user","admin");
    }
    @Test
    public void authentication(){
        // 创建一个 securityManager 环境
        SecurityManager securityManager = new DefaultSecurityManager(simpleAccountRealm);
        // 设置环境
        SecurityUtils.setSecurityManager(securityManager);
        // 获取认证主体
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken("lwen", "1234");
        subject.login(token);
        subject.checkRole("user");
        subject.checkRoles("user", "admin");
    }
```

## 2. 内置 Realm 和自定义 Realm

### 1. 配置文件 (initRealm)

创建一个配置文件，如下

```Ini
[users]
lwen=1234,admin
mark=1234,user
[roles]
admin=update,delete,select
user=select
```

在代码中使用 IniRealm 即可。可用下面的代码检查用户权限。

```java
subject.checkPermission("update");
subject.checkPermission("select");
```

### 2. 数据库(JdbcRealm)

首先需要创建表，这里我们采用了默认的查询方式，但是这种查询方式是不可用的，这里只做演示用。后面会说到关于自定义 Realm ，会详细说数据库的设计。

这里我们采用了 Jdbc 默认的 sql 语句建表。

![default](https://user-images.githubusercontent.com/22151420/39239770-8314f4dc-48b4-11e8-9293-112ce08f712e.png)

```java
public class AuthenticationJdbcRealmTest {
    // 创建一个简单的认证 realm 也就是认证信息存放在对象中的
    JdbcRealm jdbcRealm = new JdbcRealm();

    @Test
    public void authentication(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl("jdbc:mysql://127.0.0.1/shiro");
        dataSource.setUsername("root");
        dataSource.setPassword("12345678");
        jdbcRealm.setDataSource(dataSource);
        jdbcRealm.setPermissionsLookupEnabled(true);//这是一个坑 ，默认权限表不进行查找
        // 创建一个 securityManager 环境
        SecurityManager securityManager = new DefaultSecurityManager(jdbcRealm);
        // 设置环境
        SecurityUtils.setSecurityManager(securityManager);
        // 获取认证主体
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken("lwen", "1234");
        subject.login(token);
        System.out.println(subject.isAuthenticated());
        subject.checkPermission("update");
        subject.checkPermission("select");
    }
}
```

注意这里有一个坑，就是默认的情况下 Jdbc 不回去查询权限表，导致失败。所以我们需要添加上 `setPermissionsLookupEnabled` 让他开启。

### 3. 自定义 Realm

自定义 Realm 就是说我们只要继承一个 `AuthorizingRealm` 类，然后重写里面的认证和授权的方法就可以了，非常的简单。一个是认证的方法也就是 `doGetAuthenticationInfo` 另外一个就是 授权的方法 `doGetAuthorizationInfo` ，里很多操作这里采用了一个静态数据表示的，实际上这些是需要从数据库或者缓存中取出来然后进行逻辑判断。

```java
public class CustomRealm extends AuthorizingRealm {
    {
        this.setName("customRealm");
    }

    // 授权  -> 权限 角色
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String userName = (String) principalCollection.getPrimaryPrincipal();
        // 这个需要使用 service 层然后调用 dao 层去数据库或者缓存中查询，然后返回结果这里为了方便采用静态数据
        Set<String> roles = getRolesByUsername(userName);
        Set<String> permissions = getPermissionsByUsername(userName);
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo(roles);
        authorizationInfo.setStringPermissions(permissions);
        return authorizationInfo;
    }
    // 认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        String userName = (String) authenticationToken.getPrincipal();
        String password = getPasswordByUsername(userName);
        if (password != null) {
            return new SimpleAuthenticationInfo(userName, password,"customRealm");
        }
        return null;
    }

    private String getPasswordByUsername(String userName) {
        return "1234";
    }

    private Set<String> getPermissionsByUsername(String userName) {
        return new HashSet<>(Arrays.asList("select", "delete", "update"));
    }

    private Set<String> getRolesByUsername(String userName) {
        return new HashSet<>(Arrays.asList("admin"));
    }
}
```

### 4. 密码加密加盐

```java
HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
matcher.setHashAlgorithmName("md5");
customRealm.setCredentialsMatcher(matcher);
```

在代码逻辑中加上这段，然后在 Relam 中需要自定义盐。

```java
// 加盐（lwen）
authenticationInfo.setCredentialsSalt(ByteSource.Util.bytes("lwen"));
```

## 2. 整合 Spring

### 1. 首先配置过滤器

也就是类似于其他的登陆授权方式都是通过拦截器或者过滤器来完成登陆的，这里我们也是这样的思路。在 web.xml 中配置过滤器，过滤所有请求，交给 shiro 处理。

```xml
<!--shiro 配置-->
<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 2. 配置 beans 

以前我们手动创建的 securityManager 对象以及 Realm 和 加密函数对象现在我们都交给 Spring 管理，进行自动的依赖注入和管理。

```xml
<bean class="org.apache.shiro.spring.web.ShiroFilterFactoryBean" id="shiroFilter">
    <property name="securityManager" ref="securityManager"/>
    <property name="loginUrl" value="login.html"/>
    <property name="unauthorizedUrl" value="403.html"/>
    <property name="filterChainDefinitions">
        <value>
            /login.html=anon
            /subLogin=anon
            /*=authc
        </value>
    </property>
</bean>

<bean class="org.apache.shiro.web.mgt.DefaultWebSecurityManager" id="securityManager">
    <property name="realm" ref="customRealm"/>
</bean>

<bean class="realms.CustomRealm" id="customRealm">
    <property name="credentialsMatcher" ref="matcher"/>

</bean>
<bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher" id="matcher">
    <property name="hashAlgorithmName" value="md5"/>
    <property name="hashIterations" value="1"/>
</bean>
```

可以看到上面我们配置了登录页的 url 和未认证页，最后比较重要的就是我们是否需要认证的页面的配置。我们需要把登录页，和登录数据提交页给排除掉，因为如果有这些的页面的话我们将会进入死循环中，没办法进行登录操作。其中 anon 就是无需认证的，而后面 authc 需要认证，这个东西是按照顺序来的。所以 /* 放在最后面匹配。

### 3. 编写 controller 进行认证

```java
public class IndexController {
    @RequestMapping(value = "/subLogin", method = RequestMethod.POST,produces = "application/json;charset=utf-8")
    @ResponseBody
    public String login(User user){
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(user.getUsername(), user.getPassword());
        try {
            subject.login(token);
        } catch (AuthenticationException e) {
            return e.getMessage();
        }
        return "登陆成功！";
    }
}
```

认证没有抛出异常的话我们就算认证成功了，否则的话就是认证失败。

### 4. 角色和权限认证

首先需要导入 aop 的注解，然后我们进行授权操作的时候只需要进行写一个注解就行了，当然也可以直接编码实现。

导入注解 jar 配置注解springmvc 中：

```xml
<!--开启 aop-->
<aop:config proxy-target-class="true"/>
<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
    <property name="securityManager" ref="securityManager"/>
</bean>
<bean class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>
```

```Xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.11</version>
</dependency>
```

```java
@RequiresRoles("admin")
@GetMapping("/testRole")
@ResponseBody
public String testRole(){
    return "testRole";
}

@RequiresPermissions("select")
@GetMapping("/testPermission")
@ResponseBody
public String testPermission(){
    return "has";
}
```

## 3. 自动登录

要创建一个 cookie 对象，然后在自动登录管理器中注入 cookie ，接着需要传递给 securityManager。

```xml
<bean class="org.apache.shiro.web.mgt.CookieRememberMeManager" id="rememberMeManager">
    <property name="cookie" ref="cookie"/>
</bean>
<bean class="org.apache.shiro.web.servlet.SimpleCookie" id="cookie">
    <constructor-arg value="rememberMe"/>
    <property name="maxAge" value="200000"/>
</bean>
<bean class="org.apache.shiro.web.mgt.DefaultWebSecurityManager" id="securityManager">
	<property name="realm" ref="customRealm"/>
	<property name="rememberMeManager" ref="rememberMeManager"/>
</bean>	
```

接着只需要在 controller 的登陆方法中进行判断就行了。

```java
// 设置记住我
token.setRememberMe(user.isRememberMe());
```