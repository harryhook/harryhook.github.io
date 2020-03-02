---
title: LeetCode 61. Rotate List
date: 2020-02-23 21:46:53
tags: [leetcode, linkedlist]
categories: [刷题]
---

Given a linked list, rotate the list to the right by k places, where k is non-negative.

<!--more-->

# Example 1:

> Input: 1->2->3->4->5->NULL, k = 2
> Output: **4->5**->1->2->3->NULL
> Explanation:
> rotate 1 steps to the right: 5->1->2->3->4->NULL
> rotate 2 steps to the right: 4->5->1->2->3->NULL

# Example 2:

> Input: 0->1->2->NULL, k = 4
> Output: **2**->0->1->NULL
> Explanation:
> rotate 1 steps to the right: 2->0->1->NULL
> rotate 2 steps to the right: 1->2->0->NULL
> rotate 3 steps to the right: 0->1->2->NULL
> rotate 4 steps to the right: 2->0->1->NULL

# 思路

题意从链表右边的第 k 个结点， 然后进行翻转。
如果 k < 链表的长度， 从链表的倒数第 k 个结点翻转； 
如果 k > 链表的长度， 从链表的第 len -  k%len 个结点翻转；

具体思路：

> * 遍历链表得到当前链表的长度 length；
> * 头尾相连；
> * 若k > length，从头遍历到第 k 个结点， 然后断开；若 k > 链表的长度， 从链表的第 len -  k%len 个结点断开。

# 代码

```java
public ListNode rotateRight(ListNode head, int k) {

    if(head==null || head.next == null) return head;
    
    int len = 1;
    ListNode tail  = head;
    
    // 计算链表长度
    while(tail.next != null) {
        len++;
        tail = tail.next;
    }
    
    // 头尾相连
    tail.next = head;
    ListNode temp = head;
    
    //找到倒数第 k 个结点
    for(int i=1; i<len - k%len; i++) {
        temp = temp.next;
    }
    // 新的头结点
    head  = temp.next;
   	 // 断开链表
    temp.next = null;
    
    return head;
}
```

