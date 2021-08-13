---
title: LeetCode 83.Remove Duplicates from Sorted List
date: 2020-2-23 23:00:00
tags: [leetcode, linkedlist]
categories: [每周一题]
---

Given a sorted linked list, delete all duplicates such that each element appear only once.

<!--more-->

# Example 1:

> Input: 1->1->2
> Output: 1->2


# Example 2:

> Input: 1->1->2->3->3
> Output: 1->2->3


# 思路
题意是给出一个有序链表， 删除重复的元素只保留一个。

具体思路：

* 从头结点遍历， 如果下一个结点与当前结点的 value 一致，删除下一个结点， node.next = node.next.next
* 如果下一个结点与当前结点的 value 不一致， node = node.next， 继续遍历

# 代码

```java
public ListNode deleteDuplicates(ListNode head) {
        ListNode ref = head;
        while (ref != null && ref.next != null) {
            if (ref.val == ref.next.val) {
                ref.next = ref.next.next;
            } else {
                ref = ref.next;
            }
        }
        return head;
}
```