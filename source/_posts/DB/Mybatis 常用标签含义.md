---
title: Mybatis 常用标签
date: 2021-01-10 08:01:28
tags: Mybatis
categories:
 - Mybatis
---

## trim 

这个标签的作用就是帮你给标签的内容的头部或者尾部 `删除` 或者 `添加` 特定字符。

 举个例子：

我们的 where 下面经常会有各种条件，假如这些条件我们一个都不传那么我们就应该删除这个 where ，也或者说如果标签里面的内容不为空的话那么就给我们加上 where

```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ...
</trim>
```

这段的意思就是：

1. 如果 trim 标签里面的内容为空，那么不要加 where ，否则加上
2. 如果标签里面的第一个元素是 and 或者是 or 那么删除这个 and 或者是 or



## Where 

这个标签如果看懂了上面的话，那么他的意思就是上面的那段 tirm

