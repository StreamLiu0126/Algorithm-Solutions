# [ABC283E] Don‘t Isolate Elements 题解

[AT_abc283_e [ABC283E] Don‘t Isolate Elements](https://www.luogu.com.cn/problem/AT_abc283_e)

### 核心思路

首先我们可以观察到，对某一行进行操作时，该行内部的“孤立”状态是没有发生改变的，只会与上下两行建立关系而改变“孤立”状态。所以当我判断第 i行是否需要翻转的时候，我只需要关心 i-1和 i+1。

因此当我们处理到第 i+1行时，就相当于对第 i 行做最终的判决，同时第 i 行的状态只由自己、i-1 和 i+1 决定。也就意味着，当我处理到第 i+1 行时，从此以后就与前 i-2 行都无关了。

对于某一个元素来说，若想判断这个元素是不是孤立的，取决于四周的四个元素。如果行内已经有了相同的邻居，那么这个元素就一定不是**孤立元素**（因为翻转不影响行内的元素相对关系）。如果没有相同的邻居，那么它只能等着上下两行有元素来救它。

### 状态定义

为了处理这种“上下”依赖，我们需要在 DP 状态里记录连续两行的操作状态：

```c++
dp[i][j][k]
```

- i：当前处理到第 i 行。
- j：第 i 行的操作（0 不翻，1 翻）。
- k：第 i-1 行的操作（0 不翻，1 翻）。

DP 表存的是达到这个状态的最小操作数量。

**为什么要把这个 k（也就是第 i-1行的操作）也给带上呢？**

当我们尝试决定第 i+1行的状态时，我们需要知道第 i 行是否已经满足题意了（即不存在孤立点）。假设第 i-1 行没有把第 i 行的元素全部救出来，那就需要第 i+1行的元素来救它了，所以我们需要知道第 i-1 行的信息来判断第 i 行的哪些元素得救了。

### 状态转移

对于每一个

```c++
 dp[i][j][k]
```

，尝试第 i+1 行的两种可能（设为 s）。每种可能我们都要检查第 i 行是否还有孤立元素。

如果有孤立元素，那么就不能转移；没有的话才可以转移：

```c++
dp[i+1][s][j] = min(dp[i+1][s][j], dp[i][j][k] + s);
```

------

### 完整代码

C++

```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long 

int h, w;
int a[1005][1005];
const int INF = 1e9;
int dp[1005][2][2];

bool check(int idx, int f1, int f0, int f2) {	
    bool ok[1005];
    for (int i = 1; i <= w; i++) ok[i] = true;
    
    for (int i = 1; i <= w; i++) {
        if (i == 1 && a[idx][1] != a[idx][2]) { ok[i] = false; continue; }
        if (i == w && a[idx][w] != a[idx][w - 1]) { ok[i] = false; continue; }
        if (i > 1 && i < w && a[idx][i - 1] != a[idx][i] && a[idx][i + 1] != a[idx][i]) ok[i] = false;
    } 
    
    for (int i = 1; i <= w; i++) {
        if (ok[i]) continue;
        int tmp1 = a[idx - 1][i];
        int tmp0 = a[idx][i];
        int tmp2 = a[idx + 1][i];
        
        if (f1) tmp1 = 1 - tmp1;
        if (f0) tmp0 = 1 - tmp0;
        if (f2) tmp2 = 1 - tmp2;

        if (idx == 1) {
            if (tmp0 != tmp2) return false;
            continue;
        }

        if (idx == h) {
            if (tmp0 != tmp1) return false;
            continue;
        }

        if (tmp0 != tmp1 && tmp0 != tmp2) {
            return false;
        }
    }

    return true;
}

signed main() {	
    ios::sync_with_stdio(false);	
    cin.tie(0);

    cin >> h >> w;
    for (int i = 1; i <= h; i++) {
        for (int j = 1; j <= w; j++) {
            cin >> a[i][j];
        }
    }

    for (int i = 0; i <= h; i++) {
        for (int j = 0; j < 2; j++) {
            for (int k = 0; k < 2; k++) {
                dp[i][j][k] = INF;
            }
        }
    }

    dp[1][0][0] = 0;
    dp[1][1][0] = 1;

    for (int i = 1; i < h; i++) {
        for (int j = 0; j < 2; j++) {     // 当前行 i 的状态
            for (int k = 0; k < 2; k++) { // 前一行 i-1 的状态
                if (dp[i][j][k] == INF) continue; // 剪枝，会稍微快一点，不要也行
                
                for (int s = 0; s < 2; ++s) { // 下一行 i+1 的状态
                    if (check(i, k, j, s)) {
                        dp[i+1][s][j] = min(dp[i+1][s][j], dp[i][j][k] + s);
                    }
                }
            }
        }
    }

    int ans = INF;
    for (int j = 0; j < 2; ++j) {
        for (int k = 0; k < 2; ++k) {
            if (check(h, k, j, 0)) { 
                ans = min(ans, dp[h][j][k]);
            }
        }
    }

    cout << (ans >= INF ? -1 : ans) << "\n";
    return 0;
}
```