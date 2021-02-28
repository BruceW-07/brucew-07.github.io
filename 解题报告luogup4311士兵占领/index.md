# [解题报告][luoguP4311]士兵占领


# Statement

[传送门](https://www.luogu.com.cn/problem/P4311)

一个 $n \times m$ 的网格，有 $K$ 个障碍格子，其余格子均可以放一个士兵。

求使得第 $i$ 行士兵数量大于等于 $L_i$，第 $j$ 列士兵数量大于等于 $C_i$ 时的最小士兵放置数量。

$n, m \le 100, K \le n \times m$。

# Solution

这题有一个比较显然的上下界网络流解法：

对每一行 $i$，从源点 $S$ 向点 $i$ 连一条下界为 $L_i$ 的边；

对每一列 $j$，从 $n + j$ 向汇点 $T$ 连一条下界为 $C_i$ 的边；

对每一行 $i$ 以及每一列 $j$， 如果格子 $(i,j)$ 不是障碍格子，则从 $i$ 向 $j$ 连一条上界为 $1$ 的边，表示在格子 $(i,j)$ 上放置一个士兵。

建完图后跑一个有源汇的上下界网络流即可。

---

但是我们会发现，这张图从 $S$ 连向 $1 \sim n$ 以及从 $n + 1 \sim n + m$ 连向 $T$ 的这些边都只有下界没有上界，那么我们是否可以对模型做一些改变，把下界改为上界，就可以使用普通最大流算法来解决这个问题了。

我们从逆向考虑这个问题。先假设一开始在所有的非障碍格子都放了一个士兵，然后接下来我们要计算的就是：在满足限制的情况下，最多能够删掉多少个士兵。

我们设 $totL_i$ 表示第 $i$ 行的非障碍格子数量，$totC_j$ 表示第 $j$ 列的非障碍格子数量。

对于每一行 $i$，从源点 $S$ 向点 $i$ 连一条容量为 $totL_i - L_i$ 的边；

对于每一行 $j$，从 $n + j$ 向汇点 $T$ 连一条容量为 $totC_i - C_i$ 的边；

对于每一行 $i$ 和每一行 $j$，从 $i$ 向 $n + j$ 连一条容量为 $1$ 的边。

在这张图上跑最大流算法，最终的答案就是 $n \times m - K - \mathrm{max\\_flow}$。


# Code
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <queue>

using namespace std;

const int _ = 2e2 + 7;
const int __ = 2e4 + 7;
const int inf = 1e9;
const int S = 2e2 + 1, T = 2e2 + 2;

int n, m, K, L[_], C[_], totL[_], totC[_], maxflow;
bool b[_][_];
int dis[_], lst[_], cur[_], nxt[__], to[__], cap[__], tot = 1;
queue<int> q;

void Add(int x, int y, int c) {
  nxt[++tot] = lst[x], to[tot] = y, cap[tot] = c, lst[x] = tot;
  nxt[++tot] = lst[y], to[tot] = x, cap[tot] = 0, lst[y] = tot;
}

bool Bfs() {
  memset(dis, 0, sizeof dis);
  while (!q.empty()) q.pop();
  dis[S] = 1, q.push(S);
  while (!q.empty()) {
    int u = q.front(); q.pop();
    for (int i = lst[u]; i; i = nxt[i]) {
      int v = to[i];
      if (dis[v] or !cap[i]) continue;
      dis[v] = dis[u] + 1, q.push(v);
      if (v == T) return 1;
    }
  }
  return 0;
}

int Extend(int u, int flow) {
  if (u == T) return flow;
  int res = flow;
  for (int &i = cur[u]; i; i = nxt[i]) {
    int v = to[i];
    if (dis[v] != dis[u] + 1 or !cap[i]) continue;
    int cst = Extend(v, min(cap[i], res));
    cap[i] -= cst, cap[i ^ 1] += cst, res -= cst;
    if (!res) break;
  }
  return flow - res;
}

void Dinic() {
  int flow;
  while (Bfs()) {
    memcpy(cur, lst, sizeof cur);
    do {
      flow = Extend(S, inf);
      maxflow += flow;
    } while (flow);
  }
}

int main() {
  cin >> n >> m >> K;
  for (int i = 1; i <= n; ++i) cin >> L[i], totL[i] = m;
  for (int i = 1; i <= m; ++i) cin >> C[i], totC[i] = n;
  for (int i = 1, x, y; i <= K; ++i) {
    cin >> x >> y;
    if (!b[x][y]) b[x][y] = 1, --totL[x], --totC[y];
  }

  for (int i = 1; i <= n; ++i) {
    if (totL[i] < L[i]) { puts("JIONG!"); exit(0); }
    Add(S, i, totL[i] - L[i]);
  }
  for (int i = 1; i <= m; ++i) {
    if (totC[i] < C[i]) { puts("JIONG!"); exit(0); }
    Add(n + i, T, totC[i] - C[i]);
  }
  for (int i = 1; i <= n; ++i)
    for (int j = 1; j <= m; ++j)
      if (!b[i][j]) Add(i, n + j, 1);

  Dinic();
  cout << n * m - K - maxflow << endl;
  return 0;
}
```

# Reference
[题解 P4311 【士兵占领】](https://www.luogu.com.cn/blog/user36456/solution-p4311) by GGN_2015。

