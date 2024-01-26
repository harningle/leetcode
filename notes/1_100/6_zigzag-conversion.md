# 6. Zigzag Conversion

The string `"PAYPALISHIRING"` is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

> ```
> P   A   H   N
> A P L S I I G
> Y   I   R
> ```

And then read line by line: `"PAHNAPLSIIGYIR"`

Write the code that will take a string and make this conversion given a number of rows:

> `string convert(string s, int numRows);`

**Example:**

> **Input:** `s = "PAYPALISHIRING"`, `numRows = 4`
>
> **Output:** `"PINALSIGYAHRPI"`
>
> **Explanation:**
>
> ```
> P     I    N
> A   L S  I G
> Y A   H R
> P     I
> ```


## Brute force

In the first row, we have `"PIN"`, where

* `'P'` is the $0$<sup>th</sup> char. in the input string,
* `'I'` is the $6$<sup>th</sup> ($= 0 + 6$) (`'P'` going <font color="red">to bottom</font> and <font color="red">then up to</font> the `'I'` in <font color="red">top</font>), and
* `'N'` is the $12$<sup>th</sup> ($= 6 + 6$) (`'I'` <font color="red">to bottom</font> and <font color="red">then up to</font> `'N'` in <font color="red">top</font>).

In the second row, we have `"ALSIG"`, where

* `'A'` is the $1$<sup>st</sup> char. in the input string,
* `'L'` the $5$<sup>th</sup> ($= 1 + 4$) (`'A'` going <font color="blue">down to bottom</font> and <font color="blue">then up</font> to `'L'`),
* `'S'` the $7$<sup>th</sup> ($= 5 + 2$) (`'L'` <font color="orange">up to top</font> and <font color="orange">then down</font> to `'S'`),
* `'I'` the $11$<sup>th</sup> ($= 7 + 4$) (`'S'` <font color="blue">down to bottom</font> and <font color="blue">then up</font> to `'I'`), and finally
* `'G'` the $13$<sup>th</sup> ($= 11 + 2$) (`'I'` <font color="orange">up to top</font> and <font color="orange">then down</font> to `'G'`)

All the way to the final row, where we have `"PI"`:

* `'P'` is the $3$<sup>rd</sup> char. in the input string,
* `'I'` is the $9$<sup>th</sup> ($= 3 + 6$) (`'P'` going <font color="green">to top</font> and <font color="green">then down to</font> the `'I'` in <font color="green">bottom</font>)


We observe the pattern:

* for the top row, we go <font color="red">to bottom then up to top</font>
* for the bottom row, we go <font color="green">to top then down to bottom</font>
* for all rows inbetween, we do <font color="blue">down to bottom then up</font> and <font color="orange">up to top then down</font> in turns

There is nothing hard; just calc. the indices carefully.

```c
char* convert(char* s, int numRows) {
    int len_of_zig = 2 * (numRows - 1);  // How many steps it takes to start
                                         // from the first row and go back to
                                         // the first row
    int gap;      // The 2's and 4's in the above example, i.e. how many steps
                  // it takes to start from a row and be back to this row
    char* ans = (char*) malloc((strlen(s) + 1) * sizeof(char));
    int pos = 0;  // Index in the answer string
    int j;        // Index in the input string
    int temp;     // We have 2's and 4's in turn, so `temp` takes care of them

    // Corner case: if one single row, then return the original input string
    if (numRows == 1) {
        return s;
    }

    // Go through the rows
    for (int i = 0; i < numRows; i++) {

        // How many steps it takes to start from i-th row, go down to the
        // bottom row, and go up back to the i-th row
        gap = 2 * (numRows - i - 1);

        // Corner case: if we are at the bottom row, how many steps we need to
        //              go back again to the bottom row
        if (i == numRows - 1) {
            gap = 2 * (numRows - 1);
        }

        // Take the 2, 2 + 2, 2 + 2 + 4, 2 + 2 + 4 + 2, ... chars. from the
        // input string
        j = i;     // Start from the i-th char.
        temp = 0;  // We are going down first
        while (j < strlen(s)) {
            ans[pos] = s[j];
            // If we are going down, then the next step to take is j + gap-th
            if (temp == 0) {
                j += gap;
                temp = 1;  // If now we are going down, then for the next char.
                           // we will be going up

            // If we are going up
            } else {

                // Corner case: if we are at the top row
                if (2 * (numRows - 1) - gap > 0) {
                    j += 2 * (numRows - 1) - gap;
                } else {
                    j += gap;
                }
                temp = 0;  // Next we are going down
            }
            pos += 1;
        }
    }
    ans[strlen(s)] = '\0';
    return ans;
}
```

As we visit all chars. only once, so it's obviously $O(n)$.


## Get the string in each row and concat. them

Instead of the tedious indexing above, we can do it sequentially:

<div class="highlight"><pre>
<font color="red">P     I    N</font>
<font color="blue">A   L S  I G</font>
<font color="orange">Y A   H R</font>
<font color="green">P     I</font>
</pre></div>

| <code><font color="red">P</font></code>   | <code><font color="blue">A</font></code>  | <code><font color="orange">Y</font></code>    | <code><font color="green">P</font></code>    | <code><font color="orange">A</font></code>    | <code><font color="blue">L</font></code>  | <code><font color="red">I</font></code>   | <code><font color="blue">S</font></code>   | <code><font color="orange">H</font></code>    | <code><font color="green">I</font></code>     | <code><font color="orange">R</font></code>    | <code><font color="blue">I</font></code>  | <code><font color="red">N</font></code>   | <code><font color="blue">G</font></code> |
|-----------------------------------------  |------------------------------------------ |--------------------------------------------   |------------------------------ |--------------------------------------------   |------------------------------------------ |-----------------------------------------  |-----------------------------  |--------------------------------------------   |-------------------------------------------    |--------------------------------------------   |------------------------------------------ |-----------------------------------------  |-----------------------------  |
| <font color="red">1</font>                | <font color="blue">2</font>               | <font color="orange">3</font>                 | <font color="green">4</font>  | <font color="orange">3</font>                 | <font color="blue">2</font>               | <font color="red">1</font>                | <font color="blue">2</font>   | <font color="orange">3</font>                 | <font color="green">4</font>                  | <font color="orange">3</font>                 | <font color="blue">2</font>               | <font color="red">1</font>                | <font color="blue">2</font>   |

So we just assign 1, 2, 3, 4, 3, 2, 1, 2, 3, 4, ... to the input chars., and then output 1's first <code><font color="red">PIN</font></code>, then output 2's <code><font color="blue">ALSIG</font></code>, and then 3's <code><font color="orange">YAHR</font></code>, and finally 4's <code><font color="green">PI</font></code>. Concating them gives the answer <code><font color="red">PIN</font><font color="blue">ALSIG</font><font color="orange">YAHR</font><font color="green">PI</font></code>.

```c
char* convert(char* s, int numRows) {
    if (numRows == 1) {
        return s;
    }

    // Chars. in each row
    char** rows = (char**) malloc(numRows * sizeof(char*));
    for (int i = 0; i < numRows; i++) {
        rows[i] = (char*) malloc((strlen(s) + 1) * sizeof(char));
        rows[i][0] = '\0';
    }
    
    // Put input chars. into rows
    int step = 1;
    int row_idx = 0;
    int temp;
    for (int i = 0; i < strlen(s); i++) {
        temp = strlen(rows[row_idx]);
        rows[row_idx][temp] = s[i];
        rows[row_idx][temp + 1] = '\0';

        // If we are at 4, then next 4 3 2 1, so step = -1
        if (row_idx == numRows - 1) {
            step = -1;
        // If we are at 1, then next 1 2 3 4, so step = 1
        } else if (row_idx == 0) {
            step = 1;
        }
        row_idx += step;
    }
    char* ans = (char*) malloc((strlen(s) + 1) * sizeof(char));
    ans[0] = '\0';
    for (int i = 0; i < numRows; i++) {
        strcat(ans, rows[i]);
    }
    free(rows);
    return ans;
}
```

Please note that this approach is not necessarily faster than the the one above. We are using more spaces (bc. of `rows`), and the `strcat` runs at a linear time complexity. Moreover, though the intuition is simpler, managing the strings using `malloc` is not that easy to write in C...
