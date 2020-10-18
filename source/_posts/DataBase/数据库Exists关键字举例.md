---
title: 数据库Exists关键字举例
date: 2017-05-12 07:30:00
tags:
categories:
  -数据库
---
## 一.问题描述：

>查询所有未选择03号课程的学生的姓名
规定使用存在量词

<!--more-->
#### student表：
![](http://ojrw3x8j2.bkt.clouddn.com/17-5-11/11695243-file_1494553101102_14292.png)

#### grade表：
![](http://ojrw3x8j2.bkt.clouddn.com/17-5-11/67277042-file_1494553101231_3769.png)



## 二.思路：
既然是存在量词那么也就是Exists和Not Exists两个存在两次来做判断条件。
那么就可以想：
#### 1.选出所有学生然后除去那些选了03号课程的同学
#### 2.直接选出那些没有选择03号课程的同学

## 三.开始写Sql：

### 1.尝试

首先按照第一种思路就是先连接student和grade（或者不连接）然后除掉grade中有03课程记录的学生，那么我们会写出以下Sql:

```bash
SELECT DISTINCT sname from Student,Grade WHERE NOT exists(
  SELECT * FROM Grade WHERE cno='03'
);
或者
SELECT DISTINCT sname from Student WHERE NOT exists(
  SELECT * FROM Grade WHERE cno='03'
);
```
执行以后我们发现我们得到的结果是空

接下来我们看看student表中到底有多少人：
执行以下语句：
```bash
SELECT DISTINCT sno,sname FROM Student GROUP BY sno;
```
![](http://ojrw3x8j2.bkt.clouddn.com/17-5-11/62530586-file_1494547912119_5cb4.png)

这里我们发现这条语句根本没有进行筛选，这是因为Exists不知道使用什么条件去筛选数据，前面是一个结果集后面为另一个结果集数据库不清楚按照哪个字段来判断前面的某条记录是否存在与后面的集合中。
### 2.修正错误：
这里在两个表之间显然只有sno字段是相关的，所以在最后添加判断条件：
```bash
Grade.sno=Student.sno
也就是
SELECT DISTINCT sname from Student WHERE NOT exists(
  SELECT * FROM Grade,Student WHERE cno='03' AND Grade.sno=Student.sno
);
```
此时出现了10条记录经过比对发现是正确的。
### 3.接下来用sql语句验证结论：
```bash
SELECT DISTINCT sno,sname FROM Student GROUP BY sno;
SELECT sno FROM Grade WHERE cno='03';
```
第一条语句就是找到student表中的所有人，第二条语句就是看看哪些人选择了03号课程，结果如下。
![](http://ojrw3x8j2.bkt.clouddn.com/17-5-11/83704707-file_1494548125454_171c0.png)

发现结论正确，但是这两个语句貌似可以拼接成一条新的sql来完成这道题：

```bash
SELECT DISTINCT sname
FROM Student  where sno NOT IN (
  SELECT sno FROM Grade WHERE cno='03'
);
```
结果同上。

### 4.反向思维：
既然not Exists是可以的那么是不是可以把那条语句修改成exists语句然后把where里面的条件再非一下，执行以下语句：

```bash
SELECT sname FROM Student WHERE  exists(
    SELECT DISTINCT Student.sno FROM Grade WHERE Grade.sno=Student.sno
    AND cno!='03'
);
```
![](http://ojrw3x8j2.bkt.clouddn.com/17-5-11/7425028-file_1494548893077_17bc2.png)

但是结论是只有七条数据，显然差距比较大。
也就是子查询是有问题的，查表以后发现在student表中是有十二个同学，但是grade表中选课的同学总数并不是十二个，也就是说有很多人根本没有选课。并且另外一个问题就是有的人选了不止一门课可能某个同学既选了03又选了别的课程，而在使用exits判断的时候只要有满足条件的就返回，自然是没等判断到选了03就能返回真。所以这个方法行不通。

### 5.修改：
上面出现问题，说明使用exists思路没问题只是子查询错误,试试运用course表看能不能写出其他语句：

```bash
SELECT sname FROM Student WHERE  exists(
  SELECT cno FROM Course WHERE sno NOT IN (
    SELECT sno FROM Grade WHERE cno='03'
  )
);
```
把exits换成IN
```bash
SELECT DISTINCT sno
FROM Student  where sno  IN (
  SELECT sno FROM Course WHERE sno NOT IN (
    SELECT sno FROM Grade WHERE cno='03'
  )
);
```

看以下的句子：

```bash
SELECT sname FROM Student WHERE  exists(
  SELECT dno FROM Department WHERE sno NOT IN (
    SELECT sno FROM Grade WHERE cno='03'
  )
);
```

仔细看就会发现其实上面的句子其实就是：
```bash
SELECT DISTINCT sname
FROM Student  where sno NOT IN (
  SELECT sno FROM Grade WHERE cno='03'
);
```
只是中间添加了一些毫无逻辑的语句，但是对结果没有任何影响。
