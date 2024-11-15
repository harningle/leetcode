# 2064. Minimized Maximum of Products Distributed to Any Store

You are given an integer `n` indicating there are `n` specialty retail stores. There are m product types of varying amounts, which are given as a **0-indexed** integer array `quantities`, where `quantities[i]` represents the number of products of the `i`<sup>th</sup> product type.

You need to distribute **all products** to the retail stores following these rules:

* a store can only be given **at most one product type** but can be given **any** amount of it
* after distribution, each store will have been given some number of products (possibly $0$). Let $x$ represent the maximum number of products given to any store. You want $x$ to be as small as possible, i.e., you want to minimize the maximum number of products that are given to any store

Return *the minimum possible $x$*.


**Example:**

> **Input:** `n = 6`, `quantities = [11, 6]`
>
> **Output:** `3`
> 
> **Explanation:** One optimal way is:
>
> - The $11$ products of type $0$ are distributed to the first four stores in these amounts: $2$, $3$, $3$, $3$
>
> - The $6$ products of type $1$ are distributed to the other two stores in these amounts: $3$, $3$
The maximum number of products given to any store is $\max(2, 3, 3, 3, 3, 3) = 3$.


## Binary search

I wasn't able to come up with this binary search. My initial idea is, the optimal allocation must be very close to "equally distributed". That is, we like $[3, 3, 4, 3]$ more than $1, 5, 1, 6$. And I thought we can somehow sort and assign the products to stores. But I couldn't make it work.

The binary search solution isn't very straightforward. We binary search the *answer*! Suppose the answer is $x$, then all stores should have (weakly) less than $x$ products. Let's say $x = 7$ and we have a product with quantity $18$. How many stores does this product need? At least $3$ (e.g. $[7, 7, 4]$, or $[6, 6, 6]$, etc.). So given $x$ and quantity $q$, we need $\left\lceil \dfrac{q}{x}\right\rceil$ stores. If the total #. of stores needed is larger than $n$, we know $x$ is too small. Otherwise, $x$ is too big and we are able to improve it. So we can run a binary search for $x$, with at least one store and at most $\max \{quantities_i\}$ stores.

```cpp
int minimizedMaximum(int n, vector<int>& quantities) {
    int l = 1;
    int r = INT_MIN;
    for (int q : quantities) r = max(r, q);

    while (l < r) {
        int m = (l + r) / 2;
        int s = 0;
        for (int q : quantities) s += (q + m - 1) / m;  // This is ceil(q/m)

        if (s <= n) {
            r = m;
        } else {
            l = m + 1;
        }
    }
    return l;
}
```

**Notes.** I always struggle with the conditions of binary search. Why it's `l = m + 1` here? Why return `l`, not `r` or `m`? Why `l < r` not `l <= r`?

First, the ending condition clearly should be `l` being strictly less than `r`. Otherwise, we exit the while loop only if `l` is to the right of `r`, which is not very sensible.

After deciding `l < r`, then we end when `l = r`, so it doesn't matter if we return `l` or `r` or `m`, bc. they are all equal.

Now the difficult part is why `r = m` and `l = m + 1`, not `l = m` or `r = m - 1`. It's helpful to think cases like `l = 4` and `r = 5`. In this example, `m = 4 = l`. When we want to update `r`, it's fine to change from `r = 5` to `r = m = 4`. However, if we want to update `l`, changing from `l = 4` to `l = m = 4` makes no difference and we will stuck inside the loop. So we want to make `l` slightly bigger than `m`.
