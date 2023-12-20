# [P1507 NASA的食物计划](https://www.luogu.com.cn/problem/P1507)

## 题意

二维费用 DP 模板。

## 思路

同一维费用 DP，只是多开一维记录费用。

## 代码

```cpp
#include <iostream>

using namespace std;

int h, t, n;
struct node {
    int h, t, k;
} a[405];
int dp[20005][20005];

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> h >> t >> n;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i].h >> a[i].t >> a[i].k;
    }
    for (int i = 1; i <= n; ++i) {
        for (int j = h; j >= a[i].h; --j) {
            for (int k = t; k >= a[i].t; --k) {
                dp[j][k] = max(dp[j][k], dp[j - a[i].h][k - a[i].t] + a[i].k);
            }
        }
    }
    cout << dp[h][t] << endl;
    return 0;
}
```