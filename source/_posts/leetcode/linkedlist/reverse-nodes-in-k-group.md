---
title: LeetCode 25. Reverse Nodes In k Group
date: 2020-02-23 17:50:54
tags: [leetcode, linkedlist]
categories: [每周一题]
---

Given a linked list, reverse the nodes of a linked list k at a time and return its modified list.

<!--more-->

k is a positive integer and is less than or equal to the length of the linked list. If the number of nodes is not a multiple of k then left-out nodes in the end should remain as it is.

# Example:

> Given this linked list: 1->2->3->4->5
>
> For k = 2, you should return: 2->1->4->3->5
>
> For k = 3, you should return: 3->2->1->4->5


* Note

	Only constant extra memory is allowed.
	
	You may not alter the values in the list's nodes, only nodes itself may be changed.


# 思路
题意为一次翻转链表中 k 个元素，如果 k = 2， 即每两个两个元素进行翻转，然后返回翻转后的链表。

具体思路：
> 每次翻转 k 个结点， 循环进行
> 循环终止的条件是循环到最后一个节点


示意图如下：

![demo](/reverse-nodes-k.jpg)

# 代码

```java

public ListNode reverseKGroup(ListNode head, int k) {
        if (k < 2) {
            return head;
        }
        ListNode dummyHead = new ListNode(-1);
        dummyHead.next = head;
        ListNode pre = dummyHead;
        ListNode tail = dummyHead;
        ListNode temp = null;
        
        while (true) {
        
            int count = 0;

            while (count < k && tail != null) {
                count++;
                tail = tail.next;
            }
            if (tail == null) {
                break;
            }
            
            head = pre.next;
            while (pre.next != tail) {
                temp = pre.next;
                pre.next = temp.next;
                temp.next = tail.next;
                tail.next = temp;
            }
            tail = head;
            pre = head;
        }

        return dummyHead.next;  
}


```