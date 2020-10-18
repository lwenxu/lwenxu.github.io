---
title: 数据结构Queue
date: 2018-03-02 16:24:11
tags: 数据结构

---

​	栈和队列其实是相同的，只是名字不一样 入栈换成了入队（enqueue），出栈换成了出队（dequeue）。语义 是不同的。入队操作向队尾添加元素，而出队操作从 队首移除元素。<!--more-->

​	现在，队列的链表表示中 我们需要维护两个指针引用。一个是链表中的第一个 元素，另一个是链表最后一个元素。插入的时候我们在 链表末端添加元素，而不是在链表头。移除元素的时候 不变，依然从链表头取出元素。那么这就是出队操作的实现 和栈的出栈操作的代码是一样的。保存元素，前进指针 指向下一个节点，这样就删除了第一个节点，然后返回该元素。一模一样 添加节点，或者入队操作时，向链表添加新节点。我们要把它放在链表末端 这样它就是最后一个出队的元素。首先 要做的是保存指向最后一个节点的指针，因为我们需要 将它指向下一个节点的指针从null变为新的节点。然后给 链表末端创建新的节点并对其属性赋值，将旧的指针 从null变为指向新节点。

​	那么用数组实现呢？用可调大小的数组实现并不难 。我们维护两个指针，分别指向队列中的 第一个元素和队尾，即下一个元素要加入的地方 那么对于入队操作在tail指向的地方加入新元素，出队操作移除 head指向的元素。棘手的地方是一旦指针的位置超过了数组 的容量，必须重置指针回到0，这里需要多写一些代码 而且和栈一样实现数据结构的时候你需要加上调整容量的方法 。下面给出完整的实现。

```java
package Queue;

/**
 * 同 Stack 我们遇到的问题就是调整队列大小的问题，以及处理碎片空间的问题
 * 调整大小我们依然使用倍增法，碎片空间就涉及到了循环队列的处理方法
 */
public class QueueArray {
    private int[] arr;
    private int head = 0;
    private int tail = 0;

    public QueueArray() {
        arr = new int[20];
    }

    private void resize(int size) {
        int[] temp = new int[size];
        System.arraycopy(arr, 0, temp, 0, arr.length);
        arr = temp;
    }

    public boolean isEmpty() {
        return head == tail;
    }

    public void enqueue(int num) {
        if (isFull()) {
            resize(arr.length * 2);
        }
        tail = (++tail) % arr.length;
        arr[tail] = num;
    }

    public int dequeue() {
        return arr[++head % arr.length];
    }

    public boolean isFull() {
        return (tail + 1) % arr.length == head;
    }
}

```

