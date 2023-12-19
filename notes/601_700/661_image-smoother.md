# 661. Image Smoother

An **image smoother** is a filter of the size $3\times 3$ that can be applied to each cell of an image by rounding down the average of the cell and the eight surrounding cells (i.e., the average of the nine cells in the blue smoother). If one or more of the surrounding cells of a cell is not present, we do not consider it in the average (i.e., the average of the four cells in the red smoother).

![](https://assets.leetcode.com/uploads/2021/05/03/smoother-grid.jpg)

Given an $m\times n$ integer matrix `img` representing the grayscale of an image, return *the image after applying the smoother on each cell of it*.

**Example:**

> **Input:** `img = [[1,1,1],[1,0,1],[1,1,1]]`
> 
> **Output:** `[[0,0,0],[0,0,0],[0,0,0]]`
> 
> **Explanation:**
> 
> For the points $(0, 0)$, $(0, 2)$, $(2, 0)$, $(2, 2)$: `floor(3/4) = floor(0.75) = 0`
> 
> For the points $(0, 1)$, $(1, 0)$, $(1, 2)$, $(2, 1)$: `floor(5/6) = floor(0.83333333) = 0`
> 
> For the point $(1, 1)$: `floor(8/9) = floor(0.88888889) = 0`

**Constraints:**

* `m == img.length`
* `n == img[i].length`
* `1 <= m, n <= 200`
* `0 <= img[i][j] <= 255`


## Brute force + bit manipulation

Init. the answer matrix. Go through all cells, and calc. the avg. of the surrounding cells, and put the avg. into the answer mat. However, we don't really need to make another new answer matrix: we can save them "in-place". Every cell takes an integer value between 0 to 255, or $2^8$, and most programming languages save an integer as `int32`. That is, we have 32 bits available, and 0 to 255 only needs 8. We can save the smoothed value in another 8 bits, and that takes 16 bits, which still fits in `int32`. Therefore, we don't need any extra space.

E.g., look at the nine blue cells above. The original value is $7$, or 111 in binary, which in binary is:

$0\cdots 0\ 0\ 0\ 0\ 0\ 0\ 0\ 0\ 0\ \underline{0\ 0\ 0\ 0\ 0\ 1\ 1\ 1}$

The avg. of the surrounding cells is $(1 + 2 + 3 + 6 + 7 + 8 + 11 + 12 + 13)/9 = 9$, or 1001 in binary, and we can save it in the 9th to 16th places in this `int32`:

$0\cdots 0\ \underline{0\ 0\ 0\ 0\ 1\ 0\ 0\ 1}\ \underline{0\ 0\ 0\ 0\ 0\ 1\ 1\ 1}$

In other words, the original matrix is in the 1st to 8th places, and we compute, store, and return the surrounding avg. in the 9th to 16th places.

The remaining problem is, how to read and write numbers to the 1st to 8th, 9th to 16th places. To read, we can do bitwise multiplication:

$0\cdots 0\ \underline{1\ 0\ 0}\ 1\ 0\ 0\ 1\ 1$

bitwise times

$0\cdots 0\ \underline{1\ 1\ 1}\ 0\ 0\ 0\ 0\ 0$

will get us the 6th to 8th places.

To write, we can bitwise shift:

$0\cdots 0\ \underline{1\ 0\ 0\ 1}\ 0\ 0\ 1\ 1$

bitwise right shifted two places is

$0\cdots 0\ 0\ 0\ \underline{1\ 0\ 0\ 1}\ 1\ 1$.


## Sol.

```python
def imageSmoother(self, img: List[List[int]]) -> List[List[int]]:
    n_rows = len(img)
    n_cols = len(img[0])

    for i in range(n_rows):
        for j in range(n_cols):
            ssum = 0  # Avoid naming conflict with built-in `sum`
            cnt = 0

            for k in [-1, 0, 1]:
                for l in [-1, 0, 1]:
                    x = i + k
                    y = j + l
                    if (0 <= x < n_rows) and (0 <= y < n_cols):  # Border, i.e. red cells above
                        ssum += img[x][y] & 255  # need 1st to 8th places, 11111111 in binary = 
                        cnt += 1                 # 255 in decimal
            
            img[i][j] |= ssum // cnt << 8  # Bitwise left shift 8 places

    for i in range(n_rows):
        for j in range(n_cols):
            img[i][j] >>= 8  # Bitwise right shift 8 places to return smoothed values
    return img

```


