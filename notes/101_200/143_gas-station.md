# 134. Gas Station

There are $n$ gas stations along a circular route, where the amount of gas at the $i$<sup>th</sup> station is `gas[i]`.

You have a car with an unlimited gas tank and it costs `cost[i]` of gas to travel from the $i$<sup>th</sup> station to its next $(i + 1)$<sup>th</sup> station. You begin the journey with an empty tank at one of the gas stations.

Given two integer arrays `gas` and `cost`, return *the starting gas station's index if you can travel around the circuit once in the clockwise direction, otherwise return* `-1`. If there exists a solution, it is **guaranteed** to be **unique**.

 
**Example:**

> **Input:** `gas = [1, 2, 3, 4, 5], cost = [3, 4, 5, 1, 2]`
> 
> **Output:** `3`
> 
> **Explanation:**
>
> Start at station $3$ (index $3$) and fill up with $4$ unit of gas. Your tank $= 0 + 4 = 4$
> 
> Travel to station $4$. Your tank = $4 - 1 + 5 = 8$
>
> Travel to station $0$. Your tank = $8 - 2 + 1 = 7$
>
> Travel to station $1$. Your tank = $7 - 3 + 2 = 6$
>
> Travel to station $2$. Your tank $= 6 - 4 + 3 = 5$
>
> Travel to station $3$. The cost is $5$. Your gas is just enough to travel back to station $3$.
> 
> Therefore, return `3` as the starting index.


## One pass

The solution is not easy to come up. There are two ways to understand it. Let's begin with the $O(n^2)$ brute force:

1. try all possible starting position
1. see if we have enough gas in tank to move to the next station
1. see if we run out of gas at some point or we successfully reach the starting position again

### Skip some starting positions

However, we are able to skip some starting point. Say we start at index $i$, move all the way to index $j$, and then don't have enough gas to move to $j + 1$, and fail. In this case, all positions between $i$ and $j$ are all destined to fail. To prove it, the gas in tank when we arrive at $j$ is $tank_j = (gas_i + gas_{i + 1} + \ldots + gas_{j}) - (cost_i + cost_{i + 1} + \ldots + cost_{j - 1})$. We are able to start at position $i$, which means $gas_i > cost_i$. Say now we want to try starting at position $i + 1$. Now $tank_j' = (\cancel{gas_i +} gas_{i + 1} + \ldots + gas_{j}) - (\cancel{cost_i +} cost_{i + 1} + \ldots + cost_{j - 1})$. $tank_j'$ cannot be greater than $tank_j$ because we lose the $gas_i$ and $cost_i$ with $gas_i > cost_i$. So if starting at index $i$ fails, starting at index $i + 1$ will also fail bc. it gives us same or strictly less amount of gas when we arrive at $j$. By induction, all positions between $i$ and $j$ fail. So the next position to check is $j + 1$.

Then, how to "check"? We know we fail if we don't have enough gas to move forward. But is not failing sufficient to guarantee success? That is, we start at $i$, move all the way to the end. Is it enough? No! Of course we need to check if we can move back to $i - 1$. But there is a better way to check: the sufficient (and necessary) condition for the existence of a solution is we have "net gains". That is, total amount of gas is greater than total costs. It's obviously necessary, the we will prove the sufficiency in the second ways of understanding.


```cpp
int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
    int n = gas.size();
    int net_gains = 0;
    int tank = 0;
    int start = 0;

    for(int i = 0; i < n; i++){
        net_gains += gas[i] - cost[i];
        tank += gas[i];
        if (tank < cost[i]) {
            tank = 0;
            start = i + 1;
        } else {
            tank -= cost[i];
        }
    }
    return (net_gains < 0) ? -1 : start;
}
```


### Start at the worst position



