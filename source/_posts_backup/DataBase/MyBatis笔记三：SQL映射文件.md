---
title: MyBatis笔记三：SQL映射文件
date: 2018-05-31 19:56:49
categories: MyBatis
tags:
- MyBatis
- 数据库
- Java
---

# MyBatis笔记三：SQL映射文件

## 1.简单的CRUD

### 1.绑定

首先呢我们还是需要把我们的 Mapper 接口和我们的 mapper xml 进行绑定，绑定的方式就是采用 `namespace` 了。不具体多说了。

### 2.接口

接着就是写我们的 Mapper 接口了，写Mapper接口的时候有一些注意事项，就是我们的 `插删改` 操作是可以有返回值的，默认情况下这些都是返回我们影响的行数，但是这里我们可以返回 `Integer` , `Long` , `Boolean` 类型，前两个好理解，就是我们常见的那种行数。然后后面具体就是布尔值是我们的行数大于0 就返回真。当然我们也可以不返回：

```java
package lwen.dao;

import lwen.entries.Employee;

public interface EmployeeMapper {
    Employee getEmployeeById(Integer id);

    boolean addEmpl(Employee employee);

    boolean updateEmpl(Employee employee);

    boolean deleteEmpl(Integer id);
}
```

### 3.mapper xml

然后就是在 mapper 的 xml文件中写 sql 语句了，具体就是几个 `动作标签` ，然后里面配置上我们的 sql ，至于我们的 sql 的参数我们采用了 `#{..}` 的方式，然后就是我们需要注意的一点就是我们的 sql 标签的 select 标签是含有一个返回值类型的，但是其他的标签是没有的，需要注意一下。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC " -//mybatis.org//DTD Mapper 3.e//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="lwen.dao.EmployeeMapper">
    <select id="selectOneEmployee" resultType="lwen.entries.Employee">
      select * from employee where id=#{id}
    </select>

    <!--添加-->
    <insert id="addEmpl">
        insert into employee (name, age) values (#{name}, #{age})
    </insert>

    <!--修改-->
    <update id="updateEmpl">
        update employee
        set name = #{name}, age = #{age}
    </update>

    <!--删除-->
    <delete id="deleteEmpl">
        delete from employee where id=#{id}
    </delete>
</mapper>
```

### 4.测试类

```java
@Test
public void CURDTest() throws IOException {
    SqlSessionFactory sessionFactory = GettingStart.getSessionFactory("mybatis.xml");
    SqlSession sqlSession = null;
    try {
        sqlSession = sessionFactory.openSession(true);
        EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
        System.out.println(mapper.addEmpl(new Employee(1,"lwen",18)));
    }finally {
        if (sqlSession != null) {
            sqlSession.close();
        }
    }
}
```

测试类的地方我们也看到了一个比较奇怪的地方就是 ` sqlSession = sessionFactory.openSession(true);` 我们在获取 session 的时候加了一个参数，其实这个参数就是说我们的每一条sql是否为一个事物，或者说是不是一个自动提交的 sql 。如果我们不开启这个自动提交的话我们在执行完若干条sql以后我们需要手动的调用 sql 的 `sqlSession.commit();` 方法了。

## 2.获取自增主键

我们在 Mybatis 中需要将我们插入的值获取到，然后返回给我们传入的 JavaBean 的话我们就可以才用这个方式把自动增长的主键的值封装获取我们在 Service  层就可以获取到我们插入的数据的 id 。

```xml
<insert id="addEmpl" useGeneratedKeys="true" keyProperty="id">
    insert into employee (name, age) values (#{name}, #{age})
</insert>
```

`useGeneratedKeys` 这个属性说我们需要开启自动增长的策略并获取增长的值，然后我们需要把这个值封装进我们的 Java bean 的哪个属性就是我们的 `keyProperty` 来确定了。

```java
Employee employee = new Employee(1, "lwen", 18);
System.out.println(mapper.addEmpl(employee));
System.out.println(employee);
```

`Employee(id=13, name=lwen, age=18)` 虽然我们传入了 id=1 但是 bean 被修改成了 13。

## 3.参数处理

### 1.单个参数

这个时候我们直接采用 `#{}` 取出值，不做特殊处理，我们的 `#{}` 里面写什么都可以，不用和我们的方法参数对应。

### 2.多个参数

多个参数就设计到绑定的问题了，也就是要么我们进行参数的绑定，要么我们就进行参数的顺序处理。

#### 1.param顺序处理

```xml
<select id="getEmplByIdAndName" resultType="lwen.entries.Employee">
    select * from employee where id=#{param1} and name=#{param2}
</select>
```

这里我们的 Java 代码是有两个参数的，所以注意我们的这个地方参数是从 1 开始的不是 0.

#### 2.args顺序处理

```xml
<select id="getEmplByIdAndName2" resultType="lwen.entries.Employee">
    select * from employee where id=#{arg0} and name=#{arg1}
</select>
```

这里的参数有是从 0 开始的。

**注意：** 以上两种代码我们的 Mapper 接口不需要做任何的额外的配置，就可以直接可以工作了如下：

```java
Employee getEmplByIdAndName(Integer id, String name);
```

#### 3.命名参数

上面的代码虽然可以工作但是我们需要注意的就是这个地方我们的代码是没有任何的具体的意义的，因为就是我们看不出来这个变量的具体意义。我们就可以和我们的接口的参数绑定，那么就是用命名参数。

首先我们需要在Mapper接口中写相关的注解，确定参数：

```java
Employee getEmplByIdAndName1(@Param("id") Integer id,@Param("name") String name);
```

然后我们在 xml 中就可以应用这些参数了：

```xml
<select id="getEmplByIdAndName1" resultType="lwen.entries.Employee">
    select * from employee where id=#{id} and name=#{name}
</select>
```

当然采用命名参数肯定是最好的方式了。

### 3.传递pojo

如果我们的参数太多了我们使用命名参数就很麻烦，我们就可以自己构建 pojo 。但是如果这些属性之间没什么关系，然后我们不用自己创建一个没什么用的封装类，我们直接使用 map 就好了。然后我们在 sql 就可以直接使用 `#{key}` 就可以获取对应的 value 。但是如果这个参数经常被使用的话我们就可以自己封装一个类数据传输类 TO。

其实真正在 Mybatis 中他自动把我们的参数封装到了 map中所以我们这么写也是很自然的。

``` java
Employee getEmplByIdAndName3(Map<String, Object> map);
```

我们的xml 可以直接取出我们的map中的key

```xml
<select id="getEmplByIdAndName3" resultType="lwen.entries.Employee">
    select * from employee where id=#{id} and name=#{name}
</select>
```

```java
Map<String, Object> map = new HashMap();
map.put("id", 12);
map.put("name", "lwen");
Employee employee4 = mapper.getEmplByIdAndName3(map);
```

### 4.List/Array 特殊情况

- public Employee getEmp(@Param("id")Integer id,String lastName)；

  取值：id ： #{id/param1}  lastName ：#{param2}

- public Employee getEmp(Integer id,@Param("e")Employee emp)；

  取值：id： #{param1}   lastName：#{param2.lastName/e.lastName}

**特别注意**：

如果是Collection(List、Set)类型或者是数组，也会特殊处理。也是把传入的list或者数组封装在map中。

比如：public Employee getEmpById(List<Integer>ids)；

取值：取出第一个id的值：#{list[e]}

## 4.参数处理原理

我们从源码的角度来看看我们的 Mybatis 框架是如何处理我们的传入的参数，以及绑定的参数的。

### 1.代理对象

首先我们在获取到的 mapper 上打断点，然后 step into：

![image](https://user-images.githubusercontent.com/22151420/40818854-e5a7c90a-658b-11e8-93db-196954b83166.png)

接着我们就能来到代理类了：`org.apache.ibatis.binding.MapperProxy`

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  try {
    //放行Object对象的方法，不做代理
    if (Object.class.equals(method.getDeclaringClass())) {
      return method.invoke(this, args);
    } else if (isDefaultMethod(method)) {
      return invokeDefaultMethod(proxy, method, args);
    }
  } catch (Throwable t) {
    throw ExceptionUtil.unwrapThrowable(t);
  }
  final MapperMethod mapperMethod = cachedMapperMethod(method);
   //真正执行的 sqlSession
  return mapperMethod.execute(sqlSession, args);
}
```

可以看到我们在这里生成了对应方法的代理对象，也就是给我们的 Mapper 接口生成了代理对象，接着就用代理对象 `mapperMethod` 来调用我们的接口方法。

### 2.execute核心逻辑

我们单步进入 execute 方法可以看到主要的逻辑如下：

```java
switch (command.getType()) {
  case INSERT: {
    Object param = method.convertArgsToSqlCommandParam(args);
    result = rowCountResult(sqlSession.insert(command.getName(), param));
    break;
  }
  case UPDATE: {
    Object param = method.convertArgsToSqlCommandParam(args);
    result = rowCountResult(sqlSession.update(command.getName(), param));
    break;
  }
  case DELETE: {
    Object param = method.convertArgsToSqlCommandParam(args);
    result = rowCountResult(sqlSession.delete(command.getName(), param));
    break;
  }
  case SELECT:
    if (method.returnsVoid() && method.hasResultHandler()) {
      executeWithResultHandler(sqlSession, args);
      result = null;
    } else if (method.returnsMany()) {
      result = executeForMany(sqlSession, args);
    } else if (method.returnsMap()) {
      result = executeForMap(sqlSession, args);
    } else if (method.returnsCursor()) {
      result = executeForCursor(sqlSession, args);
    } else {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = sqlSession.selectOne(command.getName(), param);
    }
    break;
  case FLUSH:
    result = sqlSession.flushStatements();
    break;
  default:
    throw new BindingException("Unknown execution method for: " + command.getName());
}
```

也就是我们所有的采用Mapper 接口的方法开发的最后采用的代理生成代理对象以后我们的插删改操作还是调用了我们 sqlSession 底层的 Select、Delete 方法等等。

里面的套路就是先对我们的方法的参数进行转换，然后执行对应的动作方法，传入我们的参数。

### 3.参数转换

`convertArgsToSqlCommandParam（）` 主要完成了这个功能，这里我们就着重看看这个方法。

最后追溯到下面这个方法，代码补偿我就全部贴上来了：

```java
public Object getNamedParams(Object[] args) {
  final int paramCount = names.size();
  if (args == null || paramCount == 0) {
    return null;
  } else if (!hasParamAnnotation && paramCount == 1) {
    return args[names.firstKey()];
  } else {
    final Map<String, Object> param = new ParamMap<Object>();
    int i = 0;
    for (Map.Entry<Integer, String> entry : names.entrySet()) {
      param.put(entry.getValue(), args[entry.getKey()]);
      // add generic param names (param1, param2, ...)
      final String genericParamName = GENERIC_NAME_PREFIX + String.valueOf(i + 1);
      // ensure not to overwrite parameter named with @Param
      if (!names.containsValue(genericParamName)) {
        param.put(genericParamName, args[entry.getKey()]);
      }
      i++;
    }
    return param;
  }
}
```

我们看看这个方法的执行逻辑：

#### 1.names获取

首先就是一堆names的获取，这个names又是什么？其实这是当前类的一个属性，记录的东西是Mapper 接口的参数的名字他是在这个类的构造方法中初始化的，也就是在我们调用方法之前 names 就已经确定好了。

```java
// get names from @Param annotations
for (int paramIndex = 0; paramIndex < paramCount; paramIndex++) {
  if (isSpecialParameter(paramTypes[paramIndex])) {
    // skip special parameters
    continue;
  }
  String name = null;
  for (Annotation annotation : paramAnnotations[paramIndex]) {
    if (annotation instanceof Param) {
      hasParamAnnotation = true;
      name = ((Param) annotation).value();
      break;
    }
  }
  if (name == null) {
    // @Param was not specified.
    if (config.isUseActualParamName()) {
      name = getActualParamName(method, paramIndex);
    }
    if (name == null) {
      // use the parameter index as the name ("0", "1", ...)
      // gcode issue #71
      name = String.valueOf(map.size());
    }
  }
  map.put(paramIndex, name);
}
```

这就是 `ParamNameResolver` 的构造函数，里面最重要的逻辑就是确定我们的 names ：

1.获取 `@param` 注解标注的参数名，然后把这个注解的value值作为 names 的value，然后把参数的位置作为 key 。比如我们的方法是 `findAll(@Parma("name") String name,Integer age)` 最后生成的names这个map里面就是 {0->name,1->1} 至于为什么是 `1->1` 我们接下来再说

2.如果我们的 `@param` 不存在的话并且我们配置了 `isUseActualParamName` 我们会尝试采用 JDK1.8 的新特性也就是使用 -paramters 编译参数，通过反射直接获取到我们方法的参数名。

3.如果还是不行的话我们就使用参数的索引值，也就是 0,1 上面的注释也是特别清楚。

#### 2.方法没有参数

没有参数的时候就会直接返回null

#### 3.单参数

只有一个参数并且没有 `@param` 注解的时候，我们就直接获取names的第一个key也就是0

#### 4.多参数

这个地方有两部分，当有 `@Param` 注解的时候就是把 names 的 value 作为 key ，然后我们的真正的参数作为 value 。然后当然为了保险起见 Mybatis 还未每一个参数生成了一个 以 paramIndex 作为key 以值作为 value 的 也就是我们用的 param0 .. paramN .可是我们会说为啥我们还可以用 args0 和 args1 来取值呢？

这个就看上面的 names 获取值的时候我们可以采用编译器的反射机制获取，因为如果我们编译器不支持这个特性的话我们的参数就会被抹掉，用args0 args1 来代替。

## 5.取值规则

### 1.#{}与${}区别

\#{}：是以预编译的形式，将参数设置到sq1语句中；PreparedStatement；防止sq1注入

${}：取出的值直接拼装在sq1语句中；会有安全问题；
大多情况下，我们去参数的值都应该去使用#{}；
原生jdbc不支持占位符的地方我们就可以使用${}进行取值,比如分表；按照年份分表拆分
`select * from ${year}_salary where xxx；`

### 2.#{}注意事项

在#{} 中我们可以放入 javaType、jdbcType、mode(存储过程）、numericScale、
resultMap、typeHandler、jdbcTypeName、expression(未来准备支持的功能）；

比较重要的就是：jdbcType通常需要在某种特定的条件下被设置：
在我们数据为null的时候，有些数据库可能不能识别mybatis对nu11的默认处理。比如Oracle（报错），因为他们对null的映射是到了 Other 类型，然后就会导致JdbcType OTHER：无效的类型；因为mybatis对所有的nu11都映射的是原生Jdbc的OTHER类型，
由于全局配置中：jdbcTypeForNull=OTHER；oracle不支持；两种办法
1、`#{email,jdbcType=OTHER}；`
2、`<setting name="jdbcTypeForNull" value="NULL"/>`

## 6.查询

### 1.返回集合结果

返回集合结果的时候我们的 resultType 不能写成 list 、set 类型而是要写做我们的集合里面的元素的类型。

```xml
<select id="getLikeByName" resultType="lwen.entries.Employee">
    select *
    from employee
    where name like #{name};
</select>
```

### 2.返回map

也就是将我们的 bean 的属性作为 key 然后 值作为 value 来存储到一个 map 中，其实这个只存一条数据，那么我们的语句的返回类型就是我们的 map 他会自动的做封装。

```xml
<select id="getEmplReturenMap" resultType="map">
    select * from employee where id=#{id};
</select>
```

```java
Map<String, Object> getEmplReturenMap(Integer id);
```

### 3.返回多条记录的map

如果我们想让map的key是我们的某一条属性，然后value是我们的实体对象，那么我们的封装结果就必须是元素的类型，也就是我们的实体类的类型，但是我们的key则需要我们另行指定我们采用的方式就是使用一个注解在方法上。

``` java
@MapKey("name")
Map<String, Employee> getEmplReturnMaps(String name);
```

```xml
<select id="getEmplReturnMaps" resultType="lwen.entries.Employee">
    select *
    from employee where name like #{name};
</select>
```

### 4.返回自定义结果集

有时候对于Mybatis自带的一些默认封装的规则不能满足我们的需求的时候，我们可以采用 resultMap 自定义结果集。这里就演示一些自定义结果集，但是注意 resultMap 与 resultType 只能存在一个。

```xml
<resultMap id="MyEmp" type="lwen.entries.Employee">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="age" property="age"/>
</resultMap>

<select id="getEmplByResultMap" resultMap="MyEmp">
    select *
    from employee
    where id = #{id};
</select>
```

可以看到重点就是写我们的 resultMap 标签，然后再标签中我们自己封装规则。注意的一点就是如果说我们在 result 中没有封装 bean 中的其他属性他会自动帮我们封装，也就是我们可以把一些特殊的字段和我们的 bean 结合起来。其他的正常的自动封装。然后就是 id 字段我们在 resultMap 中指明以后Mybatis就会帮我们自动封装，并且做一些查询优化。

### 5.关联查询

#### 1.union

``` xml
<select id="getById" resultMap="EmpNew">
    select employee.id id,employee.name name,age,d_id did,department.name dname
    from employee,department where employee.id=#{id} and employee.d_id=department.id;
</select>
```

我们采用的联合查询，得到了关联查询的结果，这也是常用的套路，但是注意的一点就是我们的 result 的封装并不是使用的自带的封装规则而是采用了 我们自定义的 resultMap 因为我们的 employee 中有一个 department 的对象，我们无法直接封装，至少列名都没办法对应。

```xml
<resultMap id="EmpNew" type="lwen.entries.EmployeeNew">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="age" property="age"/>
    <result column="did" property="department.id"/>
    <result column="dname" property="department.name"/>
</resultMap>
```

#### 2.association关联

使用 association 标签可以给一个 javaBean 的内部引用创建关联关系，与上面的效果类似，但是方法不同。

```xml
<resultMap id="EmpNew1" type="lwen.entries.EmployeeNew">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="age" property="age"/>
    <association property="department" javaType="lwen.entries.Department">
        <id column="did" property="id"/>
        <result column="dname" property="name"/>
    </association>
</resultMap>
```

```xml
<select id="getByIdAssociation" resultMap="EmpNew1">
    select employee.id id,employee.name name,age,d_id did,department.name dname
    from employee,department where employee.id=#{id} and employee.d_id=department.id;
</select>
```
#### 3.association多步查询

当我们需要 department 的 id 然后作为我们封装 Employee 的条件的时候我们就需要用 id 查询这个 Department

```xml
<resultMap id="EmpNew2" type="EmployeeNew">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="age" property="age"/>
    <association property="department" select="lwen.dao.DepartmentMapper.getDepById" column="d_id">
    </association>
</resultMap>

<select id="getEmplStepByStep" resultMap="EmpNew2">
    select * from employee where id=#{id}
</select>

<select id="getDepById" resultType="lwen.entries.Department">
    select * from department where id=#{id}
</select>
```

可以看到我们在 `association` 中调用了  `lwen.dao.DepartmentMapper.getDepById`  这个方法其实就是只进行了一个简单的 按照 id 查询，然后我们传入的 column 就是作为我们查询 department 的参数。最后做关联。

#### 4.懒加载

需要我们在以前的分布查询的基础之上添加上一个配置项，这个配置项是在我们的全局配置文件中的。

```xml
<setting name="lazyloadingEnabled" value="true"/>
<setting name="aggressivelazyLoading" value="false"/>
```

配置这两项的时候我们在查询获取一个对象的时候我们只有在引用他们的值得时候才真的去加载这些东西，否则不会加载。

#### 5.一对多映射

```xml
<resultMap id="EmpNew3" type="Department">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <association property="employees" select="lwen.dao.EmployeeNewMapper.getEmplsByDeptId" column="id"/>
</resultMap>

<select id="getEmplStepByDepIdStep" resultMap="EmpNew3">
    select *
    from department where id=#{id};
</select>

<select id="getEmplsByDeptId" resultType="lwen.entries.Employee">
    select * from employee where d_id=#{did}
</select>
```