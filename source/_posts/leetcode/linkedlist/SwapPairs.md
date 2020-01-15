---
title: LeetCode 24. Swap Nodes in Pairs
date: 2020-01-15 20:29:41
tags: [leetcode, linkedlist]
categories: [刷题]
---

Given a linked list, swap every two adjacent nodes and return its head.

You may not modify the values in the list's nodes, only nodes itself may be changed.

<!-- more -->

# Example:

> Given 1->2->3->4, you should return the list as 2->1->4->3.



# 思路
题意为两两交换链表中的元素，**不能改变链表的值**， 而是要交换链表结点；
具体思路：
cur为待交换结点的前一个结点，用于保存头结点信息
当两个节点交换完毕后， cur指向下一次待交换两个节点的前一个结点
示意图如下：

![demo](/demo.jpg)

# 代码

```java
class Solution {
    public ListNode swapPairs(ListNode head) {
         if (head == null || head.next == null) {
            return head;
        }
        ListNode dummyHead = new ListNode(-1);
        dummyHead.next = head;
        ListNode cur = dummyHead;
        while (cur.next != null && cur.next.next != null) {
            ListNode first = cur.next;
            ListNode second = first.next;
            first.next = second.next;
            second.next = first;
            cur.next = second;
            cur = first;
        }
        return dummyHead.next;
    }
}
```
