---
title: LeetCode 2. Add Two Numbers
date: 2019-12-02 17:07:23
tags: [leetcode, linkedlist]
categories: [刷题]
---

**两链表相加， 以链表形式顺序输出结果**

<!-- more -->

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

# Example

> Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
> 
> Output: 7 -> 0 -> 8
> 
> Explanation: 342 + 465 = 807.


> Input: (1 -> 2 -> 4) + (1 -> 5 -> 6)
> 
> Output: 2 -> 7 -> 0 -> 1
> 
> Explanation: 421 + 651 = 1072.


# 思路
Medium 难度题，两个链表相加， 同步遍历两个链表结点， 将两个节点值的和存储为新结点。

> 需要考虑三个点：

> * 一是头结点如何处理(利用虚拟结点处理)；
> * 二是结点为空如何处理（空值取0计算）；
> * 三是进位如何处理（进位保存到参与下一节点计算）

# 代码

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
public ListNode addTwoNumbers(ListNode head1, ListNode head2) {

    ListNode dummyHead = new ListNode(-1);
    ListNode l1 = head1;
    ListNode l2 = head2;
    ListNode s = dummyHead;

    int carry = 0, sum = 0;

    while (l1 != null || l2 != null) {

        int x = (l1 == null ? 0 : l1.val);
        int y = (l2 == null ? 0 : l2.val);
        sum = carry + x + y;
        s.next = new ListNode(sum % 10);
        s = s.next;
        carry = sum / 10;
        if (l1 != null) l1 = l1.next;
        if (l2 != null) l2 = l2.next;

    }

    if (carry != 0) {
        s.next = new ListNode(carry);
    }

    return dummyHead.next;
}
```