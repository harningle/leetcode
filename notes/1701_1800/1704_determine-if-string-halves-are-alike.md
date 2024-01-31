# 1704. Determine if String Halves Are Alike

You are given a string `s` of even length. Split this string into two halves of equal lengths, and let `a` be the first half and `b` be the second half.

Two strings are **alike** if they have the same number of vowels (`'a'`, `'e'`, `'i'`, `'o'`, `'u'`, `'A'`, `'E'`, `'I'`, `'O'`, `'U'`). Notice that `s` contains uppercase and lowercase letters.

Return `true` *if* `a` *and* `b` *are __alike__*. Otherwise, return `false`.

**Example:**

> **Input:** `s = "book"`
> 
> **Output:** `true`
> 
> **Explanation:** <code>a = \"b<u>o</u>\"</code> and <code>a = \"<u>o</u>k\"</code>. `a` has 1 vowel and `b` has 1 vowel. Therefore, they are alike.


## Simple loop

```python
def halvesAreAlike(s: str) -> bool:
    vowels = {'a', 'e', 'i', 'o', 'u', 'A', 'E', 'I', 'O', 'U'}
    n_vowels_left = 0
    n_vowels_right = 0
    length = len(s)
    for i in range(length):
        c = s[i]
        if c in vowels:
            if i < length // 2:
                n_vowels_left += 1
            else:
                n_vowels_right += 1
    return n_vowels_left == n_vowels_right
```

Can also use two pointers: start from the beginning and end of string, and move towards the centre until two pointers meet with each other. Looks better but time complexity is exactly the same.
