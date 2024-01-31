# 2870. Minimum Number of Operations to Make Array Empty

You are given a **0-indexed** array `nums` consisting of positive integers.

There are two types of operations that you can apply on the array **any** number of times:

* Choose **two** elements with **equal** values and **delete** them from the array.
* Choose **three** elements with **equal** values and **delete** them from the array.

Return <i>the <b>minimum</b> number of operations required to make the array empty, or</i> `-1` <i>if it is not possible</i>.


**Example:**

> **Input:** `nums = [2, 3, 3, 2, 2, 4, 2, 3, 4]`
>
> **Output:** `4`
>
> **Explanation:** We can apply the following operations to make the array empty:
>
> \- Apply the first operation on the elements at indices $0$ and $3$. The resulting array is `nums = [3, 3, 2, 4, 2, 3, 4]`.
> 
> \- Apply the first operation on the elements at indices $2$ and $4$. The resulting array is `nums = [3, 3, 4, 3, 4]`.
> 
> \- Apply the second operation on the elements at indices $0$, $1$, and $3$. The resulting array is `nums = [4, 4]`.
> 
> \- Apply the first operation on the elements at indices $0$ and $1$. The resulting array is `nums = []`.
> 
> It can be shown that we cannot make the array empty in less than 4 operations.


## Counting + hashmap

Count the #. of appearance of each number in `nums`. Call the #. of appearance $a \in \mathbb{Z}^+$, the #. of ops. 1 and 2 we need to delete all appearances is $m$ and $n$, $m, n\in \mathbb{N}^*$. If $a = 1$, then we are not able to delete it. Otherwise, $a$ can be $2, 3, 4, \cdots$. We can prove when $a \geq 2$, we can delete them. All those integers greater than one can be written as $a = 3k$, $3k + 1$, or $3k + 2$, $k\in \mathbb{N}^*$. We can write:

* $3k = 2\times \underset{m}{\underbrace{0}} + 3\times \underset{n}{\underbrace{k}}$,
* $3k + 1 = 2\times \underset{m}{\underbrace{2}} + 3\times \underset{n}{\underbrace{(k - 1)}}$, and
* $3k + 2 = 2\times \underset{m}{\underbrace{1}} + 3\times \underset{n}{\underbrace{k}}$. $\blacksquare$

To sum, count and get $a$’s, and check if $a$ is $3k$, $3k + 1$, or $3k + 2$ and get the $m$’s and $n$’s. The sum of $m$’s and $n$’s are the answer.

Or even simpler, observe that rounding $a/3$ is $m + n$.

```python
from collections import defaultdict


def minOperations(self, nums: List[int]) -> int:
    # Count the #. of appearance of numbers in the array
    hashmap = defaultdict(int)
    for i in nums:
        hashmap[i] += 1

    # Simple math above
    ans = 0
    for i in hashmap.values():
        if i == 1:
            return -1
        elif i % 3 == 0:
            ans += i // 3
        elif i % 3 == 1:
            ans += i // 3 - 1 + 2
        else:
            ans += i // 3 + 1
    return ans
```
