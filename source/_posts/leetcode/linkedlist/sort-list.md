---
title: LeetCode 148. Sort List
date: 2020-05-13 22:59:43
tags: [leetcode, linkedlist]
categories: [每周一题]
---

Sort a linked list in O(n log n) time using constant space complexity.

<!--more-->

# Example 1

> Input: 4->2->1->3
>
> Output: 1->2->3->4


# Example 2

> Input: -1->5->3->4->0
>
> Output: -1->0->3->4->5


# 题意

给出一个链表， 使用常量空间，O(n log n)时间复杂度下对链表进行排序。


# 思路

1. 首先常量空间意味着不能借助于辅助的数据结构， 比如数组、栈之类的；

2. 其次 n log n 的时间复杂度， 可以采用插入排序、选择排序或者归并排序；

3. 在  [insertion-sort-list](2020/05/12/leetcode/linkedlist/insertion-sort-list)  中采用了插入排序， 在本题中采用归并排序来解决这个问题。


# 代码

```java
    public ListNode sortList(ListNode head) {
        if(head == null || head.next == null) return head;
    
        ListNode pre = null;
        ListNode slow = head;
        ListNode fast = head;
        
        while (fast != null && fast.next != null) {
            pre = slow;
            slow = slow.next;
            fast = fast.next.next;
        }
        
        pre.next = null;
        
        ListNode l1 = sortList(head);
        ListNode l2 = sortList(slow);
        
        return mergeSort(l1, l2);
    }
    
    public ListNode mergeSort(ListNode l1, ListNode l2) {
        
        ListNode p = new ListNode(-1);
        ListNode dummyNode = p;
        
        while (l1 != null && l2 != null) {
            
            if(l1.val < l2.val) {
                p.next = l1;
                l1 = l1.next;
            } else {
                p.next = l2;
                l2 = l2.next;
            }
            
            p = p.next;
        }
        
        if (l1 != null) {
            p.next = l1;
        }
        
        if (l2 != null) {
            p.next = l2;
        }
    
        return dummyNode.next;
    }
```