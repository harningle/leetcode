# 3224. Minimum Array Changes to Make Differences Equal

You are given an integer array `nums` of size `n` where `n` is even, and an integer `k`.

You can perform some changes on the array, where in one change you can replace **any** element in the array with **any** integer in the range from $0$ to `k`.

You need to perform some changes (possibly none) such that the final array satisfies the following condition:

* there exists an integer $X$ such that $\text{abs}(a_i - a_{n - i - 1}) = X$ for all $0 \leq i < n$

Return the **minimum** number of changes required to satisfy the above condition.


**Constraints:**

* <code>2 <= n == nums.length <= 10<sup>5</sup></code>
* `n` is even
* $0 \leq nums_i \leq k \leq 10^5$


**Example:**

> **Input:** `nums = [0, 1, 2, 3, 3, 6, 5, 4]`, `k = 6`
> 
> **Output:** `2`
> 
> **Explanation:**
> 
> We can perform the following operations:
> 
> * replace `nums[3]` by $0$. The resulting array is <code>nums = [0, 1, 2, <b><u>0</u></b>, 3, 6, 5, 4]</code>
> * replace `nums[4]` by $4$. The resulting array is <code>nums = [0, 1, 2, 0, <b><u>4</u></b>, 6, 5, 4]</code>
>
> The integer $X$ will be $4$.


## Motivation

Let's look at the example `nums = [0, 1, 2, 3, 3, 6, 5, 4]` first. The differences between number pairs are:

| $nums_i$ | $nums_{n - i - 1}$ | Diff. |
|----------|--------------------|-------|
| $0$      | $4$                | $4$   |
| $1$      | $5$                | $4$   |
| $2$      | $6$                | $4$   |
| $3$      | $3$                | $0$   |


### Pick the optimal $X$?

It seems the best to change the last $3$ and $3$ to something that gives diff. = $4$. We cannot do that in one change, bc. $k = 6$ and any number between $0$ and $6$ minus $3$ cannot reach $4$, so we have to do two changes. Therefore, my initial through was that we should change all diffs. to the most frequent diff. This is *wrong*! Consider the below example, which simply removes $0$, $4$ from `nums`:

| $nums_i$ | $nums_{n - i - 1}$ | Diff. |
|----------|--------------------|-------|
| $1$      | $5$                | $4$   |
| $2$      | $6$                | $4$   |
| $3$      | $3$                | $0$   |

Here it's equally good if we change $3$, $3$ to a pair with diff. $4$ (, which gives us $X = 4$) or if we change $1$ to $5$ and $2$ to $6$ (, which gives $X = 0$). Now it's less obvious which $X$ we should target. Another extreme example would be, if the number pairs have diff. = $1$, $2$, $3$, and $4$, which should be $X$, or maybe the optimal $X$ can be different from any existing diff.


### Brute force through all $X$’s?

So mathematically finding $X$ and then computing the #. of changes won't work. Then the most important clue, which I failed to notice during [Biweekly Contest 135](https://leetcode.com/contest/biweekly-contest-135/), is the constraint $0 \leq nums_i \leq k \leq 10^5$. This constraint implies that all elements in `nums` are bounded below $k$. Therefore, $X$ must also be bounded below $k$, i.e. the largest diff. we can have is $0$ and the largest element in `nums`, which is at most $k$. Now at least we have a brute force solution: try all possible $X$’s from $0$ to $k$, make all diff. to be $X$, count the #. of changes for each $X$, and then return the minimum one.

Brute force won't work, as $k$ and $n$ can be $10^5$ and the complexity is $O(nk)$, and $10^{10}$ will have TLE. To save time, we can either (1) skip some $X$’s rather than trying all of them, or (2) skip some counting #. of changes. We already know from above that we don't know how to pick $X$, so (1) doesn't work. (2) is the only choice: how to skip some counting? From the example and especially the change from $3$, $3$ to something with diff. of $4$, we have several important observations:

1. if we have a $6$ (, which is equal to $k$), one change can give us any diff.
1. the same holds for $0$
1. if we have $3$, $3$, then one change can only give diff. = $0$, $1$, $2$, or $3$

1\. and 2. provides a way to skip some counting: if we see a $6$, then for whichever $X$, we know it can do within one change (or maybe zero); we don't have to count this $6$ for every $X$. 3. generalise this: the diff. we get from a number pair has some bounds. The bound is trivial: (1) change the first number to $0$, (2) to $k$, (3) change the second number to $0$, or (4) to $k$. The max. possible diff. is the max of the four changes.[^continuous diff. range] Taking stock, for each number pair, we know its possible diffs. after a change, and we don't have to re-check if it's possible to reach $X$ for every $X$.

[^continuous diff. range]: I also didn't realise that the possible diffs. must be continuous. That is, after one change, it's impossible to have diffs. like $1$, $2$, (without $3$, $4$,) $5$, $6$. If we can make the diff. to be $6$, then any diff. from $0$ to $6$ is possible. The proof is easy. Say the number pair is $a, b$, and if we change $a$ to $c$, we have $c - b = 6$ (assuming $c$ is the larger one without loss of generality). To get $5$, we just need to replace $c$ with $c - 1$. To get $4$, we just need to replace $c - 1$ with $c - 2$, etc. So all numbers from $0$ to $6$ is doable.


### Prefix sum

Let's formalise the above idea now. For each number pair, we get their current diff., which obviously is needed, and their max. possible diff. after one change. Then for each $X$, since we know the max. possible diff. for all pairs, we know how many changes we need. Wait, it's still $O(nk)$!??? Wait wait let's go back to the example.

| $nums_i$ | $nums_{n - i - 1}$ | Diff. | Possible diffs.    |
|----------|--------------------|-------|--------------------|
| $0$      | $4$                | $4$   | $0$ to $6$         |
| $1$      | $5$                | $4$   | $0$ to $5$         |
| $2$      | $6$                | $4$   | $0$ to $6$         |
| $3$      | $3$                | $0$   | $0$ to $3$         |

Say we are trying $X = 5$. We know $0$ to $6$, $0$ to $5$, and $0$ to $6$ can be $5$ with one change, and $0$ to $3$ needs two changes. Ok is this "we know" $O(1)$? No if we are checking pairs one by one. Can we know/is there any data structure that tells us how many ranges contain $5$ in constant time? To put it in another way, given $a = \{6, 5, 6, 3\}$, how to count $\#(a\leq i)$? It's easy to count $\#(a = i)$ using a hashmap, and then $\#(a\leq i) = \sum\limits_{j\leq i}\#(a = j)$. This is prefix sum! To count using hashmap is linear time, and building prefix sum is also linear time, and after knowing the prefix sum, counting how many pairs can be made into $X = 5$ after one change is then constant!

Final small question: what if we need two changes. Easy. There are $\frac{n}{2}$ pairs. For a given $X$, we know how many pairs already have diff. $= X$, and we also know how many pairs can be made into $X$ within one change from the prefix sum. Then the result pairs need two changes.


## Prefix sum

```cpp
int minChanges(vector<int>& nums, int k) {
    // Diff.: #. of pairs with this diff.
    unordered_map<int, int> diff;

    // Max. possible diff. after one change: #. of such pairs
    vector<int> max_possible_diff(k + 1, 0);

    // Count diff. and max. possible diff.
    int n = nums.size();
    for (int i = 0; i < n / 2; i++) {
        diff[abs(nums[i] - nums[n - i - 1])]++;
        int max_diff = max({abs(k - nums[i]), abs(k - nums[n - i - 1]),
                            nums[i], nums[n - i - 1]});
        max_possible_diff[max_diff]++;
    }

    // Build prefix sum: max. possible diff. after one change: #. of pairs with
    // this max. possible diff. or a larger diff.
    for (int i = k - 1; i >= 0; i--) {
        max_possible_diff[i] += max_possible_diff[i + 1];
    }

    // Try all X's
    int ans = n / 2;  // Can set to INT_MAX. n/2 is changing all diff. to zero
    for (int d = 0; d <= k; d++) {
        ans = min(ans, max_possible_diff[d]  // Pairs needing one or no change
                       - diff[d]             // Minus pairs already have `d`
                       + (n / 2 - max_possible_diff[d]) * 2);  // Plus those
    }                                                          // need two
    return ans;
}
```

### We can actually skip some $X$’s

The above solution is already linear, but we can optimise a bit: the optimal must be within pre-existing diff. For a given $X$, the #. of changes we need is

1. pairs with max. possible diff. $= X$ after one change or no change
1. minus pairs already having this $X$
1. plus pairs need two changes to achieve $X$

Simplifying the above gives $-max\_possible\_diff_X - diff_X + n$, where $max\_possible\_diff$ is (weakly) monotonically decreasing in $X$. Suppose the pre-existing diff. are $2, 7, 10$. Assume $X = 3$ is optimal. Then $X = 2$ must also be optimal, bc. now we have a non-negative $diff_X$ and also a bigger (or at least the same) $max\_possible\_diff_X$ due to monotonicity. Therefore, optimality must be achieved (but of course there can be other equally good $X$ elsewhere) at pre-existing diffs. Thus, the `for (int d = 0; d <= k; d++)` loop can be further simplified to `for (auto& [d, cnt] : diff)`.

