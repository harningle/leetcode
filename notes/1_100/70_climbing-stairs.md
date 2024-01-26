# 70. Climbing Stairs

You are climbing a staircase. It takes `n` steps to reach the top.

Each time you can either climb $1$ or $2$ steps. In how many distinct ways can you climb to the top?


**Example:**

> **Input:** `n = 3`
> 
> **Output:** `3`
> 
> **Explanation:** There are three ways to climb to the top.
>
> 1. $1$ step + $1$ step + $1$ step
> 1. $1$ step + $2$ steps
> 1. $2$ steps + $1$ step

**Constraints:**

* `1 <= n <= 45`


## Fibonacci

Observe:

| `n`   | Answer    | Ways                                                                                                                              |
|-----  |--------   |--------------------------------------------------------------------------------------------------------------------------------   |
| 2     | 2         | $1 + 1$<br>$2$                                                                                                                      |
| 3     | 3         | $1 + 1 + 1$<br>$1 + 2$<br>$2 + 1$                                                                                                     |
| 4     | 5         | $1 + 1 + 1 + 1$<br>$1 + 1 + 2$<br>$1 + 2 + 1$<br>$2 + 1 + 1$<br>$2 + 2$                                                                   |
| 5     | 8         | $1 + 1 + 1 + 1 + 1$<br>$1 + 1 + 1 + 2$<br>$1 + 1 + 2 + 1$<br>$1 + 2 + 1 + 1$<br>$2 + 1 + 1 + 1$<br>$1 + 2 + 2$<br>$2 + 1 + 2$<br>$2 + 2 + 1$    |

The $\{2, 3, 5, 8, \ldots\}$ is actually a Fibonacci sequence, so just return the general term.

```c
int climbStairs(int n) {
    n++;
    double root5 = pow(5, 0.5);
    double result = 1 / root5 * (pow((1 + root5) / 2, n) - pow((1 - root5)/2, n));
    return (int) result;
}
```


## Recursion/DP

(The recurrence relation is actually not that easy to discover...)

If we have $a$ ways to climb up to $4$<sup>th</sup> floor, then to the $5$<sup>th</sup> floor, we only need to take one more step, so at least we will have $a$ ways to climb to the $5$<sup>th</sup> floor. Following the same reason, if there are $b$ ways to the $3$<sup>rd</sup> floor, and we need to take 2 steps to the $5$<sup>th</sup> floor, so we have $b$ more ways to the $5$<sup>th</sup> floor. So in total there are $a + b$ ways. The $a + b$ will not contain any duplicate ways, as the $a$ ways end in one step as the final step, and $b$ ways have a two-step as the final step. $a + b$ also doesn't miss out any ways, bc. any way ends in a one step or two steps. If we have one additional way with a final step of size one, then excluding this final step, we will end up on the $4$<sup>th</sup> floor, and we shall also have one more way to the $4$<sup>th</sup> floor, which contradicts with the induction hypothesis. So the problem has optimal substructure.


### Top-down with memorisation

```c
int cache[46] = {0};  // Bc. `n` is bounded below 45

int climbStairs(int n) {
    // Base case
    if (n == 1) {
        return 1;
    } else if (n == 2) {
        return 2;
    }

    // Use cache if possible
    int prev_1 = (cache[n - 1] > 0)? cache[n - 1]: climbStairs(n - 1);
    int prev_2 = (cache[n - 2] > 0)? cache[n - 2]: climbStairs(n - 2);
    
    // Recursion
    cache[n] = prev_1 + prev_2;
    return prev_1 + prev_2;
}
```

### Bottom-up

| `n`   | $Fi(n - 1)$                       | $Fi(n - 2)$           | $Fi(n)$               |
|-----  |--------------------------------   |--------------------   |---------------------  |
| 3     | $\color{green}{2}$ (base case)    | 1 (base case)         | $\color{red}{3}$      |
| 4     | $\color{red}{3}$                  | $\color{green}{2}$    | $\color{blue}{5}$     |
| 5     | $\color{blue}{5}$                 | $\color{red}{3}$      | $\color{orange}{8}$   |
| 6     | $\color{orange}{8}$               | $\color{blue}{5}$     | 13                    |

```c
int climbStairs(int n) {
    // Base case
    if (n == 1) {
        return 1;
    } else if (n == 2) {
        return 2;
    }

    int prev_1 = 2;
    int prev_2 = 1;
    int ans = 0;

    // Recursion
    for (int i = 2; i < n; i++) {
        ans = prev_1 + prev_2;
        prev_2 = prev_1;
        prev_1 = ans;
    }
    return ans;
}
```
