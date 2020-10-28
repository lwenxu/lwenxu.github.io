---
title: SpringMVC 踩坑记录
date: 2018-04-22 08:01:28
tags: SpringMVC
categories:
 - SpringMVC
---

## 1. 处理静态资源

静态资源直接放在 webapp/web 下，而我们的模板一般是在 WEB-INF 下，但是 WEN-INF	 下的东西一般不让访问的，模板之所以能访问到是因为有模板引擎的映射，但是我们的警惕资源应该是直接能访问的东西，直接放在 web 下类似于 jsp 直接访问，而不能放在 WEB-INF 下，并且我们要开启静态资源访问 `<mvc:default-servlet-handler/>`

## 2. Thymeleaf 乱码问题

一开始除了乱码想着直接在 web.xml 中配置编码过滤器，接着发现根本没用还是一样的乱码，并且返回的是 ISO-8859-1的西欧字符集。后来想着如果是不是 SpringMVC 的问题，那么可能就是因为视图是被 Thymeleaf 渲染的导致的问题，然后找到 Thymeleaf 在 SpringMVC 中的配置，然后更改编码。重新启动项目才行。