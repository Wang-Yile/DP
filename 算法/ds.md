# Data Structures
## Segment Tree
**SP1043(GSS1)**

_Problem_  
静态求区间内最大子段和。$1 \le n \le 5 \cdot 10^4$。

_Note_  
对于一连续区间，维护前缀最大值 lmax，后缀最大值 rmax，最大子段和 gss，区间和 sum。设法合并相邻区间即可。

_Review_  
设计维护一些区间信息用于更新。

<details>
<summary><i>Code</i></summary>

```cpp
#include <initializer_list>
#include <iostream>

using namespace std;

int n, m;
int a[50005];

template <typename T, typename... Args>
inline T max(T first, Args... args) {
    int ret = first;
    (void)initializer_list<int>{(ret = max(ret, args), 0)...};
    return ret;
}

struct node {
    int sum = 0;
    int lmax = -1e9;
    int rmax = -1e9;
    int gss = -1e9;
    inline node operator+(const node &nd) const {
        node ret;
        ret.sum = sum + nd.sum;
        ret.lmax = max(lmax, sum + nd.lmax);
        ret.rmax = max(nd.rmax, rmax + nd.sum);
        ret.gss = max(ret.sum, gss, nd.gss, rmax + nd.lmax);
        return ret;
    }
} d[200005];
static inline void build(int s, int t, int p) {
    if (s == t) {
        d[p].sum = a[s];
        d[p].lmax = d[p].rmax = d[p].gss = a[s];
        return;
    }
    int mid = (s + t) >> 1;
    build(s, mid, p << 1);
    build(mid + 1, t, p << 1 | 1);
    d[p] = d[p << 1] + d[p << 1 | 1];
}
static inline node query(int l, int r, int s, int t, int p) {
    if (l <= s && r >= t) {
        return d[p];
    }
    int mid = (s + t) >> 1;
    node ret;
    if (l <= mid) {
        ret = query(l, r, s, mid, p << 1);
    }
    if (r > mid) {
        ret = ret + query(l, r, mid + 1, t, p << 1 | 1);
    }
    return ret;
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("SP1043.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
    }
    build(1, n, 1);
    cin >> m;
    while (m--) {
        int l, r;
        cin >> l >> r;
        cout << query(l, r, 1, n, 1).gss << endl;
    }
    return 0;
}
```
</details>

---

**SP1557(GSS2)** $\orange\star$

_Problem_  
静态求区间最大子段和，相同的数只算一次。子段可以为空。$1 \le n,q \le 10^5,-10^5 \le a_i \le 10^5$。

_Note_  
设 $f_l$ 表示区间 $[l,i]$ 的贡献（贡献已排除重复元素干扰），$pre_i$ 表示数字 $i$ 最后一次出现的位置。考虑添加 $i$，$a_i$ 会对区间 $(pre_{a_i},i]$ 的贡献产生影响，看作区间加。询问区间 $[l,r]$ 的 GSS 为 $f_l$ 的历史最大值。最终可以把询问按右端点排序，依次添加元素处理。

_Review_  
有特殊条件可以考虑去掉特殊条件怎么做，然后排除特殊条件对结果的影响。

<details>
<summary><i>Code</i></summary>

```cpp
#include <algorithm>
#include <iostream>
#include <map>

using namespace std;

const int block_size = 320;

int n, q;
int a[100005];
map<int, int> pre;

struct ques {
    int l, r, id, ans;
    inline bool operator<(const ques &nd) const {
        return r < nd.r;
    }
} qs[100005];

struct node {
    int sum = 0, tag = 0;
    int hismaxsum = 0, hismaxtag = 0; // SGT 区间历史最大值模板
    inline node operator+(const node &nd) const {
        node ret;
        ret.sum = max(sum, nd.sum);
        ret.hismaxsum = max(hismaxsum, nd.hismaxsum);
        return ret;
    }
} d[400005];
static inline void pushdown(int p) {
    d[p << 1].hismaxsum = max(d[p << 1].hismaxsum, d[p << 1].sum + d[p].hismaxtag);
    d[p << 1].hismaxtag = max(d[p << 1].hismaxtag, d[p << 1].tag + d[p].hismaxtag);
    d[p << 1].sum += d[p].tag;
    d[p << 1].tag += d[p].tag;
    d[p << 1 | 1].hismaxsum = max(d[p << 1 | 1].hismaxsum, d[p << 1 | 1].sum + d[p].hismaxtag);
    d[p << 1 | 1].hismaxtag = max(d[p << 1 | 1].hismaxtag, d[p << 1 | 1].tag + d[p].hismaxtag);
    d[p << 1 | 1].sum += d[p].tag;
    d[p << 1 | 1].tag += d[p].tag;
    d[p].tag = 0;
    d[p].hismaxtag = 0;
}
static inline void update(int l, int r, int s, int t, int c, int p) {
    if (l <= s && r >= t) {
        d[p].sum += c;
        d[p].hismaxsum = max(d[p].hismaxsum, d[p].sum);
        d[p].tag += c;
        d[p].hismaxtag = max(d[p].hismaxtag, d[p].tag);
        return;
    }
    int mid = (s + t) >> 1;
    pushdown(p);
    if (l <= mid) {
        update(l, r, s, mid, c, p << 1);
    }
    if (r > mid) {
        update(l, r, mid + 1, t, c, p << 1 | 1);
    }
    d[p] = d[p << 1] + d[p << 1 | 1];
}
static inline node query(int l, int r, int s, int t, int p) {
    if (l <= s && r >= t) {
        return d[p];
    }
    int mid = (s + t) >> 1;
    pushdown(p);
    node ret;
    if (l <= mid) {
        ret = query(l, r, s, mid, p << 1);
    }
    if (r > mid) {
        ret = ret + query(l, r, mid + 1, t, p << 1 | 1);
    }
    return ret;
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("SP1557.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
    }
    cin >> q;
    for (int i = 1; i <= q; ++i) {
        cin >> qs[i].l >> qs[i].r;
        qs[i].id = i;
    }
    sort(qs + 1, qs + q + 1);
    int now = 1;
    for (int i = 1; i <= n; ++i) {
        update(pre[a[i]] + 1, i, 1, n, a[i], 1);
        pre[a[i]] = i;
        while (now <= q && qs[now].r == i) {
            qs[now].ans = query(qs[now].l, qs[now].r, 1, n, 1).hismaxsum;
            ++now;
        }
    }
    sort(qs + 1, qs + q + 1, [](const ques &x, const ques &y) { return x.id < y.id; });
    for (int i = 1; i <= q; ++i) {
        cout << qs[i].ans << endl;
    }
    return 0;
}
```
</details>

## Li-Chao Segment Tree

**P4097** [HEOI2013] Segment

_Problem_  
$n$ 次操作。1. 插入一条第一象限内的线段 $(x_0,y_0,x_1,y_1)$，编号为当前已插入的线段数；2. 询问与直线 $x=k$ 相交的线段中交点 $y$ 坐标最高的线段的编号。特别地，若 $x_0=x_1$，其与 $x=x_0$ 的交点的坐标被认为是 $(x_0,\max(y_0,y_1))$。$1 \le n \le 10^5,1 \le k,x_0,x_1 \le 39989,1 \le y_0,y_1 \le 10^9$。

_Note_  
考虑维护：$x$ 坐标区间 $[s,t]$ 中点为 $mid$ 时，与直线 $x=mid$ 相交的线段中交点 $y$ 坐标最高的线段编号。用线段树维护。插入操作：当待插入线段完全覆盖当前区间时，检查它是否是该区间的最优答案。注意，如果一条线段不再是区间 $[s,t]$ 的最优答案，它仍可能是区间 $[s,mid]$ 或 $(mid,t]$ 的最优答案，因此递归处理标记。查询时在路径上每个点的最优答案中取最优。时间复杂度 $O(n \log^2 n)$。

_Review_  

<details>
<summary><i>Code</i></summary>

```cpp
#include <iostream>

using namespace std;

const int modfish = 39989;
const int modcat = 1e9;
const double eps = 1e-6;

int n;

struct segment {
    double k = 0, b = 0;
    inline double get(int x) const {
        return k * x + b;
    }
} seg[100005];
int cnt;

int d[160005];
static inline void updatetag(int x, int s, int t, int p) {
    int mid = (s + t) >> 1;
    int u = x;
    int &v = d[p];
    double s1 = seg[u].get(mid);
    double s2 = seg[v].get(mid);
    if (s1 - s2 > eps || (abs(s1 - s2) < eps && u < v)) {
        swap(u, v);
    }
    s1 = seg[u].get(s);
    s2 = seg[v].get(s);
    if (s1 - s2 > eps || (abs(s1 - s2) < eps && u < v)) {
        updatetag(u, s, mid, p << 1);
    }
    s1 = seg[u].get(t);
    s2 = seg[v].get(t);
    if (s1 - s2 > eps || (abs(s1 - s2) < eps && u < v)) {
        updatetag(u, mid + 1, t, p << 1 | 1);
    }
}
static inline void insert(int x, int l, int r, int s, int t, int p) {
    if (l <= s && r >= t) {
        updatetag(x, s, t, p);
        return;
    }
    int mid = (s + t) >> 1;
    if (l <= mid) {
        insert(x, l, r, s, mid, p << 1);
    }
    if (r > mid) {
        insert(x, l, r, mid + 1, t, p << 1 | 1);
    }
}
static inline pair<double, int> max(const pair<double, int> &x, const pair<double, int> &y) {
    if (y.first - x.first > eps) {
        return y;
    }
    if (x.first - y.first > eps) {
        return x;
    }
    if (x.second < y.second) {
        return x;
    }
    return y;
}
static inline pair<double, int> query(int x, int s, int t, int p) {
    double res = seg[d[p]].get(x);
    if (s == t) {
        return make_pair(res, d[p]);
    }
    int mid = (s + t) >> 1;
    pair<double, int> ret;
    if (x <= mid) {
        ret = query(x, s, mid, p << 1);
    } else {
        ret = max(ret, query(x, mid + 1, t, p << 1 | 1));
    }
    ret = max(ret, make_pair(res, d[p]));
    return ret;
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P4097.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n;
    int las = 0;
    while (n--) {
        int op;
        cin >> op;
        if (op == 0) {
            int k;
            cin >> k;
            k = (k + las - 1) % modfish + 1;
            las = query(k, 1, 39989, 1).second;
            cout << las << '\n';
        } else {
            int x0, y0, x1, y1;
            cin >> x0 >> y0 >> x1 >> y1;
            x0 = (x0 + las - 1) % modfish + 1;
            y0 = (y0 + las - 1) % modcat + 1;
            x1 = (x1 + las - 1) % modfish + 1;
            y1 = (y1 + las - 1) % modcat + 1;
            if (x0 > x1) {
                swap(x0, x1);
                swap(y0, y1);
            }
            ++cnt;
            if (x0 != x1) {
                seg[cnt].k = (double)(y1 - y0) / (x1 - x0);
                seg[cnt].b = y0 - seg[cnt].k * x0;
            } else {
                seg[cnt].k = 0;
                seg[cnt].b = max(y0, y1);
            }
            insert(cnt, x0, x1, 1, 39989, 1);
        }
    }
    cout << flush;
    return 0;
}
```
</details>
