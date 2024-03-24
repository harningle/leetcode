# 231. Power of Two

Given an integer `n`, return `true` *if it is a power of two. Otherwise, return* `false`.

An integer `n` is a power of two, if there exists an integer $x$ such that $n = 2^x$.


**Example:**

> **Input:** `n = 16`
>
> **Output:** `true`
>
> **Explanation:** $2^4 = 16$


## Brute force

```cpp
bool isPowerOfTwo(int n) {
    // Negative numbers can't be the power of two
    if (n <= 0) {
        return false;
    }

    while (n > 1) {
        if (n % 2 != 0) {
            return false;
        }
        n /= 2;
    }
    return true;
}
```


## Bit manipulation

(Not easy to come up with the idea)


### Counting $1$’s

Observe that a power of two always has a binary representation of $1$ followed by $0$’s. E.g. $2 = 10_2, 4 = 100_2, 8 = 1000_2, \ldots$ Thus, simply counting the $1$’s will do the trick.[^Hamming weight]

[^Hamming weight]: The count is called [Hamming weight](https://en.wikipedia.org/wiki/Hamming_weight), or population count (, which is why the func. is named `popcount`).

```cpp
bool isPowerOfTwo(int n) {
    if (n <= 0) {
        return false;
    }

    if (__builtin_popcount(n) == 1) {
        return true;
    }
    return false;
}
```


### Bitwise AND

**Theorem.** $n$ is a power of two, iff $n_2\ \mathtt{AND}\ (n - 1)_2 = 0$.

**Proof of Sketch.** The "only if" part is easy. If $n$ is a power of two, its binary representation $n_2 = \underset{k\text{ digits}}{\underbrace{100\ldots0}}$, and $(n - 1)_2 = \underset{k - 1\text{ digits}}{0\underbrace{11\ldots1}}$. Bitwise AND gives zero as all digits are different between $n$ and $n - 1$.

To prove the "if" part, we need to prove that when $n_2\ \mathtt{AND}\ (n - 1)_2 = 0$, $n$ is a power of two. It itself is not easy to show, so we work on its contrapositive: if $n$ is not a power of two, $n_2\ \mathtt{AND}\ (n - 1)_2 \neq 0$. Bc. $n$ is not a power of two, then $n$ must have at least two $1$’s in its binary form $n_2 = 1\underset{\text{any }0\text{' or }1\text{'s}}{\underbrace{\ldots}}\ \underset{\text{last }1\text{ in the binary form}}{\underbrace{{\color{red}1}}}\color{blue}0\ldots 0$. Then $(n - 1)_2 = 1\underset{\text{any }0\text{' or }1\text{'s}}{\underbrace{\ldots}}\ \underset{\text{last }1\text{ becomes }0}{\underbrace{{\color{red}0}}}\ \underset{\text{all following }0\text{'s becomes }1}{\underbrace{{\color{blue}1\ldots 1}}}$. Therefore, bitwise AND gives $1\underset{\text{any }0\text{' or }1\text{'s}}{\underbrace{\ldots}}$. The leading $1$ is always there, so bitwise AND is positive. The contrapositive is true, so the original "if" part is also true. $\blacksquare$

```cpp
bool isPowerOfTwo(int n) {
    return n > 0 && (n & n - 1) == 0;
}
```
