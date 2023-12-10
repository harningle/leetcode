# 1. Two Sum

Given an array of integers nums and an integer target, return indices of the two numbers such that they add up to target.

You may assume that each input would have **exactly one solution**, and you may not use the same element twice.

You can return the answer in any order.


Example 1:

> Input: `nums = [2,7,11,15], target = 9`
> 
> Output: `[0,1]`
> 
> Explanation: Because `nums[0] + nums[1] == 9`, we return `[0, 1]`.


## Brute force

For each number $i$ in `nums`, go through `nums`, and check if any of the other numbers and $i$ add up to target. If there are $N$ numbers in `nums`, then in the worst case we need to do $(N - 1) + (N - 2) + \cdots + 1$, or $O(n^2)$ comparisons.


## Hash map

What we really want to know is, for a given number $i \in$ `nums`, does target $- i$ exist in `nums`? So instead of checking whether $i + j =$ target, $\forall i, j \in$ `nums`, we just need to check if target $- i \in$ `nums`, $\forall i$. This reduces nested loops above to one single loop plus a few existence check. Checking existence is super fast, almost constant time, for hash table.

To make it even faster, it's sufficient to check whether target $- i \in$ the numbers we already checked before $i$, rather than the entire `nums`. This follows the same intuition as the brute force above: it's not $N + N + \cdots + N$, but $(N - 1) + (N - 2) + \cdots + 1$: if $a$ and $b$ is compared, we don't need to compare $b$ and $a$ again.

```python
hashmap = dict()  # {i: i's index in `nums`}
for idx, i in enumerate(nums):
    j = target - i
    if j in hashmap:
        return [idx, hashmap[j]]
    hashmap[i] = idx
```

Pure C implementation is more complicated than I expected... There is no easy-to-use builtin hash table in C, and implementing itself (even with the help of some third party lib.) is very tedious... See [https://leetcode.com/problems/two-sum/solutions/1024482/solution-with-c/comments/1668705](https://leetcode.com/problems/two-sum/solutions/1024482/solution-with-c/comments/1668705).
