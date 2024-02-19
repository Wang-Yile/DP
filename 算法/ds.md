# Data Structures
## Segment Tree

线段树通过逐层划分序列建树（每层平均分开），使得树高不超过 $\log_2 n$，因此在线段树上进行操作是 $O(\log_2 n)$ 的。还有划分权值建树的线段树，称为权值线段树。

---

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

## Persistent Segment Tree

可持久化线段树是一种可持久化数据结构，可持久化权值线段树又称为主席树。

## Li-Chao Segment Tree

李超线段树是一种维护平面信息的特殊线段树。（至少我这样理解）

---

**P4097** [HEOI2013] Segment

_Problem_  
$n$ 次操作。1. 插入一条第一象限内的线段 $(x_0,y_0,x_1,y_1)$，编号为当前已插入的线段数；2. 询问与直线 $x=k$ 相交的线段中交点 $y$ 坐标最高的线段的编号。特别地，若 $x_0=x_1$，其与 $x=x_0$ 的交点的坐标被认为是 $(x_0,\max(y_0,y_1))$。$1 \le n \le 10^5,1 \le k,x_0,x_1 \le 39989,1 \le y_0,y_1 \le 10^9$。

_Note_  
考虑维护：$x$ 坐标区间 $[s,t]$ 中点为 $mid$ 时，与直线 $x=mid$ 相交的线段中交点 $y$ 坐标最高的线段编号。用线段树维护。插入操作：当待插入线段完全覆盖当前区间时，检查它是否是该区间的最优答案。注意，如果一条线段不再是区间 $[s,t]$ 的最优答案，它仍可能是区间 $[s,mid]$ 或 $(mid,t]$ 的最优答案，因此递归处理标记。查询时在路径上每个点的最优答案中取最优。时间复杂度 $O(n \log_2^2 n)$。

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

## Balanced Tree

平衡树是一种二叉搜索树（BST），通过旋转操作使得树不退化，以维持 $O(\log_2 n)$ 时间复杂度。

**P3369 / P6136** 【模板】普通平衡树 / 【模板】普通平衡树（数据加强版）

_Problem_  
支持动态加数，删数，查询数的排名，查询排名处的数。$$。

_Note_  
以下给出几种平衡树的特点
- AVL 在高度差大于 $1$ 时执行旋转。  
- Splay 每次把要操作的点旋转到根。
- 替罪羊树不旋转，如果子树失衡（一棵子树大小占总的大小比例超过平衡因子），那么重构整棵子树。
- 还有一种 BST，在进行 B 次操作后全部重构，这被称为根号重构，有应用于 K-D Tree。

<details>
<summary><i>AVL Code</i></summary>

```cpp
#include <iostream>

using namespace std;

int n, m;

class AVL {
  private:
    struct AVLNode {
        AVLNode *ls = nullptr, *rs = nullptr;
        int siz = 0;
        int dep = 0;
        int key = 0;
    } *__rt;
    inline int _HeightDiff(const AVLNode *const p) {
        if (p->ls == nullptr && p->rs == nullptr) {
            return 0;
        } else if (p->ls == nullptr) {
            return -p->rs->dep;
        } else if (p->rs == nullptr) {
            return p->ls->dep;
        }
        return p->ls->dep - p->rs->dep;
    }
    inline void _Update(AVLNode *p) {
        if (p == nullptr) {
            return;
        }
        p->siz = 1;
        if (p->ls != nullptr) {
            p->siz += p->ls->siz;
        }
        if (p->rs != nullptr) {
            p->siz += p->rs->siz;
        }
        p->dep = 0;
        if (p->ls != nullptr) {
            p->dep = p->ls->dep;
        }
        if (p->rs != nullptr) {
            p->dep = max(p->dep, p->rs->dep);
        }
        ++p->dep;
    }
    inline AVLNode *_TurnLeft(AVLNode *p) {
        if (p == nullptr) {
            return p;
        }
        auto tmp = p->rs;
        p->rs = tmp->ls;
        tmp->ls = p;
        p = tmp;
        _Update(p->ls);
        _Update(p);
        return p;
    }
    inline AVLNode *_TurnRight(AVLNode *p) {
        if (p == nullptr) {
            return p;
        }
        auto tmp = p->ls;
        p->ls = tmp->rs;
        tmp->rs = p;
        p = tmp;
        _Update(p->rs);
        _Update(p);
        return p;
    }
    inline AVLNode *_Maintain(AVLNode *p) {
        if (p == nullptr) {
            return p;
        }
        int s = _HeightDiff(p);
        if (s > 1) {
            int ss = _HeightDiff(p->ls);
            if (ss > 0) { // LL
                p = _TurnRight(p);
            } else { // LR
                p->ls = _TurnLeft(p->ls);
                p = _TurnRight(p);
            }
        } else if (s < -1) {
            int ss = _HeightDiff(p->rs);
            if (ss < 0) { // RR
                p = _TurnLeft(p);
            } else { // RL
                p->rs = _TurnRight(p->rs);
                p = _TurnLeft(p);
            }
        } else {
            _Update(p);
        }
        return p;
    }
    inline AVLNode *_InsertNode(AVLNode *p, int key) {
        if (p == nullptr) {
            p = new AVLNode();
            p->key = key;
            p->siz = 1;
            p->dep = 0;
        } else if (key < p->key) {
            p->ls = _InsertNode(p->ls, key);
        } else if (key >= p->key) {
            p->rs = _InsertNode(p->rs, key);
        }
        p = _Maintain(p);
        return p;
    }
    inline AVLNode *_DeleteNode(AVLNode *p, int key) { // 旋转至叶子节点然后删除
        if (p == nullptr) {
            return p;
        }
        if (key < p->key) {
            p->ls = _DeleteNode(p->ls, key);
        } else if (key > p->key) {
            p->rs = _DeleteNode(p->rs, key);
        } else {
            if (p->ls == nullptr || p->rs == nullptr) {
                p = (AVLNode *)((long)p->ls ^ (long)p->rs);
            } else if (_HeightDiff(p) < 0) {
                p = _TurnRight(p);
                p = _DeleteNode(p, key);
            } else {
                p = _TurnLeft(p);
                p = _DeleteNode(p, key);
            }
        }
        p = _Maintain(p);
        return p;
    }
    inline void _Clear(AVLNode *p) {
        if (p == nullptr) {
            return;
        }
        _Clear(p->ls);
        _Clear(p->rs);
        delete p;
    }
    inline void _Print(AVLNode *p) {
        if (p == nullptr) {
            return;
        }
        _Print(p->ls);
        cerr << p->key << ' ';
        _Print(p->rs);
    }
    inline int _QueryRank(int key) {
        auto cur = __rt;
        int ret = 0;
        while (cur != nullptr) {
            if (cur->key >= key) {
                cur = cur->ls;
            } else {
                if (cur->ls != nullptr) {
                    ret += cur->ls->siz;
                }
                ++ret;
                cur = cur->rs;
            }
        }
        return ret + 1;
    }
    inline int _QueryByRank(int key) {
        auto cur = __rt;
        while (cur != nullptr) {
            if (cur->ls != nullptr && cur->ls->siz + 1 == key) {
                return cur->key;
            }
            if (cur->ls == nullptr && key == 1) {
                return cur->key;
            }
            if (cur->ls != nullptr && cur->ls->siz >= key) {
                cur = cur->ls;
            } else {
                if (cur->ls != nullptr) {
                    key -= cur->ls->siz;
                }
                --key;
                cur = cur->rs;
            }
        }
        return 0;
    }

  public:
    AVL() {}
    AVL(const AVL &avl) = delete;
    ~AVL() { clear(); }
    inline void insert(int key) {
        __rt = _InsertNode(__rt, key);
    }
    inline void erase(int key) {
        __rt = _DeleteNode(__rt, key);
    }
    inline int rank(int key) {
        return _QueryRank(key);
    }
    inline int byrank(int key) {
        return _QueryByRank(key);
    }
    inline int size() {
        if (__rt == nullptr) {
            return 0;
        }
        return __rt->siz;
    }
    inline void clear() {
        _Clear(__rt);
        __rt = nullptr;
    }
    inline void print() {
        _Print(__rt);
    }
};
AVL tr;

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P6136.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) {
        int x;
        cin >> x;
        tr.insert(x);
    }
    int ans = 0;
    int lastAns = 0;
    while (m--) {
        int opt, x;
        cin >> opt >> x;
        x ^= lastAns;
        if (opt == 1) {
            tr.insert(x);
        } else if (opt == 2) {
            tr.erase(x);
        } else if (opt == 3) {
            lastAns = tr.rank(x);
            ans ^= lastAns;
        } else if (opt == 4) {
            lastAns = tr.byrank(x);
            ans ^= lastAns;
        } else if (opt == 5) {
            lastAns = tr.byrank(tr.rank(x) - 1);
            ans ^= lastAns;
        } else if (opt == 6) {
            lastAns = tr.byrank(tr.rank(x + 1));
            ans ^= lastAns;
        } else {
            cerr << "unknown operation: " << opt << endl;
        }
    }
    cout << ans << endl;
    return 0;
}
```
</details>
<details>
<summary><i>Splay Code (Empty)</i></summary>

```cpp
```
</details>
<details>
<summary><i>替罪羊 Code (Empty)</i></summary>

```cpp
```
</details>

## K-D Tree

K-D Tree 是一种通过划分空间实现在高维度上查询的数据结构，树高至多为 $\log_2 n$。

建树  
K-D Tree 的划分方式是选择一个维度，然后选择一个点作为这棵子树的根节点，该维度上小于这个点的进入左子树，大于这个点的进入右子树。为保证复杂度，选择中位点作为根节点，并按最大方差或者交替选择维度。K-D Tree 需要维护 $mx_i$ 和 $mn_i$，表示其子树第 $i$ 维坐标的极值。

查询  
K-D Tree 上一个节点管辖一个超立方体，当查询立方体 $R$ 时，节点分为三类：1. 包含 $R$；2. 与 $R$ 无交；3. 部分交 $R$。第一类答案为子树答案总和，第二类答案为 $0$，递归求解第三类即可。复杂度分析部分见 [OI-Wiki](https://oi-wiki.org/ds/kdt/#%E5%A4%8D%E6%9D%82%E5%BA%A6%E5%88%86%E6%9E%90)，最坏时间复杂度是 $O(\log_2 n + n^{1 - \frac 1 k})$。

插入 / 删除  
K-D Tree 使用暴力插入 / 删除，定期重构。可以使用替罪羊重构子树（即平衡因子）。也可以使用根号重构，操作达到 $B$ 次就重构整棵树。均摊复杂度修改 $O(n \log_2 n \div B)$，查询 $O(B + n^{1 - \frac 1 k})$。一般取 $B = \sqrt{n \log_2 n}$。

---

**P4148** 简单题

_Problem_  
平面直角坐标系第一象限中，动态加点，维护形覆盖的点之和。$1 \le T \le 2 \cdot 10^5$。

_Note_  
K-D Tree 模板，如上述维护即可。

_Review_  
K-D Tree 的 `maintain` 函数需要注意常数对程序运行效率的影响，这是大规模访问内存。

<details>
<summary><i>Code</i></summary>

使用替罪羊重构。

```cpp
#include <algorithm>
#include <ctype.h>
#include <stdio.h>

using namespace std;

const double balance = 0.6;

static inline int read() {
    char ch = getchar();
    int x = 0;
    while (ch < 48) {
        ch = getchar();
    }
    while (ch >= 48 && ch <= 57) {
        x = (x << 3) + (x << 1) + (ch ^ 48);
        ch = getchar();
    }
    return x;
}

int n;
int globalCompareTag;
struct node {
    int x[2], w;
    inline int operator<(const node &nd) const {
        return x[globalCompareTag] < nd.x[globalCompareTag];
    }
} f[200005];

struct kdt {
    int ls, rs;
    int w;
    int x[2];
    int mn[2];
    int mx[2];
    int sum, siz;
} kd[200005];
int cnt;
int rubbish_bin[200005], rb_tail;
static inline int allocateNewKDT() {
    if (rb_tail) {
        return rubbish_bin[rb_tail--];
    }
    return ++cnt;
}
static inline void maintain(int p) {
    int l = kd[p].ls;
    int r = kd[p].rs;
    kd[p].mn[0] = kd[p].x[0];
    kd[p].mn[1] = kd[p].x[1];
    kd[p].mx[0] = kd[p].x[0];
    kd[p].mx[1] = kd[p].x[1];
    if (l) {
        kd[p].mn[0] = min(kd[p].mn[0], kd[l].mn[0]);
        kd[p].mn[1] = min(kd[p].mn[1], kd[l].mn[1]);
        kd[p].mx[0] = max(kd[p].mx[0], kd[l].mx[0]);
        kd[p].mx[1] = max(kd[p].mx[1], kd[l].mx[1]);
    }
    if (r) {
        kd[p].mn[0] = min(kd[p].mn[0], kd[r].mn[0]);
        kd[p].mn[1] = min(kd[p].mn[1], kd[r].mn[1]);
        kd[p].mx[0] = max(kd[p].mx[0], kd[r].mx[0]);
        kd[p].mx[1] = max(kd[p].mx[1], kd[r].mx[1]);
    }
    kd[p].sum = kd[l].sum + kd[r].sum + kd[p].w;
    kd[p].siz = kd[l].siz + kd[r].siz + 1;
}
static inline int build(int l, int r, int tag) {
    if (l > r) {
        return 0;
    }
    int mid = (l + r) >> 1;
    globalCompareTag = tag;
    nth_element(f + l, f + mid, f + r + 1);
    int p = allocateNewKDT();
    kd[p].x[0] = f[mid].x[0];
    kd[p].x[1] = f[mid].x[1];
    kd[p].w = f[mid].w;
    kd[p].ls = build(l, mid - 1, tag ^ 1);
    kd[p].rs = build(mid + 1, r, tag ^ 1);
    maintain(p);
    return p;
}
int tail = 0;
static inline void ttoa(int p) {
    if (!p) {
        return;
    }
    ttoa(kd[p].ls);
    f[++tail] = {kd[p].x[0], kd[p].x[1], kd[p].w};
    ttoa(kd[p].rs);
    rubbish_bin[++rb_tail] = p;
}
static inline int rebuild(int p, int tag) {
    if ((double)kd[kd[p].ls].siz > balance * kd[p].siz || (double)kd[kd[p].rs].siz > balance * kd[p].siz) {
        tail = 0;
        ttoa(p);
        p = build(1, kd[p].siz, tag);
    }
    return p;
}
static inline int insert(int x, int y, int w, int p, int tag) {
    if (!p) {
        p = allocateNewKDT();
        kd[p].x[0] = x;
        kd[p].x[1] = y;
        kd[p].w = w;
        kd[p].ls = kd[p].rs = 0;
        maintain(p);
        return p;
    }
    int cur = tag ? y : x;
    if (cur <= kd[p].x[tag]) {
        kd[p].ls = insert(x, y, w, kd[p].ls, tag ^ 1);
    } else {
        kd[p].rs = insert(x, y, w, kd[p].rs, tag ^ 1);
    }
    maintain(p);
    p = rebuild(p, tag);
    return p;
}
static inline int query(int x1, int y1, int x2, int y2, int p) {
    if (!p || x1 > kd[p].mx[0] || kd[p].mn[0] > x2 || y1 > kd[p].mx[1] || kd[p].mn[1] > y2) {
        return 0;
    }
    if (x1 <= kd[p].mn[0] && kd[p].mx[0] <= x2 && y1 <= kd[p].mn[1] && kd[p].mx[1] <= y2) {
        return kd[p].sum;
    }
    int ret = 0;
    if (x1 <= kd[p].x[0] && kd[p].x[0] <= x2 && y1 <= kd[p].x[1] && kd[p].x[1] <= y2) {
        ret += kd[p].w;
    }
    ret += query(x1, y1, x2, y2, kd[p].ls) + query(x1, y1, x2, y2, kd[p].rs);
    return ret;
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P4148.in", "r", stdin);
#endif
    n = read();
    int rt = 0;
    int las = 0;
    while (true) {
        int op;
        op = read();
        if (op == 1) {
            int x = read() ^ las;
            int y = read() ^ las;
            int w = read() ^ las;
            rt = insert(x, y, w, rt, 0);
        } else if (op == 2) {
            int x1 = read() ^ las;
            int y1 = read() ^ las;
            int x2 = read() ^ las;
            int y2 = read() ^ las;
            las = query(x1, y1, x2, y2, rt);
            printf("%d\n", las);
        } else {
            break;
        }
    }
    return 0;
}
```
</details>

## Tree Cover Tree

树套树用于解决一类一棵树难以解决的问题。经典的应用场景有动态区间 Kth。

**P3380** 【模板】树套树

_Problem_  
维护一个数列。1. 查询 $k$ 在区间内的排名；2. 查询区间内排名为 $k$ 的值；3. 单点修改；4. 查询 $k$ 在区间内的前驱；5. 查询 $k$ 在区间内的后继。前驱定义为严格小于 $x$ 的最大的数，后继定义为严格大于 $x$ 的最小的数。$1 \le n,m \le 5 \cdot 10^4,0 \le a_i \le 10^8$。

_Note_  
可以用树状数组套主席树解决本题。

_Review_  
本题相当于 Dynamic Rankings 的加强版，需要注意主席树的细节。

<details>
<summary><i>Code</i></summary>

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

using namespace std;

int n, m;
int a[100005];
int val[100005];

struct ques {
    int opt, l, r, k;
} qs[50005];

int rt[100005];
struct node {
    int ls, rs, val;
} d[24000005];
int cnt;
static inline int clone(int p) {
    d[++cnt] = d[p];
    return cnt;
}
static inline int insert(int x, int s, int t, int va, int p) {
    p = clone(p);
    if (s == t) {
        d[p].val += va;
        return p;
    }
    int mid = (s + t) >> 1;
    if (x <= mid) {
        d[p].ls = insert(x, s, mid, va, d[p].ls);
    } else {
        d[p].rs = insert(x, mid + 1, t, va, d[p].rs);
    }
    d[p].val = d[d[p].ls].val + d[d[p].rs].val;
    return p;
}
vector<int> rt1, rt2;
static inline int calc() {
    int ret = 0;
    for (auto x : rt1) {
        ret -= d[d[x].ls].val;
    }
    for (auto x : rt2) {
        ret += d[d[x].ls].val;
    }
    return ret;
}
static inline void jump_ls() {
    for (auto &x : rt1) {
        x = d[x].ls;
    }
    for (auto &x : rt2) {
        x = d[x].ls;
    }
}
static inline void jump_rs() {
    for (auto &x : rt1) {
        x = d[x].rs;
    }
    for (auto &x : rt2) {
        x = d[x].rs;
    }
}
static inline int query(int s, int t, int k) {
    if (s == t) {
        return 1;
    }
    int x = calc();
    int mid = (s + t) >> 1;
    if (k <= mid) {
        jump_ls();
        return query(s, mid, k);
    } else {
        jump_rs();
        return x + query(mid + 1, t, k);
    }
}
static inline int queryby(int s, int t, int k) {
    if (s == t) {
        return s;
    }
    int x = calc();
    int mid = (s + t) >> 1;
    if (k <= x) {
        jump_ls();
        return queryby(s, mid, k);
    } else {
        jump_rs();
        return queryby(mid + 1, t, k - x);
    }
}
static inline int querycount(int s, int t, int k) {
    if (s == t) {
        int ret = 0;
        for (auto x : rt1) {
            ret -= d[x].val;
        }
        for (auto x : rt2) {
            ret += d[x].val;
        }
        return ret;
    }
    int mid = (s + t) >> 1;
    if (k <= mid) {
        jump_ls();
        return querycount(s, mid, k);
    } else {
        jump_rs();
        return querycount(mid + 1, t, k);
    }
}

int len;
static inline int lowbit(int x) {
    return x & (-x);
}
static inline void BIT_insert(int x, int w) {
    while (x <= n) {
        rt[x] = insert(w, 1, len, 1, rt[x]);
        x += lowbit(x);
    }
}
static inline void BIT_erase(int x, int w) {
    while (x <= n) {
        rt[x] = insert(w, 1, len, -1, rt[x]);
        x += lowbit(x);
    }
}
static inline void BIT_init(int l, int r) {
    rt1.clear();
    rt2.clear();
    --l;
    while (l) {
        rt1.push_back(rt[l]);
        l -= lowbit(l);
    }
    while (r) {
        rt2.push_back(rt[r]);
        r -= lowbit(r);
    }
}
static inline int BIT_query(int l, int r, int k) {
    BIT_init(l, r);
    return query(1, len, k);
}
static inline int BIT_queryby(int l, int r, int k) {
    BIT_init(l, r);
    return queryby(1, len, k);
}
static inline int BIT_querycount(int l, int r, int k) {
    BIT_init(l, r);
    return querycount(1, len, k);
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P3380.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        val[++len] = a[i];
    }
    for (int i = 1; i <= m; ++i) {
        int opt;
        cin >> opt;
        qs[i].opt = opt;
        if (opt == 1) {
            cin >> qs[i].l >> qs[i].r >> qs[i].k;
            val[++len] = qs[i].k;
        } else if (opt == 2) {
            cin >> qs[i].l >> qs[i].r >> qs[i].k;
        } else if (opt == 3) {
            cin >> qs[i].l >> qs[i].k;
            val[++len] = qs[i].k;
        } else if (opt == 4) {
            cin >> qs[i].l >> qs[i].r >> qs[i].k;
            val[++len] = qs[i].k;
        } else {
            cin >> qs[i].l >> qs[i].r >> qs[i].k;
            val[++len] = qs[i].k;
        }
    }
    sort(val + 1, val + len + 1);
    len = (int)(unique(val + 1, val + len + 1) - val - 1);
    for (int i = 1; i <= n; ++i) {
        a[i] = (int)(lower_bound(val + 1, val + len + 1, a[i]) - val);
        BIT_insert(i, a[i]);
    }
    for (int i = 1; i <= m; ++i) {
        auto [opt, l, r, k] = qs[i];
        if (opt == 1) {
            k = (int)(lower_bound(val + 1, val + len + 1, k) - val);
            int rnk = BIT_query(l, r, k);
            cout << rnk << endl;
        } else if (opt == 2) {
            cout << val[BIT_queryby(l, r, k)] << endl;
        } else if (opt == 3) {
            k = (int)(lower_bound(val + 1, val + len + 1, k) - val);
            BIT_erase(l, a[l]);
            a[l] = k;
            BIT_insert(l, a[l]);
        } else if (opt == 4) {
            k = (int)(lower_bound(val + 1, val + len + 1, k) - val);
            int rnk = BIT_query(l, r, k);
            if (rnk <= 1) {
                cout << "-2147483647" << endl;
            } else {
                cout << val[BIT_queryby(l, r, rnk - 1)] << endl;
            }
        } else {
            k = (int)(lower_bound(val + 1, val + len + 1, k) - val);
            int rnk = BIT_query(l, r, k);
            int count = BIT_querycount(l, r, k);
            if (rnk + count > r - l + 1) {
                cout << "2147483647" << endl;
            } else {
                cout << val[BIT_queryby(l, r, rnk + count)] << endl;
            }
        }
    }
    return 0;
}
```
</details>

## Dynamic Tree

动态树是一类动态连接 / 断开边的森林上信息维护问题。

### Link-Cut Tree

### Top Tree
