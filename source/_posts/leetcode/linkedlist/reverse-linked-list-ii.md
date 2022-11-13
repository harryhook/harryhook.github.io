---
title: LeetCode 92. Reverse Linked List II
date: 2020-03-22 23:47:35
tags: [leetcode, linkedlist]
categories: [每周一题]
---

Reverse a linked list from position m to n. Do it in one-pass.

<!--more-->

Note: 1 ≤ m ≤ n ≤ length of list.

# Example:

> Input: 1->2->3->4->5->NULL, m = 2, n = 4

> Output: 1->4->3->2->5->NULL


# 思路

题意为给定 m、n， 翻转从第 m 到第 n 个结点。

具体分三步：

1.  先遍历 m-1 个节点， 找到待翻转的结点为尾节点
2.  遍历 n-m-1 次， 依次翻转

# 示意图

![示意图](demo.png)

# 代码
```java
public ListNode reverseBetween(ListNode head, int m, int n) {

    ListNode dummyHead = new ListNode(-1);
    dummyHead.next = head;
    ListNode pre = dummyHead;
    
    // 先遍历 m 个结点
    for (int i = 0; i < m - 1; i++) {
        pre = pre.next;
    }
    
    // 翻转前的尾结点
    ListNode tail = pre.next;
    ListNode temp;
    
    // 依次翻转
    for (int i = 0; i < n - m; i++) {
        temp = pre.next;
        pre.next = tail.next;
        tail.next = tail.next.next;
        pre.next.next = temp;
    }
    return dummyHead.next;
}
```