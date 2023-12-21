# 1637. Widest Vertical Area Between Two Points Containing No Points

Given $n$ `points` on a 2D plane where <code>points[i] = [x<sub>i</sub>, y<sub>i</sub>]</code>, return *the <b>widest vertical area</b> between two points such that no points are inside the area*.

A **vertical area** is an area of fixed-width extending infinitely along the y-axis (i.e., infinite height). The **widest vertical area** is the one with the maximum width.

Note that points **on the edge** of a vertical area **are not** considered included in the area.


**Example:**

![](https://assets.leetcode.com/uploads/2020/09/19/points3.png)

> **Input:** `points = [[8, 7], [9, 9], [7, 4], [9, 7]]`
> 
> **Output:** `1`
> 
> **Explanation:** Both the red and the blue area are optimal.


## Sort and linear search

Since we only care about width, x-axis is all we need. We can sort the x coords., find all gaps between two neighbouring points, and return the largest one.

```python
def maxWidthOfVerticalArea(self, points: List[List[int]]) -> int:
    # Sort according to the x coord.
    points.sort(key=lambda x: x[0])

    # Linear search for the widest gap between two neighbouring points
    ans = 0
    for i in range(1, len(points)):
        gap = points[i][0] - points[i - 1][0]
        if gap > ans:
            ans = gap
    return ans
```


