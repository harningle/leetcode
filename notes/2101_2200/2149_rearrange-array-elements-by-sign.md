# 2149. Rearrange Array Elements by Sign

You are given a **0-indexed** integer array `nums` of **even** length consisting of an **equal** number of positive and negative integers.

You should **rearrange** the elements of `nums` such that the modified array follows the given conditions:

1. every **consecutive pair** of integers have **opposite signs**
1. for all integers with the same sign, the **order** in which they were present in `nums` is **preserved**
1. the rearranged array begins with a positive integer

Return *the modified array after rearranging the elements to satisfy the aforementioned conditions*.


**Example:**

> **Input:** `nums = [3, 1, -2, -5, 2, -4]`
>
> **Output:** `[3, -2, 1, -5, 2, -4]`
> 
> **Explanation:** The positive integers in nums are `[3, 1, 2]`. The negative integers are `[-2, -5, -4]`.

> The only possible way to rearrange them such that they satisfy all conditions is `[3, -2, 1, -5, 2, -4]`.
> 
> Other ways such as `[1, -2, 2, -5, 3, -4]`, `[3, 1, 2, -2, -5, -4]`, `[-2, 3, -5, 1, -4, 2]` are incorrect because they do not satisfy one or more conditions.  


## Two pointers

Bc. we have equal amount of positive and negative numbers, so the first positive integer is in index $0$ in `ans`, and the second positive number is in index $2$ in `ans`. So just create two pointers for the positive and negative numbers in `ans`.

```cpp
vector<int> rearrangeArray(vector<int>& nums) {
    int n = nums.size();
    vector<int> ans(n);
    int idx_pos = 0;
    int idx_neg = 1;
    for (int i : nums) {
        if (i > 0) {
            ans[idx_pos] = i;
            idx_pos += 2;
        } else {
            ans[idx_neg] = i;
            idx_neg += 2;
        }
    }
    return ans;
}
```

Why this is medium not easy...
