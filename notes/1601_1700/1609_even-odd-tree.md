# 1609. Even Odd Tree

A binary tree is named **Even-Odd** if it meets the following conditions:

* the root of the binary tree is at level index $0$, its children are at level index $1$, their children are at level index $2$, etc.
* for every **even-indexed** level, all nodes at the level have **odd** integer values in **strictly increasing** order (from left to right)
* for every **odd-indexed** level, all nodes at the level have **even** integer values in **strictly decreasing** order (from left to right)

Given the `root` of a binary tree, *return* `true` *if the binary tree is __Even-Odd__, otherwise return* `false`.


**Example:**

![](https://assets.leetcode.com/uploads/2020/09/15/sample_1_1966.png)

> **Input:** `root = [1, 10, 4, 3, null, 7, 9, 12, 8, 6, null, null, 2]`
> 
> **Output:** `true`
> 
> **Explanation:** The node values on each level are:
>
> Level $0$: `[1]`
> 
> Level $1$: `[10, 4]`
> 
> Level $2$: `[3, 7, 9]`
> 
> Level $3$: `[12, 8, 6, 2]`
> 
> Since levels $0$ and $2$ are all odd and increasing and levels $1$ and $3$ are all even and decreasing, the tree is Even-Odd.


## BFS

On each level, we need to go from left to right, check the parity (odd/even) of node values, and to check if they are increasing/decreasing correctly. This level by level, left to right traversal is exactly how BFS visits the tree. Whenever some condition is not satisfied, we end the BFS and return `false`.

```cpp
bool isEvenOddTree(TreeNode* root) {
    queue<pair<TreeNode*, int>> q;  // Unvisited nodes and their level
    q.push({root, 0});

    TreeNode* node;       // Current node
    int level;            // Current level
    int val;              // Current node value
    int prev_level = -1;  // -1 so root node is viewed as a node at a new level
    int prev_val ;
    while (!q.empty()) {
        node = q.front().first;
        level = q.front().second;
        val = node -> val;
        q.pop();

        // Check parity
        if ((level % 2 == 0) && (val % 2 == 0)) {
            return false;
        }
        if ((level % 2 == 1) && (val % 2 == 1)) {
            return false;
        }

        // If we are at the same level as the previous node, check monotonicity
        // No need to check this if we are moving to a new deeper level
        if (level == prev_level) {
            if ((level % 2 == 0) && (val <= prev_val)) {
                return false;
            }
            if ((level % 2 == 1) && (val >= prev_val)) {
                return false;
            }
        }

        prev_level = level;
        prev_val = val;
        if (node -> left != NULL) {
            q.push({node -> left, level + 1});
        }
        if (node -> right != NULL) {
            q.push({node -> right, level + 1});
        }
    }
    return true;
}
```
