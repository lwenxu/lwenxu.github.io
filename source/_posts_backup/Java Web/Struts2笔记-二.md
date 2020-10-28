---
title: Struts2笔记(二)
date: 2017-08-22 18:52:11
tags: struts2
categories:
  -JAVA
---
### 1.结果页面
#### 1.全局结果页面
当一些方法的返回值一样的时候并且需要跳转的页面也一样的时候就可以使用全局结果页面。  
```xml
<package name="struts" extends="struts-default" namespace="/">
    <global-results>
        <result name="done">index.jsp</result>
    </global-results>
    <action name="hello" class="demo01.Hello">
        <result name="success">/index.jsp</result>
    </action>
</package>
```
注意他只是太某一个package中，而不是整个项目<!--more-->
#### 2.局部结果页面
全局页面和局部页面同事存在并且冲突的时候是以局部的为准。

### 2.action获取表单数据。
#### 1.使用ActionContext对象
在action中没办法调用request对象，所以我们智力只能使用actionContext对象获取。
首先需要获取actionContext对象，然后这个对象不能new，而是通过他自己的一个静态方法获取的，名字就叫做
getContext(),最后使用getParamters方法就能获取到表单的数据。
```java
ActionContext actionContext=ActionContext.getContext();
Map<String, Object> map = actionContext.getParameters();
```
#### 2.ServletActionContext对象
里面的方法大多数都是静态的，我们直接可以调用方法。我们可以获取HttpServletRequest对象也可以获取ActionContext对象。
```java
//获取request对象
HttpServletRequest request= ServletActionContext.getRequest();
request.getParameter("name");
```
#### 3.接口注入
让action实现ServletRequestAware接口，然后实现接口的方法，这个方法其实会有一个参数，这个参数就是我们需要的request对象，这样tomcat注入以后我们就可以保存起来直接使用。

### 3.action操作域对象
其实action更像是一个servlet，这里action中可以操作的域对象和servlet一样也是三个。
* request  使用ServletActionContext对象的getRequest方法获取
* session  使用request对象的getSession获取
* ServletContext  使用ServletActionContext的getServletContext方法获取

### 4.表单数据自动注入
#### 1.属性封装
直接把要提交的属性的name值放在action的属性中，然后生成属性的set方法这样就能完表单数据属性的自动注入
#### 2.模型驱动封装
模型驱动封装就可以直接把这个表单的数据直接封装到一个实体类中  
首先需要实现ModelDriven接口，这个借口需要传入一个对象类型，这个对象就是实体类的类型。然后我们在这个action中必须要有一个实体类的对象，必须要new出来。然后需要实现这个接口的方法，getModel方法，返回值返回的就是我们创建的实体类的对象。  
属性封装和模型驱动不能同时使用，到时候只有模型驱动起作用，属性封装获取的都是null。
#### 3.表达式封装
* 声明实体类，不实例化。
* 生成变量的set和get方法
* 在表单name中使用表达式 user.username 就是这个属性。通过get方法获取user，设置完属性以后set方法更新值。

### 5.值栈
值栈就类似于一个域对象
#### 1.存储位置
每一个action都只有一个值栈区，每一次访问的时候action都会被重新创建，也就是action多例的。
#### 2.获取值栈的对象
使用ActionContext对象  
```java
 actionContext.getValueStack();
 ```
#### 3.值栈的结构
值栈主要分为两部分，root和context。root是List集合而context是一个map集合。context中存放了三大域对象。
而我们主要使用的空间就是root，在root中默认的栈顶元素就是当前的action对象。
#### 4.向值栈中存数据
* 调用值栈对象的set方法
* 调用push方法
* 在action定义属性，然后生成get方法，最后再调用的方法中设置这个属性的值，这样这个值就被放到了值栈中。
#### 5.获取值栈中的数据
* 字符串  <s:property value="username"/>   直接根据名称获取值
* 对象  <s:property value="user.username"/>  user中的username属性
* list集合  
    * 方案一 ： <s:property value="list[0].username"/>
    * 方案二 ：
    ```xml
              <s:iterator value="list">
                <s:property value="username"/>
              </s:iterator>
    ```
    * 方案三 ：
    使用var的时候每次遍历的user对象的引用放在了值栈中的context中了，但是我们要取值栈中的context中的内容需要使用#，所以就有以下的表达式。
    ```xml
      <s:iterator value="list" var="user">
        <s:property value="#user.username"/>
      </s:iterator>
    ```
* 获取set方法放的数据     <s:property value="username"/>
* 获取push方法放的数据    <s:property value="[0].top"/>  push是放在了一个栈中了，这个就是一个数组，取数组的第一个元素
* 使用EL表达式也是可以获取值栈的数据。他的原理就是如果能在域对象中得到变量的时候就是域对象中的，获取不到的时候就去值栈查找，然后再放到域对象中。而这些操作能执行的原因就是在filter中给域对象做了增强也就是用了一个装饰设计模式。
### 6.OGNL表达式以及struts2标签库
ognl表达式类似于el表达式，他的主要用途就是操作值栈里面的内容。  
struts2标签库的使用必须要导入他的标签库。
#### 1.调用方法
获取字符串hello的长度
```html
<s:property value="'hello'.lenth()"/>
```
#### 2.两个特殊符号的使用
* # 这个就是获取context中的数据。  <s:property value="#request.username"/>  获取域对象中的username
* % 表示在struts标签中使用ognl表达式，不识别，我们为了让他识别使用%  例如  <s:textarea value="%#request.username"/>
