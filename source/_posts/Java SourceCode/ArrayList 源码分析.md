---
title: ArrayList 源码分析
date: 2018-03-25 21:15:02
tags: JDK 源码分析
categories:
  - JDK 源码分析
---
# ArrayList 源码分析

> ### 1. 在阅读源码时做了大量的注释，并且做了一些测试分析源码内的执行流程，由于博客篇幅有限，并且代码阅读起来没有 IDE 方便，所以在 [github](https://github.com/lwenxu/JDKSourceCode) 上提供JDK1.8 的源码、详细的注释及测试用例。欢迎大家 star、fork ！
> ### 2. 由于个人水平有限，对源码的分析理解可能存在偏差或不透彻的地方还请大家在评论区指出，谢谢！

## 1. 结构
&emsp;&emsp;首先我们需要对 ArrayList 有一个大致的了解就从结构来看看吧.
### 1. 继承
&emsp;&emsp;该类继承自 AbstractList 这个比较好说
### 2. 实现
这个类实现的接口比较多，具体如下：

1. 首先这个类是一个 List 自然有 List 接口
2. 然后由于这个类需要进行随机访问，所谓随机访问就是用下标任一访问，所以实现了RandomAccess
3. 然后就是两个集合框架肯定会实现的两个接口 Cloneable, Serializable 前面这个好说序列化一会我们具体再说说
<!--more-->

### 3. 主要字段

```java
    // 默认大小为10
    private static final int DEFAULT_CAPACITY = 10;
    // 空数组  
    private static final Object[] EMPTY_ELEMENTDATA = {};
    // 默认的空数组  这个是在传入无参的是构造函数会调用的待会再 add 方法中会看到
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    // 用来存放 ArrayList 中的元素 注意他的修饰符是一个 transient 也就是不会自动序列化
    transient Object[] elementData; 
    // 大小
    private int size;
```
### 4. 主要方法 
下面的方法后面标有数字的就是表示重载方法
1. ctor-3 
2. get
3. set
4. add-2
5. remove-2
6. clear 
7. addAll
8. write/readObject
9. fast-fail 机制
10. subList
11. iterator
12. forEach
13. sort
14. removeIf

## 2. 构造方法分析
### 1. 无参的构造方法
&emsp;&emsp;  里面只有一个操作就是把 `elementData` 设置为 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA` 这个空数组。

```java
// 无参的构造函数，传入一个空数组  这时候会创建一个大小为10的数组，具体操作在 add 中
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```
### 2. 传入数组大小的构造
&emsp;&emsp;  这个就是 new 一个数组，如果数组大小为0就 赋值为 `EMPTY_ELEMENTDATA` 

```java
// 按传入的参数创建新的底层数组
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```
### 3. 传入 Collection 接口
&emsp;&emsp;  在这个方法里面主要就是把这个 Collection 转成一个数组，然后把这个数组 copy 一下，如果这个接口的 size 为0 和上面那个方法一样传入 `EMPTY_ELEMENTDATA` 

```java
public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            // 上面的注释的意思是说 jdk 有一个 bug 具体来说就是一个 Object 类型的数组不一定能够存放 Object类型的对象，有可能抛异常
            // 主要是因为 Object 类型的数组可能指向的是他的子类的数组，存 Object 类型的东西会报错
            if (elementData.getClass() != Object[].class)
                // 这个操作是首先new 了新的数组，然后再调用 System.arraycopy 拷贝值。也就是产生新的数组
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 传入的是空的就直接使用空数组初始化
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
 
```

&emsp;&emsp;  但是注意一点这里有一个 jdk 的 bug 也就是一个 Object 类型的数组不一定能够存放 Object类型的对象，有可能抛异常，主要是因为 Object 类型的数组可能指向的是他的子类的数组，存 Object 类型的东西会报错。 为了测试这个 bug 写了几行代码测试一下。这个测试是通不过的，就是存在上面的原因。

&emsp;&emsp;  一个典型的例子就是 我们创建一个 string 类型的 list 然后调用 toArray 方法发现返回的是一个 string[] 这时候自然就不能随便存放元素了。

```java
class A{
}

class B extends A {
}

public class JDKBug {

    @Test
    public void test1() {
        B[] arrB = new B[10];
        A[] arrA = arrB;
        arrA[0]=new A();
    }
}
```

## 3. 修改方法分析
### 1. Set 方法

&emsp;&emsp;  这个方法也很简单 ，首先进行范围判断，然后就是直接更新下标即可。

```java
    // 也没啥好说的就是，设置新值返回老值
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

### 2. Add(E e) 方法

&emsp;&emsp;这个方法首先调用了 `ensureCapacityInternal()` 这个方法里面就判断了当前的 `elementData` 是否等于 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA` 如果是的话，就把数组的大小设置为 10 然后进行扩容操作,这里刚好解释了为什么采用无参构造的List 的大小是 10 ，这里扩容操作调用的方法是 `ensureExplicitCapacity` 里面就干了一件事如果用户指定的大小 大于当前长度就扩容，扩容的方法采用了 `Arrays.copy` 方法，这个方法实现原理是 new 出一个新的数组，然后调用 `System.arraycopy` 拷贝数组，最后返回新的数组。

```java
    public boolean add(E e) {
        // 当调用了无参构造，设置大小为10
        ensureCapacityInternal(size + 1);  // Increments modCount        
        elementData[size++] = e;
        return true;
    }

    private void ensureCapacityInternal(int minCapacity) {
        // 如果当前数组是默认空数组就设置为 10和 size+1中的最小值
        // 这也就是说为什么说无参构造 new 的数组大小是 10
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // 若用户指定的最小容量 > 最小扩充容量，则以用户指定的为准，否则还是 10
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 1.5倍增长
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
### 3. Add(int index, E e) 方法
&emsp;&emsp;  这个方法比较简单和上面基本一样，然后只是最后放元素的时候的操作不一样，他是采用了 System.arraycopy 从自己向自己拷贝，目的就在于覆盖元素。 注意一个规律这里面只要涉及下标的操作的很多不是自己手写 for 循环而是采用类似的拷贝覆盖的方法。算是一个小技巧。

```java
public void add(int index, E element) {
        rangeCheckForAdd(index);
        ensureCapacityInternal(size + 1);  // Increments modCount
        // 覆盖
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```

### 4. remove(int index) 
&emsp;&emsp;同理这里面还是用了拷贝覆盖的技巧。 但是有一点注意的就是不用的节点需要手动的触发 gc ，这也是在 Efftive Java 中作者举的一个例子。

```java
public E remove(int index) {
        rangeCheck(index);
        modCount++;
        E oldValue = elementData(index);
        int numMoved = size - index - 1;
        //覆盖
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
        return oldValue;
    }

```
### 5. remove(E e)
&emsp;&emsp;  这个方法操作很显然会判断 e 是不是 null 如果是 null 的话直接采用 `==` 比较，否则的话就直接调用 `equals` 方法然后执行拷贝覆盖。

```java
public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    // 覆盖
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                // 调用 equals 方法
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
```
### 6. clear()
&emsp;&emsp;  这个方法就干了一件事，把数组中的引用全都设置为 null 以便 gc 。而不是仅仅把 size 设置为 0 。

```java
// gc 所有节点
    public void clear() {
        modCount++;
        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;
        size = 0;
    }
```

### 7. addAll(Collection e)
&emsp;&emsp;  这个没啥好说的就是，采用转数组然后 copy

```java
    // 一个套路 只要涉及到 Collection接口的方法都是把这个接口转成一个数组然后对数组操作
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
```

## 4. 访问方法分析
### 1. get
&emsp;&emsp;  直接访问数组下标。

```java
    // 没啥好说的直接去找数组下标
    public E get(int index) {
        rangeCheck(index);
        return elementData(index);
    }
    
```
### 2. subList
&emsp;&emsp;  这个方法的实现比较有意思，他不是直接截取一个新的 List 返回，而是在这个类的内部还有一个 subList 的内部类，然后这个类就记录了 subList 的开始结束下标，然后返回的是这个 subList 对象。你可能会想返回的 subList 他不是 List 不会有问题吗，这里这个 subList 是继承的 AbstractList 所以还是正确的。

```java
public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }
    // subList 返回的是一个位置标记实例，就是在原来的数组上放了一些标志，没有修改或者拷贝新的空间
private class SubList extends AbstractList<E> implements RandomAccess {
        private final AbstractList<E> parent;
        private final int parentOffset;
        private final int offset;
        int size;
        // other functions .....
     }

```

## 5. 其他功能方法
### 1. write/readObject
&emsp;&emsp;前面在介绍数据域的时候我就有标注 elementData 是一个 transition 的变量也就是在自动序列化的时候会忽略这个字段。

&emsp;&emsp;  然后我们又在源码中找到到了 `write/readObject` 方法，这两个方法是用来序列化 `elementData` 中的每一个元素，也就是手动的对这个字段进行序列化和反序列化。这不是多此一举吗？

&emsp;&emsp;  既然要将ArrayList的字段序列化（即将elementData序列化），那为什么又要用transient修饰elementData呢？

&emsp;&emsp; 回想ArrayList的自动扩容机制，elementData数组相当于容器，当容器不足时就会再扩充容量，但是容器的容量往往都是大于或者等于ArrayList所存元素的个数。

&emsp;&emsp; 比如，现在实际有了8个元素，那么elementData数组的容量可能是8x1.5=12，如果直接序列化elementData数组，那么就会浪费4个元素的空间，特别是当元素个数非常多时，这种浪费是非常不合算的。

&emsp;&emsp; 所以ArrayList的设计者将elementData设计为transient，然后在writeObject方法中手动将其序列化，并且只序列化了实际存储的那些元素，而不是整个数组。

```java
private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();
        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);
        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```
### 2. fast-fail
&emsp;&emsp;  所谓的 `fast-fail` 就是在我们进行 `iterator` 遍历的时候不允许调用 `Collection` 接口的方法进行对容器修改，否则就会抛异常。这个实现的机制是在 `iterator` 中维护了两个变量，分别是 `modCount` 和 `expectedModCount` 由于 `Collection` 接口的方法在每次修改操作的时候都会对 `modCount++` 所以如果在 `iterator` 中检测到他们不相等的时候就抛异常。

```java
private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;
        
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
   }
```
### 3. forEach
&emsp;&emsp;  这个是一个函数式编程的方法，看看他的参数 `forEach(Consumer<? super E> action)` 很有意思里面接受是一个函数式的接口，我们里面回调了 `Consumer` 的 `accept` 所以我们只需要传入一个函数接口就能对每一个元素处理。

```java
    @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        final int expectedModCount = modCount;
        @SuppressWarnings("unchecked")
        final E[] elementData = (E[]) this.elementData;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            //回调
            action.accept(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

写了一段测试代码，但是这个方法不常用，主要是 Collection 是可以自己生成 Stream 对象，然后调用上面的方法即可。这里提一下。

```java
public class ArrayListTest {

    @Test
    public void foreach() {
        ArrayList<Integer> list = new ArrayList<>();
        list.add(2);
        list.add(1);
        list.add(4);
        list.add(6);
        list.forEach(System.out::print);  //打印每一次元素。
    }
}
```
### 4. sort
底层调用了 Arrays.sort 方法没什么好说的。

```java
public void sort(Comparator<? super E> c) {
        final int expectedModCount = modCount;
        Arrays.sort((E[]) elementData, 0, size, c);
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }
```
### 5. removeIf
&emsp;&emsp;  这个和 forEach 差不多，就是回调写好了。

## 6. Vector
以上基本是把 `ArrayList` 的重要的方法和属性介绍完了，我们已经比较清楚他底层的实现和数据结构了。然后提到 `ArrayList` 自然也少不了一个比较古老的容器 `Vector` 这个容器真的和 `ArrayList` 太像了。因为你会发现他们连继承和实现的接口都是一样的。但是也会有一些不同的地方，下面分条介绍一下。

1. 在 `Vector` 中基本所有的方法都是 `synchronized` 的方法，所以说他是线程安全的 `ArrayList` 

2. 构造方法不一样，在属性中没有两个比较特殊的常量，所以说他的构造方法直接初始化一个容量为 10 的数组。然后他有四个构造方法。

3. 遍历的接口不一样。他还是有 `iterator` 的但是他以前的遍历的方法是 `Enumeration` 接口，通过 `elements` 获取 `Enumeration` 然后使用 `hasMoreElements` 和 `nextElement` 获取元素。

4. 缺少一些函数式编程的方法。

