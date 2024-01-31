# 198. House Robber

You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security systems connected and **it will automatically contact the police if two adjacent houses were broken into on the same night**.

Given an integer array `nums` representing the amount of money of each house, return *the maximum amount of money you can rob tonight __without alerting the police__*.

 
**Example:**

> **Input:** `nums = [2, 7, 9, 3, 1]`
> 
> **Output:** `12`
> 
> **Explanation:** Rob house $1$ (money = $2$), rob house $3$ (money = $9$) and rob house $5$ (money = $1$).
>
> Total amount you can rob $= 2 + 9 + 1 = 12$.


## DP

It's a classic DP problem, very similar to knapsack. For each house, we decide whether to rob or to skip it. Let $dp(i)$ be the max. money after robbing or skipping house $i$. Obviously it has an optimal substructure, and

* base case: $dp(0) = nums_0$, i.e. if we only have one house, rob it
* recurrence relation: $dp(i) = \max(dp(i - 1), dp(i - 2) + nums_i)$. If we skip house $i$, we carry the money at house $i - 1$ over. If we rob it, then we add house $i$’s money to the money after house $i - 2$, as we are not allowed to rob adjacent houses. We take the larger one of the two possible choices.

```c
int rob(int* nums, int numsSize) {
    // If only one house, rob it
    if (numsSize == 1) {
        return nums[0];
    }

    int dp[numsSize];  // dp[i] = max. money after house i
    dp[0] = nums[0];   // Base case
    dp[1] = (nums[1] > nums[0]) ? nums[1] : nums[0];  // Corner case, as there
                                                      // is no house i - 2 for
    int rob;                                          // house 1 
    int skip;
    for (int i = 2; i < numsSize; i++) {
        rob = dp[i - 2] + nums[i];
        skip = dp[i - 1];
        dp[i] = (rob > skip) ? rob : skip;
    }
    return dp[numsSize - 1];
}
```
