---
title: LeetCode 82. Remove Duplicates from Sorted List II 
date: 2020-02-23 23:25:03
tags: [leetcode, linkedlist]
categories: [每周一题]
---

Given a sorted linked list, delete all nodes that have duplicate numbers, leaving only distinct numbers from the original list.

<!--more-->

# Example 1:

> Input: 1->2->3->3->4->4->5
> 
> Output: 1->2->5


# Example 2:

> Input: 1->1->1->2->3
> 
> Output: 2->3

# 思路

题意是给出一个有序链表， 删除所有的重复的元素，对重复元素不做保留。

例子2 可以看到有头结点和之后元素重复的情况， 所有需要利用虚拟头结点；

*  1. pre指向虚拟头结点，pre.next 指向头结点，cur 指针从头结点进行遍历；
*    2. 判断否存在多个相同值的结点，若有遍历到最后一个重复元素：
	* 当pre.next  != cur时， 说明中间遍历的元素是重复的， 删除重复的元素
	* pre.next  == cur 时， 说明前后没有重复的元素， pre 与 cur 指针分别指向下一个元素
*  3.  重复 **步骤 2.** 继续遍历元素进行判断

# 示意图
示意图如下：
![demo](demo.jpg)

# 代码

```java
public ListNode deleteDuplicates(ListNode head) {

    ListNode dummyHead = new ListNode(-1);
    
    dummyHead.next = head;
    ListNode pre = dummyHead;
    ListNode curr = head;
    
    while(curr != null) {
    
        // 遍历重复的元素， 直到最后一个
        while(curr.next != null && curr.val == curr.next.val) {
            curr = curr.next;
        }
        
        // 两个结点不同， 继续遍历
        if(pre.next == curr) {
            pre = pre.next;
        } else {
        	  //删除重复的元素
            pre.next = curr.next;
        }
        curr = curr.next;
    }
    
    return dummyHead.next;
    
}
```