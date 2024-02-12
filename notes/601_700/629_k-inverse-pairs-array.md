# 629. K Inverse Pairs Array

For an integer array `nums`, an **inverse pair** is a pair of integers $[i, j]$ where $0 <= i < j <$ `nums.length` and `nums[i] > nums[j]`.

Given two integers `n` and `k`, return the number of different arrays consist of numbers from $1$ to $n$ such that there are exactly `k` **inverse pairs**. Since the answer can be huge, return it **modulo** $10^9 + 7$.


**Example:**

> **Input:** `n = 3`, `k = 1`
> 
> **Output:** `2`
> 
> **Explanation:** The array `[1, 3, 2]` and `[2, 1, 3]` have exactly $1$ inverse pair.


## Top-down DP

It seems we could have a "math" sol.:

| $k$         | $0$ | $1$ | $2$ | $3$ |
|-------------|-----|-----|-----|-----|
| `[1]`       | 1   |     |     |     |
| `[1, 2]`    | 1   | 1   |     |     |
| `[1, 2, 3]` | 1   | 2   | 2   | 1   |

But actually we cannot observe the pattern. So DP is a naturally way to do it, bc. `[1, 2, 3, 4]` is basically `[1, 2, 3]` plus `4`. For each permutation of `[1, 2, 3]`, e.g. $1\ 3\ 2$, there are four potential <font color="red">po</font><font color="green">si</font><font color="orange">ti</font><font color="blue">on</font>s to put $4$: ${\color{red}\underline{}}1{\color{green}\underline{}}3{\color{orange}\underline{}}2\color{blue}\underline{}$. If we insert $4$ to the <font color="red">last position</font>, we won't create additional inverse pair, as $4$ is larger than any of `[1, 2, 3]`. Similarly, if we insert it to the <font color="blue">last but one position</font>, it creates one more inverse pair, bc. all numbers ahead are smaller than $4$ (so no extra pairs here), and any number after this position is smaller than $4$ (one number after $4$ so one more pair).

More generally, we have $i + 1$ positions to insert number $i + 1$ into a given permutation of $\{1, 2, \ldots, i\}$. Let $j$ (zero-indexed) be the position where we insert $i + 1$. Insertion creates $i - j$ new inverse pairs, and won't destroy any existing pairs as we doesn't change the relative order of the existing $i$ numbers. If there are $a$ inverse pairs in this permutation, then the new permutation of $i + 1$ numbers after insertion $i$ into $j$<sup>th</sup> position has $a + i - j$ inverse pairs in total. Let $dp(i, k)$ be the number of permutations of $\{1, 2, \ldots, i\}$ which have $k$ inverse pairs, then:

* base case: $dp(1, 0) = 1$, i.e. if there is one number only, there is no pair at all
* recurrence relation: $\displaystyle dp(i + 1, k) = \sum_{l = \max \{k - i, 0\}}^k dp(i, l)$. For any permutation that has $l$ inverse pairs, inserting $i$ to a proper position can lead to a new permutation with $k$ pairs. We can at most create $i$ new pairs by inserting $i + 1$ to the zero<sup>th</sup> position, so $l + i\geq k$. And we creates no new pair if inserting $i + 1$ to the last position, so $l\leq k$. Also we know that for $i + 1$ numbers, we can create at most $\dfrac{(i + 1)i}{2}$ by $\{i + 1, i, \ldots, 2, 1\}$.

### Implementation

```cpp
int kInversePairs(int n, int k) {
    // No sol.
    if (k > n * (n - 1) / 2) {
        return 0;
    }

    // Base case
    vector<long> prev_dp(2, 0);  // Don't need the entire DP table; only last
    prev_dp[0] = 1;              // one is needed
    prev_dp[1] = 1;

    // Recursion
    int prev_max_k;
    int ending_j;
    int ending_u;
    int mod = 1e9 + 7;
    for (int i = 2; i <= n; i++) {
        vector<long> this_dp(i * (i - 1) / 2 + 1, 0);
        ending_j = (i - 1) * (i - 2) / 2;
        ending_j = (ending_j >= k) ? k : ending_j;
        for (int j = 0; j <= ending_j; j++) {
            ending_u = (j + i - 1 >= k) ? k : j + i - 1;
            for (int u = j; u <= ending_u; u++) {
                this_dp[u] += prev_dp[j] % mod;
            }
        }
        prev_dp = this_dp;
    }
    return prev_dp[k] % mod;
}
```

This approach is $O(nk^2)$ and will in fact get `Time Limit Exceeded`. We can optimise it by using [telescoping sum](https://mathworld.wolfram.com/TelescopingSum.html). (Yes very hard to observe...)


## Top-down DP with optimised recurrence relation

Let's brute force the recurrence relation.[^summation limit]

[^summation limit]: Let's ignore the $0$ lower bound of the summation limit for now, and simply set it to be $l = k - i$.

$$
\begin{align*}
dp(i + 1, k) &= \sum_{l = k - i}^k dp(i, l)\\ 
 &= {\color{blue}dp(i, k - i) + dp(i, k - i + 1) + \ldots + dp(i, k - 1)} + dp(i, k)\\
dp(i + 1, k - 1) &= \sum_{l = k - 1 - i}^{k - 1} dp(i, l)\\ 
 &= dp(i, k - i - 1) + {\color{blue}dp(i, k - i) + dp(i, k - i + 1) + \ldots + dp(i, k - 2) + dp(i, k - 1)}\\
\end{align*}
$$

Therefore, $\displaystyle dp(i + 1, k) = dp(i + 1, k - 1) - dp(i, k - i - 1) + dp(i, k)$.[^sliding window] When computing $dp(i + 1, k)$, we already know $dp(i, \cdot)$ and $dp(\leq i, k)$, so all three terms on the RHS are known. That is, we no longer need the loop over $l$, so the time complexity is $O(nk)$.

[^sliding window]: You can also see it as some sort of sliding window. See [https://leetcode.com/problems/k-inverse-pairs-array/solutions/846076/c-4-solutions-with-picture](https://leetcode.com/problems/k-inverse-pairs-array/solutions/846076/c-4-solutions-with-picture).

```cpp
int kInversePairs(int n, int k) {
    // No sol. case
    if (k > n * (n - 1) / 2) {  // At most that many pairs for n numbers
        return 0;
    }

    // Base case: `nums = [1]`
    vector<long> prev_dp(k + 1, 0);
    prev_dp[0] = 1;

    // Recursion
    vector<long> this_dp;
    long term_1, term_2, term_3;
    int mod = 1e9 + 7;
    for (int i = 2; i <= n; i++) {
        this_dp = vector<long>(k + 1);
        for (int j = 0; j <= k; j++) {
            term_1 = (j >= 1) ? this_dp[j - 1] : 0;
            term_2 = (j >= i) ? prev_dp[j - i] : 0;
            term_3 = prev_dp[j];
            this_dp[j] = (term_1 - term_2 + term_3 + mod) % mod;
        }
        prev_dp = this_dp;
    }
    return prev_dp[k];
}
```

*Notes*: We many have some `int` overflow so use `long`. `term_1`, `term_2`, and `term_3` are surely within the range of `int`, but `term_1 - term_2 + term_3` can be larger than that. Also we need `+ mod` in the end bc. in C/C++ the `%` can return negative numbers, as opposed to Python.
