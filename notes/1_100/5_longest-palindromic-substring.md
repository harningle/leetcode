# 5. Longest Palindromic Substring

Given a string `s`, return *the longest palindromic substring*[^1] in `s`.

[^1]: A string is **palindromic** if **it reads the same forward and backward**.

**Example:**

> **Input:** `s = "azabcbdedbcbabb"`
> 
> **Output:** `"abcbdedbcba"`


## Brute force: expand from centres

For each letter, we look at substrings centring at it, and find the longest palindrome. E.g., at index $4$ of <code>\"azab<u>c</u>bdedbcbab\"</code>, we search for all palindrome centring at `c`. `"c"` itself of course is one. `"bcb"` is another one. `"abcbd"` is not, so we stop there. The longest palindrome centring at `c` has length of 3.

| $i$ | Letter at centre | Longest palindrome | Longest palindrome length |
|-----|------------------|--------------------|---------------------------|
| 0   | `a`                | `"a"`                  | 1                         |
| 1   | `z`                | `"aza"`                | 3                         |
| 2   | `a`                | `"a"`                  | 1                         |
| 3   | `b`                | `"b"`                  | 1                         |
| 4   | `c`                | `"bcb"`                | 3                         |
| 5   | `b`                | `"b"`                  | 1                         |
| 6   | `d`                | `"d"`                  | 1                         |
| 7   | `e`                | `"abcbdedbcba"`        | 11                        |
| 8   | `d`                | `"d"`                  | 1                         |
| 9   | `b`                | `"b"`                  | 1                         |
| 10  | `c`                | `"bcb"`                | 3                         |
| 11  | `b`                | `"b"`                  | 1                         |
| 12  | `a`                | `"bab"`                | 3                         |
| 13  | `b`                | `"b"`                | 1                         |
| 14  | `b`                | `"b"`                | 1                         |

The time complexity is at most

$$\underset{\text{nothing to check}}{\underbrace{0}} + \underset{\text{check “aza"}}{\underbrace{1}} + \underset{\text{check “zab", “azabc"}}{\underbrace{2}} + \underset{\text{check “abc", “zabcb", “azabcbd"}}{\underbrace{3}} + \cdots + \frac{n - 1}{2} + \cdots + 1 + 0 = O(n^2)$$

. Actually it can be much faster in some cases as we can early stop: if `"abc"` is not a palindrome, we don't need to check `"zabcb"` and `"azabcbd"` any more.


## Manacher's algo.: intuition and visual representation

We can reduce the #. of checks by exploiting "palindrome" inside "palindrome". E.g. <code>\"<u>bcb</u>\"</code> is a palindrome inside the palindrome <code>\"a<u>bcb</u>ded<b>bcb</b>a\"</code>. By symmetry, <code>\"<b>bcb</b>\"</code> must also be a palindrome of the same length. So for <code><b>c</b></code>, we don't need to check <code>\"<b>c</b>\"</code> or <code>\"<b>bcb</b>\"</code>, and can directly check <code>\"d<b>bcb</b>a\"</code> Such symmetry reduces the algo. to linear time. The high level steps are:

1. for each letter, find the longest palindrome centring at it, same as the above brute force
2. however, if the letter (<code><b>c</b></code>) is inside the the current palindrome, the brute force starts with length of its mirror (<code><u>c</u></code>) or the boundary of the current palindrome, instead of three
3. if we find a new palindrome above, update the current palindrome

To see it more clearly, suppose we are at index $i$ letter $s_i$.

* If the current palindrome doesn't cover $s_i$, then we just brute force. E.g., <code>s = \"<u>aba</u>x<font color="#000080"><b>y</b></font>z\"</code>, current palindrome is <code><u>aba</u></code>, $i = 4$, $s_i =$ <code><font color="#000080"><b>y</b></font></code>
* If the current palindrome covers $s_i$, then we know $s_i = s_{2c - i}$, where $c$ is the index of the centre of the current palindrome. Since we already know the longest palindrome centring at $s_{2c - i}$ before, let its length be $l_{2c - i}$, or radius $r_{2c - i} = \dfrac{l_{2c - i} - 1}{2}$
    - As $s_{2c - i}$ and $s_i$ are symmetric, if $i + r_{2c - i}$ is within the current palindrome, then $s_i$ must have a palindrome of length at least $r_{2c - i}$. E.g. <code>s = \"<u>xa<font color="#800000"><b>b</b></font>a<font color="#000080"><b>b</b></font>ax</u>yz\"</code>, current palindrome is <code><u>xa<font color="#800000"><b>b</b></font>a<font color="#000080"><b>b</b></font>ax</u></code>, $i = 4$, $s_i =$ <code><font color="#000080"><b>b</b></font></code>, $s_{2c - i} =$ <code><font color="#800000"><b>b</b></font></code>. Since <code><font color="#800000"><b>b</b></font></code> has a palindrome <code><u>a<font color="#800000"><b>b</b></font>a</u></code> of length 3, <code><font color="#000080"><b>b</b></font></code> has at least a palindrome <code><u>a<font color="#000080"><b>b</b></font>a</u></code> of length 3. The brute force starts at length 5 then
    - If $r_{2c - i}$ is so long that a palindrome of length $r_{2c - i}$ centring at $s_i$ goes beyond the current palindrome, we cannot say the same as we know nothing right to the current palindrome. But we can still guarantee that up until the end of the current palindrome it's a palindrome centring at $s_i$. E.g. <code>s = \"aa<u><font color="#800000"><b>a</b></font>aabaa<font color="#000090"><b>a</b></font></u>yz\"</code>, current palindrome is <code><u><font color="#800000"><b>a</b></font>aabaa<font color="#000090"><b>a</b></font></u></code>, $i = 8$, $s_i =$ <code><font color="#000080"><b>a</b></font></code>, $s_{2c - i} =$ <code><font color="#800000"><b>a</b></font></code>. <code><font color="#800000"><b>a</b></font></code> has a palindrome <code>aa<u><font color="#800000"><b>a</b></font>aa</u></code> of length 5. However, if we take the length-5 palindrome <code><u>aa<font color="#000080"><b>a</b></font></u>yz</code> at <code><font color="#000080"><b>a</b></font></code>, it exceeds the boundary of the current palindrome <code><u><font color="#800000"><b>a</b></font>aabaa<font color="#000090"><b>a</b></font></u></code>, so such length-5 palindrome may not be a palindrome. All we can say is that up until the boundary of the current palindrome, the palindrome centring at <code><font color="#000080"><b>a</b></font></code>, which in this case is itself, is a palindrome


## Manacher's algo.: what if not odd?

The above algo. only works if the length of palindrome is odd. E.g. `"aaaa"` will have a palindrome of length 3 following the above algo., but in fact it's 4. We propose the following transformation to make everything odd:

`"xyaaaauv"` $\mapsto$ `"|x|y|a|a|a|a|u|v|"`

so that palindromes centring at actual letters are `"a"`, `"|a|"`, `"a|a|a"`, ..., while palindromes centring at bogus letters are `"|"`, `"a|a"`, `"|a|a|"`, ... Therefore, both even and odd palindromes are captured.


## Solution

```c
#include <stdlib.h>
#include <string.h>

char* longestPalindrome(char* s) {
    // Insert bogus letters "|" to fix the odd/even thing
    int n_chars = strlen(s) * 2 + 1;
    char ss[n_chars + 1];  // +1 for '\0'
    ss[0] = '|';
    for (int i = 0; i < strlen(s); i++) {
        ss[2 * i + 1] = s[i];
        ss[2 * i + 2] = '|';
    }
    ss[n_chars] = '\0';
    
    // Manacher's algo.
    int* r = calloc(n_chars, sizeof(int)); // Radius of the longest palindrome
                                           // at each letter
    int current_r = 0;   // Radius of the current palindrome
    int c = 0;           // Centre of the current palindrome
    int j;               // Symmetric/mirror letter index
    int k;               // From which radius shall we start brute force
    int ans = 1;         // Longest palindrome length
    int l_idx = 0;       // Beginning index of the longest palindrome
    int r_idx = 0;       // Ending index of the longest palindrome

    for (int i = 0; i < n_chars; i++) {
        j = 2 * c - i;  // i's symmetric part/mirror in the current palindrome
        k = 0;  // If the current palindrome doesn't cover i, brute force from
                // scratch

        // If the current palindrome covers i
        if (i <= c + current_r) {
            // The brute force starts at the smaller of i's mirror's palindrome
            // length and the dist. to the boundary of the current palindrome
            if (j > 0) {  // Need to check if the mirror index >= 0
                if (r[j] < c + current_r - i) {
                    k = r[j];
                } else {
                    k = c + current_r - i;
                }
            }
        }

        // Brute force
        while ((i + k < n_chars) & (i - k >= 0)) {
            if (ss[i + k] == ss[i - k]) {
                k++;
            } else {
                break;
            }          
        }
        k--;  // Subtract the letter ifself
        r[i] = k;
        if (k > ans) {
            ans = k;
            l_idx = (i - k) / 2;  // /2 to excl. the bogus letters
            r_idx = l_idx + k - 1;
        }

        // Update the current palindrome if we find a "righter" one
        if (i + k > i + c) {
            current_r = k;
            c = i;
        }
    }

    // Return the longest palindrome
    char* palindrome = malloc((ans + 1) * sizeof(char));
    memcpy(palindrome, &s[l_idx], ans);
    palindrome[ans] = '\0';
    return palindrome;
}
```
