# 451. Sort Characters By Frequency

Given a string `s`, sort it in **decreasing order** based on the **frequency** of the characters. The **frequency** of a character is the number of times it appears in the string.

Return *the sorted string*. If there are multiple answers, return *any of them*.


**Example:**

> **Input:** `s = "tree"`
>
> **Output:** `eert`
>
> **Explanation:** `'e'` appears twice while `'r'` and `'t'` both appear once.
So `'e'` must appear before both `'r'` and `'t'`. Therefore `"eetr"` is also a valid answer.


**Constraints:**

* `s` consists of uppercase and lowercase English letters and digits


## Hashmap + (quick) sort

The straightforward solution is to count the #. of appearances of each char. in `s`, then sort the #. of appearances descendingly, and finally return the chars. of the "sorted" hashmap. To make it slightly faster, we can use an array rather than `unordered_map`, where the index is the ASCII of the char. and value is the #. of appearances. However, when we sort the array, we lose the original ASCII index. So the value has to have both the ASCII and the #. of appearances.


```cpp
string frequencySort(string s) {
    // Counting the chars.
    vector<vector<int>> hashmap(123);  // 'z' has ASCII 122
    for (int i = 0; i <= 122; i++) {
        hashmap[i] = {i, 0};
    }
    for (auto c: s) {
        hashmap[c][1]++;  // {'e': ['e', 2], 'r': ['r', 1], 't': ['t', 1]
    }

    // Sort by #. of appearances
    sort(hashmap.begin(), hashmap.end(),
         [](const auto& a, const auto& b) { return a[1] > b[1]; });

    // Appending chars. to answer by the descending order above
    string ans = "";
    for (int i = 0; i < 122; i++) {
        for (int j = 0; j < hashmap[i][1]; j++) {
            ans += hashmap[i][0];
        }
    }
    return ans;
}
```

The time complexity is not obvious. Let the length of `s` be $n$, and the most freq. char. has $k$ appearances. Counting using hashmap/array is $O(n)$. Sorting is $O(122\log 122)$ as we only have numbers and digits, and `'z'`’s ASCII is 122, so we are sorting a fixed length array. So it's $O(n + 122\log 122)$.[^122]

[^122]: The 122 can be slightly improved, as all digits plus upper and lower letters is $10 + 26 + 26 < 122$.


## Hashmap + bucket sort

Interestingly there is a way to avoid sorting.[^source] In general we can't, but here we do have a constraint: `s` has a finite length. So if `s` has $k$ chars., then the most frequent char. will at most have $k$ appearances. So we loop from $k$ to $1$, go into the hashmap, and get all keys with value $k$, with value $k - 1$, ..., with value $1$. But now we have to loop over the hashmap $k$ times. Another way of doing this is bucket sort (but only bucket no sort). We go into the hashmap, if the value is $a$, we append the key into bucket $a$ (as if we are "reversing the key and value"). And finally we loop over the buckets and get all the chars.

[^source]: [https://leetcode.com/problems/sort-characters-by-frequency/solutions/1503201/c-python-3-solutions-sorting-bucket-sort-o-n-clean-concise](https://leetcode.com/problems/sort-characters-by-frequency/solutions/1503201/c-python-3-solutions-sorting-bucket-sort-o-n-clean-concise).


```cpp
string frequencySort(string s) {
    // Hashmap counting
    vector<int> hashmap(123, 0);
    for (auto c: s) {
        hashmap[c]++;
    }

    // Bucket sort, but only bucket no sort
    vector<vector<int>> bucket(s.size() + 1);
    for (int i = 0; i <= 122; i++) {
        if (hashmap[i]) {
            bucket[hashmap[i]].push_back(i);
        }
    }
    // {'e': 2, 'r': 1, 't': 1} --> bucket[2] = ['e'], bucket[1] = ['r', 't']

    // Appending the chars. in all buckets
    string ans = "";
    for (int i = s.size(); i >= 0; i--) {
        for (auto j: bucket[i]) {
            ans.append(i, j);
        }
    }
    return ans;
}
```

Is this really faster? We can the same $O(n)$ counting, and bucket sorting is $O(122)$, so in the end it's $O(n + 122)$, slightly better than $O(n + 122\log 122)$.
