# 2. Add Two Numbers

You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order**, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.


**Example:**

![](https://assets.leetcode.com/uploads/2020/10/02/addtwonumber1.jpg)

> **Input:** `l1 = [2, 4, 3]`, `l2 = [5, 6, 4]`
> 
> **Output:** `[7, 0, 8]`
> 
> **Explanation:** $342 + 465 = 807$.


## Just add

Add the two digits in first nodes, and then the second node, etc. If we get to the end of either node, we may still want to continue, bc. the other list node may still have something remaining, or we still get a carry for the next node. So we end the process until all list nodes are depleted, and there is no carry.

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2) {
    // Pointer to the first node of ans
    struct ListNode* ans = malloc(sizeof(struct ListNode));
    ans -> val = 0;
    ans -> next = NULL;
    struct ListNode* node = ans;  // Pointer to the current node in `ans`
    
    int carry = 0;
    int val_1;
    int val_2;
    int sum;

    // Keep going to the next node until we deplete both input node list and
    // have no carry
    while (l1 != NULL || l2 != NULL || carry == 1) {
        // Check if we reach the end of the input node. If yes, set val to 0
        val_1 = (l1 != NULL)? l1 -> val: 0;  
        val_2 = (l2 != NULL)? l2 -> val: 0;

        // Just add
        sum = val_1 + val_2 + carry;
        carry = (sum >= 10)? 1: 0;
        node -> val = sum % 10;

        // Go to the next node in the input node list, if haven't deplete them
        if (l1 != NULL) {
            l1 = (l1 -> next != NULL)? l1 -> next: NULL;
        }
        if (l2 != NULL) {
            l2 = (l2 -> next != NULL)? l2 -> next: NULL;
        }

        // Create a next node in `ans` if we need to continue
        if(l1 != NULL || l2 != NULL || carry == 1) {
            node -> next = malloc(sizeof(struct ListNode));
            node = node -> next;
            node -> next = NULL;
        }
    }
    return ans;
}
```

**Set `node -> next -> next` to `NULL`!** Unlike Python where we can have a default null value (`ListNode.next = None`), C cannot do this. We have `struct ListNode *next;` inside the struct, so `*next` must point to some place. Whenever we create a list node, always point the `*next` to `NULL` so we won't get memory error.
