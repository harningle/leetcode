# 141. Linked List Cycle

Given `head`, the head of a linked list, determine if the linked list has a cycle in it.

There is a cycle in a linked list if there is some node in the list that can be reached again by continuously following the `next` pointer. Internally, `pos` is used to denote the index of the node that tail's `next` pointer is connected to. **Note that `pos` is not passed as a parameter.**

Return `true` *if there is a cycle in the linked list*. Otherwise, return `false`.

 
**Example:**

![](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist.png)

> **Input:** `head = [3, 2, 0, -4], pos = 1`
> 
> **Output:** `true`


## Fast and slow pointer

The fast pointer moves two steps at a time, and the slow pointer moves one step. As long as there is a cycle, the slower pointer will be caught and lapped by the fast pointer.

```cpp
bool hasCycle(ListNode *head) {
    ListNode* fast = head;
    ListNode* slow = head;

    while (fast != NULL && fast -> next != NULL) {
        fast = fast -> next -> next;
        slow = slow -> next;
        if (fast == slow) {
            return true;
        }
    }
    return false;
}
```
