# 第 $k$ 小算法总结

不保证在除了 VSCode 的其它平台上有好的浏览体验。

## 综述

### 静态整体 Kth

**排序**

sort 一遍即可。

时间复杂度 $O(n\log n)$，空间复杂度 $O(n)$。

### 动态整体 Kth

**权值线段树**

离散化后开一棵权值线段树，每个位置的值表示这个位置对应的数（离散化后的）有多少个，向上维护和；

查询时先查询左子树个数和 $sum$，比较 $k$ 和 $sum$ 的大小：若 $k\le sum$ 则说明第 $k$ 小数在左子树中，递归查询左子树；否则，第 $k$ 小数在右子树中，第 $k-sum$ 小的数，$k-=sum$，递归查询右子树。

时间复杂度 $O(n\log n)$，空间复杂度 $O(n\log n)$。

### 静态区间 Kth

**前缀和**，**主席树**

对每个点开一棵权值线段树，记录这个点之前出现的数的个数，那么任意一段区间的数的个数均可以表示成为两棵权值线段树作差，即右端点位置的线段树减去左端点左边位置上的线段树，查询同动态整体 Kth。

### 动态区间 Kth

**前缀和**，**树状数组**，**主席树**

仍然想办法维护前缀和，考虑用树状数组维护。

修改时，可以只修改 $\log n$ 个位置。

查询时，将 $\log n$ 棵线段树作差即可。

### 图上路径 Kth

**A\***

### 高维距离 Kth

**K-D Tree**

### 更多神奇的 Kth 在拓展部分

## 例题

### 静态整体 Kth

[P1138 第 k 小整数](https://www.luogu.com.cn/problem/P1138)  
[P1923 【深基9.例4】求第 k 小的数](https://www.luogu.com.cn/problem/P1923)

代码（P1138）

```cpp
#include <algorithm>
#include <iostream>

using namespace std;

int n, k;
int a[10005];

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> k;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
    }
    sort(a + 1, a + n + 1);
    int tot = unique(a + 1, a + n + 1) - a - 1;
    if (k > tot) {
        cout << "NO RESULT" << endl;
    } else {
        cout << a[k] << endl;
    }
    return 0;
}
```

### 动态整体 Kth

没找到例题。

### 静态区间 Kth

[P3834 【模板】可持久化线段树 2](https://www.luogu.com.cn/problem/P3834)

代码

```cpp
#include <algorithm>
#include <iostream>

using namespace std;

int n, q;
int a[200005];
int b[200005];
int rt[200005];
struct node {
    int ls, rs, val;
} d[10000005];
int cnt;
static inline int clone(int p) {
    d[++cnt] = d[p];
    return cnt;
}
static inline int build(int s, int t) {
    int p = ++cnt;
    d[p].val = 0;
    if (s == t) {
        return p;
    }
    int mid = (s + t) >> 1;
    d[p].ls = build(s, mid);
    d[p].rs = build(mid + 1, t);
    return p;
}
static inline int insert(int x, int s, int t, int p) {
    p = clone(p);
    ++d[p].val;
    if (s == t) {
        return p;
    }
    int mid = (s + t) >> 1;
    if (x <= mid) {
        d[p].ls = insert(x, s, mid, d[p].ls);
    } else {
        d[p].rs = insert(x, mid + 1, t, d[p].rs);
    }
    return p;
}
static inline int query(int u, int v, int s, int t, int k) {
    if (s == t) {
        return s;
    }
    int x = d[d[v].ls].val - d[d[u].ls].val;
    int mid = (s + t) >> 1;
    if (k > x) {
        return query(d[u].rs, d[v].rs, mid + 1, t, k - x);
    } else {
        return query(d[u].ls, d[v].ls, s, mid, k);
    }
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P3834.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> q;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        b[i] = a[i];
    }
    sort(b + 1, b + n + 1);
    int m = unique(b + 1, b + n + 1) - b - 1;
    rt[0] = build(1, m);
    for (int i = 1; i <= n; ++i) {
        int t = lower_bound(b + 1, b + m + 1, a[i]) - b;
        rt[i] = insert(t, 1, m, rt[i - 1]);
    }
    while (q--) {
        int l, r, k;
        cin >> l >> r >> k;
        cout << b[query(rt[l - 1], rt[r], 1, m, k)] << endl;
    }
    return 0;
}
```

### 动态区间 Kth

[P2617 Dynamic Rankings](https://www.luogu.com.cn/problem/P2617)

代码

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

using namespace std;

int n, m, len;
int a[100005];

// 询问离线
struct question {
    char ch;
    int x, y, k;
} questions[100005];

// 离散化
int b[200005], tail;

// 主席树
int rt[100005];
struct node {
    int ls, rs, val;
} d[40000005];
int cnt;
static inline int clone(int p) {
    d[++cnt] = d[p];
    return cnt;
}
static inline int insert(int x, int s, int t, int p) {
    p = clone(p);
    if (s == t) {
        ++d[p].val;
        return p;
    }
    int mid = (s + t) >> 1;
    if (x <= mid) {
        d[p].ls = insert(x, s, mid, d[p].ls);
    } else {
        d[p].rs = insert(x, mid + 1, t, d[p].rs);
    }
    d[p].val = d[d[p].ls].val + d[d[p].rs].val;
    return p;
}
static inline int erase(int x, int s, int t, int p) {
    p = clone(p);
    if (s == t) {
        --d[p].val;
        return p;
    }
    int mid = (s + t) >> 1;
    if (x <= mid) {
        d[p].ls = erase(x, s, mid, d[p].ls);
    } else {
        d[p].rs = erase(x, mid + 1, t, d[p].rs);
    }
    d[p].val = d[d[p].ls].val + d[d[p].rs].val;
    return p;
}
vector<int> root1, root2;
static inline int difference() {
    int ret = 0;
    for (auto x : root1) {
        ret += d[d[x].ls].val;
    }
    for (auto x : root2) {
        ret -= d[d[x].ls].val;
    }
    return ret;
}
static inline void jump_to_left() {
    for (auto &x : root1) {
        x = d[x].ls;
    }
    for (auto &x : root2) {
        x = d[x].ls;
    }
}
static inline void jump_to_right() {
    for (auto &x : root1) {
        x = d[x].rs;
    }
    for (auto &x : root2) {
        x = d[x].rs;
    }
}
static inline int query(int s, int t, int k) {
    if (s == t) {
        return s;
    }
    int x = difference();
    int mid = (s + t) >> 1;
    if (k <= x) {
        jump_to_left();
        return query(s, mid, k);
    } else {
        jump_to_right();
        return query(mid + 1, t, k - x);
    }
}

// 树状树组
static inline int lowbit(int x) {
    return x & (-x);
}
static inline void BIT_insert(int x) {
    int k = lower_bound(b + 1, b + len + 1, a[x]) - b;
    while (x <= n) {
        rt[x] = insert(k, 1, len, rt[x]);
        x += lowbit(x);
    }
}
static inline void BIT_erase(int x) {
    int k = lower_bound(b + 1, b + len + 1, a[x]) - b;
    while (x <= n) {
        rt[x] = erase(k, 1, len, rt[x]);
        x += lowbit(x);
    }
}
static inline int BIT_query(int l, int r, int k) {
    root1.clear();
    root2.clear();
    for (int i = r; i; i -= lowbit(i)) {
        root1.push_back(rt[i]);
    }
    for (int i = l - 1; i; i -= lowbit(i)) {
        root2.push_back(rt[i]);
    }
    return query(1, len, k);
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P2617.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        b[++tail] = a[i];
    }
    for (int i = 1; i <= m; ++i) {
        cin >> questions[i].ch;
        if (questions[i].ch == 'Q') {
            cin >> questions[i].x >> questions[i].y >> questions[i].k;
        } else {
            cin >> questions[i].x >> questions[i].y;
            b[++tail] = questions[i].y;
        }
    }
    sort(b + 1, b + tail + 1);
    len = unique(b + 1, b + tail + 1) - b - 1;
    for (int i = 1; i <= n; ++i) {
        BIT_insert(i);
    }
    for (int i = 1; i <= m; ++i) {
        auto [ch, x, y, k] = questions[i];
        if (ch == 'Q') {
            cout << b[BIT_query(x, y, k)] << endl;
        } else {
            BIT_erase(x);
            a[x] = y;
            BIT_insert(x);
        }
    }
    return 0;
}
```

### 图上路径 Kth

[P2483 【模板】k 短路 / [SDOI2010] 魔法猪学院](https://www.luogu.com.cn/problem/P2483)

### 高维距离 Kth

[P4357 [CQOI2016] K 远点对](https://www.luogu.com.cn/problem/P4357)

## 拓展

这里都是紫以上的究极 Kth。

[P4278 带插入区间K小值](https://www.luogu.com.cn/problem/P4278)

> 题意：插入；修改；区间 $k$ 小值查询。

[P4119 [Ynoi2018] 未来日记](https://www.luogu.com.cn/problem/P4119)

> 题意：区间 $x$ 变成 $y$；区间第 $k$ 小。$1\le n,m,a_i\le10^5$。

[P3332 [ZJOI2013] K大数查询](https://www.luogu.com.cn/problem/P3332)

> 题意：维护 $n$ 个可重集合，将 $c$ 加入区间 $[l,r]$ 的所有集合；查询区间 $[l,r]$ 的集合的并集的第 $k$ 大。

[P4467 [SCOI2007] k短路](https://www.luogu.com.cn/problem/P4467)

> 题意：求 $a$ 到 $b$ 的 $k$ 短路。（长度相同按经过节点的字典序排序）

[P2093 [国家集训队] JZPFAR](https://www.luogu.com.cn/problem/P2093)

> 题意：求平面上到点 $(px,py)$ 第 $k$ 远的点的编号。（相同距离下，编号小的点距离被认为更大）

[P4479 [BJWC2018] 第k大斜率](https://www.luogu.com.cn/problem/P4479)

> 题意：平面上 $n$ 个点两两连成一条直线，问这些直线斜率的第 $k$ 大值。

[P4153 [WC2015] k 小割](https://www.luogu.com.cn/problem/P4153)

> 题意：看不懂。
