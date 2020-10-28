---
title: Hibernate笔记(二)
date: 2017-08-21 16:00:58
tags: Hibernate
categories:
  -JAVA
---
### 1.一对多的关系映射
对于一的一方：
<!--more-->
```java
package domain;

import java.util.HashSet;
import java.util.Set;

public class Customer {
    private int cid;
    private String name;
    //表示多个联系人
    private Set<Person> people=new HashSet<>();

    public Set<Person> getPeople() {
        return people;
    }

    public void setPeople(Set<Person> people) {
        this.people = people;
    }

    public int getCid() {
        return cid;
    }

    public void setCid(int cid) {
        this.cid = cid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
对于多的一方
```java
package domain;

public class Person {
    private int uid;
    private String name;
    //表示客户
    private Customer customer;

    public Customer getCustomer() {
        return customer;
    }

    public void setCustomer(Customer customer) {
        this.customer = customer;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getUid() {
        return uid;
    }

    public void setUid(int uid) {
        this.uid = uid;
    }
}
```
他们的mapping文件分别就是：
一的一方
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="domain.Customer" table="customer">
        <id name="cid" column="id">
            <generator class="native"/>
        </id>
        <property name="name" column="name"/>

          <!-- 单方面的维护关系 -->
        <set name="people" inverse="true">
            <key column="cpid"/>
            <one-to-many class="domain.Person"/>
        </set>
    </class>
</hibernate-mapping>
```
多的一方
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="domain.Person" table="people">
        <id name="uid" column="id">
            <generator class="native"/>
        </id>
        <property name="name" column="name"/>
        <many-to-one name="customer" class="domain.Customer" column="clid"/>
    </class>
</hibernate-mapping>
```

### 2.一对多的级联操作
首先在一的一方的配置文件的set标签中写一个属性 cascade="save-update,delete" 这个就是级联配置，然后我们在写具体的代码的时候我们只需要把多的一方设置到一的一方的对象set中就可了，此时直接操作了一的一方就能把多的一方顺便就操作了。

### 3.多对多的配置
```java
package domain1;

import java.util.HashSet;
import java.util.Set;

public class User {
    private int cid;
    private String name;
    //表示多个联系人
    private Set<Role> roles=new HashSet<>();

    public Set<Role> getRoles() {
        return roles;
    }

    public void setRoles(Set<Role> roles) {
        this.roles = roles;
    }

    public int getCid() {
        return cid;
    }

    public void setCid(int cid) {
        this.cid = cid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
```java
package domain1;

import java.util.HashSet;
import java.util.Set;

public class Role {
    private int cid;
    private String name;
    //表示多个联系人
    private Set<User> users=new HashSet<>();


    public Set<User> getUsers() {
        return users;
    }

    public void setUsers(Set<User> users) {
        this.users = users;
    }

    public int getCid() {
        return cid;
    }

    public void setCid(int cid) {
        this.cid = cid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="domain1.Role" table="role">
        <id name="uid" column="id">
            <generator class="native"/>
        </id>
        <property name="name" column="name"/>


        <set name="users" table="user_role" cascade="save-update,delete">
            <key column="rid"/>
            <many-to-many class="domain1.User" column="uid"/>
        </set>
    </class>
</hibernate-mapping>
```
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="domain1.User" table="user">
        <id name="cid" column="id">
            <generator class="native"/>
        </id>
        <property name="name" column="name"/>

        <set name="roles" table="user_role">
            <key column="uid"/>
            <many-to-many class="domain1.Role" column="rid"/>
        </set>
    </class>
</hibernate-mapping>
```
