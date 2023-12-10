# 1218. Longest Arithmetic Subsequence of Given Difference

Given an integer array arr and an integer difference, return the length of the longest subsequence in arr which is an arithmetic sequence such that the difference between adjacent elements in the subsequence equals difference.

A subsequence is a sequence that can be derived from arr by deleting some or no elements without changing the order of the remaining elements.

Example:

> **Input**: arr = [1,5,7,8,5,3,4,2,1], difference = -2
> 
> **Output**: 4
> 
> **Explanation**: The longest arithmetic subsequence is [7,5,3,1].


## Brute force

Start from the first number $i$, and check if $i + diff$ is in the sub-array to the left of $i$, and then check $i + 2diff$, until not found. Then start again from the second number...

| Index | Number | Subseq. found    | Len. of subseq. |
|-------|--------|------------------|-----------------|
| 0     | 1      | [1]              | 1               |
| 1     | 5      | [5, 3, 1]        | 3               |
| **2** | **7**  | **[7, 5, 3, 1]** | **4**           |
| 3     | 8      | [8]              | 1               |
| 4     | 5      | [5, 3, 1]        | 3               |
| 5     | 3      | [3, 1]           | 2               |
| 6     | 4      | [4, 2]           | 2               |
| 7     | 2      | [2]              | 1               |
| 8     | 1      | [1]              | 1               |


## DP.

Observe that for any given subseq. found, it is the concat. of the current number and the subseq. the current number, if exists to the right of the current number, minus diff. E.g., when we go to index 1 number 5, the subseq. [5, 3, 1] is (the current number 5 plus the subseq. of index 5 number 3 [3, 1]). Another example: the subseq. for index 6 number 4 [4, 2] is (the current number 4 plus the subseq. of number index 7 number 2 [2]).

This implies the following induction:

$$
A_n = \left\{ \begin{array}{ll}
A_{n + diff} + 1 & \text{if} \ n + diff \text{ is to the right of } n  \\
1 & \text{if} \ n + diff \text{ is not found to the right of } n
\end{array} \right.
$$

, where $A_n$ is the length of the subseq. starting with number $n$. The base case is $A_{-1} = 1$, i.e. the last number always has a subseq. of length one.

So we start from the right, and get all $A_i$. The largest of $A_i$ is the ans.

| Index | Number | $n + diff$ | $A_{n + diff}$                | $A_n$ | $\{A_i\}$                                                         |
|-------|--------|------------|-------------------------------|-------|-----------------------------------------------------------------|
| 8     | 1      | base case  | base case                     | 1     | $A_1 = 1$                                                       |
| 7     | 2      | 0          | No $A_0$ found before      | 1     | $A_1 = 1, A_2 = 1$                                              |
| 6     | 3      | 1          | $A_1 = 1$ found at index 8 | 2     | $A_1 = 1, A_2 = 1, A_3 = 2$                                     |
| 5     | 4      | 2          | $A_2 = 1$ found at index 7 | 2     | $A_1 = 1, A_2 = 1, A_3 = 2, A_4 = 2$                            |
| 4     | 5      | 3          | $A_3 = 2$ found at index 6 | 3     | $A_1 = 1, A_2 = 1, A_3 = 2, A_4 = 2, A_5 = 3$                   |
| 3     | 8      | 6          | No $A_6$ found before      | 1     | $A_1 = 1, A_2 = 1, A_3 = 2, A_4 = 2, A_5 = 3, A_8 = 1$          |
| 2     | 7      | 5          | $A_5 = 3$ found at index 4 | 4     | $A_1 = 1, A_2 = 1, A_3 = 2, A_4 = 2, A_5 = 3, A_8 = 1, A_7 = 4$ |
| 1     | 5      | 3          | $A_3 = 2$ found at index 6 | 3     | $A_1 = 1, A_2 = 1, A_3 = 2, A_4 = 2, A_5 = 3, A_8 = 1, A_7 = 4$ |
| 0     | 1      | -1         | No $A_{-1}$ found before   | 1     | $A_1 = 1, A_2 = 1, A_3 = 2, A_4 = 2, A_5 = 3, A_8 = 1, A_7 = 4$ |

```python
dp = dict()
for num in arr[::-1]:
    if num + diff in dp:
        dp[num] = dp[num + diff] + 1
    else:
        dp[num] = 1
return max(dp.values())
```

Of course we can do the loop from left to right. But I think the current sol. is more natural.
