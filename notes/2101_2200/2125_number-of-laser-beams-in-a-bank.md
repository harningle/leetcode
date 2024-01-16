# 2125. Number of Laser Beams in a Bank

Anti-theft security devices are activated inside a bank. You are given a **0-indexed** binary string array `bank` representing the floor plan of the bank, which is an $m\times n$ 2D matrix. `bank[i]` represents the $i$<sup>th</sup> row, consisting of `'0'`s and `'1'`s. `'0'` means the cell is empty, while `'1'` means the cell has a security device.

There is **one** laser beam between any **two** security devices **if both** conditions are met:

* The two devices are located on two **different rows**: $r_1$ and $r_2$, where $r_1 < r_2$.
* For **each** row $i$ where $r_1 < i < r_2$, there are **no security devices** in the $i$<sup>th</sup> row.

Laser beams are independent, i.e., one beam does not interfere nor join with another.

Return *the total number of laser beams in the bank*.


**Example:**

![](https://assets.leetcode.com/uploads/2021/12/24/laser1.jpg)

> **Input:** `bank = ["011001", "000000", "010100", "001000"]`
>
> **Output:** `8`
> 
> **Explanation:** Between each of the following device pairs, there is one beam. In total, there are 8 beams:
>
> * `bank[0][1]` -- `bank[2][1]`
> * `bank[0][1]` -- `bank[2][3]`
> * `bank[0][2]` -- `bank[2][1]`
> * `bank[0][2]` -- `bank[2][3]`
> * `bank[0][5]` -- `bank[2][1]`
> * `bank[0][5]` -- `bank[2][3]`
> * `bank[2][1]` -- `bank[3][2]`
> * `bank[2][3]` -- `bank[3][2]`
> 
> Note that there is no beam between any device on the 0<sup>th</sup> row with any on the 3<sup>rd</sup> row.
> 
> This is because the 2<sup>nd</sup> row contains security devices, which breaks the second condition.


## Brute force and simple combinatorics

We go through all rows, and count the #. of one's in each row. For two adjacent rows with $a$ and $b$ devices (can have empty rows inbetween), the #. of beams between them is $C_a^1 C_b^1 = a\times b$.

```c
#include <string.h>

int numberOfBeams(char** bank, int bankSize) {
    int n_cols = strlen(bank[0]);
    int n_ones_prev = 0;
    int n_ones_this;
    int ans = 0;

    // Go through all rows and count the one's in each row
    for (int i = 0; i < bankSize; i++) {
        n_ones_this = 0;
        for (int j = 0; j < n_cols; j++) {
            if (bank[i][j] == '1') {
                n_ones_this++;
            }
        }

        // If we have positive #. of devices in this row, find the previous positive row and calc.
        // the #. of beams
        if (n_ones_this > 0) {
            ans += n_ones_prev * n_ones_this;
            n_ones_prev = n_ones_this;  // Now the current row becomes the previous row in next
        }                               // iteration
    }
    return ans;
}
```
