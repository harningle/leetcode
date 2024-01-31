# 1207. Unique Number of Occurrences

Given an array of integers `arr`, return `true` i*f the number of occurrences of each value in the array is __unique__ or* `false` *otherwise*.

Example:

> **Input**: `arr = [1, 2, 2, 1, 1, 3]`
> 
> **Output**: `true`
>
> **Explanation:** The value $1$ has 3 occurrences, $2$ has 2 and $3$ has 1. No two values have the same number of occurrences.


## Hashmap

```c
bool uniqueOccurrences(int* arr, int arrSize) {
    // Array elements are bounded in [-1000, 1000], so an array of size 2001
    // can store all of them. We use array bc. there is no builtin hashmap in C
    int hashmap[2001] = {0};
    int num;
    for (int i = 0; i < arrSize; i++) {
        num = arr[i] + 1000;
        hashmap[num]++;
    }

    // Another array to store if a #. of appearance has appeared before
    int appeared[2001] = {0};
    for (int i = 0; i < 2001; i++) {
        if (hashmap[i] > 0) {
            if (appeared[hashmap[i]] > 0) {
                return false;
            } else {
                appeared[hashmap[i]] = 1;
            }
        }
    }
    return true;
}
```
