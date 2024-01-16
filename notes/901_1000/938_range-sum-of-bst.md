# 938. Range Sum of BST

Given the `root` node of a binary search tree and two integers `low` and `high`, return *the sum of values of all nodes with a value in the ___inclusive___ range* `[low, high]`.

 
**Example:**

![](https://assets.leetcode.com/uploads/2020/11/05/bst1.jpg)

> **Input:** `root = [10, 5, 15, 3, 7, null, 18], low = 7, high = 15`
> 
> **Output:** `32`
> 
> **Explanation:** Nodes `7`, `10`, and `15` are in the range $[7, 15]$. $7 + 10 + 15 = 32$.


## DFS

The brute force would be, DFS through all nodes, and add up all node values in the given range. We can do better than this by skipping certain halves of the tree: if the current node value is too large (small), we only need to check the left (right) half.

The time complexity is $O(n)$, as in the worst case (all nodes's values are in the given range), it degenerates to a usual DFS, which is $O(n)$.

```c
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
int rangeSumBST(struct TreeNode* root, int low, int high) {
    // If we already read the end, stop
    if (root == NULL) {
        return 0;

    // If the current node value is too large, check the left half tree
    } else if (root -> val > high) {
        return rangeSumBST(root -> left, low, high);

    // If too small, check the right half
    } else if (root -> val < low) {
        return rangeSumBST(root -> right, low, high);

    // If in range, add this value and check both halves of the tree
    } else {
        return root -> val + rangeSumBST(root -> left, low, high)
            + rangeSumBST(root -> right, low, high);
    }
}
```
