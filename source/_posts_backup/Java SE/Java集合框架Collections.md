---
title: Java集合框架Collections
date: 2017-08-09 14:38:06
tags: 集合框架
categories:
  -Java
---
### 1.基本介绍：
1. 集合就是存放对象的，他比数组好的一点就是他一开始不清楚自己长度
    容器一般是分为很多种的，很多的容器在一起然后进过断的抽象和抽取就成了一个体系，我们称之为集合框架
    我们看体系首先是看顶层的容器，他是底层的容器都有的特性，然后在逐步求精
    最顶层的我们称之为collection 在util包中的
<!--more-->
2.  在collection中分为两个比较常用的子接口分别是list和set。
    list是类似于数组的那种，也就是集合元素可重复，有序有脚标。
    set则为无序的，所以集合元素不可重复，不可脚标查找
    以下是List的通用方法：
    * 添加元素：
    add(index,data)
    * 删除元素：
    remove(index)
    * 修改元素：
    set(index,data)
    * 获取元素：
    get(index)   配合for循环
    * 迭代器
    indexOf（data）
    subList（start,end）

    在list中有一个特殊的迭代器，其他的集合都没有只是list有叫做ListIterater
    这个迭代器比一般的迭代器会多非常多的功能

    另外注意的一点就是在使用iterater进行list迭代的时候，不能够使用集合的方法对集合进行增删改查的操作
    否则就会出现一个运行时异常，主要原因就是同时操作一个集合导致不合法，类似于同时IO同一块数据块
    因此在迭代的过程中只能使用迭代器提供的操作集合的方法
    或者不使用迭代器，直接使用原生的for循环，然后直接就可以使用迭代器的方法进行对集合的操作

3.  add方法接受的参数类型为object以便于接受任意类型的参数
    集合中存放的是对象的地址而不是对象本身
    对象可以直接被打印

4.  List可以分为三种，但是常用的只有两种，他们之间的主要区别就是底层的数据及结构不一样。
 * **ArrayList**  底层的实现使用的是数组，也就是说类似于数组构成的线性表，查询快增删慢，而且他是不同步的
 * **LinkedList**  底层使用的是链表，那么就会出现查询慢，增删快
 * **Vector**  底层和ArrayList完全一样，只不过vector就是jdk1.0出现的那时候还没有集合框架，后来有了集合框架被分到List里面，目前被ArrayList代替因为他是同步的速度慢，我们都是用Arraylist然后自己写锁来手动同步

5.  arraylist和vector之间区别还有就是他们们的迭代方式可以有不同，由于vector是最先出来的，所以说他一开始用的并不是
    iterator迭代器而是枚举enumeration他和迭代器很相似，目前由于枚举不好记就用iterator。另外arraylist和vector使用的
    是变长数组，也就是本来都是固定长度10个元素，然后如果查过十个以后ArrayList使用的是50%的增长也就是会变成15个，二vector则是
    直接100%增长20个
    枚举的代码如下：
    ```java
        public class Enmu {
            public static void main(String[] args) {
                Vector v=new Vector();
                v.add("1");
                v.add("2");
                v.add("3");
                Enumeration e=v.elements(); //定义枚举
                while (e.hasMoreElements()){  //循环遍历
                    System.out.println(e.nextElement());
                }
            }
        }
    ```

6.  LinkedList中除了一般的List的通用方法还有他自己特有的方法，而且比较重要

    * addFirst
    * addLast
    * getFirst    获取元素但是不删除元素
    * getLast
    * removeFirst   获取元素而且删除元素，但是如果给的是一个空的链表列表使用此方法会产生异常因此有了以下替代方法
    * removeLast
    * offerFirst  添加
    * offerLast
    * peekFirst 获取
    * peekLast
    * pollFirst 删除
    * pollLast   空链表列表也不会有异常而是直接返回null

    代码如下：
    ```java
        public class LinkedList_5 {
            public static void main(String[] args) {
                LinkedList list=new LinkedList();
                list.addFirst("1");
                list.addFirst("2");
                list.addFirst("3");
                System.out.println(list.getFirst());
                System.out.println(list.removeFirst());

                LinkedList list1=new LinkedList();
                list1.offerFirst("1");
                list1.offerFirst("2");
                list1.offerFirst("3");
                System.out.println(list1.peekFirst());
                System.out.println(list1.pollFirst());

            }
        }
    ```


7.使用LinkedList实现堆栈和队列结构也就是他所特有的addFirst   addLast  以及remove方法的使用
     只是要注意在LinkedList里面添加和删除的元素都是object而不是一般的对象，应该说
     所有的集合框架里面的东西都是接受object对象的  所以在写具体的函数的时候注意一下


8.使用ArrayList写几个小程序，第一个就是使用ArrayList去处重复元素，其中的元素就是字符串，而第二个则是
去除的某些自定义对象。第二个程序也揭示了对于List集合中的元素的比较的接口就是contains他底层是调用
的对象的equals方法，对于remove也是底层调用了equals方法从而进行比较和删除。所以说重点在于重写
equals方法。
另外在迭代器中每使用一次next方法必须要进行一次hasNext的判断否则很有可能出现找不到元素的情况
例如一个hasNext里面有两个next方法，而元素又为奇数个时候会抛异常
```Java
        class Person{
            private String name;
            private int age;
            Person(String name,int age){
                this.name=name;
                this.age=age;
            }
            String getName(){
                return this.name;
            }
            private int getAge(){
                return this.age;
            }
            public boolean equals(Object object){
                if (!(object instanceof Person)){
                    return false;
                }
                Person person=(Person)object;  //注意这个地方必须要强转否则会出现下面的person无法调用方法
                return this.name.equals(person.getName()) && this.age==person.getAge();
            }
        }
        public class ArrayList_7 {
            private static ArrayList<Person> singleElement(List<Person> al){
                ArrayList<Person> newAl=new ArrayList<Person>();
                Iterator<Person> it=al.iterator();
                while (it.hasNext()){
                    Person tmp= it.next();  //iterator返回的是object所以必须要强转
                    if (!newAl.contains(tmp)){
                        newAl.add(tmp);
                    }
                }
                return newAl;
            }

            public static void main(String[] args) {
                ArrayList<Person> al=new ArrayList<>();
                al.add(new Person("zhang01",1));
                al.add(new Person("zhang02",2));
                al.add(new Person("zhang03",1));
                al.add(new Person("zhang03",1));
                al=singleElement(al);
                for (Person per : al) {   //可以使用这种更好的语言结构来实现迭代省去了iterator迭代 也不用考虑向下类型转换
                    System.out.println(per.getName());
                }
            }
        }
```
9.Set中存放的元素都是无序的并且里面的元素都是不可重复的，显然如果只存放字符串的话很容易就知道是否重复
  也就无序我们自己去判断是否重复了。
  set中的公共方法就是集合框架中共有的方法，重要的还是他的子类，这里有两个就是hashSet和TreeSet

  而存放一般的自定义对象的时候我们发现如果想要某些属性一致的对象作为重复对象
  的话hashSet自身是做不到的，所以我们需要了解hashSet的底层层放原理，hashSet底层就是hash表，在存放元素的时候
  首先来判断存放的元素的hashCode值是否一样也就是调用他们的hashCode方法，注意hashCode是object对象的方法，所以
  所有的对象都有此方法，另外如果他们的hashCode是一样的然后就调用他们的equals方法.不一样则就存进去一样则被踢出去
  那么说白了hashSet底层判断是否为重复元素做了两件事第一个就是判断他们的hashCode第二个就是equals方法
  如果要自定义对象如何存放就要重写这两个方法，但是重写的时候一定要注意他们的参数列表否则肯定不会生效，hashCode
  一般来说也尽量不要让不同的对象的hashCode一致造成多余的比较

  对于元素判断是否存在和删除元素都是hashCode和equals方法

  下面是hashSet的代码示例：
  ```java
    class People{
        private String name;
        private int age;

        People(String name,int age){
            this.name=name;
            this.age=age;
        }

        String getName(){
            return this.name;
        }

        private int getAge(){
            return this.age;
        }

        public int hashCode(){
            return name.hashCode()+age;
        }

        public boolean equals(Object object){
            if (!(object instanceof People)){
                return false;
            }
            People person=(People)object;  //注意这个地方必须要强转否则会出现下面的person无法调用方法
            return this.name.equals(person.getName()) && this.age==person.getAge();
        }

    }

    public class HashSet_8 {
        public static void main(String[] args) {
            HashSet<People> hs=new HashSet<People>();
            hs.add(new People("1",11));
            hs.add(new People("2",11));
            hs.add(new People("1",11));
            for (People pe:hs){
                System.out.println(pe.getName());
            }
        }
    }
   ```

10.treeSet是在集合中的元素会自动排序，如果是字符串什么的他们都可以自动比较，因为字符串是已经实现了Compareable接口
   但是如果要存放一般的元素对象的时候注意一定要让改类实现compareable接口，因为此接口会让类强制具有比较性
   然后复写此接口中的compareTo方法，大于返回正数等于为零小于则为负，这里要注意如果有多个排序元素的话然后在比较
   的时候相等条件判断要注意对其他排序元素的判断，否则会造成某个条件相等但是并不是同一个元素而无法存入

   ```java
class Student implements Comparable{
    private String name;
    private int age;

    Student(String name,int age){
        this.name=name;
        this.age=age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public int compareTo(Object o) {
        if (!(o instanceof Student)){
            throw new RuntimeException("not same type");
        }
        if (this.age>((Student) o).age){
            return 1;
        }else if (this.age==((Student) o).age){
            return this.name.compareTo(((Student) o).name);
            //注意多重判断，要是age一样的话他们就会被当成相同元素而无法插入  string类已经实现了comparable接口
            //其实java中很多类都实现了comparable接口，让类具有可比性
        }
        return -1;
    }
}

public class TreeSet_9 {
    public static void main(String[] args) {
        TreeSet<Student> ts=new TreeSet<>();
        ts.add(new Student("ab",1));
        ts.add(new Student("kb",3));
        ts.add(new Student("mb",3));
        ts.add(new Student("am",7));
        for (Student stu :
                ts) {
            System.out.println(stu.getName()+"--------"+stu.getAge());
        }
    }
}
```

11.TreeSet底层的数据结构就是二叉树，元素的排列方式就是第一个进入的元素作为根节点然后按照compareTo方法
   返回的值来判定是左孩子还是右孩子，这里就是如果说compareTo方法返回的正数则右孩子,负数为左孩子相等就说明
   是同一个元素不用再比较。
   所以说决定了TreeSet中的元素的重复与否就是compareTo函数的返回值
   那么这样的话我们也可以规定一个TreeSet按照放进去的顺序取出来 或者倒序取出来就是compareTo全部返回1
   或者-1即可，当然如果返回的始终为0那么最后集合中只有一个元素就是第一个元素


12.在TreeSet中除了实现comparable接口复写compareTo方法以外还有一种排序方法就是比较器，前面的comparable
   接口是让元素具有了比较性而比较器则是让集合具有了比较性这个优先级跟高，具体的方法就是在集合实例化的时候传入一个
   自定义的比较器，也就是构造方法传入一个比较器对象，这个比较器也是一个接口要实例化的话需要实现他的compare方法
   比较器也是更加常用的

   ```java

class MyComparator implements Comparator{

    public int compare(Object o1,Object o2){
        Student obj1=(Student)o1;
        Student obj2=(Student)o2;
        int num=obj1.getName().compareTo(obj2.getName());
        if (num==0){
           return new Integer(obj1.getAge()).compareTo(obj2.getAge());
        }
        return num;
    }
}

public class TreeSet_10 {
    public static void main(String[] args) {
        TreeSet<Student> ts=new TreeSet<Student>(new MyComparator());
        ts.add(new Student("dsf",11));
        ts.add(new Student("dmf",12));
        ts.add(new Student("ddf",13));
        ts.add(new Student("jf",14));
        for (Student stu :
                ts) {
            System.out.println(stu.getName()+"----------"+stu.getAge());
        }
    }
}

```
