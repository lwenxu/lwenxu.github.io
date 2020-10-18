---
title: Java面向对象基础(二)
date: 2017-08-09 11:14:38
tags: 面向对象
categories:
  -Java
---
### 1.构造器：
1. 构造函数在一个类没有写任何的构造函数的时候，系统会生成一个默认的空参数构造函数，这个构造函数的修饰符就是类的修饰符，当我们定义了一个构造函数，默认的构造函数就不存在了而不会出现重载
2. 构造函数是没有返回值的，他没有返回值不是指他就是void，因为void指的是函数的返回值为void类型，他是有返回值的。而没有返回值则是说明根本不用写，这两个有很大区别。
<!--more-->
3. 没有return语句
4. 函数的名字只能和类名完全一样
5. 构造函数是可以用私用的，但是当所有的构造函数的修饰符都是私有的话对象就无法创建了，其实在看 Java 文档的时候我们也会发现有些类的构造器真的就是私有的，这也就代表了我们没办法使用 new 关键字调用这个类的构造函数，因为他的构造函数对我们来说是透明的，而且既然他自己定义了构造函数那么默认构造函数也不会生成，哪怕他是私有的。再仔细我们会发现他必然会提供一个静态方法让我们获取这个类的实例，类似于单例模式的样子，对外提供一个接口。
6. 对象一建立构造函数就会被调用
7. 构造函数的作用主要就是给对象做初始化
8. 在类中有一种特殊的代码块叫做构造代码块，和一般的代码一样，只是在要执行的代码上添加一个花括号就行。  
&nbsp;&nbsp;&nbsp;&nbsp;这个构造代码块也是在每次构造对象的时候执行，看起来和构造函数非常类似，但是注意他和构造函数是有很大区别的  
&nbsp;&nbsp;&nbsp;&nbsp;首先按照执行顺序来说，构造代码块要是中先于构造函数执行。  
&nbsp;&nbsp;&nbsp;&nbsp;另外对于功能来说构造代码块是针对类的所有对象，而构造函数则是针对类的不同对象。因为构造函数是可以重载的而代码块则是无脑执行。
```Java
class PersonDemo{
    private String name;

    {
        System.out.println("this is construct code block !");
    }

    PersonDemo(){
        this.name="zhangsan";
    }

    PersonDemo(String name){
        this.name=name;
    }

    public void say(){
        System.out.println("name :"+this.name);
    }
}
```

### 2. Final 关键字：
1. final可以修饰类，函数，变量
2. 被final修饰的类无法再次进行继承，也就是为了避免功能被覆盖，保护封装性
3. 被final修饰的方法无法被复写
4. 被final修饰的变量只能赋值一次，就是一个常量

### 3. 内部类：
1. 内部类就是在一个类中定义了另外一个类
2. 内部类可以直接访问外部类中的成员和方法，包括私有的成员与方法   其原因在于内部类存在一个指向外部类的引用
    ：就叫做    外部类类名.this
3. 内部类是可以私有的 在私有了以后在别处无法直接创建内部类的对象 只能通过外部类来访问
4. 内部类可以是静态static的，也可用public，default，protected和private修饰。（而外部顶级类即类名和文件名相同的只能使用public和default）。
5. 内部类是一个编译时的概念，一旦编译成功，就会成为完全不同的两类。对于一个名为outer的外部类和其内部定义的名为inner的内部类。编译完成后出现outer.class和outer$inner.class两类。所以内部类的成员变量/方法名可以和外部类的相同。
6. 内部类分为以下几种：  
  1. 成员内部类：因为成员内部类需要先创建了外部类，才能创建它自己的  
  ```Java
  class Outer{
    private int x;

    public Outer(int x) {
        this.x = x;
    }

    class Inner{
        public void method(){
            System.out.println(x);
            System.out.println(Outer.this.x);
        }
    }

    public void getInner(){
        Inner in=new Inner();
        in.method();
    }
}
```
  2. 局部内部类：就可以在函数作用域类创建内部类
  3. 嵌套内部类
       嵌套内部类，就是修饰为static的内部类。声明为static的内部类，不需要内部类对象和外部类对象之间的联系，就是说我们可以直接引用outer.inner，即不需要创建外部类，也不需要创建内部类。
  4. 匿名内部类
      有时候我为了免去给内部类命名，便倾向于使用匿名内部类，因为它没有名字。例如：
      ```Java
      public void onClick(View v) {
        new Thread() {

            @Override
            public void run() {

            }

        }.start();
    }
    ```

### 4. 抽象类与抽象方法：
1. 抽象方法就是没有方法体
2. 有抽象方法的类必须声明为抽象类
3. 抽象类没有办法实例化
4. 当子类继承了抽象类，必须把抽象类的所有抽象方法都重写了才能实例化，否则他还是抽象类。

### 5.接口：
1. 当一个类中的方法都是抽象方法的时候可以把抽象类修改成接口
2. 接口中一般只能定义抽象方法和静态常量，当然如果不写这些修饰符的话java会自动给加上
3. 接口可以被类多实现，也就是说一个类可以实现多个接口  也就是前面所说的多实现
4. java中的接口是可以多继承的，也就是说如果说java支持多继承的话那么一定给加上限定条件，就是他只在接口中可以

### 6. Static 关键字：
1. 静态static可以修饰方法，修饰属性
当成员被static修饰的时候，这个成员就相当于被类的所有对象共享了，所有对象都可以访问修改
2. 静态成员并不是存放在堆栈中的而是存放在一块特殊的内存叫做『方法区』或者『数据区』，『共享区』等，并且这个地方分的很细，对象的方法也是存放在这个地方
3. 静态成员在调用的时候不仅仅是使用对象来调用还可以使用类名调用，因为静态成员就是属于类的而不是单单属于某个对象
4. 静态成员的生命周期是与类的生命周期相同
5. 静态成员是先于对象加载入内存，也就是静态属性先存在然后才有的对象

#### 1.实例变量和类变量的区别：
1. 存储位置：
    类变量存放在方法区中，而实例变量存放在堆内存中
2. 生命周期：
    类变量的生命周期是和类一样，但是实例变量的生命周期则是与实例相同

#### 2.使用static的注意事项：
1. 静态方法只能访问静态成员和静态方法。（静态的是独立于对象的，而非静态的则是依赖于对象存在的。另外静态的是先于对象载入内存）
2. 非静态方法即可以访问静态也可以访问非静态成员和方法
3. 静态方法中不可以出现this，super因为静态成员先于对象存在

#### 3.那什么时候使用static：
1. 对于静态方法就是当这个方法中不存在关于对象的成员的时候就把该方法修改成静态的
2. 对于变量则当该变量需要共享的时候就要把属性变成static

#### 4.主函数为什么是静态的？
主函数是一个特殊的函数，作为程序的入口，可被jvm调用  
public：代表访问权限是最大的  
static：代表主函数是独立于类存在的，并且该函数是在类的加载的时候被加载  
void：不用呗jvm返回一个参数  
String[] args :jvm需要的参数    

注意这个main函数除了args是可以改的以外其他都不可改，不然不可以被jvm识别
同时main是可以被重载的，但是只有仅仅满足以上格式的才能被jvm执行，
string[] 这个参数其实就是命令行传的参数
#### 4.静态代码块
```Java
//静态代码块
 static {
        //static code block
    }
```
1. 他是随着类的加载而加载，并且仅仅执行一次，因为类进入内存以后再次new对象的时候类不会再次加载
2. 他可以先于main函数执行
3. 它的作用是给类初始化
4. 但是注意当用一个类定义一个变量的时候不加载类
### 7.编译小问题
在java编译的时候如果在某个类中调用了其他的类，那么jvm则会在当前目录或者指定目录（classpath）寻找该类的class文件（编译过后的），没找到则找java文件
都没找到肯定报错  
在堆内存中的变量都是有初始值的，而在站内存中的变量在不初始化的情况下是无法参与运算的，对内存中可以

### 8.单例设计模式
单例设计模式一般来说就有两种方式，一个是懒汉式，一个是饿汉式。其中懒汉式使用的类的延时加载，但是会出现安全问题，需要使用同步代码块包裹返回的对象，并且做双重判断才可以。
```Java
package learn;
/**
 * 单例模式的第一种实现方式，也是最常用的一种叫做饿汉式
 * 他是先初始化对象
 */
class SingleHunger{
    private String name;
    private int age;
    private static SingleHunger s=new SingleHunger("lwen",19);

    private SingleHunger(String name,int age){
        this.name=name;
        this.age=age;
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

    public static SingleHunger getSingle() {
        return s;
    }

}

/**
 * 这个又成为懒汉式也叫做对象的延时加载，他则使用的比较少，因为在进行new新对象的时候可能会有两个线程同时进入临界区导致对相爱那个不唯一
 * 多一需要做同步操作，但是同步的时候又会消耗时间，下面的方式要稍微好一点一般只做了一次同步操作，还有一种解决方案就是直接在class后面添加同步关键字
 * 每次都要判断非常耗时
 */

class SingleLazy{
    private String name;
    private int age;
    private static SingleLazy s=null;

    /**
     * 使用private限定词是为了防止外部的方法直接调用本类中的构造方法生成新的对象
     * @param name
     * @param age
     */
    private SingleLazy(String name,int age){
        this.age=age;
        this.name=name;
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

    public static SingleLazy getSingle(){
        /*
        同步机制加双重判断，防止意外情况发生
         */
        if (s==null){
            synchronized (SingleLazy.class){
                if (s==null) {
                    s = new SingleLazy("lwen", 19);
                }
            }
        }
        return s;
    }
}

public class SingleModel {
    public static void main(String[] args){
        SingleHunger s1=SingleHunger.getSingle();
        s1.setAge(20);
        s1.setName("lwenxu");
        SingleHunger s2=SingleHunger.getSingle();
        System.out.println(s2.getName());


    }
}
```
### 9.模板设计模式
模板设计模式,就是在一段确定的方法中调用了一些不确定的方法。那么就把这些不确定的方法定义成抽象的方法，并且把这些不确定的方法的接口给暴露出去

```Java
abstract class codeTime{
    final public long getTime(){
        long start=System.currentTimeMillis();
        runCode();
        long end=System.currentTimeMillis();
        return end-start;
    }

    abstract void runCode();
}

class ACodeTime extends codeTime{
    void runCode(){
        for (int i=0;i<1000;i++){
            System.out.println(i);
        }
    }
}

public class Template {
    public static void main(String[] args) {
        ACodeTime time=new ACodeTime();
        System.out.println("time is :"+time.getTime());
    }
}
```
