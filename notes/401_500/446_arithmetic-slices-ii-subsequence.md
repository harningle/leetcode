# 446. Arithmetic Slices II - Subsequence

Given an integer array `nums`, return <i>the number of all the <b>arithmetic subsequences</b> of</i> `nums`.

A sequence of numbers is called arithmetic if it consists of **at least three elements** and if the difference between any two consecutive elements is the same.

* For example, `[1, 3, 5, 7, 9]`, `[7, 7, 7, 7]`, and `[3, -1, -5, -9]` are arithmetic sequences.
* For example, `[1, 1, 2, 5, 7]` is not an arithmetic sequence.

A **subsequence** of an array is a sequence that can be formed by removing some elements (possibly none) of the array.

* For example, `[2, 5, 10]` is a subsequence of `[1, 2, 1, 2, 4, 1, 5, 10]`.

The test cases are generated so that the answer fits in **32-bit** integer.


**Example:**

> **Input:** `nums = [2, 4, 6, 8, 10]`
>
> **Output:** `7`
>
> **Explanation:** All arithmetic subsequence slices are:
>
> `[2, 4, 6]`
> 
> `[4, 6, 8]`
> 
> `[6, 8, 10]`
> 
> `[2, 4, 6, 8]`
> 
> `[4, 6, 8, 10]`
> 
> `[2, 4, 6, 8, 10]`
> 
> `[2, 6, 10]`


## DP

For a given `nums = [2, 4, 6, 8, 10]`, as a human we often start with left to right, and look at two numbers at the same time. When we see $2$ and $4$, we look for $6$. When we see $4$ and $8$, we check if $12$ exists. This translates naturally to DP. That is, if we have an arithmetic sequence $\{2, 4\}$ with common diff. $2$ found before,[^1] and now we see $6$, we then know $\{2, 4, 6\}$ is also an arithmetic sequence. If we have ten arithmetic sequences with common diff. $2$ ending with number $4$, then the #. of arithmetic sequences with common diff. $2$ ending with $6$ is ten plus one equalling eleven.

[^1]: Note that this length-2 arithmetic sequence $\{2, 4\}$ is not a valid answer, as the definition here asks for at least three numbers.

Formally, let the #. of arithmetic sequences ending with number at index $i$ with common diff. $d$ be $dp(i, d)$.

* base case: $dp(0, \cdot) = 0$, i.e. the first number itself cannot be an arithmetic sequence of any common diff.
* recurrence relation: $dp(i, d) = dp(i - d, d) + 1$, e.g. if we have $\{\color{blue}{0, 2, 4}\}, \{\color{blue}{2, 4}\}$ for $dp(\color{blue}{4}, 2)$, now we have $\{\color{blue}{0, 2, 4}, \color{red}{6}\}, \{\color{blue}{2, 4}, \color{red}{6}\}, \{4, \color{red}{6}\}$ for $dp(\color{red}{6}, 2)$

This problem has obvious optimal substructure: if $dp(10, 2)$ counts all arithmetic sequences like $\{\ldots, 8, 10\}$, then $dp(8, 2)$ must also count all sequences like $\{\ldots, 8\}$ with common diff. $2$. If we have more or less sequences like $\{\ldots, 8\}$ than $dp(8, 2)$, $dp(10, 2)$ won't be the solution to the original problem. Contradiction. $\blacksquare$

**NB:** $i$ is easy, but how to list all possible $d$’s? Since we are going to loop over $i$ anyway, we can do a second/nested loop over all numbers $j$ before $i$, then `nums[i]` $-$ `nums[j]` will include all possible $d$’s. And actually we cannot use `nums[i]`$- d$. Bc. `nums` may contain duplicate elements, so we have to use index to identify numbers, not the values themselves.

| | $\color{blue}i$              | $\color{red}j$             | $\color{green}d$ | <code>dp[<font color="red">j</font>][<font color="green">d</font>]</code>     | <code>dp[<font color="blue">i</font>][<font color="green">j</font>]</code>     | Arithmetic seq.                                                | Ans. |
|------|------------------|-----------------|-----------------------------|----------------|----------------|----------------------------------------------------------------|------|
| 1    | <code>nums[<font color="blue">1</font>]</code>$ = 4$  | <code>nums[<font color="red">0</font>]$ = 2$ | $\color{green}2$                 | <code>dp[<font color="red">0</font>][<font color="green">2</font>]</code>     | <code><font color="purple"><b>dp[<font color="blue">1</font>][<font color="green">2</font>] = 1</b></font></code> | $\color{purple}{\{2, 4\}}$                                                     |      |
| 2    | <code>nums[<font color="blue">2</font>]</code>$ = 6$  | <code>nums[<font color="red">0</font>]</code>$ = 2$ | $\color{green}4$                 | <code>dp[<font color="red">0</font>][<font color="green">4</font>]</code>     | <code><font color="orange"><b>dp[<font color="blue">2</font>][<font color="green">4</font>] = 1</b></font></code> | $\color{orange}{\{2, 6\}}$                                                     |      |
| 3    | <code>nums[<font color="blue">2</font>]</code>$ = 6$  | <code>nums[<font color="red">1</font>]</code>$ = 4$ | $\color{green}2$                 | <code><font color="purple"><b>dp[<font color="red">1</font>][<font color="green">2</font>] = 1</b></font></code> | <code><font color="seagreen"><b>dp[<font color="blue">2</font>][<font color="green">2</font>] = 2</b></font></code> | $\{\color{purple}{2, 4}, \color{seagreen}{6}\}, \{\color{purple}{4}, \color{seagreen}{6}\}$                                        | 1    |
| 4    | <code>nums[<font color="blue">3</font>]</code>$ = 8$  | <code>nums[<font color="red">0</font>]</code>$ = 2$ | $\color{green}6$                 | <code>dp[<font color="red">0</font>][<font color="green">6</font>]</code>      | <code>dp[<font color="blue">3</font>][<font color="green">6</font>] = 1</code> | $\{2, 8\}$                                                     |      |
| 5    | <code>nums[<font color="blue">3</font>]</code>$ = 8$  | <code>nums[<font color="red">1</font>]</code>$ = 4$ | $\color{green}4$                 | <code>dp[<font color="red">1</font>][<font color="green">4</font>]</code>      | <code>dp[<font color="blue">3</font>][<font color="green">4</font>] = 1</code> | $\{4, 8\}$                                                     |      |
| 6    | <code>nums[<font color="blue">3</font>]</code>$ = 8$  | <code>nums[<font color="red">2</font>]</code>$ = 6$ | $\color{green}2$                 | <code><font color="seagreen"><b>dp[<font color="red">2</font>][<font color="green">2</font>] = 2</b></font></code> | <code><b><font color="fuchsia">dp[<font color="blue">3</font>][<font color="green">2</font>] = 3</font></b></code> | $\{\color{seagreen}{2, 4, 6}, \color{fuchsia}{8}\}, \{\color{seagreen}{4, 6}, 8\}, \{\color{seagreen}{6}, \color{fuchsia}{8}\}$                        | 3    |
| 7    | <code>nums[<font color="blue">4</font>]</code>$ = 10$ | <code>nums[<font color="red">0</font>]</code>$ = 2$ | $\color{green}8$                | <code>dp[<font color="red">0</font>][<font color="green">8</font>]</code>      | <code>dp[<font color="blue">4</font>][<font color="green">8</font>] = 1</code> | $\{2, 10\}$                                                    |      |
| 8    | <code>nums[<font color="blue">4</font>]</code>$ = 10$ | <code>nums[<font color="red">1</font>]</code>$ = 4$ | $\color{green}6$                | <code>dp[<font color="red">1</font>][<font color="green">6</font>]</code>      | <code>dp[<font color="blue">4</font>][<font color="green">6</font>] = 1</code> | $\{4, 10\}$                                                    |      |
| 9    | <code>nums[<font color="blue">4</font>]</code>$ = 10$ | <code>nums[<font color="red">2</font>]</code>$ = 6$ | $\color{green}4$                | <code><font color="orange"><b>dp[<font color="red">2</font>][<font color="green">4</font>] = 1</b></font></code> | <code>dp[<font color="blue">4</font>][<font color="green">4</font>] = 2</code> | $\{\color{orange}{2, 6}, 10\}, \{\color{orange}{6}, 10\}$                                      | 4    |
| 10   | <code>nums[<font color="blue">4</font>]</code>$ = 10$ | <code>nums[<font color="red">3</font>]</code>$ = 8$ | $\color{green}2$                | <code><b><font color="fuchsia">dp[<font color="red">3</font>][<font color="green">2</font>] = 3</font></b></code> | <code>dp[<font color="blue">4</font>][<font color="green">2</font>] = 4</code> | $\{\color{fuchsia}{2, 4, 6, 8}, 10\}, \{\color{fuchsia}{4, 6, 8}, 10\}, \{\color{fuchsia}{6, 8}, 10\}, \{\color{fuchsia}{8}, 10\}$ | 7    |



## Implementation

```python
from collections import defaultdict


def numberOfArithmeticSlices(nums: List[int]) -> int:
    dp = [defaultdict(int) for _ in range(len(nums))]  # (1)
    ans = 0

    for i in range(1, len(nums)):
        for j in range(0, i):
            d = nums[i] - nums[j]
            if d in dp[j]:  # (2)
                dp[i][d] += dp[j][d] + 1
                ans += dp[j][d]  # (3)
            else:
                dp[i][d] += 1
    return ans
```

**(1). Why `[defaultdict _ in range(n)]` instead of `[defaultdict] * n`?** Bc. `[defaultdict] * n` creates only one `defaultdict` obj. and all `n` elements in the list point to the same obj. `[defaultdict _ in range(n)]` creates `n` independent objs. Generally speaking, all elements in `[x] * n` always point to the same `x`. But when `x` is immutable, e.g. `int`, changing the $i$<sup>th</sup> element will change where it points to, and other `x`’s are untouched. However, when `x` is mutable, e.g. `dict`, change one `x` will change the only obj. in RAM. And since all `n` `x`’s point to the same obj., all of them are changed.

**(2). Why check `d in dp[j]` while already use `defaultdict`?** Yes we can simply `dp[i][d] += dp[j][d] + 1` without checking `if d in dp[j]`. It works and the code is simpler. But it may waste more time and definitely more space. Checking if an element is in a hashmap is very fast ($O(1)$). If it's there, fine. If not, by checking it, we directly jump to `else`, without having to create `dp[j][d]`. If not checking, bc. we are using `defaultdict`, `dp[j][d]` will insert key `d` with value `0` to this hashmap. Inserting to a hashmap is slower, although still $O(1)$. 

**(3). Why `ans += dp[j][d]` gives the answer in the end?** It counts all arithmetic sequences ending with $\{$`nums[j]` $,$ `nums[i]`$\}$ with common diff. $d$. No duplicates, no omission. Note that here we don't `+ 1` as $\{$`nums[j]` $,$ `nums[i]`$\}$ itself is length-two and isn't a valid arithmetic sequence. Also see the diagram above: whenever we have a non-zero `dp[j][d]`, we find some new sequences and add that to `ans`.
