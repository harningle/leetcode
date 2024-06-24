# 995. Minimum Number of K Consecutive Bit Flips

You are given a binary array `nums` and an integer `k`.

A **k-bit flip** is choosing a **subarray** of length `k` from `nums` and simultaneously changing every `0` in the subarray to `1`, and every `1` in the subarray to `0`.

Return *the minimum number of __k-bit flips__ required so that there is no* `0` *in the array*. If it is not possible, return `-1`.

A **subarray** is a **contiguous** part of an array.

 
**Example:**

> **Input:** `nums = [0, 0, 0, 1, 0, 1, 1, 0], k = 3`
> 
> **Output:** `3`
> 
> **Explanation:**
>
> Flip `nums[0]`, `nums[1]`, `nums[2]`: `nums` becomes `[1, 1, 1, 1, 0, 1, 1, 0]`
>
> Flip `nums[4]`, `nums[5]`, `nums[6]`: `nums` becomes `[1, 1, 1, 1, 1, 0, 0, 0]`
>
> Flip `nums[5]`, `nums[6]`, `nums[7]`: `nums` becomes `[1, 1, 1, 1, 1, 1, 1, 1]`


## Brute force

### Proof

This is effectively the same as problem 2 in [Biweekly Contest 133](https://leetcode.com/contest/biweekly-contest-133/). It's not difficult to come up with a left-to-right brute force approach: start from the left and move towards the right, and whenever we see a `0`, flip it. I tried a few examples during the biweekly contest and it seemed to be correct. Yet I couldn't figure out a formal proof to prove this left-to-right approach gives the min. #. of flips.

**Order Doesn't Matter.** It's easy to show that if there exists a way to flip all `0`’s to `1`’s, then changing the order of flips in this way also gives the answer. Say the answer first flips indices $[1, 2, 3]$, then $[3, 4, 5]$. Then indices $1, 2, 4, 5$ are flipped only once, and index $3$ twice. If we change the order, flip $[3, 4, 5]$ first then $[1, 2, 3]$, the total #. of flips of each number is the same. As long as the #. of flips is the same, the result of flipping is also the same, so we will get the same array in the end.

**Starting Point of Each Flip Is Unique.** A number can be (and sometimes has to be) flipped multiple times, but the starting point of each flip won't. E.g., flipping $\color{red}[2, 3, 4]$, then $[1, 2, 3]$, then $[4, 5, 6]$, and finally $\color{red}[2, 3, 4]$ again is never optimal. In the current flipping, number $1, 2, 3, 4, 5, 6$ gets $1, 3, 3, 3, 1, 1$ flips, respectively. Removing the duplicated $\color{red}[2, 3, 4]$’s gives $1, 1, 1, 1, 1, 1$ flips, and gives exactly the same resulting array, as flipping three times is the same as flipping once only. So no matter how the final flipping looks like, the starting point of each flip must be unique; the optimal way won't start a flip on the same index more than once.

**Induction.** All `0`’s need at least one flip to become `1` (or maybe three, or five, ...). Since the order of flips doesn't matter and the starting point of each flip is unique, the starting point of the answer flipping must be strictly (by uniqueness) increasing (by order invariance). Consider the leftmost `0` at index $i$. To get this `0` being flipped, we can start a flip right at $i$, or somewhere to the left of it $j$ (within the range of `k`). If we choose somewhere left $j$, we must have to flipped them back again later, as by definition all numbers to the left of this leftmost `0` are `1` and we need two or four or ... flips to keep these `1`’s as `1`. Because the starting point of flips are strictly increasing, we won't have the chance to flip those `1`’s back later. Otherwise the starting point of flips will either be $j$ again (violates uniqueness) or even lefter to $j$ (violates monotonicity). So the optimal way to flip the leftmost $0$ is to start a flip right at it. After flipping it, we can with the new leftmost $0$, and repeat the same process. By induction the optimal way to flip is left-to-right flipping.


### Solution

```cpp
int minKBitFlips(vector<int>& nums, int k) {
    int n = nums.size();
    int ans = 0;

    // Left-to-right flip
    for (int i = 0; i <= n - k; i++) {
        if (nums[i] == 0) {
            ans++;
            for (int j = 0; j < k; j++) {
                nums[i + j] = 1 - nums[i + j];
            }
        }
    }

    // As each flip has to flip `k` numbers, so after all the flipping above,
    // if the last `k` numbers are not all 1's, there is no solution
    for (int i = n - k + 1; i < n; i++) {
        if (nums[i] == 0) return -1;
    }

    return ans;
}
```

This solution *looks like* linear, but is actually $O(nk)$, bc. within the one pass, each time we see a `0`, we flip `k` numbers. When $k$ is big, we get `Time Limit Exceeded`.


## True one pass

We can save the #. of flips in the "`k`-window" to avoid the flipping above. The intuition is, if `k` is 3, and if $nums_5$ is flipped, then we know $nums_6$ and $nums_7$ are also flipped. We don't need to actually flip $nums_6$ and $nums_7$, but simply memorise the #. of previous flips affect the two numbers.

| $i$ | $nums$ before flip         | flip        | $nums$ after flip          | #. of previous flips affecting $i$ | notes                                                          |
|-----|----------------------------|-------------|----------------------------|------------------------------------|----------------------------------------------------------------|
| $0$ | $[1, 1, 0, 1, 1, 1, 0, 0]$ |             | $[1, 1, 0, 1, 1, 1, 0, 0]$ | $0$                                |                                                                |
| $1$ | $[1, 1, 0, 1, 1, 1, 0, 0]$ |             | $[1, 1, 0, 1, 1, 1, 0, 0]$ | $0$                                |                                                                |
| $2$ | $[1, 1, 0, 1, 1, 1, 0, 0]$ | $[2, 3, 4]$ | $[1, 1, 1, 0, 0, 1, 0, 0]$ | $0$                                |                                                                |
| $3$ | $[1, 1, 1, 0, 0, 1, 0, 0]$ | $[3, 4, 5]$ | $[1, 1, 1, 1, 1, 0, 0, 0]$ | $1 = 0 + 1$                        | as we flipped $2$                                              |
| $4$ | $[1, 1, 1, 1, 1, 0, 0, 0]$ |             | $[1, 1, 1, 1, 1, 0, 0, 0]$ | $2 = 1 + 1$                        | as we further flipped $3$                                      |
| $5$ | $[1, 1, 1, 1, 1, 0, 0, 0]$ | $[5, 6, 7]$ | $[1, 1, 1, 1, 1, 1, 1, 1]$ | $1 = 2 - 1$                        | as we move out of $[2, 3, 4]$ so the flip on $2$ is subtracted |
| $6$ | $[1, 1, 1, 1, 1, 1, 1, 1]$ |             | $[1, 1, 1, 1, 1, 1, 1, 1]$ | $1 = 1 + 1 - 1$                    | $+1$ as we flipped $5$, and $-1$ as we are out of $[3, 4, 5]$  |
| $7$ | $[1, 1, 1, 1, 1, 1, 1, 1]$ |             | $[1, 1, 1, 1, 1, 1, 1, 1]$ | $1$                                |                                                                |


```cpp
int minKBitFlips(vector<int>& nums, int k) {
    int n = nums.size();
    int ans = 0;
    int n_flips = 0;  // Total #. of flips affecting the starting point of the
                      // current flip
    for (int i = 0; i < n ; i++) {

        // Check if index i - k is flipped. If yes, minus one to `n_flips`, as
        // we are now out of the window of [i, ..., i - k - 1] so the flipping
        // of i - k shouldn't be counted
        if (i >= k && nums[i - k] == -1) {
            n_flips--;
        }

        // Check if the current number needs flip
        if ((n_flips % 2 == 0 && nums[i] == 0)
                || (n_flips % 2 == 1 && nums[i] == 1)) {
            if (i + k > n) return -1;
            nums[i] = -1;  // Use -1 to mark if an index is flipped
            n_flips++;
            ans++;
        }
    }

    return ans;
}
```


