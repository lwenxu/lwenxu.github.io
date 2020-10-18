---
title: Hibernate笔记(一)
date: 2017-08-21 16:00:52
tags: Hibernate
categories:
  -Java Web
---
### 1.导入jar包
jar包主要需要导入的有两个文件夹下面的jar包
* required  必须要导入的核心包
* jpa 实体规范的包
### 2.写一个实体类
实体类必须要有一个唯一的id值，对应于表中的主键。其他的就是字段值，字段值不一定和数据库的字段一样，而是可以不一样，然后在配置文件中进行映射。
然后生成对应的get与set方法。
### 3.写映射配置文件
一般位置就是放在该实体类的位置，名字后缀最好是hbm.xml,然后导入dtd的约束。这个约束是mapping的约束注意不要导错了。  
```xml
<class name="类路径" table="表名">
  <id name="实体字段" column="表字段">
    <!-- 主键的生成策略，常用的就是native自增长，还有就是uuid使用128位的uuid算法产生uuid -->
    <generter>native</gengrter>
  </id>
  <property name="" column=""/>
  <property name="" column=""/>
  <property name="" column=""/>
</class>
```
### 4. 写Hibernate的核心配置文件
```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!--DB base config-->
        <property name="connection.url"/>
        <property name="connection.driver_class"/>
        <property name="connection.username"/>
        <property name="connection.password"/>

         <!--DB schema will be updated if needed-->
        <property name="hbm2ddl.auto">update</property>
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>

        <!--DB Mapping-->
        <mapping resource=""/>
    </session-factory>
</hibernate-configuration>
```
### 4.Hibernate代码
```java
public class Main {
    private static final SessionFactory ourSessionFactory;
    //由于创建sessionFactory的时候他会去创建表，非常耗资源，所以一个项目之创建一个SessionFactory
    //所以我们写一个工具类，在静态代码快中，只被加载一次
    static {
        try {
            Configuration configuration = new Configuration();
            configuration.configure();
            ourSessionFactory = configuration.buildSessionFactory();
        } catch (Throwable ex) {
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static Session getSession() throws HibernateException {
        return ourSessionFactory.openSession();
    }

    public static void main(final String[] args) throws Exception {
      //session是单例的，不可多线程公用，也就是一个session只能被一个线程使用。我们可以使用threadLocal来把session和本地线程绑定。
        final Session session = getSession();
        try {
            System.out.println("querying all the managed entities...");
            final Map metadataMap = session.getSessionFactory().getAllClassMetadata();
            for (Object key : metadataMap.keySet()) {
                final ClassMetadata classMetadata = (ClassMetadata) metadataMap.get(key);
                final String entityName = classMetadata.getEntityName();
                final Query query = session.createQuery("from " + entityName);
                for (Object o : query.list()) {
                    System.out.println("  " + o);
                }
            }
        } finally {
            session.close();
        }
    }
}
```

### 5.实体类编写规范
* 属性为私有的
* 必须有一个唯一的值一般都是id或者uid
* 属性一般都是使用包装类型，不要使用基本数据类型，就是防止出现null类型的。

### 6.CRUD操作
#### 1.增加
首先需要一个对象，这个也就是实体类的对象，然后设置字段的值，最后只需要调用session.save(user)即可插入一条数据。
#### 2.查询
根据id查询，直接使用session.get(User.class,id) 第一个参数就是类，第二个是id值。最后返回的不是数组或者list什么的，而是一个实体类的对象。
#### 3.修改
首先就需要进行查询，然后修改实体的属性，最后再使用update方法保存即可。save方法也可以，但是如果记录不存在他就会添加。
#### 4.删除
还是先查到该对象，然后在调用delete方法。

### 7.一级缓存
Hibernate的一级缓存默认是开启的，他的范围是在session开启导session关闭之间的，一级缓存的存放的对象只能就是持久太的数据。  
一级缓存一个简单的例子就是，我们第一次查询一条数据的时候这条数据就被被放在内存中，而第二次再次查询此数据的时候他不会取数据库获取数据，而是直接从内存中取到这条数据。
Hibernate也是有耳机缓存的，但是现在基本不再使用，而是通过一些NoSQL数据库来完成，存放类似于SessionFactory对象之类的东西。

### 8.session和本地线程绑定
因为session是单例的不可以多个线程同时访问，我们可以使用threadLocal来绑定session和Thread，但是这样比较麻烦，我们Hibernate已经提供了session和thread绑定的方法。  
首先需要配置核心配置文件
<property name="hibernate.current_session_context_class">thread</property>
然后SessionFactory.getCurrentSession()就是获得的与本地线程绑定的线程。线程结束以后session自动被销毁，不需要我们自己关闭session，否则会报错。

### 9.Hibernate三个查询对象
* Query  执行hql语句，调用对应的方法获取结果
```java
Query query=session.createQuery("hql");  //执行hql
query.list();  //获取一个list集合
```
* Criteria  无需执行hql，直接调用方法获取结果
```java
Criteria criteria=session.createCriteria(User.class);  //实体类的class
criteria.list();
```
* SQLQuery  调用底层sql
```java
SQLQuery sql=session.createSQLQuery("sql");//执行sql
sql.list() //每一条记录是一个数组
```
