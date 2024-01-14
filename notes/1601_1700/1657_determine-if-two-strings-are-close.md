# 1657. Determine if Two Strings Are Close

Two strings are considered **close** if you can attain one from the other using the following operations:

* Operation 1: Swap any two **existing** characters
    * For example, <code>a<u>b</u>cd<u>e</u> -> a<u>e</u>cd<u>b</u></code>
* Operation 2: Transform **every** occurrence of one **existing** character into another **existing** character, and do the same with the other character
    * For example, <code><u>aa</u>ca<u>bb</u> -> <u>bb</u>cb<u>aa</u></code> (all `a`’s turn into `b`’s, and all `b`’s turn into `a`’s)

You can use the operations on either string as many times as necessary.

Given two strings, `word1` and `word2`, return `true` *if* `word1` *and* `word2` *are __close__**, and* `false` *otherwise*.


**Example:**

> **Input:** `word1 = "cabbba"`, `word2 = "abbccc"`
> 
> **Output:** `true`
> 
> **Explanation:** You can attain `word2` from `word1` in 3 operations.
>
> Apply Operation 1: `"cabbba"` -> `"caabbb"`
> 
> Apply Operation 2: `"caabbb"` -> `"baaccc"`
> 
> Apply Operation 2: `"baaccc"` -> `"abbccc"`

**Constraints:**

`word1` and `word2` contain only lowercase English letters.


## Hashmap

Operation 1 swaps two letters and does not change the #. of appearances of any letter. Operation 2 exchange two letters and swaps of the #. of appearances of the two letters. So as long as `word1` and `word2` have

1. the same set of unique letters, bc. neither operation can create non-existing letter, and
1. the same #. of appearances of letters, irrespective of the order. E.g. say `word1 = "aaab"` (three `a`’s and one `b`’s) and `word2 = "bbba"` (three `b`’s and one `a`’s). As long as the #. of appearances is 3 and 1, no matter which letter has three appearances or one appearance, we can always make `word1` and `word2` the same by operation 2

, they are close.

```python
def closeStrings(self, word1: str, word2: str) -> bool:
    if len(word1) != len(word2):
        return False

    hashmap_1 = defaultdict(int)
    hashmap_2 = defaultdict(int)
    for i in range(len(word1)):
        hashmap_1[word1[i]] += 1
        hashmap_2[word2[i]] += 1
    
    if sorted(hashmap_1.keys()) == sorted(hashmap_2.keys()):
        if sorted(hashmap_1.values()) == sorted(hashmap_2.values()):
            return True
    return False

```

The algo. is $O(n)$,[^sort constant length array] but can be optimised a bit. We don't really need to check both keys and values. E.g., for any key in `hashmap_1`, if it's not in `hashmap_2` (i.e. zero appearance of the letter), we directly return `False` as this letter is non-existing in `word2`. By doing this, we only need one loop and one sorting but not two sorting. But in practice this brings little improvement, so the above solution is good enough.

[^sort constant length array]: We do have two sorting but as we have at most 26 letters, the array size is fixed/bounded below 26. I.e., we are not sorting an array of size $n$, but sorting 26 numbers. Sorting a fixed size array is $O(1)$.

```c
int cmp(const void* a, const void* b) {
    return (*(int*) a > *(int*) b) - (*(int*) a < *(int*) b);
}

bool closeStrings(char* word1, char* word2) {
    if (strlen(word1) != strlen(word2)) {
        return false;
    }
    
    int hashmap_1[26] = {0};
    int hashmap_2[26] = {0};
    for (int i = 0; i < strlen(word1); i++) {
        hashmap_1[word1[i] - 97]++;
        hashmap_2[word2[i] - 97]++;
    }

    for (int i = 0; i < 26; i++) {
        if (   ((hashmap_1[i] == 0) && (hashmap_2[i] > 0))
            || ((hashmap_1[i] > 0)  && (hashmap_2[i] == 0))) {
            return false;
        }
    }

    qsort(hashmap_1, 26, sizeof(int), cmp);
    qsort(hashmap_2, 26, sizeof(int), cmp);
    for (int i = 0; i < 26; i++) {
        if (hashmap_1[i] != hashmap_2[i]) {
            return false;
        }
    }
    return true;
}
```
