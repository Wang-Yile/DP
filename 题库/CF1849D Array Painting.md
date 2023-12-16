# [CF1849D Array Painting](https://www.luogu.com.cn/problem/CF1849D)

## 题意

有一个长为 $n$ 的序列 $a_n$，$\forall a_i\in\{0,1,2\}$，初始均未染色。

有两种操作：

1. 用 $1$ 的代价，染一个颜色；
2. 选择一个 $a_i\ne0$ 的已染色元素和一个与其相邻的未染色元素，将 $a_i-1$，然后将未染色元素染色。

问全部染色的最小代价。

## 思路

把所有对答案有影响的量设进状态：设 $dp_{i,j}$ 表示前 $i$ 个元素均已染色，第 $i$ 个元素的 $a_i=j$ 时的最小代价。

设 $f_i$ 表示从 $i$ 往前第一个 $0$ 的下标。

## 初始化

全部赋正无穷大。

$dp_{0,0}=dp_{0,1}=dp_{0,2}=0$

$dp_{1,a_1}=1$

## 转移

考虑操作一，有 $dp_{i,a_i}=dp_{i-1,0}+1$。

考虑操作二，有 $dp_{i,a_i}=\min(dp_{i-1,1},dp_{i-1,2})$。

再考虑当前 $a_i\ne0$，向前找到第一个 $a_{f_i}=0$，$dp_{i,a_i-1}=\min(dp_{pre_i-1,0},dp_{pre_i-1,1},dp_{pre_i-1,2})+1$，相当于之前的一段染色整体前移一位。

## 答案

$\min(dp_{n,0},dp_{n,1},dp_{n,2})$

## 代码

```cpp
#include <iostream>
#include <string.h>

using namespace std;

int n;
int a[200005];
int f[200005];
int dp[200005][3];

signed main() {
#ifndef ONLINE_JUDGE
    freopen("CF1849D.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n;
    memset(dp, 0x3f3f3f3f, sizeof dp);
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        if (a[i]) {
            f[i] = f[i - 1];
        } else {
            f[i] = i;
        }
    }
    dp[0][0] = dp[0][1] = dp[0][2] = 0;
    dp[1][a[1]] = 1;
    for (int i = 2; i <= n; ++i) {
        dp[i][a[i]] = min(dp[i - 1][0] + 1, min(dp[i - 1][1], dp[i - 1][2]));
        if (a[i]) {
            dp[i][a[i] - 1] = min(dp[f[i] - 1][0], min(dp[f[i] - 1][1], dp[f[i] - 1][2])) + 1;
        }
    }
    cout << min(dp[n][0], min(dp[n][1], dp[n][2])) << endl;
    return 0;
}
```
