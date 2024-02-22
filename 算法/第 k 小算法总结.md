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

[P3157 [CQOI2011] 动态逆序对](https://www.luogu.com.cn/problem/P3157?contestId=149615)

> 题意：给定一个排列，维护每次删除元素前的逆序对个数。

代码

```cpp
#include <iostream>
#include <vector>

using namespace std;

int n, m;
int a[100005];
int b[100005];

int rt[100005];
struct node {
    int ls, rs, val;
} d[40000005];
int cnt;
static inline int clone(int p) {
    d[++cnt] = d[p];
    return cnt;
}
static inline int insert(int x, int s, int t, int val, int p) {
    p = clone(p);
    if (s == t) {
        d[p].val += val;
        return p;
    }
    int mid = (s + t) >> 1;
    if (x <= mid) {
        d[p].ls = insert(x, s, mid, val, d[p].ls);
    } else {
        d[p].rs = insert(x, mid + 1, t, val, d[p].rs);
    }
    d[p].val = d[d[p].ls].val + d[d[p].rs].val;
    return p;
}
vector<int> rt1, rt2;
static inline int query(int x, bool op) {
    int s = 1;
    int t = n;
    int ret = 0;
    while (s != t) {
        int mid = (s + t) >> 1;
        if (x <= mid) {
            if (!op) {
                for (auto x : rt1) {
                    ret += d[d[x].rs].val;
                }
                for (auto x : rt2) {
                    ret -= d[d[x].rs].val;
                }
            }
            for (auto &x : rt1) {
                x = d[x].ls;
            }
            for (auto &x : rt2) {
                x = d[x].ls;
            }
            t = mid;
        } else {
            if (op) {
                for (auto x : rt1) {
                    ret += d[d[x].ls].val;
                }
                for (auto x : rt2) {
                    ret -= d[d[x].ls].val;
                }
            }
            for (auto &x : rt1) {
                x = d[x].rs;
            }
            for (auto &x : rt2) {
                x = d[x].rs;
            }
            s = mid + 1;
        }
    }
    return ret;
}

static inline int lowbit(int x) {
    return x & (-x);
}
static inline void BIT_insert(int x, int val) {
    while (x <= n) {
        rt[x] = insert(val, 1, n, 1, rt[x]);
        x += lowbit(x);
    }
}
static inline void BIT_erase(int x, int val) {
    while (x <= n) {
        rt[x] = insert(val, 1, n, -1, rt[x]);
        x += lowbit(x);
    }
}
static inline int BIT_query(int l, int r, int k, bool op) {
    rt1.clear();
    rt2.clear();
    for (int i = r; i; i -= lowbit(i)) {
        rt1.push_back(rt[i]);
    }
    for (int i = l - 1; i; i -= lowbit(i)) {
        rt2.push_back(rt[i]);
    }
    return query(k, op);
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P3157.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> m;
    long long ans = 0;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        b[a[i]] = i;
        ans += BIT_query(1, i - 1, a[i], 0);
        BIT_insert(i, a[i]);
    }
    while (m--) {
        cout << ans << endl;
        int x;
        cin >> x;
        ans -= BIT_query(1, b[x] - 1, x, 0);
        ans -= BIT_query(b[x] + 1, n, x, 1);
        BIT_erase(b[x], x);
    }
    return 0;
}
```

[P4278 带插入区间K小值](https://www.luogu.com.cn/problem/P4278)

> 题意：插入；修改；区间 $k$ 小值查询。

[P4119 [Ynoi2018] 未来日记](https://www.luogu.com.cn/problem/P4119)

> 题意：区间 $x$ 变成 $y$；区间第 $k$ 小。$1\le n,m,a_i\le10^5$。

[P3332 [ZJOI2013] K大数查询](https://www.luogu.com.cn/problem/P3332)

> 题意：维护 $n$ 个可重集合，将 $c$ 加入区间 $[l,r]$ 的所有集合；查询区间 $[l,r]$ 的集合的并集的第 $k$ 大。

**整体二分**

整体二分即同时处理多组询问，减少二分次数。

对于本题，统称询问与修改为操作，设当前处理的操作区间为 $[ql,qr]$，答案区间为 $[l,r]$。

本题的修改操作会影响后面的询问，因此操作的相对顺序不能变。（整体二分类似于归并排序，因此相对顺序是不变的）

1. 如果 $l=r$，那么操作 $[ql,qr]$ 的答案可以确定均为 $l$。

2. 否则，记当前答案区间的中点 $mid=\dfrac{l+r}{2}$，维护一个初始为空的区间线段树：

    - 对于一个修改操作 $(l,r,c)$，如果 $c>mid$，则该操作对答案区间 $(mid,r]$ 有贡献，并将区间线段树的 $[q_i.l,q_i.r]$ 加 $1$；反之，操作对答案区间 $[l,mid]$ 有贡献。

    - 对于一个查询操作 $(l,r,c)$，如果区间线段树的 $[q_i.l,q_i.r]$ 之和大于等于 $q_i.c$，那么这个查询操作的答案的在答案区间 $(mid,r]$；反之，答案在答案区间 $[l,mid]$。

    - 其它操作同普通整体二分。

代码

```cpp
#include <iostream>

#define int long long

using namespace std;

int n, m;
struct question {
    int opt, l, r, c, id;
} qs[50005], q1[50005], q2[50005];
int qs_cnt;

int ans[50005];

int d[200005];
int tag[200005];
bool reset[200005];
static inline void pushdown(int s, int t, int p) {
    if (reset[p]) {
        tag[p << 1] = tag[p << 1 | 1] = d[p << 1] = d[p << 1 | 1] = 0;
        reset[p << 1] = reset[p << 1 | 1] = 1;
        reset[p] = false;
    }
    if (tag[p]) {
        int mid = (s + t) >> 1;
        d[p << 1] += (mid - s + 1) * tag[p];
        tag[p << 1] += tag[p];
        d[p << 1 | 1] += (t - mid) * tag[p];
        tag[p << 1 | 1] += tag[p];
        tag[p] = 0;
    }
}
static inline void update(int l, int r, int s, int t, int c, int p) {
    if (l <= s && r >= t) {
        d[p] += (t - s + 1) * c;
        tag[p] += c;
        return;
    }
    int mid = (s + t) >> 1;
    pushdown(s, t, p);
    if (l <= mid) {
        update(l, r, s, mid, c, p << 1);
    }
    if (r > mid) {
        update(l, r, mid + 1, t, c, p << 1 | 1);
    }
    d[p] = d[p << 1] + d[p << 1 | 1];
}
static inline int query(int l, int r, int s, int t, int p) {
    if (l <= s && r >= t) {
        return d[p];
    }
    int mid = (s + t) >> 1;
    int ret = 0;
    pushdown(s, t, p);
    if (l <= mid) {
        ret += query(l, r, s, mid, p << 1);
    }
    if (r > mid) {
        ret += query(l, r, mid + 1, t, p << 1 | 1);
    }
    d[p] = d[p << 1] + d[p << 1 | 1];
    return ret;
}

static inline void solve(int ql, int qr, int l, int r) {
    if (l == r) {
        for (int i = ql; i <= qr; ++i) {
            if (qs[i].opt == 2) {
                ans[qs[i].id] = l;
            }
        }
        return;
    }
    int mid = (l + r) >> 1;
    int t1 = 0;
    int t2 = 0;
    tag[1] = 0;
    d[1] = 0;
    reset[1] = true;
    for (int i = ql; i <= qr; ++i) {
        if (qs[i].opt == 1) {
            if (qs[i].c > mid) {
                update(qs[i].l, qs[i].r, 1, n, 1, 1);
                q2[++t2] = qs[i];
            } else {
                q1[++t1] = qs[i];
            }
        } else {
            int val = query(qs[i].l, qs[i].r, 1, n, 1);
            if (qs[i].c > val) {
                qs[i].c -= val;
                q1[++t1] = qs[i];
            } else {
                q2[++t2] = qs[i];
            }
        }
    }
    for (int i = 1; i <= t1; ++i) {
        qs[ql + i - 1] = q1[i];
    }
    for (int i = 1; i <= t2; ++i) {
        qs[ql + t1 + i - 1] = q2[i];
    }
    solve(ql, ql + t1 - 1, l, mid);
    solve(ql + t1, qr, mid + 1, r);
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P3332.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> m;
    for (int i = 1; i <= m; ++i) {
        cin >> qs[i].opt >> qs[i].l >> qs[i].r >> qs[i].c;
        if (qs[i].opt == 2) {
            qs[i].id = ++qs_cnt;
        }
    }
    solve(1, m, -n, n);
    for (int i = 1; i <= qs_cnt; ++i) {
        cout << ans[i] << endl;
    }
    return 0;
}
```

本题也可以树套树，但我没看懂。

[P3527 [POI2011] MET-Meteors](https://www.luogu.com.cn/problem/P3527)

> 题意：有一个长为 $m$ 的环，被染成 $n$ 种颜色。有 $k$ 条线段价值为 $a_i$，问对于每一种颜色，第几条线段覆盖后可以得到总价值不少于 $p_i$。

[P4467 [SCOI2007] k短路](https://www.luogu.com.cn/problem/P4467)

> 题意：求 $a$ 到 $b$ 的 $k$ 短路。（长度相同按经过节点的字典序排序）

[P2093 [国家集训队] JZPFAR](https://www.luogu.com.cn/problem/P2093)

> 题意：求平面上到点 $(px,py)$ 第 $k$ 远的点的编号。（相同距离下，编号小的点距离被认为更大）

[P4479 [BJWC2018] 第k大斜率](https://www.luogu.com.cn/problem/P4479)

> 题意：平面上 $n$ 个点两两连成一条直线，问这些直线斜率的第 $k$ 大值。

[P4153 [WC2015] k 小割](https://www.luogu.com.cn/problem/P4153)

> 题意：看不懂。

[P4848 崂山白花蛇草水](https://www.luogu.com.cn/problem/P4848)

> 题意：矩形区域第 $k$ 大，带修。强制在线。