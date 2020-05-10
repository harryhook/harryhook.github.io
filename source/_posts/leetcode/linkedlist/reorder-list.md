---
title: LeetCode 143. Reorder List
date: 2020-05-10 23:04:48
tags: [leetcode, linkedlist]
categories: [每周一题]
---


Given a singly linked list L: L0→L1→…→Ln-1→Ln,
reorder it to: L0→Ln→L1→Ln-1→L2→Ln-2→…

<!--more-->


You may not modify the values in the list's nodes, only nodes itself may be changed.

# Example 1:

> Given 1->2->3->4, 

> reorder it to 1->4->2->3.

# Example 2:

> Given 1->2->3->4->5, 

> reorder it to 1->5->2->4->3.



# 题意

给出一个链表， 将 1-2-3-4 这样的顺序重新排序成 1-4-2-3

# 思路

1. 将链表分成两半(利用快慢指针找到中间结点)
2. 后半部分链表翻转
3. 同时遍历两个链表将对应的结点进行链接， l0->ln, l1->ln-1

# 代码

```java
    public void reorderList(ListNode head) {
        if(head == null || head.next == null) return ;
        
        ListNode pre = null;
        ListNode slow = head;
        ListNode fast = head;
        
        while(fast != null && fast.next != null) {
            pre = slow;
            slow = slow.next;
            fast = fast.next.next;
        }
        
        // 断开前后链表
        pre.next = null;
        
        // 后半部分链表翻转
        ListNode secondList = reverse(slow);
        
        ListNode l1 = head;
        ListNode l2 = secondList;
        	
        // 合并
        while(l1 != null) {
            ListNode n1 = l1.next, n2 = l2.next;
            l1.next = l2;
            if(n1 == null) break;
            l2.next = n1;
            l2 = n2;
            l1 = n1;
        }
        
    }
    
    public ListNode reverse(ListNode head) {
        if(head == null || head.next == null) return head;
        ListNode newHead = reverse(head.next);
        head.next.next = head;
        head.next = null;
        return newHead;
    }
```