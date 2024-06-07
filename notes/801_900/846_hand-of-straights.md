# 846. Hand of Straights

Alice has some number of cards and she wants to rearrange the cards into groups so that each group is of size `groupSize`, and consists of `groupSize` consecutive cards.

Given an integer array `hand` where `hand[i]` is the value written on the $i$<sup>th</sup> card and an integer `groupSize`, return `true` if she can rearrange the cards, or `false` otherwise.

 
**Example:**

> **Input:** `hand = [1, 2, 3, 6, 2, 3, 4, 7, 8], groupSize = 3`
> 
> **Output:** `true`
>
> **Explanation:** Alice's hand can be rearranged as $[1, 2, 3]$, $[2, 3, 4]$, $[6, 7, 8]$


## Ordered map

Simply count the #. of appearances of each unique number in the input array, and then, from small to big, check if we have the correct number to make a valid hand. The "check" needs to be ordered: say we have five $1$’s, then we must have at least five $2$’s. Otherwise some $1%’s cannot make a straight hand. We may have more than five $2$’s, which may be closed by $3$’s later. Such check should follow an increasing order, which is why we need a `map` instead of `unordered_map`.


```cpp
bool isNStraightHand(vector<int>& hand, int groupSize) {
    // Count. I call it `bucket` bc. I previously misread something and thought
    // it can be solved by bucket sort...
    map<int, int> bucket;
    for (int card : hand) bucket[card]++;

    // For each unique value in `hand`
    for (auto [i, v] : bucket) {

        // If we already use up its appearance, skip it
        if (v == 0) continue;

        // When we have `v` appearances of number `i`, need to check if we have
        // at least the same amount of number `i`, `i + 1`, all the way up to
        // number `i + groupSize - 1`
        for (int j = 0; j < groupSize; j++) {

            // If don't have number `i + j` at all, return `false`. We may be
            // able to combine this with the next check, but that could create
            // some new key if `i + j` does not exist before. I don't like it
            if (bucket.find(i + j) == bucket.end()) return false;

            // If we have that number, but the #. of appearances is not enough,
            // also return false
            if (bucket[i + j] < v) return false;

            // If we do have enough appearances of `i + j`, use it
            bucket[i + j] -= v;
        }
    }

    return true;
}
```

`map` is based on binary search tree (or some similar data structure), so it has $\log$ complexity for searching. We loop over all distinct values, so the overall time complexity is $O(m\log m)$, where $m$ is the #. of distinct values.


## Hashmap

There is actually a linear solution. The $\log$ above comes from the ordered map. Imagine we have $[0, 2, 3, 4]$ and `groupSize = 3`. If we start with $3$, only checking the following number $3$, $4$, and $5$ is wrong. We also need to check numbers ahead of $3$. `map` guarantees we always start with the smallest number, so checking numbers behind is enough. Alternative, we can start with number $3$, but now we have to check if we have $3 - 1$, if we have $3 - 2$, ... After we find the smallest consecutive number before $3$, we can start with that smallest number and proceed as usual. The search is constant in hashmap.


```cpp
bool isNStraightHand(vector<int>& hand, int groupSize) {
    // Count. Now use a hashmap
    unordered_map<int, int> hashmap;
    for (int card : hand) hashmap[card]++;
    
    while (!hashmap.empty()) {
        // Get whichever number from hashmap. Order not important here
        int card = hashmap.begin()->first;
        
        // Find the smallest consecutive number less than `card`, in O(1) time
        // E.g., if `card` = 6 and `hand` = [1, 3, 4, 5, 6, 7, 8], `smallest`
        // should be `3`
        int smallest = card;
        while (hashmap.find(smallest) != hashmap.end()) smallest--;
        smallest++;

        // Now start with `smallest`, and check if we have enough appearances
        // for number behind
        int curr = smallest;

        // Total #. of cards haven't been closed yet
        int n_opened = 0;

        // Starting number --> #. of unclosed hands starting with this number
        unordered_map<int, int> open_hands;

        // Loop over `smallest`, `smallest + 1`, `smallest + 2`, ...
        while (hashmap.find(curr) != hashmap.end()) {

            // #. of appearances of number `curr`
            int n = hashmap[curr];

            // Delete `curr` from the hashmap, as we are checking from small to
            // big, so each number will only be checked once, as the first sol.
            /**
             * For sure we don't have to delete it. We can set its appearances
             * to be zero, but then all the checks need to check if #. of
             * appearances > 0, which makes the code longer
             */
            hashmap.erase(curr);

            // Check if we have enough appearances
            /**
              * The intuition is, if we have some unclosed hands before and
              * `groupSize` is three, e.g. [1, 2], [1, 2], [1, 2], and [2], we
              * must have at least four 3's. Otherwise these hands will never
              * be able to close.
              */
            if (n < n_opened) return false;

            // Store the new hands that start with `curr`
            /**
              * Using the same example, if we have six 3's, and we have [1, 2],
              * [1, 2], [1, 2], and [2] four unclosed hands. Four 3's will be
              * appended to those hands, and two 3's will start their own new
              * hand.
              */
            open_hands[curr] = n - n_opened;

            // After appending 3's to previous hands, some hands will now be
            // closed
            /**
             * If `groupSize` is three, and `curr` is 3, we need to check how
             * many unclosed hands are starting with 1. After appending 3's to
             * them, [1, 2] will now become [1, 2, 3] and close. These closed
             * [1, 2, 3]'s are done and don't participate in later processing
             */
            n_opened += open_hands[curr] - open_hands[curr - groupSize + 1];
            curr++;
        }

        // After checking all consecutive numbers after `smallest`, if still
        // some unclosed hands, return false, as there is no more consecutive
        // numbers behind
        if (n_opened) return false;
    }
    
    return true;
}
```

The complexity is $O(m)$. For each distinct value, we first create the key in hashmap, then it is visited at `int card = hashmap.begin()->first;` and at `int n = hashmap[curr];`. So each unique value will be process three times, making the sol. linear.
