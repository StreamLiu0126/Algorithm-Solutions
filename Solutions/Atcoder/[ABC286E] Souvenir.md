------

# [ABC286E] Souvenir 题解

## 思路分析

这是一个单向连通图求最短路问题，同时还要维护金币的最大值。

因为这道题是不带权的，无需使用 dij 算法，用 BFS 就能搞定。而且这道题是多源最短路，我们只需要遍历 1 到 n，分别作为起点进行 BFS，同时维护二维的 `dist` 和 `ans` 数组。由于我们已经在 `dist` 和 `ans` 数组中存放了所有的点作为起点和终点的 `dist` 和 `ans` 的值，最后根据每次询问的起点和终点，输出相应的答案即可

## 细节注意

- **下标问题**：细节之一是在输入的时候字符串是 0-based，所以存放的时候要加一
- **起点初始化**：BFS 的时候，**一定要**对起点s的所有信息进行初始化后，再加入队列中

我们在扩展相邻节点 `v` 的时候，会遇到两种情况：

1. **发现更短的路**：如果 `dist[s][u] + 1 < dist[s][v]`

   意味着发现了一条更短的路。实际上通常是因为第一次访问到 `v`，因为 BFS 是均匀向外扩散的，天生就是最短路。此时要更新距离和金币，并把 `v` 加入队列。

2. **发现长度相同、路线不同**：而当 `dist[s][u] + 1 == dist[s][v]`

   说明发现长度相同，但路线不同的路。因为 BFS 是一层一层向外探索的，既然能用相同的步数走到 `v`，说明之前肯定有一个与 `u` 同层的点发现了 `v`，并且**把 `v` 放在了队列里面**。

   如果此时再把 `v` 加入队列，会让 `v` 后面的路线原封不动地探索两遍，非常的耗时。所以只需要更新金币数量即可，**不需要把 `v` 加入队列**。

C++

```c++
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int INF = 1e9;

int ans[305][305];
int coin[305];
vector<int> adj[305];
int dist[305][305];
int n;

void bfs(int s) {
    queue<int> q;
    q.push(s);
    dist[s][s] = 0;
    ans[s][s] = coin[s];
    
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        for (int v : adj[u]) {
            if (dist[s][u] + 1 < dist[s][v]) {
                ans[s][v] = ans[s][u] + coin[v];
                dist[s][v] = dist[s][u] + 1;
                q.push(v);
            } else if (dist[s][u] + 1 == dist[s][v]) {
                ans[s][v] = max(ans[s][v], ans[s][u] + coin[v]);
            }
        }
    }
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);

    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> coin[i];
    }
    
    for (int i = 1; i <= n; i++) {
        string s;
        cin >> s;
        for (int j = 0; j < (int)s.length(); j++) {
            if (s[j] == 'Y') {
                adj[i].push_back(j + 1);
            }
        }
    }

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            ans[i][j] = -1;
            dist[i][j] = INF;
        }
    }

    for (int i = 1; i <= n; i++) {
        bfs(i);
    }

    int q;
    cin >> q;
    while (q--) {
        int u, v;
        cin >> u >> v;
        if (ans[u][v] == -1) {
            cout << "Impossible" << "\n";
            continue;
        } 
        
        cout << dist[u][v] << " " << ans[u][v] << "\n";
    }

    return 0;
}
```