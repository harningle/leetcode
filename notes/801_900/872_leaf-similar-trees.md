# 872. Leaf-Similar Trees

Consider all the leaves of a binary tree, from left to right order, the values of those leaves form a **leaf value sequence**.

<img src="https://s3-lc-upload.s3.amazonaws.com/uploads/2018/07/16/tree.png" width="40%"/>

For example, in the given tree above, the leaf value sequence is `(6, 7, 4, 9, 8)`.

Two binary trees are considered *leaf-similar* if their leaf value sequence is the same.

Return `true` if and only if the two given trees with head nodes `root1` and `root2` are leaf-similar.
 
**Example:**

<img src="https://assets.leetcode.com/uploads/2020/09/03/leaf-similar-1.jpg" width="70%"/>

> **Input:** `root1 = [3, 5, 1, 6, 2, 9, 8, null, null, 7, 4], root2 = [3, 5, 1, 6, 7, 4, 2, null, null, null, null, null, null, 9, 8]`
> 
> **Output:** `true`


## DFS

DFS to get all leaves, from left to right, and compare the leaves of the two input trees.

```python
class Solution:
    def dfs(self, node: TreeNode) -> int:
        if node:
            if (node.left is None) and (node.right is None):
                yield node.val
            yield from self.dfs(node.left)
            yield from self.dfs(node.right)

    def leafSimilar(self,
                    root1: Optional[TreeNode],
                    root2: Optional[TreeNode]) -> bool:
        return list(self.dfs(root1)) == list(self.dfs(root2))
```

The `yield from dfs(node.left)` is simply a shortcut for:

```python
for i in dfs(node.left):
    yield i
```

We can improve the time by not DFS through the entire tree: if the first leaf in `root1` is different from first leaf in `root2`, we can return `False` and stop right there, which can be done by:

```python
return all(a == b for a, b in
           itertools.izip_longest(self.dfs(root1), self.dfs(root2)))
```
