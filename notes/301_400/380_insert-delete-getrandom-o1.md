# 380. Insert Delete GetRandom O(1)

Implement the `RandomizedSet` class:

* `RandomizedSet()` initializes the `RandomizedSet` object
* `bool insert(int val)` inserts an item `val` into the set if not present. Returns `true` if the item was not present, `false` otherwise
* `bool remove(int val)` removes an item `val` from the set if present. Returns `true` if the item was present, `false` otherwise
* `int getRandom()` returns a random element from the current set of elements (it's guaranteed that at least one element exists when this method is called). Each element must have the same probability of being returned.

You must implement the functions of the class such that each function works in average $O(1)$ time complexity

 
**Example:**

> **Input:**
> 
> `["RandomizedSet", "insert", "remove", "insert", "getRandom", "remove", "insert", "getRandom"]`
> 
> `[[], [1], [2], [2], [], [1], [2], []]`
> 
> **Output**
> 
> `[null, true, false, true, 2, true, false, 2]`
> 
> **Explanation**
>
> ```java
> RandomizedSet randomizedSet = new RandomizedSet();
> randomizedSet.insert(1);   // Inserts 1 to the set. Returns true as 1 was
>                            // inserted successfully
> randomizedSet.remove(2);   // Returns false as 2 does not exist in the set
> randomizedSet.insert(2);   // Inserts 2 to the set, returns true. Set now
>                            // contains [1,2]
> randomizedSet.getRandom(); // getRandom() should return either 1 or 2
>                            // randomly
> randomizedSet.remove(1);   // Removes 1 from the set, returns true. Set now
>                            // contains [2]
> randomizedSet.insert(2);   // 2 was already in the set, so return false
> randomizedSet.getRandom(); // Since 2 is the only number in the set,
>                            // getRandom() will always return 2
> ```



## One pass

The problem is easy but doing it in $O(1)$ is hard.

|         | Insert | Check element existence | Remove | Randomly choose |
|---------|--------|-------------------------|--------|-----------------|
| Array   | $O(1)$ | $\color{red}{O(n)}$                  | $\color{red}{O(n)}$ | $O(1)$          |
| Hashmap | $O(1)$ | $O(1)$                  | $O(1)$ | $\color{red}{O(n)}$[^hashmap keys]          |

[^hashmap keys]: Getting all keys in a hashmap and populating them in an array is $O(n)$, e.g. `list(dict.keys())` in Python. We have to populate the keys in an array bc. we can't randomly pick a key otherwise.

There is no single data structure that can have everything in $O(1)$, so we need both. Basically, we need an array to store the keys of the hashmap in $O(1)$ so we can call `random.choice` on this array. When we insert a new key, we append a new element to the end of the array, which is $O(1)$. However, as we sometimes want to remove a key, we need to figure out a way to delete element in the array in $O(1)$. The trick is to use index. E.g. `keys = [1, 4, 7, 3, 10]`. We want to delete $7$, whose index is $2$, and the new `keys = [1, 4, 3, 10]`. So we basically want to remove $7$ and shift $3$ and $10$ into the positions previously held by $7$ and $3$. However, such shifting is $O(n)$. Alternatively, as we don't care about the order of keys, we can keep $3$ untouched, and replace $3$ with $10$, which is one single operation, giving `keys = [1, 4, 10, 3]`. That is, we can save the final element, and put that element into the index of the element to be deleted, and pop out the final element. Such deletion is possible only if we can get the index of keys in $O(1)$. So far we only care about the keys of the hashmap, so now we can store the index as values to make the query $O(1)$.


```python
import random


class RandomizedSet:
    def __init__(self):
        self.vals = list()
        self.val_to_idx = dict()  # Key = element value, value = element idx.

    def insert(self, val: int) -> bool:
        if val in self.val_to_idx:
            return False
        else:
            self.vals.append(val)
            self.val_to_idx[val] = len(self.vals) - 1
            return True

    def remove(self, val: int) -> bool:
        if val in self.val_to_idx:
            idx_to_remove = self.val_to_idx[val]
            self.vals[idx_to_remove] = self.vals[-1]
            self.val_to_idx[self.vals[-1]] = idx_to_remove
            self.vals.pop()
            del self.val_to_idx[val]
            return True
        else:
            return False

    def getRandom(self) -> int:
        return random.choice(self.vals)
```
