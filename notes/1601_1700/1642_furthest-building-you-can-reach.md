# 1642. Furthest Building You Can Reach

You are given an integer array `heights` representing the heights of buildings, some `bricks`, and some `ladders`.

You start your journey from building $0$ and move to the next building by possibly using bricks or ladders.

While moving from building $i$ to building $i + 1$ (0-indexed),

* if the current building's height is **greater than or equal** to the next building's height, you do **not** need a ladder or bricks
* if the current building's height is **less than** the next building's height, you can either use **one ladder** or $(heights_{i + 1} - h_i)$ bricks

*Return the furthest building index (0-indexed) you can reach if you use the given ladders and bricks optimally*.


**Example:**

![](https://assets.leetcode.com/uploads/2020/10/27/q4.gif)

> **Input:** `heights = [4, 2, 7, 6, 9, 14, 12]`, `bricks = 5`, `ladders = 1`
> 
> **Output:** `4`
> 
> **Explanation:** Starting at building $0$, you can follow these steps:
>
> - go to building $1$ without using ladders nor bricks since $4\geq 2$
> - go to building $2$ using $5$ bricks. You must use either bricks or ladders because $2 < 7$
> - go to building $3$ without using ladders nor bricks since $7\geq 6$
> - go to building $4$ using your only ladder. You must use either bricks or ladders because $6 < 9$
>
> It is impossible to go beyond building $4$ because you do not have any more bricks or ladders.


## Priority queue + greedy

It seems we shall use ladders for larger height diff. and use bricks for smaller diff. I don't have a formal proof. Intuitively, the opportunity cost for using one ladder for $n$ height diff. is $n$ bricks, which is dependent on height diff., while using one brick is just one brick. So when the height diff. is small, using bricks are more economic, as ladders can have great potential when we get a larger height diff.

Now the problem is, use the ladders for the largest diff., and then use bricks for the rest, and when we use up everything, we reach the farthest building. But how can we know which diff. is the largest without knowing long far we can go? Brute force is a sol.: we try first two buildings, sort the diff., use ladders and bricks. If viable, we try first three buildings, sort, try, etc.

| $i$ | `heights`                 | `first_diff`           | Sorted `first_diff`                            |
|-----|---------------------------|------------------------|------------------------------------------------|
| $1$ | `[4, 2]`                  | `[-2]`                 | `[-2]`                                         |
| $2$ | `[4, 2, 7]`               | `[-2, 5]`              | <code>[<font color="red">5</font>, -2]</code>, <font color="red">red</font> means ladder |
| $3$ | `[4, 2, 7, 6]`            | `[-2, 5, -1]`          | <code>[<font color="red">5</font>, -1, -2]</code>                       |
| $4$ | `[4, 2, 7, 6, 9]`         | `[-2, 5, -1, 3]`       | <code>[<font color="red">5</font>, <font color="blue">3</font>, -1, -2]</code>, <font color="blue">blue</font> means bricks |
| $5$ | `[4, 2, 7, 6, 9, 14]`     | `[-2, 5, -1, 3, 5]`    | <code>[<font color="red">5</font>, <font color="blue">5</font>, 3, -1, -2]</code>, cannot jump over $3$ so not viable |

This brute force copies the first $i$ elements into a new arr. and sorts the arr. multiple times, and will get `Time Limit Exceeded`. There is some waste: if we have already sort the first three elements `[5, -1, -2]`, then when we get a new diff `3`, we don't need to sort the four numbers from scratch; we can start with the already ordered arr. from the previous step. This motivates the priority queue:

1. start from the $1$<sup>st</sup> building (zero-indexed)
1. calc. the height diff. between the current building and previous building, and push this diff. (if positive) into the priority queue
1. use ladders for the largest diff. in the queue. If we have some more diff., use bricks
    * if viable, move to the next building and go to step 2.
    * if not viable, it means we cannot jump to the current building. Return the previous building's index
    
```cpp
int furthestBuilding(vector<int>& heights, int bricks, int ladders) {
    int n_buildings = heights.size();
    priority_queue<int> pq;
    int first_diff;

    // Start from the first building
    for (int i = 1; i < n_buildings; i++) {
        first_diff = heights[i] - heights[i - 1];

        // Only need to consider positive diff., as jumping to a lower building
        // does not need any bricks or ladders
        if (first_diff > 0) {
            pq.push(-first_diff);  // Sort descendingly so take negative
        }

        // After using up all the ladders for the largest diff., do we have any
        // remaining diff.?
        if (pq.size() > ladders) {

            // If yes, use bricks for these smaller diff.
            bricks += pq.top();  // We took negative above, so plus instead of
            pq.pop();            // minus
        }

        // If have negative bricks after jumping to the current building, it
        // means we cannot make the jump, so return the previous building
        if (bricks < 0) {
            return i - 1;
        }
    }

    // If code reaches here, we jump through all buildings
    return n_buildings - 1;
}
```

Inserting new elements to a max heap is $\log n$ and we insert at most $n$ elements, so it's $O(n\log n)$.
