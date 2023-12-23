# 2. Add Two Numbers

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.


## Just add

Add the two digits in first nodes, and then the second node, etc. If we get to the end of either node, we may still want to continue, bc. the other list node may still have something remaining, or we still get a carry for the next node. So we end the process until all list nodes are depleted, and there is no carry.

```python
ans = ListNode()  # Init. an empty list node
current_node = ans
carry = 0

# Keep adding the digits in the nodes
while True:

    # Add two numbers and carry (if any). If result >= 10, carry to next node
    num_1 = l1.val if l1 else 0  # If we reach the end of the node list, set
    num_2 = l2.val if l2 else 0  # the value to be zero, so the below addition
    num = num_1 + num_2 + carry  # can still work
    if num >= 10:
        num -= 10
        carry = 1
    else:
        carry = 0
    
    current_node.val = num

    # Move to the next nodes
    l1 = l1.next if l1 else None
    l2 = l2.next if l2 else None

    # If we already reach the end of all list nodes, and there is no carry
    if not (l1 or l2 or carry):
        return ans

    # Otherwise, create a next node in our answer
    current_node.next = ListNode()
    current_node = current_node.next
return ans
```

