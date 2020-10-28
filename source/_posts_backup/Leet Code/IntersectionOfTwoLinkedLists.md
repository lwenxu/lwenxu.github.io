---
title: IntersectionOfTwoLinkedLists
date: 2017-09-07 20:44:02
tags: 链表
categories:
    -LeetCode
---
<pre>
Write a program to find the node at which the intersection of two singly linked lists begins.


For example, the following two linked lists:

A:          a1 → a2
                   ↘
                     c1 → c2 → c3
                   ↗
B:     b1 → b2 → b3
begin to intersect at node c1.


Notes:

If the two linked lists have no intersection at all, return null.
The linked lists must retain their original structure after the function returns.
You may assume there are no cycles anywhere in the entire linked structure.
Your code should preferably run in O(n) time and use only O(1) memory.
Credits:
Special thanks to @stellari for adding this problem and creating all test cases.
</pre>

### 两个思路:
##### 一个就是把链表转化成一个有环链表  这样的话我们就可以当做那个有环的求入点问题了  但是人家说了不能修改链表的结构很抱歉没办法使用

##### 第二个思路就是既然是这个入点是重复的那么我们就找出重复元素即可  这样自然想到了 hash 表


```java
import java.util.HashSet;
import java.util.Set;
class IntersectionOfTwoLinkedLists_1 {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA==null||headB==null){
            return null;
        }
        ListNode circle=headA;
        ListNode fast=headB;
        ListNode slow=headB;
        while (circle.next!=null){
            circle=circle.next;
        }
        circle.next=headA;
        while (fast!=null&&slow!=null){
            if (fast.next!=null&&fast.next.next!=null){
                fast=fast.next.next;
            }else {
                return null;
            }
            if (slow.next!=null){
                slow=slow.next;
            }else {
                return null;
            }
            if (fast==slow){
                ListNode x=fast;
                ListNode y=headB;
                while(x!=null&&y!=null){
                    x=x.next;
                    y=y.next;
                    if (x==y){
                        return x;
                    }
                }
            }
        }
        return null;
    }
}

class IntersectionOfTwoLinkedLists {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA==null||headB==null){
            return null;
        }
        Set<ListNode> nodes=new HashSet<>();
        ListNode listA=headA;
        ListNode listB=headB;
        while (listA!=null){
            nodes.add(listA);
            listA=listA.next;
        }
        while (listB!=null){
            if (!nodes.add(listB)){
                return listB;
            }
            listB=listB.next;
        }
        return null;
    }
}
```
