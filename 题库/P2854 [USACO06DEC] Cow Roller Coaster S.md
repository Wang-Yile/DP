# [P2854 [USACO06DEC] Cow Roller Coaster S](https://www.luogu.com.cn/problem/P2854)

## 题意

给定 $n$ 条铁轨，每个铁轨从位置 $x_i$ 开始，长为 $w_i$，价值为 $f_i$，代价为 $c_i$。选择数条铁轨使其铺满区间 $[0,l]$，选出的铁轨不能重叠，代价不超过 $b$，问最大价值，或判断无解。

## 思路

本题有多个变数，全部设进状态。设 $dp_{i,j}$ 表示已铺满区间 $[0,i]$，代价为 $j$ 时的最大价值。

## 初始化

负无穷大。（后文阐述为什么）

$$dp_{0,0}=0$$

## 转移

每次转移必须从可行的情况转移，为了方便转移，定义不可行的状态为负无穷大。

首先应该对输入的铁轨排序，按左端点为关键字。

有转移方程

$$dp_{x_i+w_i,j+c_i}=\max(dp_{x_i+w_i,j+c_i},dp_{x_i,j}+f_i)$$

## 答案

$$\max\limits_{i=0}^{b}(dp_{l,i})$$

如果上式取到负数值，那么答案为无解。

## 代码

```cpp
#include <algorithm>
#include <iostream>
#include <string.h>

using namespace std;

int l, n, b;
struct node {
    int x, w, f, c;
    inline bool operator<(const node &nd) const {
        return x < nd.x;
    }
} a[10005];
int dp[1005][1005];

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P2854.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> l >> n >> b;
    memset(dp, 0xc0, sizeof dp);
    for (int i = 1; i <= n; ++i) {
        cin >> a[i].x >> a[i].w >> a[i].f >> a[i].c;
    }
    sort(a + 1, a + n + 1);
    dp[0][0] = 0;
    for (int i = 1; i <= n; ++i) {
        for (int j = 0; j <= b - a[i].c; ++j) {
            if (dp[a[i].x][j] != -1) {
                dp[a[i].x + a[i].w][j + a[i].c] = max(dp[a[i].x + a[i].w][j + a[i].c], dp[a[i].x][j] + a[i].f);
            }
        }
    }
    int ans = -1e9;
    for (int i = 0; i <= b; ++i) {
        ans = max(ans, dp[l][i]);
    }
    if (ans < 0) {
        cout << -1 << endl;
    } else {
        cout << ans << endl;
    }
    return 0;
}
```