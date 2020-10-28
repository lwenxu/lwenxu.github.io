---
title: MyBatis笔记四：动态SQL
date: 2018-06-2 22:01:49
categories: MyBatis
tags:
- MyBatis
- 数据库
- Java
---

# MyBatis笔记四：动态SQL

什么是动态SQL？ 简单来说就是类似于OGNL 表达式的这种 SQL 标签的嵌套然后帮助我们生成SQL语句而避免另外我们的拼字符串的操作。

### 1.if标签

if 标签中的 test 属性就是用来测试条件的，然后里面的条件之间可以采用 and or来连接，当然我们也可以使用 && 这种，但是注意我们只能使用它们的实体符号而不能直接使用 && 这种。

```java
@Test
public void DynamicSelectIf() throws IOException {
    SqlSessionFactory sessionFactory = GettingStart.getSessionFactory("mybatis.xml");
    SqlSession sqlSession = null;
    try {
        sqlSession = sessionFactory.openSession(true);
        EmployeeDynamicMapper mapper = sqlSession.getMapper(EmployeeDynamicMapper.class);
        Employee emp = mapper.getEmp(new Employee(13, "lwen", 12));
        System.out.println(emp);

    }finally {
        if (sqlSession != null) {
            sqlSession.close();
        }
    }
}
```

```xml
<select id="getEmp" resultType="lwen.entries.Employee">
    select *
    from employee where
    <if test="id!=null">
        id=#{id}
    </if>
    <if test="name!=null">
        and name=#{name}
    </if>
</select>
```

但是注意的一点就是假如我们没有传入我们的 id 字段的话，我们的sql就会报错，因为拼接会出问题啊。那么出来的 sql 就会是：

```sql
 select * from employee where and name=#{name}
```

显然会报错！

**解决方案1：**

```xml
<select id="getEmp" resultType="lwen.entries.Employee">
    select *
    from employee where
    <if test="1=1">
        1=1
    </if>
    <if test="id!=null">
        and id=#{id}
    </if>
    <if test="name!=null">
        and name=#{name}
    </if>
</select>
```

**解决方案2：**

### 2.where标签

```xml
<select id="getEmpWhere" resultType="lwen.entries.Employee">
    select *
    from employee
    <where>
        <if test="id!=null">
            id=#{id}
        </if>
        <if test="name!=null">
            and name=#{name}
        </if>
    </where>
</select>
```

可以看到我们删除了 where 关键字而是加上了 where标签这样的话虽然我们不传 id 我们的 sql 也是拼装正常的，不会报错。注意的一点就是我们的 where 标签只能解决当我们的 条件前面多出来的 and 或者 or 而不能解决后面的 and 比如说我们的 sql写成了：

```xml
<select id="getEmpWhere" resultType="lwen.entries.Employee">
    select *
    from employee
    <where>
        <if test="id!=null">
            id=#{id} and
        </if>
        <if test="name!=null">
            name=#{name}
        </if>
    </where>
</select>
```

如果不传 name 的话就会报错。

可以看到我们的sql 语句是这样的：

```
EmployeeDynamicMapper.getEmpWhere1 - ==>  Preparing: select * from employee where id=? and ; 
```

### 3.trim标签

trim标签就是可以给一个语句加上一个前缀一个后缀，删除某个前缀删除某个后缀。

- prefix：加前缀
- prefixOverrides：删前缀
- suffix：加后缀
- suffixOverrides：删后缀

``` xml
<select id="getEmpWhere1" resultType="lwen.entries.Employee">
    select *
    from employee
    <trim prefix="where" suffixOverrides="and">
        <if test="id!=null">
            id=#{id} and
        </if>
        <if test="name!=null">
            name=#{name}
        </if>
    </trim>;
</select>
```

解决上述问题呢。

### 4.choose标签

这个标签的功能就类似于带有 break 的 switch-case 语句，就是只会进入一个分支。

``` xml
<select id="getEmpWhere2" resultType="lwen.entries.Employee">
    select *
    from employee
    <where>
        <choose>
            <when test="id!=null">
                id=#{id}
            </when>
            <when test="name!=null">
                name like #{name}
            </when>
            <otherwise>
                age=1
            </otherwise>
        </choose>
    </where>
</select>
```

可以看到这里的功能说白了就是根据我们传入了哪个属性就用哪个属性查询，如果啥都没有的话就使用我们默认的就好。

### 5.set标签

我们上面都是在讨论关于选择的 条件判断问题，但是如果我们希望是更新也是条件进行的呢？我们这里就有与 where 对应的标签来保证我们的逗号不会多出来。

```xml
<update id="updateEmp">
    update employee
        <set>
            <if test="name!=null">
                name=#{name},
            </if>
            <if test="age!=null">
                age=#{age}
            </if>
        </set>
</update>
```

**同样的既然我们上面可以使用 trim 做一个通用的查询，那我们肯定可以使用 trim 做一个更通用的更新**

### 6.foreach标签

```xml
<select id="getEmpIn" resultType="lwen.entries.Employee">
    select *
    from employee where id in
    <foreach collection="ids" item="id" separator="," open="(" close=")" index="index">
        #{id}
    </foreach>
</select>
```

主要用于 in 这种枚举类型的。自然的我们也是**可以批量保存的** 就是采用这种方式，也就是  values 的位置数据很多。

### 7.内置参数

- _databaseId  这个表示的就是 databaseProvider 也就是我们配置的数据源厂商的名字了
- _parameter   这个表示的就是我们的方法的入参，如果单个参数就是它本身，如果是一个对象什么的就是一个 map

### 8.变量声明，绑定

我们的某一个参数过来的值我们没办法修改，或者说给他加一些修饰，修改。比如我们的某个like查询我们希望给传过来的东西加上 % 那么我们没办法直接在 sql 中添加，这时候我们就可以使用一个新变量，然后再新变量里修饰原来的变量。

```xml
<bind name="_new" value="'%'+name"/>
```

### 9.sql抽取

```xml
<sql id="select">
    select name,id,${other}
    from employee;
</sql>

<select id="ha">
    <include refid="select">
        <property name="other" value="age"/>
    </include>
    where id=#{id}
</select>
```

我们可以使用 sql 标签抽取 sql模板，然后使用 include 标签应用模板，之所以称为模板说的是他们是可以被重用的并且里面的内容是可变的。我们可以在 include 中使用 property 标签向模板中注入我们的定义的变量，只不过要注意的是变量的名字必须是采用 ${} 不能是 #{}