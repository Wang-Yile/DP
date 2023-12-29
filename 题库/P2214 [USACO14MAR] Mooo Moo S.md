# [P2214 [USACO14MAR] Mooo Moo S](https://www.luogu.com.cn/problem/P2214)

## 题意

有 $n$ 个沿直线分布的牧场，每一个牧场可能有许多种品种的奶牛。

有 $b$ 个不同品种的奶牛，第 $i$ 种奶牛产生的音量为 $v_i$。

如果某个牧场的总音量是 $x$ ，那么它将传递 $x-1$ 的音量到右边的牧场。

给定每个牧场的总音量，求满足要求的最小奶牛数量，或者判断这不可能。

## 思路

先判断每个音量是否可达，如果可达，那么最少需要几头牛。

然后可以模拟得到答案。

## 代码

```cpp
#include <iostream>
#include <string.h>

using namespace std;

int n, b;
int a[105];
int v[25];
int dp[100005];

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P2144.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> b;
    for (int i = 1; i <= b; ++i) {
        cin >> v[i];
    }
    memset(dp, 0x3f, sizeof dp);
    dp[0] = 0;
    for (int i = 1; i <= b; ++i) {
        for (int j = v[i]; j <= 1e5; ++j) {
            dp[j] = min(dp[j], dp[j - v[i]] + 1);
        }
    }
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
    }
    int start = -1;
    for (int i = 1; i <= n; ++i) {
        if (a[i]) {
            start = i;
            break;
        }
    }
    if (start == -1) {
        cout << 0 << endl;
        return 0;
    }
    int ans = dp[a[start]];
    if (ans >= 1e9) {
        cout << -1 << endl;
        return 0;
    }
    int now = a[start];
    for (int i = start + 1; i <= n; ++i) {
        --now;
        if (a[i] < now) {
            cout << -1 << endl;
            return 0;
        }
        if (a[i] == now) {
            continue;
        }
        if (dp[a[i] - now] >= 1e9) {
            cout << -1 << endl;
            return 0;
        } else {
            ans += dp[a[i] - now];
            now = a[i];
        }
    }
    cout << ans << endl;
    return 0;
}
```