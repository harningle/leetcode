# 1347. Minimum Number of Steps to Make Two Strings Anagram

You are given two strings of the same length `s` and `t`. In one step you can choose **any character** of `t` and replace it with **another character**.

Return *the minimum number of steps* to make `t` an anagram of `s`.

An **Anagram** of a string is a string that contains the same characters with a different (or the same) ordering.


**Example:**

> **Input:** `s = "leetcode"`, `t = "practice"`
> 
> **Output:** `5`
> 
> **Explanation:** Replace `'p'`, `'r'`, `'a'`, `'i'` and `'c'` from `t` with proper characters to make `t` anagram of `s`.


## Hashmap

<style>
    .strikediag {
        background: linear-gradient(to left top, transparent 45%, currentColor 45%, currentColor 55%, transparent 55%);
    }
</style>

For "leetcode" and "practice", the "e", "t", "c" are already there. We need two more "e", one more "o", one more "l", one more "d", one less "p", one less "r", one less "a", one less "c", and one less "i". To visualise it:

| `s`                                 | `t`        |
|-------------------------------------|------------|
| "leetcode"                          | "practice" |
| "l<span class="strikediag">e</span>etcode" | "practic<span class="strikediag">e</span>” |
| "l<span class="strikediag">e</span>e<span class="strikediag">t</span>code"                          | "prac<span class="strikediag">t</span>ic<span class="strikediag">e</span>” |
| "l<span class="strikediag">e</span>e<span class="strikediag">t</span><span class="strikediag">c</span>ode"                          | "pra<span class="strikediag">c</span><span class="strikediag">t</span>ic<span class="strikediag">e</span>” |

, or

| `s`   | `t`   | Hashmap (positive means too many letters, negative means lacking letters) |
|-------|-------|------------------------------------------------------------------------------------------|
| `'l'` | `'p'` | `{'l': -1, 'p': 1}`                                                                      |
| `'e'` | `'r'` | `{'l': -1,, 'e': -1, 'p': 1, 'r': 1}`                                                    |
| `'e'` | `'a'` | `{'l': -1,, 'e': -2, 'p': 1, 'r': 1, 'a': 1}`                                            |
| `'t'` | `'c'` | `{'l': -1,, 'e': -2, 't': -1, 'p': 1, 'r': 1, 'a': 1, 'c': 1}`                          |
| `'c'` | `'t'` | `{'l': -1,, 'e': -2, 't': 0, 'p': 1, 'r': 1, 'a': 1, 'c': 0}`                           |
| `'o'` | `'i'` | `{'l': -1,, 'e': -2, 't': 0, 'o': -1, 'p': 1, 'r': 1, 'a': 1, 'c': 0, 'i': 1}`          |
| `'d'` | `'c'` | `{'l': -1,, 'e': -2, 't': 0, 'o': -1, 'd': -1, 'p': 1, 'r': 1, 'a': 1, 'c': 1, 'i': 1}` |
| `'e'` | `'e'` | `{'l': -1,, 'e': -2, 't': 0, 'o': -1, 'd': -1, 'p': 1, 'r': 1, 'a': 1, 'c': 1, 'i': 1}` |

Count the #. of appearances for each letter in the two strings, and calc. the diff. in appearances. Summing up all diff. gives the answer.


## Sol.

```c
#include <string.h>

int minSteps(char* s, char* t) {
    int hashmap[26] = {0};  // 26 letters
    int n_chars = strlen(s);
    for (int i = 0; i < n_chars; i++) {
        hashmap[s[i] - 97]++;  // "a" has an ascii of 97
        hashmap[t[i] - 97]--;
    }
    int ans = 0;
    for (int i = 0; i < 26; i++) {
        if (hashmap[i] > 0) {   // E.g., we lack five letters and have five
            ans += hashmap[i];  // more letters. The #. of steps is five, not 
        }                       // ten
    }
    return ans;
}
```

Just a note: there is no builtin hashmap in C, so here we use an array.
