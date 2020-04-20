---
title: LeetCode 142. Linked List Cycle II
date: 2020-04-19 17:00:52
tags: [leetcode, linkedlist]
categories: [每周一题]
---

Given a linked list, return the node where the cycle begins. If there is no cycle, return null.

<!--more-->

To represent a cycle in the given linked list, we use an integer pos which represents the position (0-indexed) in the linked list where tail connects to. If pos is -1, then there is no cycle in the linked list.

Note: Do not modify the linked list.

 

# Example 1:

> Input: head = [3,2,0,-4], pos = 1
> 
> Output: tail connects to node index 1
> 
> Explanation: There is a cycle in the linked list, where tail connects to the second node.

![ex1](/circularlinkedlist.png)

# 题意

给出一个链表， 判断是否有环， 有环的话返回环开始的结点。 若环不存在，返回 null

# 思路

返回环相交的结点需要分两步
1. 判断环是否存在， 不存在返回 null， 此处需要利用快慢指针
2. 若环存在， 需要利用以下公式

假设头结点到环开始的位置为 x1， 环开始的结点到相交的结点 x2, ， 相遇结点到环开始的结点 x3， 环的长度为 N 。

因为快指针是慢指针走过路程的两倍,  所以有以下结论

慢指针走过的距离： x1 + x2
快指针走过的距离： x1 + x2 + x3 + x2
环的长度：N = x2 + x3

2(x1 + x2) =  x1 + x2 + x3 + x2  =====>>>> x1 = x3， 所以当快慢指针相遇时从头结点遍历到与慢指针相遇即为环形结点开始处。

![示意图](/ex.png)

详细推导：

	2*(x1 + x2) + 2n*(x3 + x2) = (x1 + x2 + x3 + x2) + m*(x3 + x2)
	x1 + 2n*(x3 + x2) = x3 + m*(x3 + x2)
	x1 = x3 + (x3 + x2)*(m - 2n)

# 代码

```java
    public ListNode detectCycle(ListNode head) {
        if(head == null || head.next == null) {
            return null;
        }
        
        ListNode fast = head;
        ListNode slow = head;
        
        while(fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
            // 快慢指针相遇
            if(slow == fast) {
                break;
            }
        }
        // 环不存在
        if(fast != slow) return null;
        fast = head;
        // fast 从头结点遍历， slow 指针从相遇处遍历， 再次相遇即为环开始处
        while(fast != slow) {
            fast = fast.next;
            slow = slow.next;
        }
        return fast;
    }
```