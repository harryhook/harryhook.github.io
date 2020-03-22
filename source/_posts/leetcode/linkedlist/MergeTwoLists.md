---
title: LeetCode 21. Merge Two Sorted Lists
date: 2019-12-17 22:42:42
tags: [leetcode, linkedlist]
categories: [每周一题]
---
Merge two sorted linked lists and return it as a new list.The new list should be made by splicing together the nodes of the first two lists.

<!--more-->

# Example
> Input: 1->2->4, 1->3->4
> Output: 1->1->2->3->4->4

# 思路

LeetCode简单难度题， 合并两个**有序**链表，因为两个链表都是有序的就好办了。
* 循环遍历两个链表，进行大小比较；
* 利用临时结点， 将每次较小的结点链接在临时结点之后；
* 当其中一个链表遍历结束后， 将另一个结点链接到临时结点之后。

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
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    
        ListNode dummyHead = new ListNode(-1);
        ListNode curr = dummyHead;
        
        while(l1 != null && l2 != null) {
        
            if(l1.val <= l2.val) {
                curr.next = l1;
                l1 = l1.next;
            } else {
                curr.next = l2;
                l2 = l2.next;
            }
            
            curr = curr.next;
        }
        
        if(l1 != null) {
            curr.next = l1;
        }
        
        if(l2 != null) {
            curr.next = l2;
        }
        
        return dummyHead.next;
    }
}
```