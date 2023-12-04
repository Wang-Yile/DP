# 树的直径 DP

## 思路

设当 $1$ 为根时，每个节点向下所能延伸的最长路径长度 $d_1$ 与次长路径（与最长路径无公共边）长度 $d_2$，那么树的直径就是对于每一个点，该点 $d_1$ + $d_2$ 能取到的值中的最大值。

树形 DP 可以在存在负权边的情况下求解出树的直径（也可以边权加定值，用 4 次 DFS，两次求直径、两次求节点数）。

## 代码

```cpp
#include <iostream>
#include <vector>

using namespace std;

int n, d = 0;
int d1[10005], d2[10005];
vector<int> vec[10005];

static inline void dfs(int u, int fa) {
    d1[u] = d2[u] = 0;
    for (int v : vec[u]) {
        if (v == fa)
            continue;
        dfs(v, u);
        int t = d1[v] + 1;
        if (t > d1[u])
            d2[u] = d1[u], d1[u] = t;
        else if (t > d2[u])
            d2[u] = t;
    }
    d = max(d, d1[u] + d2[u]);
}

signed main() {
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
    cout << d << endl;
    return 0;
}
```

## 参考

[树形 DP 求树的直径 - OI-Wiki](https://oi-wiki.org/graph/tree-diameter/#%E5%81%9A%E6%B3%95-2-%E6%A0%91%E5%BD%A2-dp)