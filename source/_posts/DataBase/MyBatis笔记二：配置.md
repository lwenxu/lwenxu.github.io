---
title: MyBatis笔记二：配置
date: 2018-05-30 21:02:49
categories: MyBatis
tags:
- MyBatis
- 数据库
- Java
---

# MyBatis笔记二：配置

## 1.全局配置

### 1.properites

这个配置主要是引入我们的 properites 配置文件的：

```xml
<properties resource="db.properties"/>

<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="${driver}"/>
            <property name="url" value="${url}"/>
            <property name="username" value="${username}"/>
            <property name="password" value="${password}"/>
        </dataSource>
    </environment>
</environments>
```

可以看到我们使用  `<properties resource="db.properties"/>` 引入了我们的数据据库的配置文件，然后这个标签有两个属性 ： `resource` 和 `uri` 第一种直接是引用项目下的文件。第二个就是引用网络路径的和我们本地文件系统的资源。<!--more-->

### 2.settings

**非常重要**！！！

```xml
<!--全局配置-->
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

装了Mybatis 插件的话我们会看到我们的设置的代码提示，都不用自己去记的。

### 3.typeAliases 

可以为我们的 Java 类型取别名。避免我们去写很长的类包名，等等。并且这里提供了三种取别名的方式：

#### 1. typeAlias

```xml
<typeAliases>
    <typeAlias type="lwen.entries.Employee" alias="emp"/>
</typeAliases>
    
```

这就是给我们的 Java 类取的别名，我们在 xml 中配置返回值，参数，命名空间的时候就不用写那么长了。我们直接写 `emp` 即可，但是注意的是我们如果不写 `alias` 属性他就会配置默认的别名，也就是我们的类名首字母小写。`在这里就是employee`

#### 2.package

批量取别名，有时候我们的一个包下面的类太多了我们希望给他们都取上默认别名，我们就可以使用这个标签，但是注意这个标签不能和 **typeAlias标签共存** ，这个标签指定的包其实是对我们的**这个包以及他的子包**进行别名操作，并且都是**默认别名**

```xml
<typeAliases>
    <package name="lwen.entries"/>
</typeAliases>
```

#### 3.@Alias

因为上面的两个标签不能同时存在，所以我们没办法给某一个包下的特定的类取别名，这里我们就需要使用 `@Alias` 来做注解别名了，这样可以解决上面的问题。

```java
@Alias("emps")
```

其实除了这些我们需要自定义的一些别名，**系统帮我们预先设定好了很多常用的别名**：

| 别名       | 映射的类型 |
| ---------- | ---------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |



**可以看到规律就是类名小写，然后基本类型就是下划线。**

### 4.typeHandler

这个东西其实就是把我们的Java类型和数据库的类型相对应，这里暂时不具体说。

### 5.plugins

插件功能，对**下面对象的方法**进行拦截，他的原理就是动态代理。

- Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
- ParameterHandler (getParameterObject, setParameters)
- ResultSetHandler (handleResultSets, handleOutputParameters)
- StatementHandler (prepare, parameterize, batch, update, query)

### 6.environments

```xml
<environments default="dev">
    <environment id="dev">
        <transactionManager type="JDBC">
            <property name="hah" value="heh"/>
        </transactionManager>
        <dataSource type="POOLED">
            <property name="password" value="${password}"/>
        </dataSource>
    </environment>
</environments>
```

最外层就是我们的 environments 这个就是配置各种环境（比如：开发，测试，线上...），所以说我们的这个标签有一个 `default` 属性，就是用来制定我们的具体激活哪个环境的，这里用的是 dev 。然后下面就是具体的环境了，环境的 id 就是我们配置的环境的名称，每一个环境里有且只有两个属性，就是 `transactionManager` 和 `dataSource` 他们必须配置否则会报错。

可以看到 `transactionManager` 有一个 Type 就是用来指定使用哪个食物管理器，这里它使用了 JDBC 的，其实在Mybatis中只有两种：`type=”[JDBC|MANAGED]”`  

- JDBC – 这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域。 
- MANAGED – 这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。例如: 

```xml
<transactionManager type="MANAGED">
  <property name="closeConnection" value="false"/>
</transactionManager>
```

然后我们就很好奇这些 `MANAGED` 之类的东西在哪有定义，我们是否可以配置自己的书屋管理器比如强大的 Spring的事务管理。

`org.apache.ibatis.session.Configuration` 在这个类里我们发现了

```java
typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);

typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);
typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);
```

也就是说他们都是别名而已。那么我们就可以配置自己的类了，**我们直接在 type 位置写上我们的事务管理器全类名**，或者使用别名机制也可以。具体的对应的类需要什么特性我们直接看看他本来自带的两个类就明白了。

显然下面的配置数据源也是如此，默认的采用了连接池，也就是我们的 sqlSession 对象会被缓存起来不用每次去数据库里获取。

### 7.databaseIdProvider

MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持是基于映射语句中的 `databaseId` 属性。 MyBatis 会加载不带 `databaseId` 属性和带有匹配当前数据库 `databaseId` 属性的所有语句。 如果同时找到带有 `databaseId` 和不带 `databaseId` 的相同语句，则后者会被舍弃。  

我们通过设置属性别名来使其变短 ：

```xml
<databaseIdProvider type="DB_VENDOR">
  <property name="SQL Server" value="sqlserver"/>
  <property name="DB2" value="db2"/>        
  <property name="Oracle" value="oracle" />
  <property name="MySQL" value="mysql" />
</databaseIdProvider>
```

然后我们在 mapper 的xml文件中就可以匹配这些数据提供商：

```xml
<select id="getEmployeeById" resultType="lwen.entries.Employee" databaseId="mysql">
    select * from employee where id=#{id}
</select>
<select id="getEmployeeById" resultType="lwen.entries.Employee" databaseId="oracle">
    select * from employee where id=#{id}
</select>
```

那么他会按照数据源来确定当前是哪个数据源，我们需要使用哪个sql语句。这些都是自动进行的，无需我们的干预。

### 8.mappers

这个就是用来配置我们的 mapper 的 xml 标签了。我们在里面配置 xml 有以下三种方式：

#### 1.使用 mapper 标签

```xml
<!--我们的mapper文件的位置-->
<mappers>
    <mapper resource="EmployeeMapper.xml"/>
    <mapper resource="EmployeeMapperInterface.xml"/>
</mappers>
```

显然这个地方的 mapper 标签还有两个属性 分别就是 `resource` 和 `uri` 就是和上面是一样的。

#### 2.使用包扫描的方式

```xml
<package name="lwen"/>
```

lwen 包下面的xml 映射文件都被加载进去。

#### 3.注解

我们可以使用对应的注解 `注解名` 就是我们的 sql 语句的动作。 `@Select` `Update` 等等

**注意以上的标签都是有顺序的，顺序不能随便配置**

