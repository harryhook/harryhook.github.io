---
title: LeetCode 19. Remove Nth From End of List
date: 2019-12-09 19:07:58
tags: [leetcode, linkedlist]
categories: [刷题]
---

Given a linked list, remove the n-th node from the end of list and return its head.

<!-- more -->

## Example


> Given linked list:
> **1->2->3->4->5** ,  and n = 2.
> After removing the second node from the end, the linked list becomes
> **1->2->3->5**.

Note:

Given n will always be valid.

**Could you do this in one pass?**

## 思路

本题重点在于**一次遍历**解决问题， 原本的思路是：

* 先遍历算出链表的长度**length**
* 再从head结点移动到第**length-n-1**个结点
* 删除第**length-n-1**结点的下一个结点来解决

但是这样是不符合题意的， 因为遍历了两遍链表。

正确的解题思路，利用三指针（slow, fast, pre）来解决问题：

* fast指针先遍历n个节点
* 定义pre结点，位于slow结点之前， slow结点此时指向head结点
* pre, slow, fast结点同时移动， 当fast结点为null时， slow结点位于待删除结点
* pre结点指向slow结点的下一个结点，即删除倒数第n个节点

## 示意图

![示意图](/demo.jpg)

 
## 代码

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
    
        if(head==null) return head;
        
        ListNode first = head;
        ListNode dummyHead = new ListNode(-1);
        dummyHead.next = head;
        
        while(n-- > 0 && first != null) {
            first = first.next;
        }
        
        ListNode slow = head;
        ListNode pre = dummyHead;
        
        while(first!=null) {
            pre = slow;
            slow = slow.next;
            first = first.next;
        }
        
        pre.next = slow.next;
        
        return dummyHead.next;
    }
}
```

