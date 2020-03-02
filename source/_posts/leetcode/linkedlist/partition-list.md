---
title: LeetCode 86. Partition List
date: 2020-02-26 22:56:12
tags: [leetcode, linkedlist]
categories: [刷题]
---
Given a linked list and a value x, partition it such that all nodes less than x come before nodes greater than or equal to x.

<!--more-->

You should preserve the original relative order of the nodes in each of the two partitions.


# Example:

> Input: head = 1->4->3->2->5->2, x = 3
> 
> Output: 1->2->2->4->3->5

# 题意
给出一个值 x ， 把两边分成两部分， 链表的结点的值比 x 小的位于大于等于 x 的节点之前， 并保持相对位置不变。
给出  1->4->3->2->5->2, x = 3 。 比 3 小的有 1，2，2, 大于等于 3 的结点有 4，3，5
分区后得到结果： 1->2->2->4->3->5

# 思路

既然分区， 分别使用两个链表存储小于 x 的结点以及大于等于 x 的结点， 遍历结束后， 将两个节点相连接；
![demo](/demo.jpg)

# 代码
```java
public ListNode partition(ListNode head, int x) {

        ListNode smallHead = new ListNode(-1);
        ListNode bigHead = new ListNode(-1);
        ListNode smallNode = smallHead;
        ListNode bigNode =  bigHead;
        
        while(head != null) {
            if(head.val < x) {
                smallNode.next = head;
                smallNode = smallNode.next;
            } else {
                bigNode.next = head;
                bigNode = bigNode.next;
            }
            head = head.next;
        }
        
        smallNode.next = bigHead.next;
        // 断开与原始链表的链接
        bigNode.next = null;
        
        return smallHead.next;
  }
  ```