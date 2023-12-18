# 1913. Maximum Product Difference Between Two Pairs

The **product difference** between two pairs $(a, b)$ and $(c, d)$ is defined as $(a \times b) - (c \times d)$.

* For example, the product difference between $(5, 6)$ and $(2, 7)$ is $(5 \times 6) - (2 \times 7) = 16$.

Given an integer array `nums`, choose four distinct indices `w`, `x`, `y`, and `z` such that the product difference between pairs `(nums[w], nums[x])` and `(nums[y], nums[z])` is maximized.

Return the **maximum** such product difference.

**Constraints:**

* `4 <= nums.length <= 104`
* `1 <= nums[i] <= 104`
 

**Example:**

> **Input:** `nums = [5, 6, 2, 7, 4]`
> 
> **Output:** `34`
> 
> **Explanation:** We can choose indices `1` and `3` for the first pair `(6, 7)` and indices `2` and `4` for the second pair `(2, 4)`. The product difference is `(6 * 7) - (2 * 4) = 34`.


## Linear search

Since all numbers are positive, we will have the largest product diff. $(a \times b) - (c \times d)$ when $a$ and $b$ are the two largest numbers, and $c$ and $d$ are the smallest. So just go through the array, and find the four numbers.

```c
int maxProductDifference(int* nums, int numsSize){
    int l1 = 0;
    int l2 = 0;
    int s1 = INT_MAX;
    int s2 = INT_MAX;

    // Linear search for the smallest and largest two nums.
    int num;
    for (int i = 0; i < numsSize; i++) {
        num = nums[i];
        if (num >= l1) {
            l2 = l1;
            l1 = num;
        } else if (num > l2) {
            l2 = num;
        }
        if (num <= s1) {
            s2 = s1;
            s1 = num;
        } else if (num < s2) {
            s2 = num;
        }
    }

    return (l1 * l2) - (s1 * s2);
}
```


