# Find Bottom Left Tree Value

Given the `root` of a binary tree, return the leftmost value in the last row of the tree.

 
**Example:**

![](https://assets.leetcode.com/uploads/2020/12/14/tree2.jpg)

> **Input:** `root = [1, 2, 3, 4, null, 5, 6, null, null, 7]`
> 
> **Output:** `7`


## BFS (from right to left)

The intuition is easy, we go down level by level, and within each level, we find the leftmost node. To know if we have some more levels below, we have to visit all nodes in the current level, and check if any of them has a child. This is essentially a BFS, but from right to left (rather than the usual left to right, but otherwise exactly the same).

```cpp
int findBottomLeftValue(TreeNode* root) {
    queue<TreeNode*> q;
    q.push(root);

    TreeNode* node;
    while (!q.empty()) {
        node = q.front();
        q.pop();
        
        if (node -> right != NULL) {  // First right, then left
            q.push(node -> right);
        }
        if (node -> left != NULL) {
            q.push(node -> left);
        }
    }

    return node -> val;
}
```
