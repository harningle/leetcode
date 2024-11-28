# 279. Perfect Squares

Given an integer `n`, return *the least number of perfect square numbers that sum to* `n`.

A **perfect square** is an integer that is the square of an integer; in other words, it is the product of some integer with itself. For example, $1$, $4$, $9$, and $16$ are perfect squares while $3$ and $11$ are not.


**Example:**

> **Input:** `n = 12`
>
> **Output:** `3`
>
> **Explanation:** $12 = 4 + 4 + 4$.


## DP

I first thought we could have a greedy sol. Below, $12$, we have $9$, $4$, and $1$. We always try to use the largest possible sq. number, so here $12 - 9 = 3$, and then $3 = 1 + 1 + 1$. But then we need four numbers. But if we use $12 - 4 = 8$, and then $8 = 4 + 4$, we only need three numbers. So greedily taking the largest possible sq. number does not give the optimal sol. We have to try all possibilities in recursion:

1. start with $12$, we know $9$, $4$, and $1$ are possible sq. numbers
1. for each of those sq. numbers $i$
    1. take $i$ away from $12$, leaving us with $12 - i$
    1. if $12 - i = 0$, then we know we only need $1$ sq. number
    1. if not, start from the beginning with the input number $12 - i$, i.e. recursion

This is essentially a BFS, or DP, or recursion, whatever you call it.

<img src="279.svg" width="80%"/>


```cpp
int dp(int n, vector<int>& cache) {
    // Use memory if possible
    if (cache[n] >= 0) {
        return cache[n];
    }

    // Find the largest sq. number <= `n`
    int max_sq = sqrt(n);
    int ans = INT_MAX;
    int temp;
    int remaining;

    // Try all possible sq. numbers <= `n`
    for (int i = max_sq; i >= 1; i--) {
        remaining = n - i * i;

        // If `remaining` itself is a sq. number, then we only need one
        if (remaining == 0) {
            return 1;
        }

        // If not, recursion with `n - i`
        temp = dp(remaining, cache);
        ans = min(temp + 1, ans);
    }
    cache[n] = ans;  // Save the computed sol. to number `n`
    return ans;
}

int numSquares(int n) {
    vector<int> cache(n + 1, -1);
    return dp(n, cache);
}
```

The complexity is $n\sqrt{n}$, bc. in the worst case, we need to try all $n - 1, n - 2, \ldots, 1$, and for each of them, we try all square numbers below them, and there are at most $\sqrt{k}$ square numbers $\leq k$.

*Notes*: BFS may be slightly faster, as we can early stop at the shallowest depth, while the above recursion implementation tries all. But with memorisation, probably it won't matter much. There is a ["static" DP solution in LeetCode discussion](https://leetcode.com/problems/perfect-squares/solutions/71488/summary-of-4-different-solutions-bfs-dp-static-dp-and-mathematics). I think it basically cheats the LeetCode evaluator: it stores the cache in some sort of "global", and all tests use the same cache, which we shouldn't do.


## Lagrange's four-square theorem[^source]

[^source]: See [https://leetcode.com/problems/perfect-squares/solutions/71532/o-sqrt-n-about-0-034-ms-and-0-018-ms](https://leetcode.com/problems/perfect-squares/solutions/71532/o-sqrt-n-about-0-034-ms-and-0-018-ms) and [Wikipedia](https://en.wikipedia.org/wiki/Lagrange%27s_four-square_theorem).

(OK even faster than linear time...)

**Lagrange's Four-Square Theorem.** Any natural number $p$ can be represented as a sum of four square numbers, i.e. $p = a^2 + b^2 + c^2 + d^2$, where $p, a, b, c, d\in \mathbb{N}$.

**Lemma 1 (Euler's Four-Square Identity).** The product of two numbers, each of which is a sum of four square numbers, can be written as a sum of four square numbers. That is,

$$
\begin{align*}
(a_1^2 + a_2^2 + a_3^2 + a_4^2)\times (b_1^2 + b_2^2 + b_3^2 + b_4^2) &= (a_1 b_1 - a_2 b_2 - a_3 b_3 - a_4 b_4)^2\\ 
 &+ (a_1 b_2 + a_2 b_1 + a_3 b_4 - a_4 b_3)^2 \\ 
 &+ (a_1 b_3 - a_2 b_4 + a_3 b_1 + a_4 b_2)^2\\ 
 &+ (a_1 b_4 + a_2 b_3 - a_3 b_2 + a_4 b_1)^2
\end{align*}
$$

, where $a_i, b_i \in \mathbb{N}, \forall i\in \{1, 2, 3, 4\}$.

**Proof of Lemma 1.** Brute force algebra.


**Lemma 2.** Given a prime number $p\geq 3$, $\forall a\in\mathbb{N}$ and $a \leq \dfrac{p - 1}{2}$, $a^2 \mathrm{\ mod\ } p$ are all distinct.

**Proof of Lemma 2.** Let's first see an example. Let $p = 13$. Then $a \in \{0, 1, 2, 3, 4, 5, 6\}$. $0^2 \mathrm{\ mod\ } 13 = \color{grey}0$, $1^2 \mathrm{\ mod\ } 13 = \color{red}1$, $2^2 \mathrm{\ mod\ } 13 = \color{blue}4$, $3^2 \mathrm{\ mod\ } 13 = \color{pink}9$, $4^2 \mathrm{\ mod\ } 13 = \color{green}3$, $5^2 \mathrm{\ mod\ } 13 = \color{orange}12$, $6^2 \mathrm{\ mod\ } 13 = \color{purple}10$. The $\{{\color{grey}0}, {\color{red}1}, {\color{blue}4}, {\color{pink}9}, {\color{green}3}, {\color{orange}12}, {\color{purple}10}\}$ are all distinct.

We can prove it by contradiction. Assume there are two different numbers $u > v$ with $u^2 \mathrm{\ mod\ } p = v^2 \mathrm{\ mod\ } p$. This means we have two different roots for the eq. $x^2 \mathrm{\ mod\ } p = r$, where $r\in\mathbb{Z}^*$ is some given integer less than $p$. This implies $x^2 = r + kp, k\in\mathbb{Z}^*$. This is a quadratic func. and have at most two roots, so in principle we can have two different roots $u$ and $v$. Now we are going to prove the two roots must be the same.

Factorising the assumption, we have $(u^2 - v^2) \mathrm{\ mod\ } p = 0$, or $\big ( (u + v)(u - v) \big ) \mathrm{\ mod\ }p = 0$. Bc. $p$ is a prime, we need either $u + v$ being a multiple of $p$, or $u - v$ being a multiple of $p$.

* assume $u + v$ is a multiple of $p$. Then we have $(u + v) \mathrm{\ mod \ } p = 0$. Bc. $0\leq u, v\leq \dfrac{p - 1}{2}$ and $u > v$, we have $\underset{u = 1, v = 0}{\underbrace{1}}\leq u + v\leq \underset{u = \frac{p - 1}{2}, v = \frac{p - 1}{2} - 1}{\underbrace{p - 2}}$. $p$ is a prime, so all numbers less than $p$ cannot be a multiple of $p$, so $u + v\leq p - 2 < p$ cannot be a multiple of $p$
* assume $u - v$ is a multiple of $p$. Similarity, $\underset{v = u - 1}{\underbrace{1}}\leq u - v\leq \underset{u = \frac{p - 1}{2}, v = 0}{\underbrace{\frac{p - 1}{2}}}$. Therefore, $u - v\leq \frac{p - 1}{2}$ is less than $p$, so cannot be a multiple of $p$

Contradiction. Therefore $u\neq v$, so we don't have any duplicate $a^2 \mathrm{\ mod\ } p$ when $0\leq a \leq \dfrac{p - 1}{2}$. $\blacksquare$


**Proof.** Let's skip $0$, $1$, and $2$ as they are trivial. When $p\geq 3$, it can be a composite number or a prime number. Suppose the theorem holds for all greater-than-3 prime numbers. If $p$ is composite, then by fundamental theorem of arithmetic, $p$ can be uniquely factored into a product of multiple primes, e.g. $\displaystyle p = \prod_i q_i, i\geq 2$, where all $q$’s are primes. Then we take the first two $q$’s. Bc. we assume the theorem holds for primes numbers, $q_1 = a_1^2 + b_1^2 + c_1^2 + d_1^2$ and $q_2 = a_2^2 + b_2^2 + c_2^2 + d_2^2$. Then by Euler's four-square identity, $Q_1 := q_1 \times q_2 = e_1^2 + f_1^2 + g_1^2 + h_1^2$. Then we take $Q_1$ and $q_3$, both of which are sum of four squares. Then apply Euler's four-square identity again, we can show their product is also a sum of four squares. By induction, it's easy to show that $\displaystyle p = \prod_i q_i$ can be written as a sum of four squares.

Now the problem is to prove the theorem for greater-than-3 primes. Let's assume 


