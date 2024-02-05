# 76. Minimum Window Substring

Given two strings `s` and `t` of lengths `m` and `n` respectively, return *the __minimum window substring__[^substring] of* `s` *such that every character in* `t` *(__including duplicates__) is included in the window*. If there is no such substring, return *the empty string* `""`.

[^substring]: A **substring** is a contiguous **non-empty** sequence of characters within a string.

The testcases will be generated such that the answer is **unique**.


**Example:**

> **Input:** `s = "ADOBECODEBANC"`, `t = "ABC"`
> 
> **Output:** `BANC`
> 
> **Explanation:** The minimum window substring `"BANC"` includes `'A'`, `'B'`, and `'C'` from string `t`.


**Constraints:**

* `s` and `t` consist of uppercase and lowercase English letters.


## Two pointers/sliding window

We need a substring in `s`, which is determined by a starting char. and an ending char. This naturally motivates two pointers. The rough idea is:

1. put left and right pointers at the beginning of `s`
1. move right pointer right by one char., see if it is the char. we need, i.e. a char. in `t`
1. repeat 2, until we find all chars. need
1. now focus on the left pointer. Move it right by one char., and see if we can narrow the gap between the two pointers
1. during the move in 4, we may lose some chars. needed. If it's the case, hold the left pointer, and go to 2 and 3 again
1. exit when both pointer are at the end of `s`

**What Is "see if it is the char. we need" in Step 2.?** `t` may be `"aabc"`. When right pointer is "`a`", how can we know if we have found only one `'a'` or both `'a'`’s? Hashmap! We count the #. of appearance of each char. in `t`, and when right pointer finds a char. in the hashmap, we decrease its value. When a char. in the hashmap has a value of zero (or negative) we find all of it.

**What if "we may lose some chars. needed"? in Step 4.** We will lose a char. when we move the left pointer. If it's a char. in `t`, we may or may not be in trouble.

* sometimes we have redundant chars. between two pointers, e.g. `s = "aaaaaaabc"` and `t = "aabc"`. It's fine if we move the left pointer from index $0$ to index $1$, bc. we still have enough `'a'`’s between the two pointers. As said above, whenever right pointer finds a char. in `t`, it decrease the value, so we do have negative values, which accounts for redundancy
* sometimes the moving of left pointer does kill a char. we need and we have to find another. In this case, move the right pointer in step 2. and 3.

**A Better Way to Check if Done.** When all values in the hashmap is zero or negative (bc. we may have redundancy), we get a candidate answer. But checking all values of hashmap is $O(m)$. A faster way to do this is create a counter. Say `t = "aabc"`, then counter starts at four. When we find a char. in `t`, we decrease it. However, when we find a third `'a'`, which is a redundant one (and now it `'a'` has a negative value in the hashmap), we don't decrease it any more. Now when the counter is zero, we should find every char. we need.

**Array instead of Hashmap.** In step 2., we need to check if a key exists in the hashmap. `unordered_map::find` is $O(1)$ generally, but it's slower than array lookup. We can simply replace the hashmap with an array. The key lookup can be replaced with checking if the element in array has a positive value.


### Visualisation

| `l` | `r` | Array/hashmap before checking | Array/hashmap after checking                                    | Counter                           | Candidate ans. | Notes                                                                   |
|-----|-----|-------------------------------|-----------------------------------------------------------------|-----------------------------------|----------------|-------------------------------------------------------------------------|
|     |     | `{'A': 1, 'B': 1, 'C': 1}`    |                                                                 | 3                                 |                | Initial hashmap comes from counting chars. in `t`                       |
| `A` | `A` | `{'A': 1, 'B': 1, 'C': 1}`    | <code>\{<font color="red">\'A\': 0</font>, \'B\': 1, \'C\': 1\}</code>  | <font color="red">2</font>        |                | Find `'A'`                                                              |
| `A` | `D` | `{'A': 0, 'B': 1, 'C': 1}`    | `{'A': 0, 'B': 1, 'C': 1}`                                      | 2                                 |                |                                                                         |
| `A` | `O` | `{'A': 0, 'B': 1, 'C': 1}`    | `{'A': 0, 'B': 1, 'C': 1}`                                      | 2                                 |                |                                                                         |
| `A` | `B` | `{'A': 0, 'B': 1, 'C': 1}`    | <code>\{\'A\': 0, <font color="red">\'B\': 0</font>, \'C\': 1\}</code>  | <font color="red">1</font>        |                | Find `'B'`                                                              |
| `A` | `E` | `{'A': 0, 'B': 0, 'C': 1}`    | `{'A': 0, 'B': 0, 'C': 1}`                                      | 1                                 |                |                                                                         |
| `A` | `C` | `{'A': 0, 'B': 0, 'C': 1}`    | <code>\{\'A\': 0, \'B\': 0, <font color="red">\'C\': 0</font>\}</code>  | <font color="red"><b>0</b></font> | `ADOBEC`       | Find `'C'`. Everything found                                            |
| `D` | `C` | `{'A': 0, 'B': 0, 'C': 0}`    | <code>\{<font color="blue">\'A\': 1</font>, \'B\': 0, \'C\': 0\}</code> | <font color="blue">1</font>       |                | Move `l` right and we lose one `'A'`                                    |
| `D` | `O` | `{'A': 1, 'B': 0, 'C': 0}`    | `{'A': 1, 'B': 0, 'C': 0}`                                      | 1                                 |                |                                                                         |
| `D` | `D` | `{'A': 1, 'B': 0, 'C': 0}`    | `{'A': 1, 'B': 0, 'C': 0}`                                      | 1                                 |                |                                                                         |
| `D` | `E` | `{'A': 1, 'B': 0, 'C': 0}`    | `{'A': 1, 'B': 0, 'C': 0}`                                      | 1                                 |                |                                                                         |
| `D` | `B` | `{'A': 1, 'B': 0, 'C': 0}`    | <code>\{\'A\': 1, <font color="red">\'B\': -1</font>, \'C\': 1\}</code> | 1                                 |                | Find a redundant `'B'`                                                  |
| `D` | `A` | `{'A': 1, 'B': -1, 'C': 0}`   | <code>\{<font color="red">\'A\': 0</font>, \'B\': -1, \'C\': 1\}</code> | <font color="red"><b>0</b></font> | `DOBECODEBA`   | Find `'A'`. Everything found                                            |
| `O` | `A` | `{'A': 0, 'B': -1, 'C': 0}`   | `{'A': 0, 'B': -1, 'C': 0}`                                     | 0                                 | `OBECODEBA`    | Move `l` right and we lose an unneeded `'D'`                            |
| `B` | `A` | `{'A': 0, 'B': -1, 'C': 0}`   | `{'A': 0, 'B': -1, 'C': 0}`                                     | 0                                 | `BECODEBA`     | Move `l` right and we lose an unneeded `'O'`                            |
| `E` | `A` | `{'A': 0, 'B': -1, 'C': 0}`   | <code>\{\'A\': 0, <font color="blue">\'B\': 0</font>, \'C\': 0\}</code> | 0                                 | `ECODEBA`      | Move `l` right and we lose a <font color="blue">redundant</font> `'B'`  |
| `C` | `A` | `{'A': 0, 'B': 0, 'C': 0}`    | `{'A': 0, 'B': 0, 'C': 0}`                                      | 0                                 | `CODEBA`       | Move `l` right and we lose an unneeded `'E'`                            |
| `O` | `A` | `{'A': 0, 'B': 0, 'C': 0}`    | <code>\{\'A\': 0, \'B\': 0, <font color="blue">\'C\': 1</font>\}</code> | <font color="blue">1</font>       |                | Move `l` right and we lose a <code><font color="blue">\'C\'</font></code> |
| `O` | `N` | `{'A': 0, 'B': 0, 'C': 1}`    | `{'A': 0, 'B': 0, 'C': 1}`                                      | 1                                 |                |                                                                         |
| `O` | `C` | `{'A': 0, 'B': 0, 'C': 1}`    | <code>\{\'A\': 0, \'B\': 0, <font color="red">\'C\': 0</font>\}</code>  | <font color="red"><b>0</b></font> | `ODEBANC`      | Find `'C'`. Everything found                                            |
| `D` | `C` | `{'A': 0, 'B': 0, 'C': 0}`    | `{'A': 0, 'B': 0, 'C': 0}`                                      | 0                                 | `DEBANC`       | Move `l` right and we lose an unneeded `'O'`                            |
| `E` | `C` | `{'A': 0, 'B': 0, 'C': 0}`    | `{'A': 0, 'B': 0, 'C': 0}`                                      | 0                                 | `EBANC`        | Move `l` right and we lose an unneeded `'D'`                            |
| `B` | `C` | `{'A': 0, 'B': 0, 'C': 0}`    | `{'A': 0, 'B': 0, 'C': 0}`                                      | 0                                 | `BANC`         | Move `l` right and we lose an unneeded `'E'`                            |
| `A` | `C` | `{'A': 0, 'B': 0, 'C': 0}`    | <code>\{\'A\': 0, <font color="blue">\'B\': 1</font>, \'C\': 0\}</code> | <font color="blue">1</font>       |                | Move `l` right and we lose a <code><font color="blue">\'B\'</font></code> |
| `N` | `C` | `{'A': 0, 'B': 1, 'C': 0}`    | `{'A': 0, 'B': 1, 'C': 0}`                                      | 1                                 |                |                                                                         |
| `C` | `C` | `{'A': 0, 'B': 1, 'C': 0}`    | `{'A': 0, 'B': 1, 'C': 0}`                                      | 1                                 |                |                                                                         |


### Implementation

```cpp
string minWindow(string s, string t) {
    int n_remaining = t.size();  // #. of chars. in `t` to be matched
    int n = s.size();            // #. of chars. in `s`
    int hashmap[123] = {0};      // 123 as 'z' has an ASCII of 122
    for (int i = 0; i < n_remaining; i++) {  // Which chars. are needed
        hashmap[t[i]]++;
    }

    // Two pointers
    int l = 0, r = 0;
    int best_window = INT_MAX, best_l = INT_MAX;
    while (r < n && l < n) {

        // If we have not found everything
        if (n_remaining > 0) {

            // Check if `r` is needed
            if (hashmap[s[r]] > 0) {
                n_remaining--;
            }

            // Decrease the value, no matter if `r` is needed
            /*
               We don't care if `s[r]` is needed. If it's needed, fine. If it's
               not needed, it becomes negative and code below can handle it
            */
            hashmap[s[r]]--;
        }

        // After checking `r`, have we found everything now?
        if (n_remaining == 0) {

            // If yes, update the ans
            if (r - l < best_window) {
                best_window = r - l;
                best_l = l;
            }

            // If yes, move `l` right one char. and we lose one char.
            /*
               Again, we don't care if `s[l]` is needed when we increase its
               value. If it's needed, fine. If not, bc. `r` has already touched
               this char. above, it is a negative value. Plus one makes it zero
               or less negative, but never positive, so code below still works
            */
            hashmap[s[l]]++;
            if (hashmap[s[l]] > 0) {  // Check if we still have enough `s[l]`
                n_remaining++;        // after moving `l` right
                r++;
            }
            l++;

        // If we haven't found all chars., move `r` right
        } else {
            r++;
        }
    }

    // If we find a sol. above
    if (best_l < INT_MAX) {
        return s.substr(best_l, best_window + 1);
    } else {
        return "";
    }
}
```
