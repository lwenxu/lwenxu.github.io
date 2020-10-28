---
title: SpringData笔记
date: 2018-12-17 21:01:28
tags: SpringBoot
categories:
 - SpringBoot
---

# SpringData 笔记

# 1. 配置项目

## 1.pom.xml

<!--more-->

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lwen</groupId>
    <artifactId>SpringData</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--MySQL Driver-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
        </dependency>

        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
        </dependency>

        <!--spring-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>4.3.5.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.5.RELEASE</version>
        </dependency>

        <!--spring data jpa-->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jpa</artifactId>
            <version>1.8.0.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>4.3.6.Final</version>
        </dependency>

    </dependencies>

</project>
```

## 2.bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
      http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa-1.3.xsd
      http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
      http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

    <!--1 配置数据源-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="username" value="root"/>
        <property name="password" value=""/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring_data"/>
    </bean>

    <!--2 配置EntityManagerFactory-->
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"/>
        </property>
        <property name="packagesToScan" value="com.lwen"/>

        <property name="jpaProperties">
            <props>
                <prop key="hibernate.ejb.naming_strategy">org.hibernate.cfg.ImprovedNamingStrategy</prop>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5InnoDBDialect</prop>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.format_sql">true</prop>
                <prop key="hibernate.hbm2ddl.auto">update</prop>
            </props>
        </property>

    </bean>

    <!--3 配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
    </bean>

    <!--4 配置支持注解的事务-->
    <tx:annotation-driven/>

    <!--5 配置spring data-->
    <jpa:repositories base-package="com.lwen" entity-manager-factory-ref="entityManagerFactory"/>

    <context:component-scan base-package="com.lwen"/>

</beans>
```

## 3.实体类

```java
package com.lwen.entry;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

/**
 * Student实体类
 */
@Entity
public class Student {

  @Id@GeneratedValue
  private int id;
  private String name;
  private int age;

  public int getId() {
    return id;
  }

  public void setId(int id) {
    this.id = id;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

这里注意的一点就是我们在使用注解的时候一定要注意导入的包，我们都是导入的javax中的类。

# 2.Repository

## 1. 使用

这个东西是SpringData的核心，但是我们实际去看的时候会发现他是一个空接口，也就是这个一个标记接口。我们自己的接口必须继承这个接口才会具备查询的功能，所以说我们的自定的查询器必须要继承这个接口。这个接口是泛型接口也就是我们需要输入两个参数，第一个就是我们查询器的类型，针对于那个表进行查询，另外一个就是表的Id的类型，这个类型必须是序列化接口的子类型，所以说不能使用基本类型，我们只能使用包装类型。

```java
public interface EmployeeRepository extends Repository<Employee,Integer> {
    Employee findByName(String name);
}
```

但是我们还有另外一种方式，就是使用注解的方式不用继承这个接口。

```java
@RepositoryDefinition(domainClass = Student.class,idClass = Integer.class)
public interface StudentRepository {
}
```



## 2. 常用的子接口

- CrudRepository：继承Repository，实现了CRUD相关的方法
- PagingAndSortingRepository：继承CrudRepository，实现了分页排序相关的方法

- JpaRepository：继承PagingAndSortingRepository，实现JPA规范相关的方法

这些接口的功能都是非常强大并且实用的我们在使用的时候就可以直接继承这些接口。

## 3.查询规则

## 1.约定方法签名查询

[![CcX89e.md.png](https://s1.ax1x.com/2018/05/19/CcX89e.md.png)](https://imgchr.com/i/CcX89e)

[![CcXG1H.md.png](https://s1.ax1x.com/2018/05/19/CcXG1H.md.png)](https://imgchr.com/i/CcXG1H)

## 2.手动查询 @Query

### 1.复杂查询

```java
@Query("select o from Employee o where id = (select max(id) from Employee)")
Employee getMaxIdEmployee();
```



### 2.占位符查询

```java
@Query("select o from Employee o where name=?1 and age=?2")
Employee getByNameAndAge(String name,Integer age);

@Query("select o from Employee o where name like %?1%")
Employee getByLikeName(String name);
```



### 3.命名参数

```java
@Query("select o from Employee o where name=:name and age=:age")
Employee getByNameAndAge1(@Param("name") String name,@Param("age") Integer age);
```



### 4.原生态查询

```java
@Query(nativeQuery = true,value = "select * from spring_data.employee where name = ?1")
void getNative(String name)；
```

## 3. 更新删除操作

在SpringData中使用插删改操作的时候我们必须定义一个Service层，然后我们在Service层调用Dao的Repository来更新数据库，接着我们需要将那个Repository的方法设置为 @Modifying ，最后最重要的就是我们在Service层的那个方法中写 @Transactional 注解。才能更新成功，所以所有的事务只能出现在 Service 层。但是注意因为我们的Service没有继承任何的Spring相关的东西我们要把它放到容器的时候需要使用@Service注解，否则是不行的。

``` java
@Modifying
@Query("update Employee o set o.name=:name where o.id=:id")
void update(@Param("id") Integer id,@Param("name") String name);
```

```java
@Service
public class EmployeeService {
    @Autowired
    EmployeeRepository repository;

    @Transactional //这个是javax里面的
    public void update(){
        repository.update(1, "lwenxu");
    }
}
```

**小提示，很多时候我们发现有些东西在Spring中有在javax中也有，我们优先导入Javax中的，如果出现了什么方法无法调用估计就是包导错了。**

# 3. 常用的Repository

这三个高级的Repository实际上是从上到下依次继承的。

## 1. CrudRepository

我们的Repository必须要继承这个接口，然后我们就有crud的一些操作了。接着我们需要创建service层，然后再service中使用事务，并且注入我们的Repository，这里我们用了一张新的表我们在bean上指定我们的表名就是使用@Table(name = "employee_test") 注解。最后进行save操作。

``` java
@Transactional
public void saveAll(List<Employee> employees){
    employeeCrudRepository.save(employees);
}
```

``` java
EmployeeCrudRepository employeeCrudRepository = ctx.getBean(EmployeeCrudRepository.class);
ArrayList<Employee> employees = new ArrayList<Employee>();
for (int i = 0; i < 100; i++) {
    employees.add(new Employee(i, "lwen" + i, i));
}
employeeCrudRepository.save(employees);
```

## 2.PagingAndSortingRespository

他是分页和排序功能。

``` java
EmployeePageAndSortRepository pageAndSort = ctx.getBean(EmployeePageAndSortRepository.class);

//创建一个排序器，是按照id的降续排列的
Sort sort = new Sort(new Sort.Order(Sort.Direction.DESC, "id"));
//第一个参数是当前的页码他是从0开始的
//第二个参数就是每一页的大小
//第三个参数是可选参数，传入一个sort，就是按照哪种方式分页  由于这里用的是降续所以出来的结果应该是 0 在最后一页 99在第一页
Pageable pageable = new PageRequest(0, 9, sort);
Page<Employee> page = pageAndSort.findAll(pageable);
//获取当前页的内容
System.out.println(page.getContent());
//获取所有的页数
System.out.println(page.getTotalPages());
```

## 3.JpaRepository

- findAll
- save(entries)
- deleteInBatch
- findAll(sort)
- flush

# 4. JpaSpecificationExecutor

这里吧这个接口单独拿出来说主要是因为这个接口实际上不是继承自 Repository 这个接口，他的作用从表面上貌似也是看不出来，实际上我们前面看到我们可以进行简单的分页，但是那些分页并没有一个让我们传入查询条件的地方，我们这个接口就是实现了这个功能也就是有条件的分页查询。

jpa接口需要继承 JpaSpecificationExecutor 然后这里就包含了五个方法，分别是 findAll(三个重载) 和 findOne 以及 count 。

然后在使用的时候只调用对应的方法即可，Page<T> findAll(@Nullable Specification<T> var1, Pageable var2); 这个算是里面最复杂的方法了主要是需要传递一个 Specification 对象，这个一般我们直接传递一个匿名的对象即可，实现后的代码如下：

```java
new Specification<User>() {
    public Predicate toPredicate(Root<User> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
        return criteriaBuilder.gt(root.<Number>get("id"),4);
    }
}
```

可以看到这个函数式接口里面的第一个参数是 root 这个相当于一个导航器，也就是用它可以获取到我们实体类中的属性，也就是我们获取到表的字段，CriteriaQuery 则是可以进行语句的拼装，里面有 where ，groupby 以及having 等方法，进行sql组合的。二最后一个参数就是 CriteriaBuilder 用来创建 Predicate 对象的，也就是生成查询条件对象。

上面的程序生成的最后的条件就是获取id大于4 的所有信息，然后分页展示。

# 5. 表的映射关系

## 1.一对一

一对一关系这里定义了一个Person对象以及一个IDCard对象

Person类：

```java
@Entity
@Table(name="t_person")
public class Person
{
    private int id;
    private String name;
    private IDCard card;
    
    @OneToOne(mappedBy="person")　　--->　　指定了OneToOne的关联关系，mappedBy同样指定由对方来进行维护关联关系
    public IDCard getCard()
    {
        return card;
    }
    public void setCard(IDCard card)
    {
        this.card = card;
    }
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    
}
```

IDCard类：

```java
@Entity
@Table(name="t_id_card")
public class IDCard
{
    private int id;
    private String no;
    private Person person;
    
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getNo()
    {
        return no;
    }
    public void setNo(String no)
    {
        this.no = no;
    }
    @OneToOne　　--->　　OnetoOne指定了一对一的关联关系，一对一中随便指定一方来维护映射关系，这里选择IDCard来进行维护
    @JoinColumn(name="pid")　　--->　　指定外键的名字 pid
    public Person getPerson()
    {
        return person;
    }
    public void setPerson(Person person)
    {
        this.person = person;
    }
}
```

**注意**:在判断到底是谁维护关联关系时，可以通过查看外键，哪个实体类定义了外键，哪个类就负责维护关联关系。

## 2.多对一

这里我们定义了两个实体类，一个是ClassRoom，一个是Student，这两者是一对多的关联关系。

ClassRoom类：

```java
@Entity
@Table(name="t_classroom")
public class ClassRoom
{
    private Set<Student> students;
    
    public ClassRoom()
    {
        students = new HashSet<Student>();
    }
    
    public void addStudent(Student student)
    {
        students.add(student);
    }

    @OneToMany(mappedBy="room")　　--->　　OneToMany指定了一对多的关系，mappedBy="room"指定了由多的那一方来维护关联关系，mappedBy指的是多的一方对1的这一方的依赖的属性，(注意：如果没有指定由谁来维护关联关系，则系统会给我们创建一张中间表)
    @LazyCollection(LazyCollectionOption.EXTRA)　　--->　　LazyCollection属性设置成EXTRA指定了当如果查询数据的个数时候，只会发出一条 count(*)的语句，提高性能
    public Set<Student> getStudents()
    {
        return students;
    }

    public void setStudents(Set<Student> students)
    {
        this.students = students;
    }
    
}
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Student类：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
@Entity
@Table(name="t_student")
public class Student
{
    private ClassRoom room;
    
    @ManyToOne(fetch=FetchType.LAZY)　　---> ManyToOne指定了多对一的关系，fetch=FetchType.LAZY属性表示在多的那一方通过延迟加载的方式加载对象(默认不是延迟加载)
    @JoinColumn(name="rid")　　--->　　通过 JoinColumn 的name属性指定了外键的名称 rid　(注意：如果我们不通过JoinColum来指定外键的名称，系统会给我们声明一个名称)
    public ClassRoom getRoom()
    {
        return room;
    }
    public void setRoom(ClassRoom room)
    {
        this.room = room;
    }
}
```

## 3.多对多

多对多这里通常有两种处理方式，一种是通过建立一张中间表，然后由任一一个多的一方来维护关联关系，另一种就是将多对多拆分成两个一对多的关联关系

### 1. 不使用中间表的实体类

采用中间表的时候由任一一个多的一方来维护关联关系

Teacher类：

```java
@Entity
@Table(name="t_teacher")
public class Teacher
{
    private int id;
    private String name;
    private Set<Course> courses;
    
    public Teacher()
    {
        courses = new HashSet<Course>();
    }
    public void addCourse(Course course)
    {
        courses.add(course);
    }
    
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    @ManyToMany(mappedBy="teachers")　　--->　　表示由Course那一方来进行维护
    public Set<Course> getCourses()
    {
        return courses;
    }
    public void setCourses(Set<Course> courses)
    {
        this.courses = courses;
    }
    
}
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Course类：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
@Entity
@Table(name="t_course")
public class Course
{
    private int id;
    private String name;
    private Set<Teacher> teachers;
    
    public Course()
    {
        teachers = new HashSet<Teacher>();
    }
    public void addTeacher(Teacher teacher)
    {
        teachers.add(teacher);
    }
    @ManyToMany　　　--->　ManyToMany指定多对多的关联关系
    @JoinTable(name="t_teacher_course", joinColumns={ @JoinColumn(name="cid")}, 
    inverseJoinColumns={ @JoinColumn(name = "tid") })　　--->　　因为多对多之间会通过一张中间表来维护两表直接的关系，所以通过 JoinTable 这个注解来声明，name就是指定了中间表的名字，JoinColumns是一个 @JoinColumn类型的数组，表示的是我这方在对方中的外键名称，我方是Course，所以在对方外键的名称就是 rid，inverseJoinColumns也是一个 @JoinColumn类型的数组，表示的是对方在我这放中的外键名称，对方是Teacher，所以在我方外键的名称就是 tid
    public Set<Teacher> getTeachers()
    {
        return teachers;
    }

    public void setTeachers(Set<Teacher> teachers)
    {
        this.teachers = teachers;
    }

    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

}
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 2. 采用中间表实体类

把之前的ManyToMany拆分成两个One-to-Many的映射

Admin类：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
@Entity
@Table(name="t_admin")
public class Admin
{
    private int id;
    private String name;
    private Set<AdminRole> ars;
    public Admin()
    {
        ars = new HashSet<AdminRole>();
    }
    public void add(AdminRole ar)
    {
        ars.add(ar);
    }
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    @OneToMany(mappedBy="admin")　　--->　　OneToMany关联到了AdminRole这个类，由AdminRole这个类来维护多对一的关系，mappedBy="admin"
    @LazyCollection(LazyCollectionOption.EXTRA)　　
    public Set<AdminRole> getArs()
    {
        return ars;
    }
    public void setArs(Set<AdminRole> ars)
    {
        this.ars = ars;
    }
}
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Role类：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
@Entity
@Table(name="t_role")
public class Role
{
    private int id;
    private String name;
    private Set<AdminRole> ars;
    public Role()
    {
        ars = new HashSet<AdminRole>();
    }
    public void add(AdminRole ar)
    {
        ars.add(ar);
    }
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    @OneToMany(mappedBy="role")　　--->　　OneToMany指定了由AdminRole这个类来维护多对一的关联关系，mappedBy="role"
    @LazyCollection(LazyCollectionOption.EXTRA)
    public Set<AdminRole> getArs()
    {
        return ars;
    }
    public void setArs(Set<AdminRole> ars)
    {
        this.ars = ars;
    }
}
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

AdminRole类：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
@Entity
@Table(name="t_admin_role")
public class AdminRole
{
    private int id;
    private String name;
    private Admin admin;
    private Role role;
    @Id
    @GeneratedValue
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
    @ManyToOne　　--->　　ManyToOne关联到Admin
    @JoinColumn(name="aid")　　
    public Admin getAdmin()
    {
        return admin;
    }
    public void setAdmin(Admin admin)
    {
        this.admin = admin;
    }
    @ManyToOne　　--->　　
    @JoinColumn(name="rid")
    public Role getRole()
    {
        return role;
    }
    public void setRole(Role role)
    {
        this.role = role;
    }
}
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**小技巧**:通过hibernate来进行插入操作的时候，不管是一对多、一对一还是多对多，都只需要记住一点，在哪个实体类声明了外键，就由哪个类来维护关系，在保存数据时，总是先保存的是没有维护关联关系的那一方的数据，后保存维护了关联关系的那一方的数据

## 4.中间表

两个实体tb_user,tb_role 
现在我们再tb_user或者tb_role中任意一个里面进行维护关系，多对对的情况下我们需要创建一个中间表来完成这个关系的映射，我们再tb_user中添加注解@ManyToMany然后再添加一个注解@JoinTable因为我们是要创建中间表所以要使用这个注解。JoinTable注解中我们添加如下例子中的内容，joinColumns当前表中的字段在中间表中的字段名称，inverseJoinColumns关联的外键表在中间表中的字段名称


```java
@Entity
@Table(name = "tb_user")
@SequenceGenerator(name = "tb_user_sq",sequenceName = "tb_user_sqe")
public class TbUser extends BaseEntity{
/**
 * 用户名
 */
private String userName;
/**
 * 登录名
 */
private String loginName;
/**
 * 登陆密码
 */
private String passWord;
/**
 * 手机号
 */
private String telPhone;
/**
 * 一个用户可以有多个角色
 */
private List<TbRole> tbRoleList=new ArrayList<>();

public String getUserName() {
    return userName;
}

public void setUserName(String userName) {
    this.userName = userName;
}

public String getLoginName() {
    return loginName;
}

public void setLoginName(String loginName) {
    this.loginName = loginName;
}

public String getPassWord() {
    return passWord;
}

public void setPassWord(String passWord) {
    this.passWord = passWord;
}

public String getTelPhone() {
    return telPhone;
}

public void setTelPhone(String telPhone) {
    this.telPhone = telPhone;
}

@Id
@Override
@GeneratedValue(generator = "tb_user_sq",strategy = GenerationType.SEQUENCE)
public Long getId() {
    return this.id;
}
@ManyToMany(cascade = CascadeType.REMOVE,fetch = FetchType.LAZY)
@JoinTable(name = "tb_user_role",joinColumns = @JoinColumn(name="tb_user_id",referencedColumnName = "id"),inverseJoinColumns = @JoinColumn(name = "tb_role_id",referencedColumnName = "id"))
public List<TbRole> getTbRoleList() {
    return tbRoleList;
}

public void setTbRoleList(List<TbRole> tbRoleList) {
    this.tbRoleList = tbRoleList;
}
}
```

因为在tb_user中我们维护了两个表的关系，所以如果我们在tb_role中如果不想创建关联字段的话就不用添加tbUser的关系字段


```java
@Entity
@Table(name = "tb_role")
@SequenceGenerator(name = "tb_role_sq",sequenceName = "tb_role_sqe")
public class TbRole extends BaseEntity{
@Override
@Id
@GeneratedValue(generator = "tb_role_sq",strategy = GenerationType.SEQUENCE)
public Long getId() {
    return this.id;
}

private String roleName;
@ManyToMany(mappedBy = "tbRoleList")
public String getRoleName() {
    return roleName;
}

public void setRoleName(String roleName) {
    this.roleName = roleName;
}
```

如果想要在tb_role中进行维护关联字段的话如下，我们在字段中添加注解 @ManyToMany然后使用直接属性mappedBy值是当前表在关联表中的字段名称


```java
@Entity
@Table(name = "tb_role")
@SequenceGenerator(name = "tb_role_sq",sequenceName = "tb_role_sqe")
public class TbRole extends BaseEntity{
@Override
@Id
@GeneratedValue(generator = "tb_role_sq",strategy = GenerationType.SEQUENCE)
public Long getId() {
    return this.id;
}

private String roleName;

private List<TbUser> tbUserList=new ArrayList<>();

public String getRoleName() {
    return roleName;
}

public void setRoleName(String roleName) {
    this.roleName = roleName;
}
@ManyToMany(mappedBy = "tbRoleList")
public List<TbUser> getTbUserList() {
    return tbUserList;
}

public void setTbUserList(List<TbUser> tbUserList) {
    this.tbUserList = tbUserList;
}
```