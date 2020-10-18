---
title: Java集合框架Map
date: 2017-08-09 14:38:12
tags: 集合框架
categories:
  -Java
---
1. map集合是一对一对的存放，而且要保证键名的唯一性。

2. map的共性方法：  
    1.添加：
        put(K key,V value)
        putAll(K key,V value)
    2.删除：
        clear()
        remove(Object key)
        <!--more-->
    3.判断：
        isEmpty()
        constrainKey(object key)
        constrainValue(object value)
    4.获取：
        get(Object key)
        size()
        values()

```java
public class MapDemo01 {
    public static void main(String[] args) {
        HashMap<String,String> hashMap=new HashMap<>();
        //当第一次存放元素的时候这个键不存在所以返回的是null
        //第二次再存放同一个键的值的时候不会报错而是返回的改建原有的值这个和Set以及collection是不一样的
        System.out.println(hashMap.put("01","zhansgan01"));
        System.out.println(hashMap.put("01","zhansgan02"));
        hashMap.put("03","zhansgan03");
        hashMap.put("04","zhansgan04");

        System.out.println(hashMap.get("01"));
        Collection<String> collection=hashMap.values();
        System.out.println(collection);

        System.out.println(hashMap.containsKey("02"));
        System.out.println(hashMap.containsValue("zhansgan03"));
        //remove返回的value
        System.out.println(hashMap.remove("03"));
    }
}
```

3.Map的三个常用的小弟：
* hashTable   底层是hash表不可以使用null作为键或者值，线程同步  jdk1.0
* hashMap     底层是hash表可以使用null作为键或者值，线程不同步  jdk1.2  效率 更高
* TreeMap     底层是二叉树，可对map的键进行排序
其实set的底层都是Map

4.两个特别重要的函数keySet和entrySet
  首先是keySet，其实在Map集合中里面的结构是双列结构也就是；两个Set构成的，里面分别存放
  键值的引用，而keySet就是只把键取出来，类似于values取出所有的值，当吧所有的键都取出来的时候就意味着我们
  具备了所有的值，这时候我们就可以遍历Map了，map本身没有遍历的方法
  ```java
    public class KeyMapDemo {
        public static void main(String[] args) {
            HashMap<String,String> hashMap=new HashMap<>();
            hashMap.put("1","zs1");
            hashMap.put("2","zs2");
            hashMap.put("3","zs3");
            hashMap.put("4","zs4");
            Set<String> keySet =hashMap.keySet();
            Iterator<String> it=keySet.iterator();
            for (;it.hasNext();){
                String key=it.next();
                System.out.println(hashMap.get(key));
            }

        }
    }
  ```

5.entrySet则是直接返回了这个Map的映射关系，仅仅是关系而不是元组。
  这个关系的类型是Map.Entry类型的里面有getValue和getKey方法
  Entry其实是Map接口的内部接口就像是内部类一样，这是内部接口
  并且这个接口是静态的

  ```java
    public class EntrySet {
        public static void main(String[] args) {
            Map<String,String> map=new HashMap<>();
            map.put("1","s1");
            map.put("2","s2");
            map.put("3","s3");
            //返回的还是set集合只里面的类型是Map.Entry类型的
            Set<Map.Entry<String, String>> mapEntry=map.entrySet();
            for (Map.Entry me :
                    mapEntry) {
                System.out.println(me.getKey()+"--------"+me.getValue());
            }
        }
    }

    //类似于种东西
    interface Map{
        static interface Entry{
            abstract Object getKey();
            abstract Object getValue();
        }
    }
```


5.一个小demo
 ```java
    /**
     * Created by lwen on 2017/7/13.
     * 把一个字符串里面的字符的数目统计一下，，并且按照字符的顺序显示
     */
    public class MapDemo03 {
        public static void main(String[] args) {
            String str=charCount("aasdfasadcl");
            System.out.println(str);
        }

        static String charCount(String str){
            char[] chs=str.toCharArray();
            TreeMap<Character,Integer> treeMap=new TreeMap<>();
            int count=0;
            for (char ch:chs){
                if (treeMap.get(ch)!=null){
                    count=treeMap.get(ch);
                }
                count++;
                treeMap.put(ch,count);
                count=0;
            }
            StringBuilder sb=new StringBuilder();
            Set<Map.Entry<Character,Integer>> set=treeMap.entrySet();
            for (Map.Entry<Character,Integer> em:set){
                sb.append(em.getKey()).append("(").append(em.getValue()).append(")");
            }
            return sb.toString();
        }

    }
 ```

 7.集合框架总共有两大部分分别就是：
 <pre>
    ---Collection
        |--List  ArrayList LinkedList
        |--Set  HashSet  TreeSet
    ---Map
        |--HashTable
        |--HashMap
        |--TreeMap
</pre>        
另外还有两个常用的类就是两个工具类Collections,Collections里面提供了:
* 1) 一个sort方法他是专门对Collection排序的
* 2)还有，max方法用法和sort方法一模一样
* 3）binarySearch就是二分法查找元素可以自己实现一个试试
* 5) replaceAll(list,oldEle,newEle)
* 6)reverseOrder()强行翻转一个比较器，当不传入参数的时候强行逆转默认的自然排序，传入一个比较器则是逆转比较器

还有一个工具类就是Arrays这个类里面的方法主要就是为了操作数组的：
* 1）sort
* 2）fill   全部替换
* 3）shuflle  乱序
* 4）asList  把数组转化为List集合，但是注意这个list集合不能进行增删操作否则异常，还有就是如果数组是一个对象类型的数组，转化为list最后就是对象的list
而数组时基本数据类型则会把数组当做一个list元素


    集合变数组，集合变数组使用的就是Collection接口里面的共性方法，toArray(type [collection.size()])
    一般转过来的数组的长度最好就是集合的长度否则他会再开辟一个数组，而小于等于的情况会直接使用原来的数组

    数组变成集合是为了扩展数组的操作让数组作为集合来处理例如contains数组要判断元素是否存在必须遍历而集合就不用
    把集合变成数组是为了限制操作，也就是让数组无法操作，例如再返回的时候最好返回数组

    可变参数的方法，就是在函数声明或者定义的时候使用一个数组当做参数，但是我们传参的时候传入的并不是一个数组而是一般状况的传值，个数不限
    但是这个函数的固定参数必须放在可变参数的前面
    void fun(string str,int... arr)   后面就是可变参数个数不限但是类型有限制
```java
    class StrLenComparator implements Comparator<String>{
        public int compare(String s1,String s2){
            if (s1.length()>s2.length()){
                return 1;
            }else if(s1.length()<s2.length()){
                return -1;
            }
            return s1.compareTo(s2);
        }
    }

    public class CollectionsDemo {
        public static void main(String[] args) {
            List<String> list=new ArrayList<>();
            list.add("aaa");
            list.add("ab");
            list.add("adc");
            list.add("dd");
            list.add("cd");
            list.add("efcasdc");
            Collections.sort(list);
            for (String string:list){
                System.out.println(string);
            }

            Collections.sort(list,new StrLenComparator());
            for (String string:list){
                System.out.println(string);
            }
        }
    }

  ```


10，静态导入：
    当一般的我们import一个包的时候我们导入的是包中的类
    而我们使用静态导入的时候import static则是导入类中的所有的静态成员，无论是属性还是方法
  ```java
import static java.util.Arrays.*;
import static java.util.Collections.*;

public class StaticImport {
    public static void main(String[] args) {
        int[] arr=new int[11];
        for (int i=10,j=0;i>0;i--,j++){
            arr[j]=i;
        }
        sort(arr);   //可以直接使用Arrays中的方法而不用写Arrays.sort但是如果同一个类中名字冲突的话需要加上
        for (int ele:arr){
            System.out.println(ele);
        }
    }
}

```

9.HashMap与HashTable的区别：
    1）hashmap不是线程同步的，而hashtable则是线程同步的，也就是前者效率较高后者效率较低，安全性相反
    2）hashmap他是Map的子类。而hashtable则是dictionary的子类
    3）hashMap的键值均可以为null，hashtable的键值均不可为null
10.hashTable的使用。
    一般来说hashtable使用的很少，这里也有一个常用的子类就是hashTable的常用子类Properties
    类，这个类的作用主要就是读，写，加载配置文件，这个容器里面只允许存放字符串，不论是键值均如此
    存放的键值可以转化为普通的配置文件也可以转化为xml文件，之后下一次还可以使用load进行加载
```java
public class HashTableDemo {
    public static void main(String[] args) throws IOException{
        Properties properties=new Properties();
        properties.setProperty("user","lwen");
        properties.setProperty("pwd","123");
        properties.store(new FileWriter("info.properties"),"用户配置文件");
        properties.store(new FileOutputStream("info_1.properties"),"用户配置文件");  //可以字符流也可以字节流
        properties.storeToXML(new FileOutputStream("info.xml"),"xml");  //这个只能使用字节流对象  xml没有编码
        System.out.println(properties.getProperty("user","not this "));  //如果找不到这个属性，则会返回默认值
        properties.load(new FileReader("info.properties"));  //加载配置文件是直接加载到这个对象里面，而不是打印


    }
}
```

11.内存的中的变量一般分为4种，这四种对于垃圾回收机制来说是不一样的：
    强引用：无论什么时候都不会被gc回收
    软引用：可能会被gc回收，只有在jvm内存不足的时候才会被回收
    弱引用：当gc运行的时候就被回收
    虚引用：类似于无引用，主要跟踪对象被回收的状态，不能单独使用，一般和引用队列一起使用
    这里重点说一下强和弱引用，所谓强引用一般来说就是一些字符串常量，他们长期驻留在内存，用于共享
    当我们在一个hashMap中存放了很多内容的时候，有时候我们需要清除一些没用的数据，我们就可以运行gc
    因此就诞生了WeakHashMap这样的话，这里面的弱类型的就会被干掉
```java
public class WeakHashMapDemo {
    public static void main(String[] args) {
        WeakHashMap<String,String> weakHashMap=new WeakHashMap<>();
        //这两个事在常量池中的常量，所以放到weakmap中也不会被gc
        weakHashMap.put("zhangsan0","122");
        weakHashMap.put("zhangsan1","123");
        //这两个则是两个对象在堆内存中，放到这里了以后，肯定被gc
        weakHashMap.put(new String("zhangsan2"),"102");
        weakHashMap.put(new String("zhangsan3"),"1245");
        //运行垃圾回收机制之前
        Set<Map.Entry<String,String>> me=weakHashMap.entrySet();
        for (Map.Entry<String, String> stringStringEntry : me) {
            System.out.println(stringStringEntry.getKey());
            System.out.println(stringStringEntry.getValue());
        }

        //运行垃圾回收机制
        System.gc();
        System.runFinalization();

        //垃圾回收之后
        System.out.println("=========================");
        Set<Map.Entry<String,String>> me1=weakHashMap.entrySet();
        for (Map.Entry<String, String> stringStringEntry : me1) {
            System.out.println(stringStringEntry.getKey());
            System.out.println(stringStringEntry.getValue());
        }
    }
}
```

12.IdentityHashMap这个容器是以键的地址作为比较的对象
   注意是地址而不是其他的，例如说如果是一个字符串常量，显然这个只要内容相同就重复了，而如果是字符串对象就是不同的地址值，哪怕他们的内容一样
```java
public class IdentifiedHashMapDemo {
    public static void main(String[] args) {
        IdentityHashMap identityHashMap=new IdentityHashMap();
        //前两个是字符串常量，在常量池中，地址是一样的
        identityHashMap.put("a","21");
        identityHashMap.put("a","22");
        //后面两个是不同的对象，因此他们都被存入
        identityHashMap.put(new String("a"),"23");
        identityHashMap.put(new String("a"),"24");
        System.out.println(identityHashMap.size());
    }
}
```

13.对于集合框架中有时候遇到多线程，我们又需要使用那些非同步的集合的时候我们必须手动的去同步这些集合，以及希望集合不可修改的时候也需要加一些限定
```java
public class SyncDemo {
    public static void main(String[] args) {
        List<String> list=new ArrayList<>();
        List<String> list1=Collections.synchronizedList(list);
//        这样一来list1就是一个同步的arraylist
//        其他的几个容器类似
        List<String> list2=Collections.unmodifiableList(list1);
        //这样一来list2就是一个不可修改的容器也就是此时的容器不再支持增删操作

    }
}
```
