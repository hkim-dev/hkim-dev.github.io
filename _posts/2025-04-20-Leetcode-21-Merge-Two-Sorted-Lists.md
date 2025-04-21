---
title: Leetcode 21 Merge Two Sorted Lists - Problem Solving & Intuition
date: 2025-04-20 23:00:00 -0400
categories: "Problem-Solving"
tags:
  - Leetcode
  - Python
toc: false
classes: wide
---


This problem is about merging two "sorted" linked lists into one sorted list. It is a good problem to practice working with linked lists and you can commonly see problems like this in interviews. In this article, I'll discuss the approach I took and techniques used in the solution.

## Problem Statement
You can find the full description here: [link](https://leetcode.com/problems/merge-two-sorted-lists/)

- **Given:** Two sorted linked lists 
- **Goal:** Return the head of the merged list

## My Thought Process
Using two pointers was the obvious choice here. They are even given as the parameters - `list1` and `list2`. As the lists are already sorted, meaning the nodes are in the ascending order, so we can use the given pointers to determine which one is pointing to a smaller node. Whichever points to a smaller value should move on to the next one, and we should keep repeating it until there is no more node.
On a side note, it's worth noting that the solution is required to return the head node, which is, to be exact, a pointer to the head node. In my solution, I used a dummy node to keep track of the head.

## Solution
Refer to the following for my final solution:

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
        # curr: pointer for the "current node" we're building into the merged list
        # dummy: dummy node that helps us keep a reference to the head
        curr = dummy = ListNode()

        # continue looping while both list1 and list2 have nodes
        while list1 and list2:
            # compare the current values of list1 and list2
            # whichever one is smaller, link it to `curr.next`
            # move forward in the list from which the smaller node was taken
            if list1.val < list2.val:
                curr.next = list1 
                list1 = list1.next
            else:
                curr.next = list2
                list2 = list2.next
            # move curr to the tail
            curr = curr.next
        
        # if there are remaining nodes, append them to the end of the merged list
        if list1:
            curr.next = list1
        elif list2:
            curr.next = list2
        
        # return the "next" node of the dummy node
        return dummy.next
```

- Time Complexity: O(n) where n is `len(list1) + len(list2)`
- Space Complexity: O(1)


## Takeaways
This problem helped me practice a few techniques: two pointers and using a dummy node. It also reminded me of the concept of "pointers", which are essentially references to nodes, and how they can be used to effectively build and update linked lists.