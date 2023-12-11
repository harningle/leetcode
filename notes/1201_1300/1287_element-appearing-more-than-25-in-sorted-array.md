# 1287. Element Appearing More Than 25% In Sorted Array

Given an integer array **sorted** in non-decreasing order, there is exactly one integer in the array that occurs more than 25% of the time, return that integer.

Example:

> **Input**: `arr = [1,2,2,6,6,6,6,7,10]`
> 
> **Output**: `6`


## Hashmap

```python
def findSpecialInteger(self, arr: List[int]) -> int:
    hashmap = {}
    threshold = int(len(arr) / 4)
    for i in arr:
        if i in hashmap:
            hashmap[i] += 1
        else:
            hashmap[i] = 1
        if hashmap[i] > threshold:
            return i
```
