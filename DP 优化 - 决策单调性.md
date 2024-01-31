# DP 优化 - 决策单调性

[P3515 [POI2011] Lightning Conductor](https://www.luogu.com.cn/problem/P3515)

> 题意：给定长为 $n$ 的序列 $\{a_n\}$，对于每一个 $1\le i\le n$，求出一个最小的非负整数 $p$，使得 $\forall 1\le j\le n,a_j\le a_i+p-\sqrt{|i-j|}$。$1\le n\le 5\times 10^5,0\le a_i\le 10^9$。

---

变换原式：

$$\begin{aligned}
&a_j\le a_i+p-\sqrt{|i-j|}\\
\Rightarrow&p\ge a_j-a_i+\sqrt{|i-j|}\\
\Rightarrow&p_{\min}=\max(a_j+\sqrt{|i-j|})-a_i\\
\Rightarrow&
\begin{cases}
p_{\min}=\max(a_j+\sqrt{i-j})&i>j\\
p_{\min}=\max(a_j+\sqrt{j-i})&i<j
\end{cases}-a_i
\end{aligned}$$

先考虑 $p_{\min}=\max(a_j+\sqrt{i-j})$：

根号函数的增长趋势随被开方数增大而逐渐减小，例如：对于函数 $f(x)=5+\sqrt{x+5},g(x)=6+\sqrt{x-1}$，在 $[1,7.25)$ 上 $f(x)>g(x)$，而在 $(7.25,+\infin)$ 上 $f(x)<g(x)$，这两个函数只有一个交点 $(7.25,8.5)$。因此如果把 $a_j+\sqrt{i-j}$ 看成关于 $j$ 的函数。

对于函数 $h(x)=a+\sqrt{x+b}$，函数形态由根号确定，起点横坐标（定义域下界）由 $-b$ 确定，高度（值域下界）由 $a$ 确定。

本题中的估价函数关于 $i$ 的函数 $w(i,j)=a_j+\sqrt{j-i}$，其中定义域下界为 $i$，高度下界为 $a_j$。

当定义域下界 $i$ 增大时，函数在同一自变量下的取值会减小（或者脱离定义域），因此，本题估价函数有单调性。

---

所以考虑维护一个二分栈。

记 $k_j$ 为栈中相邻两个函数 $w(i,j)$ 和 $w(i,j+1)$ 的交点 $i$，为了方便，向下取整。

对于队尾决策 $q_{qr}$，若 $w(k_{q_{qr-1}},q_{qr})<w(k_{q_{qr-1}},i)$，那么说明队尾不更优，弹出队尾。

此时二分计算 $k_{qr}$，再把关于 $x$ 的估价函数 $w(x,i)$ 加入队尾。

然后清除过期的队首决策，若 $k_{ql}\le i$，弹出队首。

因为上面只考虑了 $i>j$，那么把数组倒转再跑一遍就考虑了 $i<j$，注意结果要倒转输出，并对 $0$ 取 $\max$（这是 $i=j$ 的情况）。

时间复杂度 $O(n\log n)$。

```cpp
#include <iostream>
#include <math.h>
#include <string.h>

#define int long long

using namespace std;

int n;
int a[500005];
int q[500005], ql = 1, qr;
int k[500005];      // 存储临界值
double ans[500005]; // 如果用 int 会有精度问题

static inline double w(int i, int j) { // 估价函数
    // cout << "w: " << i << " " << j << " => " << i - j << " => " << sqrt(i - j) << endl;
    return a[j] + sqrt(i - j);
}
static inline int bound(int x, int y) { // 函数 x 与 y 的临界值
    int l = y, r = k[x] ? k[x] : n;
    int ret = r + 1;
    while (l <= r) {
        int mid = (l + r) >> 1;
        if (w(mid, x) <= w(mid, y)) {
            ret = mid;
            r = mid - 1;
        } else {
            l = mid + 1;
        }
    }
    return ret;
}
static inline void work() {
    ql = 1;
    qr = 0;
    for (int i = 1; i <= n; ++i) {
        while (ql < qr && w(k[qr - 1], q[qr]) < w(k[qr - 1], i)) {
            --qr;
        }
        k[qr] = bound(q[qr], i);
        q[++qr] = i;
        while (ql < qr && k[ql] <= i) {
            ++ql;
        }
        ans[i] = max(ans[i], w(i, q[ql]));
    }
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P3515.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
    }
    work();
    for (int i = 1, j = n; i < j; ++i, --j) {
        swap(a[i], a[j]), swap(ans[i], ans[j]);
    }
    memset(k, 0, sizeof k);
    work();
    for (int i = n; i; --i) {
        cout << max((int)(ceil(ans[i]) - a[i]), 0ll) << endl;
    }
    return 0;
}
```
