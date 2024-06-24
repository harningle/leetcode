# 1438. Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit

Given an array of integers `nums` and an integer limit, return the size of the longest **non-empty** subarray such that the absolute difference between any two elements of this subarray is less than or equal to `limit`.


**Example:**

> **Input:** `nums = [10, 1, 2, 4, 7, 2, 80]`
> 
> **Output:** `4`
> 
> **Explanation:** The subarray $[2, 4, 7, 2]$ is the longest.


## Sliding window + deque

### Motivation

It's not difficult to come up with a sliding window idea, as we want to find a contiguous subarray satisfying some minimum and maximum criteria. The deque part is more difficult. Say the window is now $[10, 1]$, and it's not valid. We want to move the left pointer right. However, when we do the move, the minimum element changes from $10$ to $1$. Therefore, we want to somehow memorise which element is the minimum and maximum, so we know if the move kills the minimum. But this is not enough. After the move, the left pointer points to a new number, which may or may not be the new minimum. To know whether the after-move left pointer is the new minimum, we not only want to know *the* minimum, but also *all* small numbers in the window. A more concrete example is $[{\color{blue}2}, 4, 7, {\color{red}2}]$. The window is valid now, and we continue to move the right pointer right, resulting in $[{\color{blue}2}, 4, 7, {\color{red}2}, 80]$, which is no longer valid. So we move the left pointer right. After the move, ${\color{red}2}$ is still the minimum (even if the move kills the ${\color{blue}2}$).

I initially thought we could use monotonic stack: for each element, record the smaller element to its right in linear time using stack. Then after any move, if the next smaller element is within the window, and the left pointer does not point to the next smaller element, the minimum won't be updated. I think this is correct, but requires two passes: one to find the next smaller, and the second for sliding window. A better way is monotonic deque.[^linkedin post]

[^linkedin post]: See [https://www.linkedin.com/pulse/monotonic-stack-vs-deque-sarthak-gupta/](https://www.linkedin.com/pulse/monotonic-stack-vs-deque-sarthak-gupta/) also.

The idea is to maintain a monotonically weakly increasing deque to store the smallest elements. If we are at an invalid window, and want to move the left pointer, we can just go through the deque and see who is the new minimum.


<style>
    th {
        text-wrap: nowrap;
    }
</style>

| $l$ | $r$ | window         | min. deque | max. deque | notes                                                       |
|-----|-----|----------------|----------------|----------------|-------------------------------------------------------------|
| $0$ | $0$ | $[10]$         | $[10]$         | $[10]$         |                                                             |
| $0$ | $1$ | $[10, 1]$      | $[1]$          | $[10, 1]$      | $1$ smaller than last element in min. deque, so remove $10$ |
| $1$ | $1$ | $[1]$          | $[1]$          | $[1]$          | Move $l$ right. This removes $10$ from max. deque as it's no longer in the new window      |
| $1$ | $2$ | $[1, 2]$       | $[1, 2]$       | $[2]$          | $2$ larger than the last element in max. deque, so remove $1$  |
| $1$ | $3$ | $[1, 2, 4]$    | $[1, 2, 4]$    | $[4]$          | $4$ larger than the last element in max. deque, so remove $2$  |
| $1$ | $4$ | $[1, 2, 4, 7]$ | $[1, 2, 4, 7]$ | $[7]$          | $7$ larger than the last element in max. deque, so remove $4$  |
| $2$ | $4$ | $[2, 4, 7]$    | $[2, 4, 7]$    | $[7]$          | Move $l$ right. The move removes $1$ from min. deque as it's no longer in the new window        |
| $2$ | $5$ | $[2, 4, 7, 2]$ | $[2, 2]$       | $[7, 2]$       | $2$ is smaller than the last two elements, so remove both of them  |
| ... | ... | ... | ...       | ...       | the final $80$ kills everything  |


### Solution

```cpp
int longestSubarray(vector<int>& nums, int limit) {
    int ans = 1;

    deque<int> mx, mn;
    int l = 0;
    int n = nums.size();
    for (int r = 0; r < n; r++) {
        // Update the two deques
        while (!mx.empty() && nums[r] > mx.back()) mx.pop_back();
        while (!mn.empty() && nums[r] < mn.back()) mn.pop_back();
        mx.push_back(nums[r]);
        mn.push_back(nums[r]);

        // Move the left pointer right until the window is valid
        while (mx.front() - mn.front() > limit) {
            if (nums[l] == mx.front()) mx.pop_front();
            if (nums[l] == mn.front()) mn.pop_front();
            l++;
        }
        ans = max(ans, r - l + 1);
    }

    return ans;
}
```


### Notes

When I review this I still can't very clearly see why deque. The motivation above is not very well written. Here is more detailed and hopefully more clear explanation:

* after any move, we always can update the new min. and max. by checking the new element vs all elements in the window. This is $O(n)$ brute force linear search
* however, we don't have to brute force all elements, if we know *the* smallest element:
    + if the new element is smaller than the previous smallest element, we have a new min.
    + if the new element is larger than the previous smallest element, the min. doesn't change
* ok seems we have to store *the* smallest. But why deque?
* well what if we shrink the window and delete an element
    + e.g. we shrink from $[1, 2, 4, 7]$ to $[2, 4, 7]$
    + yes we already record *the* smallest element, which is $1$. So we know the shrink will delete the smallest element $1$ from the window. We will have a new minimum as the old is gone
    + ok then who is the new smallest?
    + oh no back to brute force linear search $[2, 4, 7]$
    + ok seems we not only need to record *the* smallest, but also *the second* smallest, in case the move of the left pointer removes the smallest element
    + then recursively, we also need *the third* smallest if there is another move that kills the second smallest later
    + so eventually we need a sort array of all smallest elements
* this is deque!
    + we maintain the smallest elements in increasing order in the current window
    + whenever we shrink the window, just check the front element:
        - if the shrink kills the front element in the deque, we know we lose the minimum. We pop it from the deque, and the new minimum (the current front after pop) is the second element in the previous deque before pop
        - if the shrink does not touch the front element in the deque, fine, nothing happens
    + is this enough? Shouldn't we go through the entire deque and delete the number that is being shrunk? no need
        - if it's the smallest, it's at the front
        - if it's not the smallest in the window, it won't be in the deque in the first place. E.g. $[4, 7, 1, 5]$ --> $[7, 1, 5]$. The deque before moving is $[1, 5]$, and $4$ is not even there!
    + the intuition is, we only care the smallest number and numbers *after* it. Numbers in the window that are larger but in front of the smallest don't matter, bc. even if we delete them, there is a smaller number waiting behind it
