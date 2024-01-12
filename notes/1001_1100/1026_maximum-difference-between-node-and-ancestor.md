# 1026. Maximum Difference Between Node and Ancestor

Given the `root` of a binary tree, find the maximum value `v` for which there exist **different** nodes `a` and `b` where `v = |a.val - b.val|` and `a` is an ancestor of `b`.

A node `a` is an ancestor of `b` if either: any child of `a` is equal to `b` or any child of `a` is an ancestor of `b`.

 
**Example:**

![](https://assets.leetcode.com/uploads/2020/11/09/tmp-tree.jpg)

> **Input:** `root = [8, 3, 10, 1, 6, null, 14, null, null, 4, 7, 13]`
> 
> **Output:** `7`
> 
> **Explanation:** We have various ancestor-node differences, some of which are given below:
> 
> $|8 - 3| = 5$
> 
> $|3 - 7| = 4$
> 
> $|8 - 1| = 7$
> 
> $|10 - 13| = 3$
> 
> Among all possible differences, the maximum value of $7$ is obtained by $|8 - 1| = 7$.


## DFS

The brute force solution is: memorise all parent/ancestor nodes, and calc. the abs. diff. between all of them and the current node, and finally find the largest one. But we don't actually need all the parents. The target is $\max v = |a - b|$. WLOG, assume $a \geq b$. Then we want to $\max v = a - b$, so all we need is the largest $a$ and the smallest $b$ in the tree. Then the solution is very clear: DFS from root to end leaf, in each path, find the largest and smallest node value, and update the max. abs. diff.

The min. and max. above is the min. and max. along a path, not the tree global min. and max. This guarantees we are tracking the correct min. and max. E.g., $8\to 10 \to 14\to 13$ has a max. of $14$ and min. of $8$, but this $14$ and $8$ will not affect $8\to 3\to 6\to 7$, as when we call DFS on the left node $3$ of $8$, we carry the max. and min. at $8$, not at other nodes.

```c
#include <math.h>

int dfs(struct TreeNode* node, int min_val, int max_val) {
    if (node == NULL) {
        return max_val - min_val;
    }

    min_val = fmin(node -> val, min_val);
    max_val = fmax(node -> val, max_val);
    return fmax(dfs(node -> left, min_val, max_val),
                dfs(node -> right, min_val, max_val));
}

int maxAncestorDiff(struct TreeNode* root) {
    return dfs(root, INT_MAX, INT_MIN);
}
``` 
