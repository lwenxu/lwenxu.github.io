---
title: Scala 入门
date: 2018-05-08 14:01:28
tags: Spark
categories:
 - Spark
---

# 1. 基本语法

### 1. 变量 var

```scala
var name:string="hello";
var name="hello";
var num=3.0;
```

### 2. 常量 val

```scala
val name="string";
```

### 3. 这里是没有 ++ 操作和 -- 操作的

### 4. 函数调用

不用使用对象来调用，导入包以后直接调用即可

```scala
import scala.math._
sqrt(2);
pow(1,2);
min(1,2);
```

### 5.apply()

默认调用了object的apply函数

```scala
"hello"<6>;
```

### 6.if-else

