# 242. Valid Anagram

Given two strings `s` and `t`, return `true` *if* `t` *is an anagram of* `s`*, and* `false` *otherwise.*

An **Anagram** is a word or phrase formed by rearranging the letters of a different word or phrase, typically using all the original letters exactly once.

**Example:**

> **Input:** `s = "anagram", t = "nagaram"`
>
> **Output:** `true`


## Hashmap

Count how many times a letter appears in the first string. Count the second string, but this time minus one for each appearance. In the end we should have a hashmap with all values equalling zero.

```python
def isAnagram(self, s: str, t: str) -> bool:
    # Hashmap
    hashmap = defaultdict(int)
    for i in s:
        hashmap[i] += 1
    for i in t:
        hashmap[i] -= 1
    for i in hashmap.values():
        if i:
            return False
    return True
```
