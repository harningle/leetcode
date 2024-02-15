# 2108. Find First Palindromic String in the Array

Given an array of strings `words`, return *the first __palindromic__ string in the array*. If there is no such string, return *an __empty string__* `""`.

A string is **palindromic** if it reads the same forward and backward.


**Example:**

> **Input:** `words = ["abc", "car", "ada", "racecar", "cool"]`
>
> **Output:** `"ada"`
> 
> **Explanation:** The first string that is palindromic is `"ada"`.
>
> Note that `"racecar"` is also palindromic, but it is not the first.


## Two pointers

```cpp
bool isPalindrome(string word) {
    int n = word.size();
    int l = 0, r = n - 1;
    while (l < r) {
        if (word[l] != word[r]) {
            return false;
        }
        l++;
        r--;
    }
    return true;
}

string firstPalindrome(vector<string>& words) {
    for (auto word: words) {
        if (isPalindrome(word)) {
            return word;
        }
    }
    return "";
}
```

Surprisingly it only beats 50% users??? No idea why.
