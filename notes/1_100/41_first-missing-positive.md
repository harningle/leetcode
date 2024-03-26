# 41. First Missing Positive

Given an unsorted integer array `nums`. Return the *smallest positive integer* that is *not* present in `nums`.

You must implement an algorithm that runs in $O(n)$ time and uses $O(1)$ auxiliary space.


**Example:**

> **Input:** `nums = [3,4,-1,1]`
> 
> **Output:** `2`


## Mapping number to index

Very similar to the yesterday's daily question [LC442](https://leetcode.com/problems/find-all-duplicates-in-an-array), we can usually solve this type of problems by creating a mapping between array values and indices. First, if there are $n$ numbers in `nums`, then the answer must be bounded in $[1, n + 1]$ by pigeonhole principle, as the first $n + 1$ positive numbers cannot fit in a size-$n$ array, so one of them will be the answer. Once observed this, we have $\{1, 2, \ldots, n + 1\}$ and need to figure out which does not exist in `nums`. Whenever a problem has array values and indices in the same range, it can often be solve by (not necessarily one-to-one) mapping values to indices.


### Swap

One way to think this is, we are rearranging `nums = [6, -1, 0, 5, 1, 2, 600]` into something like $\{1, 2, \ldots, n + 1\}$. That is, we want to put $1$ in the first place, $2$ in the second place, ... This is the **mapping**! This can be done by:

1. we see $6$, and put it into the sixth place in the array (one-indexed)
1. but here is already a number $2$ there, and we shouldn't discard $2$
1. then swap $2$ and $6$, so $6$ is in its place, and $2$ is not lost
1. now $2$ is there, and we want to put it into the second place, and swap again
1. now $-1$ is the zero-th element, it's not in $[1, n + 1]$, so we don't care, move on
1. $2$ is already in its place, move on
1. $0$ is not in the range, skip
1. until we reach the end of the array


```cpp
int firstMissingPositive(vector<int>& nums) {
    int n = nums.size();

    // Put numbers in their place
    for (int i = 0; i < n; i++) {
        while (nums[i] >= 1 && nums[i] <= n && nums[i] != nums[nums[i] - 1]) {
            swap(nums[nums[i] - 1], nums[i]);
        }
    }

    // First non-appearing number in [1, n + 1] is the answer
    for (int i = 0; i < n; i++) {
        if (nums[i] != i + 1) {
            return i + 1;
        }
    }

    return n + 1;
}
```

It seems we have a nested loop: a `while` inside a `for`, but it's linear. To see it, we have at most $n$ swaps to make, as we only care about $\{1, 2, \ldots, n + 1\}$ and skip everything else. After each swap, we have one (or maybe two if we are lucky) more number in its place. So at most $n$ swaps, and thus $O(n)$.


### Mask

Instead of swapping, we can also mask the index when we find the value. That is:

1. we see $6$, and put it into the sixth place in the array (one-indexed)
1. but here is already a number $2$ there, and we shouldn't discard $2$
1. we can mask $2$ to be $-2$, so $2$ is still there, and the negative sign together with index being $6$ means we already find $6$. Here is the **mapping**!
1. bc. now positive and negative signs carry meaning, we need to discard negative numbers beforehand, as they are useless anyway


```cpp
int firstMissingPositive(vector<int>& nums) {
    int n = nums.size();

    // Replace negative number with n + 1
    for (int i = 0; i < n; i++) {
        if (nums[i] <= 0) {
            nums[i] = n + 1;
        }
    }

    // Mask index if we find the number
    for (int i = 0; i < n; i++) {
        if (abs(nums[i]) <= n && nums[abs(nums[i]) - 1] > 0) {
            nums[abs(nums[i]) - 1] *= -1;
        }
    }

    // Return the first index with positive value
    for (int i = 0; i < n; i++) {
        if (nums[i] > 0) {
            return i + 1;
        }
    }

    return n + 1;
}
```
