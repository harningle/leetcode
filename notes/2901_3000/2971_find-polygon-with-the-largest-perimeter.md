# 2971. Find Polygon With the Largest Perimeter

You are given an array of **positive** integers `nums` of length $n$.

A **polygon** is a closed plane figure that has at least $3$ sides. The **longest side** of a polygon is **smaller** than the sum of its other sides.

Conversely, if you have $k$ ($k\geq 3$) **positive** real numbers $a_1, a_2, a_3, \ldots, a_k$ where $a1\leq a2\leq a_3 \leq \ldots \leq a_k$ and $a_1 + a_2 + a_3 + \ldots + a_{k-1} > a_k$, then there **always** exists a polygon with k sides whose lengths are $a_1, a_2, a_3, \ldots, a_k$.

The **perimeter** of a polygon is the sum of lengths of its sides.

Return *the __largest__ possible __perimeter__ of a __polygon__ whose sides can be formed from* `nums`, *or* `-1` *if it is not possible to create a polygon*.

 
**Example:**

> **Input:** `nums = [1, 12, 1, 2, 5, 50, 3]`
> 
> **Output:** `12`
> 
> **Explanation:** The polygon with the largest perimeter which can be made from nums has $5$ sides: $1$, $1$, $2$, $3$, and $5$. The perimeter is $1 + 1 + 2 + 3 + 5 = 12$.
>
> We cannot have a polygon with either $12$ or $50$ as the longest side because it is not possible to include $2$ or more smaller sides that have a greater sum than either of them.
> 
> It can be shown that the largest possible perimeter is $12$.

**Constraints:**

* $3\leq n\leq 10^5$
* $1\leq$ `nums[i]` $\leq 10^9$


## Prefix sum

First, observe that given we find a polygon, adding shorter sides makes you strictly better off. That is, if $3, 4, 5$ makes a triangle, then adding shorter sides like $1, 2, 1$ to make it a hexagon is always viable. Bc. all we need is the longest side being smaller than the sum of the rest sides, so adding something to the rest will not violate the inequality, as long as the newly added side is shorter than the longest side. Then it becomes clear that if there is an answer, than the answer should take all numbers smaller than the longest side.

This naturally motivates sorting and prefix sum, so we can (1) know earlier sides are always smaller than later sides, and (2) compare a side with the rest of sides. From the third (or second if zero-indexed) number onwards, we check if it's longer than the sum of previous sides' length, which is the prefix sum. If yes, it's a valid polygon and a candidate answer. We check all numbers and return the maximum of those candidates.

```cpp
long long largestPerimeter(vector<int>& nums) {
    // If less than three edges, no polygon
    int n = nums.size();
    if (n <= 2) {
        return -1;
    }

    // Sort
    sort(nums.begin(), nums.end());

    // From the second element onwards, check if it's larger than the sum of
    // previous elements, and update `ans`
    long long ans = -1;
    long long prefix_sum = nums[0] + nums[1];
    for (int i = 2; i < n; i++) {
        if (prefix_sum > nums[i]) {
            ans = max(ans, prefix_sum + nums[i]);
        }
        prefix_sum += nums[i];
    }
    return ans;
}
```

*Notes.* It seems to be more efficient if we do it *reversely*. That is, take all sides, check if a polygon. If not, remove the longest side and check again. Repeat this until we find a polygon, which is the answer. Both directions are $O(n\log n)$, as sorting dominates the complexity. But the reverse one:

* needs a total sum and a reverse traversal, so goes through the arr. twice
* doesn't need to `max(ans, prefix_sum + nums[i])`  many times

It's not clear which is faster. Seems quite close.


## Priority queue

Exactly the same reverse traversal intuition as above, but now we use priority queue instead of sorting. This gives a complexity of $O(\underset{\text{Build a priority queue from a known collection}}{\underbrace{n}} + \underset{\text{Total #. of elements removed when we find the ans.}}{\underbrace{k}}\times \underset{\text{Removing an element from the queue}}{\underbrace{\log n}})$. $k$ is the number of sides we have removed before we find the answer. In the worst case $k = n$, but in most cases we are better than this.

The tricky part is why sorting is $O(n\log n)$ while building a priority queue is $O(n)$. See the proof here: [https://stackoverflow.com/a/18742428/12867291](https://stackoverflow.com/a/18742428/12867291).

```cpp
long long largestPerimeter(vector<int>& nums) {
    long long prefix_sum = 0;
    for (int i : nums) {
        prefix_sum += i;
    }

    priority_queue<int> q(nums.begin(), nums.end());
    while (!q.empty()){
        if (prefix_sum > 2 * q.top()) {  // `prefix_sum - q.top() < q.top()`
            return prefix_sum;
        }
        prefix_sum -= q.top();
        q.pop();
    }
    return -1;
}
```
