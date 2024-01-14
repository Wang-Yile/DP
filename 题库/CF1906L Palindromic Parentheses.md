# [CF1906L Palindromic Parentheses](https://codeforces.com/contest/1906/problem/L)

考虑串 `(((...()...)))`，其 LPS 长度为 $\dfrac{n}{2}$。那么若 $k=\dfrac{n}{2}$，输出这个串即可。

接下来考虑串 `()(((...()...)))`，其 LPS 长度为 $\dfrac{n}{2}+1$，写个程序跑一下即可。（求 LPS 即为求它反转后的串与原串的 LCS）

而 $k<\dfrac{n}{2}$ 是无解的，因为一个平衡的括号串里 `(` 的个数有 $\dfrac{n}{2}$ 个，这些 `(` 构成一个回文子序列，因此 $k<\dfrac{n}{2}$ 不会成为 LPS，故无解。

$k=n$ 也是无解的，因为一个平衡的括号串开头一定是 `(`，结尾一定是 `)`，因此不存在长度为 $n$ 的回文序列。

$k>\dfrac{n}{2}$ 的规律可以借助打表程序找。

打表程序代码，输出每个长度为 $n$ 的平衡串的 LPS：

```cpp
#include <iostream>
#include <map>
#include <set>
#include <string>

using namespace std;

int n;
int a[2005];
int b[2005];
int dp[2005][2005];
map<int, set<string>> mp;

static inline void LPS() {
    for (int i = 1; i <= n; ++i) {
        b[n - i + 1] = a[i];
    }
    for (int i = 0; i <= n; ++i) {
        for (int j = 0; j <= n; ++j) {
            dp[i][j] = 0;
        }
    }
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= n; ++j) {
            dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            if (a[i] == b[j]) {
                dp[i][j] = max(dp[i][j], dp[i - 1][j - 1] + 1);
            }
        }
    }
    string s;
    for (int i = 1; i <= n; ++i) {
        s.push_back(a[i] + '(');
    }
    mp[dp[n][n]].insert(s);
}

static inline void dfs(int x) {
    if (x > n) {
        bool flag = true;
        int cur = 0;
        for (int i = 1; i <= n; ++i) {
            if (a[i] == 0) {
                ++cur;
            } else {
                --cur;
            }
            if (cur < 0) {
                flag = false;
                break;
            }
        }
        if (cur != 0) {
            flag = false;
        }
        if (flag) {
            LPS();
        }
        return;
    }
    a[x] = 0;
    dfs(x + 1);
    a[x] = 1;
    dfs(x + 1);
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n;
    dfs(1);
    for (auto [L, s] : mp) {
        cout << L << endl;
        for (auto str : s) {
            cout << "    " << str << endl;
        }
    }
    return 0;
}
```

以 $n=10$ 为例，可以提取出一组特殊的串：

```cpp
((((())))) // k = 5
(((())))() // k = 6
()((()))() // k = 7
()(())()() // k = 8
()()()()() // k = 9
```

发现只需保留一个长度为 $2\cdot(n-k)$ 的形如 `(((...()...)))` 的串，剩余的空位由串 `()` 尽量均匀地填补在两侧即可。

> 注：这个很像我在文章 [LPS](https://github.com/Wang-Yile/DP/blob/main/LPS.md) 里提到的朴素 LPS 的优化。

总结：构造题就是去尝试的过程，暴力程序可以更好地帮助找规律。

代码

```cpp
#include <iostream>

using namespace std;

int n, k;

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> k;
    if (k < n / 2 || k == n) {
        cout << -1 << endl;
        return 0;
    }
    int extra = n - 2 * (n - k); // 剩余的空位长度
    int cur = 0;                 // 已经输出的串长度
    for (int i = 1; i <= extra / 4; ++i) {
        cout << "()";
        cur += 2;
    }
    for (int i = 1; i <= n - k; ++i) {
        cout << "(";
        ++cur;
    }
    for (int i = 1; i <= n - k; ++i) {
        cout << ")";
        ++cur;
    }
    while (cur + 2 <= n) {
        cout << "()";
        cur += 2;
    }
    cout << endl;
    return 0;
}
```

打表程序代码比正解长，这或许就是构造题的魅力。