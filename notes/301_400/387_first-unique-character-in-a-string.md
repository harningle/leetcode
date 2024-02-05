# 387. First Unique Character in a String

Given a string `s`, *find the first non-repeating character in it and return its index*. If it does not exist, return `-1`.

 
**Example:**

> **Input:** `s = "leetcode"`
> 
> **Output:** `0`


**Constraints:**

* `s` consists of only lowercase English letters


## Hashmap

It's very easy to count the #. of appearance for each char. in `s` using hashmap. Then among all chars. with the only one appearance, we search for the index of the first such char. in `s`. The trick is, we don't have to traverse `s` again to find its index. We can save its index in the hashmap, and only need to go through the hashmap, which has at most 26 chars.

```c++
int firstUniqChar(string s) {
    /*
       Hashmap: char. --> its first appearance index in `s`, which is >= 0
                          -1 if multiple appearances
                          -2 if not seen this char. before
    */
    vector<int> hashmap(123, -2);  // Init. with -2. 123 bc. 'z' has ASCII 122

    // Find the first appearance for each char.
    int n = s.size();
    for (int i = 0; i < n; i++) {

        // If not seen before, assign the value
        if (hashmap[s[i]] == -2) {
            hashmap[s[i]] = i;
        
        // If seen before, set the value to -1 to mark it as duplicated
        } else {
            hashmap[s[i]] = -1;
        }
    }

    // Go through 'a' to 'z' and find the first char. with one appearance
    int ans = INT_MAX;
    for (int i = 97; i <= 122; i++) {  // 'a' has ASCII 97
        if (hashmap[i] >= 0) {
            ans = (hashmap[i] < ans) ? hashmap[i] : ans;
        }
    }

    // Check if found any char. above
    if (ans < INT_MAX) {
        return ans;
    } else {
        return -1;
    }
}
```
