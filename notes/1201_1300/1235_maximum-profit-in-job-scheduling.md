# 1235. Maximum Profit in Job Scheduling

We have `n` jobs, where every job is scheduled to be done from `startTime[i]` to `endTime[i]`, obtaining a profit of `profit[i]`.

You're given the `startTime`, `endTime` and `profit` arrays, return the maximum profit you can take such that there are no two jobs in the subset with overlapping time range.

If you choose a job that ends at time `X` you will be able to start another job that starts at time `X`.

**Example:**

![](https://assets.leetcode.com/uploads/2019/10/10/sample1_1584.png)

> **Input:** `startTime = [1, 2, 3, 3]`, `endTime = [3, 4, 5, 6]`, `profit = [50, 10, 40, 70]`
>
> **Output:** `120`
>
> **Explanation:** The subset chosen is the first and fourth job.
> Time range $[1-3] + [3-6]$, we get profit of $120 = 50 + 70$.


## DP + binary search

This is a typical DP problem. Say we already decide the first few jobs to do, and are now wondering if we shall take the next job, or the next next job, or etc.? We first sort jobs by their ending time. Let $dp(t)$ be the maximum profit we can achieve by time $t = t$. Now consider job $i$ with $startTime_i$, $endTime_i$, and $profit_i$, where $endTime_i \geq t$. We can:

* ignore it. Then the profit is still the highest profit we can get before this job
* take it. Now the profit becomes the profit of job $i$ plus what we can get before this job. That is, find the maximum profit at time $t \leq startTime_i$ (otherwise two jobs overlap), and add $profit_i$ to this previous maximum profit.

If taking makes more money, we set $dp(endTime_i) = profit_i + dp(\sup \{t \leq startTime_i\})$.[^1]

[^1]: If ignoring is better, we don't actually need to change $dp$, as the supremum will find the previous maximum profit automatically.

**So What's the Answer in the End?** Observe that $dp(\cdot)$ is an increasing func. Bc. $\forall t_i > t_j$, if $dp(t_j) < dp(t_i)$, we can always make $dp(t_j) = dp(t_i)$ by ignoring all jobs starting after $t = t_i$. So the maximum profit must be the end period $dp$.

**Why or When Is DP Optimal?** Generally speaking, Bellman equation applies to problems which have optimal substructure. E.g., given an directed graph, the shortest path from $A$ to $B$ is $A \to C \to X \to B$. The shortest path from $C$ to $B$ is also $C \to X \to B$. By solving the optimal solutions to subproblems like $C$ to $B$, we can construct the solution to the original problem. A counterexample is flight tickets. The cheapest flight ticket from $A$ to $B$ is $A \to C \to B$. But the sum of tickets $A \to C$ and $C \to B$ may not be the cheapest ticket for the original problem.

**What Is the Optimal Substructure?** In this problem, we do a series of take and ignore actions. Say the optimality comprises of three actions at $t = t_0, t_1$, and $t_2$, and the solution to the problem is $dp(t_2$). Then before this action, at $t = t_1$, $dp(t_1)$ must also be the optimal for the subproblem $\displaystyle\max_{t \leq t_1} profit$. If not, let the maximum profit of the subproblem be $x >$ current $dp(t_1)$. Then in the original problem we can get $x + profit_{\text{action}_3} > dp(t_1) + profit_{\text{action}_3} = dp(t_2)$. However, by assumption $dp(t_2)$ is the solution to the original problem. Contradiction. $\blacksquare$


## Implementation

We set DP as usual, and use binary search to pin down $\sup \{t \leq startTime_i\}$.

```python
def binary_search(dp: List[List], start_time: int) -> int:
    l = 0
    r = len(dp) - 1

    # Corner case: if target is smaller than the smallest element in the input
    # array, or larger than the largest element
    if start_time <= dp[l][0]:
        return 0
    elif start_time >= dp[r][0]:
        return r

    # Binary search
    while l <= r:
        mid = (l + r) // 2

        # If mid point is larger than target
        if dp[mid][0] > start_time:
            r = mid - 1  # We can safely minus one, as the answer must be less
                         # or equal to target, and here is strictly larger
                         # than. Not necessary though

        # If mid point is smaller than target
        else:

            # If the next element is also smaller than target, then move left
            # pointer to mid plus one
            if dp[mid + 1][0] <= start_time:
                l = mid + 1

            # If the next element is larger than target, and the mid point
            # itself is smaller than target, it means the target is precisely
            # between mid point and the next element from mid point
            else:
                return mid
    return mid


def jobScheduling(startTime: List[int],
                  endTime: List[int],
                  profit: List[int]) -> int:
    jobs = sorted(zip(startTime, endTime, profit), key=lambda x: x[1])

    dp = [[0, 0]]
    for job in jobs:
        ignore = dp[-1][1]
        prev_job = self.binary_search(dp, job[0])
        take = job[2] + dp[prev_job][1]
        if take > ignore:
            if dp[-1][0] == job[1]:  # See notes below
                dp[-1][1] = take
            else:
                dp.append([job[1], take])
    return dp[-1][1]
```

**Binary Search.** The binary search is actually not that straightforward. We need to find the largest element that are less or equal than the target. This is not off-the-shelf `bisect.bisect_left` or `_right`. One solution is to make target slightly larger (e.g. `target + 0.5`) and use `bisect.bisect_left`. But such solution works only when we know the minimum gap between numbers, e.g. here we have integers so `+ 0.5` is fine. Our solution above implements the binary search from scratch:

1. check if the target is within the range of the input array, i.e. if smaller than the first element or larger than the last element. We check for this bc. otherwise we would get an index error in step 4 when checking for the next element to the mid
1. repeat step 3 and 4 the following until left pointer meets right pointer[^2]
1. get the mid of left and right pointer. If the mid is larger or smaller than the target, we move right pointer to mid minus one. We don't really need the "minus one"; just to make it slightly faster, as we are looking for the largest element less or equal to the target, and here mid is strictly larger than target, so mid won't be the answer
1. if mid is equal to or less than the target, we need to check two things:
    * if the next element to mid is also less or equal than target, we move left pointer to mid **plus one**. We have to check the next element bc. the input can have duplicate elements, e.g. `[1, 2, 2, 2, 6]`. When the target is $2$, we need to find the rightmost $2$. In other words, when we see mid is equal to target, maybe the next element is still equal to target, so we have to check. And here **plus one** is a must! E.g. `left = 3`, `right = 4`, and `mid = (3 + 4) // 2 = 3`. If we simply move `left` to `mid`, we are never able to exit the while loop
    * if the next element to mid is greater than target, and mid itself is less or equal to the target, then we know target is lying between $[mid, mid + 1]$, and thus mid is the answer

[^2]: This is, <code>left <<b><font color="red">=</font></b> right</code> instead of `left < right`, the same as in the usual binary search.

**Why We Update `dp` in Such a Way?** In principle, we can simply `dp.append([job[1], take])` when `take > ignore`. It's still correct, but wastes some space. E.g., `dp` can be `[[1, 10], [5, 13], [5, 50], [5, 57], [6, 70]]`. We only need `[5, 57]` for $endTime = 5$. So when we have `take = [5, 50]`, we check if the last element in `dp` has $endTime = 5$. If yes, we update it from `[[1, 10], [5, 13]]` to `[[1, 10], [5, 50]`. Similarly, when we see `[5, 57]`, we update from `[[1, 10], [5, 50]` to `[[1, 10], [5, 57]`. By doing this, we have `[[1, 10], [5, 57], [6, 70]]` in the end, saving space as well as time in binary search.
