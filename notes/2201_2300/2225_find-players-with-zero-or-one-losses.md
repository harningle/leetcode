# 2225. Find Players With Zero or One Losses

You are given an integer array `matches` where <code>matches[i] = [winner<sub>i</sub>, loser<sub>i</sub>]</code> indicates that the player $winner_i$ defeated player $loser_i$ in a match.

Return *a list* `answer` *of size* `2` *where*:

* `answer[0]` is a list of all players that have **not** lost any matches
* `answer[1]` is a list of all players that have lost exactly **one** match

The values in the two lists should be returned in **increasing** order.

**Note:**

* you should only consider the players that have played **at least one** match
* the testcases will be generated such that **no** two matches will have the **same** outcome.

 
**Example:**

> **Input:** `matches = [[1, 3], [2, 3], [3, 6], [5, 6], [5, 7], [4, 5], [4, 8], [4, 9], [10, 4], [10, 9]]`
> 
> **Output:** `[[1, 2, 10], [4, 5, 7, 8]]`
> 
> **Explanation:** 
>
> Players $1$, $2$, and $10$ have not lost any matches.
> 
> Players $4$, $5$, $7$, and $8$ each have lost one match.
>
> Players $3$, $6$, and $9$ each have lost two matches.
>
> Thus, `answer[0] = [1, 2, 10]` and `answer[1] = [4, 5, 7, 8]`.


## Hashmap

Loop over the matches, and count who has lost how many matches. Note that we need to save the winner info. as well. Otherwise, the counting will not include players without any losing.

```python
def findWinners(self, matches: List[List[int]]) -> List[List[int]]:
    hashmap = dict()
    for match in matches:
        if match[1] in hashmap:
            hashmap[match[1]] += 1
        else:
            hashmap[match[1]] = 1
        if match[0] not in hashmap:  # Winner as lose zero match. Can also use
            hashmap[match[0]] = 0    # `defaultdict`: `hashmap[winner] += 0`
    return [sorted([i for i in hashmap if hashmap[i] == 0]),
            sorted([i for i in hashmap if hashmap[i] == 1])]
```
