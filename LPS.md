# LPS - 最长回文子序列 _[1]_

## 概念

LPS，最长回文子序列。

以下默认原数列为 $a$，其长度为 $n$。如果不特别提到，“分界点”等词语指的是数列下标。

## 求解

### 朴素 DP

因为“回文”指的是前后一致，所以可以考虑枚举前后段的分界点 $k$，把数列分为两个闭区间 $A,B$。

如图所示：

```
a: 1 2 3 ... k ... n-2 n-1 n
A: |---------|
B:           |-------------|
```

把 $B$ 串反转得到 $B'$ 串，在 $A,B'$ 两串上跑 LCS。得到的最大的 LCS 长度乘二即为所求。

**注意：需要记录 LCS 的路径，如果 $A,B$ 的 LCS 的尾部都是 $k$，那么答案需要减一。**

朴素 DP 的时间复杂度是 $O(n^3)$。

### 优化暴力

当分界点 $k$ 越趋向于中心时 LCS 可能越长，那么先枚举中心，设当前最大的 LCS 长度为 $L$。

如果在枚举中心左侧分界点 $k'$ 时，发现此时 LCS 长度小于等于 $L$，那么就无需枚举更左侧的点；右侧同理。

一种可能的实现是从 $1$ 到 $n$ 枚举数 $i$，则 $k=\begin{cases}\lfloor\frac{n+1}{2}\rfloor-\frac{i-1}{2}&i\small\texttt{为奇数}\\\lfloor\frac{n+1}{2}\rfloor+\frac{i}{2}&i\small\texttt{为偶数}\end{cases}$。

本算法的最优时间复杂度是 $O(n^2)$，期望时间复杂度是 $???$，最坏时间复杂度是 $O(n^3)$。

处理 $k$ 的代码如下：

```cpp
for (int i = 1; i <= n; ++i) {
    int k;
    if (i & 1) {
        k = ((n + 1) >> 1) - (i >> 1);
    } else {
        k = ((n + 1) >> 1) + (i >> 1);
    }
    cout << k << endl;
}
```

### 转化问题

回文正反相同，可以考虑反转整个数列 $a$ 得到 $a'$，只需计算 $a,a'$ 的 LCS 即可。

本算法的时间复杂度是 $O(n^2)$。

代码如下：

```cpp
#include <iostream>

using namespace std;

int n;
int a[10005];
int b[10005];
int dp[10005][10005];

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        b[n - i + 1] = a[i];
    }
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= n; ++j) {
            dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            if (a[i] == b[j]) {
                dp[i][j] = max(dp[i][j], dp[i - 1][j - 1] + 1);
            }
        }
    }
    cout << dp[n][n] << endl;
    return 0;
}
```

## 例题

[P3815 [FJOI2017] 回文子序列问题](https://www.luogu.com.cn/problem/P3815)

> 题意：对于给定的两个序列，求它们的**最长公共回文子序列**的长度。$1\le T\le 10,1\le n\le 500$。

[Palindromic Parentheses](https://www.luogu.com.cn/problem/CF1906L)

> 题意：构造一个长度为 $n$ **合法**括号串使得这个串的 LPS 长度为 $k$，或说明无解。$2\le n\le 2000,1\le k\le n$。

The 2023 ICPC Asia Jakarta Regional Contest Problem L. [Problem details on CodeForces.](https://codeforces.com/contest/1906/problem/L)

这里写得比较简单，具体思路可以看我写的题解。

考虑串 `(((...()...)))` 的 LPS 长度为 $\dfrac{n}{2}$；串 `()(((...()...)))` 的 LPS 长度为 $\dfrac{n}{2}+1$；串 `()(((...()/...)))()` 的 LPS 长度为 $\dfrac{n}{2}+2$；……；串 `()()()...()()()` 的 LPS 长度为 $n-1$。

而 $k<\dfrac{n}{2}$ 是无解的，因为一个合法的括号串里 `(` 的个数有 $\dfrac{n}{2}$ 个，这些 `(` 构成一个回文子序列，因此 $k<\dfrac{n}{2}$ 不会成为 LPS，故无解。

$k=n$ 也是无解的，因为一个合法的括号串开头一定是 `(`，结尾一定是 `)`，因此不存在长度为 $n$ 的回文序列。

如果有解，只需保留一个长度为 $2\cdot(n-k)$ 的形如 `(((...()...)))` 的串，剩余的空位由串 `()` 尽量均匀地填补在两侧即可。_[2]_

```cpp
#include <iostream>

using namespace std;

int n, k;

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> k;
    if (k < n / 2 || k == n) {
        cout << -1 << endl;
        return 0;
    }
    int extra = n - 2 * (n - k); // 剩余的空位长度
    int cur = 0;                 // 已经输出的串长度
    for (int i = 1; i <= extra / 4; ++i) {
        cout << "()";
        cur += 2;
    }
    for (int i = 1; i <= n - k; ++i) {
        cout << "(";
        ++cur;
    }
    for (int i = 1; i <= n - k; ++i) {
        cout << ")";
        ++cur;
    }
    while (cur + 2 <= n) {
        cout << "()";
        cur += 2;
    }
    cout << endl;
    return 0;
}
```

## 参考文献

[1] Cormen, T. H. & Leiserson C. E. & Rivest R. L. & Stein C. ; _Introduction to Algorithms, Third Edition_ [M]. 殷建平等译. 北京：机械工业出版社，2013：231.

[2] Sabili, A. F. et al. ; [_Problem Analysis of The 2023 ICPC Asia Jakarta Regional Contest_](https://competition.binus.ac.id/icpc2023/problem_analysis.pdf) [DB/OL]. (2023-12)[2024-01-14].
