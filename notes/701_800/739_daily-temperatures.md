# 739. Daily Temperatures

Given an array of integers `temperatures` represents the daily temperatures, return *an array* `answer` *such that* `answer[i]` *is the number of days you have to wait after the* `i`*<sup>th</sup> day to get a warmer temperature*. If there is no future day for which this is possible, keep `answer[i] == 0` instead.


**Example:**

> **Input:** `temperatures = [73, 74, 75, 71, 69, 72, 76, 73]`
> 
> **Output:** `[1, 1, 4, 2, 1, 1, 0, 0]`


## Monotonic decreasing stack

It's obviously a monotonic stack problem, especially after daily question 1335... See [here](../../data-structure/monotonic-stack.md).

```cpp
vector<int> dailyTemperatures(vector<int>& temperatures) {
    int n_days = temperatures.size();
    stack<int> s;  // Monotonic decreasing stack
    vector<int> ans(n_days, 0);

    int temp;
    for (int i = 0; i < n_days; i++) {
        temp = temperatures[i];
        while (s.size() && (temp > temperatures[s.top()])) {
            ans[s.top()] = i - s.top();
            s.pop();
        }
        s.push(i);
    }
return ans;
}
```

You can save some space by using `ans` itself as the stack. The intuition is that if we go through `temperatures` reversely, and if we already know `ans[i]`, then for day $i - 1$, instead of checking $i$, $i + 1$, ..., we only need to check `ans[i]`, `ans[ans[i]]`, etc. That is, if day $i$ is cooler than day $i - 1$, then all days between $i$ and `ans[i]` are also cooler than $i - 1$ by definition of `ans`, so we can jump to `ans[i]`. If `ans[i]` is still cooler, than we can jump to `ans[ans[i]]`.
