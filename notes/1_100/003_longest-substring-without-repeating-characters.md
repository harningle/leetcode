# 3. Longest Substring Without Repeating Characters

Given a string `s`, find the length of the longest substring without repeating characters.


## Two pointer + hashmap

|        | `left` | `right` | Hashmap                            | `right` char. in hashmap? | Hashmap after updating             | Substring | Answer |
|--------|--------|---------|------------------------------------|---------------------------|------------------------------------|-----------|--------|
| <ins>p</ins><ins>w</ins>wkew | 0      | 1       | `{'p': 0}`                         | `False`                   | `{'p': 0, 'w': 1}`                 | "pw"      |        |
| <ins>p</ins>w<ins>w</ins>kew | 0      | 2       | `{'p': 0, 'w': 1}`                 | `True`                    | `{'p': 0, 'w': 2}`                 | "pw"      | 2      |
| pw<ins>w</ins><ins>k</ins>ew | 2      | 3       | `{'p': 0, 'w': 2}`                 | `False`                   | `{'p': 0, 'w': 2, 'k': 3}`         | "wk"      |        |
| pw<ins>w</ins>k<ins>e</ins>w | 2      | 4       | `{'p': 0, 'w': 2, 'k': 3}`         | `False`                   | `{'p': 0, 'w': 2, 'k': 3, 'e': 4}` | "wke"     |        |
| pw<ins>w</ins>ke<ins>w</ins> | 2      | 5       | `{'p': 0, 'w': 2, 'k': 3, 'e': 4}` | `True`                    | `{'p': 0, 'w': 5, 'k': 3, 'e': 4}` | "wke"     | 3      |

```python
def lengthOfLongestSubstring(s: str) -> int:
    # Empty string or one char. only: the longest substring is the string it self
    if len(s) <= 1:
        return len(s)
    
    # Two pointers
    left = 0
    right = 1
    hashmap = {s[left]: 0}
    ans = 1
    while right < len(s):
        right_c = s[right]
        if right_c in hashmap:  # Right char. appeared before?
            if hashmap[right_c] >= left:  # Appeared to the right of `left`?
                ans = max(ans, right - left)
                left = hashmap[right_c] + 1  # Move to the appearance + 1
        hashmap[right_c] = right
        right += 1
    return max(ans, right - left)
```
