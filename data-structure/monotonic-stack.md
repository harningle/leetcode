# Monotonic Stack

Generally speaking, a monotonic increasing stack is a stack whose elements are monotonically increasing from bottom to top.


## How to push an element to a monotonic increasing stack

If we want to push a new element to it, we have to pop all larger elements, if any, first, and then push it

1. check if the top element is larger than the new element. If yes, pop it
1. repeat Step 1., until we pop out everything, or when the top element is smaller than the new element
1. push the new element to the stack
    
```cpp
int new_element = 5;
while (!stack.empty() || stack.top() > new_element) {
    stack.pop()
}
stack.push(new_element);
```

**Strictly Increasing or Weakly Increasing.** Be careful about <code>stack.top() <font color="red"><b>></b></font> new_element</code> or <code>stack.top() <font color="red"><b>>=</b></font> new_element</code>. See the application below.


## Use monotonic increasing stack to track the previous/next smaller element in an array

A very common application of monotonic stack is to find the previous and next smaller element for all elements in an array in $O(n)$.

For example, given `arr = [5, 9, 2, 7, 3, 4]`, we want to find the previous and next smaller element index for all numbers in `arr`, which are `[-1, 0, -1, 2, 2, 4]` and `[2, 2, 6, 4, 6, 6]`, respectively. To brute force it, we usually need $O(n^2)$:

```cpp
vector<int> arr = {5, 9, 2, 7, 3, 4};
vector<int> prev_smaller(arr.size() - 1);
int smaller_element;
for (int i = 0; i < arr.size(); i++) {
    smaller_element = -1;
    for (int j = i - 1; j >= 0; j--) {
        if (arr[j] < arr[i]) {
            smaller_element = j;
            break;
        }
    }
    prev_smaller[i] = smaller_element;
}
```

However, many `if (arr[j] < arr[i])` comparisons are not necessary: we know the relative size of some numbers in previous comparisons, and we don't need to double-compare them when checking later numbers. Monotonic increasing stack can skip all those redundant comparisons:

| $i$ | `arr[i]` | `stack` before | `stack` ops.               | `stack` after | `prev_smaller` | `next_smaller`                                                                        |
|-----|--------|----------------|----------------------------|---------------|----------------|---------------------------------------------------------------------------------------|
| `0`   | 5      | `[]`             | Push $5$                   | `[0]`         | $-1$           |                                                                                       |
| `1`   | 9      | `[0]`          | Push $9$                   | `[0, 1]`      | $0$            |                                                                                       |
| `2`   | 2      | `[0, 1]`       | Pop $9$, pop $5$, push $2$ | `[2]`         | $-1$           | `next_smaller` for $9$ and $5$ are $1$, as we have to pop them out before pushing $2$ |
| `3`   | 7      | `[2]`          | Push $7$                   | `[2, 3]`      | $2$            |                                                                                       |
| `4`   | 3      | `[2, 3]`       | Pop $7$, push $3$          | `[2, 4]`      | $2$            | `next_smaller` for $7$ is $4$, as we have to pop $7$ before pushing $3$               |
| `5`   | 4      | `[2, 4]`       | Push $4$                   | `[2, 4, 5]`   | $4$            |                                                                                       |

```cpp
vector<int> arr = {5, 9, 2, 7, 3, 4};
vector<int> prev_smaller(arr.size() - 1);
vector<int> next_smaller(arr.size() - 1, arr.size());
stack<int> s;
int smaller_element;
for (int i = 0; i < arr.size(); i++) {
    while (!s.empty() && arr[s.top()] > arr[i]) {
        smaller_element = s.top();
        next_smaller[smaller_element] = i;
        s.pop();
    }
    prev_smaller[i] = (s.empty()) ? -1 : s.top();
    s.push(i);
}
```
