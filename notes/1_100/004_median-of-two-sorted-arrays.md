# 4. Median of Two Sorted Arrays

Given two sorted arrays `nums1` and `nums2` of size `m` and `n` respectively, return **the median** of the two sorted arrays.

The overall run time complexity should be $O(\log(m + n))$.


## Two pointers + binary search

The median is either a number in one array, or the avg. of two numbers, one from each array. For simplicity, we start with the below example with seven numbers in total:

```python
nums1 = [1, 3, 5, 10]
nums2 = [2, 40, 90]
```

Since we have seven numbers, the median is thus the fourth number in the sorted merged array. So all we need is the fourth smallest number. In this example, the four smallest numbers are `[1, 3, 5]` from `nums1` and `[2]` from `nums2`. And the answer is `5`.

**Observation.** We take the first few numbers as a subarray from both array to find the smallest four numbers, and the answer is the rightmost number in one of the two subarrays. The two subarray must satisfy that the rightmost, i.e. largest, of one subarray is smaller than the leftmost non-taken number from the other array. In this example, `5` in `[1, 3, 5]` must be smaller than `40` in `[40, 90]`, and `2` in `[2]` must be smaller than `10` in `[10]`. Call this "smallest condition".

**Target.** Find the subarrays that meet the "smallest condition" in $\log$ time.

**Implementation.** Put left and right pointer at the beginning and end of `nums1`, and then take the left half of numbers between two pointers. This gives us two numbers `[1, 3]`. Since we need the smallest four, we take two addition numbers from `nums2`, giving us `[2, 40]`. Check "smallest condition": `3` is smaller than `90`, good. `40` is greater than `5`, so the current four are not the smallest four. We should take more from `nums1` and less from `nums2`. So move the left pointer to the current middle plus one. The new left and (unchanged) right pointer is now at `5` and `10`. The new middle is at `5`, and now we take `[1, 3, 5]` from `nums1`. Again we need four smallest numbers, we need to take one more number from `nums2`, which is `[2]`. Now "smallest condition" is meet, so the answer is the largest one of the four rightmost numbers, two from each subarray. So `max(3, 5, 2) = 5`.


## Visualisation of another example

| Pointers                                                                                                          | Middle of binary search              | Subarrays                                                                           | Rightmost in subarrays | Leftmost in untaken nums. | "Smallest condition"                               |
|-------------------------------------------------------------------------------------------------------------------|--------------------------------------|-------------------------------------------------------------------------------------|--------------------------------|------------------------------------|----------------------------------------------------|
| <code>[<u>1</u>, 4, 5, 6, <u>90</u>]</code>                                                                       | <code>[1, 4, <u>5</u>, 6, 90]</code> | <code>[<u>1, 4, 5</u>, 6, 90]</code><br><code>[<u>1, 90, 91, 92</u>, 100, 101, 102] | $5$<br>$92$                    | $6$<br>$100$                       | $5 < 100$ &check;<br>$92 < 6$ &cross;              |
| <code>[1, 4, 5, <u>6</u>, <u>90</u>]</code>                                                                       | <code>[1, 4, 5, <u>6</u>, 90]</code> | <code>[<u>1, 4, 5, 6</u>, 90]</code><br><code>[<u>1, 90, 91</u>, 92, 100, 101, 102] | $6$<br>$91$                    | $90$<br>$92$                       | $6 < 92$ &check;<br>$91 < 90$ &cross;              |
| <code>[1, 4, 5, 6, <span style="text-decoration-line: underline; text-decoration-style: double">90</span>]</code> | <code>[1, 4, 5, 6, <u>90</u>]</code> | <code>[<u>1, 4, 5, 6, 90</u>]</code><br><code>[<u>1, 90</u>, 91, 92, 100, 101, 102] | $90$<br>$90$                   | No number left<br>$91$             | $90 < 91$ &check;<br>$90$ < no number left &check; |


## Solution

Make sure you check that all indices are valid index.

```c
double findMedianSortedArrays(int* nums1, int nums1Size, int* nums2, int nums2Size) {
    // Let the shorter array be `nums1`. This is important as we may deplete one array during the
    // binary search and subarray construction. Make sure it's always `nums1` being depleted
    if (nums1Size > nums2Size) {
        return findMedianSortedArrays(nums2, nums2Size, nums1, nums1Size);
    }

    // How many smallest numbers shall we take?
    // nums1Size = 4, nums2Size = 5 --> 5:
    //     nine numbers in total, median is the fifth
    // nums1Size = 2, nums2Size = 4 --> 4:
    //     six numbers in total, median is the avg. of the third and fourth
    int n_smallest = (nums1Size + nums2Size) / 2 + 1;  // Always rounded down and plus one so good
    bool is_even = (nums1Size + nums2Size + 1) % 2;

    // Two pointers
    int left = 0;
    int right = nums1Size - 1;

    // Corner case: the pointers shall be positive, i.e. need at least one number in `nums1`.
    // Otherwise the right pointer will point to -1
    if (nums1Size == 0) {
        if (is_even) {
            return (double) (nums2[n_smallest - 1] + nums2[n_smallest - 2]) / 2;
        } else {
            return (double) nums2[n_smallest - 1];
        }
    }

    // Binary search
    int r1_idx;
    int r2_idx;
    int r1;    // The rightmost number in the subarray
    int r2;
    int l1;    // The leftmost number in the untaken subarray
    int l2;
    int r1_2;  // The second rightmost number in the subarray. May need this if total #. is even
    int r2_2;
    while (left <= right) {
        // Middle of the binary search. Subarray 1 is nums1[0:r1_idx]
        r1_idx = (left + right) / 2;

        // Construct subarray 2: nums2[0:r2_idx]
        r2_idx = n_smallest - r1_idx - 2;  // `-2` bc. `n_smallest` is number count (1-indexed) and
                                           // `r1_idx` is 0-indexed
        // Corner case:
        //     `r1_idx` and `r2_idx` should be valid index
        if (r1_idx < nums1Size) {
            r1 = nums1[r1_idx];
        } else {
            // If the rightmost number index exceed the size of array, set it to be minus infinity
            // as we already deplete the entire array and all other smallest numbers shall come
            // from the other array, so we want the "smallest condition" to be always satisfied
            r1 = INT_MIN;
        }
        if (r2_idx < nums2Size) {
            r2 = nums2[r2_idx];
        } else {
            r2 = INT_MIN;
        }

        // Corner case:
        //     The leftmost untaken number should exist, so `r1` shouldn't be the final number
        if (r1_idx + 1 < nums1Size) {
            l1 = nums1[r1_idx + 1];
        } else {
            // If `r1` is already the final number of the array, again it means we already deplete
            // everything and all other smallest number shall come from the other array. So set the
            // leftmost untaken number to be positive infinity so the "smallest condition" is
            // always satisfied
            l1 = INT_MAX;
        }
        if (r2_idx + 1 < nums2Size) {
            l2 = nums2[r2_idx + 1];
        } else {
            l2 = INT_MAX;
        }

        // Corner case:
        //     The second rightmost number shall exist, bc. we may need it if the total #. of
        //     numbers is even
        if (r1_idx - 1 >= 0) {
            r1_2 = nums1[r1_idx - 1];
        } else {
            // If the subarray is a singleton, the median will not use the second rightmost number.
            // So set it to be negative infinity and thus it will never be selected during by
            // `fmax` below
            r1_2 = INT_MIN;
        }
        if (r2_idx - 1 >= 0) {
            r2_2 = nums2[r2_idx - 1];
        } else {
            r2_2 = INT_MIN;
        }

        // If "smallest condition" is meet, it means the two subarrays successfully taken the
        // smallest `n_smallest` numbers in the merged array
        // The answer is then decided by the largest (two) number(s) in the four rightmost numbers
        // from both subarrays, two from each
        if (r1 <= l2 && r2 <= l1) {
            if (is_even) {
                if (r1 >= r2) {
                    return (r1 + fmax(r2, r1_2)) / 2;
                } else {
                    return (r2 + fmax(r1, r2_2)) / 2;
                }
            } else {
                return fmax(r1, r2);
            }
        }

        // If the rightmost element in subarray 1 is greater than the leftmost untaken number in
        // `nums2`, then `right` should be decreased, i.e. take less from `nums1` as it's too large
        if (r1 > l2) {
            right = r1_idx - 1;
        }
        // Otherwise, increace `left`
        else {
            left = r1_idx + 1;
        }
    }

    // If the above loop doesn't return anything, it means we deplete `nums1` and both left and
    // right pointers are now at the end of array 1, but still cannot find an answer, which means
    // the median must be inside `nums2`
    if (is_even) {
        return (double) (nums2[n_smallest - 1] + nums2[n_smallest - 2]) / 2;
    } else {
        return (double) nums2[n_smallest - 1];
    }
    return 0;
}
```

