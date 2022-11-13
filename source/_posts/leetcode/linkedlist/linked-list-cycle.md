---
title: LeetCode 141. Linked List Cycle
date: 2020-04-19 11:33:05
tags: [leetcode, linkedlist]
categories: [每周一题]
---

Given a linked list, determine if it has a cycle in it.

<!--more-->

To represent a cycle in the given linked list, we use an integer pos which represents the position (0-indexed) in the linked list where tail connects to. If pos is -1, then there is no cycle in the linked list.

 

# Example 1:

> Input: head = [3,2,0,-4], pos = 1
> 
> Output: true
> 
> Explanation: There is a cycle in the linked list, where tail connects to the second node.

![example1](circularlinkedlist.png)

# 题意

给出一个链表， 判断链表是否存在环。

# 思路

判断是否有环， 与无环的链表相比， 无环的链表如果进行遍历最终会走向 null 结点。
有环的结点因为环的存在会在环内循环， 所以利用快慢指针， 判断快慢指针是否会重合。

![环形链表循环遍历](示意图.jpg)

#  代码

```java
public boolean hasCycle(ListNode head) {
        if(head == null || head.next == null) {
            return false;
        }
        
        ListNode fast = head;
        ListNode slow = head;
        
        while(fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
            // 快慢指针相遇
            if(fast == slow){
                return true;
            }
        }
        
        return false;
        
    }
 ```