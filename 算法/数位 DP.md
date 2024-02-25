# 数位 DP

数位 DP 是通过处理数字的每一位解决一类区间满足条件的数的计数问题。

这个问题一般的处理方法是记忆化搜索，当然也有迭代写法。

形象地，数位 DP 就是依次填数以处理答案，然后通过复用答案加速。

## 例题

[P4999 烦人的数学作业](https://www.luogu.com.cn/problem/P4999)

> 题意：求 $[l,r]$ 内每个数的数字和之和。$1\le l\le r\le 10^{18},1\le T\le 20$。

这是经典例题，因此介绍将十分详细。

原问题可以转化为求 $[1,l]$ 和 $[1,r]$ 内每个数的数字和之和。

考虑求 $[1,2024]$ 内每个数的数字和之和，下面是填数的步骤。

如果最高位填 `2`，那么第二位就只能填 `0`；而如果最高位填 $[0,1]$，第二位不受此限制。

因此，记录变量 $limited$ 表示当前是否受限制。

考虑答案复用，设 $f_{pos,sum}$ 表示已经填了第 $pos$ 高位，已填入的数数字和为 $sum$ 时的答案。

```cpp
#include <iostream>

#define int long long

using namespace std;

const int mod = 1e9 + 7;

int T, l, r;
int a[20];
int f[20][180];

static inline int dfs(int x, int sum, bool limited) {
    if (!x) {
        return sum; // 全部填完
    }
    if (!limited && f[x][sum]) { // 记忆化
        return f[x][sum];
    }
    int up = limited ? a[x] : 9; // 可以填的数的上界
    int ret = 0;
    for (int i = 0; i <= up; ++i) {
        ret += dfs(x - 1, sum + i, limited & i == up);
        ret %= mod;
    }
    if (!limited) {
        f[x][sum] = ret;
    }
    return ret;
}
static inline int solve(int x) {
    int len = 0; // 解包数字
    while (x) {
        a[++len] = x % 10;
        x /= 10;
    }
    return dfs(len, 0, true);
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P4999.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> T;
    while (T--) {
        cin >> l >> r;
        cout << (solve(r) - solve(l - 1) + mod) % mod << endl;
    }
    return 0;
}
```

[P2602 [ZJOI2010] 数字计数](https://www.luogu.com.cn/problem/P2602)

> 题意：求 $[a,b]$ 中每个数码出现的次数。$1\le a\le b\le 10^{12}$。

对每个数码分开计数。

```cpp
#include <iostream>
#include <string.h>

#define int long long

using namespace std;

int a, b;
int val[20];
int f[20][20][2][2];

static inline int dfs(int x, int sum, int now, bool limited, bool prefix_zero) {
    if (x == 0) {
        return sum;
    }
    if (f[x][sum][limited][prefix_zero] >= 0) {
        return f[x][sum][limited][prefix_zero];
    }
    int up = limited ? val[x] : 9;
    int ret = 0;
    for (int i = 0; i <= up; ++i) {
        ret += dfs(x - 1, sum + ((!prefix_zero || i) && (i == now)), now, limited && (i == up), prefix_zero && (i == 0));
    }
    f[x][sum][limited][prefix_zero] = ret;
    return ret;
}
static inline int unpack(int x) {
    int len = 0;
    while (x) {
        val[++len] = x % 10;
        x /= 10;
    }
    return len;
}
static inline int solve(int x, int now) {
    int len = unpack(x);
    memset(f, -1, sizeof f);
    return dfs(len, 0, now, true, true);
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P2602.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> a >> b;
    for (int i = 0; i <= 9; ++i) {
        cout << solve(b, i) - solve(a - 1, i) << ' ';
    }
    cout << flush;
    return 0;
}
```

[P1831 杠杆数](https://www.luogu.com.cn/problem/P1831)

> 题意：如果把一个数的某一位当成支点，使得两侧的力矩和相等（力矩平衡），那么这个数被称为杠杆数。问 $[x,y]$ 中有几个杠杆数。$1\le x\le y\le 10^{18}$。

```cpp
#include <iostream>
#include <string.h>

#define int long long

using namespace std;

int x, y;
int val[20];
//  [pos][now][sum]
int f[20][20][1600];

static inline int dfs(int pos, int now, int sum, bool limited) {
    if (pos == 0) {
        return sum == 0;
    }
    if (sum < 0) {
        return 0;
    }
    if (!limited && ~f[pos][now][sum]) {
        return f[pos][now][sum];
    }
    int up = limited ? val[pos] : 9;
    int ret = 0;
    for (int i = 0; i <= up; ++i) {
        ret += dfs(pos - 1, now, sum + i * (pos - now), limited & (i == up));
    }
    if (!limited) {
        f[pos][now][sum] = ret;
    }
    return ret;
}

static inline int solve(int num) {
    int len = 0;
    while (num) {
        val[++len] = num % 10;
        num /= 10;
    }
    int ans = 0;
    for (int i = 1; i <= len; ++i) {
        ans += dfs(len, i, 0, true);
    }
    return ans - len + 1;
}

signed main() {
#ifndef ONLINE_JUDGE
    freopen("P1831.in", "r", stdin);
#endif
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> x >> y;
    memset(f, -1, sizeof f);
    cout << solve(y) - solve(x - 1) << endl;
    return 0;
}
```