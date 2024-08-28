# 1905. Count Sub Islands

You are given two $m\times n$ binary matrices `grid1` and `grid2` containing only `0`’s (representing water) and `1`’s (representing land). An **island** is a group of `1`’s connected **4-directionally** (horizontal or vertical). Any cells outside of the grid are considered water cells.

An island in `grid2` is considered a **sub-island** if there is an island in `grid1` that contains **all** the cells that make up **this** island in `grid2`.

Return the *__number__ of islands in* `grid2` *that are considered __sub-islands__*.


**Example:**

![](https://assets.leetcode.com/uploads/2021/06/10/test1.png)

> **Input:** `grid1 = [[1, 1, 1, 0, 0], [0, 1, 1, 1, 1], [0, 0, 0, 0, 0], [1, 0, 0, 0, 0], [1, 1, 0, 1, 1]]`, `grid2 = [[1, 1, 1, 0, 0], [0, 0, 1, 1, 1], [0, 1, 0, 0, 0], [1, 0, 1, 1, 0], [0, 1, 0, 1, 0]]`
> 
> **Output:** `3`
> 
> **Explanation:** In the picture above, the grid on the left is `grid1` and the grid on the right is `grid2`.
>
> The `1`’s coloured red in `grid2` are those considered to be part of a sub-island.
> 
> There are three sub-islands.


## DFS

The intuition is straightforward: for each `1`’s in `grid2`, DFS. If there is any cell in the corresponding position in `grid1` is `0`, then not a valid sub-island. However, the order of implementation is important. In a usual DFS, we maintain a `visited` array to track whether a cell has already been visited or not. During DFS, if some corresponding cell in `grid1` is water, we want to early stop, as the island in `grid2` won't be a valid sub-island, so there is no point keep checking.

Is it?

NO!!! If we early stop at `grid1[i][j] == 0`, then we don't fully explore the island in `grid2`. E.g., an island in `grid2` is $[x, x, x]$. When we check vertically for the first $x$, maybe in `grid1` there is water, so we early stop. At this point, the second and third $x$ aren't touched! So when we loop over `grid2` and see the second $x$, we think it's a new island and start a DFS again. This way would split islands in `grid2`. So we can't early stop. No matter what is in `grid1`, we have to fully DFS through an island in `grid2`. The rest is trivial.


```cpp
bool dfs(vector<vector<int>>& grid1, vector<vector<int>>& grid2, int i, int j,
         vector<vector<bool>>& visited) {
    if (i >= grid2.size() || i < 0) return true;
    if (j >= grid2[0].size() || j < 0) return true;
    if (grid2[i][j] == 0) return true;
    
    if (visited[i][j]) return true;
    visited[i][j] = true;
    
    bool ans = true;
    ans &= dfs(grid1, grid2, i + 1, j, visited);
    ans &= dfs(grid1, grid2, i - 1, j, visited);
    ans &= dfs(grid1, grid2, i, j + 1, visited);
    ans &= dfs(grid1, grid2, i, j - 1, visited);
    return ans & grid1[i][j];  // Must check grid1[i][j] here. No early stop!
}

int countSubIslands(vector<vector<int>>& grid1, vector<vector<int>>& grid2) {
    int ans = 0;
    int n = grid2.size();
    int m = grid2[0].size();
    vector<vector<bool>> visited(n, vector<bool>(m, false));
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (grid2[i][j] == 1 && visited[i][j] == false) {
                ans += dfs(grid1, grid2, i, j, visited);
            }
        }
    }
    return ans;
}
```

If you want to be faster, we can modify `grid2` and use it as `visited`, instead of building another new array.

```cpp
bool dfs(vector<vector<int>>& grid1, vector<vector<int>>& grid2, int i, int j) {
    if (i >= grid2.size() || i < 0) return true;
    if (j >= grid2[0].size() || j < 0) return true;
    if (grid2[i][j] == 0) return true;

    grid2[i][j] = 0;  // This is `visited` basically
    
    bool ans = true;
    ans &= dfs(grid1, grid2, i + 1, j);
    ans &= dfs(grid1, grid2, i - 1, j);
    ans &= dfs(grid1, grid2, i, j - 1);
    ans &= dfs(grid1, grid2, i, j + 1);

    return ans & grid1[i][j];
}

int countSubIslands(vector<vector<int>>& grid1, vector<vector<int>>& grid2) {
    ...
}
```
