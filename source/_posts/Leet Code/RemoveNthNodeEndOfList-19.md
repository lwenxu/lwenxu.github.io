---
title: RemoveNthNodeEndOfList-19
date: 2019-07-24 10:00:50
tags: 
	- LeetCode
categories:
    - LeetCode
---

> Given a linked list, remove the *n*-th node from the end of list and return its head.
>
> **Example:**
>
> ```
> Given linked list: 1->2->3->4->5, and n = 2.
> 
> After removing the second node from the end, the linked list becomes 1->2->3->5.
> ```
>
> **Note:**
>
> Given *n* will always be valid.
>
> **Follow up:**
>
> Could you do this in one pass?



题目要求就是通过一次遍历删除倒数第 n 个元素。

## 思路

1. 一开始想着就是 `链表长度 - n`  这样遍历到这个位置把这个节点跳过就行了。可是这样的话需要遍历链表两次。
2. 那么我们要想办法一边遍历一边跳过节点，这个时候想到了数组中删除某个节点使用了 p,q 指针，那么这里是否可以呢？
3. 确实，我们 pq 指针开始指示的位置不同，q指针指向 n 节点后的位置，这样只要 q 指针到了链表的结尾我们就能找到 需要删除的节点就是 p 的位置。

## 优化

经过上面的一些想法基本形成了解题思路但是在实现的时候会有几个特例这个算法过不了比如 `[1] 1  [1,2]  2`  ,一开始我还尝试使用了一些方案来修补代码，但是这样总觉得不是很好，算法没有比较好的适用性。然后我想到这种状态主要是head指针一开始指向一个空节点那么删除head元素的时候就不会出现各种问题了。所以有下面的代码。

## 代码

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        // add null head
        ListNode nullNode = new ListNode(0);
        nullNode.next = head;
        head = nullNode;

        ListNode first = head, second = head;
        for (int i = 0; i < n; i++) {
            second = second.next;
        }
        while (second != null && second.next != null) {
            first = first.next;
            second = second.next;
        }
        first.next = first.next.next;
        return head.next;
    }
}
```

## 结果

>Runtime: 0 ms, faster than 100.00% of Java online submissions for Remove Nth Node From End of List.
>
>Memory Usage: 35 MB, less than 100.00% of Java online submissions for Remove Nth Node From End of List.

 