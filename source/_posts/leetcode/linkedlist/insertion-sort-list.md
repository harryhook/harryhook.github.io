---
title: LeetCode 147. Insertion Sort List
date: 2020-05-12 19:59:20
tags: [leetcode, linkedlist]
categories: [每周一题]
---

Sort a linked list using insertion sort.

<!--more-->

# Example 1

> Input: 4->2->1->3

> Output: 1->2->3->4

# Example 2

> Input: -1->5->3->4->0

> Output: -1->0->3->4->5


![示意图](/example.gif)

# 题意

给出一个链表， 采用插入排序对链表进行排序

# 思路

1.  利用虚拟头结点使得所有结点按同样的逻辑处理；
2.  通过 pre 指针遍历链表， 找到第一个比之前结点小的结点， 记录 pre 信息；
3.  进行插入操作。

# 示意图


![示意图](/插入排序.png)

# 代码

```java
    public ListNode insertionSortList(ListNode head) {
        if(head == null || head.next == null) return head;
        
        ListNode dummyHead  = new ListNode(-1);
        ListNode pre = dummyHead;
        ListNode cur = head;
        ListNode next = null;
        
        while(cur != null) {
        	// 下次遍历的起点
            next = cur.next;
            // 找到待插入的结点
            while(pre.next != null && pre.next.val < cur.val) {
                pre = pre.next;
            }
            	
            // 断链
            cur.next = pre.next;
            pre.next = cur;
            // pre 回到虚拟头结点重新遍历
            pre = dummyHead;            
            cur = next;
        }
        return dummyHead.next;
    }
```