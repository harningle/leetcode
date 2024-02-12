# 1043. Partition Array for Maximum Sum

Given an integer array `arr`, partition the array into (contiguous) subarrays of length **at most** `k`. After partitioning, each subarray has their values changed to become the maximum value of that subarray.

Return *the largest sum of the given array after partitioning. Test cases are generated so that the answer fits in a __32-bit__ integer*.


**Example:**

> **Input:** `arr = [1, 15, 7, 9, 2, 5, 10]`, `k = 3`
> 
> **Output:** `84`
> 
> **Explanation:** `arr` becomes `[15, 15, 15, 9, 10, 10, 10]`


## Bottom-up DP

I don't know how to brute force it actually. It just feels like a DP problem. Let $dp(i)$ be the solution to $\{arr_0, \ldots, arr_i\}$. We have:

* base case: $dp(0) = arr_0$. If we have the zero<sup>th</sup> number only, the only split is $\{0\}$
* recurrence relation: $\displaystyle dp(i) = \max_{1\leq j \leq k, i - j\geq 0} \Big (\underset{\text{Sol. to }\{0, \ldots, i - j\}}{\underbrace{dp(i - j)}} + \underset{\text{length of last split}}{\underbrace{j}}\times \underset{\text{largest number in last split}}{\underbrace{\max_{i - j + 1\leq k \leq i} arr_k}} \Big )$. The last split can be $\{i\}$, or $\{i - 1, i\}$, or $\{i - 2, i - 1, i\}$, etc. In each possible split, a candidate $dp$ is the previous $dp$ plus the split length times the largest number in the split. We try all possible ways, when we reach the split length limit of $k$, or when we reach the beginning of $arr$


```cpp
int maxSumAfterPartitioning(vector<int>& arr, int k) {
    int n = arr.size();
    vector<int> dp(n);

    // Base case: a split of the zero-th number itself
    dp[0] = arr[0];

    // Recursion
    int last_max;  // Max. number in the last split
    for (int i = 1; i < n; i++) {
        dp[i] = 0;
        last_max = arr[i];
        // Last split can be [i], [i - 1, i], ..., [i - k + 1, ..., i]
        // Find the best starting point of the last split
        for (int j = i; (j > i - k) && (j >= 0); j--) {
            last_max = (arr[j] > last_max) ? arr[j] : last_max;
            if (j - 1 >= 0) {  // `dp[j - 1]` need j >= 1
                dp[i] = max(dp[i], dp[j - 1] + last_max * (i - j + 1));
            } else {           // When `j = 0`, last split is all numbers
                dp[i] = max(dp[i], last_max * (i + 1));
            }
        }
    }
    return dp[n - 1];
}
```


### Memory optimisation

There is a way to optimise the space complexity from the above $O(n)$ to $O(k)$, though I don't like it. If we look at the limits of the $\max$ in recurrence relation, it's from $1$ to $k$ at most (we may have less if we reach the beginning of `arr`). So when solving $dp(i)$, we don't need the entire $dp(0)$ to $dp(i - 1)$, we only need $dp(i - k)$ to $dp(i - 1)$. Thus, at most $k$ elements in `dp`. Replacing all indices with their modulo $k$ will do the trick.
