---
title: Hibernate笔记(三)
date: 2017-08-21 16:01:03
tags: Hibernate
categories:
  -JAVA
---
### 1.对象导航查询
两个相关的对象
### 2.OID查询
用id查出对象
### 3.hql查询
Query对象<!--more-->
#### 1.hql
* 查询所有   from entity
* 条件查询   from entity where name=?     setParameter(index,arg)  设置参数值
* 模糊查询   from entity where name  like ?；  
* 排序      from entity order by name desc
* 分页      setFirstResult()开始位置  setMaxResults()每页数
* 投影      select property from entity  property不能是*  
* 聚集函数
    *  count  select count(* ) from entity    query.uniqueResult()  
    *  其他的类似
* 多表查询  
    * 内连接 form entity inner join entity.set  最后返回的是数组
    * 迫切内连接 form entity inner join fetch entity.set  最后返回的是list
    * 外连接 form entity left outer join entity.set  最后返回的是数组
    * 迫切左外连接 form entity left outer join fetch entity.set  最后返回的是list

### 4.QBC查询
Criteria对象
createCriteria(entity.class)
* 查询所有   list()
* 条件查询   add(Restrictions.eq/like/("property","value")) -> list()
* 排序      addOrder(Order.asc("property"))
* 分页      setFirstResult()开始位置  setMaxResults()每页数
* 统计查询   setProjection(Projetions.rowCount(10));
* 离线查询    DetachCriteria deCriteria=DetachCriteria.forClass(entity.class)
             Criteria criteria=deCriteria.getExectueableCriteria();
             与session无关
### 5.本地sql查询
SQLQuery对象

### 6.Hibernate的查询策略
#### 1.立即查询
get方法就是立即查询，方法执行立即发送语句   get(entity.Class,id)
#### 2.延时查询
load方法是延时查询，方法调用不会立即发送语句，只有当我们获取返回的对象中的非id字段的值得时候才会发语句。
##### 1.类级别的延迟
例如根据id的查询，最后查的是一个类的某个对象
##### 2.关联级别的延迟
当表之间是有关系的，然后我们进行延迟查询
