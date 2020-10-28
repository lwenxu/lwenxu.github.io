---
title: Java泛型
date: 2017-08-09 15:49:28
tags: 范型
categories:
 -Java
---
1.java泛型及就是在jdk1.5之后出现的一个新的安全机制
  我们发现在集合框架中我们可以放入任何的元素，然而这样做并没有任何意义，绝大多时候我们是默认我们
  知道这个容器需要存放什么样的内容，但是用户的输入是不安全的如果他们输入了各种类型然后我们只对某些类型
  进行了处理显然到时候运行时必然报错
  所以为了解决这个问题，类似于数组的解决方式给集合限定了类型使用尖括号来限定，当然包括Iterator
  他的好处就是安全
<!--more-->
2.comparable接口和comparator都可以使用泛型，在使用泛型以后里面的内容就无需在强转直接写泛型的类型即可

3.泛型不仅仅是安全这一说，还有就是以前我们在不确定一个函数的具体接受对象的类型的时候我们都是使用object对象
  来接受，然后向下转型。但是如果有了泛型我们就可以直接定义泛型类或者泛型方法。
  1）.泛型类就是把泛型定义在类上面这样的话类中只要引用了泛型的地方他们的类型都一致例如集合框架，写在类名后类似于接受参数
      泛型的确定就是在类实例化的时候
  2）.当然有时候类中的函数想操作的独享并不是类上定义的那个类型我们可以把泛型定义在函数上，写在返回值的前面
      类似于参数一样，但是静态方法是无法使用定义在类上面的泛型因为类的泛型在确定类型的时候都是在类的实例化
      而函数的泛型则是自动识别无需程序员做任何操作
  3）.接口也可以定义泛型，然后类在实例化的时候可以明确泛型也可以不指定泛型然后还是一个泛型类
  三个知识点的代码分别如下：
  ```java
        /**
         * Author: lwen
         * Date: 2017/07/12
         * Description:定义在类上面的泛型
         */

        class Worker{

        }

        class Student{

        }

        class Tool<T>{
            private T t;

            public T getT() {
                return t;
            }
            public void setT(T t){
                this.t=t;
            }
        }

        public class GenericDemo01 {
            public static void main(String[] args) {
                Tool<Worker> tool=new Tool<>();
                tool.getT();
            }
        }

  ```

  ```java
    /**
     * Author: lwen
     * Date: 2017/07/12
     * Description:定义在函数上的泛型
     */
    class Out<T>{
        <M> void show(M m){
            System.out.println(m);
        }
        void print(T t){
            System.out.println(t);
        }

        static <O> void prints(O o){
            System.out.println(o);
        }
    }

    public class GenericDemo02 {
        public static void main(String[] args) {
            Out<String> out=new Out<>();
            out.print("string");  //只能传入string因为这个是类上面的泛型
            out.show(3);  //任意类型的因为这个事函数上面的泛型
        }
    }

  ```

  ```java
    /**
     * Author: lwen
     * Date: 2017/07/12
     * Description:  定义在接口上的泛型
     */
    interface Inter <T>{
        void print(T t);
    }

    class InterImp implements Inter<String>{
        @Override
        public void print(String s) {
            System.out.println(s);
        }
    }

    class InterImp1<T> implements Inter<T>{
        @Override
        public void print(T t) {

        }
    }

    public class GenericDemo03 {
        public static void main(String[] args) {

        }
    }

  ```

4.泛型的高级应用就是泛型限定：
  泛型限定：
  1）.当我们要传入的某个参数不确定的时候我们可以使用定义在函数上的泛型，当然还有一种更方便的方法就是
      使用？占位符
  2）.我们会发现？占位符能接受的类型的范围太大，无论是什么都可以接受，但是我们希望我们只能接受
      某一类对象的子对象或者某对象的父类对象
      有向下类型限定和向上类型限定：
      向上：  ? extends className
      向下:   ? super className

  ```java
/**
 * Author: lwen
 * Date: 2017/07/12
 * Description: 向下限定也就是在comparator接口中他的泛型限定其实就是 ？ super T  这个T就是TreeSet的T
 *              那么也就是说comparator中可以放T的父类
 *
 *              显然如果说我们要给TreeSet传入比较器的话我们肯定需要给每一个特定的类型的TreeSet给一个比较器
 *              但是由于在定义比较器接口的时候人家给的就是某个类型的父类这样的话我们构造一个父类的比较器就能搞定
 *
 */
class Person{
    private int age;

    Person(int age){
        this.age=age;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

class Students extends Person{
    Students(int age){
        super(age);
    }
}

class Workers extends Person{
    Workers(int age){
        super(age);
    }
}


public class GenericDemo04 {
    public static void main(String[] args) {
        TreeSet<Workers> ts1=new TreeSet<>(new Comp());
        ts1.add(new Workers(12));
        ts1.add(new Workers(13));
        ts1.add(new Workers(10));
        for (Workers ws :
                ts1) {
            System.out.println("ws--------"+ws.getAge());
        }

        TreeSet<Students> ts2=new TreeSet<>(new Comp());
        ts2.add(new Students(12));
        ts2.add(new Students(9));
        ts2.add(new Students(17));
        for (Students stu :
                ts2) {
            System.out.println("student---"+stu.getAge());
        }
    }
}

class Comp implements Comparator<Person>{
    @Override
    public int compare(Person o1, Person o2) {
        return new Integer(o1.getAge()).compareTo(o2.getAge());
    }
}
  ```

  ```java
/**
 * Author: lwen
 * Date: 2017/07/12
 * Description:
 */

class A{

}
class B extends A{

}
class C extends B{

}

public class GenericDemo05 {
    public static void main(String[] args) {
        A a=new A();
        B b=new B();
        C c=new C();
        ArrayList<A> ala=new ArrayList<>();
        ala.add(a);
        ArrayList<B> alb=new ArrayList<>();
        alb.add(b);
        ArrayList<C> alc=new ArrayList<>();
        alc.add(c);

        print(ala);
//        printSub(ala); error
        printSup(ala);
        printSup(alb);
//        printSup(alc);  error
        printSub(alc);
        printSub(alb);

    }

    public static void print(ArrayList<?> al){
        Iterator<?> it=al.iterator();
        while (it.hasNext()){
            System.out.println(it.next());;
        }
    }
    public static void printSub(ArrayList<? extends B> al){
        Iterator<?> it=al.iterator();
        while (it.hasNext()){
            System.out.println(it.next());;
        }
    }
    public static void printSup(ArrayList<? super B> al){
        Iterator<?> it=al.iterator();
        while (it.hasNext()){
            System.out.println(it.next());;
        }
    }
}

  ```
