---
title: LeetCode 23. Merge k Sorted Lists
date: 2019-12-18 22:53:55
tags: [leetcode, linkedlist]
categories: [刷题]
---

Merge k sorted linked lists and return it as one sorted list. Analyze and describe its complexity.
<!--more-->

# Example

    Input:
    [
      1->4->5,
      1->3->4,
      2->6
    ]
    Output: 1->1->2->3->4->4->5->6

# 思路

LeetCode hard难度题， 看到这道题我的第一思路是链表两两比较生成新的链表， 再将新的链表与剩余的链表进行比较生成新的链表，这个思路也是可行的， 但是效率不高， 具体代码如下：

```java
    public ListNode mergeKLists(ListNode[] lists) {
        if(lists.length == 0) return null;
        if(lists.length == 1) return lists[0];
        
        ListNode newList = mergeTwoLists(lists[0], lists[1]);
        for(int i=2; i<lists.length; i++) {
           newList =  mergeTwoLists(newList, lists[i]);
        }
        return newList;
    }
```

mergeTwoLists(ListNode l1, ListNode l2); 参见[合并两个有序联链表](/2019/12/17/leetcode/linkedlist/MergeTwoLists)

算法复杂度 k * n， ac结果如下：
![n-1次比较](/n-1次比较.JPG)

# 新的思路

利用优先级队列的排序功能， 每次队列中的结点都是有序的

* 新建一个队列， 依次存入链表数组lists中所有链表的头结点；
* 创建虚拟头结点dummyHead；
* 队列弹出元素（始终是队列中最小的元素）, 链接到现有的链表上；
* 判断当前元素的下一个结点是否为空， 不为空插入到优先级队列中（此时会继续调整队列顺序）；
* 重复之前的步骤， 直到队列为空。

示意图如下：

![示例1](/demo1.jpg)
![示例2](/demo2.jpg)
![示例3](/demo3.jpg)

# 代码

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
         if (lists == null || lists.length == 0) {
            return null;
        }
        // 创建优先级队列
        PriorityQueue<ListNode> queue = new PriorityQueue<ListNode>(lists.length, new Comparator<ListNode>() {
            @Override
            public int compare(ListNode o1, ListNode o2) {
                if(o1.val  < o2.val) {
                    return -1;
                } else  if(o1.val  > o2.val){
                    return 1;
                } else {
                    return 0;
                }
            }
        });
        
        // 所有链表的头结点保存在优先级队列
        for(ListNode node : lists) {
            if(node != null) {
                queue.add(node);
            }
        }
        
        // 构建新的链表
        ListNode dummyHead = new ListNode(-1);
        ListNode tail = dummyHead;
        
        // 进行比较
        while(!queue.isEmpty()) {
            tail.next = queue.poll();
            tail = tail.next;
            if(tail.next != null) {
                queue.add(tail.next);
            }
        }
        return dummyHead.next;
    }
}
```

可以看下优化后的方法的ac结果， 相比之前快了很多：
![better](/better.jpg)