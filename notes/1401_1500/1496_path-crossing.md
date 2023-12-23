# 1496. Path Crossing

Given a string `path`, where `path[i]` = `'N'`, `'S'`, `'E'` or `'W'`, each representing moving one unit north, south, east, or west, respectively. You start at the origin `(0, 0)` on a 2D plane and walk on the path specified by `path`.

Return `true` *if the path crosses itself at any point, that is, if at any time you are on a location you have previously visited*. Return `false` otherwise.

**Example:**

<img src="https://assets.leetcode.com/uploads/2020/06/10/screen-shot-2020-06-10-at-123843-pm.png" width="50%"/>

> **Input:** `path = "NESWW"`
> 
> **Output:** `true`
> 
> **Explanation:** Notice that the path visits the origin twice.


## Brute force

Store all points visited before, and see if the current move will move to any of those points.

```python
def isPathCrossing(path: str) -> bool:
    # Start from origin
    x, y = 0, 0
    visited = set()
    visited.add((x, y))

    # Check all moves
    for move in path:
        match move:
            case 'N':
                y += 1
            case 'S':
                y -= 1
            case 'W':
                x -= 1
            case 'E':
                x += 1
        if (x, y) in visited:
            return True
        visited.add((x, y))

    # If the code reaches here, it means we already deplete the path and there is no crossing
    return False
```
