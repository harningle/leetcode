# 1716. Calculate Money in Leetcode Bank

Hercy wants to save money for his first car. He puts money in the Leetcode bank **every day**.

He starts by putting in \$1 on Monday, the first day. Every day from Tuesday to Sunday, he will put in \$1 more than the day before. On every subsequent Monday, he will put in \$1 more than the **previous Monday**.

Given `n`, *return the total amount of money he will have in the Leetcode bank at the end of the* `nth` *day*.

**Example:**

> **Input:** `n = 20`
> 
> **Output:** `96`
> 
> **Explanation:** After the 20th day, the total is `(1 + 2 + 3 + 4 + 5 + 6 + 7) + (2 + 3 + 4 + 5 + 6 + 7 + 8) + (3 + 4 + 5 + 6 + 7 + 8) = 96`.


## Math

$$
\underset{1 + 2 + 3 + 4 + 5 + 6 + 7}{\underbrace{1 + 2 + 3 + 4 + 5 + 6 + 7}}
+ \underset{\begin{matrix}1 + 2 + 3 + 4 + 5 + 6 + 7\\ + 1 + 1 + 1 + 1 + 1 + 1 + 1\end{matrix}}{\underbrace{2 + 3 + 4 + 5 + 6 + 7 + 8}}
+ \underset{\begin{matrix}1 + 2 + 3 + 4 + 5 + 6\\ + 1 + 1 + 1 + 1 + 1 + 1\\ + 1 + 1 + 1 + 1 + 1 + 1\end{matrix}}{\underbrace{3 + 4 + 5 + 6 + 7 + 8}}
$$

```c
int totalMoney(int n) {
    // How many full weeks
    int n_weeks = n / 7;

    // How many days in the last non-full week
    int n_remaining_days = n - n_weeks * 7;

    // Money from the full weeks
    int ans = (1 + 2 + 3 + 4 + 5 + 6 + 7) * n_weeks + (0 + n_weeks - 1) * n_weeks / 2 * 7;

    // Money from the last non-full week
    ans += (1 + n_remaining_days) * n_remaining_days / 2 + n_weeks * n_remaining_days;
    return ans;
}
```
