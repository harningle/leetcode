# 2444. Count Subarrays With Fixed Bounds

You are given an integer array `nums` and two integers `minK` and `maxK`.

A **fixed-bound subarray** of `nums` is a subarray that satisfies the following conditions:

* the **minimum** value in the subarray is equal to minK
* the **maximum** value in the subarray is equal to maxK

Return *the __number__ of fixed-bound subarrays*.

A **subarray** is a **contiguous** part of an array.

 
**Example:**

> **Input:** `nums = [2, 1, 3, 5, 2, 7, 5], minK = 1, maxK = 5`
>
> **Output:** `4`
>
> **Explanation:** The fixed-bound subarrays are `[2, 1, 3, 5]`, `[1, 3, 5]`, `[2, 1, 3, 5, 2]`, and `[1, 3, 5, 2]`.


## DP

I first thought the problem can be solve by sliding window (it indeed can), but cannot figure out how. Then I was thinking how to improve on the $O(n^2)$ brute force. Some calculation can be optimised: if $[l, r]$ is a valid subarray, then if $nums_{r + 1}$ is within the bound, we know $[l, r + 1]$ is also a valid subarray, and all valid subarrays ending at $r$ are still valid if we append $r + 1$ to it. This naturally motivates DP: the #. of subarrays ending at index $i + 1$ is related to #. of subarrays ending at index $i$, depending on whether $nums_{i + 1}$ is within the bound.

After a second thought, the transitions are more complicated than I expected. Depending on the size of $nums_{i + 1}$, it may make *some* of previously invalid subarray become valid now. E.g., if we already got `minK` before, but the maximum so far is below `maxK`, then if $nums_{i + 1} =$ `maxK`, it will make all previous subarrays valid, plus previously already valid subarrays still valid. Formally, let

$$
dp(i, s) = 
\begin{cases}
\text{#. of valid subarrays ending at }i & \text{, } s = 0 \\ 
\text{#. of subarrays ending at }i\text{, w/ }\texttt{minK}\text{ but w/o }\texttt{maxK} & \text{, } s = 1 \\ 
\text{#. of subarrays ending at }i\text{, w/o }\texttt{minK}\text{ but w/ }\texttt{maxK} & \text{, } s = 2 \\ 
\text{#. of subarrays ending at }i\text{, w/o }\texttt{minK}\text{ and w/o }\texttt{maxK} & \text{, } s = 3 
\end{cases}
$$

* base case: $dp(0, \cdot)$ can easily be got by checking if $nums_0$ is within the bound
* recurrence relation: 
    
    | $nums_{i + 1}$               | $dp(i + 1, 0)$        | $dp(i + 1, 1)$            | $dp(i + 1, 2)$            | $dp(i + 1, 3)$ |
    |------------------------------|-----------------------|---------------------------|---------------------------|----------------|
    | outside bound                | $0$                   | $0$                       | $0$                       | $0$            |
    | $=$ `minK`, $=$ `maxK`       | $dp(i, 0) + 1$        | $0$                       | $0$                       | $0$            |
    | $=$ `minK`, $<$ `maxK`       | $dp(i, 0) + dp(i, 2)$ | $dp(i, 1) + 1 + dp(i, 3)$ | $0$                       | $0$            |
    | $>$ `minK`, $=$ `maxK`       | $dp(i, 0) + dp(i, 3)$ | $0$                       | $dp(i, 2) + 1 + dp(i, 3)$ | $0$            |
    | $>$ `minK`, $<$ `maxK`       | $dp(i, 0)$            | $dp(i, 1)$                | $dp(i, 2)$                | $dp(i, 3) + 1$ |

The recurrence relation is (mathematically) easy, but be careful not to miss anything. And as we only need $dp(i, \cdot)$ for $dp(i - 1, \cdot)$, i.e. don't need the entire history but only the last state, we can make it $O(1)$ space complexity.

```cpp
long long countSubarrays(vector<int>& nums, int minK, int maxK) {
    vector<long long> prev(4, 0), curr(4, 0);  // Only need last and this state
    long long ans = 0;
    for (int i = 0; i < nums.size(); i++) {
        int val = nums[i];
        if (minK <= val && val <= maxK) {
            if (val == minK && val == maxK) {
                curr[0] = prev[0] + 1;
                curr[1] = 0;
                curr[2] = 0;
                curr[3] = 0;
            } else if (val == minK && val < maxK) {
                curr[0] = prev[0] + prev[2];
                curr[1] = prev[1] + prev[3] + 1;
                curr[2] = 0;
                curr[3] = 0;
            } else if (val > minK && val == maxK) {
                curr[0] = prev[0] + prev[1];
                curr[1] = 0;
                curr[2] = prev[2] + prev[3] + 1;
                curr[3] = 0;
            } else if (val > minK && val < maxK) {
                curr[0] = prev[0];
                curr[1] = prev[1];
                curr[2] = prev[2];
                curr[3] = prev[3] + 1;
            }
        }
        ans += curr[0];
        swap(curr, prev);
        curr = vector<long long>(4, 0);
    }
    return ans;
}
```

The complexity is $O(n\times 4) = O(n)$, as the DP table $dp(i, s)$ has $0\leq i < n$ and $s \in \{1, 2, 3, 4\}$.


## Sliding window

The better solution is of course sliding window. Usually problems like counting the #. of contiguous subarray can be solved by sliding window. Here a valid window shall look like: some (or none) within-bound numbers, between which `minK` and `maxK` are located. E.g. `minK` $= \color{blue}2$ and `maxK` $= \color{red}5$, and the right pointer is now at index $i$: $\{\cdots, {\color{green}-1}, 3, {\color{red}5}, 3, 4, {\color{blue}2}, 3, nums_i\}$.

* if $nums_i$ is outside the bound, easy, zero valid subarray ending at $i$.
* if $nums_i$ is within the bound, then $\{3, {\color{red}5}, 3, 4, {\color{blue}2}, 3, nums_i\}$ and $\{{\color{red}5}, 3, 4, {\color{blue}2}, 3, nums_i\}$ are two valid subarrays ending at $i$

Now it's clear how to count the subarrays:

1. we need to know the rightmost outside-bound number $\color{green}-1$, which determines where the valid subarrays start
1. we also need the indices of `minK` $\color{blue}2$ and `maxK` $\color{red}5$
1. the gap between <font color="green">outside-bound number</font> and the lefter of <font color="blue">`minK`</font> and <font color="red">`maxK`</font> is the #. of valid subarrays ending at $i$

```cpp
long long countSubarrays(vector<int>& nums, int minK, int maxK) {
    long long ans = 0;
    int idx_bad = -1, idx_min = -1, idx_max = -1;
    int n = nums.size();
    for (int i = 0; i < n; i++) {
        if (nums[i] < minK || nums[i] > maxK) {
            idx_bad = i;
            continue;
        }
        if (nums[i] == minK) idx_min = i;
        if (nums[i] == maxK) idx_max = i;
        ans += max(0, min(idx_min, idx_max) - idx_bad);
    }
    return ans;
}
```

It's one pass so $O(n)$, and is roughly four times faster than DP above. The [code from lee215](https://leetcode.com/problems/count-subarrays-with-fixed-bounds/solutions/2708099/java-c-python-sliding-window-with-explanation) is also very nice:

* initial values of `idx_bad`, `idx_min`, and `idx_max` are carefully chosen, so we don't need to worry about the starting point
* `minK` and `maxK` can be the same, so don't use `else if` as the last two if conditions can be true at the same time
* the `max(0, gap)` take cares of where the outside-bound number is to the right of `minK` and `maxK`
