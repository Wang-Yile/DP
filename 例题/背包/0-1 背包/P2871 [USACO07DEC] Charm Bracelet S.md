# [P2871 [USACO07DEC] Charm Bracelet S](https://www.luogu.com.cn/problem/P2871)

## 题意

有 $n$ 件物品和一个容量为 $m$ 的背包。第 $i$ 件物品的重量是 $w_i$，价值是 $v_i$。将哪些物品装入背包可使这些物品的重量总和不超过背包容量，且价值总和最大。

## 思路

0-1 背包模板。

**显然枚举时循环是反向的。**

## 代码

```cpp
#include <iostream>
using namespace std;

int n, m;
int w[3500];
int d[3500];
int dp[12880];

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) {
        cin >> w[i] >> d[i];
    }
    for (int i = 1; i <= n; ++i) {        // 枚举顺序不能错
        for (int j = m; j >= w[i]; --j) { // 倒叙枚举不能错
            dp[j] = max(dp[j], dp[j - w[i]] + d[i]);
        }
    }
    cout << dp[m] << endl;
    return 0;
}
```
