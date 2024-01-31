# 2610. Convert an Array Into a 2D Array With Conditions

You are given an integer array `nums`. You need to create a 2D array from `nums` satisfying the following conditions:

* The 2D array should contain **only** the elements of the array `nums`.
* Each row in the 2D array contains **distinct** integers.
* The number of rows in the 2D array should be **minimal**.

Return *the resulting array*. If there are multiple answers, return any of them.

**Note** that the 2D array can have a different number of elements on each row.


**Example:**

> **Input:** `nums = [1,3,4,1,2,3,1]`
>
> **Output:** `[[1,3,4,2],[1,3],[1]]`
>
> **Explanation:** We can create a 2D array that contains the following rows:
>
> \- 1,3,4,2
>
> \- 1,3
> 
> \- 1
> 
> All elements of nums were used, and each row of the 2D array contains distinct integers, so it is a valid answer.
> It can be shown that we cannot have less than 3 rows in a valid array.


## Hashmap

We need to use all numbers from `nums` once and only once, put them into the rows of a new array, such that all numbers in each row in the new array are unique/distinct, and we want to minimise the #. of rows of the new array.

One way to do this is to put the first appearance of all numbers in row 0, and the second appearance of numbers in row 1, etc. The #. of rows in the new array is then the maximum appearance of numbers in the input array. It can be proved that we need at least that many of rows, bc. in a single row all numbers are unique, so if one number appears ten times, we need at least ten rows.

```text
input array = [1, 3, 4, 1, 2, 3, 1]
               |  |  |  |  |  |  |
appearance:    1  1  1  2  1  2  3
               └──┴──┴──│──┴──│──│──── [[1, 3, 4, 2],
                        └─────┴──│────  [1, 3],
                                 └────  [1]]

```

The implementation is easy: use a hashmap to count the #. of appearance of each number.

```python
from collections import defaultdict


def findMatrix(self, nums: List[int]) -> List[List[int]]:
    ans = []

    # Put the 1st appearence of a number in row 0, 2nd appearences in row 1...
    hashmap = defaultdict(int)
    for i in nums:
        hashmap[i] += 1
        n_appearances = hashmap[i]
        while len(ans) < n_appearances:
            ans.append([])
        ans[n_appearances - 1].append(i)  # Minus one bc. #. of appearance is
    return ans                            # 1-indexed, but list is 0-indexed
```


