# 1463. Cherry Pickup II

You are given a $rows\times cols$ matrix `grid` representing a field of cherries where `grid[i][j]` represents the number of cherries that you can collect from the $(i, j)$ cell.

You have two robots that can collect cherries for you:

* **robot #1** is located at the **top-left corner** $(0, 0)$, and
* **robot #2** is located at the **top-right corner** $(0, cols - 1)$.

Return *the maximum number of cherries collection using both robots by following the rules below*:

* from a cell $(i, j)$, robots can move to cell $(i + 1, j - 1)$, $(i + 1, j)$, or $(i + 1, j + 1)$
* when any robot passes through a cell, It picks up all cherries, and the cell becomes an empty cell.
* when both robots stay in the same cell, only one takes the cherries
* both robots cannot move outside of the grid at any moment
* both robots should reach the bottom row in `grid`


**Example:**

![](https://assets.leetcode.com/uploads/2020/04/29/sample_1_1802.png)

> **Input:** `grid = [[3, 1, 1], [2, 5, 1], [1, 5, 5], [2, 1, 1]]`
> 
> **Output:** `24`
> 
> **Explanation:** Path of robot #1 and #2 are described in color green and blue respectively.
>
> Cherries taken by Robot #1, $3 + 2 + 5 + 2 = 12$.
> 
> Cherries taken by Robot #2, $1 + 5 + 5 + 1 = 12$.
> 
> Total of cherries: $12 + 12 = 24$.


## DP

In each step, a robot can move to south-west, south, and south-east, and the current move will affect future moves. This is a typical DP setting.[^both move] Let $dp\big ((x_1, y_1), (x_2, y_2)\big )$ be the number of cherries when robot #1 is in cell $(x_1, y_1)$ and robot #2 is in $(x_2, y_2)$.

[^both move]: It's obvious that the DP shall consider both robots move at the same time. That is, in step $i$, both robots move from row $i - 1$ to row $i$. Bc. after any robot touches a cell, the cherries are picked, so it will affect the other robot's gain. The DP takes this into account by move both robots simultaneously.

* base case: $dp\big ((0, 0), (0, n\_cols - 1)\big ) =$ `grid[0][0]` $+$ `grid[0][n_cols]`. The starting point is robots in top left and top right of the matrix
* recurrence relation:
    
    $$
    dp\big ((x_1, y_1), (x_1, y_2)\big ) = \underset{\text{Max. #. of cherries when both robots are in the previous row before moving}}{\underbrace{\max_{\Delta y_1, \Delta y_2\in \{-1, 0, 1\}} dp\big ((x_1 - 1, y_1 - \Delta y_1), (x_1 - 1, y_2 - \Delta y_2)\big )}} + \underset{\text{Cherries in the current cells}}{\underbrace{gain}}
    $$

    , where $gain =$ <code>grid[x<sub>1</sub>][y<sub>1</sub>]</code> $+ \mathbb{1}(y_1 - \Delta y_1 \neq y_2 - \Delta y_2)$ <code>grid[x<sub>1</sub>][y<sub>2</sub>]</code>, and both $(x_1 - 1, y_1 - \Delta y_1)$ and $(x_1 - 1, y_2 - \Delta y_2)$ are doable.

    The intuition is, for each robot, it can move to the current $(x_i, y_i)$ from north-west, north, and north-east cells $(x_i - 1, y_i - 1)$, $(x_1 - 1, y_i)$, $(x_i - 1, y_i + 1)$. The same holds for the other robot, so we have $3\times 3 = 9$ possible ways of moving. But not all those ways are doable: $(x_i - 1, y_i - 1)$ may be outside the `grid` matrix, or cannot be touched in the previous move, e.g. robot #1 starts at $(0, 0)$ so it cannot touch $(1, 100)$ in row $1$. Among all doable moves, we find the best one


```cpp
int cherryPickup(vector<vector<int>>& grid) {
    int n_rows = grid.size();
    int n_cols = grid[0].size();
    vector<vector<int>> prev_dp(n_cols, vector<int>(n_cols, -1));

    // Base case
    prev_dp[0][n_cols - 1] = grid[0][0] + grid[0][n_cols - 1];

    // Recursion from row one (second row)
    int prev_y_1;
    int prev_y_2;
    int gain;
    int prev_best;
    for (int x = 1; x < n_rows; x++) {
        vector<vector<int>> current_dp(n_cols, vector<int>(n_cols, -1));

        // Try all possible col. positions in row `x` for both robots
        for (int y_1 = 0; y_1 < min(x + 1, n_cols); y_1++) {
            for (int y_2 = max(0, n_cols - 1 - x); y_2 < n_cols; y_2++) {

                // Try all possible col. positions at the previous row
                prev_best = -1;
                for (int delta_y_1 = -1; delta_y_1 <= 1; delta_y_1++) {
                    for (int delta_y_2 = -1; delta_y_2 <= 1; delta_y_2++) {
                        prev_y_1 = y_1 - delta_y_1;
                        prev_y_2 = y_2 - delta_y_2;

                        // Check if previous position is doable
                        if ((prev_y_1 >= 0) && (prev_y_1 < n_cols)
                            && (prev_y_2 >= 0) && (prev_y_2 < n_cols)
                            && prev_dp[prev_y_1][prev_y_2] >= 0) {
                            prev_best = max(prev_best,
                                            prev_dp[prev_y_1][prev_y_2]);
                        }
                    }
                }

                // Check if both robots are in the same cell or not
                gain = (y_1 == y_2) ? grid[x][y_1]
                                    : grid[x][y_1] + grid[x][y_2];
                current_dp[y_1][y_2] = prev_best + gain;
            }
        }
        prev_dp = current_dp;
    }

    // In the last row, find the largest DP
    int ans = -1;
    for (int y_1 = 0; y_1 < n_cols; y_1++) {
        for (int y_2 = 0; y_2 < n_cols; y_2++) {
            ans = max(prev_dp[y_1][y_2], ans);
        }
    }
    return ans;
}
```

The complexity is $O(n\_rows\times n\_cols^2)$, easily inferred from the nested loops.

*Notes*: we did some optimisation above:

1. we init. DP with value $-1$, a placeholder for unvisited cells
1. as both robots are always in the same row, so we don't really need a 4D DP table, but rather a 3D one $dp(x, y_1, y_2)$. In principle 4D DP table has the same time complexity, but it is *much* closer in practice
1. we don't need the entire DP, but only the DP of the current and previous row, bc. it's memoryless/has Markov property (when we are at row $x$ we only look at row $x - 1$ but don't case rows before $x - 1$)
1. we can simplify some "doable" conditions. E.g., robot #1 can move at most one cell right in each step, thus after $x$ steps, the right-est possible position is $x$ (or the right boundary of `grid`). This is where `y_1 < min(x + 1, n_cols)` and `max(0, n_cols - 1 - x)` comes from
