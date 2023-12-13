# 1582. Special Positions in a Binary Matrix

Given an $m \times n$ binary matrix `mat`, return the number of special positions in `mat`.

A position $(i , j)$ is called special if `mat[i][j] == 1` and all other elements in row $i$ and column $j$ are $0$ (rows and columns are 0-indexed).

 
Example:

| <span style="font-weight:normal">1</span> | <span style="font-weight:normal">0</span> | <span style="font-weight:normal">0</span> |
|---|---|---|
| 0 | 0 | 1 |
| 1 | 0 | 0 |

> Input: `mat = [[1,0,0],[0,0,1],[1,0,0]]`
> 
> Output: `1`
> 
> Explanation: $(1, 2)$ is a special position because `mat[1][2] == 1` and all other elements in row $1$ and column $2$ are $0$.


## One pass

If we solve it by brute force, we check $(0, 0)$ first. All other elements in row 0 are $0$. Good. There is another $1$'s in col. 0, so $(0, 0)$ is not the solution. Then we don't need to check col. 0 any further, as we already know that it has more than one $1$'s so all cells in this col. cannot be the solution. So the problem reduces to find cols. and rows with one single one's: only they can be sol. If row 1, 3 and col 3, 4 are such rows and cols., then all we need is to check if $(1, 3), (1, 4), (3, 3), (3, 4)$ is one.

## Sol.

```c
int numSpecial(int** mat, int matSize, int* matColSize) {
    // How many 1's in each row and col.
    int row_1_cnts[matSize];
    memset(row_1_cnts, 0, sizeof(row_1_cnts));
    int col_1_cnts[matColSize[0]];
    memset(col_1_cnts, 0, sizeof(col_1_cnts));
    for (int i = 0; i < matSize; i++) {
        for (int j = 0; j < matColSize[i]; j++) {
            if (mat[i][j] == 1) {
                row_1_cnts[i]++;
                col_1_cnts[j]++;
            }
        }
    }

    // When row i and col. j both have one 1's, then check if (i, j) is 1. If yes, that's the special position
    int ans = 0;
    for (int i = 0; i < matSize; i++) {
        if (row_1_cnts[i] == 1) {
            for (int j = 0; j < matColSize[i]; j++) {
                if (col_1_cnts[j] == 1) {
                    if (mat[i][j] == 1) {
                        ans += 1;
                    }
                }
            }
        }
    }
    return ans;
}
```


