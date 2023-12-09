# [P3478 [POI2008] STA-Station](https://www.luogu.com.cn/problem/P3478)

## 题意

求一个结点，使得以这个结点为根时，所有结点的深度之和最大。

## 思路

设 $dp_u$ 表示以 $u$ 为根时的答案，$siz_u$ 表示以 $1$ 为根时，$u$ 的子树的节点数。

## 预处理

预处理 $dp_1$，可以通过递归 $O(n)$ 求出。

## 转移

若 $u$ 的父亲为 $fa$，$dp_u=dp_{fa}+siz_1-2\times siz_u$。

## 答案

$dp_u$ 最大时，答案为 $u$。

## 代码

```cpp
#include <iostream>
#include <vector>

#define int long long

using namespace std;

int n, maxv = -1, ans = -1;
vector<int> vec[1000005];
int dep[1000005];
int siz[1000005];
int tot[1000005];
int dp[1000005];

static inline void dfs(int u, int fa) {
    dep[u] = dep[fa] + 1;
    siz[u] = 1;
    tot[u] = dep[u];
    for (auto v : vec[u]) {
        if (v == fa) {
            continue;
        }
        dfs(v, u);
        siz[u] += siz[v];
        tot[u] += tot[v];
    }
}
static inline void dfs2(int u, int fa) {
    if (u != 1) {
        dp[u] = dp[fa] + siz[1] - siz[u] * 2;
    }
    if (dp[u] > maxv) {
        maxv = dp[u];
        ans = u;
    }
    for (auto v : vec[u]) {
        if (v == fa) {
            continue;
        }
        dfs2(v, u);
    }
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P3478.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n;
    for (int i = 1; i < n; ++i) {
        int u, v;
        cin >> u >> v;
        vec[u].push_back(v);
        vec[v].push_back(u);
    }
    dfs(1, 0);
    dp[1] = tot[1];
    dfs2(1, 0);
    cout << ans << endl;
    return 0;
}
```
