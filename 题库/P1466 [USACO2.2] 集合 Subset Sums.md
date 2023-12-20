# [P1466 [USACO2.2] 集合 Subset Sums](https://www.luogu.com.cn/problem/P1466)

## 题意

给定从 $1$ 到 $n$ 的 $n$ 个数，划分到两个集合，问有几种方式划分使得这两个集合的数字和相等。

## 思路

显然当 $1$ 到 $n$ 的数字和为奇数时没有分割方式。

设 $dp_j$ 表示前 $i$ 个数已被划分（被压缩），第一个集合数字和为 $j$ 时的分割方式，$tot$ 表示 $1$ 到 $n$ 的数字和。

## 预处理

第一个位置选与不选：$dp_{0}=dp{1}=1$。

## 转移

$dp_{j}=dp_{j}+\begin{cases}dp_{j-i}&j>i\\0\end{cases}$

## 答案

$dp_{\frac{tot}{2}}$

## 代码

```cpp
#include <iostream>

using namespace std;

int n;
int dp[800];

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n;
    int tot = (n + 1) * n / 2;
    if (tot & 1) {
        cout << 0 << endl;
        return 0;
    }
    dp[1] = dp[0] = 1;
    for (int i = 2; i <= n; ++i) {
        for (int j = tot; j >= 0; --j) {
            dp[j] = dp[j];
            if (j > i) {
                dp[j] += dp[j - i];
            }
        }
    }
    cout << dp[tot / 2] << endl;
    return 0;
}
```