# P4342 [IOI 1998] Polygon 题解

题目链接：[P4342 [IOI 1998] Polygon](https://www.luogu.com.cn/problem/P4342)

## 思路分析

这道题的操作对象是一段连续的区间，而且每次合并的都是相邻的元素，同时问的是整体的最优值，最关键的是 `n` 很小，很显然是区间 DP。

同时这是连成了一个环，于是就可以用环状区间 DP 的思想，也就是把整个区间延长成 `2n`，最后在找最大值的时候从 `1` 到 `n` 遍历每一个可以作为起点的 `l`。

以上是正常遇到这题的思路，但是这题很坑...

因为这道题涉及到了乘法，但是我们知道，如果两个负数相乘有可能得到正数，在这题里面，我们甚至有可能用两个负数相乘得到比两个正数相乘还要大的数字。

所以我们要维护最小值，而维护最小值也有可能是由最大值和最小值相乘得到。

但是加法就不需要了，最大值一定是由两个最大值相加得到，最小值也一定是由两个最小值相加得到。

## 状态设计

设：

```cpp
mx[l][r]
```

表示区间 `[l, r]` 可以得到的最大值。

设：

```cpp
mn[l][r]
```

表示区间 `[l, r]` 可以得到的最小值。

## 初始化

当区间长度为 `1` 的时候，区间里面只有一个点，所以最大值和最小值都是它自己：

```cpp
mx[i][i] = a[i].v;
mn[i][i] = a[i].v;
```

## 状态转移

枚举区间 `[l, r]`，再枚举分割点 `k`，把区间分成：

```text
[l, k] 和 [k + 1, r]
```

中间的运算符是：

```cpp
a[k + 1].e
```

### 如果是加法

加法就不需要考虑负负得正的问题：

```cpp
mx[l][r] = max(mx[l][r], mx[l][k] + mx[k + 1][r]);
mn[l][r] = min(mn[l][r], mn[l][k] + mn[k + 1][r]);
```

### 如果是乘法

乘法需要考虑最大值和最小值之间的组合：

```cpp
mx[l][k] * mx[k + 1][r]
mx[l][k] * mn[k + 1][r]
mn[l][k] * mx[k + 1][r]
mn[l][k] * mn[k + 1][r]
```

然后用这四种情况分别更新最大值和最小值。

## 环形处理

因为这是一个环，所以我们把原来的序列复制一遍：

```cpp
a[i + n] = a[i];
```

这样删除某一条边之后，就可以看成从某个点开始的一条长度为 `n` 的链。

最后枚举每一个起点：

```cpp
for (int i = 1; i <= n; i++) {
    ans = max(ans, mx[i][i + n - 1]);
}
```

如果某个起点对应的最大值等于最终答案，就输出这个起点。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long 

int n;
const int N = 105;
const int INF = 1e18;

struct Node {
    int v; 
    char e;
} a[N];

int mx[N][N];
int mn[N][N];

void solve() {
    cin >> n;

    for (int i = 1; i <= n; i++) {
        cin >> a[i].e >> a[i].v;
        a[n + i].e = a[i].e;
        a[n + i].v = a[i].v;
    }

    for (int i = 1; i <= 2 * n; i++) {
        for (int j = 1; j <= 2 * n; j++) {
            mx[i][j] = -INF;
            mn[i][j] = INF;
        }
    }

    for (int i = 1; i <= 2 * n; i++) {
        mx[i][i] = a[i].v;
        mn[i][i] = a[i].v;
    }

    for (int len = 2; len <= n; len++) {
        for (int l = 1; l + len - 1 <= 2 * n; l++) {
            int r = l + len - 1;

            for (int k = l; k < r; k++) {
                if (a[k + 1].e == 't') {
                    mx[l][r] = max(mx[l][r], mx[l][k] + mx[k + 1][r]);
                    mn[l][r] = min(mn[l][r], mn[l][k] + mn[k + 1][r]);
                } else {
                    int v1 = mx[l][k] * mx[k + 1][r];
                    int v2 = mx[l][k] * mn[k + 1][r];
                    int v3 = mn[l][k] * mx[k + 1][r];
                    int v4 = mn[l][k] * mn[k + 1][r];

                    mx[l][r] = max(mx[l][r], max(max(v1, v2), max(v3, v4)));
                    mn[l][r] = min(mn[l][r], min(min(v1, v2), min(v3, v4)));
                }
            }
        }
    }

    int ans = -INF;

    for (int i = 1; i <= n; i++) {
        ans = max(ans, mx[i][i + n - 1]);
    }

    cout << ans << '\n';

    for (int i = 1; i <= n; i++) {
        if (ans == mx[i][i + n - 1]) {
            cout << i << " ";
        }
    }

    cout << '\n';
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);

    int _ = 1;
    while (_--) {
        solve();
    }

    return 0;
}
```

## 总结

这题整体还是区间 DP，但是要注意两个点：

1. 它是一个环，所以要复制一遍数组，用环形区间 DP 处理。
2. 因为有乘法和负数，所以不能只维护最大值，还要维护最小值。
