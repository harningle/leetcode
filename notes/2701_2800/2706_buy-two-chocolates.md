# 2706. Buy Two Chocolates

You are given an integer array `prices` representing the prices of various chocolates in a store. You are also given a single integer `money`, which represents your initial amount of money.

You must buy **exactly** two chocolates in such a way that you still have some **non-negative** leftover money. You would like to minimize the sum of the prices of the two chocolates you buy.

Return *the amount of money you will have leftover after buying the two chocolates*. If there is no way for you to buy two chocolates without ending up in debt, return `money`. Note that the leftover must be non-negative.

**Example:**

> **Input:** `prices = [1,2,2], money = 3`
> **Output:** `0`
> **Explanation:** Purchase the chocolates priced at $1$ and $2$ units respectively. You will have $3 - 3 = 0$ units of money afterwards. Thus, we return `0`.


## Linear search

Linear search for the cheapest two chocolates, same intuition as problem [1913](https://github.com/harningle/leetcode/blob/main/notes/1901_2000/1913_maximum-product-difference-between-two-pairs.md).

```c
int buyChoco(int* prices, int pricesSize, int money){
    // Linear search for the cheapest two
    int c1 = INT_MAX;  // Cheapest
    int c2 = INT_MAX;  // 2nd cheapest
    int p;
    for (int i = 0; i < pricesSize; i++) {
        p = prices[i];
        if (p <= c1) {
            c2 = c1;
            c1 = p;
        } else if (p < c2) {
            c2 = p;
        }
    }

    // If have >= 0 money left after buying the two chocolates
    if (money - c1 - c2 >= 0) {
        return money - c1 - c2;

    // Otherwise return the original money
    } else {
        return money;
    }
}
```


