---
title: Java Class对象与反射机制
date: 2018-08-04 21:38:50
tags: Java 虚拟机
categories:
	-Java 
	-虚拟机
---



# Java Class对象与反射机制

## 一、关于Class对象

### 1.基本介绍

其实我们在任何一个 Java 应用程序中都会存在 class 对象，这个 class 对象其实就是每一个类、类型、数组、接口 等等被加载的时候在 jvm 中创建的。也就是这些类型的一个映射，这些类型在虚拟机中的实体。正是由于存在这个对象在 Jvm 中我们可以通过一定的方式获取这个对象然后去用这个对象去对我们的各种类型做一些加载获取属性等等操作，当然主要是对类进行操作并能够执行他的方法，探知未知的类的各种信息。

### 2.官方文档解释

这是我摘录的 Class 类的 javadoc 内容：

> 1. Instances of the class Class represent classes and interfaces in a running Java application.
> 2. An enum is a kind of class and an annotation is a kind of interface.
> 3. Every array also belongs to a class that is reflected as a Class object that is shared by all arrays with the same element type and number of dimensions.
> 4. The primitive Java types (boolean, byte, char, short, int, long, float, and double), and the keyword void are also represented as Class objects.
> 5. Class has no public constructor.Instead Class objects are constructed automatically by the Java Virtual Machine as classes are loaded and by calls to the defineClass method in the class loader.

其实里面主要说了这么几件事：

#### 1.class 对象代表类和接口

这个比较好理解其实我们可以看一个例子就能明白：

```
Class clazz = Long.class;
```

前面是 Class 类型后面就是一个具体的变量，那么为什么我们没有 new 这个对象就能获取到呢？ 其实在这里我们使用了 Long 这个类，那么这个类就会被 jvm 加载之后在 jvm 中形成了他所对应的 class 对象，我们在赋值的时候直接从 jvm 取到即可，这里我们也比较好理解这个类的对象其实在内存中只有一份，也就是单例的，这是因为没必要存在多份，他只是一个类的一个内存映射。

#### 2.枚举是类注解是接口

注解是接口是头一回听说，但是他的定义方式比较能说明问题，应该是在编译期做了转换吧。

#### 3.同维同类型数组class 对象相同

这个也就是说他们的类型和维数一致才可以，具体看一个例子：

```
int[] arr1=new int[1];
int[] arr2=new int[2];
Assert.check(arr1.getClass()==arr2.getClass());
```

#### 4.基本类型也有 class 对象

#### 5.class 对象不用创建

class 对象是没有公共的构造方法，也没有静态的 getInsteance() 所以说没办法从外部构造方法，是直接的由 jvm 在加载类的时候调用了类加载的 defineClass() 方法完成 class 对象的创建的。

### 3.class 对象获取

对于 class 对象的获取方式有三种：

1. 使用类型属性

   ```
   Class clazz = Long.class
   ```

2. 调用对象的 getClass() 方法

   ```
   List list = new ArrayList();
   Class clazz = list.getClass();
   ```

3. 使用forName()静态方法

   ```
   try {
       Class.forName("com.apple.concurrent.Dispatch");
   } catch (ClassNotFoundException e) {
       e.printStackTrace();
   }
   ```

## 二、关于反射

先写一个简单的 javaBean 然后对这个 bean 就进行一些操作：

```java
package grammer;

public class User {
    private String name;
    private int age;
    public User() {
    }
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public int getAge() {
        return age;
    }
}
```

然后我们主要讨论关于对于属性、方法、构造器的获取以及使用。

#### 1.通过 class 对象创建对象

```java
Class<User> userClass = User.class;
User userInstance = userClass.newInstance();
```

#### 2. 获取所有的构造方法

注意以下两个都是获取构造器列表，但是前面的是可以获取到私有的构造器，而后者并不可以，其他的都是同理的对于方法和字段来说

```java
Constructor<?>[] constructors = userClass.getDeclaredConstructors();
Constructor<?>[] constructors1 = userClass.getConstructors();
```

#### 3. 获取所有的属性

```java
Field[] fields = userClass.getDeclaredFields();
```

#### 4.获取所有的方法

```java
Method[] methods = userClass.getDeclaredMethods();
```

#### 5.获取某个私有属性

这个要注意对于私有属性，只有用 `fieldName.setAccessible(true);` 设置访问权限后 jvm 再运行的时候不会做权限验证 也就不会报错了

```java
Field fieldName = userClass.getDeclaredField("name");
fieldName.setAccessible(true);
System.out.println(fieldName.getName());
// 设置值需要调用那个对应的对象  需要指明对象才可以
fieldName.set(userInstance,"lwen");
```

#### 6.获取某个方法并调用

这个地方需要指明我们调用的参数的具体类型，否则会由于我们的 java 重载机制而出现模糊不清的情况。当然如果是私有方法我们在调用之前必须要指定他们的访问权限，否则也会调用失败，对于构造方法也是这样的。

```java
Method setNameMethod = userClass.getDeclaredMethod("setName", String.class);
//调用方法也是需要指明对象
setNameMethod.invoke(userInstance,"lwen");
```

#### 7.反射性能问题

反射的性能一般比普通调用的性能要低很多，一般一个方法的执行时间应该是 30 倍的关系，我们希望能够加快反射的速度可以使用跳过安全检查 setAccessible(true) 来提升效率，一般相对使用反射来说的话是提升 4 倍效率。另外我们为了操作 class 我们还可以使用一些第三方的操作字节码的类库比如操作底层字节码的 ASM ，以及基于代码的 javassist 和 GCLIB 等等。

这里说一下关于 javassist 的使用，如何使用它来生成一个 class 文件。

```java
package grammer;
import javassist.*;
import java.io.IOException;
public class JavassistDemo {
    public static void main(String[] args) throws CannotCompileException, NotFoundException, IOException {
        ClassPool pool = ClassPool.getDefault();
        CtClass ctClass = pool.makeClass("out.Empl");

    //    创建属性
        CtField int_age = CtField.make("private int age;", ctClass);
        ctClass.addField(int_age);

    //    创建方法
        CtMethod method = (CtMethod) CtMethod.make("public int getAge(){return this.age;}", ctClass);
        ctClass.addMethod(method);

    //  创建构造器
        CtConstructor constructor = new CtConstructor(new CtClass[]{CtClass.intType}, ctClass);
        constructor.setBody("{this.age=age;}");
        ctClass.addConstructor(constructor);

    //    生成
        ctClass.writeFile();

    }
}
```