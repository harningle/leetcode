# 2482. Difference Between Ones and Zeros in Row and Column

You are given a **0-indexed** $m \times n$ binary matrix `grid`.

A **0-indexed** $m \times n$ difference matrix `diff` is created with the following procedure:

* Let the number of ones in the $i$th row be $onesRow_i$.
* Let the number of ones in the $j$th column be $onesCol_j$.
* Let the number of zeros in the $i$th row be $zerosRow_i$.
* Let the number of zeros in the $j$th column be $zerosCol_j$.
* `diff[i][j]` $= onesRow_i + onesCol_j - zerosRow_i - zerosCol_j$

Return the difference matrix `diff`.

 
Example:

`grid`:

| <span style="font-weight:normal">0</span> | <span style="font-weight:normal">1</span> | <span style="font-weight:normal">1</span> |
|---|---|---|
| 1 | 0 | 1 |
| 0 | 0 | 1 |

`diff`:

| <span style="font-weight:normal">0</span> | <span style="font-weight:normal">0</span> | <span style="font-weight:normal">4</span> |
|---|---|---|
| 0 | 0 | 4 |
| -2 | -2 | 2 |

> **Input**: `grid = [[0,1,1],[1,0,1],[0,0,1]]`
> **Output**: `[[0,0,4],[0,0,4],[-2,-2,2]]`


## Brute force

Count how many one's are in a given row. Since we know the size of the input matrix, the #. of zero's are then the #. of cols. minus the one's. E.g., given a $10 \times 7$ matrix, if there are $3$ one's in row 2, then there are $7 - 3 = 4$ zero's in this row. So we just need to loop over the matrix once.


## Sol.

```c
/**
 * Return an array of arrays of size *returnSize.
 * The sizes of the arrays are returned as *returnColumnSizes array.
 * Note: Both returned array and *columnSizes array must be malloced, assume caller calls free().
 */
int** onesMinusZeros(int** grid, int gridSize, int* gridColSize, int* returnSize, int** returnColumnSizes) {
    // Init. result array
    int n_rows = gridSize;
    int n_cols = gridColSize[0];
    int** diff = malloc(n_rows * sizeof(int*));  // Must malloc in this way
    for (int i = 0; i < n_rows; i++) {
        diff[i] = malloc(n_cols * sizeof(int));
    }

    // Count the one's in a given row/col.
    int* ones_row = calloc(n_rows, sizeof(int));
    int* ones_col = calloc(n_cols, sizeof(int));
    for (int i = 0; i < n_rows; i++) {
        for (int j = 0; j < n_cols; j++) {
            if (grid[i][j] == 1) {
                ones_row[i]++;
                ones_col[j]++;
            }
        }
    }

    // The zero's is then #. of cells in the row/col. minus the one's

    // Calc. diff
    for (int i = 0; i < n_rows; i++) {
        for (int j = 0; j < n_cols; j++) {
            diff[i][j] = ones_row[i] + ones_col[j] - (n_cols - ones_row[i]) - (n_rows - ones_col[j]);
        }
    }
    *returnSize = n_rows;
    *returnColumnSizes = malloc(n_rows * sizeof(int));
    for (int i = 0; i < n_rows; i++){
        (*returnColumnSizes)[i] = n_cols;
    }
    return diff;
}
```
