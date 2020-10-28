---
title: Filter 链
date: 2019-06-04 10:49:28
tags: Filter
categories:
 -Java
---

 在写JavaWeb的时候我们经常会遇到一个概念就是 Filter 链，比如对于路由拦截，以及文件目录拦截都有一个Filter 和Filter链，那么他们底层具体到底是怎么工作的呢？

听到Filter 链这个名词我们觉得一个 Filter 链应该可以进行链式调用，来判断我们的条件是否成立，也就是类似于下面的规则:

```java
filterChain.doFilter(pattern).doFilter(pattern)
```

这样想是没错，那每次就需要返回下一个 Filter 这么想是没错，可是我们怎么知道下一次 Filter 是谁呢？

链表！ 对滴，这时候链表就起作用了，我们直接采用了 LinkedList 就能把这些 Filter 串起来。然后根据当前的 Filter 找出下一个 Filter 。

实际上在某些框架之中不是这么实现了的，但是基本的思路已经非常对了。例如这里是 SpringSecurity 中的 Filter 的简单实现：

```java
import java.util.LinkedList;
import java.util.List;

public class Test1 {
    interface Filter{
        public void doFilter(FilterChain filterChain);
    }

    static class MyFilter implements Filter {
        @Override
        public void doFilter(FilterChain filterChain) {
            System.out.println("filter invoke");
            filterChain.doFilter();
        }
    }

    interface FilterChain{
        public void doFilter();
    }

    static class MyFilterChain implements FilterChain{
        List<Filter> filters;
        int pos = 0;

        MyFilterChain(List<Filter> filters) {
            this.filters = filters;
        }

        @Override
        public void doFilter() {
            if (pos == filters.size()) {
                System.out.println("end");
                return;
            }
            Filter filter = filters.get(pos++);
            filter.doFilter(this);
        }
    }

    public static void main(String[] args) {
        LinkedList<Filter> list = new LinkedList<>();
        list.add(new MyFilter());
        list.add(new MyFilter());
        list.add(new MyFilter());
        new MyFilterChain(list).doFilter();
    }
}

```



 