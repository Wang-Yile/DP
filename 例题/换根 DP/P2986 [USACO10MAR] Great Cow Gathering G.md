# [P2986 [USACO10MAR] Great Cow Gathering G](https://www.luogu.com.cn/problem/P2986)

## 题意

一棵树，每个点有权值 $a_i$ 和数量 $c_i$，选择一个节点，最小化其它点到它的 $a_i\times c_i$ 之和。

## 思路

假设每个点都到了节点 $1$，此时节点 $i$ 的贡献为 $w_i$，若选择的节点为 $x$，可以通过加上或减去 $1$ 到 $x$ 的距离完成动态规划，在 $O(n)$ 的时间内求解。

## 代码

```cpp
#include <iostream>
#include <vector>

#define int long long

using namespace std;

int n, sum;
int c[100005];
int cnt[100005];
int dis[100005];
int dp[100005];
vector<pair<int, int>> vec[100005];

static inline int dfs1(int u, int fa) {
    int count = 0;
    for (auto [v, w] : vec[u]) {
        if (v == fa)
            continue;
        int s = dfs1(v, u);
        dis[u] += dis[v] + w * s;
        count += s;
    }
    cnt[u] = c[u] + count;
    return cnt[u];
}

static inline void dfs2(int u, int fa) {
    for (auto [v, w] : vec[u]) {
        if (v == fa)
            continue;
        dp[v] = dp[u] - cnt[v] * w + (sum - cnt[v]) * w; // 先处理过去 / 回来的牛, 再向下递归
        dfs2(v, u);
    }
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n;
    for (int i = 1; i <= n; ++i) {
        cin >> c[i];
        sum += c[i];
    }
    for (int i = 1; i < n; ++i) {
        int u, v, w;
        cin >> u >> v >> w;
        vec[u].push_back(make_pair(v, w));
        vec[v].push_back(make_pair(u, w));
    }
    dfs1(1, 0);
    dfs2(1, 0);
    int ans = 1e9;
    for (int i = 1; i <= n; ++i) {
        ans = min(ans, dp[i]);
    }
    cout << ans + dis[1] << endl; // 答案要加上最初的距离
    return 0;
}
```
