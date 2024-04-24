# 1481. Least Number of Unique Integers after K Removals

Given an array of integers `arr` and an integer `k`. Find *the least number of unique integers* after removing **exactly** `k` elements.


**Example:**

> **Input:** `arr = [4, 3, 1, 1, 3, 3, 2], k = 3`
> 
> **Output:** `2`
> 
> **Explanation:** Remove $4$, $2$ and either one of the two $1$’s or three $3$’s. $1$ and $3$ will be left.


## Hashmap + greedy + bucket sort

It's easy to prove that we should move less frequent numbers first. E.g. we have three removals available in the above example, if we move three $3$’s, then we only deleted one unique number. But if we remove one $2$ and one $4$, with only two removals, we already deleted two unique numbers. Then the sol. is clear: counting the appearances of all numbers, and sort the appearances, and remove numbers from the least frequent number onwards.

Counting using hashmap is $O(n)$, sorting the values of the hashmap is $O(n\log n)$, and deleting is at most $O(n)$. But we can do better by avoiding sorting using bucket sort, the same intuition as [LeetCode 451](../401_500/451_sort-characters-by-frequency.md#hashmap-bucket-sort). That is, since the input `arr` has a limited length, the most frequent number will at most appear `arr.size()` times. So we can create `arr.size()` $+ 1$ buckets, and bucket $i$ will save the #. of unique numbers with an appearance of $i$. E.g., in the example, `bucket[0] = 0` as no number has zero appearance. `bucket[1] = 2` as there are two numbers ($4$ and $2$) appearing only once. `bucket[2] = 1` as only one number $1$ appears twice, etc. Then we remove numbers from those buckets sequentially. When remove a number from `bucket[i]`, we need $i$ removals, as the number appears $i$ times. Keep doing this until we use up all available removals, or when we reach the end of buckets.

As this avoids sorting at all, the time complexity is $O(n)$.

```cpp
int findLeastNumOfUniqueInts(vector<int>& arr, int k) {
    // Count
    unordered_map<int, int> hashmap;
    for (int i : arr) {
        hashmap[i]++;
    }

    // Bucket (without sort)
    vector<int> bucket(arr.size() + 1, 0);
    int ans = 0;
    for (auto it : hashmap) {
        bucket[it.second]++;
        ans++;
    }

    // Remove numbers from smaller buckets than larger buckets
    int n_buckets = bucket.size();
    int possible_removals;
    for (int i = 1; i < n_buckets; i++) {
        if (bucket[i] == 0) {
            continue;
        }

        possible_removals = min(k / i, bucket[i]);  // Need i removals for
        ans -= possible_removals;                   // bucket[i]
        k -= possible_removals * i;

        if (k == 0) {
            return ans;
        }
    }
    return ans;
}
```


