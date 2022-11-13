---
title: LeetCode 109. Convert Sorted List to Binary Search Tree
date: 2020-03-29 10:33:37
tags: [leetcode, linkedlist]
categories: [每周一题]
---

Given a singly linked list where elements are sorted in ascending order, convert it to a height balanced BST.

<!--more-->

For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1.

# Example

	Given the sorted linked list: [-10,-3,0,5,9],
	
	One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:

	      0
	     / \
	   -3   9
	   /   /
	 -10  5
	
	
# 题意

给定一个升序链表， 转换成高度平衡二叉树。 
其实就是中序遍历变前序遍历， 关键点是找到树的根结点， 也就是升序链表的中间结点， 然后递归遍历。

# 示意图

![示意图](有序链表转换为平衡二叉树.png)

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
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode sortedListToBST(ListNode head) {
        if (head == null) {
            return null;
        }

        ListNode fast = head;
        ListNode slow = head;
        ListNode temp = null;
        while (fast != null && fast.next != null) {
            temp = slow;
            fast = fast.next.next;
            slow = slow.next;    
        }
        // 断开链表
        if (temp != null) {
            temp.next = null;
        } else {
            head = null;
        }
        
        TreeNode root = new TreeNode(slow.val);
        root.left = sortedListToBST(head);
        root.right = sortedListToBST(slow.next);
        return root;
        
    }
}
```
 
 