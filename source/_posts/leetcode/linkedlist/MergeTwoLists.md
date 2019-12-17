---
title: LeetCode 21. Merge Two Sorted Lists
date: 2019-12-17 22:42:42
tags: [leetcode, linkedlist]
categories: [刷题]
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
        ListNode curr = dummyHead, p = l1, q = l2;
        while(p != null && q != null) {
            if(p.val <= q.val) {
                curr.next = p;
                p = p.next;
            } else {
                curr.next = q;
                q = q.next;
            }
            curr = curr.next;
        }
        if(p != null) {
            curr.next = p;
        } 
        if(q != null) {
            curr.next = q;
        }
        return dummyHead.next;
    }
}
```