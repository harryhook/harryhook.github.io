---
title: LeetCode 138. Copy List with Random Pointer
date: 2020-04-12 22:27:21
tags: [leetcode, linkedlist]
categories: [每周一题]
---

A linked list is given such that each node contains an additional random pointer which could point to any node in the list or null.

<!--more-->

Return a deep copy of the list.

The Linked List is represented in the input/output as a list of n nodes. Each node is represented as a pair of [val, random_index] where:

val: an integer representing Node.val
random_index: the index of the node (range from 0 to n-1) where random pointer points to, or null if it does not point to any node.
 

# Example 1:

![expample1](/e1.png)

> Input: head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
> 
> Output: [[7,null],[13,0],[11,4],[10,2],[1,0]]


# 题意

给出有个链表， 结点含有 value域， next 指针、random 指针， 其中 next 指针指向下一个结点， random 指向任意结点， 现在需要复制一个给定链表。

思路：

* 先复制 next 结点
* 再复制 random 结点
* 连接新的链表

# 代码

```java
/*
// Definition for a Node.
class Node {
    int val;
    Node next;
    Node random;

    public Node(int val) {
        this.val = val;
        this.next = null;
        this.random = null;
    }
}
*/

class Solution {
    public Node copyRandomList(Node head) {
    
        generateNext(head);
        connectRandom(head);
        
        return reconnectList(head);
    }
    
    public static void generateNext(Node head) {
    
        Node pNode = head;
        
        while(pNode != null) {
            Node pCloned = new Node(pNode.val);
            pCloned.next = pNode.next;
            pNode.next = pCloned;
            pNode = pCloned.next;
        }    
    }
    
        
    public static void connectRandom(Node head) {
        
        Node pNode = head;
        Node pCloned  = null;
        
        while(pNode != null) {
            pCloned = pNode.next;
            if(pNode.random != null) {
                pCloned.random = pNode.random.next;
            }
            pNode = pCloned.next;
        }    
    
    }
    
        
    public static Node reconnectList(Node head) {
    
        Node pNode = head;
        Node cloneHead = null;
        Node pCloned = null;
        
        if(pNode != null) {
            pCloned = cloneHead = head.next;
            pNode.next = pCloned.next;
            pNode = pCloned.next;
        }
        
        while(pNode != null) {
            pCloned.next = pNode.next;
            pCloned = pCloned.next;
            pNode.next = pCloned.next;
            pNode = pCloned.next;
        }
        return cloneHead;
    
    }
}
```