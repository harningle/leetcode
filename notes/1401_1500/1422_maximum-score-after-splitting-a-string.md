# 1422. Maximum Score After Splitting a String

Given a string `s` of zeros and ones, <i>return the maximum score after splitting the string into two <b>non-empty</b> substrings</i> (i.e. **left** substring and **right** substring).

The score after splitting a string is the number of **zeros** in the **left** substring plus the number of **ones** in the **right** substring.

**Example:**

> **Input:** `s = "011101"`
> 
> **Output:** `5`
> 
> **Explanation:**

> All possible ways of splitting `s` into two non-empty substrings are:
> 
> `left = "0"` and `right = "11101"`, $score = 1 + 4 = 5$
> 
> `left = "01"` and `right = "1101"`, $score = 1 + 3 = 4$
> 
> `left = "011"` and `right = "101"`, $score = 1 + 2 = 3$
> 
> `left = "0111"` and `right = "01"`, $score = 1 + 1 = 2$
> 
> `left = "01110"` and `right = "1"`, $score = 2 + 1 = 3$


## One pass + math

Let the string length be $N$, and there are $N_0$ zero's in total. Then the total #. of one's is $N_1 = N - N_0$. Suppose we now split the string into left and right substrings. If we have $n_{l0}$ and $n_{l1}$ zero's and one's in the left substring, then in the right substring there are by definition $n_{r1} = N_1 - n_{l1}$ one's. The score is then

$$
\begin{align*}
score &= \color{red}{n_{l0}} + \color{blue}{n_{r1}} \\
      &= \color{red}{n_{l0}} + \color{blue}{N_1 - n_{l1}} \\
      &= \color{red}{n_{l0}} - \color{blue}{n_{l1}} + \underset{\text{Const.}}{\underbrace{\color{blue}{N_1}}} \\
\end{align*}
$$

To max. $score$, we simply need to go through all possible splits and find the largest $n_{l0} - n_{l1}$.

PS: by definition substrings are non-empty, so we must guarantee the right substring has at least one char.


## Sol.

```c
#include <string.h>

int maxScore(char* s) {
    int n_chars = strlen(s);
    int n_l1 = 0;
    int n_l0 = 0;
    int ans = INT_MIN;

    // Go through all possible splits
    for (int i = 0; i < n_chars - 1; i++) {  // Loop ends at `n_chars - 1` as we need a non-empty
        if (s[i] == '0') {                   // right substring
            n_l0++;
        } else {
            n_l1++;
        }
        if (n_l0 - n_l1 > ans) {
            ans = n_l0 - n_l1;
        }
    }
    // Here `n_l1` is the #. of one's in the string except the last char.

    // Check if the last char. is a one. If yes, adds to the total count of one's
    if (s[n_chars - 1] == '1') {
        n_l1++;
    }
    return ans + n_l1;
}
```
